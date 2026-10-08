# Uyarılar: Genel Bakış

Komuta Uyarılar ile servislerinizin metriklerini ve loglarını izleyin; ilgilenmeniz gereken durumları e-posta, Slack veya Microsoft Teams üzerinden ekibinize ulaştıracak kurallar oluşturun. Şablonla başlayabilir, bir log satırını uyarıya dönüştürebilir ve aynı kuralları servis detaylarından ya da genel **Uyarılar** menüsünden yönetebilirsiniz.

[İlk uyarını oluştur](alerts-quick-start.md) · [Şablon seç](alerts-templates.md) · [Bildirim kanalını bağla](notification-guide.md)

## Nereden başlamalıyım?

| Yapmak istediğiniz | İzlenecek yol | Yolun sonunda |
| --- | --- | --- |
| İlk kez uyarı kurmak | [Doldurulmuş log örneği](alerts-quick-start.md) → kanal testi → olay ve çözülme | Bir log satırını gerçek hedefteki mesaja kadar takip edebilirsiniz. |
| Servis için anlamlı bir koşul seçmek | [Şablon karar tablosu](alerts-templates.md) → normal yükü inceleme → parametreler | Ölçümün birimini ve eşiği neden seçtiğinizi bilirsiniz. |
| Mevcut kuralı değiştirmek | [Alanlar ve güvenli değişiklik örneği](alerts-rules.md) → yayın kontrolü | Kayıtlı ayarla yayın gözlemini ayırabilirsiniz. |
| Gelen mesajı araştırmak | [Örnek olay kaydı](alerts-history.md) → doğrulanmış kapsam → ilgili servis | Hangi olayı ve hangi kaynağı araştıracağınızı seçebilirsiniz. |
| Mesajın neden gelmediğini bulmak | [Sorun giderme karar adımları](alerts-troubleshooting.md) | Sorunu veri, kural, yayın veya teslim aşamasına daraltabilirsiniz. |
| Bakım sırasında bildirimleri susturmak | [30 dakikalık bakım örneği](alerts-silences.md) | Seçili kurallar için başlangıcı ve bitişi belli bir pencere oluşturabilirsiniz. |

## Temel kavramlar

**Kapsam**, kuralın hangi servis, kendi cluster’ınız veya hesap düzeyi kayıtla ilgili olduğunu anlatır. **Metrik**, CPU yüzdesi gibi sayısal bir ölçümdür; **log** uygulamanın ürettiği metin kaydıdır. Log uyarısı da bu metinlerden sayısal bir koşul üretir.

**Eşik** karşılaştırılan sınırdır. **Veri penceresi**, her değerlendirmede ne kadar geçmişe bakıldığını; **süre**, koşulun tetiklenmeden önce ne kadar devam etmesi gerektiğini anlatır. **Bildirim aralığı** aynı durumun tekrar gönderimleriyle ilgilidir. Örneğin “son 5 dakikada 0’dan fazla eşleşme, 1 dakika sürsün, tekrar aralığı 15 dakika” üç ayrı zaman/koşul seçimidir.

**Şiddet**, olayın önceliğidir. **Kanal**, mesajın hedefidir. **Olay**, Komuta’nın aldığı tetiklenme/çözülme kaydıdır. **Sessizlik**, seçili kuralların bildirimlerini belirli bir süre susturur. Bu kavramların birlikte nasıl çalıştığını [ilk örneğin zaman çizelgesinde](alerts-quick-start.md) görebilirsiniz.

## Bir uyarının dört adımı

<ol class="docs-alert-flow" aria-label="Uyarı akışı">
<li><span>01</span><strong>Kuralı tanımla</strong><p>Servisi, koşulu, süreyi ve bildirim kanallarını seç.</p></li>
<li><span>02</span><strong>Yayını kontrol et</strong><p>Kaydedilen tanımın yayın durumunu ve kontrol zamanını incele.</p></li>
<li><span>03</span><strong>Olayı takip et</strong><p>Geçmişte son alınan tetiklenme veya çözülme kaydını gör.</p></li>
<li><span>04</span><strong>Bildirimi doğrula</strong><p>Kanal kaydını ve mesajın gerçek hedefe ulaştığını ayrı kontrol et.</p></li>
</ol>

**Kuralın kaydedilmesi, yayınlanması, tetiklenmesi ve bildirimin alınması farklı aşamalardır.** Bir aşamadaki başarı, sonraki aşamanın tamamlandığını göstermez.

## Uyarılar menüsünde neler var?

| Ekran | Ne için kullanılır? |
| --- | --- |
| **Genel Bakış** | Kural ve yayın özetini, son olayları, sık tekrarlayan kuralları, başlangıç şablonlarını ve yetkiniz varsa bildirim durumunu görmek. |
| [**Kurallar**](alerts-rules.md) | Kuralları aramak, filtrelemek, oluşturmak ve izin verilen işlemleri yapmak. |
| [**Geçmiş**](alerts-history.md) | Son alınan durumu, kapsamı, tetiklenme/çözülme zamanlarını ve mevcut olay ayrıntılarını incelemek. |
| [**Sessizlikler**](alerts-silences.md) | Seçili kurallar için belirli bir zaman aralığında bildirimleri susturmak. |
| [**Şablonlar**](alerts-templates.md) | Servise veya kendi cluster’ınıza uygun hazır koşullardan başlamak. |
| [**Kanallar**](notification-guide.md) | Tanımlı kanalları ve teslim kayıtlarını incelemek; kanal yönetimi için bildirim ayarlarına geçmek. |

Genel Bakıştaki sayılar yüklenen kapsam ve belirtilen dönemle ilgilidir. Veri alınamadıysa bunu “sıfır olay” veya “her şey sağlıklı” şeklinde yorumlamayın. **Çözülme bekleyen** kayıt sayısı da şu anda tetiklenen uyarıların canlı sayımı değildir.

## Hangi kapsamı seçmeliyim?

| Durumunuz | Uygun başlangıç |
| --- | --- |
| Komuta’da servisiniz var; kendi cluster’ınız yok | **Uygulama Düzeyi** seçin. Servis metrik şablonları, log şablonları ve metin eşleşmesi kullanılabilir. Servisin altyapı adreslerini girmeniz gerekmez. |
| Kendi cluster’ınız da var | Servis kurallarına ek olarak, listede sunulan kendi cluster’ınız için **Cluster Düzeyi** metrik şablonlarını kullanabilirsiniz. Uygun kapsamda gelişmiş metrik sorgusu da oluşturabilirsiniz. |
| Hesap genelindeki bant genişliği uyarısını inceliyorsunuz | [Kapsam açıklamasını](alerts-history.md) okuyun. Hesap toplamı, hangi servisin sorumlu olduğunu tek başına göstermez. |

Paylaşılan altyapıda özel metrik sorgusu yerine servis şablonları kullanılır. Özel log sorguları servis kapsamıyla sınırlıdır; cluster genelinde log kuralı oluşturulmaz. Kendi cluster listenizin boş olması, servis uyarılarını kullanamayacağınız anlamına gelmez.

## Görüntüleme ve değiştirme izinleri

İşlemler hesabınızın rolündeki izinlere ve ilgili kaynağa erişiminize bağlıdır; yalnız rolün adına bakarak yetki varsaymayın.

| İşlem | Gerekli izin türü |
| --- | --- |
| Genel Bakış, Kurallar, Geçmiş, Sessizlikler ve Şablonlar; kural testi | Uyarıları okuma |
| Kural oluşturma, şablondan oluşturma, çoğaltma | Uyarı oluşturma |
| Düzenleme, etkinleştirme/kapatma, silme, yayın işlemleri ve susturma | Uyarıları düzenleme; ilgili kural için değişiklik erişimi |
| Servis seçimi ve servise uygun seçeneklerin doğrulanması | Servisleri okuma ve ilgili servise erişim |
| Uyarılar → Kanallar ve kanal seçimi | Bildirimleri okuma; Uyarılar ekranları için ayrıca uyarıları okuma |
| Kanal ekleme | Bildirim oluşturma |
| Kanal düzenleme, etkinleştirme, silme, test gönderme ve bildirim yönlendirmesi | Bildirimleri düzenleme |

Okuma izni, test mesajı gönderme veya kuralı değiştirme izni vermez. İzinler yüklenirken işlem düğmeleri kullanılamayabilir. Bildirimleri okuma izni olan kullanıcı, genel ayarları görüntüleme izni olmasa da bildirim sayfasına erişebilir; değiştirme izinleri ayrıca değerlendirilir.

## Maskotla yardım

Uyarı sihirbazındaki **Rehber** ile kapsam, şablon, parametre ve kanal adımlarını izleyebilirsiniz. Servis uyarıları, logdan uyarı oluşturma ve uyarıları takip etme için de rehber turları bulunur.

Maskota “Bu ekrandaki yayın durumu neyi doğruluyor?” veya “Bu kayıt bir servise mi, hesabın tamamına mı ait?” diye sorabilirsiniz. Yardım, sayfanın paylaştığı doğrulanmış kapsam, yükleme/hata durumu, görünür kayıt özeti ve form adımına dayanır. Bu bağlam bütün geçmişi, ham logları veya alıcının posta kutusunu temsil etmez. Maskot erişim izinlerini aşamaz; bilinmeyen kapsamı veya teslimi doğrulanmış gibi kabul etmeyin. Tur, alanları açıklar ve yönlendirir; oluşturma ve diğer değişiklikleri ilgili formda gözden geçirip tamamlayın.

## Önerilen kullanım sırası

1. Bir [bildirim kanalı](notification-guide.md) ekleyin ve gerçek hedefte test edin.
2. Tek servis ve tek koşulla [ilk uyarıyı](alerts-quick-start.md) oluşturun.
3. [Yayın durumunu](alerts-rules.md), ardından [olay geçmişini](alerts-history.md) kontrol edin.
4. Gereksiz tekrarları eşik, süre ve [susturma pencereleriyle](alerts-silences.md) azaltın.

Takıldığınız adımı [sorun giderme rehberinden](alerts-troubleshooting.md) inceleyin.
