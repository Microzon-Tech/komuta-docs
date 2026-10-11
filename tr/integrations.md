# Entegrasyonlar: Git Hesapları ve Konteyner Kayıt Defterleri

Komuta servislerinizi Git reponuzdaki koddan build eder ve oluşan imajı bir konteyner kayıt defterine (registry) gönderir. Bunun için iki bağlantı gerekir: kodun okunacağı bir **Git hesabı** ve imajın gönderileceği bir **konteyner kayıt defteri**. İkisi de [Entegrasyonlar](https://console.komuta.io/integrations) sayfasından yönetilir.

İlk servisinizden önce GitHub, GitLab ya da Azure DevOps hesabınızı bağlamanız gerekir. Kayıt defteri için bir şey yapmanız gerekmez: Komuta'nın kendi kayıt defteri hazır gelir. Kendi kayıt defterinizi yalnızca imajlarınızı orada tutmak istiyorsanız eklersiniz.

---

## Entegrasyonlar Sayfası

| Bölüm | Ne gösterir |
|---|---|
| **Git Sağlayıcıları** | GitHub, GitLab ve Azure DevOps kartları. Her kartta **Bağlan** düğmesi ve o sağlayıcıdaki bağlı hesaplar bulunur. |
| **Konteyner Registry'leri** | Kullanabildiğiniz kayıt defterleri ve yenisini eklemek için **Ekle** düğmesi. |
| **GitHub App Kurulumları** | Komuta App'in kurulu olduğu GitHub hesapları ve organizasyonları. |
| **GitLab otomatik dağıtım** / **Azure DevOps otomatik dağıtım** | Push ile yeniden dağıtılan projeler ve repolar. Yalnızca o sağlayıcıda bağlı bir hesabınız varsa görünür. |

Bağlı her hesap, sağlayıcı kartında bir etiket olarak listelenir. Etiketteki **Kuruluş** veya **Kullanıcı** rozeti hesabın organizasyon mu kişisel hesap mı olduğunu gösterir. Sayfa başlığındaki rozetler toplam bağlantı, Git hesabı ve registry sayısını özetler.

GitHub ve Azure DevOps bağlantılarında etikete tıklayınca **Depolar** paneli açılır ve bağlantının erişebildiği repolar listelenir. **Özel** / **Herkese Açık** rozeti reponun görünürlüğünü gösterir; Azure DevOps bu bilgiyi vermediği için Azure DevOps repolarında rozet yoktur.

> **Not:** Organizasyon yöneticileri tüm bağlantıları görür. Diğer üyeler kendi ekledikleri ve kendilerine erişim verilmiş bağlantıları görür. Komuta'nın kayıt defteri herkese görünür.

---

## Sağlayıcılara Kısa Bakış

| | GitHub | GitLab | Azure DevOps |
|---|---|---|---|
| Nasıl bağlanır | Komuta App'i GitHub hesabınıza veya organizasyonunuza kurarsınız | gitlab.com: GitLab ile giriş. Kendi sunucunuz: erişim token'ı | Kişisel erişim token'ı (PAT) |
| İstenen yetki | Kurulumda seçtiğiniz repolar | `api` izni | **Code (Read)** ve **Service Connections (Read, query, manage)** |
| Listelenen repolar | Kurulumun erişebildiği repolar | Token sahibinin üyesi olduğu projeler | Organizasyonda token'ın görebildiği repolar |
| Push ile otomatik dağıtım | Kurulumun kapsadığı her repo için hazır | Proje başına bir webhook; servis dağıtılırken kurulur | Repo başına bir abonelik; servis dağıtılırken kurulur |
| Token yenileme | Gerekmez | **Tokenı güncelle** | **Token'ı güncelle** |

---

## GitHub

GitHub bağlantısı **Komuta App** üzerinden kurulur. Uygulamayı kişisel GitHub hesabınıza ya da bir GitHub organizasyonuna kurarsınız; Komuta yalnızca kurulumda izin verdiğiniz repolara erişir. Bu repolara yapılan push'lar ek ayar gerekmeden Komuta'ya ulaşır.

### GitHub Hesabı Bağlama

1. GitHub kartındaki **Bağlan** düğmesine ya da **GitHub App Kurulumları** bölümündeki **GitHub Hesabı Bağla** düğmesine tıklayın. GitHub'a yönlendirilirsiniz.
2. Uygulamanın kurulacağı hesabı veya organizasyonu seçin.
3. Erişimi belirleyin: tüm repolar ya da yalnızca seçtiğiniz repolar.
4. Kurulumu onaylayın. GitHub sizden Komuta'yı yetkilendirmenizi isteyebilir; bu adım kurulumu yapan kişinin hesabın sahibi olduğunu doğrular.
5. Entegrasyonlar sayfasına dönersiniz. Kurulum **GitHub App Kurulumları** bölümünde, bağlantı da GitHub kartında `github-app-<hesap>` adıyla görünür.

Kişisel bir hesapta yetkilendiren kişi hesabın kendisi olmalıdır. Bir organizasyonda yetkilendiren kişi GitHub organizasyonunun sahibi (owner) ya da uygulamayı kuran kişi olmalıdır. Doğrulama başarısız olursa "Bağlantı tamamlanamadı. Lütfen tekrar deneyin." mesajı çıkar ve bağlantı oluşmaz.

> **Not:** Bir GitHub hesabındaki kurulum tek bir Komuta organizasyonuna bağlanır. Aynı GitHub hesabı başka bir Komuta organizasyonunda zaten bağlıysa ikinci organizasyonda yeni bağlantı oluşmaz.

### Kurulum Kartı

Her kurulum kartında hesap adı, **Organizasyon** veya **Kişisel** rozeti, durum ve erişim kapsamı (**Tüm repolar** / **Seçili repolar**) görünür.

| Durum | Anlamı | Ne yapmalı |
|---|---|---|
| **Aktif** | Kurulum çalışıyor. | — |
| **Askıda** | Kurulum GitHub'da askıya alınmış. | GitHub'da kurulumu yeniden etkinleştirin. Bağlantı aynı kalır. |
| **GitHub'da kaldırıldı** | Uygulama GitHub'dan kaldırılmış; Komuta repolara erişemez. | Aşağıdaki "Kaldırıp Yeniden Kurmak" bölümünü okuyun. |

**GitHub'da Yapılandır** düğmesi kurulumun GitHub'daki ayar sayfasını açar. Repo eklemek veya çıkarmak, izinleri incelemek ve uygulamayı kaldırmak orada yapılır.

### Sonradan Repo Eklemek

Kurulumu **Seçili repolar** ile yaptıysanız yeni bir repoyu şöyle eklersiniz:

1. Kurulum kartında **GitHub'da Yapılandır**'a tıklayın.
2. GitHub'da repo erişimine yeni repoyu ekleyip kaydedin.
3. Yeni Uygulama sihirbazını yeniden açın. Repo listesi GitHub'dan okunduğu için yeni repo listede görünür.

Komuta'da başka bir işlem gerekmez; kurulum ve bağlantı aynı kalır.

### Kaldırıp Yeniden Kurmak

Kurulumu kaldırıp yeniden kurmak (Komuta'da kaldırmak ya da uygulamayı GitHub'dan silmek) repo eklemekle aynı şey değildir. Yeniden kurulumda Komuta **yeni bir bağlantı** oluşturur.

> **Uyarı:** Önceki kurulumla oluşturduğunuz servisler eski bağlantıda kalır ve yeni bağlantıya kendiliğinden geçmez. Bu servislerin sonraki build'leri kodu çekemez ve otomatik dağıtımları çalışmaz. Konsolda bir servisi başka bir Git bağlantısına taşıyan bir ekran yoktur. Repo eklemek veya çıkarmak için kurulumu kaldırmayın, **GitHub'da Yapılandır**'ı kullanın. Bu durumla karşılaştıysanız [destek talebi](support-tickets.md) açın.

Kart **GitHub'da kaldırıldı** gösteriyorsa **Yeniden bağla** kurulum akışını baştan başlatır. Bu da yeni bir bağlantı oluşturur; yukarıdaki uyarı burada da geçerlidir.

---

## GitLab

GitLab kartındaki **Bağlan** düğmesi **GitLab Bağla** penceresini açar. Önce GitLab'ınızın nerede çalıştığını seçersiniz:

- **gitlab.com** — GitLab ile giriş yap. Bu yol yalnızca gitlab.com içindir.
- **Kendi sunucum** — Erişim tokenı ile bağlan. Kendi sunucunuzda çalışan bir GitLab için tek yol budur; gitlab.com için de kullanılabilir.

### gitlab.com ile Giriş

1. **GitLab Bağla** penceresinde **gitlab.com** seçili olsun ve **GitLab'a devam et**'e tıklayın.
2. GitLab'da oturum açın ve Komuta'ya yetki verin. Komuta `api` iznini ister; bu izinle projelerinizi listeler ve otomatik dağıtımı çalıştıran push webhook'unu kurar.
3. Entegrasyonlar sayfasına dönünce "GitLab bağlandı" mesajı görünür.

GitLab giriş ekranı her seferinde açılır; doğru GitLab hesabıyla giriş yaptığınızdan emin olun. Aynı GitLab hesabıyla yeniden bağlanırsanız yeni bağlantı açılmaz, mevcut bağlantının erişimi yenilenir.

### Kendi Sunucunuzdaki GitLab

1. GitLab'da `api` iznine sahip bir token oluşturun. Personal, Group veya Project Access Token kullanılabilir.
2. **GitLab Bağla** penceresinde **Kendi sunucum**'u seçin.
3. Alanları doldurup **Bağlan**'a tıklayın.

| Alan | Açıklama |
|---|---|
| **Sunucu adresi** | GitLab sunucunuzun adresi, örneğin `https://gitlab.example.com`. Bir proje adresi yapıştırırsanız yalnızca sunucu kısmı kullanılır. Boş bırakırsanız gitlab.com kullanılır. |
| **Erişim Tokenı** | `api` izinli token. Alanın altındaki **Oluştur** bağlantısı GitLab'ın token sayfasını açar. |
| **Bağlantı Adı** | İsteğe bağlı. Boş bırakırsanız `gitlab-<sunucu>-<kullanıcı>` adı verilir. |

Komuta token'ı kaydetmeden önce GitLab'a bağlanarak doğrular ve kullanıcı adınızı oradan okur; bu yüzden sunucunuz Komuta'dan erişilebilir olmalıdır. Token reddedilirse "GitLab bu tokenı kabul etmedi. Sunucu adresini kontrol edin ve tokenın "api" iznine sahip olduğundan emin olun." mesajı çıkar ve hiçbir şey kaydedilmez.

Personal Access Token kullandıysanız Komuta son kullanma tarihini GitLab'dan okur. Tarihe 14 gün veya daha az kaldığında **GitLab otomatik dağıtım** kartında kalan günü gösteren bir rozet (örneğin **5g kaldı**), tarih geçince **Token süresi doldu** rozeti görünür. Group ve Project Access Token'larda tarih okunamaz.

### Token'ı Güncellemek

**GitLab otomatik dağıtım** kartında bağlantının yanındaki **Tokenı güncelle**'ye tıklayın, yeni token'ı girip **Tokenı güncelle** ile kaydedin. Yeni token önce doğrulanır; reddedilirse "GitLab bu tokenı kabul etmedi. Mevcut bağlantı değiştirilmedi." mesajı çıkar. Bağlantı aynı kaldığı için projeler, webhook'lar ve dağıtılmış servisler bağlı kalır.

### GitLab'da Otomatik Dağıtım

GitLab push olaylarını proje başına bir webhook ile iletir. Bir GitLab reposundan servis dağıttığınızda Komuta o projeye webhook'u kendisi kurar; webhook yalnızca push olaylarını dinler. Webhook kurulamazsa servis yine oluşturulur, ancak otomatik dağıtımı kapalı başlar.

**GitLab otomatik dağıtım** kartı her bağlantı için şunları gösterir:

- Webhook'u kurulu projeler ve sayısı (örneğin **2 repo**).
- Yenileme simgesi (Bu webhook'u GitLab'da yeniden kontrol et): webhook GitLab tarafında silindiyse yeniden kurar. Başarılı olursa "Webhook doğrulandı" görünür.
- Çöp kutusu simgesi: onay penceresinde **Otomatik dağıtımı kapat**'a tıklarsanız webhook GitLab'dan da silinir ve push'lar artık dağıtım tetiklemez.
- **Proje ekle…** listesi ve **Ekle** düğmesi: henüz webhook'u olmayan bir projeye webhook kurar.

Webhook kurmak için token'ın `api` iznine ve projede Maintainer yetkisine sahip olması gerekir. Aksi halde "Otomatik dağıtım açılamadı. Tokenın api iznine ve projede Maintainer yetkisine sahip olması gerekir." mesajı çıkar. Webhook'u sonradan eklediyseniz servisin **Otomatik dağıtım** sekmesinde otomatik dağıtımın açık olduğunu kontrol edin.

---

## Azure DevOps

Azure DevOps kişisel erişim token'ı (PAT) ile bağlanır.

### Azure DevOps Bağlama

1. Azure DevOps'ta **Code (Read)** ve **Service Connections (Read, query, manage)** yetkilerine sahip bir PAT oluşturun.
2. Azure DevOps kartındaki **Bağlan**'a tıklayın.
3. **Azure DevOps'a Bağlan** penceresindeki alanları doldurup **Bağlan**'a tıklayın.

| Alan | Açıklama |
|---|---|
| **Organizasyon** | Organizasyon adı (örneğin `contoso`) ya da tam Azure DevOps adresi. `https://dev.azure.com/contoso/...` veya `https://contoso.visualstudio.com` yapıştırırsanız ad oradan alınır; alanın altında `https://dev.azure.com/<organizasyon>` önizlemesi görünür. |
| **Kişisel Erişim Token'ı** | Yukarıdaki yetkilere sahip PAT. Organizasyonu girince çıkan **Oluştur** bağlantısı Azure DevOps'un token sayfasını açar. |
| **Bağlantı Adı** | İsteğe bağlı. Boş bırakırsanız `azuredevops-<organizasyon>` adı verilir. |
| **Kullanıcı Adı** | İsteğe bağlı. Azure DevOps yalnızca token'a bakar. |
| **Token son kullanma (opsiyonel)** | PAT'e Azure DevOps'ta verdiğiniz son kullanma tarihi. Azure bu tarihi bildirmediği için Komuta ancak siz girerseniz uyarı gösterebilir. Geçmiş bir tarih seçilemez. |

Düğme önce **Bağlanıyor...**, ardından **Doğrulanıyor...** gösterir; Komuta token'la organizasyondaki projeleri okumayı dener. Başarılı olursa "contoso bağlandı — 3 proje bulundu" biçiminde bir mesaj görünür. Token reddedilirse bağlantı geri alınır ve "Azure DevOps bu token'ı reddetti. Organizasyon adını kontrol edin ve PAT'ın Code (Read) ile Service Connections (Read, query, manage) yetkilerine sahip olduğundan emin olun." mesajı çıkar.

### Token Süresi ve Güncelleme

Son kullanma tarihini girdiyseniz tarihe 14 gün veya daha az kaldığında bağlantı etiketinin yanında kalan günü gösteren bir rozet (örneğin **5g kaldı**), tarih geçince **Token süresi doldu** rozeti çıkar. Süresi dolmuş token'la build'ler kodu çekerken başarısız olur.

Token'ı yenilemek için **Azure DevOps otomatik dağıtım** kartında bağlantının yanındaki **Token'ı güncelle**'ye tıklayın. Yeni PAT'i ve isterseniz yeni son kullanma tarihini girip **Token'ı güncelle** ile kaydedin. Yeni token önce doğrulanır; reddedilirse "Azure DevOps bu token'ı reddetti. Mevcut bağlantı değiştirilmeden bırakıldı." mesajı çıkar.

> **Uyarı:** Token'ı yenilemek için Azure DevOps'u **Bağlan** ile yeniden bağlamayın. Her bağlanma yeni bir bağlantı oluşturur; mevcut servisleriniz ve otomatik dağıtım abonelikleri eski bağlantıda kalır. **Token'ı güncelle** bağlantıyı korur.

### Azure DevOps'ta Otomatik Dağıtım

Azure DevOps push olaylarını repo başına bir abonelikle (service hook) iletir. Yeni Uygulama sihirbazıyla bir Azure DevOps reposundan servis dağıttığınızda Komuta bu aboneliği kendisi oluşturur. Oluşturamazsa servis yine dağıtılır ve şu uyarı çıkar: "Servis dağıtıldı ancak push'ta otomatik yeniden dağıtım etkinleştirilemedi. PAT'ın Service Connections (Read, query, manage) yetkisine sahip olduğunu kontrol edip Entegrasyonlar'dan tekrar deneyin."

Aboneliği **Azure DevOps otomatik dağıtım** kartından yönetirsiniz:

- Otomatik dağıtımı açmak için **Repo ekle…** listesinden repoyu seçip **Etkinleştir**'e tıklayın.
- Kapatmak için repo satırındaki çöp kutusu simgesine tıklayıp **Kaldır** ile onaylayın. Abonelik Azure DevOps'tan da silinir; mevcut servisler çalışmaya devam eder.

Azure DevOps push olayları değişen dosyaların listesini içermez. Bu yüzden Azure DevOps servislerinde yol filtreleri kullanılamaz ve izlenen branch'e gelen her push dağıtım tetikler; bekleme süresi ve atlama direktifleri çalışmaya devam eder. Ayrıntılar için [Otomatik Dağıtım](service-auto-deploy.md).

---

## Bağlantıları Sihirbazda Kullanmak

[**Uygulamalar**](https://console.komuta.io/services) sayfasında **Yeni Uygulama** ile açılan sihirbazın **Bağlan** adımı bu bağlantıları kullanır: **Git hesabı** kodun okunacağı bağlantıyı, **Konteyner kayıt defteri** build edilen imajın gönderileceği yeri belirler; ardından repo ve branch seçilir.

- Tam olarak bir Git bağlantınız ve bir kayıt defteriniz varsa ikisi kendiliğinden seçilir, kartlar gösterilmez ve doğrudan repo listesine geçersiniz.
- Birden fazla varsa kullanacağınızı seçersiniz. Kayıt defteri seçiliyse bölümü kapalı gelir ve seçili olanın adını gösterir; değiştirmek için bölümü açın.
- "Depolar, bağlı git hesaplarınızdan çekilir." satırındaki **Entegrasyonları aç** bağlantısı ve listenin sonundaki **Başka bağla** kartı Entegrasyonlar sayfasını yeni sekmede açar. Hiç Git bağlantınız yoksa adım "Kaynağınızı çekmek için bir git bağlantısına ihtiyacımız var." der. Yeni bağlantıyı ekledikten sonra sihirbaza dönün; listede görünmüyorsa sayfayı yenileyin.

Repo listesinde arama kutusuyla repo adına ve açıklamaya göre arayabilirsiniz. Bağlantının erişebildiği her repo, özel olsa da listelenir; özel repolar için ayrı bir ayar gerekmez. Herkese açık bir repo da yalnızca bağlı bir hesap üzerinden seçilebilir. Sihirbaz Git hesabı seçilmeden ilerlemez.

Repoyu seçince altında branch seçici açılır. Branch kendiliğinden seçilmez; **Dal seçin…** ile seçmeniz gerekir. Reponun varsayılan branch'i **varsayılan** etiketiyle en üstte durur, korumalı branch'ler kalkan simgesiyle işaretlidir. Branch'ler yüklenemezse durum koduyla birlikte **Başarısız** ve **Tekrar Dene** görünür.

| Mesaj | Ne yapmalı |
|---|---|
| Bir git hesabı seçin ya da yeni bir bağlantı oluşturun. | Bir Git hesabı seçin veya Entegrasyonlar'dan bağlayın. |
| Oluşturulan imaj için bir konteyner kayıt defteri seçin. | Kayıt defteri bölümünü açıp birini seçin. |
| Dağıtım için bir depo seçin. | Listeden bir repo seçin. |
| Bir dal seçin. | Branch seçiciden bir branch seçin. |
| Bu hesap için depo döndürülmedi. | Bağlantının repolara erişimi olduğundan emin olun; GitHub'da repoyu kuruluma ekleyin. |

---

## Ücretsiz Plan ve Kişisel Git Hesabı

Ücretsiz plan, organizasyonun kişisel bir GitHub, GitLab veya Azure DevOps hesabına bağlı olmasını ister. Komuta bu bağı önce kendisi kurmayı dener: ücretsiz servisin reposunun sahibi olan Git hesabını kullanır. Her Git hesabı ücretsiz planı yalnızca tek bir organizasyonda kullanabilir.

Bağ kendiliğinden kurulamadıysa hesap ayarlarınızdaki [Dış Girişler](https://console.komuta.io/account/external-logins) sayfasında **Ücretsiz plan Git hesabı** kartını kullanın:

- GitHub için **GitHub'ı bağla**'ya tıklayın. Onayı GitHub'da verirsiniz ve hesap aynı zamanda giriş yöntemi olarak eklenir.
- GitLab veya Azure DevOps için kişisel hesabınızı önce Entegrasyonlar sayfasından bağlayın, sonra kartın **Git hesaplarınız** listesinde **Bu hesapla bağla**'ya tıklayın. Bağlantıyı ekleyen kişi siz olmalısınız.

Yalnızca kişisel Git hesapları geçerlidir. Organizasyon hesapları, GitHub App kurulumları ve kendi sunucunuzda çalışan Git sunucuları ücretsiz plan için kullanılamaz. Ücretsiz servislerin kuralları için [Uyku Modu](service-sleep.md) ve [Faturalandırma](billing.md) sayfalarına bakın.

---

## Bir Git Bağlantısını Kaldırmak

**GitLab ve Azure DevOps:** Sağlayıcı kartında bağlantı etiketinin yanındaki çöp kutusu simgesine (**Bağlantıyı kaldır**) tıklayın ve **Kaldır** ile onaylayın. Komuta bağlantıya ait webhook'ları ve abonelikleri de siler.

**GitHub:** GitHub bağlantısı kurulumla birlikte yönetilir. **GitHub App Kurulumları** bölümünde kurulum kartındaki çöp kutusu simgesine tıklayıp **Kaldır** ile onaylayın. Bu işlem kurulumu ve bağlantıyı Komuta'dan kaldırır; uygulama GitHub'da kurulu kalır. GitHub'dan da kaldırmak için **GitHub'da Yapılandır**'ı kullanın.

Kaldırma geri alınamaz. Bu bağlantının repolarından oluşturulan servisler çalışmaya devam eder, ancak artık kaynak kodu çekemez ve bu bağlantıdaki otomatik dağıtım abonelikleri silinir.

> **Uyarı:** Komuta, servislerin hâlâ kullandığı bir bağlantıyı kaldırmanızı engellemez ve servisleri başka bir bağlantıya taşıyan bir ekran yoktur. Kaldırmadan önce bağlantıyı kullanan servis olmadığından emin olun. Token yenilemek için kaldırıp yeniden bağlamak yerine **Tokenı güncelle** (GitLab) veya **Token'ı güncelle** (Azure DevOps) kullanın.

---

## Konteyner Kayıt Defterleri

### Komuta'nın Kayıt Defteri

Komuta'nın kayıt defteri her organizasyonda hazırdır ve ayar gerektirmez. Listede **Platform tarafından yönetiliyor** rozetiyle işaretlidir. Bu kayıt defterini düzenleyemez ve kaldıramazsınız. Başka kayıt defteri eklemediyseniz sihirbaz bunu kendiliğinden seçer.

### Kendi Kayıt Defterinizi Eklemek

1. **Konteyner Registry'leri** bölümünde **Ekle**'ye tıklayın.
2. **Konteyner Registry'si** alanından türü seçin.
3. **Bağlantı Adı** girin (en az 2 karakter). Bu ad sihirbazda görünür.
4. Türe göre çıkan alanları doldurun ve **Registry Ekle**'ye tıklayın.

| Tür | İstenen alanlar |
|---|---|
| Docker Hub | **Registry URL** (`https://docker.io` ile dolu gelir), **Kullanıcı Adı**, **Şifre / Token** |
| GitHub Container Registry | **Registry URL** (`https://ghcr.io`), **Kullanıcı Adı**, **Şifre / Token** |
| GitLab Container Registry | **Registry URL** (`https://registry.gitlab.com`), **Kullanıcı Adı**, **Şifre / Token** |
| Azure Container Registry | **Registry URL** (`https://<your-registry>.azurecr.io` gelir; kendi kayıt defterinizin adıyla değiştirin), **Kullanıcı Adı**, **Şifre / Token** |
| AWS ECR | **ECR Registry URL** (örneğin `123456789.dkr.ecr.us-east-1.amazonaws.com`) |
| Google Container Registry | **Yapılandırma Modu**: **Keyfile JSON** (keyfile.json içeriği) ya da **Docker Config** (**Docker Config JSON** içeriği) |
| Custom Registry | **Registry URL** (`https://your.custom.registry` gelir; kendi adresinizle değiştirin), **Kullanıcı Adı**, **Şifre / Token** |

**Şifre / Token** değeri şifreli saklanır. Organizasyonunuzda aynı adda bir bağlantı varsa kayıt reddedilir ve `A service connection with the name '<ad>' already exists.` mesajı çıkar. Komuta'nın kayıt defterinin adı da kullanılamaz.

### Bağlantıyı Doğrulama

Konsolda kayıt defteri için bağlantı testi yoktur. Komuta bilgileri kaydederken kayıt defterine bağlanıp denemez; bilgiler build imajı gönderirken kullanılır. Kullanıcı adının, token'ın ve adresin doğru olduğunu kayıt defterinizin kendi panelinden kontrol edin. Bilgiler hatalıysa sorun ilk build'de ortaya çıkar; build adımları için [Pipeline'lar](service-pipeline-guide.md).

### Düzenlemek ve Kaldırmak

Kendi eklediğiniz kayıt defterlerinin kartında kalem ve çöp kutusu simgeleri bulunur.

**Düzenlemek:** **Konteyner Registry'sini Düzenle** penceresi açılır. **Şifre / Token** alanını boş bırakırsanız kayıtlı şifre korunur. **Registry URL** alanı ise boş gelir ve boş bırakıp kaydederseniz kayıtlı adres silinir; kaydetmeden önce adresi yeniden girin. **ECR Registry URL** ve Google Container Registry alanları için de aynısı geçerlidir. Başarılı olursa "Registry güncellendi" görünür.

**Kaldırmak:** **Registry'yi Kaldır** penceresinde **Kaldır** ile onaylayın. İşlem geri alınamaz. Komuta, servislerin kullandığı bir kayıt defterini kaldırmanızı engellemez; kaldırmadan önce onu kullanan servis olmadığından emin olun.

### Sihirbazda Kayıt Defteri Seçimi

Yeni Uygulama sihirbazının **Bağlan** adımındaki **Konteyner kayıt defteri** bölümü kullanabildiğiniz tüm kayıt defterlerini listeler; Komuta'nın kayıt defteri burada da **Platform tarafından yönetiliyor** rozetiyle görünür. Seçtiğiniz kayıt defteri, servisin build edilen imajının gönderileceği yerdir.

---

## Hazır Bir İmajı Çalıştırmak

**Servisler:** Yeni Uygulama sihirbazı her zaman bir repo ve branch ister; servis repodan build edilir. Sihirbazda hazır imaj seçeneği yoktur. Var olan bir servise kendi CI'nızda build ettiğiniz imajı dağıtmak için servisin **Otomatik dağıtım** sayfasındaki **CI/CD entegrasyonu** sekmesinde **Hazır imaj dağıt** iznine sahip bir dağıtım token'ı kullanın; ayrıntılar [CI'dan Deploy Tetikleme](ci-deploy.md) sayfasında.

**Job ve CronJob:** Hazır bir imajı build etmeden çalıştırabilirsiniz.

1. [Job ve CronJob'lar](https://console.komuta.io/jobs) sayfasında yeni bir iş başlatın.
2. Kaynak olarak **Hazır imajın**'ı seçin.
3. **Kayıt defteri** adımında imajı çekmek için kullanılacak kayıt defteri bağlantısını seçin.
4. **İmaj referansı** alanına imajın tam adını girin, örneğin `ghcr.io/acme/temizlik:1.4`. Seçilen kayıt defterinden değişmez bir etiket ya da digest kullanın.

Ayrıntılar için [Job ve CronJob'lar](jobs-cronjobs.md).

---

## Sık Karşılaşılan Mesajlar

| Mesaj | Nerede | Ne yapmalı |
|---|---|---|
| Bağlantı tamamlanamadı. Lütfen tekrar deneyin. | GitHub veya GitLab'dan dönüşte | Tekrar deneyin. GitHub'da yetkilendirmeyi hesabın sahibi olarak yaptığınızdan emin olun. |
| Repolar yüklenemedi | **Depolar** paneli | Bağlantı kaldırılmış, askıya alınmış ya da token'ı geçersiz olabilir. Kurulum durumuna bakın veya token'ı güncelleyin. |
| Repo bulunamadı | **Depolar** paneli | Bağlantının erişebildiği repo yok. GitHub'da **GitHub'da Yapılandır** ile repo ekleyin. |
| GitHub App bağlantısı kesildi | Servisin **Otomatik dağıtım** sekmesi | Kurulum GitHub'da kaldırılmış veya askıya alınmış. **GitHub App Kurulumları** bölümünde durumu kontrol edin. |
| Otomatik dağıtım etkinleştirilemedi. PAT'ın Service Connections (Read, query, manage) yetkisi olmalı. | **Azure DevOps otomatik dağıtım** | PAT'e bu yetkiyi verip **Token'ı güncelle** ile yenileyin, sonra tekrar deneyin. |
| Token süresi doldu | GitLab veya Azure DevOps bağlantısının yanında | **Tokenı güncelle** / **Token'ı güncelle** ile yeni token girin. |

---

## İlgili Dokümanlar

- [Hızlı Başlangıç](quick-start.md) — İlk servisi yayınlamadan önce hazırlanacaklar.
- [Yeni Servis Oluşturma](new-service-guide.md) — Sihirbazın tüm adımları.
- [Otomatik Dağıtım](service-auto-deploy.md) — Push ile dağıtımın kuralları.
- [Pipeline'lar](service-pipeline-guide.md) — Build ve yayın süreci.
- [Job ve CronJob'lar](jobs-cronjobs.md) — Hazır imajla iş çalıştırma.
- [Destek](support-tickets.md) — Destek ekibine ulaşma.
