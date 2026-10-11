# API Gateway

API Gateway, Komuta'daki servislerinizin önüne tek bir adres koyar. Her servisi kendi yolu altında (`/orders`, `/payments` gibi) yayınlar; istekleri API anahtarı ya da JWT ile doğrular, hız limiti ve kota uygular, isterseniz yanıtları önbelleğe alır. Gateway'i Komuta çalıştırır: repo, Dockerfile ya da build gerekmez.

Şu durumlarda işinize yarar:

- Birden fazla servisi tek bir adres ya da kendi alan adınız altında sunmak istiyorsanız.
- API'nizi müşterilere veya iş ortaklarına anahtarla açıp her birine ayrı limit vermek istiyorsanız.
- Servisinize ulaşmadan önce bozuk istekleri reddetmek, sürekli hata veren bir endpoint'i duraklatmak ya da bir API'yi bakıma almak istiyorsanız.

Tek bir servisi kimlik doğrulaması olmadan yayınlamak için gateway gerekmez; servisin kendi adresi yeterlidir.

Bu sayfa gateway'in kurulumunu ve yönetimini anlatır. API bağlama, endpoint'ler, anahtarlar, limitler, politikalar ve çağıranlarınızın aldığı hata yanıtları [API Gateway: API'ler, Anahtarlar ve Politikalar](api-gateway-apis.md) sayfasındadır.

---

## Temel Kavramlar

| Kavram | Anlamı |
|---|---|
| **Gateway** | Komuta'nın sizin için çalıştırdığı, kendi adresi olan servis. Bir projeye ve bölgeye aittir. |
| **API** | Gateway'e bir yol önekiyle (`/orders` gibi) bağlanan Komuta servisi. |
| **Endpoint** | Bir API'nin altındaki tek bir yol ve metot. Kendi önbellek, limit, zaman aşımı ve doğrulama ayarlarını alabilir. |
| **Çağıran** | Tek bir API anahtarı taşıyan istemci; örneğin bir müşteriniz ya da mobil uygulamanız. |
| **Kullanım planı** | Birçok çağırana aynı limitleri ve aynı API'leri veren adlandırılmış ayar. |
| **Uygulama** | Kaydettiğiniz değişikliklerin çalışan gateway'e aktarılması. Gateway'de yapılan değişiklikler siz uygulayana kadar trafiğe yansımaz. |

---

## Gateway Oluşturma

1. **Yönetilen Servisler → API Gateway'ler** sayfasını açın ([console.komuta.io/api-gateways](https://console.komuta.io/api-gateways)) ve **Yeni gateway**'e tıklayın.
2. **Yeni API Gateway** penceresini doldurun:
   - **İsim** — gateway'in adı.
   - **Proje** — gateway'in ait olacağı proje.
   - **Bölge** — gateway'in çalışacağı bölge. Şu an tek bölge vardır: Germany-East (`de-east`).
   - **Boyut** — gateway'in CPU ve bellek paketi. Servislerde kullanılan paketlerin aynısıdır.
3. Boyutu seçince altında ücret notu görünür. **Gateway oluştur**'a tıklayın.
4. **Gateway'inizin ilk API anahtarı** penceresinde anahtarı kopyalayıp güvenli bir yere kaydedin, sonra **Kaydettim**'e tıklayın. Gateway'in sayfası açılır.

Oluşturma başarılı olunca "Gateway oluşturuldu. Şu anda dağıtılıyor." bildirimi görünür. İlk dağıtım bitene kadar durum **Kuruluyor** olarak görünür ve adresin yerinde "adres bekleniyor" yazar.

> **Uyarı:** İlk API anahtarı yalnızca bu pencerede, bir kez gösterilir ve sonradan tekrar görüntülenemez. Anahtarı kaybederseniz **Anahtarlar** sekmesinden yeni bir çağıran oluşturun ya da anahtarı yenileyin. İlk anahtar, **Anahtarlar** sekmesinde `default` adlı çağıran olarak listelenir.

**Yeni gateway** düğmesi yalnızca yönetilen servis oluşturma yetkisi olan üyelere görünür.

### Fiyat

| Kalem | Tutar |
|---|---|
| İşlem gücü | Seçtiğiniz boyut, diğer servisler gibi replika başına faturalanır. Paketler: 0,5 vCPU / 1 GB 9 USD, 1 vCPU / 2 GB 18 USD, 2 vCPU / 4 GB 36 USD, 4 vCPU / 8 GB 72 USD (aylık). |
| Gateway ücreti | Aylık 9 USD (saatlik 0,0125 USD). Oluşturma penceresinde "İşlem gücü diğer servisler gibi faturalanır, ayrıca aylık 9.00 USD gateway ücreti alınır." yazar. |
| Bant genişliği | Varsayılan Starter planı ücretsizdir. Bkz. [Bant Genişliği](#bant-genişliği). |

Fiyatlar USD cinsindendir, vergi hariçtir ve saatlik ölçülür; ay ortasında silinen bir gateway yalnızca çalıştığı saatler için ücretlendirilir. Örneğin 0,5 vCPU / 1 GB paketle tek replika çalışan bir gateway aylık 9 + 9 = 18 USD tutar. Gateway ücreti ekstrelerde **API Gateway** kategorisinde görünür.

Oluştururken işlem gücü ve gateway ücretinin toplamı harcama limitinize göre kontrol edilir; aylık tahmin limiti aşacaksa gateway oluşturulmaz. Ayrıntılar [Faturalandırma](billing.md) sayfasında ve [Fiyatlandırma](https://www.komuta.io/pricing) sayfasındadır.

---

## Gateway Sayfası

Gateway'e tıkladığınızda açılan sayfanın üst kısmında şunlar bulunur:

- **Durum** — **Hazır**, **Kuruluyor**, **Yayınlanıyor**, **Sorunlu**, **Başarısız** ya da **Durduruldu**.
- **Yükselt** — gateway'in boyutunu (paketini) değiştirir.
- **Adres** — gateway'in Komuta adresi; yanındaki düğmeyle kopyalanır.
- **Servisi görüntüle** — gateway'i çalıştıran servisin sayfasını açar.
- **Replika** — replika sayısını ve otomatik ölçeklemeyi ayarlar.
- **Bekleyen değişiklik çubuğu** — uygulanmamış değişiklik varsa görünür. Bkz. [Değişiklikleri Uygulama](#değişiklikleri-uygulama).

| Sekme | İçerik |
|---|---|
| **Genel bakış** | Trafik ve telemetri, örnek istek (**Deneyin**), zaman aşımı, son uygulanma zamanı ve pod'ların gerçekte çalıştırdığı yapılandırma. |
| **API'ler** | Bağlı API'ler ve endpoint'leri. |
| **Anahtarlar** | Çağıranlar, anahtarlar, kullanım planları ve kullanım. Yalnızca gateway'i düzenleme ya da çağıranları yönetme yetkisi olan üyelere görünür. |
| **Domainler** | Gateway'e bağlı özel alan adları. |
| **Bant Genişliği** | Bant genişliği planı ve canlı trafik. |
| **Ayarlar** | Gateway geneli ayarlar, sayaç deposu, sürüm, yapılandırma kopyalama ve silme. |
| **Geçmiş** | Yapılandırma değişiklikleri ve eski bir sürüme dönme. |

**Genel bakış** sekmesindeki **Deneyin** kartı, ilk API'niz için kopyalanabilir bir `curl` komutu gösterir. Örnek: `curl -H 'X-Api-Key: <your key>' https://<gateway-adresi>/orders`.

Gateway'i çalıştıran servis **Uygulamalar** listesinde **API Gateway** etiketiyle görünür. Bu servisin ortam değişkenleri, bağlı API'leri, sağlık kontrolleri, replika sayısı ve kaynakları Komuta tarafından yönetilir; bunları servis sayfasından değil, gateway'in kendi sayfasından değiştirin. Servis tek başına silinemez; gateway'i silin.

---

## Değişiklikleri Uygulama

API, endpoint, ayar, çağıran ya da kullanım planı değişiklikleri önce kaydedilir. Kaydedilmiş ama uygulanmamış değişiklik varsa sayfanın üstünde bir çubuk ve **Değişiklikleri uygula** düğmesi görünür; çubuk sekmeler arasında gezinirken de kalır. Gateway listesindeki kartta da "uygulanmamış değişiklik" etiketi çıkar.

Çubuk, uygulamanın nasıl olacağını önceden söyler:

| Çubuktaki metin | Ne olur |
|---|---|
| "Uygulanmamış değişiklikleriniz var. Uygulamak bunları birkaç saniye içinde çalışan gateway'e yükler; pod'lar yeniden başlatılmaz." | Değişiklikler çalışan gateway'e yerinde yüklenir. |
| "Uygulanmamış değişiklikleriniz var. Uygulamak gateway'i yeni bir pod ile yeniden başlatır, çünkü şunlar değişti: …" | Yeni bir pod başlar; eskisi yenisi hazır olana kadar hizmet verir. Geçiş yaklaşık 30–60 saniye sürer. |

Yeniden başlatma gerektiren değişiklikler: motor sürümü, istek gövdesi sınırı, yanıt önbelleği boyutu, gateway'in açılışta okuduğu değişkenler, sayaç deposunun şifresi ve çağıranların kotaları (hacim hakları).

**Değişiklikleri uygula**'ya tıkladıktan sonra çubukta "Uygulanıyor — gateway değişiklikleri yüklüyor." ya da "Uygulanıyor — gateway yeniden başlıyor." yazar. Sonuç olumsuzsa:

| Çubuk | Anlamı ve yapılacak |
|---|---|
| **Değişiklikler uygulanamadı** | Dağıtım başarısız oldu ve geri alındı. Gateway önceki yapılandırmayla çalışmaya devam eder, değişiklikler beklemede kalır. Sebebi düzeltip **Tekrar dene**'ye tıklayın. |
| **Değişiklikler her pod'da onaylanmadı** | Yapılandırma bazı pod'larda ya da hepsinde zaten canlı olabilir; canlıymış gibi davranın. Platform kendiliğinden yeniden kontrol eder. |
| "Uygulama beklenenden uzun sürüyor: on dakika geçti…" | Gateway on dakika içinde yeni yapılandırmayı bildirmedi. Yayının neye takıldığını servis sayfasında görebilirsiniz. **Tekrar dene**, arkadaki dağıtım vazgeçince açılır. |

İlk dağıtım sürerken çubuk gösterilmez.

**Uygulama gerektirmeyenler:** Replika ayarı ve bant genişliği planı kaydedildiği anda geçerli olur. Son kullanma tarihi geçen bir anahtar, uygulama beklemeden yaklaşık bir dakika içinde reddedilmeye başlar (bekleyen başka değişiklik varsa bir sonraki uygulamaya kadar kabul edilmeye devam eder).

**Kim uygulayabilir:** Gateway'i düzenleme yetkisi olan her üye. Yalnızca **API gateway çağıranlarını ve kullanım planlarını yönet** yetkisi olan bir üye, bekleyen değişikliklerin hepsi çağıran ya da kullanım planı değişikliğiyse uygulayabilir; değilse çubukta "Bu değişiklikleri, bu gateway'i düzenleme yetkisi olan biri uygular." yazar.

### Pod'ların Çalıştırdığı Yapılandırma

**Genel bakış** sekmesindeki **Pod'lar ne servis ediyor** kartında **Kontrol et**'e tıklayın. Komuta, kayıtlı olana güvenmek yerine her pod'a hangi yapılandırmayı çalıştırdığını ve hangi rotaların yüklü olduğunu sorar. "Her pod, uygulandı olarak kaydedilen yapılandırmayı servis ediyor." yazıyorsa her şey yerindedir.

---

## Trafik ve İzleme

**Genel bakış** sekmesinin **Trafik ve telemetri** bölümünde beş sekme vardır:

| Sekme | Gösterdiği |
|---|---|
| **Trafik** | **API'ye göre trafik**: her API için istek/sn, 4xx/sn, 5xx/sn ve p95 gecikme. Zaman aralığı **Son 15dk**, **Son 1sa**, **Son 6sa** ya da **Son 24sa** seçilir; veriler 30 saniyede bir yenilenir. Yanında **Çağırana göre trafik** ve önbelleği açık API'ler varsa **Yanıt önbelleği** doluluğu görünür. |
| **Loglar** | Gateway servisinin logları. Bkz. [Loglar](service-logs.md). |
| **İçgörüler** | Gateway'in çalışma zamanı içgörüleri. |
| **Kaynaklar** | CPU ve bellek kullanımı. |
| **Uyarılar** | Gateway'e ait uyarı kuralları. |

API başına hızlar 5 dakikalık pencerede ölçülür; sessiz ya da az önce yeniden başlamış bir gateway bu tabloda boş görünebilir. **Çağırana göre trafik** yalnızca anahtar isteyen API'lerde dolar; herkese açık ve JWT kullanan API'ler çağıranı tanımlamaz.

**Loglar**, **İçgörüler**, **Kaynaklar** ve **Uyarılar** gateway'in servisi üzerinden okunur. Hesabınızın o servisi okuma yetkisi yoksa bu sekmeler boş kalır ve bir uyarı görünür.

**Uyarılar.** Komuta her gateway için iki uyarı kuralı ekler: `<servis adı> - API 5xx Ratio High` (bir rotada 5 dakika boyunca yanıtların %5'inden fazlası 5xx ve rota saniyede 0,1'den fazla istek taşıyor) ve `<servis adı> - Callers Throttled` (bir rota 5 dakika boyunca saniyede 1'den fazla isteği 429 ile reddediyor). Kendi kurallarınız için **API Gateway** kategorisindeki şablonları kullanabilirsiniz; bkz. [Şablonlar, Metrikler ve Loglar](alerts-templates.md).

**Bildirim.** Bir uygulama başarısız olduğunda ya da gateway bazı rotaları servis etmediğinde Komuta **API gateway yapılandırma sorunu** bildirimi gönderir. Bildirimin nereye gideceğini [Kanallar ve Bildirim Teslimi](notification-guide.md) sayfasından ayarlayın.

Çağıran başına istek sayıları ve hak kullanımı **Anahtarlar** sekmesindeki **Kullanım** kartındadır; bkz. [Kullanım](api-gateway-apis.md#kullanım).

---

## Ölçek ve Boyut

**Replika sayısı.** Sayfanın üstündeki **Replika** düğmesine tıklayın:

1. Sabit sayı için **Yüke göre otomatik ölçekle** kapalıyken **Replika sayısı**'nı girin.
2. Otomatik ölçekleme için düğmeyi açıp **En az** ve **En fazla** değerlerini girin.
3. **Kaydet**'e tıklayın.

Bu ayar "Hemen uygulanır — gateway'in kayıtlı konfigürasyonunun parçası değildir, apply beklemez." Bir gateway en az 1, en fazla 10 replika çalıştırabilir. Hız limitleri bir Valkey sayaç deposunda sayılıyorsa ve bu ayar uygulanmışsa tavan 20 replikaya çıkar. Her replika işlem gücü olarak ayrıca faturalanır.

> **Uyarı:** Sayaç deposu seçilmemişse her replika hız limitini kendi başına sayar; N'lik bir limit toplamda saniyede N × (en fazla replika) isteğe kadar izin verir. Pencere bunu ayrıca söyler. Bkz. [Hız Limitlerinin Sayılması](api-gateway-apis.md#hız-limitlerinin-sayılması).

**Boyut.** Daha fazla CPU ya da bellek için sayfanın üstündeki **Yükselt**'e tıklayıp yeni paketi seçin. Yanıt önbelleği gateway'in belleğinden harcandığı için büyük bir önbellek daha büyük bir paket ister. Otomatik ölçeklemenin genel işleyişi için bkz. [Otomatik Ölçekleme](services-autoscaling.md).

---

## Bant Genişliği

**Bant Genişliği** sekmesi gateway'in plan özetini, **Canlı Trafik** grafiğini ve kullanım göstergelerini gösterir. Grafik siz sekmedeyken örnek toplar; sekmeyi açık tutun.

Planı değiştirmek için listeden yeni planı seçin ve onaylayın. Plan değişikliği hemen geçerli olur ve trafiği kesmez.

| Plan | Aylık fiyat | Çıkış hızı sınırı |
|---|---|---|
| Starter (varsayılan) | Ücretsiz | 5 Mbit/s |
| Growth | 4,99 USD | 25 Mbit/s |
| Business | 14,99 USD | 100 Mbit/s |
| Enterprise | 49,99 USD | 500 Mbit/s |
| Scale | 149,99 USD | 2 Gbit/s |

Ayrıntılar [Bant Genişliği Planları](bandwidth-plans.md) sayfasındadır.

---

## Alan Adları

Gateway oluşturulduktan sonra kendi Komuta adresinden erişilebilir; özel alan adı isteğe bağlıdır. Tüm API'lerinizi kendi alan adınız altında sunmak için:

1. **Domainler** sekmesinde **Alan adı ekle**'ye tıklayın; **Alan Adları** sayfası açılır.
2. Alan adını ekleyip doğrulayın ve gateway'in servisine bağlayın.

Bağlanan alan adları **Domainler** sekmesinde durumlarıyla listelenir. Ayrıntılar [Özel Alan Adları](custom-domains.md) sayfasındadır.

---

## Ayarlar

**Ayarlar** sekmesinde yukarıdan aşağıya şu kartlar bulunur: **Gateway sürümü**, **Sayaçların tutulduğu yer**, **Çalışma zamanı**, **JWT doğrulama**, **CORS**, **Yapılandırmayı kopyala** ve **Tehlikeli bölge**. **Sayaçların tutulduğu yer** kartı [Hız Limitlerinin Sayılması](api-gateway-apis.md#hız-limitlerinin-sayılması) bölümünde anlatılır.

Ayarları değiştirip **Kaydet**'e tıklayın; ardından değişiklikleri uygulayın. Siz düzenlerken biri aynı ayarları kaydederse **Bu ayarlar başka bir yerde değiştirildi** uyarısı çıkar; kaydettiğinizde sizin değişiklikleriniz onların üzerine yazılır.

### Çalışma Zamanı

| Ayar | Açıklama |
|---|---|
| **Görünen ad** | Gateway'in konsolda görünen adı. |
| **Zaman aşımı** | Gateway'in API'lerinizi ne kadar bekleyeceği. Varsayılan `10s`. `500ms`, `45s` ya da `2m` gibi yazılır; en fazla 300 saniyedir, sıfır kabul edilmez. API ve endpoint düzeyinde ayrıca değiştirilebilir. |
| **İstek kimliği başlığı** | Her yanıta, çağıranlarınızın destek talebinde kullanabileceği bir `X-Request-Id` ekler. Varsayılan kapalı. |
| **İstek izleme** ve **İz örnekleme (%)** | Her istek için iz kaydı tutar; örnekleme 1–100 arasıdır. İzleme gateway'iniz için açılamıyorsa kaydederken bu söylenir. |
| **API anahtarını servislerinize ilet** | Kapalıyken (varsayılan) gateway `X-Api-Key` başlığını kabul ettikten sonra kaldırır; anahtar kodunuza ve loglarınıza ulaşmaz. Servisleriniz çağıranı `X-Consumer-Username` ve `X-Consumer-Custom-ID` başlıklarından öğrenir. Yalnızca anahtarı hâlâ kendisi okuyan bir servis için açın. |
| **Herkese açık API referansı** | Gateway'deki tüm API'lerin referansını (yollar, kimlik doğrulama, istek ve yanıt alanları) bağlantıya sahip herkes için yayımlar. Bağlantı kaydettikten sonra düğmenin altında görünür. Anahtarları ve servislerin nerede çalıştığını göstermez. Kapatmanın etkili olması bir dakikayı bulabilir. Varsayılan kapalı. |
| **Yanıt önbelleği boyutu (MiB)** | 8–512 MiB, varsayılan 32. Önbelleği açık tüm API'ler paylaşır; pod başına tutulur ve paketin belleğinden harcanır. Paketin taşıyabileceğinden büyük bir değer reddedilir. Değişikliği uygulamak gateway'i yeniden başlatır. |
| **Önbelleğe alınacak en büyük yanıt (KiB)** | 64–32768 KiB, varsayılan 512; önbellek boyutunun dörtte birini aşamaz. Daha büyük yanıtlar önbelleğe alınmadan geçer. |
| **En büyük istek gövdesi (MiB)** | 1–100 MiB, varsayılan 10. Daha büyük gövdeli istekler servisinize ulaşmadan 413 ile reddedilir. Servisinizin kendi sınırı da geçerli kalır. Değişikliği uygulamak gateway'i yeniden başlatır. |

### JWT Doğrulama

Bir API'de JWT kimlik doğrulaması kullanmadan önce bu kartı doldurun.

| Ayar | Açıklama |
|---|---|
| **OIDC discovery URL** | Kimlik sağlayıcınızın `/.well-known/openid-configuration` ile biten discovery adresi (anahtar kümesinin adresi değil). JWT kullanan bir API varken boşaltılamaz. |
| **JWT audience** | İsteğe bağlı. Token'ların `aud` değeriyle eşleşmeli. |
| **JWT issuer** | İsteğe bağlı. Sağlayıcınızın `iss` alanına yazdığı `https://` adresi. |
| **İmzalama algoritması** | Varsayılan RS256. RS384, RS512, PS256, PS384, PS512, ES256, ES384, ES512 ve EdDSA da seçilebilir. HS256 gibi paylaşılan sırra dayalı algoritmalar kabul edilmez. |

### CORS

| Ayar | Açıklama |
|---|---|
| **İzin verilen kaynaklar** | Virgülle ayrılmış liste. Yeni gateway'de `*` (tüm kaynaklar) gelir. Boş bırakırsanız hiçbir kaynak arası isteğe izin verilmez. `*` ile adlandırılmış kaynaklar aynı listede kullanılamaz. |
| **İzin verilen başlıklar** | Yeni gateway'de `Origin`, `Authorization`, `Content-Type`, `Accept`, `X-Api-Key` ve `X-Requested-With` gelir. |

### Gateway Sürümü

**Gateway sürümü** kartı çalışan sürümü gösterir. Yeni bir sürüm çıktığında kartta "{sürüm} sürümü mevcut" yazar; **{sürüm} sürümüne yükselt** ile geçebilirsiniz. Yükseltme gateway'i pod pod yeniden başlatır; süren istekler tamamlanır. Daha eski bir sürüme dönmek kendi başınıza yapabileceğiniz bir işlem değildir.

### Yapılandırmayı Kopyalama

**Yapılandırmayı kopyala** kartı, bir gateway'in ayarlarını, API'lerini, endpoint'lerini ve kullanım planlarını başka bir gateway'e taşır; örneğin staging'den production'a.

1. Kaynak gateway'de **İndir**'e tıklayın; bir `.json` dosyası iner.
2. Hedef gateway'de **Dosya içe aktar**'a tıklayıp dosyayı seçin (2 MB'tan küçük olmalı).
3. İsterseniz **Gateway ayarlarını da al** (zaman aşımları, CORS, JWT, izleme, önbellek, anahtar iletimi) ve **Kullanım planlarını da al** seçeneklerini işaretleyin. Hedef gateway kendi adını ve herkese açık referans ayarını korur.
4. Yeni API'ler için hedef servisi seçin, **Önizle** ile neyin ekleneceğini, güncelleneceğini ya da dışarıda kalacağını görün ve **İçe aktar**'a tıklayın.
5. Değişiklikleri uygulayın.

Aynı öneke sahip API'ler yerinde güncellenir ve yönlendirdikleri servisi korur. Dosyada olmayan hiçbir şey silinmez. Anahtarlar hiçbir zaman taşınmaz; her gateway kendi anahtarlarını üretir.

---

## Değişiklik Geçmişi

**Geçmiş** sekmesi, yapılandırmanın her kaydını kimin yaptığıyla birlikte en yeniden eskiye listeler (son 50 kayıt). Kayıtlar 180 gün saklanır.

| Kayıt | Anlamı |
|---|---|
| **Yapılandırma üretildi** | Bir değişiklik kaydedildi. |
| **Uygulama istendi** | Biri **Değişiklikleri uygula**'ya tıkladı. |
| **Uygulandı** | Gateway yeni yapılandırmayla sağlıklı çalışıyor. |
| **Uygulama doğrulanmadı** | Pod'ların hepsi değişikliği onaylamadı. |
| **Uygulama başarısız** | Dağıtım başarısız oldu ve geri alındı. |
| **Platform yeniden üretti** | Komuta yapılandırmayı kendi taraması sırasında yeniden üretti. |
| **Gateway imajı taşındı** | Gateway yeni bir sürüme geçti. |

"Komuta" imzalı satırlar platformun kendi işidir.

**Eski bir sürüme dönmek için:**

1. İlgili satırda **Değişiklikler**'e tıklayın. **Bu değişiklik** sekmesi o kayıtta neyin değiştiğini, **Bu sürüme dönmek** sekmesi geri dönmenin bugünkü yapılandırmada neyi değiştireceğini gösterir.
2. **Bu sürüme dön**'e tıklayın. Değişiklik kaydedilir ama uygulanmaz.
3. Değişiklikleri uygulayın.

Geri dönüş çağıranlara ve anahtarlarına dokunmaz. O tarihten sonra eklenen ya da silinen API'ler, sayaç deposu ve API'lerin yönlendirdiği servisler de olduğu gibi kalır. Dönüşümlere yazılmış başlık değerleri geçmişte `•1`, `•2` olarak gizlenir.

---

## Sorun Giderme

Gateway beklendiği gibi çalışmadığında sayfanın üstünde bir uyarı kartı çıkar:

| Kart | Anlamı ve yapılacak |
|---|---|
| **Gateway uykuda** | Gateway yeterince uzun süre boşta kaldığı için uyutuldu. Trafik ya da değişiklik uygulamak onu uyandırır; uyanmadan sonraki ilk istek bekler. |
| **Komuta, art arda çökmeler nedeniyle bu gateway'i durdurdu** | Pod art arda yeniden başladığı için gateway sıfır replikaya indirildi ve adresi yanıt vermez. Sebebi giderip değişiklikleri uygulayın; gateway yeni bir pod ile başlar. |
| **Hiçbir gateway pod'u hazır değil** | Pod'lar çalışıyor ama hiçbiri hazır değil; istekler şu an yanıtlanmıyor. Ayrıntılar servis sayfasındadır. |
| **Son yayın geri alındı** | Yeni sürüm çöktüğü için platform önceki sürümü hizmette tuttu. |
| **Bazı rotalar servis edilmiyor** | Pod sağlıklı görünse de bazı rotalara gelen istekler 404 alıyor; gateway o rotadaki bir politikayı yükleyemedi. Rotanın politikalarını ve gateway sürümünü kontrol edip yeniden uygulayın. |

**API'ler** sekmesinde "N API yapılandırılmış ama servis edilmiyor" uyarısı görürseniz o API'lere gelen istekler 404 alır. Olağan sebepler: hedef servisin silinmiş olması, gateway'in hedef adresi çözememesi, platformun uygulayamadığı bir IP kuralı ya da hakkı sayılamayan bir çağıran. Uyarıdaki API'leri, **Anahtarlar** sekmesini ve **Ayarlar**'ı kontrol edin.

Genel sorun giderme adımları için [Sorun Giderme](troubleshooting.md) sayfasına bakın.

---

## Gateway'i Silme

1. **Ayarlar** sekmesinde **Tehlikeli bölge** kartına gidin ve **Gateway'i sil**'e tıklayın.
2. Onay penceresine gateway'in adını yazın ve **Sil**'e tıklayın.

Silme işlemi gateway'in yapılandırmasını, API anahtarlarını ve gateway'i çalıştıran servisi kaldırır; bağlı özel alan adları serbest bırakılır. Gateway'e bağlı hedef servisleriniz ve sayaç deposu olarak seçtiğiniz Valkey örneği silinmez. Gateway ücreti silindiği saatten sonra işlemez.

> **Uyarı:** Bu işlem geri alınamaz. Anahtarlar da silindiği için aynı adla yeni bir gateway oluştursanız bile çağıranlarınıza yeni anahtar vermeniz gerekir.

Silme yetkisi yalnızca gateway'i düzenleyebilen üyelerdedir.

---

## Sınırlar

| Konu | Sınır |
|---|---|
| Gateway başına API | 40 |
| Gateway başına endpoint (tüm API'ler toplamı) | 300 |
| Gateway başına iptal edilmemiş anahtar | 1.000 (süresi dolan anahtarlar iptal edilene kadar sayılır) |
| Replika | 1–10; hız limitleri sayaç deposunda sayılıyorsa 20 |
| Zaman aşımı | En fazla 300 saniye (varsayılan 10 saniye) |
| İstek gövdesi | 1–100 MiB (varsayılan 10 MiB) |
| Yanıt önbelleği boyutu | 8–512 MiB (varsayılan 32 MiB), paketin belleğiyle sınırlı |
| Önbelleğe alınacak en büyük yanıt | 64–32768 KiB (varsayılan 512 KiB) |
| Yapılandırma dosyası (kopyalama) | 2 MB'tan küçük |
| Değişiklik geçmişi | 180 gün; sekmede son 50 kayıt |

API, endpoint, anahtar ve politika düzeyindeki sınırlar [API Gateway: API'ler, Anahtarlar ve Politikalar](api-gateway-apis.md#sınırlar) sayfasındadır. Hesap genelindeki sınırlar için [Sınırlar](limits.md) sayfasına bakın.

---

## İlgili Dokümanlar

- [API Gateway: API'ler, Anahtarlar ve Politikalar](api-gateway-apis.md) — API bağlama, anahtarlar, limitler, politikalar ve hata yanıtları.
- [Valkey](managed-valkey.md) — Hız limiti ve kota sayaçları için kullanılan depo.
- [Özel Alan Adları](custom-domains.md) — Gateway'i kendi alan adınızla sunmak.
- [Bant Genişliği Planları](bandwidth-plans.md) — Planlar, fiyatlar ve hız sınırları.
- [Şablonlar, Metrikler ve Loglar](alerts-templates.md) — API Gateway uyarı şablonları.
- [Faturalandırma](billing.md) — Harcama limiti ve ekstreler.
