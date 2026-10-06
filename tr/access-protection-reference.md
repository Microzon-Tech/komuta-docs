# Erişim Koruması: Başvuru

Bu sayfa erişim korumasının kesin değerlerini tek yerde toplar: sınırlar, durumlar, ziyaretçiye dönen yanıtlar, başlıklar, hata mesajları, sık sorulan sorular ve terimler sözlüğü. Özelliklerin ne işe yaradığı ve nasıl kullanıldığı ilgili rehber sayfalarında anlatılır; burada yalnızca "tam olarak ne" sorusunun cevabı vardır.

Erişim koruması rehberleri:

- [Erişim Koruması](service-access-protection.md) — genel bakış, açma ve kapatma, durumlar.
- [Giriş ve Paylaşım](access-protection-sign-in-sharing.md) — Komuta girişi, **Kişiler** sekmesi, ziyaretçinin gördükleri.
- [Kurallar](access-protection-rules.md) — IP izin listesi, yol kuralları, erişim önizlemesi.
- [Makineler ve Özel Ağ](access-protection-machines.md) — webhook yolları, servis token'ları, özel ağ.
- [Erişim Kaydı](access-protection-activity.md) — **Etkinlik** sekmesi.
- [Bitiş ve Kimlik Bildirme](access-protection-settings.md) — **Ayarlar** sekmesi.

---

## Sınırlar

| Konu | Sınır |
|---|---|
| IP izin listesi (site, her yol kuralı, her webhook yolu için ayrı) | En fazla 100 girdi, toplam 4600 karakter; yalnızca genel adresler; `/0` yok; host bitleri sıfır |
| Yol kuralı | Serviste en fazla 50 (webhook yolları dahil); her yol için tek kural |
| Yol uzunluğu | 2–256 karakter; `/` ile başlar; küçük harf, rakam ve `- . _ ~ ! $ & ' ( ) * + , = : @ /` |
| Webhook (açık) yolu | En fazla 10; yöntemler `GET`, `HEAD`, `POST`, `PUT`, `PATCH`, `DELETE` |
| Paylaşım | Serviste en fazla 200 |
| Paylaşımın sayfa sınırı | En fazla 50 sayfa (seçildiği "yalnızca seçilen kişiler" yolları dahil) |
| Paylaşım bitişi | Gelecekte olmalı; üst sınır yok |
| "Yalnızca seçilen kişiler" kuralı | Kural başına en fazla 200 kişi |
| Servis token'ı | Serviste en fazla 20; ad 1–64 karakter; bitiş en fazla 365 gün; en fazla 50 sayfa |
| Özel ağdan gelebilecek servisler | En fazla 50; aynı organizasyon |
| Koruma bitişi | Gelecekte, en fazla 365 gün sonra |
| Korunan adres (host) sayısı | Servis başına en fazla 50 |
| Ziyaretçi oturumu | En fazla 12 saat; paylaşımın ve "herkese açılsın" bitişinin ötesine geçmez |
| Giriş bağlantısı | 15 dakika (konsoldaki giriş sayfası) |
| Giriş denemesi | Kullanıcı başına dakikada 30 |
| E-posta doğrulama kodu | 8 hane, 10 dakika geçerli, 5 hatalı denemede geçersiz |
| E-posta kodu isteme | Kullanıcı başına saatte 20; kullanıcı + adres başına saatte 5; aynı tarayıcıdan adres başına saatte 3 (30/60/120 sn bekleme) |
| İstek yolu (yol kuralı ya da sayfa sınırı olan serviste) | En fazla 1024 bayt; aşarsa `400` |
| Erişim kaydı | 30 gün saklanır; 15 sn'lik paketler; servis başına saatte 500 satır (girişler hariç); sayfa başına 50 kayıt |
| Paylaşımdan sonra erişimin kesilmesi | Yaklaşık 30 saniye |
| Kimlik JWT'si | 5 dakika geçerli; `nbf` = `iat` − 30 sn |

---

## Koruma durumları

| API değeri | Arayüz (TR) | Arayüz (EN) | Kontrol devrede |
|---|---|---|---|
| `Disabled` | Kapalı | Off | Hayır |
| `Preparing` | Hazırlanıyor | Preparing | Hayır |
| `Enforcing` | Uygulanıyor | Applying | Evet |
| `Protected` | Korunuyor | Protected | Evet |
| `Disabling` | Kapatılıyor | Turning off | Son adıma kadar |

`Enforcing` sırasındaki adımlar (`RouteFilter` → `PodTokenLock` → `CachePurge`): ağ geçidi kontrolünün tüm rotalara eklenmesi, pod kilidi, Cloudflare önbelleğinin temizlenmesi. Arayüz adımların adını göstermez, "{toplam} adımın {n} tanesi tamamlandı" der (5 adım: hazırlık, üç uygulama adımı, doğrulama). `Disabling` ters sırada çalışır: önce pod kilidi, sonra ağ geçidi kontrolü kalkar.

Bitiş davranışı (`ExpiryAction`): `KeepLocked` = **Sonra kilitli kalsın** (varsayılan), `OpenToEveryone` = **Sonra herkese açılsın**.

Paylaşım türleri (`ShareKind`): `OwnOrganization` = **Organizasyonunuz**, `Member` = **Bir üye**, `LinkedOrganization` = **Bağlı bir organizasyon**, `Email` = **Bir e-posta adresi**.

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
| **Tamamen engelle** kuralı | `403` | `access denied` |
| Geçersiz servis token'ı | `401` | `invalid service token`, `WWW-Authenticate: KomutaServiceToken realm="komuta"` |
| Geçerli token, kapsam dışı sayfa | `403` | `this service token cannot open this path` |
| Okunamayan ya da 1024 bayttan uzun yol (yol kuralı/sayfa sınırı olan serviste) | `400` | `bad request` |
| Aynı Komuta çerezi iki kez gönderildi | `400` | `duplicate access cookie` |
| Koruma bilgisi geçici olarak alınamıyor | `503` | `access policy unavailable` |
| Giriş geçici olarak kullanılamıyor | `503` | `sign-in unavailable` |

Tüm ret yanıtları `Cache-Control: no-store` taşır. HTML sayfalar ziyaretçinin tarayıcı diline göre Türkçe ya da İngilizce görünür ve dış kaynak yüklemez.

---

## Başlıklar, çerezler ve ayrılmış yollar

| Ad | Yön | Açıklama |
|---|---|---|
| `x-komuta-service-token` | İstemci → Komuta | Servis token'ı (`kst_<32 onaltılık>_<43 karakter>`). Komuta kontrol ettikten sonra siler; uygulamaya ulaşmaz. |
| `x-komuta-user-email` | Komuta → uygulama | Giriş yapan ziyaretçinin e-postası (kimlik bildirme açıkken). |
| `x-komuta-user-id` | Komuta → uygulama | Giriş yapan ziyaretçinin Komuta kullanıcı kimliği (kimlik bildirme açıkken). |
| `x-komuta-identity` | Komuta → uygulama | ES256 imzalı kimlik JWT'si (kimlik bildirme açıkken). |
| `x-komuta-access` | Komuta → uygulama | Pod kilidi için servise özel gizli değer. Kullanmayın, loglamayın. |
| `Cache-Control: private, no-store` | Komuta → ziyaretçi | Korunan servisin tüm yanıtlarına yazılır. |
| `__Host-komuta_access` | Çerez | Ziyaretçi oturumu; yalnızca servisin o adresi için; en fazla 12 saat. Uygulamaya iletilmez. |
| `__Host-komuta_state` | Çerez | Giriş sürerken kullanılan kısa ömürlü çerez (10 dakika). Uygulamaya iletilmez. |
| `/.komuta-access/callback` | Yol | Girişten dönüş adresi. `/.komuta-access` ile başlayan yollar Komuta'ya ayrılmıştır; kural ya da webhook yolu tanımlanamaz. |

Kimlik bildirme başlıkları ziyaretçi tarafından taklit edilemez: Komuta'dan geçen her izinli istekte silinip yeniden yazılır. JWT alanları ve doğrulama kuralları için bkz. [Bitiş ve Kimlik Bildirme](access-protection-settings.md#kimlik-kanıtı-jwt).

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
| `login_required` | Ret | Giriş yapması gerekiyordu |
| `overflow` | Toplam | Bu saatteki diğer ziyaretler, birlikte gruplandı |

---

## Hata mesajları

Konsolda ya da API'de bir işlem reddedildiğinde gösterilen mesajlar. Süslü parantez içindeki değerler işlem sırasında doldurulur.

### Koruma ayarları

| Kod | Mesaj |
|---|---|
| `DevOpsZon:AccessProtection:NothingProtected` | En az bir IP aralığı ekleyin, Komuta girişini açın ya da bir yol kuralı ekleyin; aksi halde servis korunmaz. |
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

### Ziyaretçi girişi

| Kod | Mesaj |
|---|---|
| `DevOpsZon:AccessProtection:AccessDenied` | Bu sayfaya erişiminiz yok ya da bağlantı artık geçerli değil. |
| `DevOpsZon:AccessProtection:AccessRateLimited` | Çok fazla giriş denemesi yapıldı. Bir dakika bekleyip tekrar deneyin. |
| `DevOpsZon:AccessProtection:AccessRequestInvalid` | Giriş bağlantısı eksik ya da bozuk. Baştan başlamak için gitmek istediğiniz sayfayı yeniden açın. |
| `DevOpsZon:AccessProtection:SignInNotRequired` | Bu servis şu anda Komuta ile giriş istemiyor. Açamıyorsanız yalnızca belirli ağlardan erişilebiliyor olabilir; servis sahibiyle iletişime geçin. |
| `DevOpsZon:AccessProtection:EmailChallengeInvalid` | Doğrulama kodu hatalı ya da süresi dolmuş. Yeni bir kod isteyip tekrar deneyin. |
| `DevOpsZon:AccessProtection:EmailChallengeRateLimited` | Çok fazla doğrulama kodu istendi. Bir saat bekleyip tekrar deneyin. |
| `DevOpsZon:AccessProtection:CodeInvalid` | Giriş kodu geçersiz ya da süresi dolmuş. |
| `DevOpsZon:AccessProtection:CodeAlreadyUsed` | Giriş kodu zaten kullanılmış. |
| `DevOpsZon:AccessProtection:CodeRevoked` | Servisin erişim ayarları değiştiği için giriş kodu artık geçerli değil. |
| `DevOpsZon:AccessProtection:CodeTargetNotFound` | Giriş kodu korunan bir servise ait değil. |
| `DevOpsZon:AccessProtection:CodeStoreUnavailable` | Giriş geçici olarak kullanılamıyor. Kısa süre sonra tekrar deneyin. |

---

## Sık sorulan sorular

**Ziyaretçilerin Komuta hesabı açması gerekiyor mu?**
Komuta girişi kullanıyorsanız evet. Hesap açmak ücretsizdir ve Google ya da GitHub ile saniyeler sürer. Hesap açtırmak istemiyorsanız IP izin listesi kullanın. Kendi kimlik sağlayıcınızla (SSO) giriş şu an desteklenmez.

**Neden bir API isteğim 302 yerine 401 alıyor?**
Giriş gerektiren bir yola oturumsuz gelen `GET` ve `HEAD` istekleri giriş sayfasına yönlendirilir (`302`); diğer yöntemler, bir programın yönlendirmeyi takip edip bir HTML sayfasını cevap sanmaması için `401` alır. Programlar için [servis token'ı](access-protection-machines.md#servis-tokenları) ya da IP listesi kullanın.

**Tarayıcıdan başka bir siteden servisimin API'sini çağırıyorum, CORS hatası alıyorum.**
Tarayıcının gönderdiği `OPTIONS` ön uçuş isteği çerez taşımaz; giriş gerektiren bir yolda `401` alır. Webhook yollarında da `OPTIONS` seçilemez. Başka bir siteden tarayıcıyla çağrılması gereken uç noktaları korumanın dışında tutmak için o yolları korumasız bırakın (site korumasını kapatıp yalnızca diğer yolları yol kurallarıyla koruyarak) ya da isteği sunucu tarafından yapın.

**Korumayı açtım ama hâlâ giriş istemeden açılıyor.**
Durum etiketine bakın: **Hazırlanıyor** sırasında kontrol henüz devrede değildir. **Uygulanıyor** ya da **Korunuyor** görünüyorsa tarayıcınızda önbellekten açılmış olabilir; sayfayı yenileyin. Sorun sürüyorsa kartta bir uyarı olup olmadığına bakın.

**Kendimi dışarıda bıraktım.**
Komuta konsolu korumadan etkilenmez. Konsoldan **Kurallar** sekmesine girip IP listesine yeni adresinizi ekleyin (adresinizi **Erişim kısıtlı** sayfasında görebilirsiniz) ya da **Ayarlar → Şimdi herkese aç** ile korumayı kaldırın.

**Uyuyan bir servis ne olur?**
Koruma uyurken de geçerlidir. Servisi ancak kontrolleri geçen bir ziyaretçi uyandırabilir; giriş yapmamış ya da izinli olmayan bir adresten gelen biri uyandıramaz.

**Bir paylaşımı kaldırdım, kişi hemen çıkar mı?**
Evet, açık oturumları dahil yaklaşık 30 saniye içinde.

**Bir kişiyi tek tek oturumdan çıkarabilir miyim?**
Kişi bazında oturum kapatma yoktur. Kişinin paylaşımını kaldırmak ya da bitişini öne çekmek o paylaşımla açılmış oturumları sonlandırır. Organizasyon paylaşımıyla giren tek bir kişiyi çıkarmak için o kişiyi organizasyondan çıkarın.

**Erişim kaydını dışa aktarabilir miyim?**
Şu an hayır. Kayıt konsolda 30 gün görünür.

**Ülkeye, HTTP yöntemine ya da istek sayısına göre kural koyabilir miyim?**
Hayır. Kurallar adres, giriş ve yola göredir. Yöntem seçimi yalnızca webhook yollarında vardır.

**Uygulamam ziyaretçinin kim olduğunu nasıl öğrenir?**
**Ayarlar** sekmesinde **Giriş yapanı uygulamama bildir**'i açın ve `x-komuta-identity` JWT'sini doğrulayın. Bkz. [Bitiş ve Kimlik Bildirme](access-protection-settings.md#giriş-yapanı-uygulamama-bildir).

**Webhook göndericisi GitHub'ın IP aralıklarını değiştirirse?**
Gönderici adres listesi isteğe bağlıdır; asıl koruma uygulamanızın imza doğrulamasıdır. Listeyi kullanıyorsanız göndericinin yayımladığı aralıkları güncel tutun ya da listeyi boş bırakın.

**Koruma açıkken servisimi yeniden dağıtırsam ne olur?**
Koruma etkilenmez; yeni sürüm aynı korumayla yayına girer.

---

## Sözlük

| Terim | Anlamı |
|---|---|
| **Erişim koruması** | Servisin genel adresine kimin ulaşabileceğini Komuta'nın ağ geçidinde denetleyen özellik. |
| **Ağ geçidi (gateway)** | Servisinize internetten gelen trafiğin geçtiği Komuta katmanı; kontrol burada yapılır. |
| **Pod kilidi** | Korunan servisin pod'larının yalnızca ağ geçidinden, kontrolü geçmiş istekleri kabul etmesi. Kontrolün atlanmasını önler. |
| **Komuta girişi** | Ziyaretçinin Komuta hesabıyla giriş yapması. |
| **Paylaşım** | Bir kişiye, organizasyona ya da e-posta adresine servise giriş izni. |
| **Dış paylaşım** | Organizasyon dışına yapılan paylaşım (bağlı organizasyon ya da e-posta). Organizasyon ayarıyla izin verilir. |
| **Askıda** | Dış paylaşım kapatıldığı ya da organizasyon bağı koptuğu için geçici olarak çalışmayan paylaşım. |
| **Sayfa sınırı (kapsam)** | Bir paylaşımın ya da token'ın yalnızca belirli yolları açması. |
| **IP izin listesi** | Servise girişsiz ulaşabilecek ya da (ikisi birden seçiliyse) ulaşması gereken genel IP adresleri. |
| **CIDR** | Bir adres aralığının yazımı, örneğin `203.0.113.0/24` (256 adres). |
| **İkisi birden gereksin / Biri yeterli** | IP listesi ile girişin birlikte nasıl değerlendirileceği. |
| **Yol kuralı** | Belirli bir yol ve altındaki yollar için ek koruma ya da engelleme. |
| **Yalnızca seçilen kişiler** | Bir yolu yalnızca seçilen paylaşımlara, isteğe bağlı saat aralığında açan yol kuralı. |
| **Webhook yolu (açık yol)** | Seçilen yöntemlerle gelen istekleri giriş istemeden geçiren yol. Göndericinin imzasını uygulama doğrular. |
| **Servis token'ı** | Programların başlık olarak gönderdiği, giriş yerine geçen gizli anahtar. |
| **Özel ağ (mesh)** | Kümeleriniz arasında genel internete çıkmayan bağlantı; ağ geçidinden geçmez. |
| **Erişim önizlemesi** | Kayıtlı kurallarla kimin nereye girebildiğini gösteren, hiçbir şeyi değiştirmeyen araç. |
| **Erişim kaydı** | Girişlerin, sayfa görüntülemelerinin ve retlerin 30 günlük kaydı. |
| **Koruma bitişi** | Korumanın sona ereceği zaman ve o zaman ne olacağı. |
| **Kimlik bildirme** | Giriş yapan ziyaretçinin kimliğinin uygulamaya başlıklarla iletilmesi. |
| **JWT / JWKS** | İmzalı kimlik jetonu / imzayı doğrulamak için yayımlanan açık anahtar listesi. |
| **Cloudflare** | Komuta adreslerinin ve özel alan adlarının önündeki ağ; ziyaretçi adresini güvenilir şekilde bildirir. |
| **DNS only (gri bulut)** | Cloudflare'de bir kaydın proxy'lenmeden yalnızca DNS olarak çalışması. Komuta'ya eklenen özel alan adının kendi Cloudflare hesabınızdaki kaydı bu şekilde olmalıdır. |
