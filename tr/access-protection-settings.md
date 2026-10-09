# Bitiş ve Kimlik Bildirme

**Erişim ve portlar** sayfasının **Ayarlar** sekmesinde üç bölüm vardır:

- **Koruma bitişi** — korumanın ne zaman biteceği ve bittiğinde ne olacağı.
- **Giriş yapanı uygulamama bildir** — giriş yapan ziyaretçinin kim olduğunun uygulamanıza iletilmesi.
- **Korumayı hemen kaldır** — **Şimdi herkese aç** düğmesi.

Bu bölümler korumayı açıp kaydettikten sonra görünür (**Hazırlanıyor** sırasında da). Koruma kapalıyken sekmede **Erişim koruması kapalı** notu görünür. Ayarları değiştirmek için **Servis erişim korumasını yönet** izni gerekir; bu izni olmayanlar yalnızca özet cümlesini görür.

---

## Koruma bitişi

Bir uygulamayı belirli bir süre için korumak istiyorsanız (örneğin bir lansmana kadar ya da bir müşteri sunumu boyunca) korumaya bir bitiş tarihi verebilirsiniz.

Bölümün başındaki cümle mevcut durumu özetler:

| Özet | Anlamı |
|---|---|
| "Bitiş yok — siz kapatana kadar koruma açık kalır." | Bitiş tarihi yok. |
| "{tarih} tarihinde bitiyor; sonra servis kilitli kalır." | Bitiş var, bittiğinde koruma sürer. |
| "{tarih} tarihinde bitiyor; sonra servis herkese açılır." | Bitiş var, bittiğinde koruma kalkar. |
| "{tarih} bitiş tarihi geçti; koruma açık kaldı." | Bitiş geçti, koruma "kilitli kalsın" ayarıyla sürüyor. |
| "{tarih} bitiş tarihi geçti; servis yakında herkese açılacak." | Bitiş geçti, koruma kaldırılıyor. |

### Bitiş tarihi koyma ya da değiştirme

1. **Bitiş tarihi ve saati** alanına tarihi ve saati girin. Saatler hesabınızda seçtiğiniz saat dilimindedir (**Hesap → Genel**). Cihazınız başka bir saat dilimindeyse alanın altında bir uyarı çıkar ve seçtiğiniz saatin cihazınızda kaça denk geldiği yazar.
2. **Bittiğinde** listesinden ne olacağını seçin:
   - **Sonra kilitli kalsın** (varsayılan) — bitiş zamanı geldiğinde koruma olduğu gibi sürer; siz karar verene kadar hiçbir şey açılmaz.
   - **Sonra herkese açılsın** — bitiş zamanı geldiğinde koruma kaldırılır ve servis herkese açılır (**Şimdi herkese aç** ile aynı sonuç).
3. **Bitişi kaydet** ile kaydedin.

Kurallar:

- Bitiş gelecekte olmalı ve en fazla **365 gün** sonrası olabilir.
- Bitiş tarihi koruma açıkken (kaydedildikten sonra) konabilir. **Kurallar** sekmesinde yaptığınız kayıtlar mevcut bitiş tarihini değiştirmez.
- **Kalıcı yap** düğmesi bitiş tarihini kaldırır (**Koruma kalıcı yapılsın mı?** onayıyla). Alanı temizleyip kaydetmek de aynı onayı açar.
- Servisin genel URL'i kapalıysa ya da henüz genel adresi yoksa bitişi yalnızca ileri alabilir, servisi kilitli tutabilir ya da korumayı kalıcı yapabilirsiniz; erişimi genişletecek değişiklikler (bitişi öne çekmek, olmayan bir bitiş eklemek, "herkese açılsın" seçmek) yapılamaz.

### Bitiş zamanı geldiğinde

- **Sonra kilitli kalsın** seçiliyse hiçbir şey değişmez; koruma sürer ve size "Koruma bitti, servis hâlâ kilitli" e-postası gelir.
- **Sonra herkese açılsın** seçiliyse koruma kaldırılmaya başlar ve "Koruma bitti, servis açık" e-postası gelir. Paylaşımlar silinmez. Koruma tamamen kalkana kadar yeni giriş yapılamaz; bu seçenekle açılan oturumlar zaten hiçbir zaman bitiş zamanını geçmez.
- Bitiş genellikle tam zamanında işlenir; en geç birkaç dakika içinde gerçekleşir.
- Organizasyon aktif değilse (askıya alınmış, silinmiş ya da yasaklanmış) bitiş işlenmez ve koruma olduğu gibi kalır.

### Hatırlatma e-postaları

Bitiş tarihi olan bir korumada Komuta üç e-posta gönderir:

| Ne zaman | Konu |
|---|---|
| Bitişten 24 saat önce | "{servis} erişim koruması {tarih} tarihinde bitiyor" |
| Bitişten 1 saat önce | Aynı |
| Bitiş zamanında | "{servis} erişim koruması bitti; servis hâlâ kilitli" ya da "{servis} erişim koruması bitti; servis artık herkese açık" |

- **Alıcılar:** servise kişisel olarak düzenleme yetkisi verilmiş ve **Servis erişim korumasını yönet** iznine sahip, e-postası doğrulanmış aktif kullanıcılar. Böyle biri yoksa bu izne sahip organizasyon yöneticileri. En fazla 50 kişi.
- **Dil:** organizasyonun varsayılan dili Türkçe ise Türkçe, değilse İngilizce. E-postadaki tarihler UTC olarak yazılır.
- Bitişi 24 saatten (ya da 1 saatten) daha yakın bir zamana koyarsanız, zamanı geçmiş hatırlatmalar gönderilmez. Bitiş tarihini değiştirirseniz hatırlatmalar yeni zamana göre yeniden kurulur; yalnızca **Bittiğinde** seçimini değiştirmek gönderilmiş hatırlatmaları yeniden göndermez (sonrakiler yeni seçimle gider).
- Bitiş e-postası yalnızca bitişten sonraki 7 gün içinde gönderilir.
- Yaklaşan bitiş ve "hâlâ kilitli" e-postalarında üç bağlantı vardır: **Şimdi herkese aç**, **Süreyi uzat** ve **Kalıcı yap**. Bağlantılar konsolda **Ayarlar** sekmesini açar; işlem orada sizin onayınızla yapılır (**Şimdi herkese aç** ve **Kalıcı yap** onay penceresini açar, **Süreyi uzat** imleci tarih alanına koyar). Bağlantı bir kez çalışır ve yalnızca **Servis erişim korumasını yönet** izni olanlar için etkilidir.

Her paylaşımın kendi **Erişim bitişi** ayrıca geçerlidir; korumanın bitişi paylaşımların bitişini değiştirmez (bkz. [Giriş ve Paylaşım](access-protection-sign-in-sharing.md#paylaşım-eklerken-seçilenler)).

---

## Giriş yapanı uygulamama bildir

Bu ayar açıkken Komuta, giriş yapmış ziyaretçinin kim olduğunu her istekte uygulamanıza iletir. Böylece uygulamanız ayrı bir giriş ekranı yazmadan "kim bağlandı" bilgisini kullanabilir: kayıt tutmak, kişiye göre içerik göstermek ya da yetki vermek için.

- **Varsayılan olarak kapalıdır.**
- Komuta girişinin bir yerde gerekli olmasını ister (sitede ya da bir yol kuralında). Giriş istenmiyorsa anahtar kapalı kalır ve "Önce Kurallar sekmesinde Komuta girişini açın; giriş olmadan kim olduğu bilinmez." yazar.
- Bölüm görünmüyorsa özellik platformunuzda henüz açık değildir.
- Kurallarda girişi kaldırırsanız ayar kapanır; girişi yeniden açtığınızda kendiliğinden açılmaz. Korumayı kapatmak ise ayarı korur: korumayı Komuta girişiyle yeniden açtığınızda ziyaretçi bilgileri yeniden iletilmeye başlar.

### Açma

Anahtarı açın; ayar hemen kaydedilir ("Giriş yapan ziyaretçi artık uygulamanıza bildirilecek"). Komuta servisinizin yönlendirme ayarlarını günceller; bu sırada bölüm "Hazırlanıyor: servisin rotaları güncelleniyor. O zamana kadar uygulamanız başlıkları boş alır." der (Komuta'nın değerleri boş gelir; ziyaretçi bu başlıkları kendisi gönderirse dolu görünebilir, aşağıya bakın). Bu genellikle birkaç dakika sürer. Uzun sürerse bölüm "Servisi yeniden dağıtmak rotaları günceller." der; servisi yeniden dağıtmanız yeterlidir.

### Uygulamanızın aldığı başlıklar

| Başlık | İçerik |
|---|---|
| `x-komuta-user-email` | Ziyaretçinin doğrulanmış e-posta adresi. |
| `x-komuta-user-id` | Ziyaretçinin Komuta kullanıcı kimliği (GUID); Komuta hesabı olmadan e-posta koduyla giren biri için `eml:` ve 32 küçük harfli onaltılık karakter; kimlik sağlayıcınızla giren biri için `sso:` ve 32 küçük harfli onaltılık karakter. |
| `x-komuta-identity` | Ziyaretçiyi tanıtan, Komuta tarafından imzalanmış bir kanıt (JWT). |

Bilmeniz gerekenler:

- **Başlıklar yalnızca girişin gerektiği isteklerde dolu gelir.** Sitenin herkese açık kısımlarında, IP listesiyle girişsiz geçilen yerlerde ve webhook yollarında, ziyaretçi giriş yapmış olsa bile başlıklar boş gelir.
- **Servis token'ıyla gelen isteklerde** yalnızca `x-komuta-identity` dolu gelir; içinde `kind` değeri `service_token`, `sub` değeri token'ın kimliğidir: token değerindeki `kst_` sonrasındaki 32 onaltılık karakterin tireli GUID biçimi. Diğer iki başlık boştur.
- **Komuta hesabı olmadan e-posta koduyla giren birinin isteklerinde** üç başlık da dolu gelir: `x-komuta-user-email` kanıtlanan adres, `x-komuta-user-id` ve JWT'deki `sub` `eml:<32 onaltılık>`, `kind` ise `email`. Bu kimlik organizasyona özeldir: aynı adres organizasyonunuzun tüm servislerinde aynı, başka yerlerde farklı kimliği alır. Yani `x-komuta-user-id` her zaman GUID değildir; uygulamanız onu GUID olarak çözümlüyorsa önce `kind` değerine bakın.
- **Organizasyonunuzun kimlik sağlayıcısıyla giren birinin isteklerinde** `x-komuta-user-id` ve JWT'deki `sub` `sso:<32 onaltılık>`, `kind` ise `sso` olur. Kimlik organizasyona özeldir: aynı sağlayıcıdaki aynı hesap organizasyonunuzun tüm servislerinde aynı kimliği alır. `x-komuta-user-email` (ve JWT'deki `email`) yalnızca sağlayıcı adresi doğrulanmış olarak işaretlediğinde ve adres organizasyonunuzun doğrulanmış alan adlarından birinde, Google Workspace'te ise Workspace alan adında olduğunda dolu gelir; aksi halde boştur. Komuta sağlayıcıdan e-postayı yalnızca ID token adresin doğrulandığını söylediğinde (`email_verified`, Microsoft Entra ID için dizin üyesinde `xms_edov`) alır; Microsoft Entra ID bunları yalnızca ID token'a `email`, `xms_edov` ve `acct` isteğe bağlı claim'leri eklendiğinde gönderir, aksi halde ziyaretçileri, konukları ise her zaman, e-postasız gelir ve `sso:` kimlikleriyle tanınır. Bkz. [Organizasyonunuzun kimlik sağlayıcısıyla giriş](access-protection-sign-in-sharing.md#uygulamanızın-aldıkları).
- **Paylaşım bağlantısıyla giren ziyaretçinin isteklerinde** yalnızca `x-komuta-identity` dolu gelir; `kind` değeri `share_link`, `sub` değeri bağlantının kimliğidir (tireli GUID) ve `email` alanı yoktur. Diğer iki başlık boştur.
- **Ayarı açmadan önce giriş yapmış ziyaretçiler** yeniden giriş yapana kadar (en fazla servisin oturum süresi kadar, bkz. [Giriş ve Paylaşım](access-protection-sign-in-sharing.md#oturum-süresi)) e-postasız bildirilir: `x-komuta-user-email` boş gelir ve JWT'de `email` alanı olmaz.
- Bölüm "Hazırlanıyor" ya da "rotaları henüz güncellenmedi" demiyorsa, ziyaretçinin kendi gönderdiği aynı adlı başlıklar Komuta'dan geçerken silinir ve doğru değerle (ya da boş) yeniden yazılır; internetten gelen biri bu başlıkları taklit edemez. Hazırlık sürerken düz başlıklara güvenmeyin; imzalı `x-komuta-identity` her zaman doğrulanabilir.

### Hangi başlığa güvenmeli

Aynı kümedeki servisleriniz ve **Makineler** sekmesinde özel ağ için seçtiğiniz servisler pod'larınıza Komuta'dan geçmeden ulaşabildiği için bu başlıkları kendileri de gönderebilir. Bu sizin için önemliyse düz başlıklara değil, yalnızca **imzalı `x-komuta-identity` başlığına** güvenin ve her istekte doğrulayın. Düz başlıklar kolaylık içindir. `kind` değerine de bakın: `share_link` ziyaretçisi tanınan bir kişi değil, bağlantıyı elinde tutan herhangi biridir; `email` ziyaretçisi bir Komuta hesabı değil, bir e-posta adresini kanıtlamış biridir; `sso` ziyaretçisi de bir Komuta hesabı değil, organizasyonunuzun kimlik sağlayıcısının giriş yaptırdığı biridir.

### Kimlik kanıtı (JWT)

`x-komuta-identity`, ES256 ile imzalanmış bir JWT'dir.

Başlık (header): `{"alg": "ES256", "typ": "JWT", "kid": "<anahtar kimliği>"}`

| Alan | Değer |
|---|---|
| `iss` | Her zaman `komuta-access`. |
| `aud` | Ziyaretçinin açtığı adres (host), örneğin `panel.example.com`. Servisin her adresi (özel alan adı, `*.komuta.app` adresi, mavi-yeşil önizleme adresi) ayrı bir `aud` değeridir. |
| `sub` | Komuta kullanıcı kimliği; servis token'ında token'ın kimliği; paylaşım bağlantısında bağlantının kimliği; hesapsız e-posta ziyaretçisinde `eml:<32 onaltılık>`; kimlik sağlayıcısı ziyaretçisinde `sso:<32 onaltılık>`. |
| `kind` | `user`, `email`, `sso`, `service_token` ya da `share_link`. |
| `email` | Ziyaretçinin e-postası. Yalnızca `user`, `email` ve `sso` türünde ve e-posta bilindiğinde bulunur (`sso` için yukarıya bakın). |
| `sid` | Servisin kimliği. Konsolda servis adresindeki `/services/<kimlik>` bölümüdür. |
| `tid` | Organizasyonun (kiracının) kimliği. |
| `iat` | İmzalanma zamanı (Unix saniyesi). |
| `nbf` | `iat` − 30 saniye. |
| `exp` | `iat` + 5 dakika. |

**Açık anahtarlar:** `https://api.komuta.io/api/devopszon/access-protection/identity-keys`

Bu adres standart bir JWKS döndürür (`{"keys":[{"kty":"EC","crv":"P-256","alg":"ES256","use":"sig","kid":"…","x":"…","y":"…"}]}`). Giriş gerektirmez ve 5 dakika önbelleğe alınabilir. Liste, bir kümede ilk servis bu ayarı açtığında dolar; o zamana kadar boş olabilir. JWT'de tanımadığınız bir `kid` görürseniz anahtar listesini yeniden çekin.

### Doğrulama kuralları

Aynı anahtar bir kümedeki tüm organizasyonların servisleri için imza atar. Bu yüzden imzayı doğrulamak tek başına yetmez; başka bir organizasyonun uygulaması, kendisine gelen geçerli bir jetonu sizin uygulamanıza tekrar gönderebilir. Uygulamanız şunların **hepsini** kontrol etmelidir:

1. `alg` değeri `ES256` olmalı (başka algoritmayı kabul etmeyin).
2. `kid` yayımlanan anahtar listesinde olmalı ve imza o anahtarla doğrulanmalı.
3. `iss` değeri `komuta-access` olmalı.
4. `aud` **kendi** adreslerinizden biri olmalı. Ziyaretçinin gönderdiği `Host` başlığına değil, uygulamanızda tanımlı adres listesine bakın.
5. `sid` **kendi** servisinizin kimliği olmalı.
6. `exp` geçmemiş ve `nbf` gelmiş olmalı (saat farkı için 30 saniye pay bırakabilirsiniz).

### Örnek: Node.js (jose)

```javascript copy
import { createRemoteJWKSet, jwtVerify } from "jose";

const KOMUTA_KEYS = createRemoteJWKSet(
  new URL("https://api.komuta.io/api/devopszon/access-protection/identity-keys"),
);

const MY_HOSTS = ["panel.example.com"];
const MY_SERVICE_ID = "3a2331e6-0000-0000-0000-000000000002";

export async function komutaVisitor(headers) {
  const token = headers["x-komuta-identity"];
  if (!token) return null;

  const { payload } = await jwtVerify(token, KOMUTA_KEYS, {
    algorithms: ["ES256"],
    issuer: "komuta-access",
    audience: MY_HOSTS,
    clockTolerance: 30,
  });

  if (payload.sid !== MY_SERVICE_ID) {
    throw new Error("identity token belongs to another service");
  }

  return {
    kind: payload.kind,
    userId: payload.sub,
    email: payload.email ?? null,
  };
}
```

`jwtVerify`; imzayı, `kid`'in listede olmasını, `alg`, `iss`, `aud`, `exp` ve `nbf` alanlarını kontrol eder ve uymayan jetonda hata fırlatır. `createRemoteJWKSet` anahtarları önbelleğe alır ve tanımadığı bir `kid` gördüğünde listeyi yeniden çeker. Express'te `komutaVisitor(req.headers)` olarak çağrılır.

### Örnek: Python (PyJWT)

```python copy
import jwt

KOMUTA_KEYS = jwt.PyJWKClient(
    "https://api.komuta.io/api/devopszon/access-protection/identity-keys",
    headers={"User-Agent": "my-app/1.0"},
)

MY_HOSTS = ["panel.example.com"]
MY_SERVICE_ID = "3a2331e6-0000-0000-0000-000000000002"


def komuta_visitor(headers):
    token = headers.get("x-komuta-identity")
    if not token:
        return None

    signing_key = KOMUTA_KEYS.get_signing_key_from_jwt(token)
    claims = jwt.decode(
        token,
        signing_key.key,
        algorithms=["ES256"],
        issuer="komuta-access",
        audience=MY_HOSTS,
        leeway=30,
        options={"require": ["exp", "nbf", "iat", "sub", "sid"]},
    )

    if claims["sid"] != MY_SERVICE_ID:
        raise PermissionError("identity token belongs to another service")

    return {"kind": claims["kind"], "user_id": claims["sub"], "email": claims.get("email")}
```

`pip install "pyjwt[crypto]"` ile kurulur. `headers={"User-Agent": …}` satırını silmeyin: Komuta API'si Python'un varsayılan istemci kimliğiyle gelen istekleri reddeder.

### Kapatma

Anahtarı kapattığınızda başlıklar birkaç saniye içinde boş gelmeye başlar ("Giriş yapan ziyaretçi artık uygulamanıza bildirilmeyecek").

---

## Korumayı hemen kaldır

**Ayarlar** sekmesinin en altındaki **Şimdi herkese aç** düğmesi korumayı hemen kaldırır: giriş, IP izin listesi, yol kuralları, ülkeler ve hız sınırı artık uygulanmaz ve URL'e sahip herkes servisi açabilir. **Bu servis herkese açılsın mı?** onayı istenir. Paylaşımlar, paylaşım bağlantıları ve servis token'ları saklanır; korumayı yeniden açarsanız tekrar geçerli olur. Ayrıntılar için bkz. [Erişim Koruması](service-access-protection.md#korumayı-kapatma).

---

## İlgili Dokümanlar

- [Erişim Koruması](service-access-protection.md) — açma, kapatma ve durumlar.
- [Giriş ve Paylaşım](access-protection-sign-in-sharing.md) — ziyaretçi oturumları.
- [Makineler ve Özel Ağ](access-protection-machines.md) — servis token'ları.
- [Başvuru](access-protection-reference.md) — başlıklar ve sınırlar.
