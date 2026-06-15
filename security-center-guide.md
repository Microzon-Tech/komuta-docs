# Güvenlik Merkezi (Security Center)

Komuta Güvenlik Merkezi, tüm kümeleriniz ve servisleriniz genelindeki güvenlik durumunu **tek bir konsoldan** izlemenizi ve yönetmenizi sağlar. Çalışma zamanı telemetrisine dayanır: workload'larınızın içinde gerçekte ne olduğunu (hangi process çalıştı, hangi dosyaya erişildi, hangi ağ bağlantısı kuruldu) kernel seviyesinde gözlemler, bulgulara dönüştürür ve aksiyon almanızı sağlar.

Sol menüden **Güvenlik** bölümüne tıklayarak erişirsiniz.

---

## Güvenlik Merkezi Nasıl Çalışır?

```
Çalışma Zamanı Sensörleri  →  Telemetri Toplama  →  Bulgu Üretimi  →  Aksiyon
(Çalışma-zamanı Koruması,     (Komuta Telemetry     (tekilleştirme,    (karar, bildirim,
 Çalışma-zamanı İzleme,        Engine)                risk skoru)        SIEM, uyumluluk)
 Ağ Akış Gözlemi, Posture)
```

1. Kümelerinize kurulan **çalışma zamanı sensörleri** container içi davranışları (process, dosya, ağ) ve duruş (posture) bilgilerini toplar.
2. Olaylar **tekilleştirilir**: aynı mantıksal olay tekrar ettiğinde yeni kayıt açılmaz, mevcut bulgunun sayacı ve "son görülme" zamanı güncellenir.
3. Her servis için **0–100 arası risk skoru** hesaplanır ve 5 dakikada bir güncellenir.
4. Bulgular; bildirim kanallarına, SIEM sisteminize ve uyumluluk panosuna otomatik akar.

### Temel Kavramlar

Güvenlik Merkezi'nde dört ayrı kavram göreceksiniz. Aralarındaki fark önemlidir:

| Kavram | Ne anlama gelir? | Nerede görünür? |
|--------|------------------|-----------------|
| **Gözlem (Observation)** | Audit modundaki bir serviste sensörün *gördüğü* ham davranış (henüz karar verilmemiş) | Servis detayı → Güvenlik → Gözlemler |
| **İhlal (Violation)** | Bir koruma politikasına takılan davranışların canlı akışı | Güvenlik → İhlaller |
| **Bulgu (Finding)** | Tekilleştirilmiş, üzerinde karar alınabilir kalıcı güvenlik kaydı | Güvenlik → Bulgular |
| **Kanıt (Evidence)** | Bir bulgu veya kararla ilişkili zaman damgalı olay geçmişi | Servis detayı → Güvenlik → Bulgular (zaman çizelgesi) |

Kısaca: **gözlemler** öğrenme aşamasının ham verisidir, **ihlaller** "şu an ne tespit ediliyor" akışıdır, **bulgular** ise yönettiğiniz iş listesidir.

---

## Genel Bakış

Güvenlik Merkezi'nin açılış sayfası, tenant genelindeki güvenlik durumunu özetler.

### Risk Skoru (0–100)

Sol üstteki risk kartı tüm servislerinizin birleşik risk seviyesini gösterir. Skor **açıklanabilirdir** — kara kutu değildir; her bileşenin katkısı ayrı ayrı izlenebilir:

- **Bulgu şiddeti**: Critical bulgular en yüksek, Info en düşük ağırlığı taşır.
- **Kaynak güvenilirliği**: Kernel seviyesi gözlemler, duruş (posture) kontrollerinden daha yüksek güven katsayısıyla değerlendirilir.
- **Tazelik**: Eski bulguların etkisi zamanla azalır (24 saatlik yarılanma eğrisi).
- **Etki alanı**: Servisin katmanı (kritiklik seviyesi), internete açık endpoint'i ve ayrıcalıklı kimlik kullanımı skoru artırır.
- **Tekrar faktörü**: Aynı bulgunun tekrar sayısı logaritmik olarak skora eklenir.

Hiç bulgusu olmayan bir servis, yalnızca internete açık olduğu için "riskli" sayılmaz — skor gürültü üretmeyecek şekilde tasarlanmıştır.

### Diğer Genel Bakış Kartları

| Kart | İçerik |
|------|--------|
| **Açık Bulgular** | Toplam açık bulgu sayısı + critical/high kırılımı |
| **Kümeler / Servisler** | Bağlı küme sayısı ve senkronizasyon durumu |
| **Kritik Bulgular** | Critical ve High şiddetindeki açık bulguların canlı listesi |
| **En Riskli Servisler** | Risk skoruna göre sıralı servis listesi; tıklayınca servisin güvenlik sayfasına gider |
| **Küme Güvenlik Matrisi** | Küme başına sürüm, node durumu, platform koruma katmanları (Baseline rozetleri) ve çevrimiçi durumu |
| **Servis Sertleştirme** | Her servis için 8 koruma katmanının uygulanma yüzdesi |

**Servis sertleştirme katmanları**: Ağ Politikası, Hız Limiti (Rate Limit), Çalışma-zamanı Koruması, TLS, Kaynak Kotası, Limit Aralığı, Servis Hesabı sıkılaştırması ve Root-olmayan çalıştırma. Her nokta üzerine gelerek hangi katmanın eksik olduğunu görebilirsiniz.

Sayfanın sağ üstündeki **"Yakın gerçek zamanlı · telemetri temelli"** rozeti, verilerin canlı telemetri akışından geldiğini belirtir.

---

## Bulgular (Findings)

Tenant genelindeki tüm güvenlik bulgularının yönetildiği ana ekrandır.

### Şiddet Seviyeleri

| Seviye | Anlamı |
|--------|--------|
| **Critical** | Acil müdahale gerektiren, aktif istismar göstergesi olabilecek bulgu |
| **High** | Kısa sürede ele alınması gereken önemli risk |
| **Medium** | Planlı şekilde giderilmesi gereken risk |
| **Low / Info** | Farkındalık amaçlı, düşük öncelikli kayıt |

### Bulgu Yaşam Döngüsü

```
Open ──► Acknowledged ──► Allowed / Blocked / Dismissed ──► Resolved (kapanış)
```

| Karar | Ne yapar? |
|-------|-----------|
| **Acknowledge** | "Gördüm, inceliyorum" işareti — bulgu açık kalır |
| **Allow** | Davranış meşru kabul edilir; **gerekçe zorunludur** |
| **Block** | Davranışın engellenmesi gerektiği kararı; **gerekçe zorunludur** |
| **Dismiss** | Bulgu geçersiz/önemsiz olarak kapatılır |
| **Resolve** | Kalıcı kapanış (geri açılamaz; aynı davranış tekrar ederse yeni bulgu oluşur) |

Önemli noktalar:

- Kararlar **tekil veya toplu** verilebilir (tek seferde en fazla 200 bulgu).
- Allow/Block kararları yalnızca yetkili rollere açıktır; **geliştirici rolü** yalnız Acknowledge ve Dismiss yapabilir.
- Her karar, kim tarafından ve hangi gerekçeyle verildiği bilgisiyle birlikte **değiştirilemez denetim zincirine** (bkz. [Denetim Deposu](#denetim-deposu-audit-storage)) kaydedilir.
- Aynı mantıksal bulgu tekrar tespit edildiğinde yeni satır açılmaz; **Tekrar sayısı** artar ve **Son görülme** güncellenir. Böylece 400.000 kez tekrar eden bir olay tek satırda yönetilir.

### Filtreleme

Arama kutusu, kaynak filtresi (Çalışma-zamanı Koruması, Çalışma-zamanı İzleme, Ağ Akış Gözlemi, Posture, Identity, Küme Sağlığı) ve şiddet filtresiyle bulgu listesini daraltabilirsiniz. Üstteki istatistik çipleri (Toplam / Açık / Critical / High) anlık durumu gösterir.

---

## İhlaller (Violations)

Koruma politikalarına takılan davranışların **canlı akışını** gösterir: hangi pod'da, hangi process, hangi aksiyonla (Engellendi / Denetlendi) yakalandı.

- Bulgular ekranından farkı: ihlaller ham ve anlıktır; bulgular tekilleştirilmiş ve karar alınabilir kayıtlardır.
- Sağ üstteki **zil rozeti** (tüm sayfalarda görünür) Critical + High ihlal sayacını canlı gösterir; kritik artışta anlık bildirim düşer.

---

## Politikalar

Tüm kümelerinizdeki güvenlik politikalarının birleşik görünümü.

### Politika Türleri

| Tür | Kapsam | Durum |
|-----|--------|-------|
| **Çalışma-zamanı Koruması** | Container içi process/dosya/ağ davranış kuralları | Aktif |
| **Ağ Politikası** | Servisler arası trafik kuralları | Yakında bu ekranda |
| **Çalışma-zamanı İzleme** | Gelişmiş davranış gözlem kuralları | Yakında bu ekranda |

Her politika satırında servis/küme bilgisi, uygulanma durumu ve aksiyon (Block/Audit) görünür; detay panelinden kural içeriğini inceleyebilirsiniz. Politika kavramlarının ayrıntısı için bkz. [Çalışma Zamanı Koruması](runtime-security-guide.md).

### Önerilen Politikalar (Suggested Policies)

Platform, servislerinizin gerçek davranışını analiz ederek **politika önerileri** üretir. Örnek gerekçe: *"Bu serviste hassas dosya okuma girişimi son 24 saatte 47 kez engellendi — kalıcı kural önerilir."*

Öneri yaşam döngüsü: **Beklemede → Kabul Edildi → Uygulandı**. Beklemedeki öneriler reddedilebilir veya 30 gün sonra otomatik olarak süresi dolar. Uygulanmış bir öneri gerektiğinde geri alınabilir (rollback). Tüm geçişler denetim zincirine yazılır.

### Politika İstisnaları (Policy Exceptions)

Meşru ama politikaya takılan bir iş akışı için **süreli istisna** tanımlayabilirsiniz (ör. bakım penceresi, üçüncü parti aracın bilinen davranışı):

- İstisnalar onay akışından geçer: **Beklemede → Onaylandı → Aktif → Süresi Doldu / İptal Edildi**.
- Azami istisna süresi **90 gündür** — kalıcı delik açılamaz.
- Her istisna gerekçeli ve denetim zinciri kayıtlıdır.

---

## Denetim Kaydı (Audit Log)

Platform yönetim düzlemindeki olayların birleşik zaman çizelgesi: kim, ne zaman, hangi işlemi yaptı, sonuç ne oldu (başarılı / hata).

- Kaynak çipleriyle filtreleyin: Identity, Çalışma-zamanı Koruması, Ağ Akış Gözlemi, Posture, Denetim vb.
- Şiddet alt sınırı ve tarih penceresi seçilebilir.
- API çağrıları ve kimlik doğrulama sunucusu istekleri (4xx/5xx hatalar dahil) bu ekranda izlenir.

---

## Oturum Aktivitesi (Login Activity)

Hesabınıza ait kimlik olaylarının kaydı:

- Başarılı / başarısız girişler, oturum kapatmalar, hesap kilitlenmeleri
- Her kayıtta IP adresi, tarayıcı/istemci bilgisi ve zaman damgası
- **CSV dışa aktarma** ile kayıtları raporlayabilirsiniz
- Şüpheli bir girişten, ilgili zaman dilimindeki diğer güvenlik olaylarına **adli inceleme bağlantısıyla** atlayabilirsiniz

Bu ekran, "bu saatte kim giriş yaptı?" ve "başarısız giriş denemesi var mı?" sorularının ilk adresidir.

---

## Olay Müdahale Playbook'ları (IR Playbooks)

Bir güvenlik bulgusuyla karşılaştığınızda **adım adım ne yapmanız gerektiğini** anlatan hazır müdahale rehberleridir. Bulgunun şiddeti, türü ve kaynağıyla otomatik eşleşir.

Platformla birlikte gelen hazır playbook'lar:

| Playbook | Tetikleyici | Önerilen adımlar |
|----------|------------|------------------|
| **Hassas dosya okuma** | Critical çalışma-zamanı bulgusu | Olay aç → Pod'u izole et → Servis sahibini çağır → Adli paket hazırla |
| **Servis hesabı token'ı + dış trafik** | Critical çalışma-zamanı bulgusu | Olay aç → Pod'u izole et → Kimlik bilgilerini döndür → Ağ engeli öner → Sahibi çağır |
| **Deploy sonrası shell çalıştırma** | Medium bulgu | Sağlık probe'unu doğrula → İnceleme için işaretle |
| **Çalışma zamanında paket yöneticisi** | High bulgu | Olay aç → Engelleme kuralı öner → İnceleme için işaretle |
| **Bulut metadata erişimi** | High ağ bulgusu | Olay aç → Ağ engeli öner → Kimlik bilgilerini döndür → Sahibi çağır |

- Hazır playbook'lar düzenlenemez; ancak **klonlayıp kendi sürecinize göre özelleştirebilirsiniz**.
- Playbook adımları şu an **rehber niteliğindedir** — adımları ekibiniz uygular; otomatik yürütme yol haritasındadır.

---

## Uyumluluk (Compliance)

Güvenlik bulgularınız ve koruma katmanlarınız, sektör standardı uyumluluk kontrollerine **otomatik olarak kanıt** sağlar. Ayrı bir tarama aracı kurmanız gerekmez.

### Desteklenen Çerçeveler

| Çerçeve | Örnek kontroller |
|---------|------------------|
| **CIS Kubernetes** | Ağ politikası varlığı, ayrıcalıklı pod denetimi |
| **NIST 800-53** | Erişim kontrolü (AC-3), denetim olayları (AU-2, AU-9), sistem izleme (SI-4) |
| **SOC 2** | Mantıksal erişim (CC6.1), sistem izleme (CC7.2) |
| **ISO 27001** | İzleme faaliyetleri (A.8.16), ağ güvenliği (A.8.34) |
| **PCI-DSS** | Denetim kaydı (10.2.1), değişiklik tespiti (11.5.1) |

### Kapsam Durumları

Her kontrol için son çeyreklik kanıt penceresine göre durum hesaplanır:

- **Karşılanıyor (Met)** — kontrol için yeterli kanıt mevcut
- **Kısmen Karşılanıyor (Partially Met)** — kanıt var ama eksik
- **Sinyal Eksik (Missing Signal)** — bu kontrol için henüz veri kaynağı bağlı değil
- **Uygulanamaz / Değerlendirilmedi**

### Aksiyonlar

- **Kanıt Dışa Aktar**: Denetçinize sunmak üzere JSON veya CSV formatında kanıt paketi indirin.
- **Yeniden Değerlendir**: Bir çerçeveyi takvim dışı, anında yeniden değerlendirin (ör. denetim öncesi).

---

## Bildirimler ve Yönlendirme Kuralları

Güvenlik bulgularının ekibinize nasıl ulaşacağını burada yapılandırırsınız.

### Bildirim Kanalları

| Kanal | Açıklama |
|-------|----------|
| **Uygulama içi** | Konsol içinde anlık bildirim |
| **E-posta** | Tanımlı adreslere e-posta |
| **Slack** | Slack webhook entegrasyonu |
| **Microsoft Teams** | Teams kanal kartı |
| **PagerDuty** | Nöbet (on-call) tetikleme |
| **Webhook** | Kendi sisteminize HTTPS POST |

Kanal yapılandırmaları (webhook adresi, token vb.) şifrelenerek saklanır ve salt-okunur kullanıcılara gösterilmez.

### Yönlendirme Kuralları

Hangi bulgunun hangi kanala gideceğini kurallarla belirlersiniz:

- **Şiddet eşiği**: ör. yalnız High ve üzeri
- **Kaynak / küme / servis / sahip ekip** filtreleri
- **Mesai saatleri penceresi**: bildirimleri çalışma saatlerinize sınırlayın (varsayılan 09:00–18:00, yerel saat)
- **Eskalasyon**: Bulgu belirlenen süre içinde (en fazla 24 saat) çözülmezse eskalasyon kanallarına ikinci bildirim gider
- **Tekilleştirme**: Aynı bulgu için tekrar tekrar bildirim gönderilmez

---

## SIEM Dışa Aktarımı

Güvenlik bulgularınızı kendi SIEM / log platformunuza otomatik aktarın. Mevcut güvenlik operasyon merkezinizle (SOC) entegrasyon için tasarlanmıştır.

| Şema | Uyumlu hedef örnekleri |
|------|------------------------|
| **OCSF** | Splunk, AWS Security Hub, Snowflake |
| **Elastic ECS** | Elasticsearch / Kibana |
| **OpenTelemetry** | OTel Collector, Honeycomb |

Çalışma şekli ve güvenlik garantileri:

- Bulgular **5 dakikada bir** artımlı olarak (kaldığı yerden) hedefinize gönderilir.
- Hedef adres **HTTPS zorunludur**; kimlik doğrulama token'ı şifrelenerek saklanır ve arayüzde asla geri gösterilmez.
- **5 ardışık hata** sonrasında aktarım otomatik devre dışı kalır ve bildirim üretilir.
- Yapılandırma değişikliklerinin tümü denetim zincirine kaydedilir.

---

## Sentetik Saldırı Tatbikatları (Synthetic Attacks)

Güvenlik izlemenizin **gerçekten çalıştığını kanıtlayın**. Platform, kendi workload'unuzda kontrollü ve zararsız bir "saldırı sinyali" tetikler; tespit hattının bunu yakalayıp yakalamadığını ve **ne kadar sürede yakaladığını** ölçer.

### Ne işe yarar?

Klasik güvenlik araçlarında "alarm gelmiyor" iki anlama gelebilir: ya her şey yolundadır ya da izleme bozuktur. Tatbikatlar bu belirsizliği ortadan kaldırır — **tespit kapsaması boşluklarını** saldırgan bulmadan önce siz bulursunuz.

### Senaryo Kataloğu

Hazır senaryolar arasında şüpheli process çalıştırma, hassas dizine yazma, kısıtlı workload'dan dış trafik ve ayrıcalıklı pod denemesi bulunur. Her senaryonun bir **tespit SLA'sı** vardır (ör. 60 saniye).

### Tatbikat Çalıştırma

1. **Sentetik Saldırılar** sayfasından bir senaryo seçin ve hedef servisinizi belirleyin.
2. **Tatbikatı Tetikle** deyin.
3. Sonuç panosunda izleyin: **Tespit Edildi** (SLA içinde) / **Tespit Edilemedi** (kapsama boşluğu!).

Üstteki metrik kartları toplam tatbikat sayısını, tespit oranını, SLA karşılama oranını ve ortalama tespit süresini gösterir.

> Tatbikat kayıtları gerçek bulgulardan **izole** tutulur: risk skorunuzu, SIEM akışınızı ve uyumluluk kanıtlarınızı kirletmez.

---

## Honey Path'ler (Tuzak Yollar)

Bir servisinizde **tuzak dosya yolu** tanımlayın: meşru kodunuzun asla dokunmaması gereken bir yol (ör. sahte bir kimlik bilgisi dosyası). Bu yola yapılan **herhangi bir erişim** anında Critical bulgu üretir ve ilgili olay müdahale playbook'unu önerir.

- Servis seçici → tuzak yolu ekleyin (ör. `/app/config/.fake-credentials`).
- **Son tetiklenme** ve **tetiklenme sayısı** kolonlarından hangi tuzakların trafik yakaladığını izleyin.
- Tuzak yollar, içeriden yanal hareket (lateral movement) ve otomatik tarama davranışlarını yakalamada düşük maliyetli, yüksek sinyalli bir yöntemdir.

---

## Denetim Deposu (Audit Storage)

Güvenlikle ilgili her operatör kararı (bulgu kararları, politika önerisi geçişleri, istisnalar, SIEM yapılandırmaları, tatbikat tetiklemeleri, yetenek/yazılabilir dizin değişiklikleri) **değiştirilemez bir denetim zincirine** yazılır.

- **Eklenebilir-yalnız (append-only)**: Kayıtlar güncellenemez ve silinemez.
- **Kriptografik zincir**: Her kayıt bir önceki kaydın özetini içerir; geçmişte tek bir kayıt değiştirilse zincirin kalanı geçersizleşir ve doğrulayıcı bunu tespit eder. Denetçiye "bu kayıtlarla oynanmadı" garantisi verirsiniz.
- **Yasal Saklama (Legal Hold)**: Hukuki bir süreç sırasında kayıtların saklama süresi dolsa bile silinmesini engelleyin (gerekçeli aç/kapat).
- **Arşiv hedefi**: Kayıtların uzun süreli arşiv kopyası için depolama hedefi ve saklama süresi yapılandırılabilir.

Bu yapı; SOC 2, ISO 27001 ve PCI-DSS denetimlerindeki "denetim kaydı bütünlüğü" gereksinimlerini doğrudan karşılamak için tasarlanmıştır.

---

## Diğer Sayfalar

| Sayfa | İçerik |
|-------|--------|
| **Çalışma Zamanı Yaptırımı** | Platform genelinde İzleme (Monitor) ↔ Yaptırım (Enforce) modu; mod değişimi gerekçe ister ve hazırlık kontrolünden geçer. Yöneticiler içindir. |
| **Küme Sağlığı** | Altyapı katmanı sağlık bulguları (depolama, node bileşenleri). |
| **Saklama Politikası** | Güvenlik verilerinin saklama süresi ve veri ikamet (residency) ayarları. Yönetici yetkisi gerektirir. |

---

## İzinler

Güvenlik Merkezi yetenekleri, **Erişim Kontrolü → Roller** üzerinden ayrı ayrı yetkilendirilir:

| Yetenek | Kim kullanmalı? |
|---------|-----------------|
| Güvenlik Merkezi okuma (genel bakış, bulgular, ihlaller) | Tüm geliştirici ekip |
| Bulgu kararları (Allow/Block/Resolve) | Güvenlik operatörü / takım lideri |
| Bulgu kararları (Acknowledge/Dismiss) | Geliştirici |
| Bildirim kanalı ve yönlendirme kuralı yönetimi | Güvenlik operatörü |
| Politika istisnası ve önerilen politika yönetimi | Güvenlik operatörü |
| SIEM dışa aktarım yapılandırması | Güvenlik operatörü |
| Uyumluluk görüntüleme / kanıt dışa aktarma | Denetçi (auditor) rolü |
| Sentetik tatbikat tetikleme, honey path yönetimi | Güvenlik operatörü |
| Denetim deposu görüntüleme | Denetçi / güvenlik operatörü |
| Saklama politikası, yaptırım modu, arşiv yönetimi | Platform yöneticisi |

En iyi uygulama: geliştiricilere okuma + acknowledge yetkisi verin; Allow/Block gibi kalıcı kararları ve kanal/istisna yönetimini güvenlik sorumlularıyla sınırlayın.

---

## Sık Sorulan Sorular

**"Bulgular" ile "İhlaller" arasındaki fark ne?**
İhlaller anlık ham akıştır; bulgular aynı olayın tekilleştirilmiş, karar alınabilir halidir. Günlük operasyonu Bulgular ekranından yürütün; İhlaller'i canlı gözlem için kullanın.

**Risk skorum neden değişti?**
Skor 5 dakikada bir yeniden hesaplanır. Yeni bir Critical bulgu, skoru hızla yükseltir; bulgu çözüldükçe ve zaman geçtikçe (tazelik eğrisi) skor düşer.

**Bir bulguyu yanlışlıkla Resolve ettim, geri alabilir miyim?**
Hayır — Resolve kalıcıdır. Aynı davranış tekrar tespit edilirse yeni bir bulgu oluşur. Emin olmadığınız durumda Acknowledge kullanın.

**Bildirim gelmiyor.**
Sırasıyla kontrol edin: (1) Yönlendirme kuralının şiddet eşiği ve filtreleri bulguyla eşleşiyor mu? (2) Mesai saatleri penceresi dışında mısınız? (3) Kanal testi başarılı mı? (4) Aynı bulgu için daha önce bildirim gittiyse tekilleştirme devrededir.

---

## İlgili Dokümanlar

- [Servis Güvenliği Çalışma Alanı](service-security-guide.md) — tek bir servisin güvenlik yönetimi
- [Çalışma Zamanı Koruması](runtime-security-guide.md) — koruma modları ve politika kavramları
- [Erişim Kontrolü](access-control-guide.md) — rol ve izin yönetimi
- [Bildirim Yönetimi](notification-guide.md) — platform geneli bildirim kanalları
