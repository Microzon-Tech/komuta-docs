# Makineler ve Özel Ağ

Komuta girişi insanlar içindir: tarayıcıda bir giriş sayfası açılır, kişi hesabıyla giriş yapar. Bazı istekler ise bir insandan değil bir programdan gelir. GitHub bir push olduğunda uygulamanıza haber verir, Stripe bir ödeme olduğunda bildirim gönderir, CI hattınız dağıtımdan sonra bir sağlık kontrolü yapar, bir izleme aracı her dakika sitenizi yoklar. Bu programlar giriş sayfasını kullanamaz.

Erişim koruması bu tür istekler için üç yol ve servisinizin kabul ettiği HTTP yöntemleri ile tarayıcı CORS kontrolleri için bir bölüm sunar:

| Yol | Ne için | Nerede |
|---|---|---|
| **Webhook yolu** (açık yol) | Giriş yapamayan ve gönderdiğini imzalayan göndericiler (GitHub, Stripe, Slack gibi); imzayı Komuta kontrol edebilir | **Makineler** sekmesi → **Webhook'lar** |
| **Servis token'ı** | Sizin kontrol ettiğiniz programlar (CI işleri, izleme araçları, betikler) | **Makineler** sekmesi → **Servis token'ları** |
| **Özel ağdan doğrudan gelebilecek servisler** | Diğer kümelerinizdeki Komuta servislerinizin bu servise doğrudan ulaşması | **Makineler** sekmesi → **Özel ağdan doğrudan gelebilecek servisler** (özel ağın kendisi **Ağ** sekmesinde) |
| **Yöntemler ve CORS** | Her yolda yalnızca gereken HTTP yöntemlerine izin vermek ve tarayıcıların CORS kontrollerini girişten önce geçirmek | **Makineler** sekmesi → **Yöntemler ve CORS** |

Webhook, servis token'ı ve **Yöntemler ve CORS** bölümleri yalnızca erişim koruması açıkken görünür. Koruma kapalıysa sekmede **Erişim koruması kapalı** notu ve korumayı yönetebiliyorsanız **Kurallara git** düğmesi görünür. Özel ağ listesi bunun tek istisnasıdır: servisin özel ağı açıksa, **Kurallar** sekmesinde korumayı açmaya başladığınızda (kaydetmeden önce) liste görünür ve doldurulabilir.

---

## Webhook yolları

Webhook yolu, sitenin belirli bir yolunu Komuta girişi olmadan açar. Örneğin site yalnızca ekibinize açıkken `/webhooks/github` yoluna GitHub'ın gönderdiği istekler giriş yapmadan uygulamanıza ulaşır.

> **Önemli:** Webhook yolu herkesin giriş yapmadan istek göndermesine izin verir. İsteğin gerçekten GitHub'dan ya da Stripe'tan geldiğini birinin, göndericinin imzasını doğrulayarak kontrol etmesi gerekir (GitHub `X-Hub-Signature-256`, Stripe `Stripe-Signature` başlığını gönderir). Ya bir [kenarda imza kontrolü](#kenarda-imza-kontrolü) seçin, böylece Komuta imzasız istekleri uygulamanıza ulaşmadan reddeder; ya da imzayı **uygulamanızda** doğrulayın. İkisi de yoksa bu yola herkes istek gönderebilir.

### Webhook yolu açma

1. **Makineler** sekmesinde **Webhook'lar** bölümünde **Yol aç** düğmesine tıklayın.
2. **Webhook için yol aç** penceresinde:
   - **Yol** — açılacak yol, örneğin `/webhooks/github`. Bu yol ve altındaki yollar açılır.
   - **Yöntemler** — bu yolda kabul edilecek HTTP yöntemleri: `POST`, `PUT`, `PATCH`, `DELETE`, `GET`, `HEAD`. Varsayılan yalnızca `POST`'tur; en az bir yöntem seçilmelidir.
   - **Gönderici adresleri (isteğe bağlı)** — her satıra bir IP adresi ya da CIDR aralığı. Doldurursanız yola yalnızca bu adreslerden gelen istekler girer. Boş bırakırsanız her adresten istek kabul edilir; imza doğrulaması (kenarda ya da uygulamanızda) yine sizi korur. GitHub'ın webhook adresleri gibi göndericinin yayımladığı aralıkları buraya yazabilirsiniz (pencere örnek olarak `140.82.112.0/20` gösterir).
   - **Kenarda imza kontrolü** — **Yok, uygulamam kontrol ediyor** (varsayılan), **GitHub (X-Hub-Signature-256)**, **Stripe (Stripe-Signature)** ya da **Başka bir HMAC-SHA256 başlığı** ([aşağıya bakın](#kenarda-imza-kontrolü)).
3. **Yolu aç** ile kaydedin. Değişiklik birkaç saniye içinde uygulanır ("Webhook yolları uygulanıyor"). Bir imza kontrolü seçtiyseniz ardından yolun altına imza sırrını ekleyin ("Yolu açtıktan sonra altına imza sırrını ekleyin. O zamana kadar bu yola gelen her istek 401 alır.").

Listede her açık yol; yolu, seçili yöntemleri, gönderici listesini ("Yalnızca: …" ya da "Her adresten"), imza kontrolünü (örneğin "GitHub imzası kontrol ediliyor") ve servisin tam adresini gösterir. Bir yolu kapatmak için satırdaki çöp kutusu simgesine tıklayın; yol onay sorulmadan hemen kapatılır.

### Webhook yolları nasıl çalışır

- **Giriş istenmez.** Seçili yöntemlerle gelen istekler Komuta girişi olmadan geçer.
- **Yalnızca seçili yöntemler açıktır.** Başka bir yöntemle gelen istek, açık yol yokmuş gibi değerlendirilir; yani sitenin normal korumasından geçmesi gerekir.
- **Sitenin IP listesi ve üst yol kuralları uygulanmaz.** Açık yolda tek adres kontrolü, yolun kendi **Gönderici adresleri** listesidir.
- **Engelleme kuralları yine geçerlidir.** Açık yolun altına yalnızca **Tamamen engelle** kuralı konabilir; örneğin `/webhooks` açıkken `/webhooks/eski` engellenebilir. Açık yolun altına giriş, IP ya da kişi kuralı konamaz. Webhook penceresi, mevcut bir kuralla çakışan yolu "Bu yol /admin kuralıyla çakışıyor. Açık yol başka bir kuralla aynı yeri paylaşamaz." (örnekte `/admin`) uyarısıyla engeller; **Kurallar** sekmesinde bir açık yolun altına kural kaydetmeye çalışırsanız "… açık yolunun altında" hatası görürsünüz.
- **Yol eşleşmesi büyük/küçük harf duyarsızdır.** `/HOOKS/x` isteği `/hooks` yolunun altındadır. Bir istek ancak yolunun okunabileceği her biçim açık yolun altında kalıyorsa açılır; `%2f`, `..` ya da benzeri hilelerle açık yoldan başka bir yola kaçılamaz.
- **Açık yollar iç içe olamaz.** Bir açık yolun altına ya da aynı yola ikinci bir açık yol açılamaz.
- **Yöntem değiştirme başlıkları dikkate alınmaz.** Komuta isteğin gerçek yöntemine bakar. Uygulamanız bu yollarda `X-HTTP-Method-Override` ya da `X-HTTP-Method` gibi başlıkları kabul etmemelidir; aksi halde yalnızca `POST` açtığınız bir yola `DELETE` gibi davranan istekler gönderilebilir.
- **Webhook yolunda `OPTIONS` açılamaz.** Seçilebilen yöntemler arasında `OPTIONS` yoktur; bu yüzden açık yola gelen tarayıcı CORS kontrolü sitenin normal korumasına göre değerlendirilir. Tarayıcıdan başka bir siteden çağrılması gereken uç noktalar için bunun yerine [Yöntemler ve CORS](#yöntemler-ve-cors) bölümünü kullanın.
- **Ülkeler uygulanmaz, hız sınırı uygulanır.** Açık yol [ülke listesinden](access-protection-rules.md#ülkeler) muaftır (adres kontrolü kendi gönderici listesidir), ama istekleri [hız sınırına](access-protection-rules.md#hız-sınırı) dahildir. Engelleme kuralları ve [yöntem kuralları](#yöntemler-ve-cors) açık yoldan önce kontrol edilir.
- **Erişim kaydı** açık yola gelen her teslimatı **Açık yol isteği** / **Açık yola teslim edildi** olarak, tam yol yerine açık yolun önekiyle kaydeder.

### Kenarda imza kontrolü

İmza kontrolü seçildiğinde Komuta, istek uygulamanıza ulaşmadan önce göndericinin imzasını ham istek gövdesi üzerinden doğrular. Geçerli imzası olmayan istek `401` alır ve uygulamanıza hiç ulaşmaz. Yalnızca Komuta ağ geçidinden gelen istekler kontrol edilir: aynı kümedeki kendi servisleriniz ve özel ağ için seçtiğiniz servisler pod'larınıza imza kontrolü olmadan doğrudan ulaşır. Bu sizin için önemliyse imzayı uygulamanızda da doğrulamaya devam edin.

Yalnızca istek gövdesi imzalanır (Stripe zamanı da imzalar). İmzalı önekin altındaki alt yol, sorgu dizesi ve `X-GitHub-Event` gibi diğer başlıklar imzaya dahil değildir; bunlara tek başına güvenmeyin.

| Seçenek | Komuta neyi kontrol eder |
|---|---|
| **Yok, uygulamam kontrol ediyor** | Hiçbir şey; imzayı uygulamanız doğrulamalıdır. |
| **GitHub (X-Hub-Signature-256)** | `X-Hub-Signature-256: sha256=<gövdenin onaltılık HMAC-SHA256 değeri>`. |
| **Stripe (Stripe-Signature)** | `Stripe-Signature: t=<zaman>,v1=<imza>`: `<zaman>.<gövde>` değerinin HMAC-SHA256'sı. Zaman, Komuta'nın saatinden en fazla **300 saniye** farklı olabilir; böylece eski isteklerin yeniden gönderilmesi engellenir. |
| **Başka bir HMAC-SHA256 başlığı** | Seçtiğiniz **Başlık**'ta gövdenin HMAC-SHA256 değeri: `x-hub-signature-256`, `stripe-signature`, `x-signature`, `x-signature-256` ya da `x-webhook-signature` (ağ geçidi yalnızca bu başlıkları iletir). İsteğe bağlı olarak imzanın önüne gelen bir **Değer öneki** (örneğin `sha256=`; en fazla 16 karakter, boşluk ve virgül yok) ve imzanın **Kodlama**'sı: onaltılık (hex, varsayılan) ya da base64. |

İmzalı yolların kuralları:

- **Yalnızca `POST`, `PUT` ve `PATCH`.** İmzalı yol yalnızca gövdeli istekleri kabul eder ("POST, PUT ya da PATCH seçin; imzalı yol yalnız gövdeli istekleri kabul eder.").
- **Gövde en fazla 65.535 bayt (yaklaşık 64 KiB).** Ağ geçidi kontrole bir gövdenin en fazla bu kadarını iletir; bu yüzden daha büyük bir gövde doğrulanamaz ve `413` ile reddedilir. Webhook'ların çoğu bundan çok küçüktür, ama büyük bir olay bu sınırı aşabilir; örneğin çok sayıda commit içeren bir GitHub `push` olayı. Göndericiniz böyle olaylar gönderiyorsa kontrolü uygulamanızda tutun.
- **İmzalı yolun altındaki diğer istekler `401` alır.** İmzalı bir yolun altına gelen ama yolun kabul etmediği bir istek (`GET` gibi başka bir yöntem ya da birden fazla biçimde okunabilen bir yol) `401` ile reddedilir; sitenin normal kurallarına göre değerlendirilmez. Böylece imzalı yola kontrolün etrafından dolaşarak ulaşılamaz.
- **Önce gönderici listesi kontrol edilir.** Yolun **Gönderici adresleri** varsa başka bir adresten gelen istek, imzaya bakılmadan `403` alır.
- **Tekrar gönderme.** GitHub ve diğer HMAC imzaları zaman taşımaz; bu yüzden Komuta, ele geçirilmiş bir isteğin göndericinin yeniden deneme süresi içinde tekrar gönderilmesini engelleyemez. Stripe imzası zaman taşıdığı için Komuta 300 saniyeden eski kopyaları reddeder. Tekrar gönderme sizin için önemliyse, uygulamanız daha önce işlediği teslimat kimliklerini reddetsin (GitHub `X-GitHub-Delivery` başlığını gönderir).

Göndericinin aldığı yanıt:

| Durum | Yanıt |
|---|---|
| Geçerli imza | İstek uygulamanıza ulaşır (**Açık yola teslim edildi**). |
| İmza yok, bozuk ya da yanlış; ya da imzalı yolun altına gelen ama yolun kabul etmediği bir istek | `401`, düz metin `invalid webhook signature` |
| Yolun henüz imza sırrı yok | `401`, düz metin `webhook signature cannot be checked` |
| Gövde 65.535 bayttan büyük | `413`, düz metin `webhook body too large to verify` |

Bu retler erişim kaydında yolun önekiyle birlikte "Geçerli imzası olmayan bir webhook gönderdi", "Henüz imza sırrı olmayan bir yola webhook gönderdi" ve "64 KiB'tan büyük bir webhook gövdesi gönderdi" olarak görünür.

#### İmza sırları

İmzalı bir yol imzaları, altında listelenen **İmza sırları** ile kontrol eder. Bir sır eklenene kadar bu yola gelen her istek `401` alır ("Henüz sır yok. Gönderenin imzaladığı sırrı ekleyene kadar bu yola gelen her istek 401 alır.").

1. Yolun altında **Sır ekle**'ye tıklayın.
2. **Sırrın kaynağı** için seçin:
   - **Güçlü bir sır üret** — Komuta bir sır oluşturur. Sır **yalnızca bir kez** gösterilir: kopyalayıp göndericinin webhook ayarlarına yapıştırın (GitHub'da webhook'un **Secret** alanı).
   - **Elimdeki bir sırrı kullan** — göndericinin verdiği bir sırrı yapıştırın. Stripe için tek seçenek budur: uç noktanın imza sırrını Stripe panelinden yapıştırın (`whsec_` ile başlar).
3. **Sır ekle** ile kaydedin. Sır bir dakika içinde ağ geçidine ulaşır ("Sır kaydedildi ve bir dakika içinde kenara ulaşır").

Bilmeniz gerekenler:

- Komuta sırrı şifreli saklar ve bir daha göstermez; listede yalnızca bir kimlik ve eklendiği zaman görünür.
- Sır, boşluk ve kontrol karakteri içermeyen 8 ile 512 bayt arasında bir değerdir.
- **Yol başına en fazla 2 sır**; böylece kesinti olmadan sır değiştirebilirsiniz: yeni sırrı ekleyin, göndereni ona geçirin, sonra eskisini kaldırın. Arada ikisi de çalışır.
- Bir sırrı kaldırmak **{key} sırrı kaldırılsın mı?** onayını ister; hâlâ onunla imzalayan göndericiler bir dakika içinde reddedilir. Bir yolun son sırrını kaldırırsanız, yeni bir sır ekleyene kadar bu yola gelen her istek `401` alır.
- Yolu kapatmak, imza kontrolünü kaldırmak ya da başka bir sağlayıcıya geçmek yolun sırlarını siler; yeni bir imzalı yol yeni bir sır ister.
- İmza sırlarını görmek ve değiştirmek için **Servis erişim korumasını yönet** izni gerekir.

**Kenarda imza kontrolü** seçimi "Kenarda imza kontrolü bu platformda henüz açık değil." diyorsa imza kontrolü platformunuzda henüz açık değildir; imzayı uygulamanızda doğrulayın.

### Sınırlar

- Bir serviste en fazla **10** açık yol olabilir. Sınıra ulaşıldığında **Yol aç** düğmesi devre dışı kalır. Açık yollar, **Kurallar** sekmesindeki yol kurallarıyla birlikte toplam 50 yol kuralı sınırına da dahildir.
- `/` (sitenin tamamı) ve Komuta'nın kendi giriş yolu (`/.komuta-access` ile başlayanlar) açılamaz.
- Yol, [yol kurallarıyla](access-protection-rules.md#yol-yazım-kuralları) aynı yazım kurallarına uyar (en fazla 256 karakter). Pencere hatalı her yol için aynı mesajı gösterir: "/webhooks/github gibi bir yol girin. Sitenin tamamı açılamaz."
- Gönderici listesi en fazla 100 girdi alır ve yalnızca genel internet adreslerini kabul eder. Listede geçersiz bir satır varsa **Yolu aç** düğmesi devre dışı kalır.
- İmzalı bir yol yalnızca `POST`, `PUT` ve `PATCH` kabul eder; gövde en fazla 65.535 bayt, en fazla 2 imza sırrı.
- Webhook yolu açmak ve kapatmak ile imza sırlarını yönetmek için **Servis erişim korumasını yönet** izni gerekir; diğerleri listeyi yalnızca görür.
- Yalnızca açık yollardan oluşan bir koruma bir şey korumaz; webhook yolu, site ya da en az bir yol başka bir koruma altındayken anlamlıdır.

---

## Servis token'ları

Servis token'ı, tarayıcı kullanmayan bir programın Komuta girişinden geçmesini sağlayan gizli bir anahtardır. CI işleri, izleme araçları ve kendi betikleriniz için kullanılır. Token, isteğe bir HTTP başlığı olarak eklenir:

```bash copy
curl -H "x-komuta-service-token: kst_..." https://servisiniz.example.com/api/health
```

### Token oluşturma

1. **Makineler** sekmesinde **Servis token'ları** bölümünde **Token oluştur** düğmesine tıklayın.
2. Pencerede:
   - **Ad** — erişim kaydında tanıyacağınız bir ad, örneğin `github-actions`. 1–64 karakter; aynı serviste iki token aynı adı taşıyamaz.
   - **Geçerlilik sonu** — isteğe bağlı ama önerilir. Bu tarihten sonra token çalışmaz. En fazla 365 gün sonrası seçilebilir. Saatler hesabınızın saat dilimindedir.
   - **Neleri açabilir** — **Tüm site** (varsayılan) ya da **Yalnızca şu sayfalar**. İkincisinde her satıra bir yol yazılır (en fazla 50); bir yol altındaki sayfaları da açar (`/reports`, `/reports/2026`'yı da açar). Yol kuralları bu sayfaların içinde yine geçerlidir.
3. **Token oluştur** ile kaydedin.
4. **Token'ı şimdi kopyalayın** ekranı açılır. Token değeri **yalnızca bu bir kez** gösterilir; Komuta yalnızca değerin özetini saklar ve değeri bir daha gösteremez. **Kopyala** ile alın ve GitHub Actions secrets gibi bir gizli değişken deposunda saklayın, asla koda yazmayın. Ekranda kullanım örneği de vardır ("Başlık olarak gönderin"). **Kaydettim** ile kapatın.

Token'ı kaybederseniz yenisini oluşturup eskisini silin.

Token'ın biçimi `kst_<32 onaltılık karakter>_<43 karakter>` şeklindedir. Değer olduğu gibi, tek bir `x-komuta-service-token` başlığında gönderilir.

### Token ne yapabilir, ne yapamaz

- **Giriş şartını karşılar.** Geçerli bir token taşıyan istek, Komuta girişi yapmış biri gibi sayılır; açabileceği sayfalar token'ın **Neleri açabilir** ayarıyla sınırlıdır.
- **IP kurallarını aşamaz.** Site ya da yol bir IP listesi istiyorsa, token taşıyan istek de listedeki bir adresten gelmelidir. İstisna: kural **Biri yeterli** birleşimini kullanıyorsa token giriş yerine geçer ve adres aranmaz.
- **Seçilen kişiler kurallarını aşamaz.** Yalnızca belirli kişilere açılan bir yol, token'la açılmaz.
- **Başlık varsa tek başına karar verir.** İstekte `x-komuta-service-token` başlığı varsa sonuç yalnızca token'a göre belirlenir: geçersiz bir token, tarayıcıda geçerli bir oturum olsa bile reddedilir.
- **Girişin gerekmediği yerlerde okunmaz.** Korumasız yollarda ve IP listesiyle girişsiz geçilen yerlerde token'a bakılmaz; geçersiz bir token olsa bile istek geçer.
- **Uygulamanıza ulaşmaz.** Komuta başlığı kontrol ettikten sonra istekten siler; token değeri uygulamanızın loglarına düşmez.
- **Komuta girişi gerekir.** Token'lar yalnızca koruma Komuta girişi isterken (sitede ya da bir yol kuralında) çalışır. Giriş istenmiyorsa bölüm "Servis token'ları yalnızca Komuta girişi gerekirken çalışır." der ve yeni token oluşturulamaz.

### Yanıtlar

| Durum | Yanıt |
|---|---|
| Geçerli token, açabileceği bir sayfa | İstek uygulamanıza ulaşır. |
| Geçerli token, kapsamı dışındaki bir sayfa | `403`, gövde `this service token cannot open this path` |
| Bilinmeyen, süresi dolmuş, silinmiş, bozuk ya da birden fazla kez gönderilmiş token | `401`, gövde `invalid service token`, başlık `WWW-Authenticate: KomutaServiceToken realm="komuta"`. Giriş sayfasına yönlendirme yapılmaz. |

### Listedeki token'lar ve silme

Listede her token'ın adı, açabildiği sayfalar ("Yalnızca: …" ya da **Tüm site**) ve geçerlilik sonu ("… tarihine kadar" ya da **Bitiş yok**) görünür. Çalışmayan bir token'da **Çalışmıyor** etiketi görünür (süresi dolmuş, koruma artık giriş istemiyor ya da özellik platformda kapatılmış olabilir). Çalışan token'larda etiket yoktur.

Silmek için çöp kutusu simgesine tıklayın ve **Bu token silinsin mi?** onayını verin. Token'ı kullanan her şey yaklaşık 30 saniye içinde reddedilir.

### İlk token

Bir serviste ilk token oluşturulduğunda Komuta, başlığın uygulamaya giden yolda kontrol edilebilmesi için servisin yönlendirme ayarlarını yeniler (yeni bir build yapılmaz). Bu birkaç dakika sürebilir; bu sırada token henüz çalışmıyor gibi görünebilir.

### Sınırlar ve izinler

- Bir serviste en fazla **20** servis token'ı olabilir.
- Token oluşturmak ve silmek için **Korunan servisin paylaşımlarını yönet** izni gerekir (paylaşımlarla aynı izin).
- Erişim kaydında token'la açılan sayfalar (yalnızca `GET` ve dosya olmayan yollar) **Servis token'ı: {ad}** olarak görünür; token'la yapılan `POST` gibi diğer istekler kaydedilmez. Reddedilen token istekleri her yöntem için kaydedilir.

---

## Yöntemler ve CORS

**Yöntemler ve CORS** bölümünde iki ayar vardır: tarayıcıların CORS kontrollerini girişten önce geçiren bir anahtar ve her yolun kabul ettiği HTTP yöntemlerinin listesi ("Tarayıcıların girişten önce CORS kontrolü göndermesine izin verin; her yolda yalnızca gereken HTTP yöntemleri kabul edilsin. Başka bir yöntemle gelen istek 405 ile yanıtlanır."). Değişiklikler yaklaşık bir dakika içinde geçerli olur ("Kaydedildi. Ağ geçidi yaklaşık bir dakika içinde uygular.").

### CORS kontrollerine girişsiz izin ver

Başka bir sitedeki bir sayfa servisinizi tarayıcıdan çağırdığında (örneğin `app.ornek.com` adresindeki bir ön yüzün `api.ornek.com` adresindeki API'yi çağırması), tarayıcı önce bir CORS kontrolü gönderir: çerezsiz bir `OPTIONS` isteği. Giriş isteyen bir sayfada bu kontrol `401` alır ve tarayıcı asıl isteği hiç göndermez. **CORS kontrollerine girişsiz izin ver** bu kontrolleri geçirir:

- Yalnızca gerçek bir tarayıcı kontrolü geçer: tam olarak bir `Origin` başlığı ve `GET`, `HEAD`, `POST`, `PUT`, `PATCH` ya da `DELETE` yöntemini bildiren tam olarak bir `Access-Control-Request-Method` başlığı taşıyan bir `OPTIONS` isteği. Diğer `OPTIONS` istekleri yine giriş ister.
- Engelleme kuralları, IP izin listesi, ülke listesi, hız sınırı ve Cloudflare kontrolü yine uygulanır. (Giriş ve IP listesinin **Biri yeterli** ile birleştiği yerlerde kontrol giriş yapmış sayılır ve her adresten geçer; **İkisi birden gereksin** seçiliyse listedeki bir adresten gelmelidir.)
- Yöntem kuralları `OPTIONS`'a değil, tarayıcının sorduğu yönteme göre kontrol edilir: `/api` yalnızca `GET` ve `POST`'a izinliyse `POST` için gelen kontrol geçer, `DELETE` için gelen kontrol `405` alır.
- Yalnızca kontrol geçer. Ardından gelen asıl istek yine giriş ister (çerez, servis token'ı ya da onu içeri alan bir IP listesi); CORS başlıklarını yine uygulamanız döndürür.
- Yalnızca koruma Komuta girişi isterken anlamlıdır ("Yalnızca Komuta girişi açıkken anlamlıdır.").

### Yöntem kuralları

Bir yöntem kuralı, bir yolun kabul ettiği HTTP yöntemlerini listeler. Başka bir yöntemle gelen istek; giriş, paylaşım bağlantısı ya da webhook yoluna bakılmadan `405`, düz metin `method not allowed` ve kuralın yöntemlerini listeleyen bir `Allow` başlığıyla (örneğin `Allow: GET, HEAD`) yanıtlanır.

1. **Yol ekle**'ye tıklayın. İlk satır `/` (sitenin tamamı) ve `GET`, `HEAD` ile başlar.
2. **Yol**'u girin ve **İzin verilen yöntemler**'i işaretleyin: `GET`, `HEAD`, `POST`, `PUT`, `PATCH`, `DELETE`, `OPTIONS`.
3. **Yöntemleri kaydet** ile kaydedin (ya da **Vazgeç** ile geri alın). **Bu yolu kaldır** bir satırı siler.

Kurallar nasıl eşleşir:

- Bir kural yolunu ve altındaki her şeyi kapsar; `/` sitenin tamamını kapsar.
- Eşleşen **en uzun** yol karar verir. Örneğin `GET`, `HEAD` ile `/` ve `GET`, `POST`, `OPTIONS` ile `/api` varsa `POST /api/siparisler` geçer, `POST /hakkimizda` `405` alır.
- Yollar [yol kurallarının yazım kurallarına](access-protection-rules.md#yol-yazım-kuralları) uyar; tek fark, tek başına `/` yazılabilmesidir. Her yol bir kez yazılabilir ("Her yol yalnızca bir kez yazılabilir.") ve her birine en az bir yöntem gerekir ("Her yol için en az bir yöntem seçin.").
- Bir isteğin yolu birden fazla biçimde okunabiliyorsa (kodlanmış karakterler, `..` ve benzerleri), eşleşen her kural yönteme izin vermelidir.
- Komuta'nın kendi giriş yolu (`/.komuta-access/callback`) yöntem kurallarına hiçbir zaman tabi değildir.
- Hiç yöntem kuralı yoksa tüm yöntemlere izin verilir ("Tüm yöntemlere izin veriliyor. Sınırlamak için bir yol ekleyin.").
- Erişim kaydında ret, kuralın yoluyla birlikte "Bu yolun izin vermediği bir yöntem kullandı" olarak görünür.

### Sınırlar ve izinler

- Bir serviste en fazla **50** yöntem kuralı olabilir. Ayrı bir listedir; 50 yol kuralı sınırına dahil değildir.
- Bu ayarları değiştirmek için **Servis erişim korumasını yönet** izni gerekir.

Bu bölümü görmüyorsanız CORS kontrolleri ve yöntem kuralları platformunuzda henüz açık değildir. Ayarlar kaydedildikten sonra özellik kapatılırsa bölüm bunları salt okunur gösterir: "Bu ayarlar bu platformda henüz değiştirilemiyor; mevcut ayarlar geçerli kalır."

---

## Özel ağdan doğrudan gelebilecek servisler

Komuta'nın **özel ağı (mesh)**, farklı kümelerdeki servislerinizin birbirine genel internete çıkmadan ulaşmasını sağlar. Özel ağ trafiği Komuta'nın erişim kontrolünden geçmez, doğrudan servisin kendisine gider. Bu yüzden korunan bir serviste özel ağdan kimin gelebileceği ayrıca seçilir.

### Nasıl çalışır

- Özel ağın kendisi **Ağ** sekmesindeki **Özel ağ (mesh)** kartından açılır ve kapatılır.
- Korunan bir serviste, **Makineler** sekmesindeki **Özel ağdan doğrudan gelebilecek servisler** bölümünde seçtiğiniz servisler, **diğer kümelerinizden** bu servise doğrudan, Komuta girişi ve adres kuralları olmadan ulaşır.
- **Liste boşsa** özel ağdan hiçbir servis doğrudan gelemez ("Servis seçilmedi. Özel ağdan hiçbir servis doğrudan gelemez.").
- **Aynı kümedeki servisleriniz** bu listeden etkilenmez; bugünkü gibi ulaşmaya devam eder.
- Bu servisin özel ağı kapalıysa liste saklanır ve özel ağ açıldığında uygulanır.
- Liste yalnızca koruma açıkken uygulanır. Korumayı kapattığınızda özel ağ eskisi gibi tüm servislerinize açılır; liste silinmez, korumayı yeniden açtığınızda tekrar geçerli olur.

### Servis ekleme ve kaldırma

1. **Bir servis seçin** listesinden organizasyonunuzun başka bir servisini seçin (her biri "servis adı · küme adı" olarak görünür).
2. **İzin ver** düğmesine tıklayın.

Kaldırmak için satırdaki çöp kutusu simgesini kullanın. Ekleme ve kaldırma onay sorulmadan hemen kaydedilir.

Listede iki uyarı görebilirsiniz:

- **Silinmiş bir servis** — seçtiğiniz servis silinmiş; satırı kaldırabilirsiniz.
- **Herkese açık, korumasız** — seçtiğiniz servise internetten herkes ulaşabiliyor. Bu servis, buraya ulaşmak için bir aracı olarak kullanılabilir. O servisi de koruyun ya da aldığı istekleri olduğu gibi iletmediğinden emin olun.

### Sınırlar ve izinler

- En fazla **50** servis seçilebilir; yalnızca aynı organizasyonun servisleri seçilebilir ve bir servis kendi listesine eklenemez.
- Bölüm yalnızca **Servis erişim korumasını yönet** izni olan kişilere görünür.

> Bu özellik platformda açık değilse eski kural geçerlidir: özel ağa açık bir serviste erişim koruması açılamaz, korunan bir servis de özel ağa açılamaz. **Ağ** sekmesindeki kart bu durumda "Erişim korumasıyla birlikte kullanılamaz" notunu gösterir. Özellik açıkken kart "Erişim korumasıyla birlikte çalışır" der.

---

## İlgili Dokümanlar

- [Kurallar](access-protection-rules.md) — sitenin ve yolların korunması, ülkeler ve hız sınırı.
- [Erişim Kaydı](access-protection-activity.md) — webhook teslimatlarının ve token kullanımının görüldüğü yer.
- [Bitiş ve Kimlik Bildirme](access-protection-settings.md) — token'la gelen isteğin uygulamaya nasıl tanıtıldığı.
- [Başvuru](access-protection-reference.md) — tüm sınırlar ve yanıtlar.
