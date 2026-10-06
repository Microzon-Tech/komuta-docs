# Erişim Koruması

Erişim koruması, servisinizin internetteki adresini kimlerin açabileceğini belirler. Geliştirme aşamasındaki bir uygulamayı yalnızca ekibinize, bir müşterinize ya da ofis ağınıza açmak; bir yönetim panelini kapatmak ya da bir sayfayı belirli kişilere belirli saatlerde açmak için kullanılır. Uygulamanıza ayrı bir giriş ekranı yazmanız ya da kodunuzu değiştirmeniz gerekmez.

Erişim koruması tüm planlara dahildir.

---

## Bir bakışta

Erişim korumasıyla şunları yapabilirsiniz:

- **Komuta ile giriş isteyin.** Ziyaretçiler Komuta hesabıyla giriş yapar; yalnızca servisi paylaştığınız kişiler, organizasyonlar ve e-posta adresleri içeri girer.
- **IP adresiyle sınırlayın.** Servis yalnızca belirlediğiniz ağlardan (örneğin ofisinizden) açılır. Giriş ile birlikte "ikisi birden" ya da "biri yeterli" olarak kullanılabilir.
- **Yol yol koruyun.** Site açıkken `/admin`'i yalnızca giriş yapanlara açın, `/internal`'ı tamamen kapatın, `/raporlar`'ı yalnızca seçtiğiniz kişilere belirli saatlerde açın.
- **Makinelere izin verin.** GitHub, Stripe gibi webhook göndericileri için giriş istemeyen yollar açın; CI ve izleme araçlarına servis token'ı verin.
- **Süre koyun.** Korumayı belirli bir tarihte bitirin; bittiğinde kilitli kalsın ya da herkese açılsın.
- **Kimin girdiğini görün.** Erişim kaydı girişleri, açılan sayfaları ve geri çevrilen ziyaretçileri gösterir.
- **Uygulamanıza kim olduğunu söyleyin.** Giriş yapan ziyaretçinin e-postası ve imzalı bir kimlik kanıtı uygulamanıza başlık olarak iletilebilir.

---

## Nasıl çalışır

Korumalı bir servise gelen her istek, uygulamanıza ulaşmadan önce Komuta'nın ağ geçidinde bir kontrolden geçer:

1. Ziyaretçi servisinizin adresini açar.
2. Komuta isteği kurallarınıza göre değerlendirir: IP adresi listede mi, ziyaretçinin bu servis için geçerli bir oturumu var mı, açılan yol için özel bir kural var mı?
3. Kurallar sağlanıyorsa istek uygulamanıza iletilir. Giriş gerekiyorsa ziyaretçi Komuta giriş sayfasına yönlendirilir. İzin yoksa ziyaretçi açıklayıcı bir sayfa görür ve istek uygulamanıza hiç ulaşmaz.

Bu kontrolün etrafından dolaşılamaz. Koruma açıldığında Komuta servisinizin pod'larını da kilitler: pod'lar yalnızca ağ geçidinden, kontrolü geçmiş olarak gelen istekleri kabul eder. Biri ağ geçidini atlayıp doğrudan pod'a ulaşmaya çalışırsa istek reddedilir. (Aynı kümedeki kendi servisleriniz pod'larınıza eskisi gibi ulaşmaya devam eder.)

Koruma servisinizin tüm genel adresleri için birlikte geçerlidir: Komuta'nın verdiği `*.komuta.app` adresi, **Alan adları** altında eklediğiniz özel alan adları ve mavi-yeşil dağıtımın önizleme adresi.

---

## Hızlı başlangıç

Aşağıdaki tarifler en sık kullanılan kurulumlardır. Tüm ayarlar **Servis Detay → Yapılandırma → Erişim ve portlar** sayfasındadır.

### Yalnızca ekibim açabilsin

1. **Kurallar** sekmesinde **Erişim koruması** anahtarını açın. **Komuta girişi iste** seçili gelir.
2. **Korumayı uygula**'ya basın. Siz otomatik olarak paylaşım listesine eklenirsiniz.
3. **Kişiler** sekmesinde **Paylaşım ekle → Organizasyonunuz** seçin.

Organizasyonunuzun tüm üyeleri Komuta hesaplarıyla giriş yapıp servisi açabilir; diğer herkes giriş sayfasında kalır.

### Bir müşterime göstereyim

1. Organizasyonunuzun dış paylaşıma izin verdiğinden emin olun: **Hesap → Organizasyonlar → Dış paylaşıma izin ver** (varsayılan olarak kapalıdır).
2. Korumayı yukarıdaki gibi açın.
3. **Kişiler** sekmesinde **Paylaşım ekle → Bir e-posta adresi** seçin ve müşterinin adresini yazın. Bir **Erişim bitişi** vermek iyi olur.

Müşteri herhangi bir Komuta hesabıyla (Google ya da GitHub ile saniyeler içinde açılabilir) giriş yapar, ardından adresine gelen 8 haneli kodla adresini doğrular.

### Yalnızca ofis ağından açılsın

1. **Kurallar** sekmesinde korumayı açın, **Komuta girişi iste**'yi kapatın.
2. **IP izin listesi**'ne ofisinizin genel IP adresini ya da aralığını yazın (bulunduğunuz yerden bağlanıyorsanız **Kendi IP'mi ekle**).
3. **Korumayı uygula**.

Listedeki adreslerden gelenler giriş yapmadan girer; diğer herkes **Erişim kısıtlı** sayfasını görür.

### Ofisten girişsiz, dışarıdan girişle

Hem **Komuta girişi iste**'yi hem **IP izin listesi**'ni açın ve "Giriş ve IP izin listesi nasıl birleşsin?" sorusunda **Biri yeterli**'yi seçin.

### Site açık kalsın, yalnızca yönetim paneli korunsun

1. Korumayı açın, **Komuta girişi iste**'yi kapatın.
2. **Yol kuralları** bölümünde **Yol kuralı ekle**: yol `/admin`, koruma **Komuta girişi**.
3. **Korumayı uygula**, ardından **Kişiler** sekmesinden paneli kullanacak kişileri ekleyin.

### GitHub webhook'u da gelebilsin

Korumalı bir serviste **Makineler** sekmesinde **Webhook'lar → Yol aç** ile `/webhooks/github` yolunu `POST` için açın ve uygulamanızda GitHub'ın imzasını (`X-Hub-Signature-256`) doğrulayın.

---

## Ekran: Erişim ve portlar

Erişim koruması **Servis Detay → Yapılandırma → Erişim ve portlar** sayfasında, sekmeler halinde yönetilir:

| Sekme | İçerik | Rehber |
|---|---|---|
| **Genel bakış** | Servisin şu anki erişim durumunun özeti, trafiğin servise nasıl ulaştığı ve son erişim olayları. | Bu sayfa |
| **Kurallar** | Korumayı açma anahtarı, durum, **Kimler girebilir** (Komuta girişi, IP izin listesi), **Yol kuralları**, **Erişim önizlemesi**. | [Kurallar](access-protection-rules.md) |
| **Kişiler** | Servisin kimlerle paylaşıldığı. | [Giriş ve Paylaşım](access-protection-sign-in-sharing.md) |
| **Makineler** | Servis token'ları, webhook yolları ve özel ağdan gelebilecek servisler. | [Makineler ve Özel Ağ](access-protection-machines.md) |
| **Etkinlik** | Erişim kaydı. | [Erişim Kaydı](access-protection-activity.md) |
| **Ağ** | Genel adresler (genel URL), portlar ve özel ağ (mesh). | [Makineler ve Özel Ağ](access-protection-machines.md#özel-ağdan-doğrudan-gelebilecek-servisler) |
| **Ayarlar** | Koruma bitişi, uygulamaya kimlik bildirme ve **Şimdi herkese aç**. | [Bitiş ve Kimlik Bildirme](access-protection-settings.md) |

Sekme adresi URL'de `?tab=` ile tutulur (`rules`, `people`, `machines`, `activity`, `network`, `settings`); bir sekmenin bağlantısını paylaşabilirsiniz.

**Genel bakış** sekmesindeki **Genel erişim** kartı durumu tek cümleyle söyler: **Herkese açık**, **Koruma uygulanıyor**, **Korunuyor**, **Belirli yollar korunuyor**, **Koruma kaldırılıyor**, **Koruma tamamlanamıyor**, **İnternete açık değil**. Altında etkin ayarlar kısa etiketlerle sıralanır (ör. "Komuta girişi", "3 izinli adres", "2 yol kuralı", "5 paylaşım", "… tarihine kadar"). Servis herkese açıkken **Erişim korumasını kur** düğmesi sizi **Kurallar** sekmesine götürür ve korumayı açmaya başlatır. Bir sorun varsa başlığın yanında **Dikkat gerekiyor** etiketi görünür.

Organizasyonunuzda erişim koruması kapatılmışsa ya da servisin korumasını görme yetkiniz yoksa yalnızca **Genel bakış** ve **Ağ** sekmeleri görünür.

---

## Korumayı açma

1. **Kurallar** sekmesinde **Erişim koruması** kartının başlığındaki anahtarı açın. Bu henüz bir şey kaydetmez; **Komuta girişi iste** seçili olarak ayarlar açılır.
2. İstediğiniz kontrolleri seçin: Komuta girişi, IP izin listesi, yol kuralları ya da bunların birleşimi (bkz. [Kurallar](access-protection-rules.md)). En az biri gerekir.
3. **Korumayı uygula**'ya basın ("Erişim koruması uygulanıyor").
4. Komuta girişi seçtiyseniz **Kişiler** sekmesinden servisi paylaşın. Girişi ilk kez zorunlu kıldığınızda siz otomatik olarak eklenirsiniz.

Vazgeçmek için **İptal**'e basın ya da anahtarı kapatın; kaydedilmemiş taslak silinir.

Korumayı servis genel bakışındaki **Servis koruması** kartından da adım adım kurabilirsiniz (bkz. [Servis koruması sihirbazı](#servis-koruması-sihirbazı)).

---

## Ne kadar sürer

- **İlk açılış** genellikle bir iki dakika sürer. Komuta servisinizin yönlendirme ayarlarını yeniler ve pod kilidini uygular; bu bir dağıtım olarak görünebilir ama yeni bir build yapılmaz.
- Koruma **Uygulanıyor** durumuna geçtiği anda kontrol devreye girer. **Hazırlanıyor** sırasında servis hâlâ eski haliyle (herkese açık) çalışır.
- **Açık bir korumada** kural, IP listesi ve paylaşım değişiklikleri genellikle birkaç saniye ile yarım dakika arasında geçerli olur. Bu sırada kart "Son değişikliğiniz uygulanıyor." der.
- **Kapatma** da birkaç saniye ile bir iki dakika arasında tamamlanır; koruma **Kapatılıyor** durumu bitene kadar sürer.

---

## Durumlar

**Kurallar** sekmesindeki kartın başlığında bir durum etiketi bulunur:

| Etiket | Anlamı | Kontrol devrede mi? |
|---|---|---|
| **Kapalı** | Koruma yok; servis herkese açık. | Hayır |
| **Hazırlanıyor** | Ayarlarınız ağ geçidine yazılıyor. | Hayır |
| **Uygulanıyor** | Kontrol devrede; pod kilidi ve önbellek temizliği tamamlanıyor. | Evet |
| **Korunuyor** | Koruma tamamen yerinde. | Evet |
| **Kapatılıyor** | Koruma kaldırılıyor; bitince URL'e sahip herkes servisi açabilir. | Son adıma kadar evet |

Koruma açılırken kartta "{toplam} adımın {n} tanesi tamamlandı" yazan 5 adımlı bir ilerleme çubuğu görünür. Adımlar sırasıyla: hazırlık, ağ geçidi kontrolünün etkinleşmesi, pod kilidi, önbellek temizliği ve korumanın doğrulanması. Komuta son adımda korumayı kendisi dener: giriş yapmamış bir isteğin gerçekten reddedildiğini ve korunan yanıtların önbelleğe alınmadığını doğrular.

Aynı korumayı başka bir sekmede ya da bir ekip arkadaşınız sizden önce değiştirdiyse kart en güncel ayarları yükler ve "Erişim koruması başka bir yerde değiştirildi. En güncel ayarlar yüklendi — değişikliğinizi yeniden yapın." der. Böylece kimse farkında olmadan bir başkasının değişikliğini ezemez.

---

## Uyarılar ve ne yapmalı

Koruma tamamlanamazsa ya da bir sorun oluşursa **Kurallar** sekmesinde (ve diğer sekmelerin üstünde) bir uyarı kutusu, altında da teknik kod ("Kod: …") görünür. Komuta geçici bekleme durumlarını 15 dakika boyunca hata saymaz ve her durumda otomatik olarak yeniden dener.

### Sizin düzeltmeniz gerekenler

Kutu başlığı: "Bu düzeltilene kadar koruma tamamlanamaz."

| Kod | Mesaj | Ne yapmalı |
|---|---|---|
| `service_not_deployed` | Bu servis henüz dağıtılmadı. Koruma, ilk başarılı dağıtımdan sonra uygulanır. | Servisi dağıtın. |
| `no_hosts` | Bu servisin korunabilecek genel bir adresi yok. Önce genel URL'ini açın. | **Ağ** sekmesinden **Genel URL**'i açın. |
| `too_many_hosts` | Bu servisin domain sayısı tek bir koruma politikasının kapsayabileceğinden fazla. | Kullanmadığınız özel alan adlarını kaldırın (sınır: 50 adres). |
| `host_invalid` | Bu servisin yönlendirme host adlarından biri geçerli bir domain adı değil. | Alan adını düzeltin ya da kaldırın. |
| `host_not_owned` | Bu servis, kendi doğrulanmış domainlerinden olmayan bir domaini yönlendiriyor. | Alan adını bu servis için doğrulayın ya da kaldırın. |
| `host_claimed_elsewhere` | Bu servisin domainlerinden biri zaten başka bir servis tarafından korunuyor. | Alan adını iki servisten birinden kaldırın. |
| `host_not_public` | Bu servisin domainlerinden biri özel ya da ayrılmış bir adresi işaret ediyor. | DNS kaydını Komuta'ya yönlendirin ya da kaldırın. |
| `host_dns_unresolved` | Bu servisin domainlerinden biri DNS'te henüz çözümlenmiyor. | DNS kaydını kontrol edin; çözümlendiğinde koruma kendiliğinden devam eder. |
| `custom_domain_o2o` | Bir özel domain kendi Cloudflare hesabınız üzerinden proxy'leniyor. | Kendi Cloudflare hesabınızda bu kaydı **DNS only** (gri bulut) yapın. |

### Komuta'nın ilgilendiği durumlar

- **"Koruma tam olarak yerinde değil."** (`exposed`, `drift_route_filter`, `drift_pod_token_lock`) — Servise şu anda erişim kontrolü olmadan ulaşılabiliyor olabilir. Komuta uyarıldı ve korumayı geri getiriyor. Bu sırada hassas bir servisi tamamen kapatmak isterseniz **Ağ** sekmesinden genel URL'i geçici olarak kapatabilirsiniz.
- **`bypass_policy_present`** — Başka bir ağ kuralı, trafiğin erişim kontrolünü atlayarak servise ulaşmasına izin veriyor. Özel ağı (mesh) yeni kapattıysanız değişiklik yayıldığında kendiliğinden düzelir.
- **`drift_mesh_peers`** — Özel ağ hâlâ seçtiğinizden fazla servisi içeri alıyor; birkaç dakika içinde kendiliğinden düzelir.
- **Diğer kodlar** — "Platform tarafındaki bir sorun nedeniyle koruma henüz uygulanamadı. Sizin bir şey yapmanız gerekmiyor."

Bu durumlarda **Genel bakış** sekmesindeki kartta **Dikkat gerekiyor** etiketi görünür.

### Açılamadığı durumlar

| Kart ne diyor | Neden | Ne yapmalı |
|---|---|---|
| "Erişim koruması genel URL üzerinden çalışır. Önce genel URL'i açın." | Servisin genel URL'i kapalı. | **Ağ** sekmesinden **Genel URL**'i açın. |
| "Servisin henüz genel adresi yok; ilk dağıtımdan sonra erişim korumasını açabilirsiniz." | Servis hiç dağıtılmamış. | Servisi dağıtın. |
| "Özel ağ (mesh) açıkken kilitli" | Özel ağ açık ve platformda özel ağ ile korumayı birlikte kullanma özelliği kapalı. | Özel ağı kapatın ya da bkz. [özel ağ](access-protection-machines.md#özel-ağdan-doğrudan-gelebilecek-servisler). |
| "Erişim koruması yalnızca Komuta'nın paylaşımlı barındırma kümelerinde çalışan servisler için kullanılabilir." | Servis kendi kümenizde ya da desteklenmeyen bir kümede çalışıyor. | — |

İş (job) ve zamanlanmış iş (cronjob) servislerinde genel URL olmadığı için erişim koruması yoktur.

---

## Korumayı kapatma

İki yol vardır, sonuçları aynıdır:

- **Şimdi herkese aç** — **Ayarlar** sekmesinin altındaki **Korumayı hemen kaldır** bölümünde. **Kurallar** sekmesindeki anahtarı kapatmak da aynı onayı açar. Onay: **Bu servis herkese açılsın mı?** ("Servis herkese açılıyor").
- **Tüm kuralları kaldırmak** — **Kurallar** sekmesinde giriş, IP listesi ve yol kurallarının hepsini kaldırırsanız düğme kırmızı **Korumayı kapat** olur ve **Erişim koruması kapatılsın mı?** onayı istenir.

Koruma **Kapatılıyor** durumu bittiğinde URL'e sahip herkes servisi açabilir.

**Ne saklanır, ne silinir:**

| Saklanır (korumayı yeniden açınca geçerli olur) | Yeniden girmeniz gerekir |
|---|---|
| Paylaşımlar (askıdakiler dahil) | Komuta girişi seçimi |
| Servis token'ları | IP izin listesi |
| Özel ağdan gelebilecek servisler listesi | Yol kuralları ve webhook yolları |
| | Koruma bitiş tarihi |
| | Uygulamaya kimlik bildirme ayarı |

Erişim kaydı koruma kapalıyken tutulmaz; daha önceki kayıtlar 30 gün sonra silinir.

---

## Servis koruması sihirbazı

Servis genel bakışında (dashboard) **Servis koruması** kartı korumanın durumunu ve kısa bir özetini gösterir ("Komuta ile giriş · IP kısıtı yok" gibi). Karta tıklayınca **Mevcut koruma** penceresi açılır; burada paylaşım listesi, koruma bitişi ve **Şimdi herkese aç** de bulunur. **Korumayı ayarla** (koruma açıksa **Korumayı düzenle**) sihirbazı başlatır:

1. **Koruma yöntemi** — **Komuta ile oturum açma**, **Belirli IP adresleri**, **Oturum açma ve belirli IP’ler** (burada **İkisi birden gereksin** ya da **Biri yeterli** seçilir) ya da **Yalnızca yol kuralları**.
2. **İzin verilen adresler** — IP kullanan yöntemlerde.
3. **Yol kuralları** — isteğe bağlı; **Yalnızca yol kuralları** seçildiyse en az bir kural gerekir.
4. **Kimler oturum açabilir?** — giriş gerekiyorsa; paylaşımlar burada eklenir. Siz listeye otomatik eklenirsiniz ("(siz)"); erişiminiz olmaması gerekiyorsa kendinizi kaldırabilirsiniz.
5. **Süre** — koruma bitişi ve tarih geldiğinde ne olacağı (**Koruma kurallarını sürdür** ya da **Herkese aç**).
6. **Değişiklikleri gözden geçir** — mevcut ve yeni ayarlar yan yana; **Korumayı uygula** ile kaydedilir.

Gözden geçirip uygulayana kadar hiçbir ayar değişmez. Kaydetme sırasında bir adım başarısız olursa başarılı adımlar korunur ve servis hiçbir zaman kendiliğinden herkese açılmaz. Webhook yolları, servis token'ları, erişim kaydı ve kimlik bildirme yalnızca **Erişim ve portlar** sayfasındadır (pencerede **Erişim ve portlar sayfasındaki gelişmiş ayarlar** bağlantısı).

Sihirbaz, servisin özel ağı (mesh) açıkken korumayı kurmaya izin vermez; bu durumda **Erişim ve portlar** sayfasını kullanın.

---

## Diğer özelliklerle ilişkisi

- **Uyku modu** — Uyuyan korumalı bir servis, ziyaretçi kontrolleri geçtikten sonra uyandırılır. Giriş yapmamış ya da izin listesinde olmayan ziyaretçiler korunan bir servisi uyandıramaz. Servis uyurken de koruma geçerlidir.
- **Önbellek** — Korunan bir servisin tüm yanıtlarına `Cache-Control: private, no-store` başlığı yazılır (uygulamanızın gönderdiği değer yerine) ve Cloudflare bu servisin yanıtlarını önbelleğe almaz. Koruma açılırken daha önce önbelleğe alınmış içerik temizlenir. Böylece korunan bir sayfa, giriş yapmamış birine önbellekten gösterilemez.
- **Alan adları** — Koruma, servisin `*.komuta.app` adresi ve **Alan adları** altında eklenen tüm aktif özel alan adları için birlikte geçerlidir. Yeni bir alan adı eklediğinizde koruma onu da kapsar. Özel alan adınızın DNS kaydı kendi Cloudflare hesabınızda proxy'li (turuncu bulut) olmamalıdır; **DNS only** olmalıdır.
- **Mavi-yeşil dağıtım** — Önizleme adresi de korunur; ziyaretçi önizleme adresi için ayrıca giriş yapar.
- **Özel ağ (mesh)** — Özel ağ trafiği ağ geçidinden geçmez. Korunan bir serviste diğer kümelerinizden kimin doğrudan gelebileceğini **Makineler** sekmesinde seçersiniz (bkz. [Makineler ve Özel Ağ](access-protection-machines.md#özel-ağdan-doğrudan-gelebilecek-servisler)).
- **Uygulamanız** — Komuta'nın oturum çerezleri ve servis token'ı başlığı isteğinizden çıkarılır; uygulamanız bunları görmez. Her izinli istekte Komuta'nın pod kilidi için eklediği `x-komuta-access` başlığı bulunur. Bu, servisinize özel gizli bir değerdir: kullanmanız gerekmez, loglamayın ve başka bir yere iletmeyin.
- **Desteklenmeyenler** — Tarayıcının başka bir siteden yaptığı CORS ön uçuş istekleri (`OPTIONS`) giriş gerektiren yollarda oturum taşımadığı için geçemez; webhook yollarında da `OPTIONS` seçilemez. Ziyaretçinin kendi kimlik sağlayıcınızla (SSO) giriş yapması şu an desteklenmez; ziyaretçinin bir Komuta hesabı olmalıdır.

---

## Gereksinimler ve izinler

- Servisin genel bir URL'i ve en az bir genel adresi olmalıdır.
- Servis, Komuta'nın paylaşımlı barındırma kümelerinde çalışmalıdır.
- IP listesi kullanılan yerlerde isteklerin Cloudflare üzerinden gelmesi gerekir; Komuta adresleri ve Komuta'ya eklenen özel alan adları bu şekilde çalışır.
- Erişim koruması organizasyonlarda varsayılan olarak açıktır. Organizasyonunuz için kapatılmışsa "Erişim koruması bu organizasyon için kapatılmış. Açtırmak için Komuta desteğiyle iletişime geçin." yazar.

| İzin | Ne sağlar |
|---|---|
| **Servis erişim korumasını yönet** | Korumayı açmak ve kapatmak; Komuta girişi, IP izin listesi, yol kuralları, webhook yolları, özel ağ listesi, bitiş tarihi, kimlik bildirme; **Şimdi herkese aç**. |
| **Korunan servisin paylaşımlarını yönet** | Paylaşımları ve servis token'larını yönetmek. |

İki izinde de servisi düzenleme erişiminiz olmalıdır. İzni olmayanlar ayarları salt okunur görür ("Salt okunur — bu ayarları değiştirmek için erişim korumasını yönetme yetkisi gerekir.").

---

## Erişim koruması rehberleri

- [Kurallar](access-protection-rules.md) — Komuta girişi, IP izin listesi, yol kuralları, erişim önizlemesi.
- [Giriş ve Paylaşım](access-protection-sign-in-sharing.md) — paylaşımlar, e-posta paylaşımı, dış paylaşım, ziyaretçinin gördükleri.
- [Makineler ve Özel Ağ](access-protection-machines.md) — webhook yolları, servis token'ları, özel ağ.
- [Erişim Kaydı](access-protection-activity.md) — **Etkinlik** sekmesi.
- [Bitiş ve Kimlik Bildirme](access-protection-settings.md) — koruma bitişi, hatırlatmalar, uygulamaya kimlik iletme ve JWT doğrulama.
- [Başvuru](access-protection-reference.md) — sınırlar, yanıtlar, başlıklar, hata mesajları, sık sorulan sorular, sözlük.

İlgili: [Servis Portları](services-ports.md) — **Erişim ve portlar** sayfasının port bölümü.
