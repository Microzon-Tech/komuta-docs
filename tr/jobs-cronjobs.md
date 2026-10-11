# Job ve CronJob'lar

Job, bir kez çalışıp biten bir iştir: veri taşıma, tek seferlik bir düzeltme ya da elle tetiklenen bir rapor gibi. CronJob aynı işi bir takvime bağlar; gece temizliği veya saatlik senkronizasyon gibi düzenli işler için kullanılır. İkisi de [**Job ve CronJob'lar**](https://console.komuta.io/jobs) sayfasından oluşturulur ve yönetilir.

Job ve CronJob'lar sürekli çalışan servislerden farklıdır: istek karşılamazlar, bu yüzden genel adresleri yoktur. Port, alan adı, otomatik ölçekleme, sağlık kontrolü ve kalıcı disk bu iş yüklerinde kullanılmaz.

---

## Job mı, CronJob mı?

| | Job | CronJob |
|---|---|---|
| Ne zaman çalışır | Yalnızca siz **Şimdi çalıştır** dediğinizde | Belirlediğiniz takvimde kendiliğinden; isterseniz **Şimdi çalıştır** ile ayrıca |
| Sihirbazdaki seçim | **Elle çalıştıracağım** | **Takvime bağla** |
| Menüdeki kısayol | **Yeni Job** | **Yeni CronJob** |
| Zamanlamayı askıya alma | Yok | **Zamanlamayı askıya al** |
| Örnek kullanım | Veri taşıma, tek seferlik düzeltme, elle istenen rapor | Gece temizliği, periyodik senkronizasyon, haftalık rapor |

Tür, iş yayınlanmadan önce seçilir. Yayınlanmış bir Job sonradan CronJob'a (ya da tersi) çevrilemez; farklı bir tür için yeni bir iş yayınlayın.

> **Not:** Yayınlamak bir çalışma başlatmaz. Job, siz **Şimdi çalıştır** diyene kadar bekler; CronJob ilk zamanlama diliminde çalışır.

---

## Sihirbazı Açma

1. Sol menüden **Job ve CronJob'lar** sayfasını açın.
2. Tek seferlik iş için **Yeni Job**, zamanlanmış iş için **Yeni CronJob** seçin. Sihirbaz bu seçime göre açılır; zamanlama tercihini **İş yapılandırması** adımında yine değiştirebilirsiniz.
3. Adımları sırayla tamamlayın. Bir adımdaki zorunlu alanlar tamamlanmadan sonraki adıma geçilmez.

Sihirbaz yalnızca servis oluşturma yetkisi olan kullanıcılara açılır.

Sihirbazın adımları, işin neyi çalıştıracağına göre değişir:

| Kaynak | Adımlar |
|---|---|
| **Depondaki bir proje** | Hedef → Kaynak → Bağlantı → Depo → İş yapılandırması → Özet |
| **Bir betik yaz** | Hedef → Kaynak → İş yapılandırması → Özet |
| **Hazır imajın** | Hedef → Kaynak → Kayıt defteri → İş yapılandırması → Özet |

---

## 1. Hedef

**Proje** listesinden işin ekleneceği projeyi seçin ya da **Yeni proje oluştur…** ile yeni bir proje açın. Ardından **Bölge** seçin. **Dağıtım hedefi** her zaman **Komuta PaaS**'tır.

## 2. Kaynak

| Seçenek | Ne yapar | Etiket |
|---|---|---|
| **Depondaki bir proje** | Repodaki kodunuz bir Dockerfile ile imaja çevrilir. Bağımlılıkları olan işler için uygundur. | **build gerekir** |
| **Bir betik yaz** | Birkaç satırlık işi doğrudan sihirbaza yazarsınız; Bash, Python 3.12 veya Node 22 üzerinde çalışır. Repo gerekmez. | **build yok** |
| **Hazır imajın** | Kayıt defterinizdeki bir container imajını olduğu gibi çalıştırır. | **build yok** |

## 3. Bağlantı ve Depo (repo kaynağı)

**Bağlantı** adımında kodun okunacağı **Git hesabı**nı ve build edilen imajın gönderileceği **Konteyner kayıt defteri**ni seçin. Komuta'nın kendi kayıt defteri hazır gelir (**Platform tarafından yönetiliyor**). Git hesabınızı ve kendi kayıt defterinizi [Entegrasyonlar](integrations.md) sayfasından bağlarsınız.

**Depo** adımında repoyu ve **Dal**ı seçin.

## 3. Kayıt Defteri (hazır imaj kaynağı)

Sihirbaz "Komuta bu imajı çekmek için hangi bağlantıyı kullansın?" diye sorar. İmajın bulunduğu kayıt defterinin bağlantısını seçin. Özel bir kayıt defteri kullanıyorsanız bağlantıyı önce [Entegrasyonlar](integrations.md) sayfasından ekleyin.

## 4. İş Yapılandırması

### Ne çalışacak

| Alan | Kaynak | Açıklama |
|---|---|---|
| **İş adı** | Hepsi | Zorunlu. Küçük harf, rakam ve kısa çizgi kullanın; ad harf ya da rakamla başlayıp bitmelidir. Örnek: `gece-temizligi`. |
| **Kaynak Paketi** | Hepsi | Zorunlu. Her çalışmaya ayrılan kapasiteyi belirler. Her paketin fiyatı listede yanında gösterilir. |
| **Yapılandırma dosyası** | Repo | Zorunlu. Dockerfile'ın repo kökünden başlayan yolu; varsayılan `Dockerfile`. |
| **İmaj referansı** | Hazır imaj | Zorunlu. Değişmez bir etiket veya digest kullanın; örneğin `ghcr.io/acme/temizlik:1.4`. |
| **Çalışma ortamı** | Betik | **Bash**, **Python 3.12** veya **Node 22**. Varsayılan Python 3.12. |
| **Betik** | Betik | Zorunlu. En fazla 64 KB; alanın altındaki sayaç kullanılan boyutu gösterir. |
| **Komut** | Repo, hazır imaj | İsteğe bağlı. Boş bırakılırsa imajın kendi giriş noktası kullanılır. Örnek: `python -m tasks.cleanup`. Tırnak içine aldığınız ifade tek argüman olarak geçer. |

### Ne zaman çalışacak

**Bu iş ne zaman çalışsın?** bölümünde **Elle çalıştıracağım** (Job) ya da **Takvime bağla** (CronJob) seçin.

**Takvime bağla** seçildiğinde **Sıklık** belirlenir:

| Sıklık | Girilen değer |
|---|---|
| **Saat başı** | **Saatin kaçıncı dakikası** (0–59) |
| **Her gün** | **Saat** |
| **Haftalık** | **Gün** ve **Saat** |
| **Aylık** | **Ayın günü** ve **Saat**. Gün, her ay çalışsın diye 1–28 ile sınırlıdır. |
| **Gelişmiş** | **Zamanlama ifadesi**: beş alanlı cron ifadesi (dakika, saat, ayın günü, ay, haftanın günü) |

**Saat dilimi**, zamanlamanın hangi yerel saate göre hesaplanacağını belirler. Varsayılan olarak konsolda kullandığınız saat dilimi gelir; listeden UTC, Europe/Istanbul, Europe/Berlin gibi sık kullanılan bölgeleri seçebilirsiniz.

Seçimlerin altında sonuç özetlenir: okunabilir bir açıklama (örneğin "Her gün 03:00 saatinde (Europe/Istanbul)"), **Oluşturulan zamanlama** ifadesi ve sıradaki çalışmanın seçili saat dilimindeki ve UTC'deki zamanı. **Sonraki çalışmalar** listesi gelecek üç çalışmayı gösterir. Yeni bir CronJob her gün 03:00 (`0 3 * * *`) ile başlar.

> **Not:** Zamanlama her dilimde yaklaşık bir kez çalışır; nadiren bir çalışma atlanabilir ya da tekrarlanabilir. İşinizi aynı veriyle tekrar çalıştığında sorun çıkarmayacak şekilde yazın.

### Gelişmiş ayarlar

**Gelişmiş ayarlar** bölümü kapalı gelir ve güvenli varsayılanlar seçilidir. Yalnızca farklı bir davranış gerekiyorsa açın.

| Alan | Varsayılan | Açıklama |
|---|---|---|
| **Çalışma başarısız olursa** | **Yeniden deneme — ilk hatayı bildir** | **Bir kez yeniden dene** veya **En fazla 3 kez yeniden dene** de seçilebilir. |
| **En uzun çalışma süresi** | 3000 (50 dakika) | Saniye cinsinden girilir. Çalışma bu sürenin sonunda durdurulur; boş bırakılırsa süre sınırı olmaz. |
| **Sıradaki çalışma üst üste binerse** (CronJob) | **Yeni çalışmayı atla — çoğu iş için en güvenlisi** | Diğer seçenek **Aktif çalışmayı iptal et ve yenisini başlat**; bu durumda etkin çalışma tamamlanmadan durdurulabilir. |
| **Kaçırılan çalışmayı telafi süresi** (CronJob) | 1800 (30 dakika) | Saniye cinsinden. Bir kesintiden sonra kaçırılan çalışma yalnızca bu süre içinde başlatılır; boş bırakılırsa sınır olmaz. |
| **Saklanacak başarılı çalışma** (CronJob) | 3 | Daha sonra incelemek için saklanacak tamamlanmış çalışma sayısı. |
| **Saklanacak başarısız çalışma** (CronJob) | 5 | Sorun giderme önemliyse artırın. |
| **Tamamlanan çalışmaları ne zaman silelim?** | 604800 (7 gün) | Saniye cinsinden. Boş bırakılırsa yalnızca yukarıdaki adet sınırları kullanılır. |
| **Çalışma içindeki yeniden başlatma davranışı** | **Önerilen — başarısız denemeyi bitir ve yeniden deneme kuralını kullan** | Diğer seçenek **Hata durumunda — programı aynı deneme içinde yeniden başlat**. |
| **Zamanlamayı başlatmadan yayınla** (CronJob) | Kapalı | Açarsanız CronJob askıda yayınlanır; ilk otomatik çalışmadan önce ayarları gözden geçirmek için kullanın. |

**İş güvenliği** bölümünde **Önerilen güvenli profil** seçili gelir: iş yönetici yetkisi olmadan, uygulama dosyaları değişikliğe kapalı ve gereksiz sistem yetkileri kaldırılmış olarak çalışır. Geçici dosyalar için `/tmp` altında boyutu sınırlı bir alan açılır (**Geçici dosya alanı sınırı**, varsayılan `64Mi`). İmajınız bu kısıtlarla çalışmıyorsa **Özel profil** seçip **Yönetici yetkisi olmadan çalıştır**, **Uygulama dosyalarını değişikliklere kapat** ve **Gereksiz sistem yetkilerini kaldır** anahtarlarını ayrı ayrı kapatabilirsiniz. **Bu iş özel platform erişimine ihtiyaç duyuyor mu?** anahtarını çoğu işte kapalı bırakın.

Adımın sonunda her şey tamamsa "Yapılandırma tamamlandı. İnceleme adımına geçebilirsiniz." yazar; eksik varsa "Devam etmek için {sayı} zorunlu alanı tamamlayın" başlığı altında düzeltilecek alanlar listelenir.

## 5. Özet ve Yayınlama

**Özet** adımı işin adını, kaynağını, zamanlamasını ve ilk çalışmanın zamanını, CronJob için **İlk durum**u (**Aktif** ya da **Askıda**), güvenlik profilini ve çalıştırma ayarlarını gösterir. **Yayınlanacak yapılandırma önizlemesi** açılarak uygulanacak ayarların tamamı görülebilir.

**İşi yayınla** ile iş kaydedilir, **İş yayınlandı** bildirimi çıkar ve **Job ve CronJob'lar** listesine dönersiniz. Repo kaynağında imaj önce build edilir; ilerleyişi işin **Pipeline'lar** sekmesinden izleyin. Betik ve hazır imaj kaynaklarında build adımı yoktur.

---

## Job ve CronJob'lar Listesi

Liste, seçili projedeki (proje seçilmediyse bütün projelerdeki) Job ve CronJob'ları gösterir. Üstteki sayaçlar **Son çalışma başarılı** ve **Son çalışma başarısız** olan işleri sayar.

| Sütun | Gösterdiği |
|---|---|
| **İş yükü** | İşin adı |
| **Tetikleme** | Job için **İsteğe bağlı**, CronJob için **Zamanlanmış** |
| **Zamanlama** | CronJob'un ifadesi, okunabilir açıklaması ve saat dilimi |
| **Son çalışma** | Son çalışmanın ne kadar önce olduğu; hiç çalışmadıysa **Hiç çalışmadı** |

Her işin adının yanında bir durum etiketi görünür:

| Durum | Anlamı |
|---|---|
| **Hazırlanıyor** | İş yayınlanıyor. |
| **Başarısız** | Yayınlama başarısız oldu ya da son çalışma başarısız bitti. |
| **Aktif** / **Askıda** | CronJob'un zamanlaması çalışıyor ya da askıda. |
| **Başarılı** / **İptal edildi** | Job'ın son çalışmasının sonucu. |
| **Çalıştırmaya hazır** | Job henüz hiç çalışmadı. |

Satırdaki düğmeler **Loglar** ve **Çalışmalar** sayfalarına götürür. **…** menüsünde **İş yükünü aç** ve **Sil** bulunur.

---

## Çalışmalar Sayfası

Bir işi açtığınızda şu sekmeler görünür: **Genel Bakış**, **Çalışmalar**, **Pipeline'lar**, **Loglar**, **Ortam değişkenleri** ve **Fatura**. **Genel Bakış** da **Çalışmalar** sayfasını gösterir.

Sayfanın üstünde işin türü (**Job** ya da **CronJob**) yer alır. CronJob'larda ayrıca zamanlama ifadesi, saat dilimi, çakışma kuralı, "Sonraki çalışma: … UTC" satırı ve bir durum rozeti görünür:

| Rozet | Anlamı |
|---|---|
| **Aktif** | Zamanlama çalışıyor. |
| **Askıda** | Zamanlama durdurulmuş; yeni zamanlanmış çalışma başlamaz. |
| **Yayınlanmadı** | CronJob henüz çalışacağı yere ulaşmadı. **Şimdi çalıştır** ve askıya alma düğmeleri bu sırada kapalıdır; **Pipeline'lar** sekmesini kontrol edin. |
| **Küme doğrulanıyor** | Durum okunuyor. |
| **Küme durumu bilinmiyor** | Durum şu an okunamadı. **Yeniden dene** ile tekrar deneyin. |

**Şimdi çalıştır**, **Zamanlamayı askıya al** ve **Image oluştur** düğmeleri ile ayarların düzenlenmesi yalnızca servisleri düzenleme yetkisi olan kullanıcılara açıktır. Diğer kullanıcılar çalışma geçmişini ve kayıtlı ayarları görür.

### Elle Çalıştırma

1. **Şimdi çalıştır**'a tıklayın.
2. "Job çalışması hedef kümede başlatıldı." bildirimi çıkar; kısa bir süre **Job oluşturuldu; pod'u bekleniyor** kartı görünür.
3. Çalışma başlayınca **Çalışma kümede aktif** kartı tetikleyiciyi, etkin pod sayısını ve başlangıç zamanını gösterir.
4. Çalışma bittiğinde **Çalışma geçmişi** tablosuna eklenir.

Her elle çalışma, işin en son yayınlanan imajını kullanır. CronJob'larda **Şimdi çalıştır** zamanlamadan bağımsızdır ve zamanlama askıdayken de kullanılabilir.

Etkin bir çalışma varken yeni çalışma başlatılamaz ve **Şimdi çalıştır** kapalı kalır. Tek istisna, çakışma kuralı **Aktif çalışmayı iptal et ve yenisini başlat** olan CronJob'lardır: önce etkin çalışma durdurulur ve geçmişe **İptal edildi** olarak yazılır, sonra yenisi başlar.

"Job çalışması başlatılamadı." bildirimi görürseniz en sık iki neden vardır: repo kaynaklı işin ilk build'i henüz bitmemiştir (**Pipeline'lar** sekmesine bakın) ya da başka bir çalışma hâlâ sürmektedir.

### Çalışmayı İptal Etme

1. Etkin çalışmanın kartındaki **Çalışmayı iptal et**'e tıklayın.
2. **Aktif Job çalışması iptal edilsin mi?** penceresinde **Çalışmayı iptal et** ile onaylayın.

"Job çalışması iptal edildi." bildirimi çıkar ve çalışma geçmişe **İptal edildi** olarak yazılır. Yalnızca hâlâ süren çalışmalar iptal edilebilir.

### Zamanlamayı Askıya Alma ve Devam Ettirme

CronJob'un otomatik çalışmalarını geçici olarak durdurmak için **Zamanlamayı askıya al**'a tıklayın. "CronJob askıya alındı." bildirimi çıkar ve rozet **Askıda** olur. Askıya almak o anda süren bir çalışmayı durdurmaz; onu durdurmak için **Çalışmayı iptal et**'i kullanın.

Zamanlamayı yeniden başlatmak için **Zamanlamayı devam ettir**'e tıklayın; "CronJob devam ettirildi." bildirimi çıkar. Sihirbazda **Zamanlamayı başlatmadan yayınla** açıksa CronJob askıda başlar ve aynı düğmeyle devreye alınır.

### Çalışma Geçmişi ve Loglar

**Çalışma geçmişi** tablosu son 50 çalışmayı en yeniden eskiye doğru listeler ve birkaç saniyede bir kendiliğinden yenilenir.

| Sütun | Gösterdiği |
|---|---|
| **Durum** | **Başarılı**, **Başarısız** veya **İptal edildi** |
| **Tetikleyici** | **Manuel** (Şimdi çalıştır) ya da **Zamanlanmış** |
| **Başlangıç** / **Süre** | Çalışmanın başladığı an ve ne kadar sürdüğü |
| **Çıkış kodu** | Programın bitişte döndürdüğü çıkış kodu |
| **Küme pod'u** / **Neden** | Çalışmanın pod'u ve varsa bitiş nedeni |
| **Job ve loglar** | Çalışmanın adı ve **Logları aç** bağlantısı |

Henüz çalışma yoksa tabloda **Henüz tamamlanan çalışma yok** yazar. **Logları aç**, işin **Loglar** sekmesini açar; programınızın çıktısını orada arayın ([Loglar](service-logs.md)).

### Ayarları Değiştirme

Kayıtlı ayarlar **Çalışmalar** sayfasındaki **Kaydedilmiş zamanlanmış iş ayarları** (Job'da **Kaydedilmiş tek seferlik iş ayarları**) panelindedir. **Ayarları incele** ile açın. Bölümler:

- **Zamanlama** (yalnızca CronJob): sıklık, saat ve **Saat dilimi**; altında zamanlamanın geçerli olduğu ve sonraki üç çalışma gösterilir.
- **Çalışma davranışı**: yeniden deneme sayısı, en uzun çalışma süresi, çakışma kuralı, telafi süresi, saklanacak çalışma sayıları, tamamlanan çalışmaların silinme zamanı ve **Zamanlamayı başlatmadan yayınla**. Süreleri burada saniye, dakika, saat veya gün olarak girebilirsiniz.
- **Kaynak profili**: her çalışmaya ayrılan ve en fazla kullanılabilecek CPU, bellek ve geçici depolama.
- **İş güvenliği**: sihirbazdaki güvenlik ayarlarının aynısı.

Değişiklik yapınca **Kaydedilmemiş değişiklikler** rozeti görünür. **Kaydet ve yayınla** ile kaydedin; "Yapılandırma kaydedildi; yayınlama sıraya alındı." bildirimi çıkar ve yeni ayarlar yayınlanır. Hatalı bir alan varsa düğme kapalı kalır ve düzeltilecek alanlar listelenir.

İşin türü (Job ya da CronJob) ve kaynağı bu panelden değiştirilemez.

### Ortam Değişkenleri

Sihirbazda ortam değişkeni adımı yoktur. Yayınladıktan sonra işin **Ortam değişkenleri** sekmesinden servislerdeki gibi ekleyin ve **Değişiklikleri Uygula** ile uygulayın. Ayrıntılar: [Ortam Değişkenleri](services-environment-variables.md).

### Yeni İmaj Oluşturma

Repo kaynaklı işlerde **Çalışmalar** sayfasında **Image oluştur** düğmesi bulunur. Kodunuzu değiştirdikten sonra **Job image'ını oluştur** penceresinden yeni bir imaj build edin; build bitince sonraki çalışmalar yeni imajı kullanır. Betik ve hazır imaj kaynaklı işlerde bu düğme yoktur.

---

## Silme

**Job ve CronJob'lar** listesinde işin satırındaki **…** menüsünden **Sil**'i seçin. Açılan pencerede işin adını yazıp onaylayın.

Birden fazla işi silmek için satırları işaretleyin ve **Seçilenleri sil**'e tıklayın. **İş yüklerini sil** penceresi, seçilen iş yüklerinin çalışma geçmişleriyle birlikte kalıcı olarak silineceğini söyler.

> **Uyarı:** Silme geri alınamaz. İşin ayarları ve çalışma geçmişi de silinir.

---

## Sınırlar ve Doğrulama Mesajları

| Konu | Sınır |
|---|---|
| İş adı | Küçük harf, rakam ve kısa çizgi; harf ya da rakamla başlayıp biter |
| Betik boyutu | En fazla 64 KB |
| Betik çalışma ortamları | Bash, Python 3.12, Node 22 |
| Zamanlama ifadesi | Beş alan; `*`, `,`, `-`, `/` kullanılabilir. Ay ve gün adları (`JAN`, `MON`) ile `@hourly`, `@daily`, `@weekly`, `@monthly`, `@yearly` kabul edilir. Saniye alanı ve `@every` kabul edilmez. Haftanın günü 0–6'dır (0 = Pazar); 7 kabul edilmez. |
| En kısa aralık | Ayrı bir alt sınır yoktur; `* * * * *` her dakika çalışır. |
| Çakışan çalışma | Yalnızca "yeni çalışmayı atla" veya "aktif çalışmayı iptal et" seçilebilir; çalışmaların üst üste sürmesine izin veren bir seçenek yoktur. |
| Çalışma geçmişi | Konsolda son 50 çalışma gösterilir |

Sık karşılaşılan mesajlar:

| Mesaj | Ne yapmalı |
|---|---|
| "İşe bir ad ver" / "Küçük harf, rakam ve tire kullan; örn. gece-temizligi" | **İş adı**'nı kurallara göre düzeltin. |
| "Bir kaynak paketi seçin." | **Kaynak Paketi** listesinden bir paket seçin. |
| "Betik çok büyük — 64 KB'ın altında tut" | Betiği kısaltın ya da kodu bir repoya taşıyıp **Depondaki bir proje** kaynağını kullanın. |
| "Bu geçerli bir cron ifadesi değil" | İfadeyi beş alanlı yazın, örneğin `0 3 * * *`. |
| "Bu zamanlama hiç gelmiyor" | Gün ve ay alanlarını kontrol edin; örneğin `0 0 30 2 *` hiçbir zaman çalışmaz. |
| "Europe/Istanbul veya UTC gibi geçerli bir IANA saat dilimi seçin" | Listeden bir saat dilimi seçin. |
| "Zaman aşımı sıfırdan büyük olmalı" | **En uzun çalışma süresi**'ne pozitif bir sayı girin ya da alanı boş bırakın. |
| "İmajı çekebilen bir kayıt defteri seç" | **Kayıt defteri** adımında bir bağlantı seçin. |
| "Bu isimde, farklı türde bir servis zaten var. …" | Bu adla farklı türde bir iş yükü zaten var; işe farklı bir ad verin. |

---

## Kaynak Paketi ve Fiyat

İş, sihirbazda seçtiğiniz kaynak paketinde çalışır. Paketlerin fiyatı **Kaynak Paketi** listesinde, seçim yaparken gösterilir. Faturalandırmanın ayrıntıları için [Faturalandırma](billing.md) ve [Fiyatlandırma](https://www.komuta.io/pricing) sayfalarına bakın.

---

## MCP ile Yönetim

İşlerinizi Komuta MCP üzerinden bir yapay zekâ asistanıyla da yönetebilirsiniz: çalışma geçmişini ve etkin çalışmaları görmek, kayıtlı ayarları okumak, **Şimdi çalıştır**, etkin çalışmayı iptal etmek ve CronJob'u askıya alıp devam ettirmek mümkündür. Kurulum için [Komuta MCP Sunucusunun Kurulumu](mcp-setup.md), araçların listesi için [Komuta MCP](mcp-tools.md) sayfasına bakın.

---

## İlgili Dokümanlar

- [Hızlı Başlangıç](quick-start.md) — Komuta'daki dört hizmete genel bakış.
- [Pipeline'lar](service-pipeline-guide.md) — Repo kaynaklı işlerin imajının nasıl build edildiği.
- [Loglar](service-logs.md) — Çalışmaların çıktısını inceleme.
- [Ortam Değişkenleri](services-environment-variables.md) — İşin okuduğu yapılandırma değerleri.
- [Entegrasyonlar](integrations.md) — Git hesabı ve kayıt defteri bağlantıları.
- [Stack'ler](stacks.md) — Servis, iş ve kaynakları tek dosyada tanımlama.
- [Faturalandırma](billing.md) — Harcama, cüzdan ve ekstreler.
