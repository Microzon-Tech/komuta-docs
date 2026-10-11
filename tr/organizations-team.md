# Organizasyonlar ve Ekip Üyeleri

Komuta'da her şey bir organizasyonun içinde durur: servisleriniz, yönetilen servisleriniz, cüzdanınız ve harcama limitiniz organizasyona aittir. Bu sayfa organizasyon oluşturmayı, organizasyonlar arasında geçmeyi, ekip arkadaşlarını davet etmeyi ve kimin neyi yönetebileceğini belirlemeyi anlatır.

---

## Organizasyon nedir

Organizasyon, Komuta'daki çalışma alanınızdır. Şunlar organizasyona aittir ve başka bir organizasyondan görünmez:

- Projeler ve uygulamalar, yönetilen servisler, alan adları, entegrasyonlar (Git hesabı ve kayıt defteri bağlantıları) ve uyarılar.
- Cüzdan, harcama limiti, harcama seviyesi ve ekstreler.
- Üyeler ve her üyenin izinleri.

Kayıt olduğunuzda size ait bir organizasyon kendiliğinden açılır ve yöneticisi siz olursunuz. Adı e-posta adresinizden türetilir (örneğin `ayse-3f9c2a1b`); **Yeniden adlandır** ile istediğiniz adı verebilirsiniz.

| Kural | Ayrıntı |
|---|---|
| Organizasyon sayısı | Hesabınız 5 organizasyonda bulunduğunda yeni organizasyon oluşturulamaz. Oluşturduklarınız ve davetle katıldıklarınız birlikte sayılır. |
| Hoş geldin kredisi | Yalnızca kayıtla açılan ilk organizasyona yüklenir: 30 gün geçerli 10 USD. Sonradan oluşturduğunuz organizasyonlara kredi yüklenmez. |
| Harcama limiti | Her organizasyonun kendi limiti vardır. Yeni bir organizasyon da aylık 20 USD'lik limitle başlar. |
| Ücretsiz plan | Kişi başına tek organizasyonda kullanılır. Ücretsiz servisin reposunun sahibi olan kişisel Git hesabı (GitHub, GitLab ya da Azure DevOps) yalnızca bir organizasyona bağlanabilir; diğer organizasyonlar ücretli plan kullanır. |

> **Not:** Entegrasyonlar da organizasyona aittir. Yeni bir organizasyonda servis yayınlamadan önce Git hesabınızı o organizasyonun **Entegrasyonlar** sayfasından bağlayın.

---

## Organizasyonlar sayfası

**Hesap → Organizasyonlar** sayfası (sağ üstteki kullanıcı menüsünde **Hesabım**) organizasyon yöneticilerine açıktır. Sayfada şunlar bulunur:

- En üstte bulunduğunuz organizasyon, **Mevcut organizasyon** etiketiyle. Yanındaki **Yeniden adlandır** organizasyonun konsolda görünen adını değiştirir. Ad 2 ile 64 karakter arasında olmalıdır.
- Altında üyesi olduğunuz diğer organizasyonlar, her birinin yanında **Geç** düğmesiyle.
- **Organizasyon oluştur** düğmesi.
- Organizasyonun erişim koruması ayarları: **Dış paylaşıma izin ver**, **Doğrulanmış alan adları**, **Kimlik sağlayıcınızla ziyaretçi girişi**, **Yeni servisleri koru** ve **Erişim kaydı saklama süresi**. Bu ayarlar [Erişim Koruması](service-access-protection.md) dokümanında anlatılır.

Üyeler bu sayfayı göremez; organizasyonlar arasında kullanıcı menüsünden geçerler.

---

## Organizasyon oluşturma

1. Sağ üstteki kullanıcı menüsünü açın, organizasyon adının yazdığı satıra tıklayın ve **Organizasyon oluştur**'u seçin. Aynı düğme **Hesap → Organizasyonlar** sayfasında da vardır.
2. **Yeni organizasyon oluştur** penceresinde **Organizasyon adı (opsiyonel)** alanına bir ad yazın (en fazla 64 karakter). Boş bırakırsanız ad e-posta adresinizden türetilir.
3. **Oluştur**'a tıklayın.

Organizasyon oluşturulur ve konsol sizi hemen ona geçirir; yeni organizasyonun yöneticisi sizsiniz. Geçiş başarısız olursa düğme **Geçişi yeniden dene** olur. Bu düğme ikinci bir organizasyon açmaz, yalnızca oluşturulan organizasyona geçmeyi yeniden dener.

Organizasyon oluşturmak için e-posta adresinizin doğrulanmış olması gerekir. Doğrulanmamışsa pencerede şu uyarı çıkar: "Organizasyon oluşturmadan önce e-posta adresinizi doğrulayın. Gelen kutunuza gönderdiğimiz doğrulama bağlantısını açıp tekrar deneyin."

> **Not:** **Organizasyon oluştur**'u yalnızca bulunduğunuz organizasyonda yöneticiyseniz görürsünüz. Başka birinin organizasyonunda üyeyseniz önce kendi organizasyonunuza geçin.

---

## Organizasyonlar arasında geçiş

Birden fazla organizasyonda bulunuyorsanız iki yerden geçiş yapabilirsiniz:

- **Kullanıcı menüsü (herkes):** Sağ üstteki kullanıcı menüsünü açın ve organizasyon adının yazdığı satıra tıklayın. **Diğer Organizasyonlar** listesinden geçmek istediğinize tıklayın.
- **Hesap → Organizasyonlar (yöneticiler):** Listedeki organizasyonun yanındaki **Geç**'e tıklayın.

Geçiş sırasında "Organizasyon değiştiriliyor..." mesajı görünür. Ardından konsol yeniden yüklenir ve ana sayfa yeni organizasyonla açılır; menüler, listeler ve izinleriniz artık o organizasyona aittir.

> **Not:** Listede, bulunduğunuz organizasyona doğrudan bağlı organizasyonlar görünür. Kayıtla açılan ilk organizasyonunuzdayken orada oluşturduğunuz ve davetle katıldığınız organizasyonların hepsini görürsünüz. Başka bir organizasyondayken liste yalnızca onunla bağlantılı olanları, genellikle ilk organizasyonunuzu gösterir. Başka bir organizasyona geçmek için önce ilk organizasyonunuza dönün. Hepsinin tek listede görünmesi için yeni organizasyonları ilk organizasyonunuzdayken oluşturun.

---

## Erişim Yönetimi sayfası

Ekip arkadaşları, roller ve izinler sol menüdeki **Yönetim → Erişim Yönetimi** sayfasından ([console.komuta.io/access-control](https://console.komuta.io/access-control)) yönetilir. Sayfayı yalnızca organizasyon yöneticileri görür.

- Üstte üç sayı: **Toplam kullanıcı**, **Aktif** ve **Yöneticiler**.
- Solda **Kullanıcılar** listesi: arama kutusu, **Üye davet et** düğmesi ve her kişinin yanında **Yönetici** ya da **Üye** etiketi. Kendi satırınızda **Siz** yazar.
- Sağda seçtiğiniz kişinin rolü, kaynak izinleri ve **Modül yetkileri**; en altta **Bekleyen davetler**.

---

## Ekip arkadaşı davet etme

1. **Yönetim → Erişim Yönetimi** sayfasını açın.
2. **Kullanıcılar** kartında **Üye davet et**'e tıklayın.
3. **E-posta** alanına kişinin adresini yazın.
4. **Rol** olarak **Üye** ya da **Yönetici** seçin. Varsayılan **Üye**'dir.
5. **Davet gönder**'e tıklayın.

"Davet gönderildi" mesajı çıkar ve davet **Bekleyen davetler** listesine eklenir. Kişiye bir davet e-postası gider.

- Davet 72 saat geçerlidir. Kişi kabul edene kadar organizasyonda yer almaz.
- Kişi için şifre belirlemezsiniz; daveti kendi Komuta hesabıyla kabul eder ya da kabul ederken hesap açar.
- Aynı adrese yeniden davet gönderirseniz önceki bekleyen davet iptal edilir; yalnızca son e-postadaki bağlantı çalışır.
- Organizasyonda bu e-posta adresine sahip bir kullanıcı zaten varsa davet gönderilmez.

---

## Davet edilen kişi ne görür

1. **E-posta.** Konu satırı "Komuta'da {organizasyon} organizasyonuna davet edildiniz" şeklindedir. E-postada daveti gönderenin adı, **Daveti kabul et** düğmesi ve davetin sona ereceği tarih (UTC) bulunur.
2. **Katılma sayfası.** Bağlantı "{organizasyon} organizasyonuna katıl" başlıklı bir sayfa açar. Sayfa kişinin hangi rolle (**Üye** ya da **Yönetici**) davet edildiğini ve davetin hangi adrese gönderildiğini gösterir.
   - Komuta hesabı olan kişi **Kabul etmek için giriş yap**'a tıklar.
   - Hesabı olmayan kişi **Kabul etmek için hesap oluştur**'a tıklar.
3. **Giriş ya da kayıt.** Giriş ve kayıt sayfalarında "{organizasyon} organizasyonuna gelen bir daveti kabul ediyorsunuz." bandı görünür. E-posta alanı davet edilen adresle doldurulur ve değiştirilemez. Google ya da GitHub ile de devam edilebilir; bu durumda hesabın e-posta adresi davet edilen adresle aynı olmalıdır.
4. **Kabul.** Giriş yapınca sayfa **Daveti kabul et** düğmesini gösterir. **Şimdi değil** daveti kabul etmeden konsola götürür.
5. **Konsol.** Kabul ettikten sonra konsol açılır. Kişi kendi organizasyonunda başlar; katıldığı organizasyon kullanıcı menüsünde **Diğer Organizasyonlar** altında görünür ve oraya geçerek çalışır.

Bilmekte fayda olanlar:

- Davet bağlantısından e-posta ve şifreyle hesap açan kişi için e-posta doğrulama adımı atlanır: davet edilen adres doğrulanmış sayılır.
- Davetle hesap açan kişiye, kayıt olan herkes gibi, kendi organizasyonu da açılır.
- Daveti yalnızca davet edilen adresin sahibi kabul edebilir. Kişi başka bir adresle giriş yapmışsa sayfada "Bu davet {adres} adresine ait, ancak {diğer adres} olarak giriş yaptınız." yazar; **Farklı bir hesapla giriş yap** ile doğru hesaba geçilir. İletilen bir bağlantı başka bir hesapla kullanılamaz.
- Davetin süresi dolduysa, davet iptal edildiyse ya da yeniden gönderildiyse eski bağlantı **Bu daveti kabul edemedik** başlıklı bir sayfa açar. Kişi yöneticiden yeni bir davet istemelidir.

> **Not:** Davet e-postası ve katılma sayfası organizasyonu, konsolda görünen adıyla değil, oluşturulurken verilen kısa adıyla gösterir (örneğin `ayse-3f9c2a1b`). Organizasyonu yeniden adlandırdıysanız davet ettiğiniz kişiye bunu belirtin.

---

## Bekleyen davetler

**Erişim Yönetimi** sayfasının altındaki **Bekleyen davetler** kartı henüz kabul edilmemiş ve süresi dolmamış davetleri listeler: **E-posta**, **Roller**, **Durum**, **Son geçerlilik** ve **Davet eden**. Satırın sonundaki menüden:

| İşlem | Ne olur |
|---|---|
| **Tekrar gönder** | Kişiye yeni bir bağlantı gönderilir ve 72 saatlik süre yeniden başlar. Önceki e-postadaki bağlantı artık çalışmaz. |
| **İptal et** | Davet hemen geçersiz olur; bağlantı açılsa da kabul edilemez. |

Kabul edilen davet bu listeden çıkar ve kişi **Kullanıcılar** listesinde görünür. Süresi dolan davet de listeden kalkar; aynı kişiyi yeniden davet etmek için **Üye davet et**'i kullanın.

---

## Roller: Üye ve Yönetici

Her kişinin organizasyonda iki rolden biri vardır:

- **Yönetici** her kaynağa tam erişir, üyeleri ve izinleri yönetir, ödeme sayfalarını görür.
- **Üye** yalnızca bir yöneticinin izin verdiği kaynaklara erişir. Yeni katılan bir üye, kendisine izin verilene kadar proje ve uygulama listelerini boş görür.

| İşlem | Üye | Yönetici |
|---|---|---|
| Projeleri, uygulamaları, yönetilen servisleri, alan adlarını, entegrasyonları ve uyarıları görmek | Yalnızca kendisine **Okuma** ya da **Düzenleme** izni verilenleri | Hepsini |
| Bunları değiştirmek ve silmek | Yalnızca **Düzenleme** izni verilenleri | Hepsini |
| Yeni proje, uygulama, yönetilen servis, alan adı, entegrasyon ya da uyarı oluşturmak | O tür için oluşturma izni verildiyse | Evet |
| Ortam değişkeni gruplarını görmek | Evet | Evet |
| Ortam değişkeni grubu oluşturmak, düzenlemek, silmek | Modül yetkisi verildiyse | Evet |
| Servis güvenliği ayarları, erişim koruması ve paylaşımları, gelişmiş gözlemlenebilirlik, API gateway çağıranları | Modül yetkisi verildiyse | Evet |
| Stack oluşturmak, planlamak ve uygulamak | Hayır | Evet |
| Bildirim kanalı oluşturmak ve düzenlemek | Hayır | Evet |
| API anahtarı oluşturmak | Hayır | Evet |
| MCP ajanını bu organizasyona bağlamak | Hayır | Evet |
| Destek talebi açmak | Hayır | Evet |
| **Cüzdan**, **Güncel Harcama**, **Ekstreler**, **Ödeme Yöntemleri** ve **Fatura Profili** | Hayır | Evet |
| Üye davet etmek, rol değiştirmek, üye çıkarmak ve izin vermek | Hayır | Evet |
| Organizasyonu yeniden adlandırmak ve **Hesap → Organizasyonlar** ayarları | Hayır | Evet |
| Bu organizasyondayken yeni organizasyon oluşturmak | Hayır | Evet |

Bir üye, oluşturma izniyle yeni bir kaynak oluşturduğunda o kaynakta **Düzenleme** izni kendiliğinden ona verilir.

---

## Üyeye kaynak izni verme

1. **Erişim Yönetimi** sayfasında **Kullanıcılar** listesinden üyeyi seçin.
2. Üstteki sekmelerden kaynak türünü seçin: **Projeler**, **Uygulamalar**, **Entegrasyonlar**, **Alan Adları**, **Uyarılar** ya da **Yönetilen Servisler**.
3. Tabloda her kaynak için **Okuma** ya da **Düzenleme** kutusunu işaretleyin. Sütun başlığına tıklamak listede görünen kaynakların hepsini birden işaretler ya da kaldırır.
4. Üyenin bu türde yeni kaynak oluşturabilmesini istiyorsanız tablonun üstündeki **Bu kullanıcının yeni … oluşturmasına izin ver** anahtarını açın.
5. **Kaydet**'e tıklayın. "İzinler kaydedildi" mesajı çıkar.

- **Okuma** yalnızca görüntülemedir. **Düzenleme** değiştirmeyi ve silmeyi de kapsar ve **Okuma**'yı da açar. **Okuma** kaldırılırsa **Düzenleme** de kalkar.
- **Uygulamalar** ve **Yönetilen Servisler** sekmelerinde kaynaklar projelerine göre gruplanır; proje satırındaki kutu o projedeki kaynakların hepsini tek seferde işaretler. Bir projeye izin vermek içindeki uygulamalara izin vermez. Projeye sonradan eklenen kaynaklar için izin ayrıca verilir.
- Kaydetmeden başka bir sekmeye ya da kişiye geçerseniz **Kaydedilmemiş değişiklikler silinsin mi?** diye sorulur.
- Seçtiğiniz kişi yöneticiyse tablo ve anahtarlar kilitlidir; yöneticiler her kaynağa zaten erişir.

---

## Modül yetkileri

Kaynak tablosunun altındaki **Modül yetkileri** kartı, tek bir kaynağa bağlı olmayan yetkileri açar. Anahtarlar değiştirdiğiniz anda kaydedilir.

| Modül | Açılabilen yetkiler |
|---|---|
| **Servis Güvenliği** | Güvenlik duruşunu görüntüle, Linux capability yönet, Yazılabilir mount yönet, Root olarak çalıştırmayı yönet, Erişim korumasını yönet, Erişim paylaşımlarını yönet |
| **Gelişmiş gözlemlenebilirlik** | Gözlemlenebilirliği görüntüle, SLO düzenle, Anomalileri görüntüle, Doğal dil sorgusu, Özel pano düzenle, Aboneliği etkinleştir |
| **Ortam Değişkeni Grupları** | Ortam değişkeni gruplarını görüntüle, Ortam değişkeni grubu oluştur, Grupları ve değişkenlerini düzenle, Ortam değişkeni gruplarını sil |
| **API Gateway'ler** | API gateway çağıranlarını ve kullanım planlarını yönet |

Bu yetkiler üyenin zaten erişebildiği kaynaklarda işe yarar. Örneğin bir ortam değişkeni grubunu bir uygulamaya aktarmak için üyenin o uygulamada **Düzenleme** izni de olmalıdır; API gateway çağıranları da yalnızca üyenin görebildiği gateway'lerde yönetilir.

---

## Rol değiştirme ve üyeyi çıkarma

**Rolü değiştirmek için:**

1. **Erişim Yönetimi** sayfasında kişiyi seçin.
2. Kişinin adının yanındaki rol listesinden **Üye** ya da **Yönetici**'yi seçin.
3. Açılan pencerede **Rolü değiştir**'e tıklayın.

Yöneticiliğe yükseltilen kişi tüm kaynaklara tam erişim kazanır ve üyeleri yönetebilir. Bir yönetici **Üye** yapılırsa erişimi sıfırlanır: bütün kaynak izinleri ve modül yetkileri kaldırılır; gerekenleri ardından yeniden verirsiniz.

**Organizasyondan çıkarmak için:** Kişiyi seçin, **Organizasyondan çıkar**'a tıklayın ve pencerede yeniden **Organizasyondan çıkar** ile onaylayın. Kişinin erişimi hemen kesilir ve bu organizasyondaki oturumu kapatılır. Kendi Komuta hesabı ve kendi organizasyonu kalır; çıkarıldığı organizasyon onun listesinden kalkar. Onu daha sonra yeniden davet edebilirsiniz.

İki kural geçerlidir:

- Kendi rolünüzü değiştiremez, kendinizi organizasyondan çıkaramazsınız. Bunu başka bir yöneticiden isteyin.
- Organizasyonda en az bir aktif yönetici kalmalıdır. Tek aktif yönetici için kartta "Bu, tek aktif yönetici. Önce başka birini yönetici yapın." yazar.

---

## Organizasyondan ayrılma ve organizasyonu silme

Konsolda bir organizasyondan kendiniz ayrılamazsınız. O organizasyonun bir yöneticisinden sizi **Organizasyondan çıkar** ile çıkarmasını isteyin. Organizasyonun tek yöneticisi sizseniz önce başka birini yönetici yapın.

Konsolda organizasyon silme işlemi yoktur. Bir organizasyonu kapatmanız gerekiyorsa [destek talebi](support-tickets.md) açın.

---

## Başka bir organizasyonda çalışırken ödeme ve limitler

Ödeme ve limitler kişiye değil organizasyona bağlıdır. Başka birinin organizasyonuna geçtiğinizde:

- Orada oluşturduğunuz her şey o organizasyonun cüzdanından ödenir ve o organizasyonun harcama limitine sayılır. Kendi organizasyonunuzun bakiyesi ve limiti kullanılmaz.
- Yeni bir şey oluşturmaya ya da planını büyütmeye çalıştığınızda o organizasyonun limiti kontrol edilir.
- Ücretsiz plan kuralları da o organizasyona göre uygulanır; her organizasyon bir servisi ücretsiz pakette çalıştırabilir.
- Ödeme sayfalarını ve kullanıcı menüsündeki bakiye kartını yalnızca o organizasyonun yöneticileri görür.

Fiyatlar, harcama seviyeleri ve limitin nasıl yükseldiği [Faturalandırma](billing.md) ve [Limitler](limits.md) dokümanlarındadır.

---

## Servisleri başka organizasyonlarla paylaşma

Erişim koruması açık bir servisi, üyesi olduğunuz başka bir organizasyondaki herkesle **Bağlı bir organizasyon** paylaşımıyla açabilirsiniz. Bu bir dış paylaşımdır ve organizasyonunuzda **Dış paylaşıma izin ver** ayarının açık olmasını gerektirir. Ayrıntılar [Giriş ve Paylaşım](access-protection-sign-in-sharing.md) dokümanındadır.

> **Not:** **Erişim Yönetimi**, Komuta konsolunda kimin hangi kaynağı görüp yönetebileceğini belirler. Yayındaki bir servisin internet adresini kimin açabileceği ise ayrı bir konudur ve [Erişim Koruması](service-access-protection.md) ile ayarlanır. Bir kişinin korumalı bir servisi açıp açamayacağını o servisin paylaşımları belirler; servis **Organizasyonunuz** ile paylaşıldıysa organizasyonun bütün aktif üyeleri onu açabilir.

---

## İlgili Dokümanlar

- [Hızlı Başlangıç](quick-start.md) — Hesap açma, ilk organizasyon ve ilk servis.
- [Erişim Koruması](service-access-protection.md) — Yayındaki servisi kimin açabileceği ve organizasyon ayarları.
- [Giriş ve Paylaşım](access-protection-sign-in-sharing.md) — Servisleri kişilerle, organizasyonlarla ve alan adlarıyla paylaşma.
- [Faturalandırma](billing.md) — Cüzdan, ekstreler ve ödeme.
- [Limitler](limits.md) — Harcama limiti ve organizasyon başına sınırlar.
- [Ortam Değişkeni Grupları](environment-groups.md) — Grupları oluşturma ve uygulamalara aktarma.
