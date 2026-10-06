# Erişim Kaydı

Erişim kaydı, korunan servisinize kimin giriş yaptığını, giriş yapanların hangi sayfaları açtığını ve kimin neden geri çevrildiğini gösterir. **Erişim ve portlar** sayfasındaki **Etkinlik** sekmesinde bulunur. **Genel bakış** sekmesindeki **Son etkinlik** kutusu son 5 kaydı gösterir; **Tümünü gör** sizi **Etkinlik** sekmesine götürür.

Kayıt, erişim koruması açıkken tutulur. Koruma kapalıyken sekme "Erişim kaydı, erişim koruması açıkken tutulur." der. Kaydı görmek için **Servis erişim korumasını yönet** ya da **Korunan servisin paylaşımlarını yönet** izinlerinden biri gerekir.

---

## Neler kaydedilir

| Olay | Ne zaman | Listede |
|---|---|---|
| **Giriş** | Bir ziyaretçi Komuta ile giriş yapıp servise döndüğünde | **Giriş yaptı** |
| **Sayfa görüntüleme** | Giriş yapmış bir ziyaretçi bir sayfa açtığında | **Sayfayı açtı** |
| **Token ile erişim** | Bir program servis token'ıyla bir sayfa açtığında (`GET`, dosya olmayan yollar) | **Sayfayı servis token'ıyla açtı** |
| **Webhook teslimatı** | Bir açık yola istek geldiğinde | **Açık yola teslim edildi** |
| **Ret** | Bir ziyaretçi kurallarınız yüzünden geri çevrildiğinde | Rettin nedeni (aşağıdaki tablo) |

Neler **kaydedilmez**:

- **Dosya istekleri.** Sayfa görüntüleme olarak yalnızca `GET` istekleri ve son bölümünde nokta olmayan yollar sayılır. `/app.js`, `/logo.png`, `/style.css` gibi istekler kaydedilmez; böylece kayıt gerçek sayfa açılışlarını gösterir.
- **Girişin gerekmediği yerlerdeki ziyaretler.** Sayfa açılışları yalnızca girişin gerektiği sayfalarda, giriş yapmış ziyaretçiler ve token'lar için kaydedilir. Herkese açık yollarda ve IP listesiyle girişsiz geçilen sayfalarda kimse kaydedilmez; ziyaretçi giriş yapmış olsa bile.
- **Giriş sayfasına yönlendirmeler.** Giriş yapmamış bir ziyaretçinin giriş sayfasına gönderilmesi ve oturumsuz program isteklerine dönen `401` ret sayılmaz; kayıtta görünmez.
- **Platform kaynaklı sorunlar.** Komuta tarafındaki geçici bir arıza yüzünden verilemeyen cevaplar ziyaretçi retti sayılmaz.
- **Komuta'nın imzalı test girişi.** Komuta, korumanın çalıştığını doğrulamak için giriş yapmış gibi bir test isteği gönderir; bu kayda yazılmaz. Koruma açılırken yapılan anonim kontroller ise IP listesi ya da **Tamamen engelle** kuralı olan servislerde birkaç **Giriş yapmamış** reddi olarak görünebilir.

---

## Listeyi okumak

Kayıtlar güne göre gruplanır; her günün başında uzun tarih yazar. Sütunlar:

| Sütun | İçerik |
|---|---|
| **Saat** | Olayın son görüldüğü saat (hesabınızda seçtiğiniz saat diliminde). |
| **Kim** | Ziyaretçinin adı (ve e-postası), ya da aşağıdaki etiketlerden biri. |
| **Ne oldu** | Olay ya da ret nedeni. |
| **Sayfa** | HTTP yöntemi ve yol, örneğin `GET /raporlar`. Sorgu dizesi (`?…`) kaydedilmez; yol 256 karakterden sonra kesilir. |
| **Adres** | Ziyaretçinin IP adresi (aşağıya bakın). |
| **Adet** | Aynı olayın kaç kez yaşandığı. |

Dar ekranlarda yalnızca **Saat** ve **Kim** sütunları görünür; neden ve sayfa adın altına yazılır.

### Kim sütunundaki etiketler

| Etiket | Anlamı |
|---|---|
| Ad ve e-posta | Organizasyonunuzdan, giriş yapmış bir kişi. E-posta yalnızca kullanıcıları görme izni olanlara gösterilir. |
| **Başka bir organizasyondan biri** | Organizasyonunuzun dışından giriş yapmış biri (bağlı bir organizasyon ya da e-posta paylaşımıyla gelen biri). Adı ve e-postası gösterilmez. |
| **Servis token'ı: {ad}** | Bu adlı servis token'ını kullanan bir program. |
| **Silinmiş bir servis token'ı** | Sonradan silinmiş bir token. |
| **Açık yol isteği** | Bir webhook yoluna gelen istek. |
| **Giriş yapmamış** | Giriş yapmamış bir ziyaretçi. |
| **Diğer ziyaretçiler** | O saatte kayda sığmayan diğer olaylar, tek satırda toplandı (aşağıya bakın). |

### Giriş yapmamış ziyaretçilerin retleri

Giriş yapmamış birinin geri çevrilmesi, kaydın kötüye kullanılmasını önlemek için daha az ayrıntıyla kaydedilir:

- **Sayfa** sütununda gerçek yol yerine **Herhangi bir sayfa** yazar. Böylece sitenizi tarayan biri kaydı binlerce farklı yolla dolduramaz.
- **Adres** sütununda tam adres yerine ağın ilk kısmı yazar: IPv4'te `/24` (örneğin `203.0.113.0/24`), IPv6'da `/48`.
- İstisna: ret bir **Tamamen engelle** kuralından ya da bir açık yoldan geliyorsa, **Sayfa** sütununda o kuralın yolu (örneğin `/internal`) görünür. Bu yolları zaten siz seçtiğiniz için hangi kuralın çalıştığını görebilirsiniz.

Giriş yapmış ziyaretçilerin adresi tam olarak gösterilir.

Ziyaretçinin adresi, istek Cloudflare üzerinden doğrulanabildiğinde görünür; doğrulanamadıysa sütunda `—` yazar.

### Ne oldu sütunundaki nedenler

| Metin | Anlamı |
|---|---|
| **Giriş yaptı** | Komuta ile giriş tamamlandı. |
| **Sayfayı açtı** | Giriş yapmış ziyaretçi sayfayı açtı. |
| **Sayfayı servis token'ıyla açtı** | Bir program token'la sayfayı açtı. |
| **Açık yola teslim edildi** | Webhook yoluna istek ulaştı. |
| **Bu sayfa kendisiyle paylaşılmamış** | Ziyaretçi giriş yaptı ama bu sayfa onunla paylaşılmamış (paylaşımı belirli sayfalarla sınırlı ya da yol yalnızca seçilen kişilere açık). |
| **Sayfa engelli** | **Tamamen engelle** kuralına takıldı. |
| **İzinli olmayan bir adresten geldi** | IP izin listesinde olmayan bir adresten geldi. |
| **İstek Komuta kenarından gelmedi** | İsteğin Cloudflare üzerinden geldiği doğrulanamadı; IP listesi isteyen bir sayfada adres güvenilir kabul edilmedi. |
| **Giriş bağlantısı geçersiz ya da süresi dolmuş** | Ziyaretçi bozuk ya da eski bir giriş bağlantısıyla döndü. |
| **Komuta girişi reddetti** | Giriş bağlantısı Komuta tarafından reddedildi (örneğin daha önce kullanılmış). |
| **Bilinmeyen ya da süresi dolmuş bir servis token'ı gönderdi** | Geçersiz token. |
| **Servis token'ı bu sayfayı açamaz** | Token geçerli ama bu sayfa kapsamında değil. |
| **Bu saatteki diğer ziyaretler, birlikte gruplandı** | Kayda sığmayan olayların toplamı. |
| **Reddedildi ({kod})** | Arayüzün tanımadığı bir neden; parantez içinde teknik kod yazar. |

---

## Gruplama ve sayılar

Kayıt her isteği ayrı satır olarak tutmaz:

- **15 saniyelik paketler.** Aynı olay (aynı kişi, aynı sonuç, aynı neden, aynı yöntem, aynı yol, aynı adres) 15 saniye içinde birden çok kez olursa tek olay olarak, sayısıyla birlikte gönderilir.
- **Saatlik satırlar.** Aynı olaylar bir saat boyunca tek satırda toplanır; **Adet** sütunu artar ve **Saat** son görüldüğü zamanı gösterir.
- **Saat başına sınır.** Bir servis için bir saatte en fazla 500 farklı satır tutulur (girişler bu sınıra dahil değildir). Fazlası **Diğer ziyaretçiler** / **Bu saatteki diğer ziyaretler, birlikte gruplandı** satırında toplanır.
- **Yoğun anlar.** Çok yoğun bir saldırı ya da tarama anında, diğer servislerin ve girişlerin kaydı bozulmasın diye bir servis için 15 saniyede kaydedilebilecek sayfa görüntüleme ve ret sayısı sınırlıdır. Bu sınırı aşan olaylar kayda hiç yazılmaz; **Diğer ziyaretçiler** satırında da sayılmaz. Bu yüzden kayıt bir güvenlik denetim kaydı değil, "kim geldi, kim geri çevrildi" görünümüdür.

Kayıtlar listede birkaç saniye ile yaklaşık yarım dakika arasında bir gecikmeyle görünür.

---

## Filtreler ve sayfalar

- **Zaman aralığı** — **Son 24 saat**, **Son 7 gün** ya da **Son 30 gün** (varsayılan).
- **Göster** — **Tümü**, **Girişler**, **Sayfa görüntülemeleri**, **Retler**.
- Her sayfada 50 kayıt gösterilir. **Daha yeni** ve **Daha eski** düğmeleriyle gezilir; altta "{toplam} kayıttan {ilk}–{son}" yazar. Toplam 10.000'i aşarsa sayının yanında `+` görünür.

Liste kendiliğinden yenilenmez; yeni kayıtları görmek için filtreyi değiştirin ya da sayfayı yenileyin.

---

## Saklama süresi

Kayıtlar **30 gün** saklanır; bir satır, son görüldüğü zamandan 30 gün sonra silinir. Kaydı dışa aktarma seçeneği şu an yoktur.

---

## İlgili Dokümanlar

- [Giriş ve Paylaşım](access-protection-sign-in-sharing.md) — ziyaretçilerin gördüğü sayfalar.
- [Kurallar](access-protection-rules.md) — retlerin nedeni olan kurallar.
- [Makineler ve Özel Ağ](access-protection-machines.md) — token ve webhook kayıtları.
- [Başvuru](access-protection-reference.md) — teknik neden kodları.
