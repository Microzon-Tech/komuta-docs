# Loglar

Loglar sekmesi, servisin container çıktısının arandığı, filtrelendiği ve canlı izlendiği ekrandır. Bir hatanın izini sürmek, bir isteğin hangi adımda düştüğünü bulmak ya da bir log satırından alarm kurmak buradan yapılır.

![Loglar sekmesinin tamamı — filtre çubuğu, hacim grafiği ve log satırları birlikte](https://cdn.komuta.io/docs/tr/images/logs/logs-overview.png)

---

## Filtreler

Filtre çubuğundaki değerler bir taslaktır; **Uygula** butonuna basılana kadar tabloyu değiştirmez. Tek istisna satır üzerinden eklenen alan filtreleridir — onlar eklendiği anda sorguyu yeniden çalıştırır.

### Zaman Aralığı

Tek bir sorgu en fazla **12 saatlik** bir aralığı tarar; varsayılan **Son 1 saat**'tir. **Özel aralık** ile başlangıç ve bitiş zamanı elle girilir. Daha geriye gitmek **Daha eski logları yükle** ile yapılır.

### Log Seviyesi

Seviye filtresi `ERROR`, `WARN`, `INFO`, `DEBUG`, `TRACE` ve `FATAL` değerlerinden birine daraltır.

Seviye etiketi taşımayan satırlarda Komuta seviyeyi mesajın kendisinden çıkarmaya çalışır — `[10:51:10 INF]`, `level=error`, `WARN -` gibi yaygın biçimler tanınır. Hiçbiri eşleşmezse satır seviyesiz sayılır ve seviye filtresi uygulandığında listede yer almaz.

### Arama Kutusu

Arama kutusu iki modda çalışır. Düz metin modunda girilen ifade, log mesajının içinde büyük/küçük harf ayrımı gözetmeden aranır. Regex modunda ifade bir RE2 deseni olarak yorumlanır. Mod, kutunun yanındaki ikonla değiştirilir.

Düz metin modunda aranan ifade sonuçlarda işaretlenir; regex modunda işaretleme yapılmaz.

### Alan Filtreleri

Arama kutusuna `alan:değer` biçiminde yazıldığında sorgu, mesaj içinde metin taraması yerine indeksli alan araması olarak çalışır — düz metin aramasından belirgin biçimde hızlıdır. Desteklenen alanlar ve eşanlamlıları:

| Alan | Ne taşır | Kabul edilen yazımlar |
|---|---|---|
| `trace_id` | Bir isteğin uçtan uca izini tanımlayan W3C trace kimliği | `trace_id`, `traceId`, `traceparent` |
| `correlation_id` | Uygulamanın kendi ilişkilendirme kimliği | `correlation_id`, `correlationId`, `cid` |
| `request_id` | Tek bir isteğin kimliği | `request_id`, `requestId`, `rid`, `reqid` |
| `span_id` | Trace içindeki tek bir adımın kimliği | `span_id`, `spanId` |
| `error_code` | Uygulamanın ürettiği hata kodu | `error_code`, `errorCode`, `errcode`, `err` |

Örnek — belirli bir isteğin tüm satırlarını getirir.

```text
request_id:abc-123
```

Bir log satırındaki alan etiketi ya da detay penceresindeki **Bununla filtrele** butonu, aynı alanı kalıcı filtreye çevirir.

Alan filtresi yalnızca uygulama o alanı loglarında üretiyorsa sonuç verir; üretmiyorsa filtre hiçbir satırla eşleşmez.

### Trace Bağlamı

**Sadece trace id'si olanlar** anahtarı, sonucu geçerli bir trace kimliği taşıyan satırlara indirger. Bu, satırın trace detayının da saklandığı anlamına gelmez — yalnızca logun bir trace'e bağlanabilir olduğunu söyler.

Bu filtre açıkken canlı izleme kullanılamaz, çünkü canlı izleme sabit bir zaman aralığını sorgulamak yerine akışı takip eder.

![Filtre çubuğu — zaman aralığı, seviye, arama kutusu ve sabitlenmiş alan filtreleri](https://cdn.komuta.io/docs/tr/images/logs/log-filters_v2.png)

---

## Zaman Çizelgesi ve Daha Eski Loglar

### Hacim Grafiği

Hacim grafiği, bir hatanın *ne zaman* başladığını bulmaya yarar: bir sütuna tıklandığında tablo yalnızca o zaman dilimine daralır.

Grafik yalnızca tek servis kapsamında ve canlı izleme kapalıyken çalışır.

### Daha Eski Logları Yükleme

**Daha eski logları yükle**, 12 saatlik sorgu sınırına takılmadan geçmişe inmeyi sağlar: her basışta arama daha geriye taşınır ve bulunan satırlar listenin sonuna eklenir.

---

## Canlı İzleme

**Canlı izle** anahtarı açıkken yeni log satırları geldikçe listenin başına eklenir ve liste akışı takip eder. Bu moddayken hacim grafiği ve sayfalama kapanır; ekran sorgu sonucu değil, akış gösterir.

Canlı izleme yalnızca tek servis kapsamında çalışır ve trace bağlamı filtresi açıkken kullanılamaz.

---

## Log Satırı Detayı

### Detay Penceresi

Bir satıra tıklandığında o kaydın tamamı açılır: mesajın kırpılmamış hali, varsa JSON içeriği, satırın taşıdığı yapılandırılmış alanlar ve etiketler. Mesaj ve alan değerleri tek tek kopyalanabilir, alanların yanındaki **Bununla filtrele** butonuyla loglar o değere göre filtrelenir.

![Log detay penceresi — mesaj, JSON içeriği ve yapılandırılmış alanlar bölümleri](https://cdn.komuta.io/docs/tr/images/logs/log-detail.png)

### Trace'e Geçiş

Satır geçerli bir trace kimliği taşıyorsa **Trace'i ara** ile o trace'in detayına geçilir; bağlantı logun zamanına sabitlenir. Trace kimliği olmayan satırlarda bu bağlantı yerine nedenini söyleyen bir açıklama görünür: Komuta bir logu yalnızca metnine ya da zamanına bakarak bir trace ile eşleştirmez.

### Logdan Alarm Oluşturma

Bir satırın üzerindeki zil ikonu, o satırdan ön doldurulmuş bir log alarmı oluşturur. Eşleşme metni satırın ilk satırından türetilir; zaman damgası ve seviye ön eki ayıklanır.

| Alan | Ne yapar |
|---|---|
| **Eşleşme metni** | Bu metni içeren log satırları sayılır. Mesajın değişmeyen kısmına kadar kırpılmalıdır. |
| **Eşik** | Alarm, eşleşme sayısı bu değeri *aştığında* tetiklenir. `0` girildiğinde tek bir eşleşme yeter; `10` girildiğinde 10'dan fazlası gerekir. |
| **Süre** | Koşulun kesintisiz ne kadar sürmesi gerektiği. `0s` hemen tetikler; `2m` anlık dalgalanmaları eler. |
| **Önem** | Alarmın önem derecesi. |

Sayım penceresi sabittir ve değiştirilemez: **5 dakika**. Süre alanı her zaman bir birim ister — `30s`, `2m`, `1h`; yalnızca sayı geçersizdir.

Nadir ama kritik bir satırda eşik `0` ve süre `0s` kullanılır: `OutOfMemoryException` bir kez bile düşse alarm anında tetiklenir.

Zaten arada bir görülen bir hatada eşik yükseltilir: eşleşme metni `payment gateway timeout`, eşik `10`, süre `2m`. Tek tük hatalar alarm üretmez; beş dakikada 10'dan fazla eşleşme iki dakika boyunca sürerse tetiklenir.

Oluşturulan kural, servisin alarm kurallarına eklenir ve oradan düzenlenir.

![Log Alarmı Oluştur penceresi — eşleşme metni, eşik ve süre alanları](https://cdn.komuta.io/docs/tr/images/logs/log-alert-create.png)

---

## Uygulamalar Arası Arama

Tek bir isteğin birden fazla servise dokunduğu durumlarda, aramanın tek servisle sınırlı kalması izi koparır. Kapsam çubuğundaki **Bunun yerine tüm uygulamalarımda ara** seçeneği aynı sorguyu birden fazla servis üzerinde çalıştırır ve sonuçları zaman sırasına göre birleştirir; her satır hangi servisten geldiğini taşır.

### Korelasyon Filtresi Zorunluluğu

Bu mod, en az bir `trace_id`, `correlation_id` veya `request_id` filtresi olmadan çalışmaz. `error_code` tek başına yeterli değildir.

### Aranacak Servisleri Daraltma

Varsayılan olarak erişilebilen tüm servisler aranır. Servis seçiciyle belirli servisler işaretlenerek kapsam daraltılabilir. Aynı anda **25'ten fazla servis** aranamaz — bu sınır aşıldığında sorgu reddedilir ve kapsamın daraltılması istenir.

### Eksik ve Kırpılmış Sonuçlar

Uygulamalar arası aramada iki durum sonucun eksik olduğunu söyler ve ikisi de farklı bir anlama gelir:

- **Kısmi sonuç** — bazı servisler sorgulanamadı. Eksik satırlar olabilir; sıfır sonuç, o servislerde log olmadığı anlamına gelmez.
- **Kırpılmış sonuç** — eşleşen satır sayısı birleştirme sınırını (500 satır) aştı, en son eşleşmeler gösteriliyor. Zaman aralığı daraltılarak tamamı görülebilir.

Bu modda sayfalama, canlı izleme ve hacim grafiği yoktur.

[RESIM EKLE — logs/cross-service-search.png: Uygulamalar arası arama; kapsam çubuğu, servis seçici ve servis adı taşıyan birleşik satırlar]

---

## Paylaşma ve Dışa Aktarma

### Filtreli Görünümün Linki

**Link kopyala**, o anki görünümün tamamını taşıyan bir bağlantı üretir: zaman aralığı, seviye, arama ifadesi ve modu, trace bağlamı anahtarı, sabitlenmiş alan filtreleri ve uygulamalar arası kapsam. Bağlantıyı açan kişi — erişimi olmak kaydıyla — aynı filtrelenmiş tabloyu görür, filtreleri tekrar kurmasına gerek kalmaz.

> **Not:** Bağlantı, erişim yetkisini taşımaz. Karşı tarafın servisi görme yetkisi yoksa link çalışmaz.

### Dışa Aktarma

**Dışa Aktar**, ekranda o an yüklü olan satırları düz metin dosyası olarak indirir. Her satır zaman damgası, seviye ve mesajı taşır; uygulamalar arası aramada satırın geldiği servis de yazılır. Dışa aktarma ekrandaki satırlarla sınırlıdır — sorgunun tamamını değil, yüklenmiş olanı verir.

---

## Komuta Platform Logları

Listedeki her satır uygulamanın çıktısı değildir; Komuta da kendi satırlarını yazar — örneğin uygulama başlatılmadan önce çalışan platform adımları. Bu satırlar ayrıca işaretlendiği için uygulamanın hatasıyla karıştırılmaz.

---

## Sorun Giderme

**Sorgu zaman aşımına uğradı.** Taranan pencere ya da filtre sayısı log altyapısının yanıt süresini aşmış demektir. Daha dar bir zaman aralığıyla ya da daha az filtreyle tekrar çalıştırılır.

**Zaman aralığı çok geniş.** Tek sorgu en fazla 12 saat tarar. Aralık daraltılır, daha gerisi **Daha eski logları yükle** ile açılır. Bu uyarı genellikle eski bir bağlantının açılmasından gelir.

**Filtre deseni çok uzun / çok fazla filtre.** Arama deseni kısaltılır ya da sabitlenmiş alan filtrelerinin bir kısmı kaldırılır.

**Tek seferde aranamayacak kadar çok uygulama.** Uygulamalar arası aramada servis seçiciyle kapsam 25 servisin altına indirilir.

**Sonuç boş.** Zaman aralığı genişletilir ya da filtreler kaldırılır. Alan filtresiyle arıyorsanız, o alanın uygulamanın loglarında gerçekten üretildiğinden emin olun — üretilmiyorsa filtre hiçbir satırla eşleşmez.

---

## İlgili Dokümanlar

- [Servis Dashboard](service-dashboard.md) — servisin anlık durumu; topolojideki bir pod'a tıklandığında aynı log paneli açılır.
- [Observability Rehberi](observability-guide.md) — trace, RED metrikleri ve veri sağlığı.
- [Alarm Rehberi](alert-guide.md) — log alarmlarının ve diğer kuralların yönetimi.
