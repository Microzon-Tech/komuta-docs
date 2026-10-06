# Makineler ve Özel Ağ

Komuta girişi insanlar içindir: tarayıcıda bir giriş sayfası açılır, kişi hesabıyla giriş yapar. Bazı istekler ise bir insandan değil bir programdan gelir. GitHub bir push olduğunda uygulamanıza haber verir, Stripe bir ödeme olduğunda bildirim gönderir, CI hattınız deploy sonrası bir sağlık kontrolü yapar, bir izleme aracı her dakika sitenizi yoklar. Bu programlar giriş sayfasını kullanamaz.

Erişim koruması bu tür istekler için üç yol sunar:

| Yol | Ne için | Nerede |
|---|---|---|
| **Webhook yolu** (açık yol) | Giriş yapamayan ve kendi imzasını gönderen göndericiler (GitHub, Stripe, Slack gibi) | **Makineler** sekmesi → **Webhook'lar** |
| **Servis token'ı** | Sizin kontrol ettiğiniz programlar (CI işleri, izleme araçları, betikler) | **Makineler** sekmesi → **Servis token'ları** |
| **Özel ağdan gelebilecek servisler** | Diğer kümelerinizdeki Komuta servislerinizin bu servise doğrudan ulaşması | **Makineler** sekmesi → **Özel ağdan doğrudan gelebilecek servisler** (özel ağın kendisi **Ağ** sekmesinde) |

Webhook ve servis token'ı bölümleri yalnızca erişim koruması açıkken görünür. Koruma kapalıysa sekmede **Erişim koruması kapalı** notu ve **Kurallara git** düğmesi görünür. Özel ağ listesi bunun tek istisnasıdır: servisin özel ağı açıksa, listeyi korumayı açmadan önce de hazırlayabilirsiniz.

---

## Webhook yolları

Webhook yolu, sitenin belirli bir yolunu Komuta girişi olmadan açar. Örneğin site yalnızca ekibinize açıkken `/webhooks/github` yoluna GitHub'ın gönderdiği istekler giriş yapmadan uygulamanıza ulaşır.

> **Önemli:** Webhook yolundaki istekleri Komuta kimlik açısından denetlemez; kapıyı yalnızca açar. İsteğin gerçekten GitHub'dan ya da Stripe'tan geldiğini **uygulamanız**, göndericinin imzasını doğrulayarak kontrol etmelidir. Örneğin GitHub `X-Hub-Signature-256`, Stripe `Stripe-Signature` başlığını gönderir. İmzayı doğrulamayan bir uygulamada bu yola herkes istek gönderebilir.

### Webhook yolu açma

1. **Makineler** sekmesinde **Webhook'lar** bölümünde **Yol aç** düğmesine tıklayın.
2. **Webhook için yol aç** penceresinde:
   - **Yol** — açılacak yol, örneğin `/webhooks/github`. Bu yol ve altındaki yollar açılır.
   - **Yöntemler** — bu yolda kabul edilecek HTTP yöntemleri: `POST`, `PUT`, `PATCH`, `DELETE`, `GET`, `HEAD`. Varsayılan yalnızca `POST`'tur; en az bir yöntem seçilmelidir.
   - **Gönderici adresleri (isteğe bağlı)** — her satıra bir IP adresi ya da CIDR aralığı. Doldurursanız yola yalnızca bu adreslerden gelen istekler girer. Boş bırakırsanız her adresten istek kabul edilir; imza doğrulaması yine sizi korur. GitHub'ın webhook adresleri gibi göndericinin yayımladığı aralıkları buraya yazabilirsiniz (pencere örnek olarak `140.82.112.0/20` gösterir).
3. **Yolu aç** ile kaydedin. Değişiklik birkaç saniye içinde uygulanır ("Webhook yolları uygulanıyor").

Listede her açık yol; yolu, seçili yöntemleri, gönderici listesini ("Yalnızca: …" ya da "Her adresten") ve servisin tam adresini gösterir. Bir yolu kapatmak için satırdaki çöp kutusu simgesine tıklayın; yol onay sorulmadan hemen kapatılır.

### Webhook yolları nasıl çalışır

- **Giriş istenmez.** Seçili yöntemlerle gelen istekler Komuta girişi olmadan geçer.
- **Yalnızca seçili yöntemler açıktır.** Başka bir yöntemle gelen istek, açık yol yokmuş gibi değerlendirilir; yani sitenin normal korumasından geçmesi gerekir.
- **Sitenin IP listesi ve üst yol kuralları uygulanmaz.** Açık yolda tek adres kontrolü, yolun kendi **Gönderici adresleri** listesidir.
- **Engelleme kuralları yine geçerlidir.** Açık yolun altına yalnızca **Tamamen engelle** kuralı konabilir; örneğin `/webhooks` açıkken `/webhooks/eski` engellenebilir. Açık yolun altına giriş, IP ya da kişi kuralı konamaz; arayüz bunu "Bu yol {kural} kuralıyla çakışıyor" uyarısıyla engeller.
- **Yol eşleşmesi büyük/küçük harf duyarsızdır.** `/HOOKS/x` isteği `/hooks` yolunun altındadır. Bir istek ancak yolunun okunabileceği her biçim açık yolun altında kalıyorsa açılır; `%2f`, `..` ya da benzeri hilelerle açık yoldan başka bir yola kaçılamaz.
- **Birden fazla açık yol bir isteği kapsıyorsa en özgül olan karar verir.**
- **Yöntem değiştirme başlıkları dikkate alınmaz.** Komuta isteğin gerçek yöntemine bakar. Uygulamanız bu yollarda `X-HTTP-Method-Override` ya da `X-HTTP-Method` gibi başlıkları kabul etmemelidir; aksi halde yalnızca `POST` açtığınız bir yola `DELETE` gibi davranan istekler gönderilebilir.
- **CORS ön uçuş istekleri (`OPTIONS`) açık yoldan geçmez.** Seçilebilen yöntemler arasında `OPTIONS` yoktur. Tarayıcıdan başka bir siteden çağrılması gereken uç noktalar için webhook yolu uygun değildir.
- **Erişim kaydı** açık yola gelen her teslimatı **Açık yol isteği** / **Açık yola teslim edildi** olarak, tam yol yerine açık yolun önekiyle kaydeder.

### Sınırlar

- Bir serviste en fazla **10** açık yol olabilir. Sınıra ulaşıldığında **Yol aç** düğmesi devre dışı kalır. Açık yollar, **Kurallar** sekmesindeki yol kurallarıyla birlikte toplam 50 yol kuralı sınırına da dahildir.
- `/` (sitenin tamamı) ve Komuta'nın kendi giriş yolu (`/.komuta-access` ile başlayanlar) açılamaz.
- Yol, [yol kurallarıyla](access-protection-rules.md#yol-yazım-kuralları) aynı yazım kurallarına uyar (en fazla 256 karakter). Pencere hatalı her yol için aynı mesajı gösterir: "/webhooks/github gibi bir yol girin. Sitenin tamamı açılamaz."
- Gönderici listesi en fazla 100 girdi alır ve yalnızca genel internet adreslerini kabul eder. Listede geçersiz bir satır varsa **Yolu aç** düğmesi devre dışı kalır.
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
- **Herkese açık yollarda okunmaz.** Zaten herkese açık bir yolda token'a bakılmaz.
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
- Erişim kaydında token'la yapılan istekler **Servis token'ı: {ad}** olarak görünür.

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

- [Kurallar](access-protection-rules.md) — sitenin ve yolların korunması.
- [Erişim Kaydı](access-protection-activity.md) — webhook teslimatlarının ve token kullanımının görüldüğü yer.
- [Bitiş ve Kimlik Bildirme](access-protection-settings.md) — token'la gelen isteğin uygulamaya nasıl tanıtıldığı.
- [Başvuru](access-protection-reference.md) — tüm sınırlar ve yanıtlar.
