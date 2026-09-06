# PostgreSQL

PostgreSQL, Komuta'nın kurup işlettiği bir ilişkisel veritabanı servisidir. Sunucu kurulumu, şifreli bağlantı, sürekli işlem günlüğü arşivlemesi, günlük yedek ve izleme kuralları instance oluşturulurken hazır gelir; uygulamaya düşen tek iş verilen bağlantı adresini kullanmaktır.

---

## Nasıl Çalışır?

Bir instance oluşturulduğunda Komuta seçilen bölgede bir PostgreSQL kümesi kurar, uygulama veritabanını ve o veritabanı üzerinde tam yetkili kullanıcıyı yaratır, işlem günlüğü (WAL) arşivlemesini ve günlük yedeği açar, izleme kurallarını yerleştirir. Bu adımların hiçbiri ayrıca yapılandırma gerektirmez.

### Instance, plan ve bölge

Bir **instance**, kendi diski, kendi kullanıcısı ve kendi yedek zinciriyle çalışan bağımsız bir veritabanı sunucusudur. CPU, bellek, disk ve sunucu (replika) sayısı bağlı olduğu **plandan** gelir; sunucu parametreleri de plandan hesaplanır ve elle ayarlanamaz (bkz. [Otomatik uygulanan sınırlar](#otomatik-uygulanan-snrlar)).

**Bölge**, instance'ın fiziksel olarak çalıştığı yerdir ve oluşturulduktan sonra değiştirilemez. Başka bir bölgeye taşımanın yolu yeni bir instance oluşturup veriyi aktarmaktır.

Instance isteğe bağlı olarak bir **projeye** bağlanabilir.

### Instance durumları

| Durum | Anlamı |
|---|---|
| **Hazırlanıyor** | Kurulum sürüyor; bağlantı bilgisi henüz hazır değildir. |
| **Aktif** | Çalışıyor ve bağlantı kabul ediyor. |
| **Askıda** / **Tamamen Askıda** | Elle askıya alınmış; verisi yerindedir (bkz. [Askıya alma ve devam ettirme](#askya-alma-ve-devam-ettirme)). |
| **Yükseltiliyor** | Plan değişikliği uygulanıyor. |
| **Yükseltme Başarısız** | Plan değişikliği tamamlanamadı; instance eski planıyla **canlıdır** ve verisi yerindedir. |
| **Geri Yükleniyor** | Bir geri yükleme işlemi sürüyor. |
| **Silinmeyi Bekliyor** | Silme planlanmış; süre dolmadan iptal edilebilir. |
| **Hata** | Kurulum tamamlanamamış instance. Bu durumdaki instance'lar platform tarafından otomatik temizlenir; yedekleri saklama süresi boyunca yerinde kalır. |

![PostgreSQL listesi](https://cdn.komuta.io/docs/tr/images/postgresql/postgresql-home-page.png)

---

## Instance Oluşturma

Yeni bir instance, PostgreSQL listesindeki **Yeni PostgreSQL örneği** butonundan üç adımda oluşturulur.

### 1. Bölge ve Plan

Bölge ve plan bu adımda seçilir. Plan seçimi sadece kaynak miktarını değil, otomatik uygulanan bağlantı sınırlarını ve sorgu zaman aşımlarını da belirler — küçük planlarda bu süreler oldukça kısadır. Toplu veri yükleme ya da raporlama yapılacaksa plan [Otomatik uygulanan sınırlar](#otomatik-uygulanan-snrlar) bölümündeki değerlere göre seçilmelidir.

### 2. Ad, Sürüm ve Veritabanı

**Örnek adı** küçük harfle başlar, yalnız küçük harf, rakam ve tire içerir, en fazla 40 karakterdir.

**İlk veritabanı adı** bir PostgreSQL tanımlayıcısıdır ve farklı bir kurala uyar: küçük harfle başlar, yalnız küçük harf, rakam ve alt çizgi içerir (tire kabul edilmez), en fazla 63 karakterdir — `^[a-z][a-z0-9_]{0,62}$`.

**Sürüm**, instance'ın çalıştıracağı PostgreSQL sürümüdür; seçilen planın desteklediği sürümler listelenir. Ana sürüm sonradan değiştirilemez — başka bir sürüme geçmek yeni bir instance oluşturup veriyi aktarmayı gerektirir.

### 3. İncele ve Oluştur

Son adımda seçilenlerin özeti ve tahmini aylık maliyet görünür. **Oluştur** ile kurulum kuyruğa alınır; instance **Hazırlanıyor** durumunda listeye düşer ve kurulum tamamlanana kadar bağlantı bilgisi üretilmez.

![Yeni PostgreSQL örneği özeti](https://cdn.komuta.io/docs/tr/images/postgresql/create-postgresql-summary.png)

---

## Bağlanma

### Bağlantı adresi

Bağlantı, instance'a verilen hostname ile kurulur ve bu ad **olduğu gibi** kullanılmalıdır. Adresi IP'ye çevirip bağlanmak çalışmaz; bağlantı kurulmuş gibi görünüp hemen düşer. Adresi IP'ye sabitleyen bağlantı havuzları ve sunucu adını göndermeyen eski istemciler de aynı nedenle çalışmaz.

Bağlantı her zaman şifrelidir; sertifikayı Komuta yönetir ve ayrıca yapılandırma gerekmez. Arayüzün verdiği bağlantı dizeleri doğru şifreleme ayarını taşıdığı için olduğu gibi kullanılmalıdır — ayarı elle değiştirmek bağlantıyı kırabilir.

### Size verilen kullanıcı

Bağlantı bilgisindeki kullanıcı, o veritabanı üzerinde **tam yetkili bir uygulama kullanıcısıdır** — sunucunun yönetici (superuser) hesabı değildir. Yönetici erişimi platform tarafında kalır ve müşteriyle paylaşılmaz.

Bu kullanıcıyla yapılabilenler:

- Şema, tablo, index, view, fonksiyon ve trigger oluşturmak, değiştirmek, silmek
- Her türlü veri işlemi ve transaction
- Kendi rollerini ve yetkilendirmelerini oluşturmak
- Hazır kurulu eklentileri kullanmak

Yönetici yetkisi gerektiren işlemler desteklenmez: sunucu düzeyi parametreleri kalıcı olarak değiştirmek, listede olmayan bir eklentiyi kurmak, dosya sistemine erişen komutlar çalıştırmak. Sunucu parametreleri plana göre otomatik ayarlanır; farklı bir değere ihtiyaç varsa destek üzerinden değerlendirilir.

### Bağlantı bilgisinin görüntülenmesi

**Genel Bakış** sekmesindeki **Bağlantı bilgisini göster** ile host, port, veritabanı adı, kullanıcı adı ve parola açılır; altında hazır bağlantı dizeleri (düz URI, .NET, JDBC, Python psycopg, Go) listelenir.

Parolalar platform tarafında şifrelenmiş saklanır ve yalnız bu ekran açıldığında çözülür. **Her görüntüleme; instance, kullanıcı ve zaman bilgisiyle denetim kaydına yazılır.** Kimlik bilgisi paylaşmak yerine ekip üyelerine kendi hesaplarından erişim verilmesi önerilir.

Instance'ın özel ağ yolu varsa bağlantı dizeleri listesinin sonunda **Bağlantı Dizesi (özel ağ)** ayrıca yer alır. Bu adres genel adresle aynı şey değildir; hangisinin kullanılacağı [Ağ erişimi](#a-eriimi) bölümündeki moda bağlıdır.

![Genel Bakış sekmesi](https://cdn.komuta.io/docs/tr/images/postgresql/dashboard.png)

---

## Otomatik Uygulanan Sınırlar

### Bağlantı ve bellek sınırları

Sunucu parametreleri planın kaynaklarından hesaplanır; elle ayarlanmaz.

| Parametre | Hesaplama | Sınırlar |
|---|---|---|
| Veritabanı bağlantısı | vCPU × 100 | en az 100, en çok 1.000 |
| İstemci bağlantısı | Veritabanı bağlantısının 5 katı | 500 – 5.000 |
| `shared_buffers` | RAM × 0,25 | — |
| `effective_cache_size` | RAM × 0,75 | — |

Araya bir bağlantı havuzu (connection pool) girdiği için istemci tarafındaki bağlantı sayısı veritabanı tarafındaki gerçek bağlantı sayısından yüksek olabilir. Yine de uygulamanın havuz boyutu tablodaki istemci sınırının altında kalmalıdır.

Daha fazla eşzamanlı bağlantıya ihtiyaç duyulduğunda tek yol daha yüksek vCPU'lu bir plana geçmektir; bu sınır ayrı olarak yükseltilemez.

### Sorgu zaman aşımları

Bir sorgunun ya da açık kalmış bir transaction'ın veritabanını kilitlemesini önlemek için plana göre otomatik zaman aşımları uygulanır. Bu değerler kullanıcı ayarı değildir.

| Plan RAM | Sorgu süresi | Boşta transaction | Kilit bekleme | Geçici dosya |
|---|---|---|---|---|
| ≤ 4 GB | 30 sn | 30 sn | 10 sn | 512 MB – 1 GB |
| ≤ 8 GB | 60 sn | 60 sn | 15 sn | 2 GB |
| ≤ 16 GB | 120 sn | 120 sn | 30 sn | sınırsız |
| > 16 GB | sınırsız | 300 sn | 60 sn | sınırsız |

Süreyi aşan sorgu iptal edilir ve uygulama hata alır. Ayrıca her instance'ta 5 dakikada bir çalışan bir denetim, bu sınırların yaklaşık 5 katını aşmış sorguları ve boşta bekleyen transaction'ları sonlandırır.

> **Uyarı:** Toplu veri yükleme, büyük index oluşturma ve raporlama sorguları küçük planlarda zaman aşımına takılır. Bu tür işler parçalara bölünerek, yoğun olmayan saatlerde ya da daha yüksek bir planda çalıştırılmalıdır.

### Disk düzeni ve dolma durumu

Veri dosyaları ve işlem günlüğü aynı diski paylaşır; işlem günlüğü için diskin yaklaşık %10'u ayrılır, kalanı veriye kalır.

Disk doluluğu **%80'i geçtiğinde veritabanı salt-okunur moda geçer**: sorgular çalışmaya devam eder, yazma denemeleri hata alır. Yazma ancak yer açıldığında ya da daha büyük diskli bir plana geçildiğinde geri gelir — disk doluluk uyarısı bu yüzden beklemeye alınmamalıdır.

---

## Hazır Kurulu Eklentiler

### Kurulu gelen eklentiler

Her yeni veritabanında şu eklentiler kurulu gelir; ayrıca bir işlem gerekmez:

`pgcrypto` · `citext` · `pg_trgm` · `unaccent` · `pgaudit` · `hstore` · `uuid-ossp` · `dblink` · `postgres_fdw`

Sunucu imajı destekliyorsa vektör arama eklentileri (`vector` / `pgvector` ve `vectorscale`) de kurulur. Sorgu istatistikleri için `pg_stat_statements`, **İçgörü** sekmesi ilk açıldığında devreye girer.

### Mantıksal replikasyon ve CDC

Sunucu `wal_level = replica` ile çalışır. Bu, mantıksal çoğaltmanın ve değişiklik yakalama (CDC) araçlarının — Debezium gibi — varsayılan olarak kullanılamayacağı anlamına gelir. Böyle bir kuruluma ihtiyaç varsa destek talebi açılmalıdır; ayar instance bazında değerlendirilir.

---

## Ağ Erişimi

Erişim modu **Genel Bakış** sekmesindeki **Ağ erişimi** kartından değiştirilir. Mod değişikliği anında geçerli olur; instance yeniden başlatılmaz ve mevcut oturumlar kesilmez.

### Erişim modları

| Mod | Anlamı |
|---|---|
| **Genel** | Genel uç nokta üzerinden internetten erişilebilir, kaynak IP kısıtı yok. Varsayılan. |
| **Genel, IP izin listesi** | Genel uç nokta yalnız belirtilen IP/CIDR aralıklarına açıktır. |
| **Yalnızca özel** | Genel uç nokta tamamen kaldırılır; instance'a yalnız özel küme ağından erişilir. |

Bir mod **Henüz kullanılamıyor** olarak işaretliyse gerekçesi kartın içinde yazar. İki gerekçe vardır ve birbirinden farklıdır: IP izin listesi, kümenin gerçek kaynak adresi göremediği durumlarda kapalı kalır (aksi hâlde liste sessizce herkesi kabul ederdi); **Yalnızca özel** ise ancak uygulama kümesi özel ağa dahil edildiğinde seçilebilir — aksi hâlde instance hiçbir yerden erişilemez hâle gelirdi.

> **Uyarı:** **Yalnızca özel** moda geçmek genel bağlantı adresini siler. Kümelerin dışından bağlanan her istemci — dizüstü bilgisayar, bir GUI istemcisi, bir CI işi, Komuta üzerinde çalışmayan her şey — anında kopar ve yeniden bağlanamaz. Özel adresi kullanan servisler etkilenmez.

### IP izin listesi

İzin listesi her satıra bir CIDR ya da IP yazılarak doldurulur:

```text
203.0.113.0/24
198.51.100.7
```

- Tek başına yazılan bir adres otomatik olarak tek makineye (`/32`) genişletilir.
- Listede olmayan her kaynak ağ geçidinde reddedilir.
- Liste boş bırakılamaz — boş liste tüm erişimi keseceği için istek reddedilir. Amaç tüm dış erişimi kapatmaksa doğru seçim **Yalnızca özel** moddur.
- Geçersiz bir ifade sessizce yok sayılmaz; hata verir ve yarı uygulanmış bir kısıtlama bırakılmaz.

### Özel adres

Instance'ın özel ağ yolu varsa **Ağ erişimi** kartının altında özel adresi görünür. Bu adres bu kümedeki servislerden, özel mesh kurulduğunda diğer kümelerden erişilebilir ve **her modda çalışmaya devam eder** — genel modda bile geçerlidir. Genel adresin kaldırılması özel adresi etkilemez.

![Ağ erişimi kartı](https://cdn.komuta.io/docs/tr/images/postgresql/network-access.png)

---

## Yedekleme

### Yedek düzeni ve saklama süresi

Koruma iki parçadan oluşur ve ikisi birlikte çalışır:

- **Tam yedek** günde bir kez alınır. Yedeğin saati her instance için ayrı belirlenir; tüm instance'lar aynı anda yedeklenmez.
- **İşlem günlüğü sürekli arşivlenir.** Zaman noktasına geri dönüş (PITR) bu sayede mümkündür; koruma iki tam yedek arasındaki değişiklikleri de kapsar.

Varsayılan saklama süresi **7 gündür** ve hem tam yedekleri hem işlem günlüğü arşivini kapsar. Yani geri dönülebilecek en eski nokta da bu süreyle belirlenir.

Günde birden fazla tam yedek almanın koruma açısından bir karşılığı yoktur: her tam yedek verinin tamamının kopyasıdır ve aradaki her an zaten işlem günlüğüyle kurtarılabilir. Belirli bir işlemden önce ek bir kopya gerekiyorsa **İşlemler** sekmesindeki **Şimdi yedekle** ile elle bir yedek alınabilir.

### Backup zamanlaması

Günlük yedeğin saati ve saklama süresi **Yedekler** sekmesindeki **Backup zamanlaması** kartından değiştirilir. Saat beş alanlı bir cron ifadesiyle verilir (`0 3 * * *` → her gün UTC 03:00), saklama süresi gün olarak yazılır.

Yedek saatini değiştirmek saklama süresini etkilemez; saklama süresi yalnız açıkça değiştirildiğinde güncellenir.

> **Uyarı:** Saklama süresini kısaltmak geri alınamaz. Süre düşürüldüğünde yeni pencerenin dışında kalan yedekler bir sonraki bakımda kalıcı olarak silinir. Süreyi tekrar uzatmak silinenleri geri getirmez; koruma penceresinin yeniden dolması o kadar gün alır ve geri dönülebilecek en eski nokta aynı oranda yakınlaşır.

### Backup sağlığı

Yedeklerin sağlığı günde bir kez denetlenir ve **Yedekler** sekmesinde üç durumdan biri gösterilir. Kartta ayrıca son base backup, PITR pencere başlangıcı, WAL arşivlemesinin son doğrulandığı an ve arşiv boyutu yer alır.

| Durum | Anlamı | Ne yapmalı |
|---|---|---|
| **Sağlıklı** | Son yedek ve işlem günlüğü akışı 26 saatten daha yeni. | — |
| **Sorunlu** | Denetim çalıştı ama son yedek ya da günlük akışı beklenenden eski. | Disk doluluğunu kontrol edin; sürerse destek talebi açın. |
| **Bilinmiyor** | Denetim 48 saattir çalışamadı; yedeklerin durumu hakkında bir şey söylenemez. | Destek talebi açın. "Bilinmiyor" sağlıklı anlamına gelmez. |

Gösterge günlük denetime dayandığı için az önce alınmış bir yedek burada hemen görünmeyebilir.

### Yedeklerin imhası

Yedekler yalnızca iki durumda kalıcı olarak silinir:

- Planlı silme işleminin geri alma süresi dolduğunda,
- Bir yöneticinin açık kalıcı silme talebiyle.

Bunların dışındaki hiçbir yol yedeklere dokunmaz. Instance hata durumuna düşüp otomatik temizlenirse, kurulum yarıda kalırsa ya da bir geri yükleme iptal edilirse yedekler saklama süresi boyunca yerinde kalır.

> **Not:** Yedeklerin durması, geri yüklemenin arayüzden yapılabileceği anlamına gelmez. Otomatik temizlenmiş bir instance'ın yedeğine ulaşmak için destek talebi açılması gerekir.

![Yedekler sekmesi](https://cdn.komuta.io/docs/tr/images/postgresql/backup-page.png)

---

## Geri Yükleme

Geri yükleme **Yedekler** ya da **İşlemler** sekmesindeki **Geri yüklemeyi aç** ile başlatılır. İki seçenek vardır: son yedekten geri yükleme ve saklama süresi içindeki belirli bir ana geri dönüş (PITR).

### Geri dönülebilecek aralık

Aralığın iki ucu da sınırlıdır:

- **Geriye doğru:** en fazla saklama süresi kadar, ancak instance'ın ilk yedeğinden öncesine gidilemez. Yeni oluşturulmuş bir instance'ta bu aralık henüz birkaç saat olabilir.
- **İleriye doğru:** üst uç "şu an" değil, **en son arşivlenmiş işlem günlüğüdür.** Az yazma alan bir veritabanında son birkaç dakika henüz kurtarılabilir olmayabilir.

Geçerli aralık **Restore noktaları** kartında **En eski** ve **En yeni** olarak yazar. Aralık dışında bir an seçildiğinde işlem başlamadan hata verir; **Hedefi doğrula** ile bir zaman damgası önceden denenebilir.

### Geri yükleme yeni bir instance oluşturur

Geri yükleme mevcut instance'ın üzerine yazmaz:

1. Seçilen andaki verilerle **yeni bir instance** oluşturulur.
2. Mevcut instance çalışmaya, hizmet vermeye ve **faturalanmaya devam eder** — iki instance bir süre yan yana durur ve ikisi de ücretlendirilir.
3. Uygulamanın bağlantı ayarlarını yeni instance'a çevirmek ve eskisini silmek kullanıcıya aittir.

> **Uyarı:** Geri yüklenen kopya ayrı bir instance'tır ve kendi bağlantı adresini alır — genel ve özel adresi kaynağınkinden farklıdır. Kaynağa bağlı kalan servisler değişmeden çalışmaya devam eder; onları kopyaya taşımak için bağlantı ayarlarının elle güncellenmesi gerekir.

### Devam eden geri yüklemeyi iptal etmek

Sürmekte olan bir geri yükleme **İşlemler** sekmesindeki **Restore'u iptal et** ile durdurulabilir. Yarı kurulmuş kardeş instance kaldırılır; kaynak instance ve yedekleri etkilenmez.

---

## Plan Değişikliği

Plan, instance başlığındaki **Plan değiştir** ile değiştirilir. Hedef plan seçildiğinde etki önizlemesi kendiliğinden çalışır.

### Önizleme

Önizleme değişikliğin türünü (yükseltme, düşürme, topoloji değişikliği), tahmini kesintiyi, kaynak değişikliklerini ve varsa uyarıları verir. Önizleme uygulanamaz bir değişiklik bildirdiğinde **Planı uygula** kapalı kalır.

> **Uyarı:** Önizleme bir garanti değildir. Kapasite kontrolü onay anında yeniden çalışır; önizleme açıldıktan sonra beklenirse küme dolmuş olabilir ve aynı değişiklik reddedilebilir.

### Kısıtlar

- **Disk küçültülemez.** Daha küçük diskli bir plana geçilse bile mevcut disk boyutu korunur. Bu bir platform kısıtı değil, blok depolamanın kendi kuralıdır.
- **PostgreSQL'de daha düşük bir plana geçilemez.** Düşük planlar listede görünse de değişiklik uygulanmaz.
- **Kapasite düğüm bazında ölçülür.** Bir replikanın diski tek bir fiziksel düğümde büyümek zorundadır; kümenin toplam boş alanı yeterli görünse bile o düğümde yer yoksa değişiklik yapılamaz.

### Kesinti ve yüksek erişilebilirlik

Kaynak (CPU/RAM/disk) değişikliğinde sunucular sırayla yeniden başlatılır; beklenen kesinti yaklaşık 30 saniye – 2 dakikadır ve çok sunuculu kurulumlarda en az bir sunucu ayakta kalır. Sunucu sayısını değiştiren bir geçişte süre 2–5 dakikaya çıkar ve kısa bağlantı kesintileri olur.

Yüksek erişilebilirlikli planlarda replikalar farklı fiziksel düğümlere zorunlu olarak dağıtılır; tek bir düğümün kaybı hizmeti durdurmaz. Tek sunuculu planlarda böyle bir koruma yoktur ve bakım ile düğüm değişimlerinde kısa kesinti yaşanabilir. Her iki durumda da uygulamanın bağlantı kopmasına dayanıklı olması (yeniden bağlanma ve transaction tekrarı) gerekir.

### Yükseltme başarısız olursa

Plan değişikliği yarıda kalırsa instance eski planıyla çalışmaya devam eder ve **Yükseltme Başarısız** durumuna geçer. Bu durumdaki bir instance canlıdır, verisi yerindedir ve aynı değişiklik yeniden denenebilir. Sorun tekrarlıyorsa destek talebi açılmalıdır.

![Plan değiştir paneli](https://cdn.komuta.io/docs/tr/images/postgresql/change-plan.png)

---

## Bakım ve Yaşam Döngüsü

Yeniden başlatma, askıya alma, kimlik bilgisi yenileme ve silme **İşlemler** sekmesinde toplanır.

### Yeniden başlatma

**Yeniden başlat**, sunucuları sırayla yeniden başlatır; replikası olan planlarda kesinti oluşmaz. Yalnız çalışır durumdaki ve yükseltmesi başarısız olmuş instance'larda kullanılabilir.

### Askıya alma ve devam ettirme

İki askıya alma biçimi vardır ve geri dönüş süreleri farklıdır:

- **Soft askıya al** istemci bağlantılarını boşaltır, sunucuları çalışır tutar. Geri açma saniyeler sürer.
- **Hard askıya al** sunucuları tamamen durdurur. **Devam et** ile geri açma, küme yeniden zamanlama yaptığı için birkaç dakika sürer.

Her iki durumda da veri yerinde kalır. Askıdaki bir instance bağlantı kabul etmez; uygulama bağlantı hatası alır.

### Parola ve yedek kimlik bilgisi yenileme

**Veritabanı parolasını rotate et**, uygulama kullanıcısına yeni bir parola üretir ve **mevcut tüm bağlantıları koparır.** Uygulama yeni parolayla yeniden bağlanana kadar hata alır.

- Yalnız çalışır durumdaki bir instance üzerinde yapılabilir.
- Eski parola işlem tamamlandığı anda geçersiz olur.
- Bir sonraki yenileme tarihi 90 gün sonrası olarak işaretlenir; bu bir zorunluluk değil, hatırlatmadır.

İşlem bakım penceresinde yapılmalı ve yeni parola dağıtılmaya hazır olunmalıdır.

**Backup kimlik bilgisini rotate et** ise yedekleme hattının kullandığı depolama kimlik bilgilerini yeniler. Bu işlem bağlantıları etkilemez; yeni kimlik bilgisi bir sonraki planlı yedekte devreye girer.

### Silme

İki yol vardır ve sonuçları farklıdır:

- **Silmeyi planla** instance'ı 7 günlük bir bekleme süresinin sonunda silinmek üzere işaretler. Süre dolmadan **Planlı silmeyi iptal et** ile geri alınabilir; instance bu süre boyunca çalışmaya ve faturalanmaya devam eder.
- **Kalıcı silme** instance'ı ve **yedekler dahil tüm verisini** anında yok eder.

> **Uyarı:** Kalıcı silmenin geri dönüşü yoktur ve yedekler de silindiği için veri destek üzerinden de kurtarılamaz. Verinin bir süre daha durması isteniyorsa doğru seçim planlı silmedir.

---

## SQL Konsolu

**Sorgu** sekmesindeki SQL konsolu, bu veritabanında ayrı bir istemci kurmadan sorgu çalıştırmak içindir. Şema listesinden tablo ya da kolon adı editöre eklenebilir; editör canlı şemadan tamamlama önerir.

Konsol **varsayılan olarak salt-okunurdur.** **Yazma modu** açıldığında `INSERT`, `UPDATE`, `DELETE` ve DDL ifadeleri veritabanı sahibi olarak çalışır ve geri alınamaz — bu modda konsol, üretim verisi üzerinde çalışan bir istemciyle aynı yetkiye sahiptir.

Sonuçlar 1.000 satırla sınırlıdır; sınıra takılan sonuç **Kırpıldı** olarak işaretlenir. Tam sonuç kümesine ihtiyaç varsa sorgu `LIMIT`/`OFFSET` ile bölünmeli ya da sonuç CSV olarak indirilmelidir.

![Sorgu sekmesi](https://cdn.komuta.io/docs/tr/images/postgresql/select-query-response.png)

---

## Loglar, İçgörü ve İzleme

### Loglar

**Loglar** sekmesi, primary, replikalar ve `pgaudit` kayıtları üzerinde arama yapar. Arama deseni büyük/küçük harf duyarsız bir alt dizedir; seviye (`LOG`, `WARNING`, `ERROR`, `FATAL`, `AUDIT`), zaman penceresi ve satır limiti ayrıca seçilir. Varsayılan pencere son 1 saattir ve tek seferde en fazla 5.000 satır getirilir.

Sonuç limitte kesildiğinde bunu belirten bir not döner; bu durumda pencere daraltılmalı ya da limit yükseltilmelidir. Daha uzun süreli saklama gerekiyorsa loglar kendi tarafınıza aktarılmalıdır — bu ekran uzun dönem log arşivi değildir.

### İçgörü

**İçgörü** sekmesinde iki iş yapılır:

- **Yavaş sorgular** listesi `pg_stat_statements` sayaçlarından gelir; toplam süre, ortalama süre, çağrı ya da satır sayısına göre sıralanabilir. **İstatistikleri sıfırla** sayaçları sıfırdan başlatır — bir index eklendikten ya da bir sorgu değiştirildikten sonra etkisini temiz bir zeminde ölçmek için kullanılır. Sıfırlama geçmiş kayıtları geri getirmez.
- **Aktif sorgular** listesi eşiğin üzerinde çalışmaya devam eden ifadeleri gösterir ve bir sorgu buradan **İptal et** ile durdurulabilir. İptal edilen sorgunun istemcisi bir sorgu iptali hatası alır; sorgu bu arada tamamlanmışsa iptal isteği bir şey değiştirmez.

Sağlık snapshot'ında veritabanı boyutu, cache hit oranı, bağlantı sayısı, replikasyon gecikmesi ve vacuum borcu yer alır. Bu değerler anlık bir fotoğraftır; zaman içindeki eğilim için **İzleme** sekmesine bakılır.

### İzleme ve uyarılar

**İzleme** sekmesi bağlantı sayısı, saniye başına işlem, replikasyon gecikmesi, cache hit oranı ve disk kullanımı grafiklerini seçilen zaman aralığında verir.

İzleme kuralları instance oluşturulurken otomatik kurulur; ayrıca yapılandırma gerekmez. PostgreSQL'de kapsanan başlıklar: disk doluluğu, bağlantı doygunluğu, replikasyon gecikmesi, WAL arşivleme hataları, yedeğin eskimesi ve kilitlenme oranı. Tetiklenen uyarılar **Uyarılar** sekmesinde listelenir.

---

## Dump Dosyasından İçe Aktarma

**İşlemler** sekmesindeki **URL'den içeri al**, bir `pg_dump` çıktısını bu instance'a geri yükler. Bu bir birleştirme işlemi değildir.

> **Uyarı:** İçe aktarma onaylandığında hedefteki uygulama veritabanı düşürülür ve dump'tan yeniden oluşturulur. Mevcut tüm veriler kalıcı olarak silinir ve bu işlem geri alınamaz. Hedef veritabanı boş değilse onay için instance adının yazılması istenir.

### Kaynak

Kaynak, dump dosyasına giden bir HTTPS adresidir. Yalnız HTTPS kabul edilir ve özel ağ adresleri reddedilir. Uyumlu kaynaklar:

- Heroku Postgres yedek adresleri (`heroku pg:backups:url`)
- Ön imzalı (pre-signed) S3 / GCS / B2 adresleri
- `pg_dump` custom format (`-Fc`) çıktıları

### Uyumluluk raporu

Adres verildiğinde önce bir ön kontrol çalışır: dump indirilip incelenir ve kaynak ile hedef karşılaştırılır — sürümler, şemalar, eklentiler, tablo sayıları ve hedefte hâlihazırda duran veri. Rapor iki tür bulgu üretir:

- **Engelleyici** bulgular içe aktarmayı reddeder; onay açılmaz.
- **Uyarılar** işleme izin verir ama karar kullanıcıya bırakılır.

Dump'ta geçen ve hedefte bulunmayan bazı eklentiler geri yüklemeden çıkarılabilir. Rapor bunları işaretli listeler; işaretli bırakıldığında geri yükleme o eklentiyi atlayarak tamamlanır, işaret kaldırıldığında içe aktarma o eklenti yüzünden açıkça hata verir.

### Geri yükleme ve sonrası

Onaydan sonra geri yükleme sürer ve ilerlemesi izlenebilir; devam eden bir işlem iptal edilebilir, başarısız olan bir işlem aynı kaynakla yeniden denenebilir. Aynı instance'a yapılan önceki içe aktarmalar **İşlemler** sekmesindeki geçmiş listesinde durur ve buradan tekrar açılabilir.

İçe aktarma tamamlandıktan sonra veritabanının yeni hâli, bir sonraki planlı yedekle koruma altına girer. Aynı gün içinde bir koruma noktası isteniyorsa **Şimdi yedekle** ile elle yedek alınmalıdır.

---

## Sık Sorulan Sorular

### Superuser erişimi alınabilir mi?

Alınamaz. Verilen kullanıcı veritabanı üzerinde tam yetkilidir ama sunucunun yönetici hesabı platform tarafında kalır. Sunucu parametresi değişikliği ya da listede olmayan bir eklenti gerekiyorsa destek üzerinden değerlendirilir.

### Bağlantı sınırı ayrı olarak yükseltilebilir mi?

Yükseltilemez. Bağlantı sayısı plandaki vCPU'dan hesaplanır; artırmanın tek yolu daha yüksek vCPU'lu bir plana geçmektir.

### Geri yükleme mevcut instance'ı bozar mı?

Bozmaz. Geri yükleme kaynağa dokunmaz, yanına yeni bir instance oluşturur. Trafiği kopyaya çevirmek ve gereksiz kalan instance'ı silmek kullanıcıya aittir; iki instance yan yana durduğu sürece ikisi de faturalanır.

### Instance silinince yedekler de gider mi?

**Kalıcı silme** yedekleri de yok eder ve geri dönüşü yoktur. **Silmeyi planla** ise 7 günlük bekleme süresi boyunca hem instance'ı hem yedekleri yerinde bırakır ve iptal edilebilir.
