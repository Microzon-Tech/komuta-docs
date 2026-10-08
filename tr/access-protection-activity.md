# Erişim Kaydı

Erişim kaydı, korunan servisinize kimin giriş yaptığını, giriş yapanların ve paylaşım bağlantısıyla girenlerin hangi sayfaları açtığını ve kimin neden geri çevrildiğini gösterir. **Erişim ve portlar** sayfasındaki **Etkinlik** sekmesinde bulunur. **Genel bakış** sekmesindeki **Son etkinlik** kutusu son 5 kaydı gösterir; **Tümünü gör** sizi **Etkinlik** sekmesine götürür.

Kayıt, erişim koruması açıkken tutulur. Koruma kapalıyken sekme "Erişim kaydı, erişim koruması açıkken tutulur." der. Kaydı görmek için **Servis erişim korumasını yönet** ya da **Korunan servisin paylaşımlarını yönet** izinlerinden biri gerekir.

---

## Neler kaydedilir

| Olay | Ne zaman | Listede |
|---|---|---|
| **Giriş** | Bir ziyaretçi Komuta ile giriş yapıp servise döndüğünde | **Giriş yaptı** |
| **Sayfa görüntüleme** | Giriş yapmış bir ziyaretçi bir sayfa açtığında | **Sayfayı açtı** |
| **Token ile erişim** | Bir program servis token'ıyla bir sayfa açtığında (`GET`, dosya olmayan yollar) | **Sayfayı servis token'ıyla açtı** |
| **Paylaşım bağlantısının açılması** | Biri bir paylaşım bağlantısını açtığında (sohbet uygulamasındaki ya da e-posta tarayıcısındaki bağlantı önizlemesi de sayılır) | **Paylaşım bağlantısıyla girdi** |
| **Paylaşım bağlantısıyla erişim** | Bağlantıyla giren ziyaretçi bir sayfa açtığında (`GET`, dosya olmayan yollar) | **Sayfayı paylaşım bağlantısıyla açtı** |
| **Webhook teslimatı** | Bir açık yola istek geldiğinde | **Açık yola teslim edildi** |
| **Ret** | Bir ziyaretçi kurallarınız yüzünden geri çevrildiğinde | Rettin nedeni (aşağıdaki tablo) |

Neler **kaydedilmez**:

- **Dosya istekleri.** Sayfa görüntüleme olarak yalnızca son yol bölümünde nokta olmayan `GET` istekleri sayılır. `/app.js`, `/logo.png`, `/style.css` gibi istekler kaydedilmez; böylece kayıt gerçek sayfa açılışlarını gösterir.
- **Girişin gerekmediği yerlerdeki ziyaretler.** Sayfa açılışları yalnızca girişin gerektiği sayfalarda, giriş yapmış ziyaretçiler, token'lar ve paylaşım bağlantıları için kaydedilir. Korumasız yollarda (hiçbir kontrolün uygulanmadığı yollar; webhook yolları hariç) ve IP listesiyle girişsiz geçilen sayfalarda kimse kaydedilmez; ziyaretçi giriş yapmış olsa bile.
- **Giriş sayfasına yönlendirmeler.** Giriş yapmamış bir ziyaretçinin giriş sayfasına gönderilmesi ve oturumsuz program isteklerine dönen `401` ret sayılmaz; kayıtta görünmez.
- **Tarayıcıların CORS kontrolleri**; **CORS kontrollerine girişsiz izin ver** ile geçirilenler.
- **Sorgu dizesi.** Hiçbir zaman kaydedilmez; bu yüzden paylaşım bağlantısının gizli değeri kayda düşmez.
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
| **E-posta koduyla giriş yapan biri** | Komuta hesabı olmadan, bir paylaşımın içeri aldığı e-posta adresini kanıtlamış biri. E-postası kullanıcıları görme ya da paylaşımları yönetme izni olanlara gösterilir. |
| **Servis token'ı: {ad}** | Bu adlı servis token'ını kullanan bir program. |
| **Silinmiş bir servis token'ı** | Sonradan silinmiş bir token. |
| **Paylaşım bağlantısı: {ad}** | Bu adlı paylaşım bağlantısıyla giren bir ziyaretçi. |
| **Silinmiş bir paylaşım bağlantısı** | Sonradan silinmiş bir paylaşım bağlantısı. |
| **Açık yol isteği** | Bir webhook yoluna gelen istek. |
| **Giriş yapmamış** | Giriş yapmamış bir ziyaretçi. |
| **Diğer ziyaretçiler** | O saatte kayda sığmayan diğer olaylar, tek satırda toplandı (aşağıya bakın). |

### Giriş yapmamış ziyaretçilerin retleri

Giriş yapmamış birinin geri çevrilmesi, kaydın kötüye kullanılmasını önlemek için daha az ayrıntıyla kaydedilir:

- **Sayfa** sütununda gerçek yol yerine **Herhangi bir sayfa** yazar. Böylece sitenizi tarayan biri kaydı binlerce farklı yolla dolduramaz.
- **Adres** sütununda tam adres yerine ağın ilk kısmı yazar: IPv4'te `/24` (örneğin `203.0.113.0/24`), IPv6'da `/48`.
- İstisna: ret bir **Tamamen engelle** kuralından, bir açık yoldan (imza kontrolü dahil) ya da bir yöntem kuralından geliyorsa, **Sayfa** sütununda o kuralın yolu (örneğin `/internal`) görünür. Bu yolları zaten siz seçtiğiniz için hangi kuralın çalıştığını görebilirsiniz.
- Ülke listesinin ya da hız sınırının retleri, ziyaretçi giriş yapmış olsa bile her zaman bu şekilde, **Giriş yapmamış** olarak kaydedilir: bu kontroller Komuta ziyaretçinin kim olduğuna bakmadan önce yapılır.

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
| **Paylaşım bağlantısıyla girdi** | Bir paylaşım bağlantısı açıldı ve oturum başladı. |
| **Sayfayı paylaşım bağlantısıyla açtı** | Bağlantıyla giren ziyaretçi sayfayı açtı. |
| **Bilinmeyen ya da süresi dolmuş bir paylaşım bağlantısı açtı** | Bağlantı yanlıştı, süresi dolmuştu ya da `GET`/`HEAD` dışında bir yöntemle açıldı. |
| **Paylaşım bağlantısı bu sayfayı açmıyor** | Sayfa bağlantının sayfalarının dışında ya da yalnızca seçilen kişilere açık. |
| **Paylaşım bağlantısı sona erdikten sonra geri geldi** | Bağlantıyla giren ziyaretçi, bağlantı silindikten ya da süresi dolduktan ya da herkes çıkarıldıktan sonra geri geldi. |
| **Bu yolun izin vermediği bir yöntem kullandı** | Bir yöntem kuralı reddetti (`405`). |
| **İzin verilmeyen bir ülkeden geldi** | Ülke listesi reddetti. |
| **Çok fazla istek gönderdi** | Hız sınırı reddetti (`429`). |
| **Geçerli imzası olmayan bir webhook gönderdi** | İmzalı bir webhook yolu, imzası olmayan ya da yanlış olan veya yolun kabul etmediği bir yöntemle gelen isteği reddetti (`401`). |
| **Henüz imza sırrı olmayan bir yola webhook gönderdi** | İmzalı yolun imza sırrı yok (`401`). |
| **64 KiB'tan büyük bir webhook gövdesi gönderdi** | Gövde doğrulanamayacak kadar büyüktü (`413`). |
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

- **Zaman aralığı** — **Son 24 saat**, **Son 7 gün** ya da **Son {gün} gün** (varsayılan; organizasyonun saklama süresi, örneğin **Son 30 gün**).
- **Göster** — **Tümü**, **Girişler**, **Sayfa görüntülemeleri**, **Retler**.
- Her sayfada 50 kayıt gösterilir. **Daha yeni** ve **Daha eski** düğmeleriyle gezilir; altta "{toplam} kayıttan {ilk}–{son}" yazar. Toplam 10.000'i aşarsa sayının yanında `+` görünür.

Liste kendiliğinden yenilenmez; yeni kayıtları görmek için filtreyi değiştirin ya da sayfayı yenileyin.

---

## Dışa aktarma

Listenin üstündeki **Dışa aktar** kaydı indirir: **CSV olarak indir (tablo)** ya da **JSON olarak indir**.

- Dışa aktarım, geçerli **Göster** filtresine ve **Zaman aralığı**'na göre, şu ana kadarki kayıtları alır. En uzun zaman aralığıyla saklama süresinin tamamını kapsar.
- En fazla **50.000** satır. Aralıkta daha fazlası varsa dışa aktarım reddedilir: "Bu aralıkta 50000 kayıttan fazlası var. Daha kısa bir aralık ya da tek bir kayıt türü seçin."
- Organizasyon başına aynı anda tek dışa aktarım. Organizasyonunuzun başka bir dışa aktarımı sürüyorsa: "Kuruluşunuzun erişim kaydının başka bir dışa aktarımı hâlâ sürüyor. Birazdan yeniden deneyin."
- Her dışa aktarım organizasyonunuzun denetim kaydına yazılır. Yazılamazsa dışa aktarım yapılmaz: "Dışa aktarım denetim kaydına yazılamadığı için yapılmadı. Birazdan yeniden deneyin."
- Dışa aktarmak için kaydı görmekle aynı izin gerekir. E-posta adresleri yalnızca kullanıcıları görme izniniz varsa dosyaya eklenir.

CSV dosyası bir UTF-8 bayt sıra işaretiyle başlar (böylece tablolama programları Türkçe karakterleri doğru okur) ve şu sütunları içerir:

| Sütun | İçerik |
|---|---|
| `first_seen_utc`, `last_seen_utc` | Satırın ilk ve son görüldüğü zaman (ISO 8601, UTC). |
| `count` | Kaç kez yaşandığı (**Adet**). |
| `outcome` | `SignIn`, `Allow` ya da `Deny`. |
| `reason` | Teknik neden kodu (bkz. [Başvuru](access-protection-reference.md#erişim-kaydı-neden-kodları)). |
| `who` | Kişinin, token'ın ya da bağlantının adı (silinmişse kimliği); e-posta ziyaretçisinde size gösteriliyorsa e-postası, değilse kimliği (tireli GUID); giriş yapmamış ziyaretçilerde boş. |
| `email` | Size gösteriliyorsa ziyaretçinin e-postası. |
| `kind` | `user`, `external_user`, `email_visitor`, `service_token`, `share_link` ya da `anonymous`. |
| `method`, `path` | HTTP yöntemi ve yol (yol gizliyse `*`). |
| `client_ip` | Adres; anonim retlerde `/24` / `/48` ağı. |

`=`, `+`, `-`, `@`, sekme ya da satır başı karakteriyle başlayan değerlerin önüne `'` eklenir; böylece tablolama programı bunları hiçbir zaman formül olarak çalıştırmaz. JSON dosyası aynı kayıtları, API'nin alan adlarıyla bir dizi olarak içerir.

---

## Saklama süresi

Kayıtlar varsayılan olarak **30 gün** saklanır; bir satır, son görüldüğü zamandan itibaren saklama süresini aşınca silinir. Bir organizasyon kayıtları **30**, **90** ya da **365** gün saklayabilir:

- Ayar: **Hesap → Organizasyonlar → Erişim kaydı saklama süresi**. Değiştirmek için organizasyonu düzenleme yetkisi gerekir.
- Eski kayıtlar her gece silinir. Süreyi uzatmak hemen geçerli olur. Kısaltmak **Erişim kayıtları daha kısa süre saklansın mı?** onayını ister ("Son {days} dışında kalan kayıtlar bir sonraki gece çalışmasında silinir ve geri getirilemez. Gerekiyorsa önce dışa aktarın.") ve **Kısalt ve eski kayıtları sil** ile onaylanır.
- **Etkinlik** sekmesi kayıtların ne kadar saklandığını söyler; en uzun zaman aralığı bu ayara göre değişir.

Bu ayarı görmüyorsanız erişim koruması platformunuzda henüz açık değildir ya da organizasyonu düzenleyemiyorsunuz.

---

## İlgili Dokümanlar

- [Giriş ve Paylaşım](access-protection-sign-in-sharing.md) — ziyaretçilerin gördüğü sayfalar.
- [Kurallar](access-protection-rules.md) — retlerin nedeni olan kurallar.
- [Makineler ve Özel Ağ](access-protection-machines.md) — token ve webhook kayıtları.
- [Başvuru](access-protection-reference.md) — teknik neden kodları.
