# Erişim Koruması: Başvuru

Bu sayfa erişim korumasının kesin değerlerini tek yerde toplar: sınırlar, durumlar, ziyaretçiye dönen yanıtlar, başlıklar, hata mesajları, sık sorulan sorular ve terimler sözlüğü. Özelliklerin ne işe yaradığı ve nasıl kullanıldığı ilgili rehber sayfalarında anlatılır; burada yalnızca "tam olarak ne" sorusunun cevabı vardır.

Erişim koruması rehberleri:

- [Erişim Koruması](service-access-protection.md) — genel bakış, açma ve kapatma, durumlar.
- [Kurulum Rehberi](access-protection-tutorial.md) — hızlı başlangıçtan ileri senaryolara adım adım kurulum.
- [Giriş ve Paylaşım](access-protection-sign-in-sharing.md) — Komuta girişi, **Kişiler** sekmesi, ziyaretçinin gördükleri.
- [Kurallar](access-protection-rules.md) — IP izin listesi, yol kuralları, erişim önizlemesi.
- [Makineler ve Özel Ağ](access-protection-machines.md) — webhook yolları, servis token'ları, özel ağ.
- [Erişim Kaydı](access-protection-activity.md) — **Etkinlik** sekmesi.
- [Bitiş ve Kimlik Bildirme](access-protection-settings.md) — **Ayarlar** sekmesi.
- [Stack Manifestinde Erişim Koruması](stack-manifest-access.md) — Stack servisinin `access` bloğu.

---

## Sınırlar

| Konu | Sınır |
|---|---|
| IP izin listesi (site, her yol kuralı, her webhook yolu için ayrı) | En fazla 100 girdi, toplam 4600 karakter; yalnızca genel adresler; `/0` yok; host bitleri sıfır |
| Yol kuralı | Serviste en fazla 50 (webhook yolları dahil); her yol için tek kural |
| Yol uzunluğu | 2–256 karakter; `/` ile başlar; küçük harf, rakam ve `- . _ ~ ! $ & ' ( ) * + , = : @ /` |
| Webhook (açık) yolu | En fazla 10; yöntemler `GET`, `HEAD`, `POST`, `PUT`, `PATCH`, `DELETE` |
| İmzalı webhook yolu | Yalnızca `POST`, `PUT`, `PATCH`; gövde en fazla 65.535 bayt (yaklaşık 64 KiB); yol başına en fazla 2 imza sırrı; sır 8–512 bayt, boşluk ve kontrol karakteri yok; Stripe zamanı en fazla 300 saniye farklı; HMAC değer öneki en fazla 16 karakter |
| Yöntem kuralı | Serviste en fazla 50 (yol kurallarından ayrı); yöntemler `GET`, `HEAD`, `POST`, `PUT`, `PATCH`, `DELETE`, `OPTIONS`; `/` yazılabilir; her yol için tek kural |
| Ülkeler | En fazla 250 iki harfli ISO kodu; `XX` (bilinmeyen) ve `T1` (Tor) listelenemez |
| Hız sınırı | 1–3600 saniyede 10–100.000 istek (konsolda saniyede, 10 saniyede, dakikada, 10 dakikada ya da saatte); adres başına, IPv6'da `/64` başına; her ağ geçidi kopyası ayrı saydığı için yaklaşık |
| Paylaşım | Serviste en fazla 200 |
| Paylaşımın sayfa sınırı | En fazla 50 sayfa (seçildiği "yalnızca seçilen kişiler" yolları dahil) |
| Paylaşım bitişi | Gelecekte olmalı; üst sınır yok |
| Alan adı paylaşımı | Yalnız alan adı (`@` öncesinde ad yok); genel e-posta ve ortak alan adları reddedilir; alt alan adları yalnız doğrulanmış alan adında; Komuta hesabı ve paylaşım başına 24 saatte en fazla 3 farklı adrese kod; 24 saatte 200 yanlış koddan sonra yeni kod yok |
| Doğrulanmış alan adları | Organizasyon başına en fazla 20; TXT `_komuta-verify.<alan adı>`; 6 saatte bir ve **Şimdi denetle** ile (10 sn'de bir) denetlenir; kayıt 2 gün boyunca bulunamazsa (art arda en az 3 denetim) ya da bir hafta DNS cevabı alınamazsa düşer |
| "Yalnızca seçilen kişiler" kuralı | Kural başına en fazla 200 kişi |
| Servis token'ı | Serviste en fazla 20; ad 1–64 karakter; bitiş en fazla 365 gün; en fazla 50 sayfa |
| Paylaşım bağlantısı | Serviste en fazla 50; ad 1–64 karakter; bitiş zorunlu (konsolda 1, 7, 30 ya da 90 gün; API'de en fazla 365 gün sonrası); en fazla 50 sayfa |
| Özel ağdan doğrudan gelebilecek servisler | En fazla 50; aynı organizasyon |
| Koruma bitişi | Gelecekte, en fazla 365 gün sonra |
| Korunan adres (host) sayısı | Servis başına en fazla 50 |
| Ziyaretçi oturumu | 15 dakika, 1 saat, 4 saat, 12 saat (varsayılan), 1 gün ya da 7 gün (**Oturum süresi**); paylaşımın, bağlantının ve "herkese açılsın" bitişinin ötesine geçmez; Komuta hesabı olmayan ziyaretçiler (e-posta kodu ya da kimlik sağlayıcısı) için en fazla 12 saat; ziyaretçi oturumları platformunuzda yönetilmiyorsa 12 saat |
| **Kimler içeride** listesi | En fazla 200 oturum |
| Tek tek çıkarma | Serviste oturum takibi başladıktan 12 saat 10 dakika sonra başlar; bir oturumun ömrü içinde 500'den fazla oturum tek tek kapatılırsa herkesi çıkarmaya döner |
| Giriş bağlantısı | Yaklaşık 9 dakika (girişin bu sürede tamamlanması gerekir); 2 dakikadan az kaldıysa yeni e-posta kodu gönderilmez |
| Giriş denemesi | Kullanıcı başına dakikada 30 |
| E-posta doğrulama kodu | 8 hane, 10 dakika geçerli, 5 hatalı denemede geçersiz |
| E-posta kodu isteme | Kullanıcı başına saatte 20; kullanıcı + adres başına saatte 5; aynı Komuta hesabı, servis ve adres için saatte 3 gönderim (30/60/120 sn bekleme); aynı hesap, servis ve adres için 24 saatte 50 yanlış koddan sonra yeni kod gönderilmez; bir serviste bir adres için 24 saatte 500 yanlış kod yalnızca Komuta'da bir alarm başlatır, engellemez |
| Komuta hesabı olmadan e-posta kodu | Ağ (IPv4 adresi ya da IPv6 `/64`) ve servis başına sayılır: saatte 60 istek, adres başına saatte 5 istek, adres başına saatte 3 gönderim, adres başına 24 saatte 50 yanlış kod; bir serviste bir adres için tüm ağlardan toplam 24 saatte 200 yanlış koddan sonra orada o adres için hesapsız kimse yeni kod alamaz (Komuta hesaplarının kodları etkilenmez); bir adrese bir serviste saatte en fazla 10, organizasyonda 30 böyle kod gider; alan adı paylaşımı bir ağdan 24 saatte en fazla 20 farklı adrese kod gönderir; yanlış kodları, alan adı paylaşımı başına, yalnız hesapsız kodları durduran ayrı bir 24 saatte 200 sınırına sayılır; alan adı paylaşımı başına saatte en fazla 100 kod; IPv6 `/48` (ya da IPv4 adresi) ve servis başına saatte 240 isteklik daha geniş bir sınır |
| Kimlik sağlayıcıları | Organizasyon başına en fazla 5; ad 1–64 karakter; grup claim'i en fazla 64 karakter |
| Kimlik sağlayıcısı paylaşımı | Sağlayıcı ve servis başına bir tane; en fazla 20 grup, her biri en fazla 256 karakter |
| Kimlik sağlayıcısıyla giriş | Ağ (IPv4 adresi ya da IPv6 `/64`) ve servis başına saatte 30 başlatma ve 30 tamamlama; yaklaşık 9 dakika içinde ve aynı tarayıcıda tamamlanmalı |
| İstek yolu (yol kuralı ya da sayfa sınırı olan serviste) | En fazla 1024 bayt; aşarsa `400` |
| Erişim kaydı | Organizasyon başına 30 (varsayılan), 90 ya da 365 gün saklanır; 15 sn'lik paketler; servis başına saatte 500 satır (girişler hariç); sayfa başına 50 kayıt |
| Erişim kaydının dışa aktarımı | CSV ya da JSON; saklama süresi içinde; en fazla 50.000 satır; organizasyon başına aynı anda tek dışa aktarım |
| Paylaşım kaldırıldıktan, bir kişi çıkarıldıktan ya da bağlantı silindikten sonra erişimin kesilmesi | Yaklaşık 30 saniye (tek tek çıkarma devreye girmeden önce paylaşımı kaldırmak servisteki tüm oturumları sonlandırır) |
| Kimlik JWT'si | 5 dakika geçerli; `nbf` = `iat` − 30 sn |

---

## Koruma durumları

| API değeri | Arayüz (TR) | Arayüz (EN) | Kontrol devrede |
|---|---|---|---|
| `Disabled` | Kapalı | Off | Hayır |
| `Preparing` | Hazırlanıyor | Preparing | Hayır |
| `Enforcing` | Uygulanıyor | Applying | Evet (ilk saniyelerde kısa bir gecikme olabilir) |
| `Protected` | Korunuyor | Protected | Evet |
| `Disabling` | Kapatılıyor | Turning off | Son adıma kadar |

`Enforcing` sırasındaki adımlar (`RouteFilter` → `PodTokenLock` → `CachePurge`): ağ geçidi kontrolünün tüm rotalara eklenmesi, pod kilidi, Cloudflare önbelleğinin temizlenmesi. Arayüz adımların adını göstermez, "{toplam} adımın {n} tanesi tamamlandı" der (5 adım: hazırlık, üç uygulama adımı, korunuyor). `Disabling` ters sırada çalışır: önce pod kilidi, sonra ağ geçidi kontrolü kalkar.

Bitiş davranışı (`ExpiryAction`): `KeepLocked` = **Sonra kilitli kalsın** (varsayılan), `OpenToEveryone` = **Sonra herkese açılsın**.

Paylaşım türleri (`ShareKind`): `OwnOrganization` = **Organizasyonunuz**, `Member` = **Bir üye**, `LinkedOrganization` = **Bağlı bir organizasyon**, `Email` = **Bir e-posta adresi**, `EmailDomain` = **Bir alan adındaki herkes**, `IdentityProvider` = **Kimlik sağlayıcınız**.

Birleşim (`Combine`): `All` = **İkisi birden gereksin**, `Any` = **Biri yeterli**.

Uyarı kodlarının (`lastError`) anlamları [Erişim Koruması → Uyarılar ve ne yapmalı](service-access-protection.md#uyarılar-ve-ne-yapmalı) bölümündedir. Orada tek tek anlatılmayan kodlar platform tarafındadır ve sizden bir şey beklemez; örneğin `capability_*` (küme bu özelliği henüz desteklemiyor), `authz_not_confirmed`, `route_filter_not_live`, `pod_token_lock_not_live`, `probe_*` (Komuta'nın kendi koruma testi), `cloudflare_*`, `cluster_*`, `reconcile_*` ve diğer `drift_*` kodları.

---

## Ziyaretçiye dönen yanıtlar

| Durum | HTTP | Yanıt |
|---|---|---|
| Kurallar sağlandı | — | İstek uygulamaya iletilir. |
| Giriş gerekli, oturum yok, `GET`/`HEAD` | `302` | Komuta giriş sayfasına (`https://console.komuta.io/access/<servis kimliği>?host=…&return=…&state=…`) yönlendirme. |
| Giriş gerekli, oturum yok, diğer yöntemler | `401` | `sign-in required` |
| Oturum var ama sayfa paylaşımın dışında / kişi kuralında seçilmemiş, `GET`/`HEAD` | `403` | "Bu sayfaya erişiminiz yok" HTML sayfası (kapsam dışıysa açabileceği en fazla 20 sayfa listelenir) |
| Aynısı, diğer yöntemler | `403` | `this path is not shared with you` |
| IP listede değil (ya da Cloudflare üzerinden geldiği doğrulanamadı), `GET`/`HEAD` | `403` | "Bu servise erişim kısıtlı" HTML sayfası; adres yalnızca doğrulanmış ve listede olmayan bir adres için gösterilir |
| Aynısı, diğer yöntemler | `403` | `access restricted to allowed networks` |
| **Tamamen engelle** kuralı (ya da koruma kurulurken adres henüz tanınmadığında) | `403` | `access denied` |
| Girişten dönüş bağlantısı geçersiz, süresi dolmuş ya da daha önce kullanılmış | `403` | `sign-in link is invalid or expired` (korunan sayfayı yeniden açın) |
| Geçersiz servis token'ı | `401` | `invalid service token`, `WWW-Authenticate: KomutaServiceToken realm="komuta"` |
| Geçerli token, kapsam dışı sayfa | `403` | `this service token cannot open this path` |
| Okunamayan ya da 1024 bayttan uzun yol (yol kuralı/sayfa sınırı olan serviste) | `400` | `bad request` |
| Aynı Komuta çerezi iki kez gönderildi | `400` | `duplicate access cookie` |
| Yöntem, bir yöntem kuralınca izinli değil | `405` | `method not allowed`, `Allow: <kuralın yöntemleri>` |
| Ülke listede değil (ya da bilinmiyor), `GET`/`HEAD` | `403` | Adresle birlikte "Bu servise erişim kısıtlı" HTML sayfası |
| Aynısı, diğer yöntemler | `403` | `access restricted to allowed networks` |
| Hız sınırı aşıldı | `429` | `too many requests`, `Retry-After: <saniye>` |
| Geçerli paylaşım bağlantısı açıldı | `302` | `komuta_link` çıkarılmış aynı adrese yönlendirme (oturum açılır; mevcut Komuta oturumu korunur) |
| Yanlış ya da süresi dolmuş paylaşım bağlantısı | `403` | `This share link is not valid or has expired. Ask the person who sent it for a new one.` |
| Paylaşım bağlantısı `GET`/`HEAD` dışında bir yöntemle açıldı | `403` | `Open a share link in a browser.` |
| Bağlantıyla giren ziyaretçi, bağlantının dışındaki sayfa | `403` | `Your share link does not open this page.` |
| Bağlantı silindikten ya da süresi dolduktan sonra bağlantıyla giren ziyaretçi | `403` | `The share link you opened this site with has ended. Ask the person who sent it for a new one.` |
| **CORS kontrollerine girişsiz izin ver** açıkken tarayıcının CORS kontrolü | — | Uygulamaya iletilir (asıl istek yine giriş ister) |
| İmzalı webhook yolu: imza yok ya da yanlış, ya da yolun kabul etmediği bir istek | `401` | `invalid webhook signature` |
| İmza sırrı olmayan imzalı webhook yolu | `401` | `webhook signature cannot be checked` |
| İmzalı webhook yolu, gövde 65.535 bayttan büyük | `413` | `webhook body too large to verify` |
| Koruma bilgisi geçici olarak alınamıyor | `503` | `access policy unavailable` |
| Giriş geçici olarak kullanılamıyor | `503` | `sign-in unavailable` |

Tüm ret yanıtları `Cache-Control: no-store` taşır. HTML sayfalar ziyaretçinin tarayıcı diline göre Türkçe ya da İngilizce görünür ve dış kaynak yüklemez.

---

## Başlıklar, çerezler ve ayrılmış yollar

| Ad | Yön | Açıklama |
|---|---|---|
| `x-komuta-service-token` | İstemci → Komuta | Servis token'ı (`kst_<32 onaltılık>_<43 karakter>`). Komuta kontrol ettikten sonra siler; uygulamaya ulaşmaz. |
| `x-komuta-user-email` | Komuta → uygulama | Giriş yapan ziyaretçinin e-postası (kimlik bildirme açıkken). Kimlik sağlayıcısıyla giren ziyaretçide yalnızca doğrulanmış alan adlarınızdan birindeki doğrulanmış bir adres (Google Workspace'te Workspace alan adındaki). |
| `x-komuta-user-id` | Komuta → uygulama | Giriş yapan ziyaretçinin Komuta kullanıcı kimliği; hesapsız e-posta koduyla giren biri için `eml:<32 onaltılık>`, kimlik sağlayıcınızla giren biri için `sso:<32 onaltılık>` (kimlik bildirme açıkken). |
| `x-komuta-identity` | Komuta → uygulama | ES256 imzalı kimlik JWT'si (kimlik bildirme açıkken). |
| `x-komuta-access` | Komuta → uygulama | Pod kilidi için servise özel gizli değer. Kullanmayın, loglamayın. |
| `Cache-Control: private, no-store` | Komuta → ziyaretçi | Korunan servisin tüm yanıtlarına yazılır. |
| `__Host-komuta_access` | Çerez | Ziyaretçi oturumu (paylaşım bağlantısıyla girenler için de); yalnızca servisin o adresi için; servisin oturum süresi kadar sürer. Uygulamaya iletilmez. |
| `__Host-komuta_state` | Çerez | Giriş sürerken kullanılan kısa ömürlü çerez (10 dakika). Uygulamaya iletilmez. |
| `komuta_link` | Sorgu parametresi | Paylaşım bağlantısını (`kl_…`) taşır. Komuta bir yönlendirmeyle siler; uygulamaya ve erişim kaydına hiçbir zaman ulaşmaz. |
| `Retry-After` | Komuta → ziyaretçi | Hız sınırından dönen `429` yanıtında beklenecek saniye. |
| `Allow` | Komuta → ziyaretçi | Yöntem kuralından dönen `405` yanıtında yolun kabul ettiği yöntemler. |
| `/.komuta-access/callback` | Yol | Girişten dönüş adresi. `/.komuta-access` ile başlayan yollar Komuta'ya ayrılmıştır; kural ya da webhook yolu tanımlanamaz. |

Kimlik bildirme etkinleştikten sonra (Ayarlar'daki "Hazırlanıyor" notu kalktığında) bu başlıklar ziyaretçi tarafından taklit edilemez: Komuta'dan geçen her izinli istekte silinip yeniden yazılır. İmzalı `x-komuta-identity` her zaman doğrulanabilir. JWT alanları ve doğrulama kuralları için bkz. [Bitiş ve Kimlik Bildirme](access-protection-settings.md#kimlik-kanıtı-jwt).

**Açık anahtarlar (JWKS):** `https://api.komuta.io/api/devopszon/access-protection/identity-keys` — anonim, 5 dakika önbelleklenebilir.

---

## Erişim kaydı neden kodları

Erişim kaydındaki teknik kodlar ve arayüzdeki karşılıkları:

| Kod | Sonuç | Arayüz |
|---|---|---|
| `login_complete` | Giriş | Giriş yaptı |
| `session_valid` | Sayfa görüntüleme | Sayfayı açtı |
| `token_valid` | Sayfa görüntüleme | Sayfayı servis token'ıyla açtı |
| `open_path` | Teslimat | Açık yola teslim edildi |
| `path_not_granted` | Ret | Bu sayfa kendisiyle paylaşılmamış |
| `path_blocked` | Ret | Sayfa engelli |
| `ip_not_allowed` | Ret | İzinli olmayan bir adresten geldi |
| `untrusted_edge`, `untrusted_source`, `edge_required`, `client_ip_invalid` | Ret | İstek Komuta kenarından gelmedi |
| `code_invalid` | Ret | Giriş bağlantısı geçersiz ya da süresi dolmuş |
| `code_rejected` | Ret | Komuta girişi reddetti |
| `token_invalid` | Ret | Bilinmeyen ya da süresi dolmuş bir servis token'ı gönderdi |
| `token_not_allowed` | Ret | Servis token'ı bu sayfayı açamaz |
| `link_opened` | Giriş | Paylaşım bağlantısıyla girdi |
| `link_valid` | Sayfa görüntüleme | Sayfayı paylaşım bağlantısıyla açtı |
| `link_invalid` | Ret | Bilinmeyen ya da süresi dolmuş bir paylaşım bağlantısı açtı |
| `link_not_allowed` | Ret | Paylaşım bağlantısı bu sayfayı açmıyor |
| `link_ended` | Ret | Paylaşım bağlantısı sona erdikten sonra geri geldi |
| `method_not_allowed` | Ret | Bu yolun izin vermediği bir yöntem kullandı |
| `country_not_allowed` | Ret | İzin verilmeyen bir ülkeden geldi |
| `rate_limited` | Ret | Çok fazla istek gönderdi |
| `signature_invalid` | Ret | Geçerli imzası olmayan bir webhook gönderdi |
| `signature_key_missing` | Ret | Henüz imza sırrı olmayan bir yola webhook gönderdi |
| `webhook_body_too_large` | Ret | 64 KiB'tan büyük bir webhook gövdesi gönderdi |
| `overflow` | Toplam | Bu saatteki diğer ziyaretler, birlikte gruplandı |

`preflight` (**CORS kontrollerine girişsiz izin ver** ile geçirilen tarayıcı CORS kontrolü) ağ geçidinin kullandığı bir nedendir, ancak erişim kaydına yazılmaz.

---

## Hata mesajları

Konsolda ya da API'de bir işlem reddedildiğinde gösterilen mesajlar. Süslü parantez içindeki değerler işlem sırasında doldurulur.

### Koruma ayarları

| Kod | Mesaj |
|---|---|
| `DevOpsZon:AccessProtection:NothingProtected` | En az bir IP aralığı ekleyin, Komuta girişini açın ya da bir yol kuralı ekleyin; aksi halde servis korunmaz. |
| `DevOpsZon:AccessProtection:TurnOffWithOpen` | Erişim korumasını kapatmak için kapatma işlemini kullanın; tüm IP aralıklarını, girişi ve yol kurallarını kaldıran bir değişiklik kaydedilmez. |
| `DevOpsZon:AccessProtection:AllowListInvalid` | '{Entry}' geçerli bir IP adresi ya da CIDR aralığı değil. 8.8.8.8 veya 8.8.8.0/24 biçimini kullanın; aralığın host bitleri sıfır olmalı. |
| `DevOpsZon:AccessProtection:AllowListNotPublic` | '{Entry}' özel, ayrılmış ya da dokümantasyon aralığıdır veya bunlarla çakışıyor. Yalnızca genel internet adreslerine izin verilebilir. |
| `DevOpsZon:AccessProtection:AllowListEverything` | '{Entry}' tüm adreslere izin verir. Bunun yerine erişim korumasını kapatın. |
| `DevOpsZon:AccessProtection:AllowListTooLong` | IP izin listesi çok uzun (sınır: {Max}). Aralıkları birleştirin ya da girdi silin. |
| `DevOpsZon:AccessProtection:InvalidCombine` | IP izin listesi ile Komuta girişinin nasıl birleşeceğini seçin: ikisi birden ya da biri yeterli. |
| `DevOpsZon:AccessProtection:PathRuleInvalid` | '{Prefix}' geçerli bir yol değil. / ile başlatın; yalnızca harf, rakam ve - . _ ~ ! $ & ' ( ) * + , = : @ kullanın; boş, . veya .. segment ve /.komuta-access kullanmayın. |
| `DevOpsZon:AccessProtection:PathRuleDuplicate` | '{Prefix}' yolu için birden fazla kural var. Her yol için tek kural tutun. |
| `DevOpsZon:AccessProtection:PathRuleBlockExclusive` | '{Prefix}' kuralı yolu engellediği için IP aralığı ya da giriş şartı içeremez. |
| `DevOpsZon:AccessProtection:PathRulePeopleExclusive` | '{Prefix}' kuralı seçilen kişiler için olduğundan IP aralığı içeremez. |
| `DevOpsZon:AccessProtection:PathRuleRequiresNothing` | '{Prefix}' kuralı hiçbir şey istemiyor. IP aralığı ekleyin, Komuta girişini açın ya da yolu engelleyin. |
| `DevOpsZon:AccessProtection:PathRuleGrantInvalid` | '{Prefix}' için seçilen kişiler geçersiz. Her kişiyi bir kez seçin, en fazla 200 kişi seçin ve her zaman aralığını başladıktan sonra bitirin. |
| `DevOpsZon:AccessProtection:PathRuleGrantUnknownShare` | '{Prefix}' için seçilen biriyle servis artık paylaşılmıyor. Önce servisi o kişiyle paylaşın, sonra onu bu yol için seçin. |
| `DevOpsZon:AccessProtection:PathRuleOpenInvalid` | Açık yol ({Prefix}) istekleri Komuta girişi olmadan geçirir. En az bir yöntem seçin; giriş, engelleme ya da kişi eklemeyin; sitenin tamamını veya Komuta'nın kendi giriş yolunu açmayın. |
| `DevOpsZon:AccessProtection:PathRuleUnderOpenPath` | {Prefix} kuralı {Open} açık yolunun altında. Açık yolun altına yalnızca engelleme kuralı konabilir; kuralı ya da açık yolu taşıyın. |
| `DevOpsZon:AccessProtection:PathRulesTooMany` | Bir serviste en fazla {Max} yol kuralı olabilir. |
| `DevOpsZon:AccessProtection:TooManyOpenPaths` | Bir serviste en fazla {Max} açık yol olabilir. |
| `DevOpsZon:AccessProtection:RulesNotAvailable` | Yol kuralları ve 'biri yeterli' birleşimi henüz kullanılamıyor. |
| `DevOpsZon:AccessProtection:ExpiryInPast` | Bitiş zamanı gelecekte olmalı. |
| `DevOpsZon:AccessProtection:ExpiryTooFar` | Bitiş en fazla {MaxDays} gün sonrası olabilir. Daha yakın bir tarih seçin ya da korumayı kalıcı yapın. |
| `DevOpsZon:AccessProtection:InvalidExpiryAction` | Süre sonu davranışı tanınmadı. Servisi kilitli tutmayı ya da herkese açmayı seçin. |
| `DevOpsZon:AccessProtection:ProtectionNotEnabled` | Bu serviste erişim koruması açık değil. Bitiş tarihini değiştirmeden önce korumayı açın. |
| `DevOpsZon:AccessProtection:StaleRevision` | Erişim koruması bu arada değişti (beklenen revizyon {Expected}, güncel {Actual}). Sayfayı yenileyip tekrar deneyin. |
| `DevOpsZon:AccessProtection:InvalidTransition` | Erişim koruması {From} durumundan {To} durumuna geçemez. |
| `DevOpsZon:AccessProtection:NoPublicUrl` | Bu servisin genel URL'i yok; korunacak bir şey yok. Önce genel URL'i açın. |
| `DevOpsZon:AccessProtection:UnsupportedCluster` | Erişim koruması yalnızca Komuta'nın paylaşımlı barındırma kümelerinde çalışan servisler için kullanılabilir. |
| `DevOpsZon:AccessProtection:FeatureDisabled` | Erişim koruması bu platformda henüz açık değil. |
| `DevOpsZon:AccessProtection:NotAvailableForOrganization` | Erişim koruması bu organizasyon için kapatılmış. Açtırmak için Komuta desteğiyle iletişime geçin. |
| `DevOpsZon:AccessProtection:ServiceNotFound` | Servis bulunamadı ya da bu organizasyona ait değil. |
| `DevOpsZon:AccessProtection:IdentityNeedsSignIn` | Ziyaretçi kimliği yalnızca Komuta girişi isteyen bir serviste uygulamaya iletilebilir. Önce girişi açın. |
| `DevOpsZon:AccessProtection:IdentityNotAvailable` | Ziyaretçi kimliğini iletme bu platformda henüz açık değil. |

### Webhook imzaları

| Kod | Mesaj |
|---|---|
| `DevOpsZon:AccessProtection:PathRuleSignatureInvalid` | {Prefix} için imza kontrolü geçerli değil. İmzalı yol açık bir yoldur, yalnız POST, PUT ve PATCH kabul eder ve GitHub, Stripe ya da ağ geçidinin ilettiği bir HMAC başlığı kullanır. |
| `DevOpsZon:AccessProtection:WebhookSignaturesNotAvailable` | Webhook imza kontrolü bu platformda henüz açık değil. |
| `DevOpsZon:AccessProtection:WebhookKeyPathNotSigned` | {Prefix} imza kontrolü olan açık bir yol değil; webhook sırrı tutamaz. |
| `DevOpsZon:AccessProtection:TooManyWebhookKeys` | İmzalı bir yol en fazla {Max} webhook sırrı tutabilir; gönderen yenisini kullanmaya başlayınca eskisini kaldırın. |
| `DevOpsZon:AccessProtection:WebhookKeyNotFound` | Webhook sırrı bulunamadı. |
| `DevOpsZon:AccessProtection:WebhookSecretInvalid` | Webhook sırrı boşluk ve kontrol karakteri içermeyen {Min} ile {Max} bayt arasında bir değerdir. |

### Ülkeler, hız sınırı ve yöntemler

| Kod | Mesaj |
|---|---|
| `DevOpsZon:AccessProtection:CountryInvalid` | {Country} iki harfli bir ISO ülke kodu değil (bilinmeyen ve Tor kodları listelenemez). |
| `DevOpsZon:AccessProtection:CountriesTooMany` | Bir servis en fazla {Max} ülke listeleyebilir. |
| `DevOpsZon:AccessProtection:RateLimitInvalid` | Hız sınırı 1 ile {MaxSeconds} saniye başına {Min} ile {Max} arasında istek olabilir; 0 istek sınırı kapatır. |
| `DevOpsZon:AccessProtection:MethodRuleInvalid` | {Prefix} için yöntem kuralı geçerli değil. Bir yol öneki ve GET, HEAD, POST, PUT, PATCH, DELETE, OPTIONS yöntemlerinden en az birini kullanın; her yol en fazla bir kez yazılabilir. |
| `DevOpsZon:AccessProtection:MethodRulesTooMany` | Bir servisin en fazla {Max} yöntem kuralı olabilir. |
| `DevOpsZon:AccessProtection:MethodRulesNotAvailable` | CORS ön kontrolü ve yöntem kuralları bu platformda henüz kullanılamıyor. |

`MethodRulesNotAvailable`, bu özelliklerin açık olmadığı bir platformda ülke ya da hız sınırı eklerken de dönen yanıttır.

### Erişim önizlemesi

| Kod | Mesaj |
|---|---|
| `DevOpsZon:AccessProtection:ExplainPathInvalid` | {Path} yolu açıklanamıyor. / ile başlayan, kodlanmış karakter, ters eğik çizgi, ';', boş ya da '.' bölüm içermeyen ve /.komuta-access altında olmayan düz bir yol verin. Bu tür yollar daha sıkı okunur ve düz yolun açtığından fazlasını asla açmaz. |
| `DevOpsZon:AccessProtection:ExplainMethodInvalid` | HTTP yöntemi geçersiz. GET ya da POST gibi bir yöntem adı kullanın. |
| `DevOpsZon:AccessProtection:ExplainAddressInvalid` | {Address} geçerli bir IPv4 ya da IPv6 adresi değil. |
| `DevOpsZon:AccessProtection:ExplainVisitorInvalid` | {Kind} ziyaretçisi eksik ya da karışık: üye için kullanıcı, paylaşım için paylaşım kimliği, e-posta ziyaretçisi için geçerli bir adres, token için token kimliği, paylaşım linki için link kimliği gerekir; anonim ziyaretçi bunların hiçbirini taşımaz. |

### Özel ağ

| Kod | Mesaj |
|---|---|
| `DevOpsZon:AccessProtection:MeshExposureEnabled` | Bu servise diğer servisleriniz özel ağ (mesh) üzerinden erişebiliyor. Bu yol erişim kontrolünü atlayarak doğrudan servise gider; açıkken erişim koruması açılamaz. Önce bu servis için özel ağı kapatın; yeni kapattıysanız kaldırılması için bir dakika bekleyin. |
| `DevOpsZon:AccessProtection:MeshBlockedByProtection` | {ServiceName} servisinde erişim koruması açık. Onu özel ağa (mesh) açmak — başka bir servisten ona bağlanmak da bunu yapar — trafiğin erişim kontrolünü atlayarak ona ulaşmasına izin verir. Önce {ServiceName} için erişim korumasını kapatın ve kapanmasının bitmesini bekleyin. |
| `DevOpsZon:AccessProtection:MeshPeerInvalid` | Bu organizasyonun başka servislerini seçin; bir servis kendi listesine eklenemez. |
| `DevOpsZon:AccessProtection:MeshPeersNotAvailable` | Korumalı bir servise özel ağdan gelebilecek servisleri seçmek henüz kullanılamıyor. |
| `DevOpsZon:AccessProtection:TooManyMeshPeers` | Korumalı bir servise özel ağdan en fazla {Max} servis doğrudan gelebilir. |

`MeshExposureEnabled` ve `MeshBlockedByProtection` yalnızca platformda özel ağ ile korumayı birlikte kullanma özelliği kapalıyken görülür.

### Paylaşımlar ve servis token'ları

| Kod | Mesaj |
|---|---|
| `DevOpsZon:AccessProtectionShare:PolicyNotConfigured` | Bu servisi paylaşmadan önce erişim korumasını açın. |
| `DevOpsZon:AccessProtectionShare:MemberNotInOrganization` | Seçilen kişi bu organizasyonun aktif bir üyesi değil. |
| `DevOpsZon:AccessProtectionShare:OrganizationNotLinked` | Yalnızca üyesi olduğunuz organizasyonlarla paylaşabilirsiniz. |
| `DevOpsZon:AccessProtectionShare:LinkedOrganizationIsOwn` | Bu sizin kendi organizasyonunuz. Bunun yerine kendi organizasyonunuzla paylaşın. |
| `DevOpsZon:AccessProtectionShare:OrganizationRequired` | Erişim paylaşımını yönetmek için bir organizasyon açın. |
| `DevOpsZon:AccessProtection:ExternalSharingDisabled` | Organizasyon dışına paylaşım kapalı. |
| `DevOpsZon:AccessProtection:DomainInvalid` | '{Domain}' example.com gibi bir alan adı değil. |
| `DevOpsZon:AccessProtection:DomainOpenToAnyone` | '{Domain}' adresinde herkes e-posta adresi alabilir; bu alan adıyla paylaşım herkesi içeri alır. Bunun yerine tek tek e-posta adresleriyle paylaşın. |
| `DevOpsZon:AccessProtection:DomainSubdomainsNeedVerification` | '{Domain}' alan adının alt alan adları ancak organizasyon alan adını doğruladıktan sonra eklenebilir. |
| `DevOpsZon:AccessProtection:DomainSharesNotAvailable` | Bir alan adındaki herkesle paylaşım henüz kullanılamıyor. |
| `DevOpsZon:AccessProtection:VerifiedDomainExists` | '{Domain}' zaten organizasyonun listesinde. |
| `DevOpsZon:AccessProtection:VerifiedDomainNotFound` | Bu alan adı organizasyonun listesinde yok. |
| `DevOpsZon:AccessProtection:TooManyVerifiedDomains` | Bir organizasyon en fazla {Max} alan adı doğrulayabilir. |
| `DevOpsZon:AccessProtection:ShareGroupsInvalid` | Grup listesi geçerli değil: en fazla 256 karakterlik en fazla 20 grup, yalnız grup gönderen bir sağlayıcıda. |
| `DevOpsZon:AccessProtection:ShareInvalid` | Paylaşım bilgileri '{Kind}' paylaşım türü için geçerli değil. |
| `DevOpsZon:AccessProtection:ShareNotFound` | Paylaşım bulunamadı. |
| `DevOpsZon:AccessProtection:ShareScopeInvalid` | Bu paylaşımın kapsamındaki '{Prefix}' sayfası geçersiz ya da kapsamda 50'den fazla sayfa var. |
| `DevOpsZon:AccessProtection:ShareScopeTooWide` | Bu kişiyi '{Prefix}' için seçmek paylaşımını {Max} sayfadan fazlaya çıkarır. Paylaşımından bazı sayfaları kaldırın ya da kişiyi daha az yolda seçin. |
| `DevOpsZon:AccessProtection:TooManyShares` | Bir serviste en fazla {Max} paylaşım olabilir. Yenisini eklemeden önce bir paylaşımı kaldırın. |
| `DevOpsZon:AccessProtection:ServiceTokenNeedsSignIn` | Servis token'ı yalnızca Komuta girişi gerektiren bir koruma ile çalışır. Önce girişi açın. |
| `DevOpsZon:AccessProtection:ServiceTokenNameInvalid` | Token adı 1 ile {Max} karakter arasında olmalı. |
| `DevOpsZon:AccessProtection:ServiceTokenNameTaken` | Bu serviste '{Name}' adında bir token zaten var. |
| `DevOpsZon:AccessProtection:ServiceTokenNotFound` | Bu servis token'ı bulunamadı; zaten silinmiş olabilir. |
| `DevOpsZon:AccessProtection:ServiceTokenScopeInvalid` | Token'ın açabileceği '{Prefix}' sayfası geçersiz ya da 50'den fazla sayfa seçildi. Sayfaları / ile başlatın; hiçbir sayfa seçmezseniz token tüm siteyi açar. |
| `DevOpsZon:AccessProtection:TooManyServiceTokens` | Bir serviste en fazla {Max} servis token'ı olabilir. Kullanmadığınız birini silin. |

### Paylaşım bağlantıları ve oturumlar

| Kod | Mesaj |
|---|---|
| `DevOpsZon:AccessProtection:ShareLinksNotAvailable` | Paylaşım bağlantıları bu platformda henüz açık değil. |
| `DevOpsZon:AccessProtection:ShareLinkNeedsSignIn` | Önce Komuta girişini açın; paylaşım bağlantısı giriş yapmadan içeri almayı sağlar. |
| `DevOpsZon:AccessProtection:ShareLinkNameInvalid` | Bağlantıya en fazla {Max} karakterlik bir ad verin. |
| `DevOpsZon:AccessProtection:ShareLinkNameTaken` | {Name} adında bir bağlantı zaten var. |
| `DevOpsZon:AccessProtection:ShareLinkEndRequired` | Paylaşım bağlantısının bir bitiş tarihi olmalı. |
| `DevOpsZon:AccessProtection:ShareLinkScopeInvalid` | Bağlantının {Prefix} yolu geçerli değil. |
| `DevOpsZon:AccessProtection:TooManyShareLinks` | Bir servisin en fazla {Max} paylaşım bağlantısı olabilir. |
| `DevOpsZon:AccessProtection:ShareLinkNotFound` | Bu paylaşım bağlantısı artık yok. |
| `DevOpsZon:AccessProtection:SessionLifetimeInvalid` | Oturum süresi olarak 15 dakika, 1 saat, 4 saat, 12 saat, 24 saat veya 7 gün seçin. |
| `DevOpsZon:AccessProtection:SessionsNotAvailable` | Ziyaretçi oturumlarını yönetme bu platformda henüz açık değil. |
| `DevOpsZon:AccessProtection:SessionNotFound` | Bu oturum zaten sona ermiş. |

### Kimlik sağlayıcıları

| Kod | Mesaj |
|---|---|
| `DevOpsZon:AccessProtection:IdentityProvidersNotAvailable` | Ziyaretçilerin kurumunuzun kimlik sağlayıcısıyla girişi henüz kullanılamıyor. |
| `DevOpsZon:AccessProtection:IdentityProviderInvalid` | Kimlik sağlayıcısı ayarı '{Field}' geçerli değil. |
| `DevOpsZon:AccessProtection:IdentityProviderIssuerNotAllowed` | '{Issuer}' adresinde herkes giriş yapabilir; bu sağlayıcı herkesi içeri alır. Kurumunuzun kendi kiracısını ya da alan adını kullanın. |
| `DevOpsZon:AccessProtection:IdentityProviderDiscoveryFailed` | Komuta, kimlik sağlayıcısının '{Issuer}' adresindeki yapılandırmasını okuyamadı ({Reason}). Issuer adresini kontrol edin. |
| `DevOpsZon:AccessProtection:TooManyIdentityProviders` | Bir organizasyonun en fazla {Max} kimlik sağlayıcısı olabilir. |
| `DevOpsZon:AccessProtection:IdentityProviderNotFound` | Kimlik sağlayıcısı bulunamadı. |
| `DevOpsZon:AccessProtection:IdentityProviderInUse` | '{Name}' hâlâ {Count} paylaşımda kullanılıyor. Önce bu paylaşımları kaldırın. |

### Erişim kaydı

| Kod | Mesaj |
|---|---|
| `DevOpsZon:AccessProtection:AccessLogRetentionInvalid` | Erişim kaydını 30, 90 ya da 365 gün saklayabilirsiniz. |
| `DevOpsZon:AccessProtection:AccessLogExportRangeInvalid` | Başlangıcı bitişinden önce olan ve erişim kaydının saklandığı günler içinde kalan bir aralık seçin. |
| `DevOpsZon:AccessProtection:AccessLogExportTooLarge` | Bu aralıkta {Max} kayıttan fazlası var. Daha kısa bir aralık ya da tek bir kayıt türü seçin. |
| `DevOpsZon:AccessProtection:AccessLogExportBusy` | Kuruluşunuzun erişim kaydının başka bir dışa aktarımı hâlâ sürüyor. Birazdan yeniden deneyin. |
| `DevOpsZon:AccessProtection:AccessLogExportUnaudited` | Dışa aktarım denetim kaydına yazılamadığı için yapılmadı. Birazdan yeniden deneyin. |

### Stack'ler

| Kod | Mesaj |
|---|---|
| `DevOpsZon:AccessProtection:DeclaredProtectionRefused` | {Service} için tanımlanan erişim koruması kurulamadı; servis oluşturulmadı. |
| `DevOpsZon:AccessProtection:DeclaredProtectionInvalid` | {Service} için tanımlanan erişim korumasında oturum açma ya da izin listesi olmalı, kuruluşla paylaşım da oturum açma ister; servis oluşturulmadı. |

Manifestin `access` bloğunun doğrulama kodları [Stack Manifestinde Erişim Koruması](stack-manifest-access.md#doğrulama) sayfasındadır.

### Ziyaretçi girişi

| Kod | Mesaj |
|---|---|
| `DevOpsZon:AccessProtection:AccessDenied` | Bu sayfaya erişiminiz yok ya da bağlantı artık geçerli değil. |
| `DevOpsZon:AccessProtection:AccessRateLimited` | Çok fazla giriş denemesi yapıldı. Bir dakika bekleyip tekrar deneyin. |
| `DevOpsZon:AccessProtection:AccessRequestInvalid` | Giriş bağlantısı eksik ya da bozuk. Baştan başlamak için gitmek istediğiniz sayfayı yeniden açın. |
| `DevOpsZon:AccessProtection:SignInNotRequired` | Bu servis şu anda Komuta ile giriş istemiyor. Açamıyorsanız yalnızca belirli ağlardan erişilebiliyor olabilir; servis sahibiyle iletişime geçin. |
| `DevOpsZon:AccessProtection:EmailChallengeInvalid` | Doğrulama kodu hatalı ya da süresi dolmuş. Yeni bir kod isteyip tekrar deneyin. |
| `DevOpsZon:AccessProtection:EmailChallengeRateLimited` | Çok fazla doğrulama kodu istendi. Bir saat bekleyip tekrar deneyin. |
| `DevOpsZon:AccessProtection:EmailVisitorsNotAvailable` | Bu serviste Komuta hesabı olmadan e-posta koduyla giriş kullanılamıyor. |
| `DevOpsZon:AccessProtection:SsoVisitorsNotAvailable` | Bu serviste kurum hesabınızla giriş kullanılamıyor. |
| `DevOpsZon:AccessProtection:SsoSignInFailed` | Kurum hesabınızla giriş tamamlanamadı ({Reason}). Tekrar deneyin ya da servisin sahibine sorun. |
| `DevOpsZon:AccessProtection:SsoSignInExpired` | Bu girişin süresi doldu ya da başka bir tarayıcıda açıldı. Korunan sayfayı yeniden açın. |
| `DevOpsZon:AccessProtection:CodeInvalid` | Giriş kodu geçersiz ya da süresi dolmuş. |
| `DevOpsZon:AccessProtection:CodeAlreadyUsed` | Giriş kodu zaten kullanılmış. |
| `DevOpsZon:AccessProtection:CodeRevoked` | Servisin erişim ayarları değiştiği için giriş kodu artık geçerli değil. |
| `DevOpsZon:AccessProtection:CodeTargetNotFound` | Giriş kodu korunan bir servise ait değil. |
| `DevOpsZon:AccessProtection:CodeStoreUnavailable` | Giriş geçici olarak kullanılamıyor. Kısa süre sonra tekrar deneyin. |

---

## Sık sorulan sorular

**Ziyaretçilerin Komuta hesabı açması gerekiyor mu?**
Her zaman değil. Organizasyonunuzla, bir üyeyle ya da bağlı bir organizasyonla yapılan paylaşımlar Komuta hesabı gerektirir. Servisi ziyaretçinin e-posta adresiyle ya da alan adıyla paylaşırsanız ([e-posta koduyla girer](access-protection-sign-in-sharing.md#komuta-hesabı-olmadan)), organizasyonunuzun kimlik sağlayıcısıyla paylaşırsanız (örneğin Microsoft Entra ID, Google Workspace ya da Okta ile [orada giriş yapar](access-protection-sign-in-sharing.md#organizasyonunuzun-kimlik-sağlayıcısıyla-giriş)) ya da ona bir [paylaşım bağlantısı](access-protection-sign-in-sharing.md#paylaşım-bağlantıları) gönderirseniz hesap gerekmez. Komuta hesabı açmak ücretsizdir ve Google ya da GitHub ile saniyeler sürer. IP izin listesi ise ziyaretçileri hiç giriş yaptırmadan içeri alır.

**Komuta hesabı olmayan birini içeri alabilir miyim?**
Evet: servisi e-posta adresiyle ya da şirketinin alan adıyla paylaşın; posta kutusuna gelen tek kullanımlık kodla girer ([Komuta hesabı olmadan](access-protection-sign-in-sharing.md#komuta-hesabı-olmadan)). Organizasyonunuzun kimlik sağlayıcısında hesabı varsa servisi o sağlayıcıyla paylaşın ([Organizasyonunuzun kimlik sağlayıcısıyla giriş](access-protection-sign-in-sharing.md#organizasyonunuzun-kimlik-sağlayıcısıyla-giriş)). Ya da bir paylaşım bağlantısıyla (**Kişiler → Paylaşım bağlantıları → Bağlantı oluştur**). Bağlantıyı elinde tutan herkes giriş yapmadan, yalnızca bağlantının sayfalarına, bağlantının süresi dolana (konsolda en fazla 90 gün) ya da siz silene kadar girer. Sohbet uygulamalarındaki ve e-postadaki bağlantı önizlemelerinin de açılış sayıldığını ve bağlantının iletildiği herkesin de girebileceğini unutmayın.

**Neden bir API isteğim 302 yerine 401 alıyor?**
Giriş gerektiren bir yola oturumsuz gelen `GET` ve `HEAD` istekleri giriş sayfasına yönlendirilir (`302`); diğer yöntemler, bir programın yönlendirmeyi takip edip bir HTML sayfasını cevap sanmaması için `401` alır. Programlar için [servis token'ı](access-protection-machines.md#servis-tokenları) ya da IP listesi kullanın.

**Tarayıcıdan başka bir siteden servisimin API'sini çağırıyorum, CORS hatası alıyorum.**
Tarayıcının önce gönderdiği `OPTIONS` kontrolü çerez taşımaz; giriş gerektiren bir yolda `401` alır. **Makineler → Yöntemler ve CORS → CORS kontrollerine girişsiz izin ver**'i açın: tarayıcı kontrolü (tek `Origin` ve tek `Access-Control-Request-Method` başlığı) bundan sonra geçer; IP listesi, engelleme kuralları ve yöntem kuralları yine uygulanır. Yalnızca kontrol geçer: asıl istek yine giriş ister (tarayıcının gönderdiği bir oturum, bir servis token'ı ya da onu içeri alan bir IP listesi) ve CORS başlıklarını yine uygulamanız döndürür. Webhook yollarında `OPTIONS` yine seçilemez. Ayarı görmüyorsanız platformunuzda henüz açık değildir; bu tür uç noktaları korumanın dışında tutun ya da isteği sunucu tarafından yapın.

**Korumayı açtım ama hâlâ giriş istemeden açılıyor.**
Durum etiketine bakın: **Hazırlanıyor** sırasında kontrol henüz devrede değildir. **Uygulanıyor**'un ilk saniyelerinde yönlendirmeler henüz yenileniyor olabilir; biraz bekleyip sayfayı yenileyin. **Korunuyor** görünüyorsa tarayıcınız sayfayı önbellekten açmış olabilir; sayfayı yenileyin. Sorun sürüyorsa kartta bir uyarı olup olmadığına bakın.

**Kendimi dışarıda bıraktım.**
Komuta konsolu korumadan etkilenmez. Konsoldan **Kurallar** sekmesine girip IP listesine yeni adresinizi ekleyin (adresinizi **Bu servise erişim kısıtlı** sayfasında görebilirsiniz) ya da **Ayarlar → Şimdi herkese aç** ile korumayı kaldırın.

**Uyuyan bir servise ne olur?**
Koruma uyurken de geçerlidir. Servisi ancak kontrolleri geçen bir ziyaretçi uyandırabilir; giriş yapmamış ya da izinli olmayan bir adresten gelen biri uyandıramaz.

**Bir paylaşımı kaldırdım, kişi hemen çıkar mı?**
Evet, açık oturumları dahil yaklaşık 30 saniye içinde. Serviste tek tek çıkarma devreye girdiyse yalnızca o paylaşımla açılmış oturumlar sona erer; öncesinde (oturum takibi başladıktan sonraki 12 saat 10 dakika boyunca) servisteki diğer ziyaretçiler de bir kez yeniden giriş yapar.

**Tek bir kişiyi oturumdan çıkarabilir miyim?**
Evet: kişinin satırında **Kişiler → Kimler içeride → Çıkar**. Açık oturumları yaklaşık 30 saniye içinde kapanır; başka kimse etkilenmez. Bir paylaşım ona hâlâ erişim veriyorsa hemen yeniden giriş yapabilir; dışarıda kalması gerekiyorsa o paylaşımı da kaldırın. Serviste tek tek çıkarma devreye girene kadar bir kişiyi çıkarmak herkesi çıkarır ve konsol bunu belirtir. Paylaşım bağlantısıyla girenler listede görünmez; erişimlerini bitirmek için bağlantıyı silin.

**Erişim kaydını dışa aktarabilir miyim?**
Evet: **Etkinlik → Dışa aktar** ile CSV ya da JSON olarak, dışa aktarım başına en fazla 50.000 satır. Kayıt varsayılan olarak 30 gün saklanır; organizasyon 90 ya da 365 gün saklamayı seçebilir (**Hesap → Organizasyonlar → Erişim kaydı saklama süresi**).

**Ülkeye, HTTP yöntemine ya da istek sayısına göre kural koyabilir miyim?**
Evet: **Kurallar** sekmesinde **Ülkeler** ve **Hız sınırı**, **Makineler** sekmesinde **Yöntemler ve CORS** altındaki yöntem kuralları. Bu bölümleri görmüyorsanız platformunuzda henüz açık değildir.

**Uygulamam ziyaretçinin kim olduğunu nasıl öğrenir?**
**Ayarlar** sekmesinde **Giriş yapanı uygulamama bildir**'i açın ve `x-komuta-identity` JWT'sini doğrulayın. Bkz. [Bitiş ve Kimlik Bildirme](access-protection-settings.md#giriş-yapanı-uygulamama-bildir).

**Webhook imzalarını Komuta benim için kontrol edebilir mi?**
Evet: webhook yolunu açarken bir **Kenarda imza kontrolü** (GitHub, Stripe ya da başka bir HMAC-SHA256 başlığı) seçin, ardından yolun altına imza sırrını ekleyin. İmzasız istekler uygulamanıza ulaşmadan `401` alır. İmzalı yollar yalnızca `POST`, `PUT` ve `PATCH` ile en fazla 65.535 baytlık gövdeleri kabul eder. Seçeneği görmüyorsanız platformunuzda henüz açık değildir. Bkz. [Kenarda imza kontrolü](access-protection-machines.md#kenarda-imza-kontrolü).

**GitHub gibi bir webhook göndericisi IP aralıklarını değiştirirse ne olur?**
Gönderici adres listesi isteğe bağlıdır; asıl koruma kenarda ya da uygulamanızda yapılan imza doğrulamasıdır. Listeyi kullanıyorsanız göndericinin yayımladığı aralıkları güncel tutun ya da listeyi boş bırakın.

**Koruma açıkken servisimi yeniden dağıtırsam ne olur?**
Koruma etkilenmez; yeni sürüm aynı korumayla yayına girer.

---

## Sözlük

| Terim | Anlamı |
|---|---|
| **Erişim koruması** | Servisin genel adresine kimin ulaşabileceğini Komuta'nın ağ geçidinde denetleyen özellik. |
| **Ağ geçidi (gateway)** | Servisinize internetten gelen trafiğin geçtiği Komuta katmanı; kontrol burada yapılır. |
| **Pod kilidi** | Korunan servisin pod'larının yalnızca ağ geçidinden, kontrolü geçmiş istekleri kabul etmesi. Kontrolün atlanmasını önler. |
| **Komuta girişi** | Ziyaretçinin Komuta hesabıyla (e-posta ve alan adı paylaşımlarında tek kullanımlık e-posta koduyla, kimlik sağlayıcısı paylaşımlarında organizasyonun kimlik sağlayıcısında da) giriş yapması. |
| **Paylaşım** | Bir kişiye, organizasyona, e-posta adresine, bir alan adındaki herkese ya da bir kimlik sağlayıcısıyla giriş yapan herkese servise giriş izni. |
| **Dış paylaşım** | Organizasyon dışına yapılan paylaşım (bağlı organizasyon, e-posta, kimlik sağlayıcısı ya da organizasyonun doğrulamadığı bir alan adı). Organizasyon ayarıyla izin verilir. |
| **Kimlik sağlayıcısı (SSO)** | Komuta hesabı olmayan ziyaretçilerin giriş yapabildiği, organizasyonun kendi giriş servisi (Microsoft Entra ID, Google Workspace, Okta ya da başka bir OIDC sağlayıcısı). |
| **Grup claim'i** | Kimlik sağlayıcısının ziyaretçinin gruplarını listelediği ID token claim'i; bir paylaşımı gruplarla sınırlamak için kullanılır. |
| **Doğrulanmış alan adı** | Organizasyonun bir DNS TXT kaydıyla sahibi olduğunu kanıtladığı alan adı; altındaki paylaşımlar organizasyonun kendi paylaşımı sayılır. |
| **Askıda** | Dış paylaşım kapatıldığı, organizasyon bağı koptuğu ya da alan adı artık doğrulanmış sayılmadığı için geçici olarak çalışmayan paylaşım. |
| **Sayfa sınırı (kapsam)** | Bir paylaşımın, token'ın ya da paylaşım bağlantısının yalnızca belirli yolları açması. |
| **Paylaşım bağlantısı** | Elinde tutan herkesi Komuta hesabı olmadan içeri alan, bitiş tarihi olan bağlantı. |
| **Oturum süresi** | Bir girişin ne kadar sürdüğü; varsayılan 12 saat. |
| **Tek tek çıkarma** | Herkesi çıkarmadan bir kişinin ya da bir paylaşımın oturumlarını kapatma; oturum takibi başladıktan 12 saat 10 dakika sonra devreye girer. |
| **IP izin listesi** | Servise girişsiz ulaşabilecek ya da (ikisi birden seçiliyse) ulaşması gereken genel IP adresleri. |
| **CIDR** | Bir adres aralığının yazımı, örneğin `203.0.113.0/24` (256 adres). |
| **İkisi birden gereksin / Biri yeterli** | IP listesi ile girişin birlikte nasıl değerlendirileceği. |
| **Yol kuralı** | Belirli bir yol ve altındaki yollar için ek koruma ya da engelleme. |
| **Yalnızca seçilen kişiler** | Bir yolu yalnızca seçilen paylaşımlara, isteğe bağlı saat aralığında açan yol kuralı. |
| **Webhook yolu (açık yol)** | Seçilen yöntemlerle gelen istekleri giriş istemeden geçiren yol. Göndericinin imzasını Komuta (imza kontrolüyle) ya da uygulama doğrular. |
| **İmza sırrı** | Webhook göndericisinin isteklerini imzaladığı ortak sır; Komuta bunu şifreli saklar ve imzalı bir webhook yolunda imzaları kontrol etmek için kullanır. |
| **Servis token'ı** | Programların başlık olarak gönderdiği, giriş yerine geçen gizli anahtar. |
| **Özel ağ (mesh)** | Kümeleriniz arasında genel internete çıkmayan bağlantı; ağ geçidinden geçmez. |
| **Yöntem kuralı** | Bir yolun kabul ettiği HTTP yöntemleri; diğer yöntemler `405` alır. |
| **CORS kontrolü (ön uçuş)** | Tarayıcının başka bir siteyi çağırmadan önce gönderdiği `OPTIONS` isteği; girişsiz geçirilebilir. |
| **Ülke listesi** | Ziyaretçilerin gelebileceği ülkeler; girişten önce kontrol edilir. |
| **Hız sınırı** | Bir adresin bir zaman aralığında gönderebileceği istek sayısı; aşılınca `429`. |
| **Erişim önizlemesi** | Kayıtlı kurallarla kimin nereye girebildiğini gösteren, hiçbir şeyi değiştirmeyen araç. |
| **Erişim kaydı** | Girişlerin, sayfa görüntülemelerinin ve retlerin 30, 90 ya da 365 gün saklanan kaydı. |
| **Yeni servisleri koru** | Her yeni genel servisi organizasyon için Komuta girişiyle korunarak başlatan organizasyon ayarı. |
| **Koruma bitişi** | Korumanın sona ereceği zaman ve o zaman ne olacağı. |
| **Kimlik bildirme** | Giriş yapan ziyaretçinin kimliğinin uygulamaya başlıklarla iletilmesi. |
| **JWT / JWKS** | İmzalı kimlik jetonu / imzayı doğrulamak için yayımlanan açık anahtar listesi. |
| **Cloudflare** | Komuta adreslerinin ve özel alan adlarının önündeki ağ; ziyaretçi adresini güvenilir şekilde bildirir. |
| **DNS only (gri bulut)** | Cloudflare'de bir kaydın proxy'lenmeden yalnızca DNS olarak çalışması. Komuta'ya eklenen özel alan adının kendi Cloudflare hesabınızdaki kaydı bu şekilde olmalıdır. |
