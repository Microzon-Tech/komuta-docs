# Kanallar ve Bildirim Teslimi

Komuta Uyarılar; **E-posta, Slack ve Microsoft Teams** kanallarını kullanır. **Uyarılar → Kanallar** tanımlı kanalları ve teslim kayıtlarını gösterir. Ekleme ve değişiklikler için **Bildirim ayarlarında yönet** bağlantısıyla **Bildirimler ve Uyarılar** sayfasını açın.

## Önce bir hedef hazırlayın

| Kanal | Gerekli yapılandırma |
| --- | --- |
| E-posta | Kanal adı ve en az bir geçerli alıcı adresi. Her adresi listeye ekleyip kaydedin. |
| Slack | Hedef konuşma için oluşturulmuş Incoming Webhook adresi. Normal Slack kanal bağlantısı yeterli değildir. |
| Microsoft Teams | Teams Workflows üzerinden oluşturulmuş HTTPS webhook adresi. Teams kanalının tarayıcı bağlantısı kullanılamaz. |

Kanal oluşturmak için bildirim oluşturma; düzenleme, açma/kapatma, silme ve **Test gönder** için bildirimleri düzenleme izni gerekir. Kanalın aktif olduğunu kontrol edin. Webhook adreslerini erişim bilgisi olarak saklayın; herkese açık dokümana, ekran görüntüsüne veya destek mesajına eklemeyin.

## E-posta: alıcıyı ekleyin, gerçek posta kutusunu kontrol edin

Kanal türünü **E-posta** seçin, ekibinizin kullanacağı alıcıları ekleyin ve kaydedin. Bu formda müşterinin Resend anahtarı veya SMTP sunucusu girmesi gerekmez; gönderim Komuta’nın e-posta altyapısı üzerinden Resend ile yapılır.

**Test gönder** sonrasında hedef posta kutusunu, spam/karantina klasörünü ve kurumunuzun e-posta filtrelerini kontrol edin. Birden fazla alıcılı bir kanalda başarılı sonuç, her alıcıya başarı anlamına gelmeyebilir; özellikle önemli alıcıları ayrı doğrulayın.

Komuta’daki e-posta teslim kaydı **sağlayıcının gönderimi kabul etmesi** düzeyindedir. Resend’in `email.sent` olayı da gönderim isteğinin kabulüyle, `email.delivered` ise alıcının posta sunucusuna teslimle ilgilidir. Bunlar mesajın gelen kutusunda görüldüğü veya okunduğu anlamına gelmez. Komuta ekranının bu sağlayıcı olaylarının tamamını gösterdiğini varsaymayın. [Resend olay açıklamaları](https://resend.com/docs/webhooks/event-types).

### Tam örnek: Docs test e-posta kanalı

1. **Uyarılar → Kanallar → Bildirim ayarlarında yönet** yolundan kanal ekleme formunu açın.
2. Adı **Docs test**, türü **E-posta** seçin. Ad en az iki karakter olmalıdır.
3. Gerçekten kontrol edebildiğiniz test alıcısını yazın ve **Ekle** düğmesine basın veya Enter kullanın. Adresin ayrı bir alıcı etiketi olarak listede görünmesi gerekir; yalnız giriş kutusunda bırakmayın.
4. İsterseniz “Dokümantasyon örneği için test hedefi” açıklamasını ekleyin. Kaydedin ve kanalın aktif listede göründüğünü doğrulayın.
5. Kanalın **Test gönder** işlemini kullanın. Komuta’daki sonucu ve gerçek posta kutusundaki mesajı birlikte kontrol edin. Birden fazla alıcı varsa her gerekli hedefi doğrulayın.
6. [İlk uyarı formuna](alerts-quick-start.md) dönün ve **Docs test** kanalını seçin. Kanal testinin geçmesi tek başına kuralı bu kanala bağlamaz.

![Örnek e-posta kanalı formu: 1 kanal adı, 2 e-posta türü, 3 listeye eklenmiş alıcı.](https://raw.githubusercontent.com/Microzon-Tech/komuta-docs/main/img/alerts/tr/channel-form.png)

*Gerçek form bileşeni örnek verilerle yerel olarak gösterilmiştir. Görseldeki `alerts@example.com` yalnız örnektir; kullanılabilir test alıcınızla değiştirin. Bu görsel hazırlanırken kanal oluşturulmamış ve mesaj gönderilmemiştir.*

**Alıcı eklenmiyorsa:** Adresin geçerli olduğunu ve listede zaten bulunmadığını kontrol edin. Virgül, noktalı virgül veya satırlarla ayrılmış bir listeyi yapıştırdıktan sonra oluşan etiketleri sayın; yazdığınız her adresin kabul edildiğini varsaymayın. Hatalı etiketi kaldırıp düzeltin.

## Slack: Incoming Webhook kullanın

Slack uygulamanız için Incoming Webhooks özelliğini açın, izin verilen hedef kanala bir webhook oluşturun ve üretilen adresi Komuta kanalına kaydedin. Özel bir kanala bağlanıyorsanız gerekli Slack erişiminiz de olmalıdır. [Slack’in kurulum rehberi](https://docs.slack.dev/messaging/sending-messages-using-incoming-webhooks/).

Test sonucunu hedef Slack kanalında doğrulayın. Webhook kaldırılır, kanal arşivlenir veya uygulama erişimi değişirse bildirim başarısız olabilir; yeni geçerli hedefle tekrar test edin.

## Microsoft Teams: kanal bağlantısı ile webhook farklıdır

1. Teams’te hedef kanalın **Workflows** bölümünü açın.
2. **Send webhook alerts to a channel** gibi uygun webhook şablonunu oluşturun; hedef ekip ve kanalı seçin.
3. Kaydedip oluşturulan webhook adresini kopyalayın; Komuta’da **Microsoft Teams** kanalına yapıştırın.
4. **Test gönder** ile deneyin; Teams kanalındaki mesajı ve gerekiyorsa workflow çalıştırma geçmişini kontrol edin.

Komuta bu bağlantıya webhook isteği gönderir; formda Microsoft kullanıcı oturumu veya OAuth bağlantısı kurmaz. Workflow’un kimlik doğrulama seçeneği bu çağrıyı kabul edebilmelidir. Kurumunuzun politikaları izin vermiyorsa Teams yöneticinizle uygun bağlantıyı belirleyin. [Microsoft’un webhook kurulum adımları](https://support.microsoft.com/en-us/workflows/send-messages-in-teams-using-incoming-webhooks).

Workflow’un çalışır durumda ve geçerli bir sahibinin olmasına dikkat edin. Gerekiyorsa ortak sahip ekleyin; sahibinin ayrılması akışı etkileyebilir. [Microsoft’un sahiplik ve Workflows açıklaması](https://learn.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook).

## Kuralı kanala yönlendirin

Uyarı oluşturma/düzenleme ekranında bildirim kanallarını açıkça seçin. Seçim boşsa uygun aktif kanallar kullanılır; “hiç kimseye gönderme” anlamına gelmez. Kanalın etkinliği ve yapılandırılmış şiddet filtresi de yönlendirmeyi etkiler.

**Bildirimler ve Uyarılar** sayfasındaki olay-kanal matrisi platform olaylarının abonelikleri içindir. Bir uyarı kuralının kendi kanal seçimiyle aynı ayar değildir. “Dağıtım tamamlandı” bildirimi almanız, belirli bir metrik/log kuralının doğru kanala yönlendiğini kanıtlamaz.

### Kanal hazır, kural bağlı: son kontrol

Örneğin **Uyarı** şiddetindeki `Docs sample alert` kuralını **Docs test** kanalına bağladınız. Kuralı yeniden açın: seçili kanal doğru mu? Kanal listesinde hedef aktif mi? Şiddet filtresi varsa Uyarı seviyesini kabul ediyor mu? Sonra kontrollü örneğin **Geçmiş** olayını ve ona ait gönderimi takip edin.

Test mesajı geliyor ama gerçek olayın mesajı gelmiyorsa bağlantıyı baştan kurmak yerine kuralın kanal seçimini, aktif sessizliği, tekrar aralığını ve başarısız/bastırılmış kayıtları inceleyin. Test de başarısızsa önce kanal yapılandırmasını düzeltin. [Karar adımları](alerts-troubleshooting.md), her sonuç için sonraki ekranı gösterir.

## Test ve teslim kayıtlarını ayırın

**Test gönder**, kanal bağlantısını dener. Kuralın metrik veya log kaynağını, yayınını, tetiklenmesini ya da çözülmesini test etmez. Test sonucu “gönderilmedi” ise kanalın aktifliğini, yapılandırmasını ve bildirim sınırlarını kontrol edin. Testin kendisi kanalın deneme istatistiklerine yansıyabilir; normal bir uyarı olayıyla aynı geçmiş satırını beklemeyin.

**Kanallar** sayfasında dönem, kanal, durum ve arama filtrelerini kullanın. Tetiklenme ve çözülme bildirimi türünü ayrı okuyun. Birden fazla aynı kayıt katlanmışsa ayrıntıları açın; bu, görüntüleme kolaylığıdır.

| Durum | Ne doğrular? |
| --- | --- |
| Bekleyen/işlenen kayıt | Ürünün gönderim akışında bir kayıt var; teslim tamamlanmış sayılmaz. |
| Başarılı/teslim edildi kaydı | Ürünün ilgili gönderim denemesi başarılı olarak kaydedilmiş. E-postada sağlayıcı kabulü; gerçek alıcı teyidi ayrıca gerekir. |
| Başarısız kayıt | Gönderim denemesi başarısız olmuş; hata ve hedef ayarları incelenmelidir. |
| Bastırılmış kayıt | Bildirim bir sınır veya kısıt nedeniyle gönderilmemiştir; teslim sayılmaz. |
| Kayıt yok | Seçili dönem/filtrede kayıt yoktur; kuralın tetiklenmediğini veya kanalın sağlıklı olduğunu tek başına kanıtlamaz. |

Dönemsel teslim istatistiği ile kanalın tüm zamanlar deneme sayısını karşılaştırırken kapsamı dikkate alın. Hiç deneme yoksa başarı oranı kanıtı da yoktur. Bildirim sınırları, tekrar aralıkları, susturmalar ve sağlayıcı hataları sonucu etkileyebilir; anlık, kesin veya yalnız bir kez teslim varsaymayın.

Sonraki adım: [Bildirim gelmiyorsa](alerts-troubleshooting.md) · [İlk uyarıyı oluştur](alerts-quick-start.md)
