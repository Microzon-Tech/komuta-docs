# API Gateway: API'ler, Anahtarlar ve Politikalar

Bu sayfa bir gateway'e API bağlamayı, endpoint tanımlamayı, çağıranlara anahtar vermeyi, hız limiti ve kotaları, gateway'in sunduğu diğer politikaları ve API'lerinizi çağıranların aldığı yanıtları anlatır. Gateway'in oluşturulması, değişikliklerin uygulanması, izleme ve gateway geneli ayarlar [API Gateway](api-gateway.md) sayfasındadır.

Bu sayfadaki her değişiklik önce kaydedilir. Trafiğe yansıması için gateway sayfasının üstündeki **Değişiklikleri uygula**'ya tıklayın.

---

## API Bağlama

API, gateway'e bir yol önekiyle bağlanan Komuta servisidir. Kaynak olarak şu an yalnızca Komuta'daki mevcut bir servis seçilebilir; harici bir URL bağlanamaz.

1. Gateway'in **API'ler** sekmesinde **API bağla**'ya tıklayın.
2. **Kaynak** bölümünde servisi arayıp seçin. Erişebildiğiniz projelerdeki servisler listelenir; job ve cronjob gibi trafik almayan iş yükleri ile bu gateway'e zaten bağlı servisler listede yoktur.
3. **Yönlendirme ve erişim** bölümünü doldurun (aşağıdaki tablo).
4. **Kimlik doğrulama** modunu seçin ve gerekirse **Anahtar başına hız limiti (istek / saniye)** girin.
5. İsterseniz **Gelişmiş politika** bölümündeki ayarları yapın.
6. **API bağla**'ya tıklayın. "Rota kaydedildi. Yayına almak için değişiklikleri uygulayın." mesajı görünür.
7. Sayfanın üstündeki **Değişiklikleri uygula**'ya tıklayın.

Servis listesinde görebileceğiniz durumlar:

| Durum | Anlamı |
|---|---|
| **Port yok** | Servis henüz port yayınlamıyor ve bağlanamaz. Önce servisin **Portlar** sekmesinden port ekleyin ([Erişim ve Portlar](services-ports.md)). |
| **Dağıtılmadı** | Servis bağlanabilir; rota hemen oluşturulur ama servisin ilk dağıtımı başarılı olana kadar hata döner. |
| "Gateway erişebilsin diye Komuta, … servisini özel mesh ağınıza açacak." | Servis özel ağa açılır. Servisin kendi genel adresi değişmez. |

| Alan | Açıklama |
|---|---|
| **API adı** | API'yi listelerde ve trafik grafiklerinde tanımlar; gateway içinde benzersiz olmalıdır. Servisi seçince servisin adıyla dolar. |
| **Dış erişim yolu** | Çağıranların kullanacağı önek, örneğin `/orders`. `/` ile başlar; küçük harf, rakam ve tire içerir; en fazla 64 karakterdir. Servisi seçince servis adından önerilir. `/__health` gateway'in kendi sağlık kontrolüne ayrılmıştır. İki API aynı yolu kullanamaz, ama bir API'nin yolu diğerinin altında olabilir (`/orders` ve `/orders/v2`). |
| **Dış erişim yolu önekini kaldır** | Açıkken (varsayılan) servis isteği önek olmadan alır: `/orders/v1/items` isteği servise `/v1/items` olarak gider. Kapalıyken `/orders/v1/items` olarak gider. Sonucu formun altındaki **İstek akışını önizle** bölümünde görebilirsiniz. |
| **Yanıtları akış olarak ilet** | Yanıt, gateway'de biriktirilmeden çağıranın okuduğu hızda iletilir. Büyük indirmeler ve uzun süren akışlar içindir. Yanıt önbelleğiyle birlikte kullanılamaz. |
| **Açıklama** | API'nin ne işe yaradığı. API referansında ve OpenAPI belgesinde görünür; gateway'in davranışını etkilemez. |

Bağlı bir API'yi değiştirmek için **API'ler** sekmesindeki listeden API'yi seçin; ayarları sağdaki panelde açılır. Kaydetmek için **Değişiklikleri kaydet**'e tıklayın. Düzenlerken hedef servis değiştirilemez; yolu başka bir servise göndermek için API'yi kaldırıp yeniden ekleyin. Yolu değiştirirseniz form "Uygulandıktan sonra {eski yol} yolunu kullanan çağrılar 404 alacak." uyarısını gösterir.

> **Not:** Gateway üzerinden gelen istekler, hedef servisin canary ya da blue-green dağıtımındaki trafik ağırlıklarına uymayabilir. Dağıtım stratejileri için bkz. [Dağıtım Stratejileri](deployment-strategies.md).

> **Not:** API'yi öneki sonunda `/` olmadan çağırın (`/orders`). `/orders/` isteği yanıt yerine `/orders` adresine yönlendirme (301) alır.

---

## Kimlik Doğrulama

Her API için üç moddan biri seçilir. Yeni bir API'de **API anahtarı** seçili gelir.

| Mod | Çağıran ne gönderir | Hız limiti kimi ayrı sayar |
|---|---|---|
| **API anahtarı** | `X-Api-Key` başlığında gateway'in verdiği anahtar. Anahtar yalnızca bu başlıkta kabul edilir. | Her çağıranı (anahtarı) |
| **JWT** | `Authorization: Bearer <token>` başlığında kimlik sağlayıcınızın imzaladığı token. | Her token'ı |
| **Herkese açık** | Hiçbir şey; kimlik doğrulama yapılmaz. | Her istemci adresini |

- **API anahtarı** modunu seçtiğinizde gateway'de aktif anahtar yoksa "Bu gateway'de henüz aktif anahtar yok. Önce Anahtarlar sekmesinden bir tane oluşturun; aksi halde bu API'ye gelen tüm istekler reddedilir." uyarısı çıkar ve kayıt reddedilir.
- **JWT** modu için önce **Ayarlar → JWT doğrulama** kartında **OIDC discovery URL** girilmelidir; girilmemişse form "JWT kullanan bir API'yi kaydetmeden önce gateway ayarlarında OIDC keşif URL'sini tanımlayın." uyarısını gösterir. Bkz. [JWT Doğrulama](api-gateway.md#jwt-doğrulama).
- Çağıranların kendi limitleri, API kısıtlamaları ve kullanım planları yalnızca **API anahtarı** isteyen API'lerde geçerlidir. JWT ve herkese açık API'ler çağıranı tanımlamaz.

**Servisinizin aldığı başlıklar.** API anahtarı isteyen bir API'de gateway, çağıranı servisinize `X-Consumer-Username` ve `X-Consumer-Custom-ID` (çağıranın **Müşteri kimliği**) başlıklarıyla bildirir. `X-Api-Key` başlığı, **Ayarlar**'daki **API anahtarını servislerinize ilet** açık değilse servisinize ulaşmaz.

**Müşteri kimliği başlığı (isteğe bağlı).** Yalnızca API anahtarı modunda görünür. Servisinizin müşteri kimliğini kendi seçtiğiniz bir başlıktan (örneğin `X-Customer-Id`) okumasını istiyorsanız adını buraya yazın. Gateway bu başlığı her isteğe koyar ve istemcinin gönderdiği kopyayı kaldırır; böylece servisiniz değere güvenebilir. Gateway'in kendi kullandığı başlıklar seçilemez.

---

## Hız Limiti

**Anahtar başına hız limiti (istek / saniye)** bir API'ye her çağıranın saniyede kaç istek gönderebileceğini belirler. `0` sınırsız demektir; konsol en fazla 10.000 kabul eder.

- Limit API genelinde tek bir bütçedir; kendi limiti olmayan endpoint'ler bu bütçeyi paylaşır.
- Önbellekten dönen yanıtlar da limite sayılır.
- Limiti aşan istek `429` alır. Bkz. [Çağıranlarınızın Aldığı Yanıtlar](#çağıranlarınızın-aldığı-yanıtlar).
- Bir çağıranın kendi hız limiti ya da kullanım planının limiti, o çağıran için API'nin limitinin yerine geçer; daha yüksek olsa bile. Bkz. [Çağıranların Limitleri](#çağıranların-limitleri).

> **Uyarı:** Sayaç deposu seçilmemişse her replika limiti kendi başına sayar. Form bunu "Replika başına sayılır. Bu gateway en fazla {replika} replikaya çıkabilir, yani {limit}/sn'lik bir limit bu API'de anahtar başına {etkin}/sn'ye kadar izin verir…" diye gösterir. Tam limit için [Hız Limitlerinin Sayılması](#hız-limitlerinin-sayılması) bölümüne bakın.

---

## API Politikaları

API formundaki **Gelişmiş politika** bölümü API'nin tüm istekleri için geçerlidir. Endpoint'ler bu ayarları devralır; bir endpoint'te ayrıca değiştirilebilirler.

| Ayar | Açıklama |
|---|---|
| **Zaman aşımı** | Bu API için gateway'in servisinizi ne kadar bekleyeceği. Boş bırakılırsa gateway'in zaman aşımı kullanılır. 1 ms ile 300 saniye arasında, `500ms`, `45s`, `2m` gibi yazılır. |
| **Yeniden deneme** | Servisinize ulaşamayan bir isteğin kaç kez daha deneneceği (0–3). Boş bırakılırsa platform varsayılanı kullanılır. |
| **WebSocket** | Çağıranların bağlantıyı WebSocket'e yükseltmesine izin verir. API'nin zaman aşımı boşta kalan soketi de sınırlar: o süre boyunca hiçbir şey taşımayan soket kapatılır. |
| **Yanıt önbelleği** | Yanıtları sabit bir süre önbellekte tutar. Aşağıya bakın. |
| **IP erişim kuralları** | Çağıranları IP adresine göre kabul eder ya da engeller. Aşağıya bakın. |
| **Gerekli scope’lar** | Yalnızca JWT modunda. Token burada listelenen her scope'u taşımalıdır; yoksa istek reddedilir. En fazla 16 scope, her biri boşluksuz 1–64 karakter. |
| **Kullanımdan kaldırma duyurusu** | API'nin emekliye ayrılacağını her yanıtla duyurur. Aşağıya bakın. |
| **Bakım** | API'yi bakım mesajıyla kapatır. Aşağıya bakın. |
| **İstek ve yanıt dönüşümleri** | Başlıkları ve yolu değiştirir. Aşağıya bakın. |
| **Doğrulama ve circuit breaker** | Bozuk istekleri reddeder, sürekli hata veren endpoint'i duraklatır. Aşağıya bakın. |

Dönüşümler, doğrulama, circuit breaker ve endpoint'ler yeni API formunda görünmez: API'yi bağlayıp kaydettikten sonra listeden yeniden açın.

### Yanıt Önbelleği

1. **Yanıt önbelleği** satırını açıp **Yanıtları önbelleğe al**'ı açın.
2. **Saklama süresi (saniye)** girin (1–3600; varsayılan 60).
3. İsterseniz **Önbelleği ayıran sorgu parametreleri** ve **Önbelleği ayıran istek başlıkları** alanlarına virgülle ayrılmış adlar yazın. Sorgu parametreleri boşsa sorgunun tamamı kullanılır; güvenli varsayılan budur.

- Yalnızca `GET` ve `HEAD` isteklerine verilen `200` yanıtlar önbelleğe alınır. `Cache-Control: private` ya da `no-store` taşıyan veya çerez (`Set-Cookie`) ayarlayan yanıtlar saklanmaz.
- Kimlik doğrulama önbellek anahtarına dahildir: anahtarlı API'lerde her çağıranın kopyası ayrıdır; JWT ve herkese açık API'lerde `Authorization` başlığı anahtara girer.
- Önbellekten dönen yanıtlar da hız limitine ve kotaya sayılır.
- Yanıttaki `X-Komuta-Cache` başlığı önbellek sonucunu (`HIT`, `MISS`, `EXPIRED`) gösterir.
- **API önbelleğini temizle**, önbellekteki yanıtları bir sonraki yapılandırmada kullanılamaz işaretler; yanıtlar siz uygulayana kadar sunulmaya devam eder.
- Önbelleğin boyutu gateway genelinde ayarlanır ([Çalışma Zamanı](api-gateway.md#çalışma-zamanı)). Dolu önbellek hata vermez; yeni yanıtları sessizce saklamayı bırakır. Doluluğu **Genel bakış → Trafik** sekmesinde görebilirsiniz.

### IP Erişim Kuralları

**Erişim biçimi** olarak **Kısıtlama yok**, **Yalnızca listedeki adreslere izin ver** ya da **Listedeki adresleri engelle**'yi seçin ve adresleri her satıra bir tane olacak şekilde yazın (`203.0.113.7`, `10.0.0.0/8`, `2001:db8::/32` gibi). Kurala uymayan çağıran `403` alır.

> **Not:** IP kuralları platformda adres filtresi açıkken uygulanabilir. Açık değilse kayıt "Adres filtresi bu platformda henüz açık değil." ile başlayan bir mesajla reddedilir. Bu durumda [Destek](support-tickets.md) üzerinden bize ulaşın.

### Kullanımdan Kaldırma Duyurusu

**Kullanımdan kaldırma** ve **Kapatma** tarihlerini ve isterseniz değişikliği anlatan bir **Bağlantı** girin. İkisi de isteğe bağlıdır; kapatma tarihi kullanımdan kaldırma tarihinden önce olamaz. Bağlantı en fazla 512 karakterlik mutlak bir `http(s)` adresi olmalı ve `$`, boşluk içermemelidir. Saatler ayarlarınızda seçili saat dilimindedir.

Duyuru açıkken API'nin her yanıtı `Deprecation`, `Sunset` ve `Link` (`rel="deprecation"`) başlıklarını taşır. Listede API'nin yanında **kullanımdan kalkıyor** etiketi görünür.

### Bakım

**Bu API’yi bakıma al**'ı açın ve **Çağıranlara mesaj** alanına en fazla 512 karakterlik bir mesaj yazın. Uyguladıktan sonra API'nin her rotası `503` ve şu gövdeyle yanıt verir:

```json
{"error":"maintenance","message":"<mesajınız>"}
```

Mesaj boşsa `"This API is under maintenance."` gönderilir. Anahtar ya da token isteyen bir API'de mesajı yalnızca bu gateway için geçerli bir anahtar ya da token gönderen çağıranlar görür; diğerleri her zamanki `401`'i alır. Herkese açık bir API'de yolu isteyen herkes görür.

### İstek ve Yanıt Dönüşümleri

- **Backend'e gönderilen başlıklar** (**İstek başlığı kuralı**) ve **Çağırana dönen başlıklar** (**Yanıt başlığı kuralı**) için kural ekleyin. İşlemler: **Yaz** (çağıranın gönderdiğinin yerine geçer), **Ekle** (onunkini bırakıp ikinci bir kopya ekler), **Kaldır** ve **Yeniden adlandır**. Backend'inizin güvendiği her başlık için **Yaz**'ı kullanın.
- Aynı tarafta bir başlık için tek kural yazılabilir. Başlık adları harf, rakam ve `- _ .` içerir (en fazla 64 karakter); değerler tek satır, en fazla 512 karakterdir ve `$` içeremez.
- Kurallara yazılan değerler gateway'in yapılandırmasında saklanır ve yapılandırmayı okuyabilen herkese görünür. **Bu değeri bir değişkenden oku** düğmesiyle değer yerine gateway servisinin bir ortam değişkeninin adını verebilirsiniz; yalnızca ad saklanır. Değişken gateway servisinde tanımlı değilse kayıt reddedilir.
- **Backend'e gönderilen Host başlığı** boş bırakılırsa çağıranınki iletilir.
- **Yolu yeniden yaz** bir desen ve `/` ile başlayan bir karşılık ister. Desen, API öneki kaldırıldıktan sonraki yola uygulanır; eşleşmeyen istekler olduğu gibi geçer. Desen gruplarını `$1`, `$2` ile kullanın.

### Doğrulama ve Circuit Breaker

**Doğrulama.** **İsteğin taşıması gereken başlıklar** listesine zorunlu başlıkları, **İstek gövdesinin sağlaması gereken alanlar** bölümüne JSON gövde alanlarını ekleyin. Her alan için tür (`String`, `Integer`, `Number`, `Boolean`, `Array`, `Object`), **zorunlu** işareti, en küçük/en büyük değer, en kısa/en uzun uzunluk ve metin alanları için bir desen (`^[A-Z]{3}-[0-9]{4}$` gibi) ya da kabul edilen değerler tanımlanabilir.

- Kurallara uymayan isteğe gateway `400` döner ve hangi alanın takıldığını söyler; servisiniz o isteği hiç görmez.
- Doğrulama kimlik doğrulamadan sonra çalışır: anahtarı ya da token'ı olmayan çağıran önce `401` alır.
- Gövde kuralları yalnızca tüm metotları gövde gönderen (`POST`, `PUT`, `PATCH`) bir endpoint'te tanımlanabilir; zorunlu başlıklar her yerde tanımlanabilir.
- `X-Request-Id` zorunlu tutulamaz; gateway bu başlığı denetimden önce kendisi doldurur.

**Circuit breaker.** **Sürekli hata veren endpoint'i duraklat**'ı açın:

| Alan | Varsayılan | Sınır |
|---|---|---|
| **Duraklatmadan önceki hata sayısı** | 3 | 1–100 |
| **En uzun duraklama (saniye)** | 300 | 3–3600 |
| **Hata sayılacak HTTP kodları** | 500, 502, 503, 504 | Yalnızca 5xx kodları |
| **Bekleme sırasında döndürülecek HTTP kodu** | 503 | 400–599 |

Hatalar yol başına sayılır; bir endpoint'in duraklaması servisinizin geri kalanını etkilemez. İlk duraklama birkaç saniyedir ve hatalar sürdükçe en uzun süreye kadar katlanır; bu sırada çağıranlar seçtiğiniz kodu alır. Duraklama sırasında reddedilen istekler de hız limitine sayılır.

---

## Endpoint'ler

Endpoint, bir API'nin altındaki tek bir yol ve metot kümesidir. Tek bir yolun kendi önbelleğine, limitine, zaman aşımına ya da kimlik doğrulamasına ihtiyacı olduğunda kullanılır; örneğin hızlı bir API'deki yavaş bir rapor için daha uzun zaman aşımı. Endpoint olarak tanımlanmayan her istek API'nin kendi ayarlarıyla servisinize ulaşmaya devam eder.

1. **API'ler** sekmesinde API'yi seçin ve **Endpoint ekle**'ye tıklayın.
2. **Temel** sekmesini doldurun:
   - **Ad** — endpoint'in trafiğini grafiklerde etiketler (en fazla 63 karakter).
   - **Özet** ve **Açıklama** — API referansında ve OpenAPI belgesinde görünür.
   - **Metotlar** — `GET`, `HEAD`, `POST`, `PUT`, `PATCH`, `DELETE`, `OPTIONS`; en az biri.
   - **Yol** — API önekine göre yazılır. `{ad}` tek bir yol parçasını karşılar, sonda `/*` altındaki her şeyi kapsar. Harf, rakam ve `- . _ ~` kullanılır; yalnızca `/` API'nin kendi önekidir.
3. Değiştirmek istediğiniz politikaların düğmesini açın. Düğmesi kapalı her politika API'nin ayarını izler.
4. **Kaydet**'e tıklayın ve değişiklikleri uygulayın.

| Sekme | Geçersiz kılınabilen politikalar |
|---|---|
| **Temel** | **Kimlik doğrulama**, **Gerekli scope’lar**, **Hız limiti**, **Kullanımdan kaldırma duyurusu** |
| **Önbellek** | **Yanıt önbelleği** (süre, sorgu parametreleri, başlıklar; her listede en fazla 8 ad) |
| **Dönüşüm** | **İstek ve yanıtı değiştir** |
| **Koruma** | **Zaman aşımı**, **Adrese göre kim çağırabilir** (en fazla 60 adres), **Bozuk istekleri reddet**, **Backend hata verince duraklat** |

- Endpoint'in **Hız limiti** çağıran başına ve yalnızca bu endpoint için sayılır (0–10.000; `0` bu endpoint'te limit yok demektir). Limit girilmezse endpoint API'nin bütçesini paylaşır; kendi kimlik doğrulaması varsa çağıranlarını ayrı sayar.
- Bir endpoint'te **API anahtarı** istemek için gateway'de en az bir anahtar olmalıdır.
- Aynı API'de aynı metot ve yolu karşılayan iki endpoint olamaz. Bir gateway toplamda en fazla 300 endpoint taşır.
- Endpoint'i silmek için API panelindeki endpoint listesinde silme düğmesine tıklayın ve "Endpoint politikası kaldırılsın mı?" penceresinde **Sil** ile onaylayın. Uyguladıktan sonra o istekler API'nin varsayılan politikalarını kullanır.

---

## OpenAPI İçe Aktarma ve Referans

**İçe aktarma.** Elinizde bir OpenAPI ya da Swagger tanımı varsa endpoint'leri tek tek eklemek yerine içe aktarabilirsiniz:

1. API'yi seçin ve **OpenAPI içe aktar**'a tıklayın.
2. **Dosya seç** ile tanımı seçin (JSON ya da YAML, 2 MB'tan küçük).
3. Tanımdaki zorunlu alan ve başlıkları da doğrulama kuralı olarak almak istiyorsanız **Tanımın belirttiği istek kurallarını da getir**'i işaretleyin. Yalnızca düz tipli zorunlu alanlar ve başlıklar alınır; bu kurallar uyguladığınız anda trafiği reddetmeye başlar.
4. **Bunun ne ekleyeceğini gör** ile önizleyin, sonra **{sayı} endpoint ekle**'ye tıklayın ve değişiklikleri uygulayın.

Yalnızca API'nin henüz sunmadığı endpoint'ler eklenir; mevcut endpoint'ler ayarlarını korur ve hiçbir şey silinmez. Gateway'in yönlendirmediği metotlar ya da ifade edemediği yollar önizlemede **reddedildi** olarak işaretlenir. Dosya kendi kendine yeterli olmalıdır: başka bir dosyaya ya da URL'ye işaret eden tanım reddedilir; referansları çözülmüş olarak dışa aktarın.

İçe aktarılan tanım API'de saklanır ("Kayıtlı tanım: …") ve **{biçim} indir** ile geri indirilebilir.

**Referans.** **Referansı indir (OpenAPI)**, API'nin endpoint'lerinden ve politikalarından üretilen bir OpenAPI belgesi indirir: gateway altındaki yollar, kimlik doğrulama ve gateway'in kendisinin verebileceği yanıtlar (`400`, `401`, `403`, `429`, `503`). Çağıranlarınıza verebileceğiniz belge budur. Tüm API'lerin referansını bir bağlantıyla yayımlamak için [Herkese açık API referansı](api-gateway.md#çalışma-zamanı) ayarını kullanın.

---

## Çağıranlar ve Anahtarlar

Çağıran, tek bir API anahtarı taşıyan istemcidir; bir müşteriniz, iş ortağınız ya da uygulamanız. Çağıranlar gateway'in **Anahtarlar** sekmesindeki **Çağıranlar** listesinde görünür: ad, anahtarın baş kısmı, erişebildiği API'ler ve durum (**aktif**, **süresi doldu**, **iptal edildi**, **API yok**).

Çağıranlar gateway yapılandırmasının parçasıdır: yeni, düzenlenmiş ya da iptal edilmiş bir çağıran trafiğe bir sonraki uygulamada yansır. **Anahtarlar** sekmesini gateway'i düzenleme yetkisi olan üyeler ve **API gateway çağıranlarını ve kullanım planlarını yönet** yetkisi verilen üyeler görür.

### Çağıran Oluşturma

1. **Anahtarlar** sekmesinde **Yeni çağıran**'a tıklayın.
2. Alanları doldurun:
   - **Ad** — çağıranı tanımanızı sağlar (örneğin "mobil uygulama").
   - **İletişim** — limitini değiştirmeden ya da anahtarını iptal etmeden önce kime haber verileceği.
   - **Müşteri kimliği** — isteğe bağlı. Servisleriniz bu çağıranın her isteğinde bunu `X-Consumer-Custom-ID` başlığıyla alır. En fazla 128 karakter: harf, rakam ve `. _ : @ + = / ~ -`; ilk karakter harf ya da rakam olmalı.
   - **Son kullanma** — isteğe bağlı. Bkz. [Son Kullanma](#son-kullanma).
   - **Kullanım planı** — çağıranı bir plana bağlar ya da **Plan yok — aşağıdaki kendi limitleri** seçilir.
   - **Kendi hız limiti**, **Hacim hakkı**, **Hangi API'leri çağırabileceğini sınırla** — bkz. [Çağıranların Limitleri](#çağıranların-limitleri).
3. **Kaydet**'e tıklayın. Anahtar gösterilir: "Bu anahtarı şimdi kopyalayın — tekrar okuyabileceğiniz hiçbir yerde saklanmıyor." Kopyalayıp **Tamam**'a tıklayın.
4. Değişiklikleri uygulayın. Anahtar ancak uygulamadan sonra çalışır.

Anahtarlar `kmt_gw_` ile başlar. Çağıranlarınız anahtarı her istekte `X-Api-Key` başlığında gönderir.

> **Uyarı:** Anahtar yalnızca kaydettiğiniz anda bir kez gösterilir ve sonradan geri okunamaz. Kaybedilen bir anahtar için [anahtarı yenileyin](#anahtarı-yenileme).

### Son Kullanma

**Son kullanma** alanına bir an girerseniz gateway bu andan yaklaşık bir dakika sonra anahtarı reddetmeye başlar ve tarihi uzatana ya da silene kadar reddeder. İptal etmenin aksine geri alınabilir. Uygulanmamış değişiklikleriniz varsa anahtar bir sonraki uygulamaya kadar kabul edilmeye devam eder. Geçmiş bir an girilemez. Saatler ayarlarınızda seçili saat dilimindedir.

### Anahtarı Yenileme

Çağıranın satırındaki **Anahtarı yenile** ile yeni bir anahtar üretilir. Çağıran geri kalan her şeyini korur: limitlerini, hakkından kullandığını, müşteri kimliğini ve API'lerini.

**Eski anahtar şu süre daha çalışsın** seçeneğiyle geçiş süresini belirleyin: **Hiç — eski anahtar bir sonraki uygulamada durur**, **1 saat**, **1 gün**, **7 gün** ya da **30 gün**. Geçiş süresi boyunca iki anahtar da çalışır ve aynı hakkı tüketir; listede eski anahtar "{önek}… şu tarihe kadar çalışır: {tarih}" olarak görünür. Eski anahtar sızdıysa **Hiç**'i seçin. Yeni anahtar bir sonraki uygulamadan itibaren çalışır.

### Anahtarı İptal Etme ve Duraklatma

Çağıranın satırında **İptal et**'e, ardından **Anahtarı iptal et**'e tıklayın. İptal kalıcıdır: anahtar bir sonraki uygulamada çalışmayı bırakır, bir daha gösterilemez ve geri getirilemez.

- Anahtar isteyen bir API ya da endpoint varken son aktif anahtar iptal edilemez: "Bu tek aktif anahtar ve en az bir API'nin ona ihtiyacı var. Önce yeni bir anahtar oluşturun, sonra bunu iptal edin."
- Bir çağıranı geçici olarak durdurmak için iptal etmek yerine düzenleyin ve erişebileceği tüm API'leri kaldırın. Çağıran her yerde reddedilir, anahtarı geçerli kalır; listede **API yok** etiketi görünür.

### Çağıranların Limitleri

| Ayar | Davranış |
|---|---|
| **Kendi hız limiti** | Bu çağıran için saniyedeki istek; anahtar isteyen her API'de geçerlidir. O API'nin kendi limitinin yerine geçer, daha yüksek olsa bile. Kapalıyken "Her API'nin belirlediğini izler". |
| **Hacim hakkı** | **24 saatte** ya da **30 günde** gönderilebilecek istek sayısı. Kayan pencerede sayılır: takvim günü ya da ayı değil, son 24 saat ya da son 30 gün. Önce bir [sayaç deposu](#hız-limitlerinin-sayılması) seçilmelidir. |
| **Hangi API'leri çağırabileceğini sınırla** | Kapalıyken çağıran gateway'deki her API'yi çağırabilir. Açıkken yalnızca seçtiğiniz API'leri çağırır; diğerlerinde `403` alır. |

> **Uyarı:** Bir çağırana hem kendi hız limiti hem hacim hakkı verilemez; birini açmak diğerini kapatır. Hacim hakkı verilen çağıranın saniyelik limiti hiç olmaz — ne kendisinin ne de API'lerinin. Hak, hız limitinin üstüne eklenmez, yerine geçer.

Hacim hakkı değişiklikleri uygulanırken gateway yeni bir pod ile yeniden başlar (yaklaşık 30–60 saniye).

---

## Kullanım Planları

Kullanım planı, birçok çağırana aynı limiti ve aynı API'leri veren adlandırılmış ayardır. Planı bir kez değiştirirsiniz; bağlı tüm çağıranlar bir sonraki uygulamada izler.

1. **Anahtarlar** sekmesindeki **Kullanım planları** kartında **Yeni plan**'a tıklayın.
2. **Ad** girin (örneğin "Partner").
3. **Her çağıran için limit** seçin: **Yok — her API'nin kendi ayarını izle**, **Saniyedeki istek sayısı** ya da **Hacim hakkı** (24 saat ya da 30 gün; her çağıran için ayrı sayılır; sayaç deposu gerekir).
4. İsterseniz **Hangi API'leri çağırabileceğini sınırla** ile API'leri seçin.
5. **Kaydet**'e tıklayın, çağıranları bu plana bağlayın ve değişiklikleri uygulayın.

Plana bağlı bir çağıranın kendi limitleri ve API listesi olmaz; planınkileri kullanır. Bağlı çağıranı olan bir plan silinemez; önce çağıranları başka bir plana ya da kendi limitlerine taşıyın.

---

## Hız Limitlerinin Sayılması

Hız limitlerinin ve hacim haklarının nerede sayılacağı **Ayarlar** sekmesindeki **Sayaçların tutulduğu yer** kartında seçilir:

| Seçim | Sonuç |
|---|---|
| **Yok — replika başına sayılır, hak yok** (varsayılan) | Her replika limitleri kendi başına sayar; N'lik bir limit toplamda N × (en fazla replika) isteğe kadar izin verir. Hacim hakkı verilemez. |
| Valkey örneklerinizden biri | Tüm hız limitleri bu örnekte, tüm replikalar genelinde tek sayaçla sayılır; ölçekleme limitin ne kadar geçirdiğini değiştirmez. Hacim hakları kullanılabilir. Limitli her istek bu örneğe bir gidiş dönüş yapar. Replika tavanı 20'ye çıkar. |

- Valkey örneği yoksa önce bir tane oluşturun ([Valkey](managed-valkey.md)); örnek ayrı faturalanır. Henüz çalışmayan bir örnek seçilemez.
- Seçimi kaydettikten sonra değişiklikleri uygulayın. Uygulanana kadar her replika kendi başına saymaya devam eder.
- Sayaçlar yazıldıkları örnekte kalır. Depoyu değiştirmek ya da kaldırmak çağıranların ve planların hacim haklarını kaldırır; konsol bunu kaydetmeden önce onaylatır.
- Valkey örneğinin parolası döndürüldüyse **Kimlik bilgisini yenile**'ye tıklayın ve değişiklikleri uygulayın.

> **Uyarı:** **Sayaç deposuna ulaşılamıyor** uyarısı görünüyorsa, bu sürdüğü sürece haklar ve hız limitleri uygulanmaz; trafik reddedilmek yerine geçirilir ve gateway'deki kayıtlar reddedilir. Valkey örneğinin çalıştığından emin olun, sonra **Kimlik bilgisini yenile**'ye tıklayın.

---

## Kullanım

**Anahtarlar** sekmesindeki **Kullanım** kartı her çağıranın son **24 saat** ve **7 gün** içindeki istek sayısını, **Reddedilen (429)** istekleri ve hacim hakkı varsa ne kadarını kullandığını ve ne zaman sıfırlanacağını gösterir. İstek sayıları gateway'in metriklerinden gelir ve yaklaşıktır; hak kullanımı gateway'in uyguladığı sayaçtan okunur ve kesindir.

Sayaç deposu seçilmemişse kartta "Ayarlarda bir Valkey örneği seçilmedi, bu yüzden haklar sayılmıyor." yazar.

---

## Çağıranlarınızın Aldığı Yanıtlar

Gateway bir isteği servisinize iletmeden reddettiğinde şu yanıtları verir:

| Kod | Ne zaman | Gövde |
|---|---|---|
| `400` | İstek bir doğrulama kuralına uymuyor. | Takılan alanı belirten bir hata mesajı. |
| `401` | Anahtar isteyen API'de `X-Api-Key` yok, geçersiz, süresi dolmuş ya da iptal edilmiş; JWT isteyen API'de token yok ya da geçersiz. | — |
| `403` | Çağıranın bu API'yi çağırma izni yok; token gerekli scope'ları taşımıyor ya da IP kuralı isteği engelliyor. | — |
| `404` | Gateway'de bu yolda bir API ya da rota yok. | `{"error_msg":"404 Route Not Found"}` |
| `413` | İstek gövdesi gateway'in **En büyük istek gövdesi** sınırını aşıyor. | `{"error":"Gateway error","gateway":"Komuta"}` |
| `429` | Hız limiti aşıldı. | `{"error_msg":"Too many requests. X-RateLimit-Reset says when to retry."}` |
| `429` | Çağıranın hacim hakkı doldu. | `{"error_msg":"This caller's allowance for the window is used up. X-RateLimit-Reset says when it renews."}` |
| `503` | API bakımda. | `{"error":"maintenance","message":"<mesajınız>"}` |
| Seçtiğiniz kod (varsayılan `503`) | Circuit breaker endpoint'i duraklattı. | — |
| `502`, `503`, `504` | Gateway servisinizden yanıt alamadı (örneğin zaman aşımı doldu). | `{"error":"Gateway error","gateway":"Komuta"}` |

- `404` ve `429` gövdeleri JSON metnidir ama `Content-Type` başlığı `text/plain` olabilir; istemcinizde gövdeyi buna göre okuyun.
- Servisinizin kendi döndürdüğü hata yanıtları (örneğin kendi `500` gövdeniz) değiştirilmeden çağırana ulaşır.

Gateway'in eklediği yanıt başlıkları:

| Başlık | Ne zaman |
|---|---|
| `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` | Hız limiti ya da hacim hakkı olan rotalarda. `Retry-After` gönderilmez; ne zaman yeniden deneneceğini `X-RateLimit-Reset` söyler. |
| `X-Komuta-Cache` | Önbelleği açık rotalarda (`HIT`, `MISS`, `EXPIRED`). |
| `X-Request-Id` | **İstek kimliği başlığı** ayarı açıksa her yanıtta. |
| `Deprecation`, `Sunset`, `Link` | Kullanımdan kaldırma duyurusu olan API'lerde. |

---

## API'yi Kaldırma

1. **API'ler** sekmesindeki listede API'nin yanındaki çöp kutusu düğmesine ya da API panelindeki **API bağlantısını kaldır**'a tıklayın.
2. "API bağlantısı kaldırılsın mı?" penceresinde **API bağlantısını kaldır**'a tıklayın.
3. Değişiklikleri uygulayın.

Uyguladıktan sonra API'nin yolu artık yönlendirilmez ve oraya gelen istekler `404` alır. API'nin endpoint'leri ve saklanan tanımı da kaldırılır. Hedef servis silinmez.

---

## Sınırlar

| Konu | Sınır |
|---|---|
| Gateway başına API | 40 |
| Gateway başına endpoint | 300 |
| Dış erişim yolu | En fazla 64 karakter |
| Hız limiti (konsol) | 0–10.000 istek/saniye |
| Önbellek saklama süresi (konsol) | 1–3600 saniye |
| Yeniden deneme | 0–3 |
| Gerekli scope'lar | En fazla 16, her biri 1–64 karakter |
| Bakım mesajı | En fazla 512 karakter |
| Duyuru bağlantısı | En fazla 512 karakter |
| Başlık kuralı | Ad en fazla 64, değer en fazla 512 karakter |
| Müşteri kimliği | En fazla 128 karakter |
| Anahtar geçiş süresi | En fazla 30 gün |
| İçe aktarılan OpenAPI dosyası | 2 MB'tan küçük |

Gateway'in tüm yapılandırmasının da bir boyut sınırı vardır. Aşılırsa kayıt, ne kadar aşıldığını söyleyen bir mesajla reddedilir; en çok yer kaplayan kalemler doğrulama alan kuralları ve başlık kurallarıdır. Her endpoint'te aynı olan kuralları API düzeyine taşıyın. Gateway geneli sınırlar [API Gateway](api-gateway.md#sınırlar) sayfasındadır.

---

## İlgili Dokümanlar

- [API Gateway](api-gateway.md) — Gateway oluşturma, değişiklikleri uygulama, izleme ve ayarlar.
- [Valkey](managed-valkey.md) — Sayaç deposu olarak kullanılan Valkey örnekleri.
- [Erişim ve Portlar](services-ports.md) — Bir servisi gateway'e bağlamadan önce port eklemek.
- [Dağıtım Stratejileri](deployment-strategies.md) — Canary ve blue-green dağıtımlar.
- [Ortam Değişkenleri](services-environment-variables.md) — Servislerin ortam değişkenleri.
