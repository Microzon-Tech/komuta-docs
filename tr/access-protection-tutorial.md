# Kurulum Rehberi: Hızlı Başlangıçtan İleri Senaryolara

Bu rehber, erişim korumasını tek bir örnek servis üzerinde adım adım kurar. En basit kurulumla başlar ve her seviyede bir yetenek ekler. Rehberin sonunda servis şunları aynı anda yapıyor olur:

- Ekibiniz Komuta hesabıyla, ofisten gelenler ise giriş yapmadan girer.
- Bir müşteri, yalnızca kendisine ayrılan sayfaları belirli bir tarihe kadar görür.
- Yönetim paneli yalnızca iki kişiye açıktır; iç sayfalar herkese kapalıdır.
- GitHub webhook'ları ve CI işleri giriş yapmadan, kendi yöntemleriyle içeri girer.
- Başka bir kümedeki bir arka plan servisi özel ağ üzerinden doğrudan ulaşır.
- Uygulama, içeri gireni imzalı bir kimlik kanıtıyla tanır.
- Koruma belirli bir tarihte biter ve kimin girip kimin reddedildiği kayıt altındadır.

Her adım dört bölümden oluşur:

- **Yapın** — ekranda ne yapacağınız. Kalın yazılar arayüzdeki adlardır.
- **Neden** — bu adımın amacı.
- **Etkisi** — kaydettikten sonra ziyaretçiler için ne değişir.
- **Doğrulayın** — çalıştığını nasıl göreceğiniz.

Bir bölüm ya da seçenek görünmüyorsa ya da "henüz kullanılamıyor" diyorsa, o özellik platformunuzda henüz açık değildir.

Özelliklerin bütün ayrıntıları için [Erişim Koruması](service-access-protection.md) ve [Başvuru](access-protection-reference.md) sayfalarına bakabilirsiniz; bu rehber "nasıl kurulur" sorusuna odaklanır.

---

## Örnek senaryo

Rehber boyunca şu servisi kullanıyoruz:

| | |
|---|---|
| Servis | `panel` — şirket içi bir yönetim uygulaması |
| Adres | `https://panel.example.com` (Komuta'ya eklenmiş özel alan adı) ve servisin `*.komuta.app` adresi |
| Organizasyon | Ekibinizin Komuta organizasyonu |
| Müşteri | `musteri@ornek.com` adresiyle çalışan, organizasyonunuzun dışından biri |
| Yollar | `/` (uygulama), `/raporlar` (müşteriye de açılacak raporlar), `/admin` (yönetim paneli), `/internal` (iç araçlar), `/webhooks/github` (GitHub bildirimleri), `/api/health` (sağlık kontrolü) |

Kendi servisinizde aynı adımları kendi yollarınızla uygulayın.

---

## Başlamadan önce: planlama

Erişim korumasında en çok zaman kazandıran adım, ayarlara girmeden önce şu beş soruyu cevaplamaktır. Her cevap rehberin bir seviyesine karşılık gelir.

| Soru | Örnek cevap | Seviye |
|---|---|---|
| Servisi kimler açabilmeli? | Ekibimiz ve bir müşteri | 1, 2 |
| Nereden açılmalı? | Ofisten girişsiz, dışarıdan girişle | 3 |
| Bazı sayfalar daha sıkı mı korunmalı ya da kapalı mı olmalı? | `/admin` iki kişiye, `/internal` kimseye | 4 |
| Servise insan olmayan istemciler de gelecek mi? | GitHub webhook'u, CI sağlık kontrolü, başka kümedeki bir servis | 5, 6 |
| Uygulama içeri gireni tanımalı mı? Koruma ne zaman bitecek? | Evet; lansmana kadar | 7, 8 |

**Kendinizi dışarıda bırakmama kuralı:** Komuta konsolu korumadan etkilenmez. Bir ayar yanlış giderse konsoldan her zaman düzeltebilir ya da **Ayarlar → Şimdi herkese aç** ile korumayı kaldırabilirsiniz.

---

## Seviye 0: Hazırlık

### Adım 0.1 — Servisin hazır olduğunu kontrol edin

**Yapın**

1. **Servis Detay → Yapılandırma → Erişim ve portlar** sayfasını açın.
2. **Ağ** sekmesinde **Genel URL**'in açık olduğunu ve **Genel adresler** listesinde en az bir adres bulunduğunu görün.
3. Servisin en az bir kez başarıyla dağıtılmış olduğundan emin olun.

**Neden** — Erişim koruması, servisin internetteki adresine gelen trafiği denetler. Genel adresi olmayan ya da hiç dağıtılmamış bir serviste korunacak bir şey yoktur; kart bu durumda açma anahtarını kilitler.

**Etkisi** — Henüz hiçbir şey değişmez.

**Doğrulayın** — **Genel bakış** sekmesindeki **Genel erişim** kartı **Herkese açık** diyor olmalı.

### Adım 0.2 — İzinlerinizi kontrol edin

**Yapın** — Organizasyon yöneticinizden şu iki izni ve servisi düzenleme erişimini aldığınızdan emin olun:

- **Servis erişim korumasını yönet** — korumayı açmak, kuralları ve ayarları değiştirmek için.
- **Korunan servisin paylaşımlarını yönet** — servisi kişilerle paylaşmak ve servis token'ı oluşturmak için.

**Neden** — İki iş bilerek ayrılmıştır: bir ekip lideri kimlerin gireceğini yönetebilir ama kuralları değiştiremez. Bu rehberi baştan sona uygulamak için ikisi de gerekir.

**Etkisi** — İzinler eksikse kartı salt okunur görürsünüz ("Salt okunur — bu ayarları değiştirmek için erişim korumasını yönetme yetkisi gerekir.").

### Adım 0.3 — Özel alan adınızı kontrol edin

**Yapın** — Özel alan adı kullanıyorsanız (örnekte `panel.example.com`), kendi DNS sağlayıcınızdaki kaydın Komuta'yı gösterdiğini ve kendi Cloudflare hesabınızda proxy'li (turuncu bulut) **olmadığını**, **DNS only** (gri bulut) olduğunu kontrol edin.

**Neden** — Koruma, servisin tüm adreslerini birlikte kapsar. Kendi Cloudflare hesabınızda proxy'lenen bir alan adını Komuta koruyamaz ve koruma "Bir özel domain kendi Cloudflare hesabınız üzerinden proxy'leniyor." uyarısıyla tamamlanmaz.

**Etkisi** — Koruma açıldığında hem `panel.example.com` hem `*.komuta.app` adresi aynı kurallarla korunur.

---

## Seviye 1: Hızlı başlangıç — yalnızca ekibim

Hedef: servisi yalnızca organizasyonunuzun üyeleri, Komuta hesaplarıyla giriş yaparak açabilsin.

### Adım 1.1 — Korumayı açın

**Yapın**

1. **Kurallar** sekmesine gidin.
2. **Erişim koruması** kartının başlığındaki anahtarı açın.
3. **Komuta girişi iste** seçili gelir; öyle bırakın.
4. **Korumayı uygula**'ya basın.

**Neden** — Anahtar yalnızca ayarları açar, hiçbir şeyi kaydetmez; **Korumayı uygula** basılana kadar servis eskisi gibi çalışır. Bu, yanlışlıkla korumayı yarım bırakmanızı önler. **Komuta girişi iste**, ziyaretçinin kim olduğunu bilmenin en basit yoludur.

**Etkisi**

- Kart önce **Hazırlanıyor**, sonra **Uygulanıyor**, en sonunda **Korunuyor** gösterir. Bu genellikle bir iki dakika sürer.
- **Uygulanıyor**'dan itibaren servisi açan herkes Komuta giriş sayfasına yönlendirilir.
- Komuta servisin pod'larını da kilitler; ağ geçidini atlayıp doğrudan pod'a gelen istekler reddedilir.
- Paylaşımları yönetme izniniz varsa siz otomatik olarak paylaşım listesine eklenirsiniz. Böylece kendi servisinizin dışında kalmazsınız.

**Doğrulayın**

- Kartın **Korunuyor** dediğini görün.
- Tarayıcınızda gizli bir pencere açıp `https://panel.example.com` adresine gidin: **Devam etmek için giriş yapın** sayfası ve **Gideceğiniz adres** kutusunda servisin adresi görünmeli.
- Komut satırından:

```bash copy
curl -sI https://panel.example.com/ | grep -i -E '^(HTTP|location)'
```

`HTTP/2 302` ve `location: https://console.komuta.io/access/…` görmelisiniz.

### Adım 1.2 — Servisi ekibinizle paylaşın

**Yapın**

1. **Kişiler** sekmesinde **Paylaşım ekle**'ye basın.
2. **Organizasyonunuz** kartını seçin.
3. **Neleri açabilir** için **Tüm site** seçili kalsın; **Erişim bitişi**'ni boş bırakın.
4. **Paylaşım ekle** ile kaydedin.

**Neden** — Giriş, kimin geldiğini öğrenir; paylaşım, kimin girebileceğine karar verir. Paylaşım olmadan giriş yapan herkes **Erişiminiz yok** sayfasında kalır.

**Etkisi** — Organizasyonunuzun bütün aktif üyeleri, bugün ve sonradan katılanlar dahil, giriş yaptıktan sonra servisi açar. Organizasyondan çıkarılan biri erişimini kaybeder.

**Doğrulayın** — Bir ekip arkadaşınız servisi açtığında giriş yapıp doğrudan uygulamaya dönmeli. **Etkinlik** sekmesinde **Giriş yaptı** ve **Sayfayı açtı** satırları görünmeli.

> Bu noktada hızlı başlangıç tamamdır. Servisiniz artık yalnızca ekibinize açıktır. Sonraki seviyeler isteğe bağlıdır ve sırayla birbirinin üzerine kurulur.

---

## Seviye 2: Bir müşteriye süreli ve sınırlı erişim

Hedef: organizasyon dışındaki bir müşteri, yalnızca `/raporlar` sayfalarını ve yalnızca belirli bir tarihe kadar görsün.

### Adım 2.1 — Dış paylaşıma izin verin

**Yapın**

1. **Hesap → Organizasyonlar** sayfasını açın.
2. **Dış paylaşıma izin ver** ayarını açın. (Bu ayarı değiştirmek için organizasyonu düzenleme yetkisi gerekir; yetkiniz yoksa bir organizasyon yöneticisinden isteyin.)

**Neden** — Organizasyon dışına erişim vermek bilinçli bir karardır; bu yüzden ayar **varsayılan olarak kapalıdır** ve organizasyon düzeyindedir.

**Etkisi** — Organizasyonun tüm korunan servislerinde **Bağlı bir organizasyon** ve **Bir e-posta adresi** paylaşım türleri kullanılabilir hale gelir. İleride bu ayarı kapatırsanız bu tür paylaşımlar **Askıda** olur ve bunlarla girenlerin erişimi biter.

### Adım 2.2 — Müşteriyi e-posta adresiyle ekleyin

**Yapın**

1. **Kişiler → Paylaşım ekle → Bir e-posta adresi** seçin.
2. **E-posta adresi** alanına `musteri@ornek.com` yazın.
3. **Neleri açabilir** için **Yalnızca şu sayfalar**'ı seçin ve alana `/raporlar` yazın.
4. **Erişim bitişi**'ne projenin bitiş tarihini girin.
5. **Paylaşım ekle** ile kaydedin.

**Neden**

- **E-posta paylaşımı**, müşterinin sizin organizasyonunuza katılmasını gerektirmez. Müşteri herhangi bir Komuta hesabıyla giriş yapar ve posta kutusunu okuyabildiğini her girişte tek kullanımlık bir kodla kanıtlar.
- **Yalnızca şu sayfalar**, en az yetki ilkesidir: müşteri `/raporlar` ve altındaki sayfaları açar, uygulamanın geri kalanını açamaz.
- **Erişim bitişi**, erişimi kaldırmayı unutma riskini ortadan kaldırır.

**Etkisi**

- Müşteri servisi açar, Komuta'ya giriş yapar (hesabı yoksa Google ya da GitHub ile saniyeler içinde açar), **Erişiminiz yok** sayfasındaki **E-posta adresinizle mi paylaşıldı?** bölümünden **Bana kod gönder** der, adresine gelen 8 haneli kodu girer ve **Doğrula ve devam et** ile içeri girer.
- `/raporlar` dışındaki bir sayfayı açarsa **Bu sayfaya erişiminiz yok** sayfasını ve açabileceği sayfaların listesini görür.
- Bitiş tarihinde erişim kendiliğinden biter; paylaşım listede **Süresi doldu** etiketiyle kalır.

**Dikkat** — **Organizasyonunuz** paylaşımı tüm siteyi açtığı için ekibiniz etkilenmez. Ancak bir ekip arkadaşını sayfa sınırlı bir üye paylaşımıyla kısıtlamaya çalışırsanız işe yaramaz: **en geniş paylaşım kazanır**.

**Doğrulayın** — **Kurallar** sekmesinin altındaki **Erişim önizlemesi**'nde **Bir kişi neyi açar** görünümünü seçin, müşterinin paylaşımını seçin: `/` için **Kendisiyle paylaşılan sayfaların dışında**, `/raporlar` için **Giriş yapınca girer** görmelisiniz.

---

## Seviye 3: Ofisten girişsiz, dışarıdan girişle

Hedef: ofis ağından gelenler giriş yapmadan girsin; evden ya da yoldan bağlananlar Komuta ile giriş yapsın.

### Adım 3.1 — Ofis adreslerini ekleyin

**Yapın**

1. **Kurallar** sekmesinde **IP izin listesi** alanına ofisinizin genel IP adresini ya da aralığını yazın (her satıra bir adres veya CIDR aralığı). Ofisteyseniz **Kendi IP'mi ekle** düğmesini kullanabilirsiniz.
2. Henüz **Korumayı uygula**'ya basmayın.

**Neden** — Ofis ağı genellikle sabit bir genel IP'den çıkar. Bu adres, "bu kişi ofisten bağlanıyor" bilgisinin güvenilir kaynağıdır: Komuta adresi yalnızca Cloudflare'in bildirdiği değerden alır ve ziyaretçinin kendi gönderdiği başlıkları dikkate almaz.

**Etkisi** — Kaydedilene kadar yok. Kart hatalı bir satırı "Satır {n}" diye işaretler; özel ağ aralıkları (`10.x`, `192.168.x` gibi) kabul edilmez, çünkü internetten gelen bir istek bu adreslerden gelemez.

**Dikkat** — **Kendi IP'mi ekle**, konsolun gördüğü adresi ekler. Tarayıcınız servise farklı bir bağlantıyla (örneğin konsola IPv6, servise IPv4) ulaşıyorsa servis başka bir adres görür. Hem IPv4 hem IPv6 adresinizi eklemek güvenlidir.

### Adım 3.2 — "Biri yeterli"yi seçin

**Yapın**

1. Hem **Komuta girişi iste** açıkken hem listede adres varken kart **Giriş ve IP izin listesi nasıl birleşsin?** diye sorar.
2. **Biri yeterli**'yi seçin.
3. **Korumayı uygula**'ya basın.

**Neden** — İki seçenek tamamen farklı sonuç verir:

| Seçenek | Ofisteki çalışan | Evdeki çalışan | Müşteri |
|---|---|---|---|
| **İkisi birden gereksin** | Giriş yapar | Giremez (403) | Giremez (403) |
| **Biri yeterli** | Girişsiz girer | Giriş yapar | Giriş yapar |

**İkisi birden gereksin**, servisi ofis dışına tamamen kapatır; müşteriniz de giremez. Bu senaryoda istediğimiz **Biri yeterli**'dir.

**Etkisi**

- Ofis adresinden gelenler giriş sayfasını görmeden uygulamayı açar.
- Diğer herkes Seviye 1 ve 2'deki gibi giriş yapar; paylaşımlar aynen geçerlidir.
- Ofis adresinden gelenler, daha önce giriş yapmış olsalar bile, bu sayfalarda erişim kaydına yazılmaz ve uygulamaya kimlikleri bildirilmez: istek IP adresiyle geçtiği için oturumlarına bakılmaz. Bu, Seviye 7'de önemlidir.

**Doğrulayın** — **Erişim önizlemesi**'nde **Bir sayfayı kim açar** görünümünü seçin, **Sayfa** `/` iken **Nereden geliyor**'u **Belirli bir adresten** yapıp ofis adresinizi yazın: giriş yapmamış biri için **İzinli bir ağdan girer** görmelisiniz. **İzinli ağların dışından** seçince aynı satır **Giriş yapması gerekiyor (site geneli)** demeli.

---

## Seviye 4: Sayfa sayfa koruma

Hedef: `/admin` yalnızca iki kişiye açık olsun, `/internal` kimseye açık olmasın.

### Adım 4.1 — İç araçları tamamen kapatın

**Yapın**

1. **Kurallar** sekmesindeki **Yol kuralları** bölümünde **Yol kuralı ekle**'ye basın.
2. **Yol**: `/internal`, **Koruma**: **Tamamen engelle**.
3. Henüz **Korumayı uygula**'ya basmayın; Adım 4.2 ile birlikte uygulayacağız.

**Neden** — Bazı sayfalar internetten hiç açılmamalıdır; uygulamanın içinde bir hata olsa bile. Engelleme kuralı en güçlü kuraldır: giriş, izinli IP, servis token'ı ya da webhook yolu onu aşamaz.

**Etkisi** — `/internal` ve altındaki tüm yollar herkese `403` döner. Kural büyük/küçük harf, `%2f`, `..` gibi yazım hileleriyle atlatılamaz.

### Adım 4.2 — Yönetim panelini iki kişiye açın

**Yapın**

1. Önce **Kişiler** sekmesinde paneli kullanacak iki kişiyi **Bir üye** olarak ekleyin. (Organizasyon paylaşımı zaten var; üye paylaşımları, kişileri kuralda tek tek seçebilmeniz içindir.)
2. **Kurallar** sekmesinde **Yol kuralı ekle**: **Yol** `/admin`, **Koruma** **Yalnızca seçilen kişiler**.
3. **Bu yolu kimler açabilir** listesinde iki kişinin paylaşımını işaretleyin. Paneli siz de kullanacaksanız Adım 1.1'de otomatik eklenen kendi paylaşımınızı da işaretleyin.
4. İsterseniz bir kişiye **Başlangıç (isteğe bağlı)** ve **Bitiş (isteğe bağlı)** saatleri verin (örneğin bir danışmana yalnızca bir hafta).
5. **Korumayı uygula**'ya basın.

**Neden** — Site bütün ekibe açık; ama yönetim paneli herkesin işi değildir. **Yalnızca seçilen kişiler**, paylaşımları silmeden yalnızca bu yolu daraltır.

**Etkisi**

- `/admin`'e yalnızca seçtiğiniz iki kişi girer; diğer ekip üyeleri sitenin geri kalanını kullanmaya devam eder ve `/admin`'de **Bu sayfaya erişiminiz yok** görür.
- **Kurallar yalnızca sıkılaştırır:** ofisten gelen biri bile `/admin` için giriş yapmak zorundadır, çünkü bu kural ofis adresini değil kişiyi sorar.
- Saat aralığı her istekte kontrol edilir; aralık bitince açık oturumlar dahil yol birkaç saniye içinde kapanır.
- Servis token'ları bu yolu hiçbir zaman açamaz.

**Doğrulayın** — **Erişim önizlemesi → Bir sayfayı kim açar** görünümünde **Sayfa** alanına `/admin` yazın: iki kişi için yeşil onay, organizasyon paylaşımı ve diğer üye paylaşımları için **/admin için seçilen kişiler arasında değil**, müşterinin paylaşımı için **Kendisiyle paylaşılan sayfaların dışında**, giriş yapmamış biri için **Giriş yapması gerekiyor (site geneli)** görmelisiniz. **Nereden geliyor**'u **Belirli bir adresten** yapıp ofis adresinizi yazarsanız giriş yapmamış biri için **Giriş yapması gerekiyor (/admin)** görünür: ofisten gelen biri de bu yolda giriş yapmalıdır. `/internal` yazınca herkes için **/internal kuralı engelliyor** görünmeli.

---

## Seviye 5: Makinelere izin verin

Hedef: GitHub webhook'ları ve CI işinin sağlık kontrolü, giriş yapamadıkları halde içeri girebilsin.

### Adım 5.1 — GitHub webhook'u için açık yol

**Yapın**

1. **Makineler** sekmesinde **Webhook'lar → Yol aç**'a basın.
2. **Yol**: `/webhooks/github`.
3. **Yöntemler**: yalnızca `POST` (varsayılan).
4. **Gönderici adresleri (isteğe bağlı)**: GitHub'ın webhook adres aralıklarını yazın. GitHub bu listeyi `https://api.github.com/meta` adresindeki `hooks` alanında yayımlar (örneğin `140.82.112.0/20`).
5. **Yolu aç** ile kaydedin.

**Neden** — GitHub bir tarayıcı değildir; Komuta giriş sayfasını açıp giriş yapamaz. Açık yol, bu tek yolu yalnızca seçtiğiniz yöntemle giriş istemeden açar. Gönderici listesi ek bir süzgeçtir; asıl güvence GitHub'ın imzasıdır.

**Etkisi**

- `POST /webhooks/github` giriş istemeden uygulamanıza ulaşır. Sitenin IP listesi bu yola uygulanmaz; tek adres kontrolü yolun kendi gönderici listesidir.
- `GET /webhooks/github` gibi başka yöntemler açık yol yokmuş gibi değerlendirilir ve sitenin normal korumasından geçmek zorundadır.
- Erişim kaydında teslimatlar **Açık yol isteği** / **Açık yola teslim edildi** olarak görünür.

**Uygulamanızda yapmanız gereken** — Kenarda imza kontrolü yoksa (aşağıdaki tarife bakın) Komuta bu yolda kimlik sormaz; isteğin gerçekten GitHub'dan geldiğini uygulamanız doğrulamalıdır. GitHub her isteğe `X-Hub-Signature-256` başlığını ekler: webhook imza sırrınızla isteğin ham gövdesinin HMAC-SHA256 özetidir. Node.js örneği:

```javascript copy
import { createHmac, timingSafeEqual } from "node:crypto";

export function isFromGitHub(rawBody, signatureHeader, secret) {
  if (!signatureHeader) return false;
  const expected = "sha256=" + createHmac("sha256", secret).update(rawBody).digest("hex");
  const a = Buffer.from(expected);
  const b = Buffer.from(signatureHeader);
  return a.length === b.length && timingSafeEqual(a, b);
}
```

İmzası uymayan isteği reddedin. Ayrıca bu yolda `X-HTTP-Method-Override` gibi yöntem değiştirme başlıklarını kabul etmeyin.

**Doğrulayın** — GitHub depo ayarlarındaki webhook sayfasında **Recent Deliveries** başarılı görünmeli; **Etkinlik** sekmesinde **Açık yol isteği** / **Açık yola teslim edildi** satırı çıkmalı. Kendi bilgisayarınızdan `curl -s -o /dev/null -w "%{http_code}\n" -X POST https://panel.example.com/webhooks/github` çalıştırırsanız `403` görürsünüz: adresiniz gönderici listesinde değildir. (**Erişim önizlemesi** yalnızca `GET` isteklerini hesaplar; yalnızca `POST` açtığınız bu yol için sitenin normal kuralını gösterir.)

### Tarif: GitHub webhook'larını imza kontrolüyle alın

GitHub'ın imzasını uygulamanızda doğrulamak yerine bunu Komuta'nın kenarda yapmasını sağlayabilirsiniz. **Kenarda imza kontrolü** seçimi bunun kullanılamadığını söylüyorsa özellik platformunuzda henüz açık değildir; kontrolü uygulamanızda tutun. Kontrol yol açılırken seçilir; `/webhooks/github` yolunu Adım 5.1'de zaten açtıysanız önce kapatın.

1. **Makineler** sekmesinde **Webhook'lar → Yol aç**'a basın: **Yol** `/webhooks/github`, **Yöntemler** `POST`, **Kenarda imza kontrolü** **GitHub (X-Hub-Signature-256)**. İsterseniz **Gönderici adresleri**'ni Adım 5.1'deki gibi doldurun. **Yolu aç**'a basın.
2. Yeni yolun altında, **İmza sırları** bölümünde **Sır ekle**'ye basın, **Güçlü bir sır üret**'i seçili bırakın, **Sır ekle**'ye basın ve gösterilen sırrı kopyalayın. Sır yalnızca bir kez gösterilir.
3. GitHub deponuzda **Settings → Webhooks**'u açın; **Payload URL**'i `https://panel.example.com/webhooks/github`, **Content type**'ı `application/json` yapın ve sırrı **Secret** alanına yapıştırın.
4. Sırrın ağ geçidine ulaşması için yaklaşık bir dakika bekleyin.

**Etkisi** — Uygulamanıza yalnızca sırrınızla imzalanmış `POST` istekleri ulaşır; `/webhooks/github` yoluna gelen diğer her şey (`GET` dahil) `401`, 65.535 bayttan büyük bir gövde `413` alır. Büyük bir `push` olayı bu sınırı aşabilir; deponuz böyle olaylar gönderiyorsa kontrolü uygulamanızda tutun.

**Doğrulayın** — GitHub'ın **Recent Deliveries** listesi başarılı görünmeli (gerekirse `ping` olayını yeniden gönderin). Listedeki bir adresten (ya da gönderici listesi boşsa herhangi bir yerden) `curl -s -o /dev/null -w "%{http_code}\n" -X POST -d '{}' https://panel.example.com/webhooks/github` çalıştırırsanız `401` görürsünüz: istek imzalı değildir. **Etkinlik** sekmesinde retler "Geçerli imzası olmayan bir webhook gönderdi" olarak görünür.

**Sırrı değiştirmek** — ikinci bir sır ekleyin, GitHub'ı ona geçirin, sonra eskisini kaldırın; arada ikisi de çalışır. Bir yol en fazla 2 sır tutar.

### Adım 5.2 — CI işi için servis token'ı

**Yapın**

1. **Makineler** sekmesinde **Servis token'ları → Token oluştur**'a basın.
2. **Ad**: `github-actions-health`.
3. **Geçerlilik sonu**: örneğin üç ay sonrası.
4. **Neleri açabilir**: **Yalnızca şu sayfalar** ve `/api/health`.
5. **Token oluştur**'a basın. **Token'ı şimdi kopyalayın** ekranındaki değeri kopyalayın ve **Kaydettim** ile kapatın.
6. Değeri GitHub deponuzda **Settings → Secrets and variables → Actions** altında `KOMUTA_SERVICE_TOKEN` adıyla saklayın.
7. CI adımında token'ı başlık olarak gönderin:

```yaml copy
- name: Sağlık kontrolü
  env:
    KOMUTA_SERVICE_TOKEN: ${{ secrets.KOMUTA_SERVICE_TOKEN }}
  run: |
    status=$(curl -sS -o /dev/null -w "%{http_code}" \
      -H "x-komuta-service-token: $KOMUTA_SERVICE_TOKEN" \
      https://panel.example.com/api/health)
    test "$status" = "200" || { echo "health check returned $status"; exit 1; }
```

`curl --fail` yerine durum kodunu kontrol edin: token okunmazsa Komuta `302` ile giriş sayfasına yönlendirir ve `curl --fail` bunu başarı sayar.

**Neden**

- Token, giriş yerine geçen bir anahtardır; tarayıcısı olmayan bir program için tasarlanmıştır.
- **Yalnızca şu sayfalar** ve **Geçerlilik sonu**, token sızarsa verebileceği zararı sınırlar: sızan bir token yalnızca `/api/health`'i ve yalnızca bitiş tarihine kadar açar.
- Değer yalnızca bir kez gösterilir; Komuta yalnızca özetini saklar. Bu yüzden hemen gizli değişken deposuna koyulur.

**Etkisi**

- Serviste ilk token'ı oluşturduğunuzda Komuta servisin yönlendirme ayarlarını yeniler; bu birkaç dakika sürer ve bu sırada token çalışmıyor gibi görünür (`302`).
- CI isteği giriş sayfasına yönlendirilmeden `/api/health`'e ulaşır. Token başlığı uygulamanıza ulaşmadan silinir.
- **Seviye 3'teki "Biri yeterli" seçiminin burada bir faydası var:** token giriş şartını karşılar ve site kuralı "biri yeterli" olduğu için CI makinesinin IP adresi ofis listesinde olmak zorunda değildir. **İkisi birden gereksin** seçilseydi, CI makinesinin de listedeki bir adresten gelmesi gerekirdi; GitHub Actions makinelerinin adresleri değiştiği için bu pratik değildir.
- Token `/admin`'i (kişi kuralı) ve `/internal`'ı (engelleme, `403 access denied`) açamaz; kapsamı dışındaki diğer sayfalarda `403 this service token cannot open this path` alır.
- Geçersiz ya da silinmiş bir token `401 invalid service token` alır (ofis listesindeki bir adresten gelmiyorsa); giriş sayfasına yönlendirilmez.

**Doğrulayın**

Bu komutu kendi bilgisayarınızda çalıştırmak için token değerine ihtiyacınız var; GitHub'daki gizli değişken sonradan okunamaz. Değeri **Kaydettim**'e basmadan önce terminalde `export KOMUTA_SERVICE_TOKEN='kst_…'` ile tanımlayın. Değeriniz yoksa en kolay doğrulama, GitHub'da işi çalıştırıp **Sağlık kontrolü** adımının yeşil geçtiğini görmektir. Değişken tanımlı değilse `curl` başlığı hiç göndermez ve `302` görürsünüz.

```bash copy
curl -s -o /dev/null -w "%{http_code}\n" -H "x-komuta-service-token: $KOMUTA_SERVICE_TOKEN" https://panel.example.com/api/health
```

`200` görmelisiniz. Aynı komutu `https://panel.example.com/` için çalıştırırsanız `403` görürsünüz (token'ın kapsamı dışında). **Bu iki denemeyi ofis ağının dışından yapın** (ör. telefonunuzun bağlantısı ya da CI): ofis adresinden gelen istek Seviye 3'teki "biri yeterli" kuralıyla girişsiz geçer, token'a hiç bakılmaz ve her iki komut da `200` döner.

---

## Seviye 6: Başka bir kümedeki servisinize izin verin

Hedef: başka bir kümede çalışan `rapor-isleyici` servisi, `panel`'e özel ağ üzerinden, Komuta girişi olmadan doğrudan ulaşsın.

Bu seviye yalnızca servislerinizi farklı kümelerde çalıştırıyor ve özel ağı (mesh) kullanıyorsanız gereklidir. **Ağ** sekmesindeki **Özel ağ (mesh)** kartında anahtar kilitliyse ve kart "Bu serviste erişim koruması açık. … Özel ağı açmak için önce erişim korumasını kapatın" diyorsa, bu özellik platformunuzda henüz açık değildir ve korunan bir servis özel ağa açılamaz; bu seviyeyi atlayın. Kart "Erişim korumasıyla birlikte çalışır" diyorsa devam edin.

### Adım 6.1 — Özel ağı açın ve izin verilecek servisi seçin

**Yapın**

1. `panel` servisinde **Ağ** sekmesindeki **Özel ağ (mesh)** kartında özel ağı açın.
2. **Makineler** sekmesindeki **Özel ağdan doğrudan gelebilecek servisler** bölümünde **Bir servis seçin** listesinden `rapor-isleyici`'yi seçip **İzin ver**'e basın.

**Neden** — Özel ağ trafiği Komuta'nın ağ geçidinden geçmez, doğrudan pod'lara gider; bu yüzden giriş ya da IP kurallarıyla denetlenemez. Korunan bir serviste özel ağdan kimin gelebileceğini bu liste belirler.

**Etkisi**

- Yalnızca listedeki servisler, diğer kümelerinizden `panel`'e doğrudan ulaşır. Liste boşsa özel ağdan hiçbir servis gelemez.
- Aynı kümedeki servisleriniz bu listeden etkilenmez.
- Listedeki bir servis internete açık ve korumasızsa **Herkese açık, korumasız** uyarısı çıkar: o servis buraya ulaşmak için bir aracı olarak kullanılabilir. O servisi de koruyun ya da aldığı istekleri olduğu gibi iletmediğinden emin olun.

**Doğrulayın** — `rapor-isleyici` içinden `panel`'in özel ağ adresine yapılan istek yanıt almalı; listede olmayan ve **başka bir kümede** çalışan bir servisten aynı istek zaman aşımına uğramalı (aynı kümedeki servisler bu listeden etkilenmez).

---

## Seviye 7: Uygulamanız içeri gireni tanısın

Hedef: `panel` uygulaması, sayfayı açan kişinin kim olduğunu ayrı bir giriş ekranı olmadan bilsin; örneğin denetim kaydına yazmak için.

### Adım 7.1 — Kimlik bildirmeyi açın

**Yapın**

1. **Ayarlar** sekmesinde **Giriş yapanı uygulamama bildir** anahtarını açın (ayrıntılar [Bitiş ve Kimlik Bildirme](access-protection-settings.md#giriş-yapanı-uygulamama-bildir) sayfasında).
2. Bölüm "Hazırlanıyor" notunu gösterirken birkaç dakika bekleyin.

**Neden** — Komuta ziyaretçinin kim olduğunu zaten biliyor. Bu ayar, bu bilgiyi uygulamanıza her istekte başlık olarak iletir; aynı kişiyi ikinci kez doğrulamak için kod yazmanız gerekmez.

**Etkisi**

- Giriş gerektiren her istekte uygulamanız `x-komuta-user-email`, `x-komuta-user-id` ve imzalı `x-komuta-identity` başlıklarını alır.
- **Seviye 3'ün sonucu:** ofis adresinden gelen isteklerde, ziyaretçi giriş yapmış olsa bile başlıklar boştur; çünkü istek IP adresiyle geçer ve oturuma bakılmaz. Uygulamanız kimliği her sayfada istiyorsa ya **İkisi birden gereksin** seçin (bu, Seviye 2'deki müşteriyi ve Seviye 5'teki CI işini de dışarıda bırakır) ya da kimlik gereken yolları (ör. `/admin`) bir giriş kuralıyla koruyun; bu yollarda ofisten gelenler de giriş yapar.
- Servis token'ıyla gelen isteklerde yalnızca `x-komuta-identity` dolu gelir (`kind: service_token`).

### Adım 7.2 — Kimliği uygulamanızda doğrulayın

**Yapın** — Uygulamanızda düz başlıklara değil, imzalı `x-komuta-identity` jetonuna güvenin ve her istekte doğrulayın: imza, `iss` (`komuta-access`), `aud` (kendi adresleriniz), `sid` (kendi servis kimliğiniz), `exp`/`nbf`. Hazır Node.js ve Python örnekleri [Bitiş ve Kimlik Bildirme](access-protection-settings.md#kimlik-kanıtı-jwt) sayfasındadır.

**Neden** — Aynı kümedeki diğer servisleriniz ve özel ağdan izin verdiğiniz servisler pod'larınıza Komuta'dan geçmeden ulaşabildiği için düz başlıkları kendileri de gönderebilir. İmzalı jeton taklit edilemez. `aud` ve `sid` kontrolü, başka bir uygulamaya verilmiş geçerli bir jetonun sizin uygulamanıza tekrar gönderilmesini engeller.

**Doğrulayın** — `/admin` kuralında kendinizi seçtiyseniz `/admin`'i açtığınızda uygulamanızın kaydında kendi e-postanız görünmeli. (Seçmediyseniz ofis dışından `/` sayfasını açarak deneyin.) Ayarı açmadan önce giriş yapmış olanlar, yeniden giriş yapana kadar (en fazla oturum süresi kadar, varsayılan 12 saat) e-postasız görünür.

---

## Seviye 8: Süre, izleme ve bakım

### Adım 8.1 — Korumaya bir bitiş tarihi verin

**Yapın**

1. **Ayarlar** sekmesinde **Bitiş tarihi ve saati** alanına lansman tarihini girin.
2. **Bittiğinde** için **Sonra kilitli kalsın**'ı seçin.
3. **Bitişi kaydet**'e basın.

**Neden** — **Sonra kilitli kalsın**, "unutursam ne olur?" sorusunun güvenli cevabıdır: tarih geldiğinde koruma sürer ve size e-posta gelir; açıp açmamaya siz karar verirsiniz. Servisin lansmanda kendiliğinden açılmasını istiyorsanız **Sonra herkese açılsın**'ı seçin.

**Etkisi** — Bitişten 24 saat ve 1 saat önce, bir de bitişte, servise düzenleme yetkisi verilmiş ve **Servis erişim korumasını yönet** iznine sahip kişilere (böyle biri yoksa organizasyon yöneticilerine) e-posta gider. E-postalardaki **Şimdi herkese aç**, **Süreyi uzat** ve **Kalıcı yap** bağlantıları konsolda ilgili ekranı açar; işlem orada onayınızla yapılır.

### Adım 8.2 — Erişim kaydını izleyin

**Yapın** — **Etkinlik** sekmesinde **Göster: Retler** filtresini seçin.

**Neden** — Retler, kuralların beklediğiniz gibi çalışıp çalışmadığını gösterir: listede olmayan bir gönderici adresinden gelen webhook (**İzinli olmayan bir adresten geldi**), yanlış kişiye kapanmış bir yol (**Bu sayfa kendisiyle paylaşılmamış**), engelleme kuralına takılan bir istek (**Sayfa engelli**) ya da süresi dolmuş bir token burada görünür.

**Etkisi** — Kayıt varsayılan olarak 30 gün saklanır (organizasyon 90 ya da 365 gün seçebilir); **Dışa aktar** kaydı CSV ya da JSON olarak indirir. Giriş yapmamış ziyaretçilerin retleri kötüye kullanımı önlemek için **Herhangi bir sayfa** ve adresin ilk kısmıyla (IPv4'te `/24`, IPv6'da `/48`) görünür; engelleme kuralı, açık yol ve yöntem kuralı retlerinde kuralın yolu yazar.

### Adım 8.3 — Hangi değişikliklerin oturum kapattığını bilin

Şu değişiklikler oturumları yaklaşık 30 saniye içinde bitirir; erişimi devam edenler bir sonraki sayfada yeniden giriş yapar (Komuta'ya giriş yapmış olanlar için bu otomatiktir, e-posta paylaşımıyla girenler yeni kod ister):

| Değişiklik | Kimin oturumu biter |
|---|---|
| **Kişiler → Kimler içeride** altında bir kişinin satırında **Çıkar** | O kişinin |
| Bir paylaşımı kaldırmak, ona bitiş eklemek ya da bitişini öne çekmek, sayfa sınırlı bir paylaşımın sayfa listesini değiştirmek | O paylaşımla açılmış oturumlar |
| **Herkesi çıkar** | Paylaşım bağlantısıyla girenler dahil herkesin |
| Dış paylaşımı kapatmak | Dış paylaşımı olan servislerde herkesin |
| **Oturum süresi**'ni kısaltmak | Yeni süreden eski oturumlar |

Serviste oturum takibi başladıktan sonraki ilk 12 saat 10 dakika boyunca Komuta oturumları henüz birbirinden ayırt edemez: bu sürede bir kişiyi çıkarmak ya da bir paylaşımı değiştirmek **herkesi** çıkarır ve **Kimler içeride** bölümü bunun ne zamana kadar süreceğini söyler. Yeni paylaşım eklemek, bitişi uzatmak ya da kural eklemek oturumları etkilemez.

### Adım 8.4 — Gerektiğinde korumayı kaldırın

**Yapın** — **Ayarlar → Korumayı hemen kaldır → Şimdi herkese aç**.

**Etkisi** — Giriş, IP listesi ve yol kuralları uygulanmaz; servis herkese açılır. Paylaşımlar, paylaşım bağlantıları, servis token'ları ve özel ağ listesi saklanır; kimlik bildirme ayarı da saklanır ve korumayı Komuta girişiyle yeniden açtığınızda kendiliğinden geri gelir. Korumayı yeniden açtığınızda Komuta girişi, IP listesi, yol kuralları, webhook yolları ve bitiş tarihini yeniden girmeniz gerekir; bu yüzden ayarlarınızı bir yere not etmeniz işinizi kolaylaştırır.

---

## Son durum: hepsi bir arada

Rehberin sonunda `panel` servisinin ayarları şöyledir:

| Sekme | Ayar | Değer |
|---|---|---|
| Kurallar | **Komuta girişi iste** | Açık |
| Kurallar | **IP izin listesi** | Ofis aralığınız |
| Kurallar | **Giriş ve IP izin listesi nasıl birleşsin?** | **Biri yeterli** |
| Kurallar | Yol kuralı `/internal` | **Tamamen engelle** |
| Kurallar | Yol kuralı `/admin` | **Yalnızca seçilen kişiler** (iki üye) |
| Kişiler | **Organizasyonunuz** | Tüm site, bitişsiz |
| Kişiler | `musteri@ornek.com` | Yalnızca `/raporlar`, proje bitişine kadar |
| Kişiler | İki **Bir üye** paylaşımı (ve Adım 1.1'de eklenen kendi paylaşımınız) | Tüm site |
| Makineler | Webhook yolu `/webhooks/github` | `POST`, GitHub adresleri |
| Makineler | Servis token'ı `github-actions-health` | Yalnızca `/api/health`, üç ay |
| Makineler | Özel ağdan doğrudan gelebilecek servisler | `rapor-isleyici` |
| Ayarlar | **Giriş yapanı uygulamama bildir** | Açık |
| Ayarlar | Koruma bitişi | Lansman tarihi, **Sonra kilitli kalsın** |

### Bir isteğe nasıl karar verilir

En ileri senaryoyu kendi başınıza kurabilmek için Komuta'nın her isteği hangi sırayla değerlendirdiğini bilmek yeterlidir:

1. **Engelleme kuralı** — Yol bir **Tamamen engelle** kuralına giriyorsa istek `403` ile reddedilir. Başka hiçbir şeye bakılmaz.
2. **Yöntem kuralları** — Yolu kapsayan bir [yöntem kuralı](access-protection-machines.md#yöntem-kuralları) yönteme izin vermiyorsa yanıt `405`'tir. Tarayıcının CORS kontrolü, sorduğu yönteme göre değerlendirilir.
3. **Ülkeler** — Serviste [ülke listesi](access-protection-rules.md#ülkeler) varsa, başka bir ülkeden gelen ya da ülkesi bilinmeyen ziyaretçi `403` ile reddedilir. Webhook yolları, üzerlerinde bir paylaşım bağlantısı açılmıyorsa bu adımı atlar.
4. **Hız sınırı** — Adres [hız sınırını](access-protection-rules.md#hız-sınırı) doldurduysa yanıt `429`'dur.
5. **Paylaşım bağlantısı** — Adreste `?komuta_link=` varsa bağlantı kontrol edilir (IP listeleri yine uygulanır) ve ziyaretçi bir oturumla aynı adrese gönderilir ya da `403` ile reddedilir.
6. **Webhook (açık) yolu** — Yol bir açık yolun altındaysa ve yöntem seçilmişse, yolun kendi gönderici listesi kontrol edilir ve istek giriş istemeden geçer. Sitenin ve diğer yolların kuralları uygulanmaz. Adres listede değilse istek `403` ile reddedilir; sitenin kurallarına geçilmez. Yolda imza kontrolü varsa ardından gövdenin imzası kontrol edilir (geçerli imza yoksa `401`, 65.535 bayttan büyük gövdede `413`). İmzalı bir yolun altına gelen ama yolun kabul etmediği bir istek de burada, sitenin kurallarına geçmeden `401` alır.
7. **Site kuralı ve eşleşen her yol kuralı** — İstek hepsini birden sağlamalıdır. IP adresi, bir kuralın IP şartını karşılayabilir; "biri yeterli" olan kurallarda listedeki adres girişin yerine geçer. Giriş yapılsa bile sağlanamayacak bir kural varsa (yalnızca IP isteyen bir kural ya da listede olmayan bir adresten **İkisi birden gereksin**), istek burada `403` ve **Bu servise erişim kısıtlı** sayfasıyla reddedilir; giriş sayfası gösterilmez. **CORS kontrollerine girişsiz izin ver** açıksa tarayıcının CORS kontrolü bu noktada geçer.
8. **Kimlik** — Hâlâ bir kimlik gerekiyorsa: istekte servis token'ı varsa yalnızca token'a bakılır; yoksa ziyaretçinin oturumuna (Komuta girişi ya da paylaşım bağlantısı) ve paylaşımlarına bakılır. Paylaşımın ya da bağlantının sayfa sınırı ve kişi kuralının saat aralığı burada uygulanır. Oturum yoksa `GET` ve `HEAD` istekleri (tarayıcı ya da `curl` fark etmez) giriş sayfasına yönlendirilir (`302`); `POST` gibi diğer yöntemler `401` alır.

Bu sıraya göre örnek istekler:

| İstek | Sonuç | Neden |
|---|---|---|
| Ofisten, giriş yapmadan `GET /` | Açılır | 7. adım: site kuralı "biri yeterli", adres listede |
| Evden, ekip üyesi `GET /` | Giriş yapınca açılır | 7–8: adres listede değil, organizasyon paylaşımı var |
| Ofisten `GET /admin`, seçilmemiş ekip üyesi | Giriş yaptıktan sonra **Bu sayfaya erişiminiz yok** | 8: `/admin` kişi kuralı, kişi seçilmemiş |
| Herhangi biri `GET /internal/araclar` | `403` | 1: engelleme kuralı |
| Müşteri `GET /raporlar/2026` | E-posta koduyla açılır | 8: e-posta paylaşımı, sayfa kapsamında |
| Müşteri `GET /` | Giriş ve e-posta kodundan sonra **Bu sayfaya erişiminiz yok** ve açabileceği sayfalar | 8: sayfa sınırı dışında |
| GitHub `POST /webhooks/github` | Açılır (imzayı uygulama, imza kontrolü seçtiyseniz Komuta doğrular) | 6: açık yol, gönderici listede |
| Herhangi biri `GET /webhooks/github` | Site kuralına göre | 6 atlanır (yöntem seçilmemiş), 7–8 uygulanır |
| CI, token ile `GET /api/health` | Açılır | 7–8: token giriş yerine geçer, kapsamda |
| CI, token ile `GET /admin` | `403` | 8: kişi kuralı token'ı kabul etmez |

Yeni bir kural eklemeden önce kendinize şunu sorun: "Bu istek hangi adımda karar bulur?" Cevaptan emin değilseniz **Erişim önizlemesi** tarayıcıyla açılan sayfalar (`GET`) için aynı sırayı kişiler, servis token'ları ve paylaşım bağlantıları için gösterir; hız sınırını, `POST` gibi diğer yöntemleri ve özel ağı hesaba katmaz.

---

## Sık yapılan hatalar

| Belirti | Neden | Çözüm |
|---|---|---|
| Herkes **Erişiminiz yok** görüyor | Giriş açık ama paylaşım yok | **Kişiler** sekmesinde paylaşım ekleyin. |
| Ofisteyken giriş sayfası çıkıyor (ya da **İkisi birden gereksin** seçiliyse **Bu servise erişim kısıtlı**) | Servis, listedekinden farklı bir adres görüyor (ör. konsola IPv6, servise IPv4) | Ofisten giriş yapıp bir sayfa açın; **Etkinlik** sekmesindeki **Sayfayı açtı** satırının **Adres** sütunu servisin gördüğü adresi gösterir (**İkisi birden gereksin** seçiliyse **Bu servise erişim kısıtlı** sayfasındaki **Adresiniz** kutusu da gösterir). Bu adresi listeye ekleyin; IPv4 ve IPv6'yı birlikte ekleyin. |
| Müşteri hiç giremiyor, 403 alıyor | **İkisi birden gereksin** seçili | **Biri yeterli**'yi seçin ya da müşterinin adresini listeye ekleyin. |
| Bir üyeyi sayfayla sınırladım ama her yeri açıyor | **Organizasyonunuz** paylaşımı tüm siteyi açıyor | En geniş paylaşım kazanır; organizasyon paylaşımını da sınırlayın ya da kişi kuralı kullanın. |
| CI sağlık kontrolü `302` döndürüyor | Token isteğe ulaşmadı: başlığın adı yanlış yazıldı, `KOMUTA_SERVICE_TOKEN` gizli değişkeni boş ya da başka adla kaydedildi, ya da ilk token'dan sonra yönlendirmeler hâlâ yenileniyor | Başlığın adının `x-komuta-service-token`, gizli değişkenin adının `KOMUTA_SERVICE_TOKEN` olduğunu kontrol edin; ilk token'dan sonra birkaç dakika bekleyin. |
| CI token'ı `401` alıyor | Token bitti, silindi ya da değeri eksik veya yanlış kopyalandı | Yeni bir token oluşturup gizli değişkeni güncelleyin. |
| CI token'ı `403` alıyor | Yol token'ın kapsamı dışında ya da IP kuralı "ikisi birden" | **Neleri açabilir**'i kontrol edin; site kuralını **Biri yeterli** yapın. |
| Webhook'lar `403` alıyor | Gönderici listesi eksik ya da eskimiş | `https://api.github.com/meta` adresindeki `hooks` listesinin tamamını (IPv6 dahil) ekleyin; **Etkinlik**'te **İzinli olmayan bir adresten geldi** satırı reddedilen ağı gösterir. |
| Webhook'lar `302` ya da `401` alıyor | Yöntem seçilmemiş ya da yol yanlış | Açık yolun yöntemlerini ve yolunu kontrol edin. |
| İmzalı bir webhook yolu her isteğe `401` veriyor | Henüz imza sırrı yok ya da gönderici başka bir sırla imzalıyor | Yolun altına sırrı ekleyin ve göndericide aynısını kullanın; **Etkinlik**'teki nedene bakın. |
| Bazı webhook'lar `413` alıyor | Gövde, imzalı bir yolun doğrulayabileceği en büyük boyut olan 65.535 bayttan büyük | Bu gönderici için imza kontrolünü uygulamanızda tutun. |
| Uygulama bazı isteklerde kimlik başlığını boş alıyor | Ziyaretçi ofis adresinden geldi (giriş yapmış olsa bile) ya da sayfa giriş gerektirmiyor | Kimlik gereken yolları bir giriş kuralıyla koruyun. |
| Ekip bir anda yeniden giriş yapmak zorunda kaldı | Biri **Herkesi çıkar**'ı seçti ya da tek tek çıkarma henüz devrede değilken bir paylaşım kaldırıldı veya sınırlandı | Beklenen davranıştır; bkz. Adım 8.3. |
| Başka bir sitedeki tarayıcı CORS hatası alıyor | Tarayıcının `OPTIONS` kontrolü oturum taşımıyor ve `401` alıyor | **Makineler → Yöntemler ve CORS → CORS kontrollerine girişsiz izin ver**'i açın. |
| İstekler `405` alıyor | **Makineler** sekmesindeki bir yöntem kuralı o yolda bu yönteme izin vermiyor | Yöntemi kurala ekleyin; `Allow` başlığı yolun kabul ettiklerini listeler. |
| İstekler `429` alıyor | Adres hız sınırını aştı | Sınırı yükseltin ya da daha uzun bir zaman aralığı seçin; giriş ve webhook isteklerinin de sayıldığını unutmayın. |
| Hiçbir IP listesine takılmayan bir ziyaretçi **Bu servise erişim kısıtlı** görüyor | Ülkesi **Ülkeler** listesinde değil ya da anlaşılamıyor | **Etkinlik** sekmesinde "İzin verilmeyen bir ülkeden geldi" görünür; ülkeyi ekleyin. |
| Paylaşım bağlantısıyla giren biri "The share link you opened this site with has ended." görüyor | Bağlantı silindi, süresi doldu ya da herkes çıkarıldı | Yeni bir bağlantı oluşturup yeniden gönderin. |

---

## İlgili Dokümanlar

- [Erişim Koruması](service-access-protection.md) — genel bakış, durumlar ve uyarılar.
- [Kurallar](access-protection-rules.md) — IP listesi, yol kuralları ve erişim önizlemesi ayrıntıları.
- [Giriş ve Paylaşım](access-protection-sign-in-sharing.md) — paylaşım türleri ve ziyaretçinin gördükleri.
- [Makineler ve Özel Ağ](access-protection-machines.md) — webhook yolları, servis token'ları, özel ağ.
- [Erişim Kaydı](access-protection-activity.md) — kaydın okunması.
- [Bitiş ve Kimlik Bildirme](access-protection-settings.md) — bitiş tarihi ve JWT doğrulama örnekleri.
- [Başvuru](access-protection-reference.md) — sınırlar, yanıtlar ve hata mesajları.
