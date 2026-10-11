# Bant Genişliği Planları

Her servisin bir bant genişliği planı vardır. Plan, servisin her pod'unun dışarıya ne hızla veri gönderebileceğini belirler. Gelen trafik sınırsızdır ve faturalandırılmaz.

Yeni servisler ücretsiz **Starter** planıyla başlar. Servisiniz büyük dosyalar sunuyorsa, çok kullanıcıya yanıt veriyorsa ya da indirmeler yavaş kalıyorsa planı servis sayfasındaki **Bant genişliği** sekmesinden yükseltin.

---

## Plan Neyi Sınırlar

| Trafik | Hız sınırı |
|---|---|
| **Giden (çıkış)** — kullanıcılara döndüğünüz yanıtlar, indirmeler ve servisinizin dışarıya gönderdiği diğer veriler | Planın çıkış hızı, her pod için ayrı ayrı |
| **Gelen** — servisinize gelen istekler ve yüklemeler | Yok; gelen trafik ücretlendirilmez |

Hız sınırı pod başınadır. Servisiniz 3 pod ile çalışıyorsa her pod plandaki hıza kadar veri gönderebilir; toplam kapasite pod sayısıyla artar.

Plan, servisin aldığı istek sayısını sınırlamaz. Saniyede kaç istek geldiğinden bağımsız olarak yalnızca giden verinin hızı sınırlanır.

---

## Planlar

| Plan | Aylık ücret | Çıkış hızı (pod başına) |
|---|---|---|
| **Starter** (varsayılan) | Ücretsiz | 5 Mbps |
| **Growth** | 4,99 USD | 25 Mbps |
| **Business** | 14,99 USD | 100 Mbps |
| **Enterprise** | 49,99 USD | 500 Mbps |
| **Scale** | 149,99 USD | 2 Gbps |

Fiyatlar USD cinsindendir ve vergi hariçtir. Güncel değerleri **Bant genişliği** sekmesindeki plan kartlarında ve [Fiyatlandırma](https://www.komuta.io/pricing) sayfasında görürsünüz.

Komuta yeni oluşturulan her servise **Starter** planını kendiliğinden atar. Stack ile oluşturduğunuz servislerde başlangıç planını manifestteki `bandwidth.plan` alanıyla (örneğin `Growth`) seçebilirsiniz; ayrıntılar [Stack Manifest Referansı](stack-manifest-reference.md) sayfasındadır.

---

## Bant Genişliği Sekmesi

**Uygulamalar** sayfasından servisi açın ve sol menüden **Bant genişliği** sekmesine geçin (`https://console.komuta.io/services/<servis-id>/bandwidth`). Sekme dört bölümden oluşur.

### Plan özeti

Sayfanın üstündeki şerit servisin şu anki durumunu gösterir:

- **Geçerli plan** — plan adı, aylık ücreti ve **Aktif** rozeti.
- **Çıkış (pod başına)** — planın her pod için uyguladığı hız sınırı.

### Bant genişliği geçmişi

Grafik, servisin **Gelen** ve **Giden** trafiğini zamana göre çizer.

- Zaman aralığını **5m**, **15m** veya **1h** olarak seçin. Açık kaldığı sürece grafik birkaç saniyede bir yeni ölçüm ekler.
- **Duraklat** grafiği dondurur, **Devam et** canlı akışı sürdürür. Yanındaki düğme veriyi hemen yeniler.
- Üstteki kutucuklar son ölçümü gösterir. **Giden** kutucuğu hızı plan limitine oranlar; **Gelen** kutucuğunda **Sınırsız, ücretsiz** yazar.
- Alttaki özetler: **Ort. gelen**, **Ort. giden**, **En yüksek gelen**, **En yüksek giden**, **Son 30 gün gelen** ve **Son 30 gün giden**.

### Canlı kullanım

Yan paneldeki gösterge, giden trafiğin plan limitine oranını yüzde olarak verir; altında **…bps limitten** yazar. Değer kırpılmaz, %100'ün üstünde de görünebilir. Değerler son 1 dakikanın ortalamasıdır; kuyruk etkisi nedeniyle istemcinizin ölçtüğü hızdan farklı olabilir.

Gönderim plan hızına yavaşlatılıyorsa göstergenin altında **… hızında şekillendiriliyor · ortalama … ms bekleme** satırı belirir. Gelen trafik ayrı bir satırda **Sınırsız** rozetiyle gösterilir.

### Mevcut planlar

Tüm planlar kart olarak listelenir. Her kartta aylık ücret ve **Çıkış (pod başına)** hızı yer alır. Varsayılan plan **Varsayılan** rozetiyle işaretlidir; servisin kullandığı planın düğmesinde **Geçerli plan** yazar.

### Grafik ve göstergelerdeki durumlar

| Görünen | Anlamı |
|---|---|
| **Canlı** | Veri güncel; ölçümler akıyor. |
| **Duraklatıldı** | Grafiği siz durdurdunuz. **Devam et** ile sürdürün. |
| **Örnekler toplanıyor…** | Seçili aralıkta henüz ölçüm yok. Trafik geldikçe grafik dolar. |
| **Henüz trafik kaydedilmedi** | Servis için canlı ölçüm yok. |
| **Kullanılamıyor** | Trafik izleme güncel ölçüm raporlamıyor. Ekranda **Trafik izleme güncel bir örnek raporlamıyor. Kullanım sıfır değil, bilinmiyor.** yazar. Bu durum limiti ve ücretlendirmeyi etkilemez. |

---

## Planı Değiştirme

1. **Bant genişliği** sekmesinde **Mevcut planlar** bölümüne inin.
2. İstediğiniz planın kartındaki düğmeye tıklayın. Daha hızlı planlarda **Yükselt**, daha yavaşlarda **Düşür** yazar.
3. **Plan değişikliğini onayla** penceresini kontrol edin:
   - **Önceki** ve **Yeni** plan ile aylık ücretleri,
   - yeni planın **Çıkış (pod başına)** hızı,
   - ücret farkı (**Aylık faturanız $… artacak** ya da **azalacak**),
   - düşürmede uyarı: **Düşük limitler, uygulamanız kapasiteye yakınsa trafik kısıtlamasına yol açabilir.**
4. **Yükseltmeyi onayla**, **Düşürmeyi onayla** veya **Değişikliği onayla** ile onaylayın. Vazgeçmek için **İptal**'e tıklayın.

Onaydan sonra **"Growth" planı kaydedildi. Küme senkronizasyonu sürüyor.** bildirimi çıkar ve plan özeti yeni planı gösterir.

Plan değiştirmek için servisi düzenleme yetkiniz olmalıdır.

### Değişiklik ne zaman geçerli olur

Plan hemen kaydedilir. Ardından servis aynı sürümle yeniden dağıtılır: yeni build alınmaz, pod'lar yeni hız sınırıyla yeniden oluşturulur. Yeni limit, yeni pod'lar canlı trafiği aldığında geçerli olur.

| Dağıtım stratejisi | Yeni limit |
|---|---|
| **Auto Promote**, **Canary** | Yeni pod'lar hazır olunca kendiliğinden canlıya alınır. |
| **Blue-Green** | Yeni pod'lar önizlemede bekler. **Geçmiş** sayfasında **Canlıya al** ile geçişi siz onaylayana kadar canlı pod'lar eski limitle çalışır. |

> **Uyarı:** Ücret, plan kaydedildiği saatten itibaren yeni plana göre işler. Blue-Green servislerde canlıya almayı unutmayın; aksi halde yeni planın ücretini öderken eski hızla çalışırsınız.

Stratejiler ve **Canlıya al** işlemi için [Dağıtım Geçmişi](service-deployment-history.md) sayfasına bakın.

### Plan değiştirilemediğinde

| Durum | Gördüğünüz mesaj |
|---|---|
| Servis henüz hiç dağıtılmadı | **Bu serviste manifest Git deposu URL'si yok; küme senkronu çalıştırılamaz.** İlk dağıtım tamamlandıktan sonra tekrar deneyin. |
| Yükseltme, aylık tahmini harcama limitinizi aşıyor | **Bu işlemle tahmini aylık harcamanız … USD olacak; mevcut harcama sınırınız … USD. Devam edebilmek için sınırınızın en az … USD daha yüksek olması gerekiyor. …** |
| Tahsil edilemeyen bir fatura var | **Bir faturanız tahsil edilemediği için hesabınız ödemesiz dönemde. Aylık harcamanızı artıran işlemler, bakiye kapatılana kadar durdurulmuştur. …** |
| Hesap askıya alındı | **Ödenmemiş faturalarınız nedeniyle hesabınız askıya alındı. İşlemlere devam etmek için Hesabım → Faturalandırma bölümünden bakiyenizi kapatın.** |

Harcama limiti yalnızca aylık ücreti artıran değişikliklerde kontrol edilir. Planı düşürmek ya da **Starter**'a dönmek bu kontrole takılmaz.

---

## Hız Sınırına Ulaşınca

Servisiniz plandaki hızdan fazlasını göndermeye çalıştığında Komuta isteği reddetmez, hata döndürmez ve bağlantıyı kesmez. Pod'un gönderdiği veri plan hızına yavaşlatılır. Kullanıcılarınız bunu daha uzun süren indirmeler ve daha yavaş gelen büyük yanıtlar olarak fark eder.

Komuta her servis için **Egress Bant Genişliği Kotası Aşımı** uyarı kuralını kendiliğinden oluşturur. En yoğun pod'un giden trafiği 5 dakika boyunca plan hızının %90'ının üstünde kalırsa uyarı tetiklenir. Kural, planın hız sınırını izler. Kendiliğinden oluşturulan kurallar için [Kurallar ve Yayın Durumu](alerts-rules.md) sayfasına bakın.

Sınıra sık takılıyorsanız iki yol vardır:

- **Planı yükseltin** — her pod daha hızlı gönderir.
- **Pod sayısını artırın** — sınır pod başına olduğundan toplam kapasite artar. [Otomatik Ölçekleme](services-autoscaling.md) yük arttığında pod ekler.

---

## Ücretlendirme

**Plan ücreti.** Aylık ücret saatlik işlenir: saatlik ücret, aylık ücretin 720'de biridir. Ücret, plan servise atanmış olduğu sürece trafik olmasa da işler; ay ortasında plan değiştirirseniz her planın ücreti kullandığınız saatler kadar yansır. **Starter** ücretsizdir.

**Gelen trafik.** Gelen trafik hiçbir planda ücretlendirilmez.

**Fatura dönemi.** Dönem her ayın 6'sından bir sonraki ayın 6'sına kadar sürer (UTC); ekstre her ayın 6'sında kesilir. Plan ücreti ekstrede servisin satırı olarak yer alır.

**Nerede görürsünüz.**

- Servisin **Fatura** sekmesi — servisin bu dönemki kalemleri.
- **Hesap → Güncel Harcama** — organizasyonun bu dönemki tüm harcaması (organizasyon yöneticileri görür).

**Harcama limiti.** Plan ücretleri organizasyonun aylık tahminine girer. Yükseltme bu tahmini harcama limitinin üstüne çıkarıyorsa plan değiştirilmez. Limit yalnızca değişiklik anında kontrol edilir; çalışan servisler durdurulmaz.

---

## API Gateway Bant Genişliği

API Gateway'lerin de bant genişliği planı vardır. Plan, servislerdekiyle aynı planlardan seçilir ve aynı şekilde çalışır: her gateway pod'unun giden trafiğini sınırlar, gelen trafik sınırsızdır. Gateway'in istemcilere döndüğü yanıtlar giden trafik olduğundan, gateway'in planı API'lerinizin yanıt hızını belirler. Arkadaki servislerin kendi planları ayrıca geçerlidir.

Yeni bir gateway **Starter** planıyla başlar. Planı değiştirmek için:

1. **API Gateway'ler** sayfasından gateway'i açın.
2. **Bant Genişliği** sekmesine geçin. Plan özeti, **Canlı kullanım** paneli ve plan kartları servislerdeki sekmeyle aynıdır. **Canlı Trafik** grafiği ise yalnızca sekme açıkken toplanan ölçümleri çizer; geçmiş veri yüklemez.
3. Planı seçip onaylayın.

Sekmenin altında **Plan değişikliği gateway'in manifestlerini yeniden senkronlar. Trafik kesilmez.** notu yer alır. Gateway'in bant genişliği planı, API Gateway'in kendi aylık ücretinden ayrı ücretlenir. Gateway'in kurulumu ve diğer ayarları için [API Gateway](api-gateway.md) sayfasına bakın.

---

## İlgili Dokümanlar

- [Otomatik Ölçekleme](services-autoscaling.md) — Pod sayısını yüke göre artırarak toplam çıkış kapasitesini büyütme.
- [Dağıtım Geçmişi](service-deployment-history.md) — Dağıtım stratejileri ve **Canlıya al** işlemi.
- [Faturalandırma](billing.md) — Fatura dönemi, ekstreler ve harcama limiti.
- [API Gateway](api-gateway.md) — Yönetilen API Gateway'in kurulumu ve ayarları.
- [Kurallar ve Yayın Durumu](alerts-rules.md) — Komuta'nın kendiliğinden oluşturduğu kurallar.
- [Stack Manifest Referansı](stack-manifest-reference.md) — `bandwidth.plan` alanı.
- [Fiyatlandırma](https://www.komuta.io/pricing) — Güncel plan fiyatları.
