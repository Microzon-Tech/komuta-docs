# Uyarılarda Sorun Giderme

Önce sorunun hangi aşamada olduğunu belirleyin: **kural → yayın → olay → bildirim**. Doğru hesap, servis, zaman aralığı ve filtrelerde olduğunuzu kontrol ederek başlayın.

## Uyarı tetiklenmiyor

1. Kuralın etkin olduğunu ve amaçladığınız kaynağı izlediğini kontrol edin.
2. [Yayın rozetini](alerts-rules.md) okuyun. **Doğrulanmadı** tek başına başarısızlık değildir; özellikle log ve ajan yönetimli kurallarda gözlem sınırlıdır.
3. Metrik/log verisinin gerçekten geldiğini inceleyin. CPU/bellek yüzde şablonları pozitif kaynak limitine ihtiyaç duyar.
4. Karşılaştırmayı, eşiği, veri penceresini ve süreyi ayrı kontrol edin. `>10` için 10 olay yeterli değildir.
5. Log metninin seçili kaynakta birebir bulunduğunu kontrol edin. Uygulama çıktısı ile telemetri akışını karıştırmayın; değişken zaman/istek bilgilerini eşleşme metninden çıkarın.
6. Geçmişte olayı arayın. Bildirim gelmemesi, olay oluşmadığını göstermez.

**Test / YAML** geçse bile veri yoksa veya koşul sağlanmıyorsa olay beklenmez. Eksik veriyi otomatik olarak “sağlıklı” veya “sıfır” saymayın.

## Kayıt çözülmüyor

Geçmiş son alınan durumu gösterir. Önce güncel veriyle koşulun gerçekten sona erdiğini kontrol edin. Beş veya 15 dakikalık sorgu penceresi eski olayları hâlâ içerebilir. Koşul düzeldiği halde çözülme kaydı alınmıyorsa zamanları not edip destek isteyin. Kuralı silmek, kapatmak veya servisi uyutmak çözülme teslimini kanıtlamaz.

## Bildirim gelmiyor

- Önce ilgili tetiklenme veya çözülme kaydının varlığını kontrol edin.
- Kuralın kanal seçimini, kanalın aktifliğini ve varsa şiddet filtresini inceleyin. Kanal silindiyse kuralın seçimini güncelleyin.
- İlgili kural için aktif susturma ve susturmanın kapsamını kontrol edin.
- Tekrar aralığını, bildirim sınırlarını ve Kanallar sayfasındaki başarısız/bastırılmış kayıtları inceleyin.
- **Test gönder** ile kanalı bağımsız deneyin. E-postada gerçek alıcıları ve spam/karantinayı; Slack’te hedef kanalı; Teams’te webhook adresi, workflow erişimi ve çalıştırma geçmişini kontrol edin.
- Ürün kaydı başarılı olsa da gerçek hedefte mesajı doğrulayın. E-postada sağlayıcı kabulü gelen kutusu kanıtı değildir.

## Yayın bekliyor veya başarısız

Kaydetme sonucu ile güncel yayın gözlemini ayırın. **Tanım farklı** eski tanımın hâlâ çalışabileceğini, **Kurulu değil** son başarılı kontrolde bulunmadığını belirtir. **Doğrulanmadı** yetersiz veya eskimiş kanıttır.

Doğrudan yayınlanan bir kuralda hata varsa düzeltip sunulan **Tekrar dene** işlemini kullanın. Otomatik yayın yolundaki bir kurala manuel yayın uygulamaya çalışmayın. Son değişiklik zamanını ve görünen hatayı kaydedin; aynı kuralı tekrar tekrar oluşturmayın. Servisin dağıtım durumundan uyarının kurulumunu çıkarmayın.

## Uygun şablon veya cluster görünmüyor

| Kontrol | Olası açıklama |
| --- | --- |
| Yalnız servisiniz var | Kendi cluster’ınızın listesi boş olabilir; servis şablonlarıyla devam edin. |
| Paylaşılan altyapıda özel metrik sorgusu istiyorsunuz | Servis metrik şablonunu kullanın. Özel metrik sorgusu uygun kendi cluster kapsamına bağlıdır. |
| API Gateway şablonu arıyorsunuz | Seçili servis yönetilen API Gateway olmalıdır. |
| Rollout şablonu arıyorsunuz | İş yükünün Rollout sahipliği doğrulanmış olmalıdır. |
| Liste yüklenemiyor | Yükleme/hata uyarısını kontrol edip tekrar deneyin; boşluğu “uygun kaynak yok” diye yorumlamayın. |
| Servis veya işlem düğmesi yok | Okuma, oluşturma ve düzenleme izinlerini ayrı kontrol ettirin. |
| Log kuralı oluşturulamıyor/etkinleşmiyor | Servis kapsamı ve kaynak gerekli; servis başına 20 etkin log kuralı sınırını kontrol edin. |

## Susturma oluşturulamıyor veya etkisiz

En az bir geçerli kural, tek cluster kapsamı ve bitişi başlangıçtan sonraki bir pencere seçin. Kural silindiyse yeniden seçin. Yayın/eşitleme hatası varsa susturulmuş kabul etmeyin. Yerel saat dilimini, başlangıcı, bitişi ve seçili kuralları kontrol edin. Yetki yalnız okumaysa değişiklik yapılamaz.

## Kapsam belirsiz veya çok fazla bildirim var

**Hesap bant genişliği** kaydı tek servis belirtmez. **Kapsam doğrulanamadı** durumunda kural adından tahmin yürütmeyin. Paket kayıplarının hepsini normal kota sınırlaması saymayın; [kayıp türünü](alerts-history.md) ayırın.

Aynı servis/koşul için mükerrer kuralları, kanallardaki ortak alıcıları, kısa süreyi ve tekrar aralığını kontrol edin. Geçmişte birden fazla gerçek olay da olabilir; bütün kayıtları tek olay varsaymayın. Gerektiğinde kısa, dar kapsamlı bir [susturma](alerts-silences.md) kullanın ve asıl nedeni araştırın.

## Destek için ne paylaşabilirim?

[Destek talebine](support-tickets.md) şunları ekleyin:

- İlgili ekran, işlem ve beklediğiniz sonuç.
- Zaman ve saat dilimi; son değişiklik zamanı.
- Kural türü, şiddeti, süre/eşik ve gördüğünüz yayın/olay/bildirim durumu.
- Görünür kapsamın servis, hesap veya cluster oluşu; kapsam doğrulanamadıysa bu bilgi.
- Kişisel ve erişim bilgileri kapatılmış ekran görüntüsü; yeniden üretmek için güvenli adımlar.

Webhook adresi, parola, token, ham log içindeki kişisel bilgi, altyapı erişim adresi veya özel kaynak kimliklerini paylaşmayın. Gerekirse servis ve kural adlarını anonimleştirin. Destek ekibi daha fazla bilgi isterse güvenli destek kanalı üzerinden ilerleyin.

[Genel Bakışa dön](alert-guide.md) · [İlk uyarı adımlarını yeniden izle](alerts-quick-start.md)
