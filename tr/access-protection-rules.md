# Kurallar: Kimler Girebilir

Kurallar, servisinize kimin girebileceğini belirler. **Erişim ve portlar** sayfasının **Kurallar** sekmesinde, **Erişim koruması** kartında bulunur. Korumanın açma anahtarı ve durum etiketi de bu kartın başlığındadır (bkz. [Erişim Koruması](service-access-protection.md#korumayı-açma)).

Kurallar iki katmandan oluşur:

1. **Sitenin tamamı için** — **Kimler girebilir** bölümü: Komuta girişi, IP izin listesi ve ikisinin nasıl birleşeceği.
2. **Belirli yollar için** — **Yol kuralları** bölümü: `/admin`'i yalnızca giriş yapanlara açmak, `/internal`'ı tamamen kapatmak, `/raporlar`'ı yalnızca seçilen kişilere belirli saatlerde açmak gibi.

Kartta yaptığınız değişiklikler **Korumayı uygula** düğmesine basana kadar kaydedilmez. Kartın altındaki özet kutusu **Şu anda** etkin olan durumu ve kaydedilmemiş değişiklik varsa **Uyguladıktan sonra** ne olacağını tek cümleyle anlatır.

---

## Kimler girebilir

Bu bölümde sitenin tamamına uygulanan iki kontrol vardır. Biri, ikisi ya da hiçbiri açık olabilir (hiçbiri açık değilse en az bir yol kuralı gerekir).

### Komuta girişi iste

Açıkken ziyaretçi servisinizi açmadan önce Komuta'ya giriş yapmaya yönlendirilir. Yalnızca **Kişiler** sekmesindeki bir paylaşımla eşleşen hesaplar içeri girer. Giriş yapan biri bir paylaşımla eşleşmiyorsa **Erişiminiz yok** sayfasını görür.

Ziyaretçinin bu süreçte ne gördüğü ve paylaşımların nasıl çalıştığı [Giriş ve Paylaşım](access-protection-sign-in-sharing.md) sayfasında anlatılır.

### IP izin listesi

Servise yalnızca listedeki genel internet adreslerinden ulaşılabilir. Her satıra bir IPv4 ya da IPv6 adresi veya CIDR aralığı yazılır:

```text
8.8.8.8
8.8.8.0/24
2001:4860::/32
```

Kurallar:

- **Yalnızca genel internet adresleri kabul edilir.** Özel, ayrılmış ve dokümantasyon aralıkları ile bunlarla çakışan aralıklar reddedilir: `0.0.0.0/8`, `10.0.0.0/8`, `100.64.0.0/10`, `127.0.0.0/8`, `169.254.0.0/16`, `172.16.0.0/12`, `192.0.0.0/24`, `192.0.2.0/24`, `192.168.0.0/16`, `198.18.0.0/15`, `198.51.100.0/24`, `203.0.113.0/24`, `224.0.0.0/4`, `240.0.0.0/4`, `::/128`, `::1/128`, `::ffff:0:0/96`, `64:ff9b:1::/48`, `100::/64`, `2001:db8::/32`, `fc00::/7`, `fe80::/10`, `ff00::/8`.
- **Aralığın host bitleri sıfır olmalıdır.** `8.8.8.0/24` geçerlidir, `8.8.8.5/24` geçerli değildir.
- **Tüm adresleri kapsayan aralıklar kabul edilmez.** `0.0.0.0/0` ve `::/0` reddedilir; herkese açmak istiyorsanız korumayı kapatın.
- Tek bir adres `/32` (IPv4) ya da `/128` (IPv6) olarak kaydedilir. Başında sıfır olan sayılar (`08.8.8.8`), `%` ile bölge belirtilen IPv6 adresleri ve IPv4'ü taşıyan IPv6 biçimi (`::ffff:8.8.8.8`) kabul edilmez.
- Satırlar virgülle de ayrılabilir. Aynı girdi iki kez yazılırsa bir kez kaydedilir.
- Liste en fazla **100** girdi ve toplam 4600 karakter alır.

Kart hatalı satırları siz yazarken "Satır {n}" diye işaretler ve hata düzelene kadar **Korumayı uygula** düğmesi kapalı kalır.

### Kendi IP'mi ekle

**Kendi IP'mi ekle** düğmesi, Komuta konsolunun gördüğü genel adresinizi listeye ekler (yalnızca taslağa; kaydetmek için **Korumayı uygula** gerekir):

- IPv4 bağlantısında yalnızca adresiniz eklenir (`/32`).
- IPv6 bağlantısında adresiniz kendi `/64` bloğu içinde değişebildiği için bloğun tamamı eklenir ve kart bunu belirtir.
- Adresiniz zaten listedeyse "Adresiniz ({adres}) zaten listede." yazar.
- Bağlantınız için genel bir adres belirlenemezse hiçbir şey eklenmez.

> **Kendinizi dışarıda bırakmayın.** Eklenen adres, konsolun gördüğü adrestir. Tarayıcınız servise farklı bir bağlantıyla ulaşıyor olabilir (örneğin konsola IPv6, servise IPv4); bu durumda servis başka bir adres görür. Uygulamadan önce kontrol edin; gerekirse hem IPv4 hem IPv6 adresinizi ekleyin. Yine de dışarıda kalırsanız **Erişim kısıtlı** sayfası servisin gördüğü adresi gösterir; bu adresi kopyalayıp listeye ekleyebilirsiniz.

### Adres nasıl belirlenir

Komuta ziyaretçinin adresini yalnızca Cloudflare'in yazdığı bilgiden alır ve isteğin gerçekten Cloudflare üzerinden geldiğini ayrıca doğrular. Ziyaretçinin kendi gönderdiği `X-Forwarded-For`, `X-Real-IP`, `True-Client-IP` ya da `Forwarded` gibi başlıklar dikkate alınmaz; adres taklit edilerek listeye girilemez.

Bu yüzden IP listesi (site için, bir yol kuralında ya da bir webhook yolunda), servise gelen isteklerin Cloudflare üzerinden geçmesini gerektirir. Komuta'nın verdiği `*.komuta.app` adresleri ve **Alan adları** altında eklenen özel alan adları bu şekilde çalışır. Cloudflare üzerinden geldiği doğrulanamayan bir istek, IP listesi isteyen bir yerde reddedilir (erişim kaydında "İstek Komuta kenarından gelmedi"); **Biri yeterli** seçiliyse bunun yerine giriş sayfasına yönlendirilir.

### Giriş ve IP izin listesi nasıl birleşsin?

İki kontrol birlikte açıkken kart bu soruyu sorar:

| Seçenek | Ne olur |
|---|---|
| **İkisi birden gereksin** (varsayılan) | Ziyaretçi listedeki bir adresten gelmeli, ardından Komuta ile giriş yapmalı. Diğer adresler 403 alır ve giriş sayfasını hiç görmez. |
| **Biri yeterli** | Listedeki adreslerden gelenler giriş yapmadan girer; diğer herkes Komuta ile giriş yapar ve yalnızca paylaştığınız kişi ve organizasyonlar geçer. Ofis ağından girişsiz, dışarıdan girişle erişim için kullanılır. |

Tek bir kontrol açıkken soru görünmez:

- **Yalnızca IP izin listesi** — listedeki adreslerden gelenler giriş yapmadan girer, diğer herkes 403 alır. Kart bunu "Giriş kapalı" uyarısıyla belirtir.
- **Yalnızca Komuta girişi** — adresi ne olursa olsun, paylaşımla eşleşen her hesap girer.
- **İkisi de kapalı, yol kuralı var** — sitenin kendisi herkese açık kalır, yalnızca yol kurallarındaki yollar korunur.
- **İkisi de kapalı, yol kuralı yok** — korunacak bir şey kalmaz. Düğme kırmızı **Korumayı kapat** olur; basarsanız **Erişim koruması kapatılsın mı?** onayından sonra koruma kapanır.

---

## Yol kuralları

Yol kuralları, sitenin geri kalanına dokunmadan belirli yolları korur. **Yol kuralları** bölümünde **Yol kuralı ekle** ile eklenir; her kuralda bir **Yol** ve bir **Koruma** türü seçilir. Yeni bir kuralın varsayılan türü **Komuta girişi**'dir.

| Koruma | Ne yapar |
|---|---|
| **Tamamen engelle** | Bu yola ve altındaki yollara kimse erişemez (403). Giriş yapmak, izinli bir adresten gelmek ya da servis token'ı kullanmak da erişim sağlamaz. |
| **Komuta girişi** | Bu yola gelenler Komuta ile giriş yapmalı; yalnızca servisi paylaştığınız kişi ve organizasyonlar geçer. |
| **IP listesi** | Bu yola yalnızca kuralın kendi listesindeki adresler erişir; site kuralına ek olarak uygulanır. |
| **IP listesi ve Komuta girişi** | Kuralın kendi IP listesi ve giriş birlikte kullanılır; **İkisi birden gereksin** ya da **Biri yeterli** seçilir ("Bu yolda IP listesi ve giriş nasıl birleşsin?"). |
| **Yalnızca seçilen kişiler** | Bu yola gelenler Komuta ile giriş yapar; yalnızca işaretlediğiniz paylaşımlar, belirlediğiniz saatlerde geçer. Paylaştığınız diğer herkes sitenin geri kalanına girmeye devam eder. |

Kuralın IP listesi, sitenin listesiyle aynı kurallara uyar (en fazla 100 girdi, yalnızca genel adresler) ve **IP listesi** içeren türlerde en az bir adres girilmelidir. Kural satırındaki **Kendi IP'mi ekle** düğmesi de kullanılabilir.

### Kurallar yalnızca sıkılaştırır

Bir istek hem sitenin kontrollerini **hem de** yoluyla eşleşen **her** kuralı geçmelidir. Bir yol kuralı sitenin istediği bir şeyi kaldıramaz, yalnızca ek koşul getirir. Örnekler:

- Site **Komuta girişi** istiyor, `/ops` için **IP listesi** kuralı var → `/ops`'a girmek için hem giriş yapmak hem listedeki bir adresten gelmek gerekir.
- Sitenin tamamı zaten giriş istiyorsa `/admin` için eklenen **Komuta girişi** kuralı bir şey değiştirmez; kart kuralın altında bunu belirtir ("Sitenin tamamı zaten Komuta girişi istiyor; bu kural bir şeyi değiştirmez.").
- `/internal` **Tamamen engelle** ise oturumu olan ya da izinli adresten gelen biri de 403 alır.
- `/admin` için **Komuta girişi**, `/admin/raporlar` için **IP listesi** kuralı varsa `/admin/raporlar`'a girmek için ikisi de gerekir.

Herkese açık bir yolun girişsiz açılması gerekiyorsa (örneğin webhook), yol kuralı değil [webhook yolu](access-protection-machines.md#webhook-yolları) kullanılır; webhook yolları sitenin kurallarından muaftır.

### Yol eşleşmesi

- Bir kural, yolun kendisiyle ve altındaki yollarla eşleşir: `/admin` kuralı `/admin` ve `/admin/ayarlar` ile eşleşir, `/administrator` ile eşleşmez.
- Büyük/küçük harf fark etmez. Yazdığınız yol küçük harfe çevrilir, sondaki `/` silinir: `/Admin/` yazarsanız `/admin` olarak kaydedilir.
- Sorgu dizesi (`?x=1`) eşleşmeyi etkilemez.
- **Kural atlatılamaz.** Kodlanmış karakterler (`/%61dmin`, `%2e%2e`, `%252e`), `//`, `/./`, `/../`, `;parametre`, `\` ve büyük harf hileleri bir kuralı atlatmak için kullanılamaz. Komuta isteğin yolunu, arkadaki bir sunucunun okuyabileceği her biçimde okur ve bu biçimlerden herhangi biri bir kurala uyuyorsa kuralı uygular. Bu, fazladan bir kuralın uygulanmasına yol açabilir, ama hiçbir zaman bir kuralın atlanmasına yol açmaz.
- Yol kuralı ya da sayfa sınırlı bir paylaşımı olan bir serviste, yolu 1024 bayttan uzun olan, okunamayacak kadar karmaşık olan (çok katmanlı kodlama, çok sayıda okuma biçimi) ya da kontrol karakteri içeren istekler `400 bad request` ile reddedilir. Normal tarayıcı istekleri bundan etkilenmez.

### Yol yazım kuralları

- Yol `/` ile başlamalıdır ve yalnızca küçük harf, rakam ve `- . _ ~ ! $ & ' ( ) * + , = : @ /` içerebilir. ASCII dışı karakterler (ör. Türkçe harfler) ve boşluk kullanılamaz.
- Boş segment (`//`), `.` ve `..` segmentleri ile `.` ile biten segmentler kullanılamaz.
- `/` tek başına kural olamaz ("“/” tüm siteyi kapsar. Bunun için yukarıdaki site ayarlarını kullanın.").
- `/.komuta-access` ile başlayan yollar Komuta'ya ayrılmıştır.
- Bir yol en fazla **256** karakter olabilir; her yol için yalnızca bir kural olabilir.
- Bir serviste en fazla **50** yol kuralı olabilir. **Makineler** sekmesindeki webhook yolları da bu 50'ye dahildir; kart sayacı ("{n} / 50 kural") yalnızca bu sekmedeki kuralları sayar.

Kart hatalı bir kuralı siz yazarken işaretler; kural düzeltilene kadar **Korumayı uygula** düğmesi kapalı kalır.

### Yalnızca seçilen kişiler

Bu tür, bir yolu servisi paylaştığınız kişilerden yalnızca bazılarına açar. Örneğin site tüm organizasyonunuza açıkken `/maaslar`'ı yalnızca iki kişiye, `/demo`'yu bir müşteriye yalnızca toplantı saatinde açabilirsiniz.

1. Kuralın türünü **Yalnızca seçilen kişiler** yapın.
2. **Bu yolu kimler açabilir** listesinde, servisin mevcut paylaşımlarından bu yola girebilecek olanları işaretleyin.
3. İsterseniz her kişi için **Başlangıç (isteğe bağlı)** ve **Bitiş (isteğe bağlı)** zamanı girin. Saatler hesabınızın saat dilimindedir; bitiş başlangıçtan sonra olmalıdır.
4. **Korumayı uygula** ile kaydedin.

Bilmeniz gerekenler:

- Liste yalnızca **Kişiler** sekmesindeki paylaşımlardan oluşur. Önce servisi kişiyle paylaşın, kaydedin; sonra burada seçin.
- Hiç kimse seçilmezse "Henüz kimse seçilmedi. Şimdi uygularsanız bu yolu kimse açamaz." uyarısı çıkar; kural geçerlidir ve yolu kimse açamaz.
- Saat aralığı her istekte yeniden kontrol edilir. Aralık başladığında kişi yeniden giriş yapmadan yolu açabilir; aralık bitince yol, açık oturumlar dahil birkaç saniye içinde kapanır.
- Bir kişiyi kurala sonradan eklediğinizde, o kişi daha önce giriş yapmışsa ilk denemesinde bir kez yeniden giriş sayfasına yönlendirilebilir; bu, yeni yetkinin oturumuna eklenmesi içindir.
- Kurala seçilen bir paylaşımı kaldırdığınızda kişi kuraldan da otomatik çıkarılır. Siz kuralı düzenlerken paylaşım başka bir yerde kaldırılırsa kart "Seçilen # kişiyle servis artık paylaşılmıyor." der; **Bu kuraldan kaldır** ile taslağı temizleyin.
- Bu türde IP listesi kullanılamaz ve **servis token'ları bu yolları açamaz**.
- Bir kurala en fazla 200 kişi seçilebilir. Paylaşımı belirli sayfalarla sınırlı bir kişiyi bir yol için seçerseniz o yol da kişinin açabileceği sayfalara eklenir; bir paylaşımın toplam sayfa sayısı 50'yi geçemez.

Seçilmemiş biri bu yolu açmaya çalışırsa **Bu sayfaya erişiminiz yok** sayfasını görür: "Bu bölüm yalnızca servis sahibinin seçtiği kişilere, belirlediği saatlerde açık."

---

## Erişim önizlemesi

**Erişim önizlemesi**, kayıtlı kurallarınızla ve paylaşımlarınızla kimin nereye girebildiğini hiçbir şeyi değiştirmeden gösterir. Koruma açıkken **Kurallar** sekmesinin en altında görünür.

İki görünümü vardır:

- **Bir sayfayı kim açar** — **Sayfa** alanına bir yol yazın (örneğin `/raporlar`). Liste; giriş yapmamış bir ziyaretçiyi ve her paylaşımı, girip giremeyeceğiyle birlikte gösterir.
- **Bir kişi neyi açar** — **Kişi** listesinden bir üye ya da paylaşım seçin. Liste; sitenin kökünü, her yol kuralını ve kişinin paylaşımındaki sayfaları gösterir.

Sitede ya da bir kuralda IP listesi varsa **Nereden geliyor** seçimi de çıkar: **İzinli ağların dışından** (varsayılan), **Tüm listelerdeki bir adresten** ya da **Belirli bir adresten**.

Her satırda yeşil onay (girer) ya da kırmızı çarpı (giremez) ve nedeni yazar:

| Neden | Anlamı |
|---|---|
| **Herkese açık** | Bu yolda koruma yok. |
| **Açık yoldan girer; imzayı uygulama doğrular** | Bir webhook yolu bu isteği giriş olmadan geçirir. Önizleme yalnızca `GET` isteklerini hesapladığı için bu satır yalnızca `GET` kabul eden açık yollarda görünür. |
| **İzinli bir ağdan girer** | IP listesindeki bir adresten giriş yapmadan girer. |
| **Giriş yapınca girer** | Komuta ile giriş yaparsa girer. |
| **İzinli bir ağdan giriş yapınca girer** | Hem izinli adres hem giriş gerekir. |
| **{yol} kuralı engelliyor** | Bir **Tamamen engelle** kuralı. |
| **İzinli bir ağdan gelmesi gerekiyor ({yol})** | Bu yol için adres listede değil. |
| **Giriş yapması gerekiyor ({yol})** | Giriş yapmamış biri. |
| **Servis bu kişiyle paylaşılmamış ya da paylaşım askıda veya bitmiş** | Paylaşım yok, askıda ya da süresi dolmuş. |
| **Kendisiyle paylaşılan sayfaların dışında** | Paylaşımı belirli sayfalarla sınırlı. |
| **{yol} için seçilen kişiler arasında değil** | Bir **Yalnızca seçilen kişiler** kuralı. |
| **{yol} erişimi {tarih} tarihinde başlıyor** / **bitti** | Kişinin saat aralığı dışında. |

Bir kişi yalnızca e-posta paylaşımıyla giriyorsa satırın sonunda "(e-postasını doğrulayarak)" yazar. Önizleme kaydedilmemiş değişiklikleri hesaba katmaz; kaydedilen kurallar henüz yayılıyorsa bunu belirtir.

---

## İlgili Dokümanlar

- [Erişim Koruması](service-access-protection.md) — açma, kapatma ve durumlar.
- [Giriş ve Paylaşım](access-protection-sign-in-sharing.md) — paylaşımlar ve ziyaretçinin gördükleri.
- [Makineler ve Özel Ağ](access-protection-machines.md) — webhook yolları ve servis token'ları.
- [Başvuru](access-protection-reference.md) — sınırlar ve hata mesajları.
