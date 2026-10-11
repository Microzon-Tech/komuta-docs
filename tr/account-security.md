# Hesap Güvenliği

Bu sayfa, Komuta hesabınızın giriş güvenliğini yöneteceğiniz yerleri anlatır: parola, iki adımlı doğrulama, açık oturumlar, güvenlik kayıtları, Google ve GitHub ile giriş ve API anahtarları. Bu ayarlar kişiseldir; organizasyondaki diğer üyeleri etkilemez. Üye davet etme ve rol verme gibi organizasyon ayarları [Organizasyon ve Ekip](organizations-team.md) sayfasındadır.

Hepsi **Hesap** sayfasının **Kişisel** grubunda yer alır:

| Sayfa | Adres | Ne işe yarar |
|---|---|---|
| **Parola** | https://console.komuta.io/account/password | Parolanızı değiştirirsiniz. |
| **İki Adımlı Doğrulama** | https://console.komuta.io/account/two-factor | Girişte doğrulayıcı uygulamadan kod istenmesini açar veya kapatırsınız. |
| **Oturumlar** | https://console.komuta.io/account/sessions | Hesabınıza giriş yapmış cihazları görür, istemediğinizi kapatırsınız. |
| **Güvenlik Günlükleri** | https://console.komuta.io/account/security-logs | Hesabınızın giriş ve güvenlik olaylarını incelersiniz. |
| **Dış Girişler** | https://console.komuta.io/account/external-logins | Bağlı Google ve GitHub girişlerini görür, kaldırırsınız. |
| **API Anahtarları** | https://console.komuta.io/account/api-keys | Kendi araçlarınızın Komuta API'sini kullanması için anahtar oluşturur, iptal edersiniz. |

---

## Parolayı Değiştirme

1. **Hesap → Parola** sayfasını açın.
2. **Mevcut Parola** alanına şu anki parolanızı yazın.
3. **Yeni Parola** ve **Yeni Parolayı Onayla** alanlarına yeni parolanızı iki kez yazın. Form en az 6 karakter ister ve iki alanın aynı olmasını bekler.
4. **Parolayı Sıfırla** düğmesine tıklayın. Değişiklik başarılıysa **Parola başarıyla değiştirildi** uyarısı çıkar ve form temizlenir.

Mevcut parolanız yanlışsa veya yeni parola Komuta'nın parola kurallarına uymuyorsa **Parola değiştirilemedi** uyarısı çıkar. Uyarı, nedeni ayrıca belirtmez; önce mevcut parolanızı kontrol edin, sonra daha uzun bir parola deneyin.

> **Not:** Hesabınızı Google veya GitHub ile açtıysanız henüz bir parolanız yoktur. Parola belirlemek için oturumu kapatın ve aşağıdaki **Şifremi unuttum** adımlarını izleyin.

---

## Parolanızı Unuttuysanız

1. Giriş sayfasında **Şifremi unuttum** bağlantısına tıklayın.
2. Hesabınızın e-posta adresini yazın ve **Sıfırlama bağlantısı gönder** düğmesine tıklayın.
3. **E-postanızı kontrol edin** sayfası açılır. Bu sayfa, adres bir hesaba ait olsun ya da olmasın aynı şekilde görünür; böylece bir e-posta adresinin Komuta'da kayıtlı olup olmadığı anlaşılamaz.
4. Gelen e-postadaki **Şifreyi sıfırla** düğmesine tıklayın. Bağlantı 24 saat geçerlidir. E-posta gelmediyse spam klasörünüze bakın.
5. Açılan **Şifre Yenileme** sayfasında yeni parolanızı iki kez yazın ve **Şifremi sıfırla** düğmesine tıklayın.
6. "Şifreniz başarıyla sıfırlandı." mesajını gördükten sonra **Uygulamaya git** ile giriş yapın.

Bağlantının süresi dolduysa veya bağlantı bozulduysa sayfada "Bu parola sıfırlama bağlantısı geçersiz veya süresi dolmuş. Devam etmek için yeni bir bağlantı isteyin." yazar. **Yeni sıfırlama bağlantısı iste** ile baştan başlayın.

### Hesap kilitlenirse

Arka arkaya çok sayıda yanlış parola girilirse hesap bir süre kilitlenir ve **Kilitli!** sayfası açılır. Sayfa, kilidin ne zaman açılacağını UTC saatiyle gösterir ("Hesabınız şu tarihte kilidi açılacak:"). Bu saatten sonra tekrar deneyin. Parolanızdan emin değilseniz kilit açıldıktan sonra **Şifremi unuttum** ile yeni parola belirleyin.

---

## İki Adımlı Doğrulama

İki adımlı doğrulama açıkken e-posta ve parolayla giriş yaptığınızda telefonunuzdaki doğrulayıcı uygulamanın ürettiği 6 haneli kod da istenir. Parolanız başkasının eline geçse bile kod olmadan giriş yapılamaz. Kullanımı isteğe bağlıdır ve her kullanıcı kendisi için açar; varsayılan olarak kapalıdır.

Google Authenticator, Authy veya TOTP destekleyen herhangi bir doğrulayıcı uygulamayı kullanabilirsiniz.

### Açma

1. **Hesap → İki Adımlı Doğrulama** sayfasını açın. Sayfanın başında **İki Faktörlü Doğrulama** durumu yer alır; kapalıyken "İki faktörlü doğrulama henüz etkin değil" yazar ve **Etkin değil** rozeti görünür.
2. **Doğrulayıcı Kurulumu** bölümündeki QR kodunu doğrulayıcı uygulamanızla tarayın. Telefonda çalışıyorsanız **Doğrulayıcı uygulamada aç** bağlantısını kullanabilirsiniz. Tarama yapamıyorsanız **Veya bu anahtarı elle girin** altındaki anahtarı uygulamaya yazın.
3. Uygulamanın gösterdiği 6 haneli kodu **Doğrulamak için uygulamanızdaki 6 haneli kodu girin** alanına yazın ve **Doğrula** düğmesine tıklayın. **Doğrulayıcı doğrulandı** uyarısı çıkar.
4. **Kurtarma Kodları** bölümü açılır. Kodları **Tümünü Kopyala** ile kopyalayıp güvenli bir yere kaydedin (aşağıya bakın).
5. Durum hâlâ **Etkin değil** gösteriyorsa **2FA'yı Etkinleştir** düğmesine tıklayın. **2FA etkinleştirildi** uyarısından sonra durum "İki faktörlü doğrulama etkin" olur ve **Etkin** rozeti görünür.

**2FA'yı Etkinleştir** düğmesi, bir doğrulayıcı uygulama doğrulanmadan görünmez. Kod yanlışsa bir hata uyarısı çıkar; uygulamadaki güncel kodla tekrar deneyin.

### Kurtarma kodları

**Doğrula** başarılı olduğunda Komuta 10 kurtarma kodu üretir. Telefonunuza erişiminizi kaybederseniz giriş yaparken kod yerine bunlardan birini kullanırsınız. Her kod yalnızca bir kez kullanılabilir.

> **Uyarı:** Kurtarma kodları yalnızca **Doğrula** işleminden hemen sonra gösterilir. Sayfadan ayrıldıktan sonra tekrar görüntülenemez. Kodları kaybettiyseniz uygulamadaki güncel kodla **Doğrula** işlemini yeniden yapın; yeni 10 kod oluşur ve eskileri geçersiz olur.

### Girişte kod

1. Giriş sayfasında e-posta adresinizi ve parolanızı girin.
2. **İki fazlı doğrulama** sayfası açılır. **Seçilen sağlayıcı** listesinden **Authenticator** seçeneğini seçin ve devam edin. Listede **Email** de görünebilir; onu seçerseniz kod e-posta adresinize gönderilir.
3. Doğrulayıcı uygulamadaki 6 haneli kodu yazın ve devam edin. Sayfada **Bu tarayıcıyı hatırla** kutusu varsa ve kendi cihazınızdaysanız işaretleyebilirsiniz; bu tarayıcıda kod bir süre tekrar sorulmaz.

### Telefonunuza erişemiyorsanız

1. **İki fazlı doğrulama** sayfasında **Authenticator** seçiliyken "Doğrulayıcı cihazınıza erişiminiz yok mu? Bir kurtarma kodu ile giriş yapabilirsiniz" bağlantısına tıklayın.
2. **Kurtarma kodu ile giriş yapın** sayfasında kodlarınızdan birini **Kurtarma kodu** alanına yazın ve devam edin.
3. Giriş yaptıktan sonra yeni telefonunuzda doğrulayıcıyı yeniden kurun (aşağıdaki **Doğrulayıcıyı Sıfırla** adımları) ve yeni kurtarma kodlarını kaydedin.

Hem telefonunuzu hem de kurtarma kodlarınızı kaybettiyseniz ve hesabınıza bağlı bir Google veya GitHub girişi varsa onunla giriş yapabilirsiniz (aşağıdaki nota bakın). Bu da yoksa Komuta destek ekibiyle iletişime geçin.

### Kapatma ve sıfırlama

- **2FA'yı Devre Dışı Bırak**: Girişte kod istenmesini kapatır; **2FA devre dışı bırakıldı** uyarısı çıkar. Doğrulayıcı uygulama hesabınıza bağlı kalır, sonra **2FA'yı Etkinleştir** ile yeniden tarama yapmadan açabilirsiniz.
- **Doğrulayıcıyı Sıfırla**: Uygulama bağlantısını ve kurtarma kodlarını siler, yeni bir QR kodu oluşturur. Telefonunuzu değiştirdiğinizde veya anahtarın başkasının eline geçtiğinden şüphelendiğinizde kullanın. Onay sorulmadan hemen uygulanır ve **Doğrulayıcı sıfırlandı** uyarısı çıkar. Uygulamadaki eski Komuta kaydı artık çalışmaz; yeni QR kodunu tarayıp **Doğrula** ile hemen yeniden kurun.

**QR Kodunu Yenile** yeni bir anahtar oluşturmaz, yalnızca ekrandaki bilgiyi yeniler.

> **Not:** Kod, e-posta ve parolayla girişte istenir. **Google ile devam et** veya **GitHub ile devam et** ile giriş yaptığınızda Komuta kod sormaz; bu girişin güvenliği o sağlayıcıdaki hesabınıza bağlıdır. Bu yüzden Google ve GitHub hesaplarınızda da iki adımlı doğrulamayı açın.

---

## Aktif Oturumlar

**Hesap → Oturumlar** sayfasındaki **Aktif Oturumlar** listesi, hesabınıza şu anda giriş yapmış cihaz ve tarayıcıları gösterir.

| Sütun | Ne gösterir |
|---|---|
| **Cihaz** | Giriş türü ve altında tarayıcı ya da istemci bilgisi. Bu tarayıcının satırında **Mevcut** işareti vardır. |
| **IP Adresi** | Oturumun geldiği IP adresleri. |
| **Son Erişim** | Oturumun en son kullanıldığı zaman. |
| **Giriş Zamanı** | Girişin yapıldığı zaman. |

Tanımadığınız ya da artık kullanmadığınız bir oturumu kapatmak için satırdaki **Sonlandır** düğmesine tıklayın. Onay sorulmaz; **Oturum sonlandırıldı** uyarısı çıkar ve o cihaz yeniden giriş yapmak zorunda kalır.

- **Mevcut** işaretli satırda **Sonlandır** yoktur. Bu oturumu kapatmak için çıkış yapın.
- 7 gün boyunca kullanılmayan oturumlar otomatik olarak kapatılır.
- API anahtarları oturum açmaz, bu listede görünmez. Bir anahtarı durdurmak için onu **API Anahtarları** sayfasından iptal edin.

---

## Güvenlik Kayıtları

**Hesap → Güvenlik Günlükleri** sayfasındaki **Güvenlik Kayıtları** listesi, hesabınız için kaydedilen son giriş ve güvenlik olaylarını gösterir. Beklemediğiniz bir giriş olup olmadığını buradan kontrol edersiniz.

| Sütun | Ne gösterir |
|---|---|
| **İşlem** | Olayın adı (aşağıdaki tabloya bakın). |
| **Uygulama** | Olayı kaydeden Komuta bileşeni. |
| **IP Adresi** | İsteğin geldiği IP adresi. |
| **Tarih** | Olayın zamanı. |

Sayfa en yeni 100 olayı yükler ve sayfa başına 10 olay gösterir. "Kayıtlarda ara..." alanı, yüklenen olaylar içinde işlem adına, uygulamaya ve IP adresine göre arar. Sütun başlıklarına tıklayarak sıralayabilirsiniz.

**İşlem** sütununda görebileceğiniz olaylar:

| İşlem | Anlamı |
|---|---|
| `LoginSucceeded` | Başarılı giriş. |
| `LoginFailed` | Yanlış parolayla başarısız giriş denemesi. |
| `LoginRequiresTwoFactor` | Parola doğru girildi, iki adımlı doğrulama kodu istendi. |
| `LoginLockedout` | Hesap kilitli olduğu için giriş reddedildi. |
| `LoginNotAllowed` | Girişe izin verilmedi; örneğin e-posta adresi henüz doğrulanmamış. |
| `Logout` | Çıkış yapıldı. |
| `ChangePassword` | Parola **Parola** sayfasından değiştirildi. |
| `ChangeEmail` | Profilde e-posta adresi değiştirildi. |
| `ExternalLoginRegister` | Hesap Google veya GitHub ile oluşturuldu. |
| `ExternalLoginLink` | Bir Google veya GitHub hesabı, onunla ilk giriş yapıldığında hesabınıza bağlandı. |
| `GitHubLinked` | Bir GitHub hesabı konsoldan hesabınıza bağlandı. |

Tanımadığınız bir IP adresinden `LoginSucceeded` görürseniz hemen parolanızı değiştirin, **Oturumlar** sayfasında tanımadığınız oturumları sonlandırın ve API anahtarlarınızı gözden geçirin.

İki adımlı doğrulamayı açıp kapatmak, bir dış girişi kaldırmak ve API anahtarı oluşturup iptal etmek bu listede görünmez. Organizasyonun tüm üyelerinin giriş olayları, yetkiniz varsa [Güvenlik Merkezi](security-center-guide.md) içindeki **Oturum Aktivitesi** sayfasındadır.

---

## Dış Girişler: Google ve GitHub

Komuta'ya e-posta ve parolanın yanında Google veya GitHub hesabınızla da giriş yapabilirsiniz. **Hesap → Dış Girişler** sayfasındaki **Harici Girişler** listesi, hesabınıza bağlı sağlayıcıları gösterir. Her satırda sağlayıcının adı ve o sağlayıcıdaki hesap kimliğiniz yer alır. Bağlı giriş yoksa **Bağlı harici giriş yok** yazar.

### Bağlama

Konsolda ayrı bir bağlama düğmesi yoktur. Giriş sayfasında **Google ile devam et** veya **GitHub ile devam et** ile giriş yaptığınızda, sağlayıcının doğrulanmış e-posta adresi Komuta hesabınızın e-posta adresiyle aynıysa sağlayıcı hesabınıza otomatik olarak bağlanır. Sonraki girişlerde aynı düğmeyi kullanabilirsiniz.

Bağlama yapılamazsa giriş sayfasında şu mesajlardan biri görünür:

| Mesaj | Ne yapmalı |
|---|---|
| "Bu dış hesap başka bir Komuta hesabına bağlı. Bunun yerine o hesapla giriş yapın." | Bu Google/GitHub hesabı başka bir Komuta hesabında kullanılıyor. O hesapla giriş yapın veya oradan kaldırın. |
| "Sağlayıcınızdan doğrulanmış bir e-posta adresi alınamadı. Sağlayıcıdaki birincil e-posta adresinizi doğrulayıp tekrar deneyin." | Sağlayıcıdaki birincil e-posta adresinizi doğrulayın ve tekrar deneyin. |
| "Bu e-posta adresini doğrulanmamış bir hesap zaten kullanıyor. …" | Bu adresle açılmış hesap henüz doğrulanmamış. Önce o hesabın doğrulama e-postasındaki bağlantıya tıklayın. |

Bu sayfada **Ücretsiz plan Git hesabı** kartını da görebilirsiniz. Kartta **GitHub'ı bağla** düğmesi çıkarsa, bağladığınız GitHub hesabı organizasyonun ücretsiz planına bağlanır; bu GitHub hesabı henüz bir giriş yönteminiz değilse onayı GitHub'da verirsiniz ve hesabınıza giriş yöntemi olarak da eklenir. Ücretsiz planın kuralları için [Faturalandırma](billing.md) sayfasına bakın.

### Kaldırma

1. Kaldırmak istediğiniz sağlayıcının satırında **Kaldır** düğmesine tıklayın.
2. **Harici girişi kaldır** penceresinde sağlayıcının hesabınızdan ayrılacağı yazar. **Kaldır** ile onaylayın.
3. **Harici giriş kaldırıldı** uyarısı çıkar.

> **Uyarı:** Hesabınızı Google veya GitHub ile açtıysanız ve parola belirlemediyseniz, kaldırmadan önce **Şifremi unuttum** ile bir parola belirleyin. Ayrıca, kaldırdığınız sağlayıcıyla giriş sayfasından yeniden giriş yaparsanız ve e-posta adresi hesabınızla aynıysa sağlayıcı hesabınıza tekrar bağlanır. Bir Google veya GitHub hesabının elinizden çıktığını düşünüyorsanız önce o sağlayıcıdaki hesabınızı güvene alın.

---

## API Anahtarları

API anahtarı, kendi agent'ınızın, betiklerinizin veya CI işlerinizin Komuta API'sini (`https://api.komuta.io`) sizin adınıza kullanmasını sağlar. Anahtar sizin kullanıcınız gibi davranır:

- Anahtarı oluşturduğunuz organizasyonda, sizin o anki rolünüz ve izinlerinizle çalışır. Rolünüz değişirse anahtarın yapabildikleri de değişir.
- Yalnızca o organizasyonda geçerlidir. Başka bir organizasyon için o organizasyona geçip ayrı bir anahtar oluşturun.
- Konsola giriş için kullanılamaz; parolanın yerine geçmez.

Yalnızca tek bir servisi CI'dan deploy etmek istiyorsanız API anahtarı yerine o servise bağlı ve izinleri sınırlı **dağıtım token'ı** kullanın, bkz. [CI'dan Deploy Tetikleme](ci-deploy.md). Claude veya Codex'i Komuta MCP'ye bağlamak için tarayıcıyla giriş yeterlidir, bkz. [Komuta MCP Sunucusunun Kurulumu](mcp-setup.md).

### Kimler oluşturabilir

Varsayılan rollerde anahtar oluşturma ve iptal etme organizasyon yöneticilerine açıktır. Üyeler **API Anahtarları** sayfasını görür ancak **API Anahtarı Oluştur** düğmesini görmez. Roller için [Organizasyon ve Ekip](organizations-team.md) sayfasına bakın.

### Oluşturma

1. **Hesap → API Anahtarları** sayfasında **API Anahtarı Oluştur** düğmesine tıklayın.
2. **Ad** alanına anahtarın nerede kullanılacağını anlatan bir ad yazın (en fazla 128 karakter), örneğin "CI pipeline".
3. **Sona Erme** listesinden süreyi seçin (tablo aşağıda).
4. **Anahtar Oluştur** düğmesine tıklayın.
5. **API Anahtarı Oluşturuldu** penceresinde anahtarı kopyalayın ve güvenli bir yere kaydedin, sonra **Tamam** ile kapatın.

> **Uyarı:** Anahtar yalnızca bu pencerede bir kez gösterilir ve tekrar görüntülenemez. Kaybederseniz yeni bir anahtar oluşturup eskisini iptal edin.

| **Sona Erme** seçeneği | Anahtarın gerçek geçerlilik süresi |
|---|---|
| **Sona erme yok** (varsayılan seçim) | 90 gün. Süresiz anahtar oluşturulmaz. |
| **30 gün** | 30 gün |
| **90 gün** | 90 gün |
| **1 yıl** | 365 gün. Bu en uzun süredir. |

Süresi dolan bir anahtar uzatılamaz; yenisini oluşturun.

### Kullanma

Anahtar `kmt_<anahtar kimliği>_<gizli kısım>` biçimindedir. İsteklerde `Authorization` başlığında `Bearer` ile gönderin:

```bash
curl -H "Authorization: Bearer $KOMUTA_API_KEY" https://api.komuta.io/...
```

Anahtarı olduğu gibi kopyalayın; `kmt_` öneki küçük harfle yazılmalıdır. Geçersiz, süresi dolmuş veya iptal edilmiş bir anahtarla yapılan istek `401` ile reddedilir.

### Liste

| Sütun | Ne gösterir |
|---|---|
| **Ad** | Anahtarın adı ve altında `kmt_<anahtar kimliği>…` başlangıcı. Gizli kısım hiçbir zaman gösterilmez. |
| **Durum** | **Aktif**, **Süresi Doldu** veya **İptal Edildi**. |
| **Kullanım** | Anahtarın son kullanıldığı zaman; hiç kullanılmadıysa **Hiç kullanılmadı**. En fazla 5 dakikada bir güncellenir. |
| **Oluşturulma** | Oluşturulma zamanı. |
| **Sona Erme** | Geçerliliğin bittiği zaman. |

Üstteki arama alanı, ada ve anahtar kimliğine göre filtreler.

### İptal etme

1. Aktif anahtarın satırındaki menüden **İptal Et** seçeneğine tıklayın.
2. **API Anahtarını İptal Et** penceresinde **Anahtarı İptal Et** ile onaylayın.

Anahtar hemen çalışmaz hâle gelir ve onu kullanan her araç erişimini kaybeder. İptal geri alınamaz. İptal edilen ve süresi dolan anahtarlar listede durumlarıyla kalır.

Bir anahtar şu durumlarda da çalışmayı bırakır: süresi dolduğunda, kullanıcınız devre dışı bırakıldığında veya organizasyondan çıkarıldığınızda.

---

## İyi Uygulamalar

- Komuta için başka hiçbir yerde kullanmadığınız uzun bir parola seçin ve bir parola yöneticisinde saklayın.
- İki adımlı doğrulamayı açın; kurtarma kodlarını telefonunuzdan ayrı bir yerde saklayın.
- Google veya GitHub ile giriş yapıyorsanız o hesaplarda da iki adımlı doğrulamayı açın.
- **Güvenlik Kayıtları** ve **Aktif Oturumlar** listelerine düzenli bakın; tanımadığınız oturumları sonlandırın.
- Paylaşılan veya ortak bilgisayarlarda **Bu tarayıcıyı hatırla** kutusunu işaretlemeyin ve işiniz bitince çıkış yapın.
- Her araç veya pipeline için ayrı ve ne için kullanıldığını anlatan adla bir API anahtarı oluşturun; işi gören en kısa süreyi seçin.
- API anahtarlarını kodun içine veya repoya yazmayın; CI'ın gizli değişkenlerinde ya da bir sır yöneticisinde tutun.
- Kullanılmayan, sızdığından şüphelendiğiniz veya sahibi ekipten ayrılan anahtarları hemen iptal edin.
- Bir servisi CI'dan deploy etmek için API anahtarı yerine [dağıtım token'ı](ci-deploy.md) kullanın.

---

## İlgili Dokümanlar

- [Organizasyon ve Ekip](organizations-team.md) — Üyeler, roller ve davetler.
- [Güvenlik Merkezi](security-center-guide.md) — Organizasyonun giriş etkinliği ve denetim kayıtları.
- [CI'dan Deploy Tetikleme](ci-deploy.md) — Tek servise bağlı dağıtım token'ları.
- [Komuta MCP Sunucusunun Kurulumu](mcp-setup.md) — Claude ve Codex'i Komuta'ya bağlama.
- [Destek](support-tickets.md) — Komuta destek ekibine talep açma.
- [Hızlı Başlangıç](quick-start.md) — Hesap açma ve ilk adımlar.
