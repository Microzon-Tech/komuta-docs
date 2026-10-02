# Servis Erişim Koruması

Erişim koruması, servisinizin genel URL'ini kimlerin açabileceğini belirler. Geliştirme aşamasındaki bir uygulamayı yalnızca ekibinize, bir müşterinize ya da ofis ağınıza açmak için kullanılır; uygulamanıza ayrıca bir giriş ekranı yazmanız gerekmez.

Erişim koruması her planda bulunur. **Servis Detay → Yapılandırma → Portlar** sayfasındaki **Erişim koruması** kartından yönetilir. Kartı değiştirmek için servisi düzenleme yetkiniz olmalıdır; bu yetki yoksa kart salt okunur görünür.

---

## Koruma Türleri

Kartta birbirinden bağımsız iki kontrol vardır. Biri ya da ikisi birlikte açılabilir.

| Kontrol | Ne yapar |
|---|---|
| **Komuta girişi iste** | Ziyaretçi önce Komuta'ya giriş yapmaya yönlendirilir. Yalnızca **Paylaşılanlar** listesindeki bir paylaşımla eşleşen hesaplar içeri girer. |
| **IP izin listesi** | Servise yalnızca listedeki genel adreslerden ulaşılabilir. Diğer adreslerden gelenler **Erişim kısıtlı** sayfasını görür (HTTP 403). |

Kontrollerin birlikte nasıl çalıştığı:

- **İkisi birlikte açık** — ziyaretçinin listedeki bir adresten gelmesi **ve** giriş yapıp bir paylaşımla eşleşmesi gerekir.
- **Yalnızca IP izin listesi** — listedeki adreslerden gelenler giriş yapmadan girer, diğer herkes 403 alır.
- **Yalnızca Komuta girişi** — adresi ne olursa olsun, paylaşımla eşleşen her hesap girer.
- **İkisi de kapalı** — servis herkese açıktır; kart "Herkese açık — URL'e sahip herkes bu servisi açabilir." uyarısını gösterir.

Ayarlar **Korumayı uygula** düğmesiyle kaydedilir.

---

## IP İzin Listesi

Her satıra bir IPv4 ya da IPv6 adresi veya CIDR aralığı yazılır (`8.8.8.8`, `8.8.8.0/24` gibi). Kurallar:

- Yalnızca genel internet adresleri kabul edilir. Özel ve ayrılmış aralıklar (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, `100.64.0.0/10`, loopback, link-local, dokümantasyon aralıkları, `fc00::/7` gibi) ve bunlarla çakışan aralıklar reddedilir.
- Aralığın host bitleri sıfır olmalıdır: `8.8.8.0/24` geçerlidir, `8.8.8.5/24` geçerli değildir.
- Tüm adresleri kapsayan `0.0.0.0/0` ya da `::/0` kabul edilmez; herkese açmak istiyorsanız korumayı kapatın.
- Liste en fazla 100 girdi alır. Aynı girdi iki kez yazılırsa bir kez kaydedilir.

### Kendi IP'mi ekle

**Kendi IP'mi ekle** düğmesi, Komuta konsolunun gördüğü genel adresinizi listeye ekler:

- IPv4 bağlantısında yalnızca adresiniz eklenir.
- IPv6 bağlantısında adresiniz `/64` bloğu içinde değişebildiği için bloğun tamamı (`/64`) eklenir.

Eklenen adres, konsolun gördüğü adrestir. Tarayıcınız servise farklı bir bağlantıyla ulaşıyor olabilir (örneğin konsola IPv6, servise IPv4); bu durumda servis başka bir adres görür ve kart sizi bu konuda uyarır. Kendinizi dışarıda bırakmamak için uygulamadan önce adresi kontrol edin; gerekirse hem IPv4 hem IPv6 adresinizi ekleyin.

Dışarıda kaldıysanız **Erişim kısıtlı** sayfası servisin gördüğü adresi gösterir. Bu adresi kopyalayıp konsoldan listeye ekleyebilirsiniz.

### Cloudflare gereksinimi

IP izin listesi, servise gelen isteklerin Cloudflare üzerinden (proxied) geçmesini gerektirir. Komuta'nın ürettiği `*.komuta.app` adresleri ve **Alan Adları** altında eklenen özel domainler bu şekilde çalışır. IP izin listesi açıkken Cloudflare üzerinden gelmeyen istekler reddedilir.

---

## Komuta Girişi ve Paylaşımlar

**Komuta girişi iste** açıkken servisinizi açan ziyaretçi Komuta giriş sayfasına yönlendirilir. Giriş yaptıktan sonra "Erişiminiz kontrol ediliyor" ekranı görünür ve hesap bir paylaşımla eşleşiyorsa ziyaretçi açmak istediği sayfaya geri döner.

Kimlerin girebileceği kartın altındaki **Paylaşılanlar** listesinden **Paylaşım ekle** ile belirlenir:

| Paylaşım türü | Kimler girer |
|---|---|
| **Organizasyonunuz** | Bu organizasyonun tüm aktif üyeleri. |
| **Bir üye** | Organizasyondaki tek bir kişi. |
| **Bağlı bir organizasyon** | Üyesi olduğunuz başka bir organizasyondaki herkes; sonradan katılanlar dahil. |
| **Bir e-posta adresi** | Organizasyonlarınızın dışındaki biri. Herhangi bir Komuta hesabıyla giriş yapar, ardından bu adrese gönderilen tek kullanımlık kodla adresi doğrular. |

Dikkat edilecekler:

- Paylaşım eklemek için önce erişim korumasını açmanız gerekir.
- Paylaşımlar yalnızca **Komuta girişi iste** açıkken geçerlidir.
- Giriş açık ama hiç paylaşım yoksa kimse girişten geçemez.
- Her paylaşıma isteğe bağlı bir **Erişim bitişi** verilebilir. Boş bırakılırsa erişim siz kaldırana kadar sürer. Bitiş tarihi listedeki kalem simgesiyle sonradan değiştirilebilir; süresi dolan paylaşım **Süresi doldu** etiketiyle görünür.
- Bir serviste en fazla 200 paylaşım olabilir.

### E-posta paylaşımı

E-posta adresiyle paylaşılan kişi servisi açtığında herhangi bir Komuta hesabıyla giriş yapar. **Erişiminiz yok** ekranındaki "E-posta adresinizle mi paylaşıldı?" bölümünden adresini girip **Bana kod gönder** der. Adrese 8 haneli, 10 dakika geçerli bir kod gönderilir; kod girilip **Doğrula ve devam et** seçildiğinde servis açılır.

Yalnızca o posta kutusunu okuyabilen biri girebilir ve her girişte yeni bir kod gerekir. Kod istekleri saat başına sınırlıdır.

### Dış paylaşım izni

**Bağlı bir organizasyon** ve **Bir e-posta adresi** paylaşımları, organizasyonunuzun dışarıyla paylaşıma izin vermesini gerektirir. Bu izin organizasyon ayarlarındaki **Dış paylaşıma izin ver** seçeneğiyle bir organizasyon yöneticisi tarafından açılır.

İzin kapatılırsa bu tür paylaşımlar eklenemez; mevcut olanlar **Askıda** etiketiyle askıya alınır ve bu paylaşımlarla giriş yapmış kişilerin erişimi sona erer. İzin tekrar açıldığında askıdaki paylaşımlar yeniden geçerli olur; bağlı organizasyon paylaşımlarının geri gelmesi için organizasyonlar arasındaki bağın hâlâ sürmesi gerekir.

### Erişimin kaldırılması

Bir paylaşımı kaldırdığınızda, o paylaşımla giren kişilerin erişimi açık oturumlar dahil yaklaşık 30 saniye içinde sona erer. Korumayı kapatmak ise tersini yapar: servis yeniden herkese açılır.

Korumayı kapatmak paylaşımları silmez; korumayı tekrar açtığınızda aynı paylaşımlar yeniden geçerli olur.

---

## Ziyaretçi Ne Görür?

| Durum | Ziyaretçinin gördüğü |
|---|---|
| Giriş yaptı ve bir paylaşımla eşleşiyor | Servis açılır. |
| Giriş yaptı ama hiçbir paylaşımla eşleşmiyor | **Erişiminiz yok** sayfası. |
| IP izin listesindeki bir adresten gelmiyor | **Erişim kısıtlı** sayfası (HTTP 403). |

**Erişiminiz yok** sayfası oturum açılan hesabı ve organizasyonu gösterir. Ziyaretçi buradan e-posta koduyla doğrulama yapabilir, **Başka bir organizasyonla devam et** ile üyesi olduğu başka bir organizasyona geçebilir ya da **Başka bir hesapla giriş yap** seçebilir.

**Erişim kısıtlı** sayfası, servisin yalnızca izin verilen ağlardan açılabildiğini söyler ve servisin gördüğü adresi kopyalanabilir şekilde gösterir. Bu sayfada giriş seçeneği sunulmaz; adres listede değilse giriş yapmak da erişim sağlamaz.

Tarayıcı dışı istemciler için: giriş isteyen bir servise oturumsuz gelen `GET`/`HEAD` dışındaki istekler (örneğin bir API'ye `POST`) giriş sayfasına yönlendirilmez, HTTP 401 alır. Makineden makineye erişim gereken servislerde IP izin listesi daha uygundur.

---

## Koruma Bitişi

Koruma açıkken kartta **Koruma bitişi** bölümü görünür. Burada korumanın ne zaman biteceği ve bittiğinde ne olacağı seçilir:

- **Sonra kilitli kalsın** — bitiş zamanı geldiğinde koruma olduğu gibi sürer; siz karar verene kadar servis açılmaz.
- **Sonra herkese açılsın** — bitiş zamanı geldiğinde koruma kaldırılır ve servis herkese açılır.

Bitiş tarihi isteğe bağlıdır ve en fazla 365 gün sonrası olabilir. Saatler, ayarlarınızda seçtiğiniz saat dilimindedir. Bitiş tarihi olmayan koruma, siz kapatana kadar açık kalır.

Bitiş tarihi olan bir korumada servisi düzenleyebilen kişilere (böyle biri yoksa organizasyon yöneticilerine) bitişten bir gün ve bir saat önce, ayrıca bitişte e-posta gönderilir. E-postadaki bağlantılarla korumayı **Şimdi herkese aç**, **Süreyi uzat** ya da **Kalıcı yap** seçeneklerine doğrudan gidilir.

Bu bölümdeki diğer düğmeler:

- **Kalıcı yap** — bitiş tarihini kaldırır; servis siz korumayı kapatana kadar korunur.
- **Şimdi herkese aç** — korumayı hemen kaldırır: giriş ve IP izin listesi artık uygulanmaz, URL'e sahip herkes servisi açabilir. Paylaşımlar saklanır.

---

## Durum ve İlerleme

Kartın sağ üstündeki rozet korumanın durumunu gösterir:

| Durum | Anlamı |
|---|---|
| **Kapalı** | Koruma yok; servis herkese açık. |
| **Hazırlanıyor** | Koruma hazırlanıyor. |
| **Uygulanıyor** | Koruma servise uygulanıyor. |
| **Korunuyor** | Koruma devrede. |
| **Kapatılıyor** | Koruma kaldırılıyor. |

Koruma açılırken kartta bir ilerleme çubuğu, o anki adım ve son durum değişikliğinin zamanı görünür. Korunan bir serviste yaptığınız bir değişiklik henüz yayılıyorsa kart "Son değişikliğiniz uygulanıyor." notunu gösterir. Ayarın tamamen devrede olduğundan emin olmak için kartın **Korunuyor** göstermesini bekleyin.

Bir uygulama denemesi başarısız olursa kart hatayı gösterir; Komuta otomatik olarak yeniden dener.

Aynı servisin koruması başka bir yerden (başka bir sekme ya da ekip arkadaşınız) değiştirildiyse kart en güncel ayarları yükler ve değişikliğinizi yeniden yapmanızı ister.

---

## Diğer Özelliklerle İlişkisi

- **Uyku modu** — uyuyan bir servis, ziyaretçi kontrolleri geçtikten sonra uyandırılır. Giriş yapmamış ya da izin listesinde olmayan ziyaretçiler korunan bir servisi uyandıramaz.
- **Önbellek** — korunan servislerin yanıtları Cloudflare tarafından önbelleğe alınmaz. Koruma uygulanırken kartta görünen **Önbellek temizliği** adımı, daha önce önbelleğe alınmış içeriği temizler.
- **Alan Adları** — koruma, servisin `*.komuta.app` adresi ve **Alan Adları** altında eklenen özel domainler için birlikte geçerlidir.

---

## Gereksinimler

- Servisin genel bir URL'i olmalıdır; genel URL'i olmayan bir serviste korunacak bir şey yoktur.
- Servis, Komuta'nın paylaşımlı barındırma kümelerinde çalışmalıdır.
- IP izin listesi için servise gelen isteklerin Cloudflare üzerinden geçmesi gerekir (bkz. [Cloudflare gereksinimi](#cloudflare-gereksinimi)).

---

## İlgili Dokümanlar

- [Portlar](services-ports.md) — Erişim koruması kartının bulunduğu sayfa.
- [Ingress ve Domainler](ingress-domains.md) — `*.komuta.app` adresleri ve özel domainler.
- [Servis Uyku Modu](service-sleep.md) — Trafik olmayan servislerin uyutulması ve uyandırılması.
