# Kalıcı Depolama

Servisinizin container'ındaki disk geçicidir. Uygulamanın çalışırken yazdığı dosyalar o pod'a aittir: pod yeniden başlatıldığında veya yeni bir sürüm dağıtıldığında kaybolur, yeni açılan bir replika da onları görmez. Yeniden başlatmalardan ve dağıtımlardan sonra da kalması gereken dosyalar için servise kalıcı bir **hacim** (volume) bağlayın. Hacim, container içinde seçtiğiniz bir klasöre bağlanır. Uygulamanız o klasöre yazdığı dosyaları sonraki her pod'da yeniden bulur.

Hacimler servis sayfasındaki **Depolama** sekmesinden yönetilir: **Uygulamalar** → servis → **Depolama**.

---

## Ne Zaman Kalıcı Hacim Gerekir

| Durum | Önerilen |
|---|---|
| Kullanıcıların yüklediği dosyalar, uygulamanın ürettiği raporlar, dosya tabanlı indeks veya önbellek | Kalıcı hacim |
| İlişkisel veritabanı | [PostgreSQL](managed-postgresql.md). Yedek alma ve geri yükleme hazır gelir. |
| Oturum verisi, anahtar-değer önbelleği | [Valkey](managed-valkey.md) |
| Kuyruk ve mesajlaşma | [RabbitMQ](managed-rabbitmq.md) |
| Yalnızca istek süresince gereken geçici dosyalar | Hacim gerekmez, container'ın kendi diski yeterlidir |

[Servis Güvenliği](service-security-guide.md) sayfasında anlatılan yazılabilir yollar da kalıcı değildir. O yollara yazılan dosyalar, pod yeniden oluşturulduğunda silinir.

Bilmeniz gereken sınırlar:

- Her hacim tek bir servise aittir. Başka bir servisle paylaşılamaz.
- Hacimler yalnızca sürekli çalışan servislere eklenebilir. Job ve CronJob'larda **Depolama** sekmesi bulunmaz.
- Depolama sayfasında yedek alma veya geri yükleme işlemi yoktur. Kaybetmek istemediğiniz dosyaların kopyasını kendiniz tutun.

---

## Depolama Sekmesi

Serviste hiç hacim yoksa sekmede **Bağlı Hacim Yok** yazar. Hacim varsa **Depolama Hacimleri** listesinde her hacim için şunlar görünür:

| Bilgi | Açıklama |
|---|---|
| Hacim adı | Hacmi oluştururken verdiğiniz ad |
| **Devre dışı** | Hacim pasif, servise bağlanmıyor (aşağıdaki Hacmi Düzenleme bölümüne bakın) |
| **Salt okunur** | Uygulama bu hacme yazamaz |
| **Ücretsiz** | Hacim ücretsiz plandadır |
| Plan ve boyut | Hacmin planı, örneğin `Small` |
| Bağlantı yolu | Hacmin container içinde bağlandığı klasör, alt yol varsa onunla birlikte |
| Aylık fiyat | Ücretli planlarda planın aylık fiyatı |

**Hacim Ekle** düğmesi, düzenleme ve silme simgeleri yalnızca servisi düzenleme izni olan kullanıcılara görünür. İzniniz yoksa listeyi görürsünüz ama değiştiremezsiniz.

---

## Hacim Ekleme

1. **Depolama** sekmesinde **Hacim Ekle**'ye tıklayın. Pencerenin üstünde üç adım görünür: **Plan**, **Hacim**, **Yol**.
2. **Depolama Planı** tablosundan bir plan seçin. Tabloda **Plan**, **Boyut**, **Replika** ve **Fiyat** sütunları bulunur. **Replika** sütunu, verinizin kaç kopya hâlinde saklandığını gösterir. Servisinizin replika sayısıyla ilgisi yoktur.
3. **Hacim Adı** alanını kontrol edin. Ad, servis adından otomatik önerilir; örneğin `orders-api` servisi için `orders-api-vol-1`. İsterseniz değiştirin.
4. **Bağlantı Yolu** alanında hacmin bağlanacağı klasörü seçin (ayrıntılar aşağıda).
5. Gerekirse **Alt Yol (isteğe bağlı)** girin ve **Salt okunur** anahtarını açın.
6. **Oluştur**'a tıklayın.

Kayıt başarılı olunca **Hacim eklendi** bildirimi görünür. Eksik veya hatalı bir alan varsa **Oluştur** düğmesi pasif kalır ve pencerenin altında ne yapmanız gerektiği yazar; örneğin **Bir depolama planı seçin** veya **Hacim adı biçimini düzeltin**.

### Planlar

| Plan | Boyut |
|---|---|
| Free | 1 GB |
| Small | 10 GB |
| Medium | 50 GB |
| Large | 200 GB |
| XLarge | 1024 GB |

Plan tablosunda ücretsiz plan **ÜCRETSİZ** etiketiyle işaretlidir. Diğer planların aylık fiyatı **Fiyat** sütununda görünür.

Bir servisteki hacimlerin hepsi ya ücretsiz ya ücretli olmalıdır:

- Ücretsiz planla bir servise yalnızca bir hacim eklenebilir. Bu hacim, servisin tek hacmi olmalıdır.
- Serviste ücretli bir hacim varsa ücretsiz hacim eklenemez.
- Serviste ücretsiz bir hacim varsa yeni bir ücretli hacim eklemeden önce o hacmi ücretli bir plana yükseltin (aşağıdaki Hacmi Düzenleme bölümüne bakın).

Bu kurallara uymayan planlar tabloda soluk görünür ve seçilemez. Üzerine geldiğinizde nedeni görünür.

### Bağlantı yolu

Servisin çalışan bir pod'u varsa Komuta, uygulamanın çalışma dizininin (WORKDIR) hemen altındaki klasörleri listeler. Listedeki ilk klasör otomatik seçilir. **Oluştur**'a basmadan önce doğru klasörün seçili olduğunu kontrol edin.

Aradığınız klasör listede yoksa **Manuel gir** bağlantısıyla yolu kendiniz yazın; örneğin `/app/data`. Listeye dönmek için **Dizinlere göz at**'a tıklayın. Yolun imajda bulunması gerekmez. Uygulamanın çalışırken oluşturacağı bir klasörü de yazabilirsiniz.

Servis henüz dağıtılmadıysa, uyuyorsa ya da imajda kabuk (`sh` veya `bash`) yoksa klasör listesi açılmaz. Bu durumda **Dizine göz atma kullanılamıyor — yolu manuel girin** yazar ve yolu elle girersiniz.

### Alt yol ve salt okunur

- **Alt Yol (isteğe bağlı)**: Hacmin kökü yerine içindeki bir klasörü bağlantı yoluna bağlar. Hacmin içindeki göreli bir yol yazın, örneğin `data/cache`.
- **Salt okunur**: Uygulama hacmi okuyabilir ama yazamaz.

### Ad ve yol kuralları

| Alan | Kural | Konsoldaki uyarı |
|---|---|---|
| **Hacim Adı** | En fazla 63 karakter. Yalnızca küçük harf, rakam ve tire içerir; harf veya rakamla başlar ve biter. Yazdığınız büyük harfler otomatik küçültülür. `fwk-` ile başlayamaz. Aynı serviste benzersiz olmalıdır. | Yalnızca küçük harf, rakam ve tire (DNS-1123) |
| **Bağlantı Yolu** | `/` ile başlayan mutlak bir yol, en fazla 256 karakter. `..` içeremez ve kök dizin (`/`) olamaz. Aynı serviste benzersiz olmalıdır. | Geçerli bir mutlak konteyner yolu girin ('/' ile başlamalı, '..' segmenti içermemeli). |
| **Alt Yol (isteğe bağlı)** | Hacmin içinde göreli bir yol, en fazla 256 karakter | — |

Bazı yollara hacim bağlanamaz, çünkü hacim container'ın o yoldaki kendi dosyalarını gizler ve uygulamayı bozar:

- `/app` klasörünün kendisi. Alt klasörleri, örneğin `/app/uploads` veya `/app/data`, kullanılabilir.
- `/app/wwwroot` ve altındaki her şey.
- Sistem klasörleri ve altındaki her şey: `/bin`, `/sbin`, `/usr`, `/lib`, `/lib64`, `/etc`, `/boot`, `/root`, `/proc`, `/sys`, `/dev`, `/var/run`.
- Platformun kendisi için ayırdığı birkaç yol.

Böyle bir yol seçerseniz alanın altında, örneğin `/etc` için şu uyarı görünür: **'/etc' rezerve bir sistem yoludur. Buraya bir birim bağlamak konteynerin yerleşik dosyalarını gölgeler ve uygulamayı bozar. /data veya /app/uploads gibi ayrı bir veri dizini seçin.** Verileriniz için `/data` veya `/app/uploads` gibi ayrı bir klasör kullanın.

---

## Değişiklik Nasıl Uygulanır

Hacim eklediğinizde, düzenlediğinizde veya kaldırdığınızda Komuta servisi otomatik olarak yeniden dağıtır. Yeni build alınmaz; servis şu an çalışan imajla yeniden dağıtılır. Bildirimde **Depolama güncellendi — dağıtım ~30 sn içinde başlayacak** yazar.

Dağıtım, servisin dağıtım stratejisine göre ilerler. Blue-Green stratejisinde yeni sürüm, siz canlıya alana kadar trafiği almaz. Bu yüzden hacim de ancak o zaman canlı trafiğe hizmet eden pod'larda görünür. Ayrıntılar için [Dağıtım Stratejileri](deployment-strategies.md) sayfasına bakın.

Otomatik dağıtım başlatılamazsa bildirimde **Depolama güncellendi — otomatik dağıtım atlandı:** ve ardından nedeni yazar. Değişiklik kaydedilmiştir. Uygulamak için [Servis Dashboard](service-dashboard.md) sayfasındaki **Dağıt** düğmesiyle bir dağıtım başlatın.

Servis henüz hiç dağıtılmadıysa hacim, servisin ilk dağıtımında bağlanır.

> **Not:** Hesabınız askıya alındıysa veya ücretsiz planın bu ayki uyanık saatleri bittiyse, servisteki diğer değişiklikler gibi hacim değişiklikleri de engellenir. Hata mesajı nedeni ve çözümü söyler. Bkz. [Uyku ve uyanma](service-sleep.md).

---

## İmajdaki Dosyalar ve İlk Bağlama

Bağlantı yolu, imajınızda dosya içeren bir klasör olabilir. Bu durumda hacim boşken bu dosyalar, ilk dağıtımda hacme bir kez kopyalanır. Böylece uygulama, imajda beklediği dosyaları hacimde de bulur.

Hacimde dosya olduğu sürece bu kopyalama bir daha yapılmaz. Sonraki dağıtımlarda imajın o klasördeki dosyaları değişse bile hacimdeki dosyaların üzerine yazılmaz. Hacimdeki içerik her zaman önceliklidir.

- Salt okunur hacimlere kopyalama yapılmaz. İmajda dosya bulunan bir klasöre boş, salt okunur bir hacim bağlarsanız uygulama o klasörü boş görür.
- Alt yol kullanıyorsanız kopyalama, hacmin içindeki o alt klasöre yapılır.
- Bağlantı yolundaki klasörde `.komuta-seed.lock` adlı gizli bir dosya görebilirsiniz. Komuta bu dosyayı, birden çok replikanın ilk kopyalamayı aynı anda yapmasını önlemek için kullanır. Bu dosyayı silmeyin. Yeni bir hacimde `lost+found` klasörü de bulunabilir.

---

## Yeniden Başlatma, Dağıtım, Ölçekleme ve Uyku

| Durum | Hacimdeki veriler |
|---|---|
| **Yeniden Başlat** ile pod'ların yeniden oluşturulması | Korunur. Yeni pod'lar aynı hacmi bağlar. |
| Yeni sürüm dağıtma veya önceki sürüme geri alma | Korunur. İmajdaki dosyalar hacmin üzerine yazılmaz. |
| Replika sayısını artırma veya [otomatik ölçekleme](services-autoscaling.md) | Tüm replikalar aynı hacmi aynı anda okur ve yazar. |
| Blue-Green, Auto Promote veya Canary geçişi | Geçiş boyunca eski ve yeni sürüm aynı hacmi birlikte kullanır. |
| Servisin uykuya geçmesi | Korunur. Servis uyandığında aynı hacim yeniden bağlanır. |
| Daha büyük bir plana geçme | Korunur. |
| Hacmi kaldırma | Kalıcı olarak silinir. |
| Servisi silme | Servisin tüm hacimleri verileriyle birlikte silinir. |

Bir hacmi servisin bütün replikaları paylaşır. Tek yazıcı sınırı yoktur. Bu nedenle aynı dosyaya aynı anda yazan replikaları uygulamanızın düzenlemesi gerekir. SQLite gibi dosya tabanlı bir veritabanını birden çok replikayla bir hacim üzerinde çalıştırmayın; veritabanı için [PostgreSQL](managed-postgresql.md) kullanın.

Dağıtım sırasında eski ve yeni sürüm aynı dosyaları görür. Yeni sürüm hacimdeki dosyaların biçimini değiştiriyorsa eski sürümün de bu dosyaları bir süre okuyacağını hesaba katın.

---

## Hacmi Düzenleme

Listede hacmin satırındaki kalem simgesine tıklayın. **Hacmi Düzenle** penceresi açılır.

| Alan | Değiştirilebilir mi? |
|---|---|
| **Depolama Planı** | Evet. Aynı veya daha büyük boyutta bir plan seçin. |
| **Alt Yol (isteğe bağlı)** | Evet |
| **Salt okunur** | Evet |
| **Aktif** | Evet |
| **Hacim** (ad) | Hayır |
| **Bağlantı Yolu** | Hayır |

Değişikliği **Güncelle** ile kaydedin. **Hacim güncellendi** bildirimi görünür ve servis yeniden dağıtılır.

### Hacmi büyütme

Daha büyük bir plan seçip **Güncelle**'ye tıklayın. Hacimdeki veriler korunur. Ücretsiz hacmi de aynı yolla ücretli bir plana yükseltebilirsiniz.

Hacimler küçültülemez. Daha küçük bir plan tabloda seçilebilir görünse de kaydettiğinizde **Hacim işlemi başarısız** bildirimi ve depolamanın küçültülemeyeceğini söyleyen bir mesaj görürsünüz. Daha küçük bir hacme geçmek için farklı bir bağlantı yoluna yeni bir hacim ekleyin, dosyaları kendiniz taşıyın, sonra eski hacmi kaldırın.

Hacmin adı ve bağlantı yolu sonradan değiştirilemez. Başka bir yol gerekiyorsa dosyaları kopyalayın, yeni bir hacim ekleyin ve eskisini kaldırın.

### Alt yolu değiştirme

Alt yolu değiştirdiğinizde uygulama, hacmin içindeki başka bir klasörü görür. Önceki alt klasördeki dosyalar hacimde kalır ama artık bağlantı yolunda görünmez.

### Aktif anahtarı

**Aktif** anahtarını kapatırsanız hacim listede **Devre dışı** olarak görünür. Servise bağlanmaz ve servisin bir sonraki dağıtımına dahil edilmez. Pasif bir hacim, adını ve bağlantı yolunu tutmaya devam eder. Aynı ad veya yolla yeni bir hacim ekleyemezsiniz.

> **Uyarı:** **Aktif** anahtarını verileri saklayarak hacmi geçici olarak devre dışı bırakmak için kullanmayın. Kapatmanın, hacmi kaldırmakla aynı veri kaybı riskini taşıdığını varsayın. Önce önemli dosyaların kopyasını alın.

---

## Hacmi Kaldırma

1. Listede hacmin satırındaki çöp kutusu simgesine tıklayın.
2. Tarayıcının onay penceresinde **{ad} kaldırılsın mı? Bu hacimdeki veriler silinecek.** sorusunu onaylayın.

**Hacim kaldırıldı** bildirimi görünür ve servis yeniden dağıtılır. Bu dağıtımdan sonra uygulama o yolda hacmi görmez.

> **Uyarı:** Kaldırma kalıcıdır. Hacimdeki bütün dosyalar, hacmin anlık görüntüleri ve yedekleriyle birlikte silinir. Komuta kaldırılan bir hacmi geri getiremez. Kaldırmadan önce ihtiyacınız olan dosyaların kopyasını alın.

Servisi sildiğinizde de servisin bütün hacimleri aynı şekilde silinir.

---

## Doluluğu İzleme

[Servis Dashboard](service-dashboard.md) sayfasındaki topolojide her aktif hacim bir **Storage** düğümü olarak görünür. Kullanım ölçülebildiğinde düğümün üstünde kullanılan ve toplam alan yazar, örneğin `1.2 GiB / 10 GiB`. Yazının altındaki çubuk doluluğu gösterir: doluluk %75 ve üzerinde turuncu, %90 ve üzerinde kırmızı olur. Kullanım ölçülemediğinde yalnızca hacmin boyutu görünür.

Hacim dolmadan önce daha büyük bir plana geçin.

---

## Hatalar ve Çözümleri

Sunucunun reddettiği işlemlerde **Hacim işlemi başarısız:** bildirimi ve ardından aşağıdaki mesajlardan biri görünür.

| Mesaj | Neden | Ne yapmalı |
|---|---|---|
| Ücretsiz plan yalnızca 1 depolama birimine izin verir. Ek depolama için lütfen ücretli bir plan seçin. | Serviste zaten ücretsiz bir hacim var | Ücretli bir plan seçin |
| Bu serviste ücretsiz plan ile oluşturulmuş bir depolama bulunuyor. Ücretli depolama eklemeden önce mevcut ücretsiz depolamayı ücretli plana yükseltin. | Ücretsiz hacmi olan bir servise ücretli hacim ekleniyor | Önce ücretsiz hacmi düzenleyip ücretli bir plana geçirin |
| Bu serviste ücretli plan ile oluşturulmuş depolama bulunuyor. Aynı servise ücretsiz plan depolama eklenemez. | Ücretli hacmi olan bir servise ücretsiz hacim ekleniyor | Ücretli bir plan seçin |
| Bu servis için aynı volume adı veya aynı mount path ile başka bir storage attachment zaten mevcut. | Ad veya bağlantı yolu bu serviste, pasif hacimler dahil, zaten kullanılıyor | Farklı bir ad veya yol girin |
| Volume adı 'fwk-' ön ekiyle başlayamaz — bu ön ek platformun kalıcı framework-keys volume'una ayrılmıştır. | Ad `fwk-` ile başlıyor | Başka bir ad seçin |
| Kalıcı disk yalnızca sürekli çalışan bir servise eklenebilir. Tek seferlik veya zamanlanmış iş kalıcı disk bağlamaz. | Bir Job veya CronJob'a hacim eklenmeye çalışılıyor (örneğin MCP ile) | Kalıcı veri için yönetilen bir servis kullanın |
| Depolamanın küçültülemeyeceğini söyleyen mesaj | Daha küçük bir plan seçildi | Aynı veya daha büyük bir plan seçin |

---

## MCP ve Stack ile Hacim Yönetimi

[Komuta MCP](mcp-tools.md) ile yapay zeka asistanınıza hacimleri ve planları listeletebilir, hacim ekletebilir, düzenletebilir ve kaldırtabilirsiniz. Konsoldaki kurallar MCP'de de aynen geçerlidir.

Bir [Stack](stacks.md) ile oluşturduğunuz servislerin hacimlerini manifest dosyasında tanımlayabilirsiniz. Alanlar için [Stack Manifest Referansı](stack-manifest-reference.md) sayfasına bakın.

---

## İlgili Dokümanlar

- [Servis Dashboard](service-dashboard.md) — Topolojide hacim doluluğu ve elle dağıtım.
- [Otomatik Ölçekleme](services-autoscaling.md) — Replika sayısının yükle değişmesi.
- [Dağıtım Stratejileri](deployment-strategies.md) — Hacim değişikliğinden sonraki dağıtımın nasıl yayına alındığı.
- [Uyku ve uyanma](service-sleep.md) — Uyuyan servislerde neler korunur.
- [PostgreSQL](managed-postgresql.md) — Veritabanları için yedekli, yönetilen seçenek.
- [Servis Güvenliği](service-security-guide.md) — Yazılabilir yollar ve salt okunur kök dosya sistemi.
- [Komuta MCP](mcp-tools.md) — Hacimleri asistanınızla yönetme.
