# Giriş ve Paylaşım

**Komuta girişi** açıkken servisinizi açmak isteyen herkes önce Komuta hesabıyla giriş yapar; yalnızca servisi paylaştığınız kişiler içeri girer. Kimlerle paylaştığınızı **Erişim ve portlar** sayfasının **Kişiler** sekmesinden yönetirsiniz.

Bu sayfa iki tarafı anlatır: servis sahibinin paylaşımları, paylaşım bağlantılarını ve ziyaretçi oturumlarını nasıl yönettiği ve ziyaretçinin giriş sırasında ne gördüğü.

---

## Ziyaretçi nasıl giriş yapar

1. Ziyaretçi servisinizin adresini açar, örneğin `https://panel-3a2331e6.edge-1.komuta.app/raporlar`.
2. Komuta ziyaretçinin bu servis için oturumu olmadığını görür ve onu Komuta konsolundaki giriş sayfasına (`https://console.komuta.io/access/…`) yönlendirir.
3. Ziyaretçi Komuta'ya giriş yapmamışsa **Devam etmek için giriş yapın** sayfasını görür. Sayfa servisin sahibinin erişimi Komuta ile sınırladığını söyler, **Gideceğiniz adres** kutusunda servisin adresini gösterir ve **Komuta ile giriş yap** düğmesini sunar. Hesabı olmayanlar için not: "Hesabınız yok mu? Sonraki sayfada Google ya da GitHub ile saniyeler içinde açabilirsiniz. Sayfa e-posta adresinizle paylaşıldıysa o adresi kullanın."
4. Giriş yapan ziyaretçi **Erişiminiz kontrol ediliyor** ekranını görür.
5. Hesabı bir paylaşımla eşleşiyorsa açmak istediği sayfaya geri döner (örnekte `/raporlar`). Eşleşmiyorsa **Erişiminiz yok** sayfasını görür (aşağıya bakın).

Komuta'ya zaten giriş yapmış bir ziyaretçi 3. adımı görmez; yönlendirme birkaç saniye içinde tamamlanır.

### Oturum

Başarılı bir girişten sonra ziyaretçinin tarayıcısına servisin kendi adresi için bir oturum çerezi yazılır:

- Oturum, servisin **Oturum süresi** ayarı kadar sürer: 15 dakika, 1 saat, 4 saat, **12 saat** (varsayılan), 1 gün ya da 7 gün (bkz. [Kimler içeride](#kimler-içeride)). Ziyaretçinin paylaşımının bitiş zamanını ve korumanın "herkese açılsın" bitişini hiçbir zaman geçmez.
- Oturum **yalnızca giriş yapılan adres için** geçerlidir. Servisin birden fazla adresi varsa (örneğin `*.komuta.app` adresi ve özel alan adınız ya da mavi-yeşil dağıtımın önizleme adresi) her biri için ayrı giriş gerekir.
- Komuta'nın çerezleri isteğe uygulamanıza ulaşmadan önce çıkarılır; uygulamanız bu çerezleri görmez ve etkilenmez.
- Girişin yaklaşık 10 dakika içinde tamamlanması gerekir. Ziyaretçi daha uzun beklerse konsol **Bu giriş bağlantısı geçersiz** der ya da servis kısa bir `sign-in link is invalid or expired` yanıtı verir; korunan sayfayı yeniden açması yeterlidir.

### Oturumların erken bittiği durumlar

Oturumlar süreleri dolmadan da, yaklaşık 30 saniye içinde sona erebilir. Erişimi devam edenler bir sonraki sayfa açılışında yeniden giriş yapar: Komuta'ya zaten giriş yapmış olanlar için bu otomatik bir yönlendirmedir, e-posta paylaşımıyla girenler yeni bir kod ister. Bu arada tarayıcı dışı istekler (örneğin bir tek sayfa uygulamasının `POST` istekleri) sayfa yenilenene kadar `401` alabilir.

**Yalnızca ilgili kişiler** — serviste tek tek çıkarma devreye girdikten sonra (bkz. [Tek tek çıkarma ne zaman başlar](#tek-tek-çıkarma-ne-zaman-başlar)):

- **Kişiler** sekmesinden bir kişiyi çıkardığınızda.
- Bir paylaşımı kaldırdığınızda, bitiş tarihini öne çektiğinizde ya da bitişi olmayan bir paylaşıma bitiş eklediğinizde: o paylaşımla açılmış oturumlar sona erer.
- Sayfa sınırlı bir paylaşımın sayfa listesini değiştirdiğinizde (sayfa eklemek dahil; **Tüm site**'ye geçmek hariç): o paylaşımla açılmış oturumlar sona erer.

Tek tek çıkarma devreye girene kadar bunların her biri **bu servisteki tüm ziyaretçilerin** oturumunu sonlandırır.

**Servisteki tüm ziyaretçiler:**

- **Herkesi çıkar**'ı seçtiğinizde.
- Organizasyonunuz dış paylaşımı kapattığında (dış paylaşımı olan servislerde; dış paylaşımla girenler erişimini kaybeder, diğerleri bir kez yeniden giriş yapar).
- Bir ziyaretçinin Komuta hesabında güvenlikle ilgili bir değişiklik olduğunda: hesap silindiğinde, kilitlendiğinde ya da devre dışı bırakıldığında, giriş bilgileri veya iki adımlı doğrulama değiştiğinde, e-posta adresi artık doğrulanmış olmadığında, hesap bağlantısı kaldırıldığında ya da organizasyonu askıya alındığında. O kişinin açabildiği servislerdeki tüm ziyaretçiler yeniden giriş yapar (bu bir dakikayı biraz aşabilir).

**Oturum süresi**'ni kısaltmak da yeni süreden eski oturumları hemen sonlandırır.

### Giriş yapamayanlar

Şu oturumlarla korunan bir servise giriş yapılamaz: başka bir kullanıcının yerine geçilen (impersonation) oturumlar, API anahtarıyla yapılan istekler, aktif olmayan ya da kilitli hesaplar ve aktif olmayan ya da yasaklanmış organizasyonların kullanıcıları. Bir kullanıcı dakikada en fazla 30 giriş denemesi yapabilir; fazlasında "Çok fazla giriş denemesi yapıldı. Bir dakika bekleyip tekrar deneyin." yazar.

### Tarayıcı dışı istemciler

Giriş isteyen bir sayfaya oturumsuz gelen `GET` ve `HEAD` istekleri giriş sayfasına yönlendirilir (`302`). Diğer yöntemler (örneğin bir API'ye `POST`) yönlendirilmez; `401` ve `sign-in required` gövdesini alır. Programların servisinize erişmesi için [servis token'ı](access-protection-machines.md#servis-tokenları), [webhook yolu](access-protection-machines.md#webhook-yolları) ya da IP izin listesi kullanın.

---

## Ziyaretçinin gördüğü sayfalar

| Durum | Ziyaretçinin gördüğü |
|---|---|
| Giriş yaptı ve bir paylaşımla eşleşiyor | Servis açılır. |
| Komuta'ya giriş yapmamış | **Devam etmek için giriş yapın** (Komuta konsolunda). |
| Giriş yaptı ama hiçbir paylaşımla eşleşmiyor | **Erişiminiz yok** (Komuta konsolunda). |
| Giriş yaptı ama açtığı sayfa paylaşımının dışında | **Bu sayfaya erişiminiz yok** — "Bu sayfa, sizinle paylaşılan sayfaların dışında." ve altında **Erişebileceğiniz sayfalar** listesi (en fazla 20 bağlantı). |
| Giriş yaptı ama sayfa **Yalnızca seçilen kişiler** kuralıyla korunuyor ve seçilmemiş ya da saat aralığı dışında | **Bu sayfaya erişiminiz yok** — "Bu bölüm yalnızca servis sahibinin seçtiği kişilere, belirlediği saatlerde açık." |
| IP izin listesinde olmayan bir adresten geldi | **Bu servise erişim kısıtlı** — "Bu servis yalnızca izin verilen ağlardan açılabilir." Sayfa **Adresiniz** kutusunda servisin gördüğü adresi gösterir; **Kopyala** ile alınıp servis sahibine iletilebilir. **Tekrar dene** sayfayı yeniden yükler. Bu sayfada giriş seçeneği yoktur. |
| **Tamamen engelle** kuralındaki bir yolu açtı | `403`, düz metin `access denied`. |
| Servis artık giriş istemiyor ama ziyaretçi eski bir giriş bağlantısıyla geldi | **Bu servis Komuta girişi istemiyor**. |
| Giriş şu an kontrol edilemiyor | **Giriş şu anda kullanılamıyor** ve **Tekrar dene** düğmesi. |
| Komuta'nın erişim kontrolü geçici olarak yanıt veremiyor | `503`, düz metin `access policy unavailable` ya da `sign-in unavailable`. `access policy unavailable` olduğunda servis kimseye açılmaz; `sign-in unavailable` yalnızca giriş isteyen sayfaları etkiler. Kısa süre sonra tekrar deneyin. |
| Servisin ülke listesinde olmayan bir ülkeden geldi | **Bu servise erişim kısıtlı** (IP listesiyle aynı sayfa). |
| Çok fazla istek gönderdi | `429`, düz metin `too many requests` ve `Retry-After` başlığı. |
| Yolun izin vermediği bir yöntem kullandı | `405`, düz metin `method not allowed` ve `Allow` başlığı. |
| Bir paylaşım bağlantısı açtı | Bkz. [Paylaşım bağlantıları](#paylaşım-bağlantıları). |

Servisinizin içinde gösterilen **Bu servise erişim kısıtlı** ve **Bu sayfaya erişiminiz yok** sayfaları, ziyaretçinin tarayıcı dili Türkçe ise Türkçe, değilse İngilizce görünür. `GET`/`HEAD` dışındaki istekler (tarayıcıdan ya da programdan) bu sayfalar yerine kısa düz metin alır: `access restricted to allowed networks` ya da `this path is not shared with you`.

### Erişiminiz yok sayfası

Bu sayfa ziyaretçiye neden giremediğini ve ne yapabileceğini gösterir:

- **Oturum açık** kutusu hangi hesapla ve hangi organizasyonla (**Organizasyon: …**) giriş yapıldığını gösterir.
- **E-posta adresinizle mi paylaşıldı?** bölümü e-posta paylaşımı için doğrulama yapar (aşağıya bakın).
- **Başka bir organizasyonla devam et** — ziyaretçi birden fazla organizasyonun üyesiyse buradan **Geç** ile diğerine geçip yeniden dener. Servisi "Organizasyonunuz" ya da "Bağlı bir organizasyon" ile paylaştıysanız, ziyaretçinin o organizasyonla devam etmesi gerekebilir.
- **Başka bir hesapla giriş yap** — farklı bir Komuta hesabıyla yeniden giriş.
- **Komuta'ya git** — Komuta konsoluna döner.

---

## Paylaşımlar

**Kişiler** sekmesindeki **Paylaşılanlar** listesi, Komuta ile giriş yaptıktan sonra içeri alınan kişi ve organizasyonları gösterir. **Paylaşım ekle** ile yeni paylaşım eklenir.

### Paylaşım türleri

| Tür | Kimler girer |
|---|---|
| **Organizasyonunuz** | Bu organizasyonun tüm aktif üyeleri. Serviste bir tane olabilir. Listede **Organizasyonunuzdaki herkes** olarak görünür. |
| **Bir üye** | Organizasyondaki tek bir kişi. Yalnızca aktif üyeler seçilebilir. |
| **Bağlı bir organizasyon** | Üyesi olduğunuz başka bir organizasyondaki herkes; sonradan katılanlar dahil. Yalnızca sizin de üyesi olduğunuz organizasyonlar seçilebilir. |
| **Bir e-posta adresi** | Organizasyonlarınızın dışındaki biri. Herhangi bir Komuta hesabıyla giriş yapar, ardından bu adrese gönderilen tek kullanımlık kodla adresi doğrular. |

**Bağlı bir organizasyon** ve **Bir e-posta adresi** paylaşımları listede **Dış** etiketiyle görünür ve organizasyonunuzun dış paylaşıma izin vermesini gerektirir (aşağıya bakın).

Zaten paylaşılmış bir kişi, organizasyon ya da adres yeniden eklenemez (pencere "Zaten paylaşıldı." der); bitişini ya da sayfa sınırını listedeki kalem simgesiyle değiştirin.

### Paylaşım eklerken seçilenler

- **Neleri açabilir** — **Tüm site** (varsayılan, "Girişin açtığı her sayfa.") ya da **Yalnızca şu sayfalar** ("Girişleri başka hiçbir sayfayı açmaz."). İkincisinde her satıra bir yol yazılır; bir yol altındaki sayfaları da açar (`/reports`, `/reports/2026`'yı da açar). Yol kuralları bu sayfaların içinde yine geçerlidir. En fazla 50 sayfa; `/` yazılamaz (bunun yerine **Tüm site**). Bu seçenek yalnızca koruma Komuta girişi isterken kullanılabilir; aksi halde pencere bunu belirtir.
- **Erişim bitişi** — isteğe bağlı. Boş bırakılırsa erişim siz kaldırana kadar sürer. Saatler hesabınızın saat dilimindedir (**Hesap → Genel**); cihazınızın saat dilimi farklıysa kart sizi uyarır ve seçtiğiniz saatin cihazınızda kaça denk geldiğini gösterir. Paylaşım bitiş tarihinin üst sınırı yoktur; geçmiş bir zaman seçilemez.

### Sayfa sınırlı paylaşımlar nasıl birleşir

- **En geniş paylaşım kazanır.** Bir kişi hem sayfa sınırlı hem tüm siteyi açan bir paylaşımla eşleşiyorsa tüm siteyi açar. Örneğin **Organizasyonunuz** tüm siteye açıkken bir üyeyi yalnızca `/raporlar`'a sınırlamak işe yaramaz; kart bunu uyarır.
- Paylaşımının dışındaki bir sayfayı açan kişi **Bu sayfaya erişiminiz yok** sayfasını ve açabileceği sayfaların listesini görür.
- Sayfa sınırı yalnızca bir kimlik gerektiren yerlerde uygulanır. Herkese açık bir yol ya da IP listesiyle girişsiz geçilen bir yer, sayfa sınırından bağımsız olarak açık kalır.
- Serviste sayfa sınırlı bir paylaşım varsa, her oturum kişinin eşleştiği paylaşımlardan en erken biteninin bitişinde sona erer.

### Listedeki etiketler ve düzenleme

Her satırda paylaşımın adı, varsa sayfa sınırı ("Yalnızca: /a, /b") ve bitiş ("… tarihine kadar" ya da **Bitiş yok**) yazar. Etiketler:

- **Dış** — bağlı organizasyon ya da e-posta paylaşımı.
- **Askıda** — dış paylaşım kapatıldığı için askıya alınmış paylaşım. Bu paylaşımla kimse giremez.
- **Süresi doldu** — bitiş zamanı geçmiş paylaşım. Listede kalır; kalem simgesiyle yeni bir bitiş verilebilir.

Kalem simgesi paylaşımı düzenler: tür ve kişi değiştirilemez; sayfa sınırı ve bitiş tarihi değiştirilebilir. Bitişi uzatmak, bitişi kaldırmak ya da paylaşımı **Tüm site**'ye açmak açık oturumları etkilemez. Bitiş eklemek, bitişi öne çekmek ya da sayfa listesini değiştirmek o paylaşımla açılmış oturumları sonlandırır; serviste tek tek çıkarma henüz devrede değilse bu servisteki tüm açık oturumlar sona erer ve herkes bir kez yeniden giriş yapar (bkz. [Oturumların erken bittiği durumlar](#oturumların-erken-bittiği-durumlar)).

### Paylaşımı kaldırma

Satırdaki çöp kutusu simgesiyle kaldırılır ve **Bu paylaşım kaldırılsın mı?** onayı istenir ("… yaklaşık 30 saniye içinde erişimini kaybeder; açık oturumları da kapanır. Komuta oturumları henüz birbirinden ayırt edemiyorsa giriş yapmış herkesten yeniden giriş yapması istenir."). Tek tek çıkarma devredeyse yalnızca o paylaşımla açılmış oturumlar sona erer; öncesinde servisteki diğer ziyaretçiler de bir kez yeniden giriş yapar. Paylaşım, seçildiği **Yalnızca seçilen kişiler** kurallarından da çıkarılır.

### Paylaşımlar ne zaman geçerlidir

- Paylaşım eklemek için koruma uygulanmış olmalıdır ("Önce korumayı uygulayın, ardından servisi paylaşın.").
- Paylaşımlar yalnızca bir yer Komuta girişi isterken geçerlidir: **Komuta girişi iste** açıkken ya da **Komuta girişi**, **IP listesi ve Komuta girişi** veya **Yalnızca seçilen kişiler** türünde bir yol kuralı varken.
- Giriş açık ama hiç paylaşım yoksa kimse girişten geçemez ("Henüz kimseyle paylaşılmadı. Bir paylaşım ekleyene kadar kimse girişten geçemez.").
- **Siz otomatik eklenirsiniz.** Komuta girişini zorunlu hale getiren bir kayıt yaptığınızda (korumayı yeniden açtığınızda da), paylaşımları yönetme izniniz varsa ve organizasyon paylaşımı ya da kendi adınıza bir paylaşım yoksa Komuta sizi **Bir üye** paylaşımı olarak ekler (bitişsiz, tüm site). Böylece kendi servisinizin dışında kalmazsınız. Erişiminiz olmaması gerekiyorsa bu paylaşımı kaldırabilirsiniz.
- Korumayı kapatmak paylaşımları silmez; korumayı yeniden açtığınızda aynı paylaşımlar geçerli olur.
- Bir serviste en fazla **200** paylaşım olabilir.

---

## Paylaşım bağlantıları

Paylaşım bağlantısı, **Komuta hesabı olmayan** birini bir süreliğine içeri almanızı sağlar: bir müşteri demosu, dışarıdan test eden biri. Bağlantıyı elinde tutan herkes, bağlantının süresi dolana ya da siz silene kadar giriş yapmadan girer. Bağlantılar **Kişiler** sekmesindeki **Paylaşım bağlantıları** bölümündedir.

### Bağlantı oluşturma

1. **Bağlantı oluştur**'a tıklayın.
2. Pencerede:
   - **Ad** — kime göndereceğinize göre adlandırın, örneğin "Müşteri demosu"; ad erişim kaydında görünür. 1–64 karakter; aynı servisin iki bağlantısı aynı adı taşıyamaz (büyük/küçük harf fark etmez).
   - **Geçerlilik süresi** — **1 gün**, **7 gün** (varsayılan), **30 gün** ya da **90 gün**. Bağlantının her zaman bir bitişi vardır; API ile en fazla 365 gün sonrası verilebilir.
   - **Neleri açabilir** — **Tüm site** (varsayılan) ya da **Yalnızca şu sayfalar**: her satıra bir yol, en fazla 50; her yol altındaki sayfaları da açar, yol kuralları içeride yine geçerlidir. Kopyalanan bağlantı bu yollardan alfabetik sırada ilkini açar.
3. **Bağlantı oluştur** ile kaydedin. **Bağlantınız hazır** ekranı bağlantıyı gösterir; **Kopyala** ile alın. Bağlantıyı daha sonra listeden yeniden kopyalayabilirsiniz.

Bir bağlantı `https://panel.ornek.com/raporlar?komuta_link=kl_…` biçimindedir. Yalnızca ilgili kişilere gönderin: bağlantıyı elinde tutan herkes girer.

### Ziyaretçi ne alır

- **Bağlantıyı açmak** — tarayıcı bağlantıyı açar (`GET` ya da `HEAD`). Komuta bağlantıyı kontrol eder, servisin bu adresi için bir oturum açar ve ziyaretçiyi `komuta_link` bölümü çıkarılmış aynı adrese gönderir; böylece gizli değer adres çubuğundan silinir ve başka sitelere aktarılmaz. Sorgu dizesi hiçbir zaman kaydedilmez.
- **Oturum**, servisin **Oturum süresi** kadar (varsayılan 12 saat) sürer ve bağlantının bitişini hiçbir zaman geçmez.
- **Yalnızca bağlantının sayfaları** — diğer sayfalar `403` "Your share link does not open this page." yanıtı verir. Engelli yollar engelli kalır, **Yalnızca seçilen kişiler** yolları bağlantıyla hiçbir zaman açılmaz; IP izin listesi, ülke listesi ve hız sınırı yine uygulanır.
- **Mevcut oturum korunur** — bu servise Komuta hesabıyla zaten giriş yapmış bir ziyaretçi kendi oturumunu korur; bağlantı bölümü çıkarılmış aynı adrese gönderilir.
- **Diğer yöntemler** (örneğin `POST`) bir bağlantı adresinde `403` "Open a share link in a browser." alır.
- **Yanlış ya da süresi dolmuş bir bağlantı** `403` "This share link is not valid or has expired. Ask the person who sent it for a new one." alır.
- **Bağlantıyı sildiğinizde ya da süresi dolduğunda**, yaklaşık 30 saniye içinde ziyaretçinin bir sonraki isteği `403` "The share link you opened this site with has ended. Ask the person who sent it for a new one." alır. Ziyaretçi hiçbir zaman Komuta giriş sayfasına gönderilmez.
- Bu yanıtlar İngilizce, kısa düz metinlerdir.
- Bağlantıyla girenler **Kimler içeride** listesinde görünmez. Oturumları, bağlantı silindiğinde ya da süresi dolduğunda ya da **Herkesi çıkar**'ı seçtiğinizde sona erer.
- **Giriş yapanı uygulamama bildir** açıksa uygulamanız yalnızca `x-komuta-identity` başlığını, `kind: share_link` değeriyle alır (bkz. [Bitiş ve Kimlik Bildirme](access-protection-settings.md#uygulamanızın-aldığı-başlıklar)).

> **Bağlantı önizlemeleri de açılış sayılır.** Önizleme göstermek için bağlantıyı çeken sohbet uygulamaları ve e-posta tarayıcıları bağlantıyı bir ziyaretçi gibi açar. Erişim kaydında bu "Paylaşım bağlantısıyla girdi" olarak görünür.

### Liste

Her satırda bağlantının adı, açabileceği sayfalar (**Tüm site** ya da "Yalnızca: …"), bitişi ("… tarihine kadar") ve son kullanımı ("son kullanım …" ya da "henüz kullanılmadı") yazar. Kimseyi içeri almayan bir bağlantıda **Çalışmıyor** etiketi görünür: süresi dolmuştur, koruma artık Komuta girişi istemiyordur ya da paylaşım bağlantıları platformda kapatılmıştır. Süresi dolan bağlantılar bir sonraki bağlantı oluşturulana kadar listede kalır.

Bir bağlantıyı silmek için çöp kutusu simgesine tıklayıp **Bu bağlantı silinsin mi?** onayını verin ("Yaklaşık 30 saniye içinde bağlantı çalışmayı bırakır ve {name} ile girmiş herkese bağlantının sona erdiği gösterilir.").

### Kurallar ve sınırlar

- Paylaşım bağlantıları yalnızca koruma Komuta girişi isterken çalışır ("Paylaşım bağlantıları yalnızca Komuta girişi zorunluyken çalışır.").
- Bir serviste en fazla **50** paylaşım bağlantısı olabilir.
- Bağlantı oluşturmak ve silmek için **Korunan servisin paylaşımlarını yönet** izni gerekir; listeyi görmek için erişim korumasının iki izninden biri yeterlidir.

Bölümde "Paylaşım bağlantıları bu serviste henüz kullanılamıyor." yazıyorsa paylaşım bağlantıları platformunuzda henüz açık değildir. Bağlantılar varken özellik kapatılırsa bölüm "Paylaşım bağlantıları şu an Komuta'da kapalı; bu bağlantılar kimseyi içeri almıyor. Yine de silebilirsiniz." der.

---

## Kimler içeride

**Kişiler** sekmesindeki **Kimler içeride** bölümü, bu servise Komuta hesabıyla giriş yapmış ve oturumu hâlâ açık olan kişileri, her kişi için bir satırda listeler:

- Ad (ya da e-posta). Adı bilinmeyen biri "Başka bir kuruluştan bir ziyaretçi" olarak görünür; organizasyonunuzun dışından gelenlerde **Kuruluşunuz dışından** etiketi bulunur. Kişi birden fazla tarayıcıda ya da cihazda giriş yaptıysa "{count} oturum" yazar.
- "Giriş … · son görülme … · bitiş …". **Son görülme** erişim kaydından gelir (girişten sonra kaydedilen bir sayfa açmadıysa "henüz yok").
- E-posta adresleri yalnızca kullanıcıları görme izni olanlara gösterilir (e-posta paylaşımıyla açılan oturumlarda paylaşımları yönetebilenlere de).
- En fazla **200** oturum listelenir; daha fazlası varsa bölüm "Yalnızca en son girişler gösteriliyor." der.
- Paylaşım bağlantısıyla ya da servis token'ıyla girenler listelenmez.

Koruma Komuta girişi istemiyorsa bölüm "Bu servise kimse giriş yapmıyor; içeridekileri görmek için Kurallar'da Komuta girişini açın." der.

### Ziyaretçileri çıkarma

Çıkarmak için **Korunan servisin paylaşımlarını yönet** izni gerekir.

- Bir kişinin satırındaki **Çıkar**, **{name} çıkarılsın mı?** diye sorar: "Açık oturumları yaklaşık 30 saniye içinde kapanır; başka kimse etkilenmez. Tekrar girmesini istemiyorsanız ona erişim veren paylaşımı da kaldırın." Bir paylaşım ona hâlâ erişim veriyorsa kişi hemen yeniden giriş yapabilir.
- **Herkesi çıkar**, **Herkes çıkarılsın mı?** diye sorar: paylaşım bağlantısıyla girenler dahil bu servise giriş yapmış herkesten yaklaşık 30 saniye içinde yeniden giriş yapması istenir.

### Oturum süresi

**Oturum süresi** bir girişin ne kadar süreceğini belirler: **15 dakika**, **1 saat**, **4 saat**, **12 saat** (varsayılan), **1 gün** ya da **7 gün**. Seçtiğiniz anda kaydedilir ("Yeni girişler artık {length} sürecek. Açık oturumlar bu süreyi aşacaksa daha erken kapanır.").

- Daha kısa bir süre eski oturumları da hemen sonlandırır; daha uzun bir süre zaten açık olan oturumları uzatmaz.
- E-posta paylaşımıyla ve paylaşım bağlantısıyla açılan oturumlar için de geçerlidir.
- Değiştirmek için **Servis erişim korumasını yönet** izni gerekir.

### Tek tek çıkarma ne zaman başlar

Komuta yalnızca birbirinden ayırt edebildiği oturumları tek tek kapatabilir. Oturumları ayırt etmeye, serviste oturum takibi başladığında başlar (Komuta girişi açıldığında ya da bu özellik platformunuza geldiğinde). Bundan önce açılmış oturumlar 12 saate kadar açık kalabildiği için tek tek çıkarma, takip başladıktan **12 saat 10 dakika** sonra devreye girer. O zamana kadar:

- bir kişiyi çıkarmak herkesi çıkarır ve onay penceresi **Herkes çıkarılsın mı?** olur;
- bölüm "Buradaki bazı oturumlar Komuta onları birbirinden ayırt edemeden açıldı. {time} saatine kadar bir kişiyi çıkarmak herkesi çıkarır." (ya da "Şimdilik bir kişiyi çıkarmak bu servisteki herkesi çıkarır.") der;
- bir paylaşımı kaldırmak ya da daraltmak da herkesi çıkarır.

Bir oturumun ömrü içinde çok sayıda (500'den fazla) oturum tek tek kapatıldıysa Komuta yine herkesi çıkarmaya döner.

Bu bölümü görmüyorsanız ziyaretçi oturumlarını yönetme platformunuzda henüz açık değildir. Bu durumda oturumlar 12 saat sürer ve bir paylaşımı kaldırmak ya da daraltmak herkesi çıkarır.

---

## E-posta paylaşımı

E-posta adresiyle paylaşılan kişinin, o adresle bir Komuta hesabı olması gerekmez; herhangi bir Komuta hesabıyla giriş yapıp posta kutusunu okuyabildiğini kanıtlar.

1. Kişi servisi açar ve herhangi bir Komuta hesabıyla giriş yapar (hesabı yoksa Google ya da GitHub ile açabilir).
2. Hesabı başka bir paylaşımla eşleşmiyorsa **Erişiminiz yok** sayfasını görür. Sayfadaki **E-posta adresinizle mi paylaşıldı?** bölümünde adres alanı hesabın e-postasıyla dolu gelir; paylaşılan adres farklıysa değiştirilir.
3. **Bana kod gönder** ile adrese **8 haneli**, **10 dakika** geçerli bir kod gönderilir. E-postanın konusu "{servis} için Komuta erişim kodunuz"dur ve kodu isteyen Komuta hesabını, servisi ve adresi gösterir.
4. Kişi kodu girip **Doğrula ve devam et** der; servis açılır.

Kurallar:

- Kod yalnızca isteyen Komuta hesabında çalışır ve tek kullanımlıktır. Her girişte yeni kod gerekir; e-posta paylaşımıyla açılan oturum da servisin **Oturum süresi** kadar sürer.
- Bir kod 5 hatalı denemeden sonra geçersiz olur. Aynı hesap, servis ve adres için 24 saatte 50 hatalı denemeden sonra yeni kod gönderilmez.
- Kod isteme sınırları: bir kullanıcı saatte en fazla 20 kod, aynı adres için saatte en fazla 5 kod isteyebilir. Aynı Komuta hesabıyla, aynı servis için bir adrese saatte en fazla 3 kod gönderilir; tarayıcı ya da cihaz değiştirmek bunu sıfırlamaz ve sınırdan sonra ekran yine "kod gönderdik" dese de e-posta gitmez. Yeniden göndermeden önce 30, 60 ve 120 saniye beklenir.
- Ekran, adresin erişimi olsa da olmasa da aynı cevabı verir ("{adres} adresinin bu sayfaya erişimi varsa, adrese {uzunluk} haneli bir kod gönderdik."); böylece hangi adreslerle paylaşım yapıldığı tahmin edilemez.
- Giriş bağlantısının süresi kodun süresinden önce doluyorsa ekran "Bu giriş bağlantısının süresi, kodun süresinden önce doluyor." der; korunan sayfayı yeniden açıp hemen kod istemek gerekir. Kodu istedikten sonra beklemeden girin: giriş, korunan sayfadan yönlendirildiğiniz andan itibaren yaklaşık 10 dakika içinde tamamlanmalıdır.
- E-posta adresi düz bir adres olmalıdır: ASCII harfler, en fazla 254 karakter; joker karakter, boşluk, IP adresi ve İngilizce dışı harf kabul edilmez.

---

## Dış paylaşım izni

**Bağlı bir organizasyon** ve **Bir e-posta adresi** paylaşımları organizasyon dışına erişim verir. Bunlar için organizasyonunuzun dış paylaşıma izin vermesi gerekir:

- Ayar: **Hesap → Organizasyonlar → Dış paylaşıma izin ver**.
- **Varsayılan olarak kapalıdır.**
- Değiştirmek için organizasyonu düzenleme yetkisi gerekir.
- Açmak hemen geçerlidir. Kapatmak bir onay ister (**Dış paylaşım kapatılsın mı?**).

İzin kapalıyken **Paylaşım ekle** penceresinde bu iki tür seçilemez ve liste bunu söyler. Ayarı değiştirme yetkiniz varsa **Organizasyon ayarlarını aç** düğmesi ayarı yeni sekmede vurgulanmış olarak açar; ayarı açıp geri döndüğünüzde seçenekler sayfa yenilemeden etkinleşir. Yetkiniz yoksa bir organizasyon yöneticisinden bu ayarı açmasını isteyin.

İzni kapatırsanız:

- Organizasyonun tüm korunan servislerindeki bağlı organizasyon ve e-posta paylaşımları **Askıda** olur.
- Bu paylaşımlarla girenlerin erişimi yaklaşık 30 saniye içinde sona erer; bu servislerdeki diğer ziyaretçiler de bir kez yeniden giriş yapar.
- Yeni dış paylaşım eklenemez.

İzni yeniden açarsanız e-posta paylaşımları geri gelir; bağlı organizasyon paylaşımları ise iki organizasyon arasındaki bağ hâlâ sürüyorsa geri gelir. Komuta bağı 5 dakikada bir denetler; bağ koptuysa (iki organizasyon arasında birbirine bağlı ve aktif hiçbir kullanıcı hesabı kalmadıysa) bağlı organizasyon paylaşımı askıya alınır.

---

## İzinler

| İzin | Ne sağlar |
|---|---|
| **Korunan servisin paylaşımlarını yönet** | Paylaşım eklemek, düzenlemek, kaldırmak; paylaşım bağlantısı ve servis token'ı oluşturmak ve silmek; ziyaretçileri oturumdan çıkarmak. |
| **Servis erişim korumasını yönet** | Korumayı açıp kapatmak, **Oturum süresi** dahil kuralları ve ayarları değiştirmek. Paylaşımları görebilir ama değiştiremez. |

İki izinden biri olan kişi paylaşım listesini, paylaşım bağlantılarını, token'ları, **Kimler içeride** listesini, erişim önizlemesini ve erişim kaydını görebilir. İkisi de olmayan ama servisi görebilen kişi **Kişiler** sekmesinde yalnızca paylaşım sayısını görür. **Paylaşım ekle → Bir üye** listesinde üyelerin e-posta adresleri yalnızca kullanıcıları görme izni olanlara gösterilir.

---

## İlgili Dokümanlar

- [Erişim Koruması](service-access-protection.md) — genel bakış.
- [Kurallar](access-protection-rules.md) — Komuta girişinin hangi yollarda istendiği.
- [Erişim Kaydı](access-protection-activity.md) — kimin giriş yaptığı ve kimin paylaşım bağlantısı kullandığı.
- [Başvuru](access-protection-reference.md) — giriş hata mesajları.
