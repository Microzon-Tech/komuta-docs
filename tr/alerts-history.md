# Geçmiş, Kapsam ve Çözülme

**Uyarılar → Geçmiş**, Komuta’nın aldığı uyarı olaylarını gösterir. Dönem, arama ve durum filtresini kullanın; satırı açarak mevcut zaman, kapsam, tetikleme değeri ve bildirim kaydı sayısı ayrıntılarını inceleyin.

> **Geçmiş son alınan durumu gösterir.** Açık bir kayıt tek başına uyarının hâlâ tetiklendiğini kanıtlamaz. Çözülme bilgisi henüz alınmamış olabilir.

## Tetiklenmeden çözülmeye

1. Kuralın veri kaynağında koşul oluşur ve seçilen süre boyunca devam eder.
2. Tetiklenme olayı Komuta’ya ulaştığında geçmiş kaydı oluşur/güncellenir.
3. Bildirim yönlendirmesi uygun kanalları kullanır; susturma ve bildirim sınırları sonucu etkileyebilir.
4. Çözülme olayı alındığında kayıt çözülmüş olarak güncellenir.

**Çözülme bekliyor**, henüz çözülme kaydı alınmamış bir olayı belirtir. **Çözüldü**, çözülme bilgisinin kaydedildiğini gösterir. **Bekliyor** durumunu yayın kuyruğuyla karıştırmayın. Olay geçmişi, kuralın canlı değerlendirme ekranı veya alıcının teslim onayı değildir.

Bir kuralın kapatılması/silinmesi ya da servisin uyuması eski olayın çözüldüğünü kanıtlamaz. Loglarda beş dakikalık veri penceresi hâlâ eski eşleşmeleri içeriyor olabilir. [Çözülmeme kontrollerini](alerts-troubleshooting.md) izleyin.

## Örnek kayıt: tek test logunu takip edin

Aşağıdaki kayıtlar **temsili anlatımdır; canlı olay veya teslim kanıtı değildir**. [İlk uyarı örneğindeki](alerts-quick-start.md) `Docs sample alert` ve `orders-api-demo` adlarını kullanır. Saatler aynı gün ve aynı saat dilimindedir; garanti edilen gecikme göstermez.

| Kontrol anı | Geçmişte okuduğunuz | Buradan çıkarılabilecek sonuç |
| --- | --- | --- |
| 12:02 | Kural: Docs sample alert; doğrulanmış servis: orders-api-demo; tetiklenme zamanı: 12:01:20; çözülme bilgisi yok | Bu servise ait tetiklenme kaydı alınmıştır. Güncel koşulu servis loglarında araştırabilirsiniz. |
| 12:03 | Aynı olay hâlâ çözülme bekliyor | Yeni çözülme bilgisi henüz görünmüyor. Bunun yeni bir tetiklenme veya canlı durum güncellemesi olduğunu varsaymayın. |
| 12:06 | Aynı olayda çözülme zamanı: 12:05:40; durum: çözüldü | Çözülme bilgisi alınmıştır. Yeni olay oluşmadığını söylemek için seçili dönem ve diğer satırlara da bakın. |

**Mesajla eşleştirme:** Önce kural adını ve doğrulanmış kapsamı, ardından olay zamanını karşılaştırın. **Kanallar** ekranında aynı dönem ve hedef için tetiklenme gönderimini arayın. Örneğin 12:02’de **Docs test** için başarılı gönderim görülmesi, o denemenin başarılı kaydedildiğini anlatır. E-postayı alıcının posta kutusunda ayrıca bulun. 12:06’daki çözülme gönderimi varsa onu ayrı kayıt olarak okuyun.

**Beklenen sonuç:** Tek olayın önce açık, sonra çözülmüş hâlini ayırt edebilmek ve ona ait gönderimi bulmak. Kural adı aynı olan farklı servisleri veya farklı saatlerdeki olayları yalnız adına bakarak birleştirmeyin.

## Olayı araştırmak için izlenecek sıra

1. **Geçmiş** içindeki dönemi olayın saatini kapsayacak şekilde ayarlayın. Aramayı kural adına daraltın; beklediğiniz satır yoksa durum filtresini temizleyin.
2. Satırı açın. Görünen kapsamı ve tetiklenme/çözülme zamanlarını not edin. Değer alanı varsa birimini kuralın şablonuyla birlikte okuyun; sayı her zaman yüzde değildir.
3. Doğrulanmış servis varsa o servisin metrik veya log ekranını aynı zaman aralığında inceleyin. Kapsam hesap ise hesap trafiği üzerinden ilerleyin; tek servis seçmeyin.
4. Mesaj eksikse **Kanallar** ekranına geçin. Olay da eksikse [tetiklenmeme adımlarından](alerts-troubleshooting.md) başlayın.
5. Kapsam doğrulanamıyorsa görünür durumu ve zamanları [destek talebine](support-tickets.md) ekleyin. Ham etiketlerden yeni bir kapsam üretmeyin.

## Hangi servis etkileniyor?

| Görülen kapsam | Güvenle söylenebilen |
| --- | --- |
| Doğrulanmış servis adı | Kayıt ilgili servis kuralıyla ilişkilidir. O servisin durumunu inceleyin. |
| **Hesap bant genişliği** | Uyarı hesap genelindeki toplamla ilgilidir; tek bir servis belirlenmez. |
| Doğrulanmış kendi cluster’ınızın adı | Kural cluster kapsamındadır. Tek servis sonucu çıkarmayın. |
| **Uyarının kapsamı doğrulanamadı** | Güvenilir kapsam bilgisi mevcut değildir. Kural adından veya geçmiş metninden servis tahmin etmeyin. |

Örneğin **Örnek uygulama — Yüksek bellek kullanımı** servisle ilişkilendirilebilirken, **Hesap Bant Genişliği Paket Kayıpları** aynı şey değildir. Birden fazla servis ortak hesap trafiğine katkıda bulunabilir. Kayıt sayısı, kaybedilen paket sayısı değildir.

## Paket kaybı uyarıları aynı anlama gelmez

| Uyarı türü | Ne anlatır? | İlk kontrol |
| --- | --- | --- |
| Hesap bant genişliği paket kayıpları | Bant genişliği veri yolundaki kayıpları, şekillendirme kuyruğu doluluğuna bağlanan kayıplardan ayıran hesap toplamıdır. | Politika, trafik kimliği veya kaynak sorunu olasılığını destekle araştırın; olağan kota sınırlaması diye kapatmayın. |
| Hesap bant genişliği kuyruk baskısı | İlgili ölçümler mevcutsa, dolu şekillendirme kuyruğu ve bant genişliği sınırıyla ilişkili kayıpları izler. | Trafik talebini ve bant genişliği kapasitesini inceleyin. |
| Hesap bant genişliği zamanlayıcı paket kayıpları | Sınırlı ağ kuyruğundaki zamanlayıcının paket kaybını izler. | Kuyruk belleği, trafiğin davranışı ve altyapı doygunluğu için inceleme isteyin. |
| Hesap bant genişliği acil durum sınırı | İlgili ölçümleri sağlayan altyapıda, politika süresinin dolmasıyla koruyucu bant genişliği sınırının devreye girmesini izler. | Politika yenilemesini destekle inceleyin; olağan kota kullanımı olarak değerlendirmeyin. |
| Servisle ilişkili yüksek/kritik paket drop oranı | Ağ gözlem kaynağındaki paket kaybı hızını izler; hesap bant genişliği sayacıyla aynı kaynak değildir. | Servisin ağ koşullarını ve varsa kayıp nedenlerini inceleyin. Bu kayıt tek başına kök nedeni açıklamaz. |
| Gelen/giden bant genişliği kota uyarısı | İlgili kuralın bant genişliği kullanım koşuludur. | Kullanım, yön ve kural eşiğini kontrol edin; bunu otomatik olarak paket kaybı kanıtı saymayın. |

Bir şablon veya yönetilen kuralın listede olması, gerekli bütün ölçümlerin o altyapıda üretildiğini göstermez. Kapsamı veya kaynağı doğrulanamayan bir olay için belirli bir servisi, kota aşımını veya arızayı kesin neden olarak göstermeyin.

## Bildirim sayısını nasıl okumalıyım?

Geçmişteki bildirim sayısı, ürünün kaydettiği bildirim bilgisiyle ilgilidir. Her alıcının mesajı aldığını, açtığını veya sorunu gördüğünü doğrulamaz. Ayrıntı için [Kanallar](notification-guide.md) sayfasındaki kayıtları inceleyin. Tetiklenme ve çözülme bildirimlerini de ayrı değerlendirin.

Sonraki adım: [Bildirim teslimini kontrol et](notification-guide.md) · [Güvenli destek bilgisi hazırla](alerts-troubleshooting.md)
