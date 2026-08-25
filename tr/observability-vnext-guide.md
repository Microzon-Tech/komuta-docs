# Observability: Yeni Menü ve Ekranlara Genel Bakış

Observability, servisinizde **ne olduğunu**, **hangi ekranın hazır olduğunu** ve **hangi kanıta dayanarak** bir sonuç gösterildiğini tek yerde toplayan bölümdür. Bu doküman seti, Observability menüsünün görev bazlı beş başlık etrafında yeniden düzenlenmiş halini ve şu an fiilen kullanılabilen ekranları anlatır.

Kısaca ne arıyorsanız oraya gidin:

- [Genel Bakış ekranı](./observability-vnext-overview-screen.md) — servisin son bir saatlik durumu, trafik ve gecikme
- [İncele ekranı](./observability-vnext-investigate.md) — kanıta dayalı inceleme kayıtları, aday nedenler
- [Trace ekranı](./observability-vnext-trace.md) — tekil isteklerin span bazlı yolculuğu ve ilişkili loglar
- [Veri Sağlığı ekranı](./observability-vnext-data-health.md) — ekranlardaki veriye ne kadar güvenebileceğiniz

---

## Observability'ye nasıl ulaşılır?

1. Komuta Console'da bir servisi açın, sol menüden **Observability** bölümüne girin. Ekranlar otomatik olarak o servise ve servisin bağlı olduğu ortama (cluster) göre sınırlandırılır — ortamı siz seçmezsiniz.
2. Servis seçmeden genel **Observability** sayfasına girerseniz, üstteki açılır menüden bir servis seçmeniz istenir. Servis seçilene kadar hiçbir sorgu çalıştırılmaz.

Servis kaydı okunamazsa (ör. geçici bir erişim sorunu) ekranda bunun servisin sağlıksız olduğu anlamına gelmediğini belirten ayrı bir uyarı görürsünüz — bu, "henüz servis seçilmedi" durumundan bilerek farklı gösterilir.

---

## Menüdeki başlıklar

Üst menü artık sinyal adlarına (trace, log, metrik...) göre değil, yapmak istediğiniz işe göre gruplanır:

| Başlık | İçerik | Durum |
|---|---|---|
| **Genel Bakış** | Servisin RED özeti (istek, hata, gecikme), trafik grafiği | Kullanılabilir |
| **İncele** | Kanıta dayalı inceleme kayıtları ve aday nedenler | Kullanılabilir |
| **Telemetri › Trace** | Tekil isteklerin span bazlı detayı, ilişkili loglar | Kullanılabilir |
| **Telemetri › Log** | Bağımsız log arama | Henüz yok — loglara yalnızca bir trace'in içinden, o trace'e bağlı (correlated) olarak ulaşılır |
| **Telemetri › Metrics** | Bağımsız metrik gezgini | Henüz yok — trace tabanlı RED özeti şimdilik yalnızca Genel Bakış'ta |
| **Telemetri › Network** | Bağımlılık ve ağ akışı görünümü | Henüz yok |
| **Güvenilirlik** | SLO, anomali, sürüm etkisi | Henüz yok |
| **Optimize Et** | Maliyet, telemetri verimliliği, talep tahmini | Henüz yok |
| **Veri Sağlığı** (üst menü, sağ tarafta) | Ekranlardaki veriye güvenilip güvenilemeyeceği | Kullanılabilir |
| **Panolar / Kayıtlı görünümler / Sorgu gezgini / Ayarlar** | — | Henüz yok |

> **Not:** Menüde henüz yapılmamış bir başlığa tıkladığınızda boş bir grafik ya da kırık bir sayfa görmezsiniz. Bunun yerine, ekranın henüz kullanılabilir olmadığını açıkça belirten bir bilgi sayfası açılır. Bu, bir arıza değildir — sadece o ekran henüz oluşturulmamıştır.

---

## Her ekranda gördüğünüz ortak kanıt sistemi

Genel Bakış, İncele, Trace ve Veri Sağlığı ekranlarının hepsi aynı iki soruyu yanıtlar: *bu sonuç ne kadar tam?* ve *ne kadar güncel?* Bunun için hepsi aynı rozet ve etiketleri kullanır — bir ekranda öğrendiğiniz kelime diğerinde de aynı anlama gelir.

### Kanıt durumu

| Etiket | Anlamı |
|---|---|
| **Tam** | Sorgunun gerektirdiği tüm veri kaynakları bu pencere için okunabildi |
| **Kısmi** | Sonuç var, ama en az bir kaynak veya zaman dilimi eksik |
| **Gecikmeli** | Kaynak veri gönderiyor, ama en güncel dakikalar henüz ulaşmadı |
| **Uyumsuz** | Kaynağın şeması veya sürümü güvenle yorumlanamıyor |
| **Kullanılamıyor** | Güvenilir bir sonuç üretmek için yeterli veri okunamadı |

### Sonuç kesinliği

| Etiket | Anlamı |
|---|---|
| **Kesin** | Sonuç, kapsam içindeki kayıtların tamamından hesaplandı |
| **Örneklenmiş** | Sonuç, kabul edilen örneklenmiş veriden hesaplandı |
| **Yaklaşık** | Sonuçta yaklaşık hesap kullanıldı |
| **Kullanılamıyor** | Kesinlik değerlendirilemedi |

Bir ekranda **"Tam kanıt"** rozeti görüyorsanız, o sonucun eksiksiz ve kesin veriden üretildiğine güvenebilirsiniz. **"Nitelikli kanıt"** rozeti ise "bu sonuç gösteriliyor, ama örnekleme, gecikme veya eksik kapsam gibi bir sebeple tam güvenilir değil — altındaki açıklamayı okuyun" anlamına gelir.

### Diğer ortak kavramlar

- **Kapsam oranı (coverage):** Gerekli verinin ne kadarının okunabildiğini gösteren yüzde. Boş bırakılmışsa "ölçülmedi" demektir — %0 ile karıştırmayın.
- **Kaynak watermark:** İlgili kaynağın güvenilir biçimde tamamlandığı son an. "Watermark yok" ifadesi "veri güncel" anlamına gelmez.
- **Sorgu kimliği (query id):** Her ekranın altında görünen bir kimlik. Bir sorun yaşadığınızda bu kimliği destek ekibiyle paylaşmanız, aynı sorguyu sunucu taraflı loglardan bulup teşhis etmelerini hızlandırır.

> **İpucu:** Bir grafik veya sayı boş göründüğünde önce ekranın kanıt rozetine bakın. "Tam" ve "Kesin" ile birlikte boşsa bu gerçekten ölçülmüş bir sıfırdır; herhangi bir uyarı rozeti varsa boşluk bir ölçüm eksikliğidir, servisin sorunsuz olduğu anlamına gelmez.

---

## İlgili Dokümanlar

- [Genel Bakış ekranı](./observability-vnext-overview-screen.md)
- [İncele ekranı](./observability-vnext-investigate.md)
- [Trace ekranı](./observability-vnext-trace.md)
- [Veri Sağlığı ekranı](./observability-vnext-data-health.md)
- [Observability Rehberi (genel kavramlar ve saklama süreleri)](./observability-guide.md)
- [İzleme ve Log Yönetimi](./monitoring-logs.md)
