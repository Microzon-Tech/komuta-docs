# Observability: Genel Bakış Ekranı

Genel Bakış, bir servisin son bir saatlik penceredeki durumunu tek karta sığdırır: kaç istek geldi, kaçı hataydı, gecikme nasıldı. Sayılar, ham bir trace taramasından değil, önceden hesaplanmış ve doğrulanmış özet verilerden üretilir.

> Bu ekran CPU, bellek, disk veya node kapasitesi göstermez. Bu bilgiler servisin **Resources** ve ilgili cluster ekranlarında izlenir.

---

## Durum kartı

Ekranın en üstünde servisin genel durumu tek kelimeyle özetlenir: **Sağlıklı**, **Bozulmuş**, **Arızalı** veya **Bilinmiyor**. Yanında **"Nitelikli"** rozeti görüyorsanız, bu değerlendirme eksiksiz kanıta dayanmıyor demektir — durumu temkinli yorumlayın.

Kartın sağında üç temel sayı yer alır:

- **İstekler** — pencerede gözlemlenen toplam istek sayısı
- **Hatalar** — hatalı istek sayısı
- **Hata oranı** — hiç istek yoksa "ölçülecek istek yok" olarak gösterilir, %0 olarak değil; bu ikisi farklı şeylerdir

Durum kartının altındaki küçük etiketler, o değerlendirmenin dayandığı gerekçeleri gösterir (ör. hangi kaynaktan geldiği). Bunlar teknik kısa kodlar olarak görünür; bir sorunu destek ekibiyle paylaşırken işinize yarayabilir.

---

## Trafik ve hata grafiği

Beş dakikalık dilimler (bucket) halinde istek ve hata sayısını çubuk grafik olarak gösterir. İki nokta arasına çizgi çekilmez — her çubuk yalnızca kendi beş dakikalık dilimini temsil eder, ölçüm gelmeyen bir dilim sıfır olarak değil, hiç çizilmeyerek gösterilir.

> **Not:** Bir dilimde çubuk yoksa bu "trafik sıfırdı" değil, "bu dilim için özet satırı üretilmedi" anlamına gelir.

---

## En kötü dilimin gecikmesi

Kartın yanında, pencere içindeki **en yavaş** beş dakikalık dilimin p50/p95/p99 gecikme değerleri gösterilir. Bu, tüm pencerenin ortalama gecikmesi değildir — sadece o tek dilimin kendi ölçümüdür ve hangi saatte ölçüldüğü ayrıca belirtilir.

Hiçbir dilimde gecikme ölçümü yoksa panel "gecikme gösterilemiyor" der; bu servisin hızlı olduğu anlamına gelmez, sadece ölçüm olmadığı anlamına gelir.

---

## Uç noktalara göre RED (istek, hata, gecikme)

Bu bölüm, aynı istek/hata/gecikme özetini tek tek uç noktalar (endpoint) bazında listeler. Liste önce en çok hata alan, sonra en çok trafik alan uç noktayı gösterecek şekilde sıralanır.

Her satırda görürsünüz:

- **Metod ve rota** (ör. `GET /orders/{id}`)
- **Trend** — o uç noktanın beş dakikalık dilimler boyunca istek eğilimini gösteren küçük bir çubuk şeridi; kırmızı çubuklar hata içeren dilimleri işaret eder
- **İstek sayısı, hata oranı, en kötü dilim p95'i**

Bir uç noktanın gecikmesi "ölçüm yok" (no sketch) diye görünüyorsa ve dilimlerin bir kısmı ölçüm taşımıyorsa, kaçı taşıdığı ayrıca belirtilir (ör. "12 dilimden 9'unda ölçüm var"). Bu, eksik veriyle üretilmiş bir p95'in tam bir p95 gibi okunmasını önler.

Hiçbir uç nokta listelenmiyorsa bu, servisin trafik almadığı anlamına gelmez — o pencere için okunabilir bir özet üretilememiş olabilir.

---

## Henüz gelmemiş bölümler

Kartın altında üç bölüm daha yer alır: **Bağımlılıklar**, **Öne çıkan bulgular** ve **Değişiklik zaman çizelgesi**. Bu bölümler şu an kesikli çerçeveyle ve "henüz kullanılamıyor" notuyla gösterilir — planlanan ama henüz teslim edilmemiş yeteneklerdir. Bu, bir arıza değildir.

---

## Sık sorulan sorular

**Grafik boş ama servisim trafik alıyor, ne oluyor?**
Önce durum kartındaki kanıt rozetine bakın. "Tam" değilse gösterilen boşluk bir ölçüm eksikliğidir, servisin trafiksiz olduğu anlamına gelmez. [Veri Sağlığı ekranından](./observability-vnext-data-health.md) ilgili kaynağın durumunu kontrol edebilirsiniz.

**"Hata oranı: ölçülecek istek yok" ne demek?**
O uç nokta veya pencerede hiç istek gözlemlenmemiş demektir. Bunu %0 hata oranıyla karıştırmayın — %0, ölçülmüş ve hatasız bir trafiği; "ölçülecek istek yok" ise hiç ölçüm olmadığını ifade eder.

**En kötü dilimin p95'i neden pencerenin tamamının p95'i değil?**
Gecikme yüzdelikleri (p50/p95/p99) dilimler arasında toplanamaz; her dilim kendi özetini taşır. Bu ekran, en yavaş dilimi dürüstçe "bu dilimin ölçümü" olarak etiketler; pencerenin tamamı için tek bir p95 iddia etmez.

---

## İlgili

- [Observability: Menü ve Ekranlara Genel Bakış](./observability-vnext-guide.md)
- [İncele ekranı](./observability-vnext-investigate.md)
- [Trace ekranı](./observability-vnext-trace.md)
- [Veri Sağlığı ekranı](./observability-vnext-data-health.md)
