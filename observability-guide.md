# Observability Rehberi

Komuta Observability, bir serviste **ne olduğunu**, **nerede yavaşladığını** ve **hangi kanıta dayanarak karar verdiğinizi** tek çalışma alanında gösterir. Ekranlar yalnızca platformun gerçekten topladığı verileri sunar; veri gelmediyse boş bir grafik çizmek yerine kaynağın neden kullanılamadığını açıkça belirtir.

Bu rehber teknik uzman olmayan kullanıcıların ekranları doğru yorumlamasına yardımcı olur. CPU ve bellek gibi Kubernetes kaynak kullanımları için [İzleme ve Log Yönetimi](monitoring-logs.md), çalışma zamanı tehditleri için [Çalışma Zamanı Koruması](runtime-security-guide.md) rehberini kullanın.

---

## Observability'ye Nasıl Ulaşılır?

1. Komuta Console'da bir servisi açın.
2. Sol menüden **Observability** bölümüne girin.
3. Üst bölümde doğru **servis** ve **ortamın** seçili olduğunu kontrol edin.

Tüm sorgular sunucu tarafında bu servis ve ortamla sınırlandırılır. Bir kullanıcı yalnızca yetkili olduğu organizasyon ve servislerin verisini görebilir.

---

## Ekranlar Ne Anlatır?

### Genel Bakış

Genel Bakış, servisin son penceredeki durumunu üç temel sinyalle özetler:

- **Rate:** Gözlemlenen istek sayısı
- **Errors:** Hatalı istek sayısı ve oranı
- **Duration:** p50, p95 ve p99 gecikme değerleri

Bu üçlü kısaca **RED** olarak adlandırılır. Değerler alınan trace span'lerinden üretilir. Trace toplama politikası örnekleme uyguluyorsa istek sayısı tüm gerçek trafiğin toplamı değil, **alınan örneklenmiş trafiğin sayısıdır**. Ekrandaki “Örneklenmiş” veya “Nitelikli” etiketi bu nedenle önemlidir.

Genel Bakış ayrıca servis bağımlılıklarını gösterir. “Öne çıkan bulgular” veya “değişiklik zaman çizelgesi” için kanıt henüz üretilemiyorsa bölüm bunu açıkça “kullanılamıyor” olarak belirtir. Bu durum servisin sağlıklı veya sağlıksız olduğu anlamına gelmez.

### İncele

**İncele** bölümü, Komuta'nın kanıta dayalı olarak oluşturduğu inceleme kayıtlarını listeler. Bir inceleme şunları bir araya getirebilir:

- Ne oldu?
- Kullanıcı veya servis üzerindeki etkisi neydi?
- Hangi değişikliklerle aynı zamana denk geldi?
- En güçlü kanıt ve destekleyici kanıtlar neler?
- Hangi kanıtlar eksik?
- Önerilen sonraki adım ve doğrulama penceresi nedir?

Bir aday neden, kesin kök neden olarak sunulmaz. Kanıt yetersizse güven düzeyi ve eksik parçalar görünür kalır.

### Trace

Trace ekranı bir isteğin servis içindeki yolculuğunu span'ler halinde gösterir. Arama bölümünde rota, HTTP metodu, hata durumu ve süre aralığıyla daraltma yapabilirsiniz.

Bir trace seçildiğinde:

- Span şelalesi ve başlangıç sırası
- Her span'in süresi ve kendi çalışma süresi
- Hata durumu ve durum mesajı
- Güvenli olarak dışa aktarılan özellikler ve olaylar
- Saat uyuşmazlığı, eksik span veya kırpılmış veri gibi kalite uyarıları
- Trace kimliğiyle ilişkilendirilebilen loglar

gösterilir.

Trace'lerde örnekleme uygulanabilir. Hata ve yavaşlık gibi önemli sinyaller daha yüksek öncelikle tutulurken normal trafik politika gereği örneklenebilir. Bu nedenle trace sayısını doğrudan “toplam kullanıcı isteği” olarak yorumlamayın.

### Log

Log ekranı merkezi log deposundaki servis kayıtlarını arar. Şu filtreler birlikte kullanılabilir:

- Zaman aralığı
- Seviye: fatal, error, warning, info, debug, trace veya bilinmeyen
- Namespace, pod ve container
- Metin içerir / regex ile eşleşir
- Metni veya regex eşleşmesini hariç tutar
- Desteklenen ilişkilendirme alanları: trace, request veya correlation kimliği

Histogram ve log tablosu aynı filtreleri kullanır. Böylece grafikteki yoğunluk ile alttaki kayıtlar aynı soruyu yanıtlar. “Daha eski kayıtları yükle” işlemi imleçli sayfalama kullanır; dışa aktarma yalnızca kullanıcının erişebildiği sonuçları içerir.

> Log mesajlarında uygulamanızın yazdığı içerik bulunur. Şifre, erişim anahtarı, kişisel veri veya oturum belirteci loglamayın.

### Trace Türevli RED

Bu ekran, Genel Bakış'taki RED verisini endpoint bazında daha ayrıntılı gösterir:

- Beş dakikalık kovalar halinde trafik ve hata
- Endpoint başına istek ve hata oranı
- En kötü kovanın p50, p95 ve p99 değeri
- Gecikme özeti üretilemeyen kovalar için “sketch yok” uyarısı

Bu ekran **CPU, bellek, disk, node kapasitesi veya Prometheus metriği göstermez**. Bu kaynaklar servisinizin **Resources** ve ilgili cluster ekranlarında izlenir.

### Bağımlılıklar ve L7

Komuta Flow Agent, servise ait gözlemlenmiş ağ akışlarını toplar. Ekran şu bilgileri sunabilir:

- Kaynak ve hedef servis/pod/namespace
- Protokol ve hedef port
- İzin verilen veya düşürülen akış
- HTTP metodu, URL/rota, durum kodu ve gözlemlenmiş gecikme
- DNS sorgusu ve yanıt kodu
- Trafik yönü ve node

Akış kayıtları byte veya paket sayısı taşımaz. Bu nedenle ağ bant genişliği, paket hızı veya aktarım boyutu bu ekrandaki flow sayısından hesaplanmaz. “HTTP flow” sayısı da trace tabanlı istek sayısıyla aynı ölçüm değildir.

### Uç Noktalar

Uç Noktalar ekranı endpoint'leri istek sayısı, hata oranı ve en kötü gecikme kovasına göre sıralar. Bir endpoint'ten Trace ekranına geçerek ilgili örnekleri inceleyebilirsiniz.

Bir zaman kovasında veri yoksa Komuta bunu otomatik olarak sıfır kabul etmez. Özellikle yeni bir rollup şeması devreye alındığında kapsam başlangıcı grafikte boşluk olarak görülebilir.

### Güvenilirlik

#### SLO ve Hata Bütçesi

SLO, bir servisin belirli bir dönem için verdiği başarı sözüdür. Örneğin “bu endpoint'in isteklerinin %99,9'u başarılı olmalı” hedefi tanımlanabilir.

Komuta:

- Servis geneli veya endpoint bazında SLO oluşturur
- Ölçülen başarı oranını gösterir
- Kalan ve tüketilen hata bütçesini hesaplar
- Bütçenin hızlı tükendiği veya tamamen bittiği durumları ayırır
- Telemetri eksikse durumu “sağlıklı” varsaymak yerine **Bilinmiyor** olarak gösterir

SLO oluşturma veya düzenleme, ayrı bir yetki gerektirebilir.

#### Anomaliler

Anomali ekranı güncel endpoint sinyallerini hareketli geçmiş taban çizgisiyle karşılaştırır. Başlangıç döneminde taban çizgisi yeterince dolmadıysa “ısınıyor” uyarısı gösterilir; erken sonuçlar daha düşük güvenle değerlendirilir.

Bir anomali, tek başına arıza veya saldırı kanıtı değildir. Sapmanın büyüklüğü, zaman aralığı ve ilgili trace/log kanıtları birlikte incelenmelidir.

#### Sürüm Etkisi

Sürüm Etkisi, deployment öncesi ve sonrası ölçüm pencerelerini karşılaştırır. Hata oranı, gecikme ve endpoint değişimleri aynı zaman çizelgesinde gösterilebilir.

- Çakışan deployment'lar görünür kalır.
- Telemetri eksikse skor verilmez.
- İlişkilendirme verisi yoksa deployment'ın değişime neden olduğu iddia edilmez.
- Güven değeri bilinmiyorsa “yüksek” varsayılmaz.

### Optimize Et

#### Maliyet ve Telemetri Verimliliği

Organizasyon bağlamında servis maliyeti, mevcut faturalama dönemindeki ölçülmüş sayaçlardan dağıtılır. “Kesinleşen zaman” bilgisi, hangi ana kadar fatura verisinin tamamlandığını gösterir.

Telemetri verimliliği analizi; örneklenen gün sayısı, istekler, benzersiz yollar ve depolama satırları gibi ölçümleri kullanabilir. Yapay zekâ tarafından üretilen başlık ve öneriler **öneridir**; ölçüm tablosu ve uygulanan kurallar ayrı gösterilir.

#### Talep Tahmini

Talep Tahmini geçmiş istek hacminden kısa ve uzun vadeli talep projeksiyonu üretir. Güven aralığı ve geçmiş kapsam yetersizse bu durum ekranda belirtilir.

Bu yüzey yalnızca **talebi** tahmin eder. CPU/bellek limiti, HPA tavanı, node boşluğu veya “kaç gün sonra kapasite dolar” gibi sonuçlar üretmez.

### Panolar

Panolar, Komuta'nın izin verdiği widget kataloğundan kalıcı servis panoları oluşturmanızı sağlar. Yetkinize bağlı olarak pano oluşturabilir, düzenleyebilir veya salt okunur görüntüleyebilirsiniz.

Kaydedilmiş bir panonun sorgusu başarısız olursa ekran bunu “pano yok” şeklinde göstermez; hata ve yeniden deneme seçeneği sunar. Bilinmeyen ya da artık desteklenmeyen widget türleri çalıştırılmaz.

### Doğal Dille Telemetri Sorgusu

Bu yüzeyde servisiniz hakkında Türkçe veya İngilizce soru sorabilirsiniz. Komuta soruyu izin verilen veri kümelerinde, tenant ve servis kapsamlı **salt okunur** sorguya dönüştürür.

Sonuçta:

- Gerçek veri tablosu
- Çalıştırılan salt okunur SQL
- Sorgu süresi ve araç bilgisi
- Yapay zekâ özeti veya hata açıklaması

görülebilir. Yapay zekâ anlatımı kanıt değildir; karar verirken sonuç tablosunu ve sorguyu esas alın.

### Veri Sağlığı

Veri Sağlığı ekranı “servis sağlıklı mı?” sorusunu değil, “bu ekrandaki veriye güvenebilir miyim?” sorusunu yanıtlar.

Gösterilen başlıca bilgiler:

- Gateway, şema ve politika sürümü
- Veri kabulü ve sorgu katmanının hazır olup olmadığı
- Flow, trace ve runtime-security kaynaklarının durumu
- Gözlemlenen, kabul edilen, reddedilen, tamponlanan ve kaybedilen kayıt sayıları
- Kaynak watermark değeri ve son kabul zamanı
- Kapsam oranı ve eksikliği açıklayan neden kodları

Reddedilen veya kaybedilen kayıt varsa ekrandaki sonuçlar “tam” kabul edilmez. Boş sonuç, yalnızca kaynak sağlıklı ve kapsam yeterliyse “gözlenmedi” olarak yorumlanabilir.

---

## Kanıt Etiketlerini Okumak

Komuta Observability her yönetilen yanıtta kanıt durumunu taşır:

| Etiket | Anlamı |
|--------|--------|
| **Tam** | Sorgunun gerektirdiği veri kaynakları pencere için kullanılabildi |
| **Kısmi** | Sonuç var, ancak en az bir gerekli kaynak veya zaman dilimi eksik |
| **Kullanılamıyor** | Güvenilir sonuç üretmek için yeterli kaynak okunamadı |
| **Kesin** | Sonuç, kapsam içindeki kayıtların tamamından hesaplandı |
| **Örneklenmiş** | Sonuç, kabul edilen örneklenmiş telemetriden hesaplandı |
| **Yaklaşık** | Sonuçta yaklaşık hesap veya alt/üst sınır kullanıldı |

**Kapsam yüzdesi**, gerekli kaynakların ne kadarının yanıt verdiğini gösterir. **Watermark**, kaynağın güvenilir biçimde tamamlandığı son zamanı belirtir. Bir sorgu kimliği, destek ekibinin aynı sorguyu izleyebilmesini sağlar.

---

## Veri Komuta'ya Nasıl Gelir?

```text
Uygulama ve cluster sensörleri
        │
        ├─ Trace: OpenTelemetry / Alloy
        ├─ Ağ akışları: Hubble / Komuta Flow Agent
        ├─ Runtime olayları: Tetragon ve KubeArmor / Security Agent
        └─ Loglar: Loki toplama hattı
        │
        ▼
Komuta Telemetry Gateway
        │  doğrulama, tenant/servis sahipliği, politika ve kayıt muhasebesi
        ▼
ClickHouse (telemetri) + Loki (log) + PostgreSQL (inceleme kayıtları)
        │
        ▼
Komuta API
        │  yetki, kapsam, sorgu sınırı ve kanıt zarfı
        ▼
Komuta UI
```

Arayüz ClickHouse veya Loki'ye doğrudan bağlanmaz. Her sorgu Komuta API ve Gateway üzerinden yetkilendirilir, servis kapsamına alınır ve süre/satır sınırlarına tabi tutulur.

---

## Saklama Süreleri

Varsayılan telemetri saklama politikası veri türüne göre değişir:

| Veri türü | Varsayılan ham veri saklama |
|-----------|-----------------------------|
| Ağ akışları | 30 gün |
| Runtime-security olayları | 30 gün |
| Trace span'leri | 14 gün |

Beş dakikalık veya saatlik özet tablolar bazı ürün ekranları için daha uzun süre tutulabilir. Log saklama süresi organizasyon planına ve log deposu politikasına bağlıdır. Arayüzde seçilebilen tarih aralığı, ilgili API'nin güvenli sorgu sınırıyla ayrıca kısıtlanabilir.

---

## Gizlilik ve Güvenli Kullanım

- Runtime-security kanıtı saklanmadan önce hassas anahtarlar temizlenir ve büyük içerik kırpılabilir. Ekran kırpılmış veya bozuk kanıtı ayrıca işaretler.
- Flow kayıtları pod/IP, DNS sorgusu ve HTTP URL'si gibi hassas olabilecek alanlar taşıyabilir. Observability yetkilerini yalnız ihtiyaç duyan kullanıcılara verin.
- Uygulama logları geliştiricinin yazdığı mesajı taşır; secret ve kişisel veri loglamayın.
- Trace özellikleri ve olayları boyut, adet ve güvenlik politikalarıyla sınırlandırılır.
- Dışa aktarılan raporları üretim verisiyle aynı gizlilik seviyesinde saklayın.

---

## Hızlı Sorun Giderme

### Grafik boş ama servis trafik alıyor

1. **Veri Sağlığı** ekranını açın.
2. İlgili kaynağın hazır ve kabul ediyor olduğunu doğrulayın.
3. Watermark'ın seçilen zaman aralığına ulaşıp ulaşmadığını kontrol edin.
4. Kapsam kısmi veya kullanılamıyor ise neden kodunu destek ekibiyle paylaşın.

### Trace sayısı gerçek istek sayısından düşük

Trace toplama politikası örnekleme uygulayabilir. Toplam iş yükü için trace sayısını değil, uygun RED veya trafik sinyalini ve ekrandaki örnekleme etiketini birlikte değerlendirin.

### Log histogramı ile tablo uyuşmuyor

Aynı zaman aralığı ve filtrelerin etkin olduğundan emin olun. Filtre değişikliğinden sonra sorguyu yenileyin. Sorun devam ederse sorgu kimliğini destek ekibine iletin.

### “Sağlıklı” yerine “Bilinmiyor” görünüyor

Komuta eksik telemetriyi sağlıklı varsaymaz. Veri Sağlığı'nda kaynak kapsamını kontrol edin; veri tamamlandığında durum yeniden hesaplanır.

### Talep tahmini yüklenemiyor

Tahmin için yeterli geçmiş olmayabilir veya ilgili sorgu rotası kullanılamıyor olabilir. Bu durum kapasitenin dolduğu anlamına gelmez.

---

## İlgili Dokümanlar

- [İzleme ve Log Yönetimi](monitoring-logs.md)
- [Uyarı Yönetimi](alert-guide.md)
- [Çalışma Zamanı Koruması](runtime-security-guide.md)
- [Güvenlik Merkezi](security-center-guide.md)
- [Erişim Kontrolü](access-control-guide.md)

