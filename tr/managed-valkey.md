# Valkey

Valkey, Komuta'nın kurup işlettiği bellek içi bir anahtar-değer deposudur. Sunucu kurulumu, TLS, günlük yedek ve izleme kuralları instance oluşturulurken hazır gelir; uygulamaya düşen tek iş verilen bağlantı adresini kullanmaktır.

---

## Valkey Nedir?

Valkey, veriyi diskte değil bellekte tutan bir anahtar-değer (key-value) deposudur. Her kayda bir anahtar üzerinden erişilir ve işlem doğrudan bellekte yapıldığı için yanıt süresi milisaniyenin altında kalır — ilişkisel bir veritabanının indeks okuyup diske gitmesi gereken yerde Valkey tek adımda cevap verir. Bunun karşılığı kapasitedir: veri diskle değil bellekle sınırlıdır (bkz. [Kullanılabilir Bellek](#kullanlabilir-bellek)).

### Ne tutabilir

Anahtarların değeri düz metinle sınırlı değildir. Valkey; metin, sayı, hash (alan-değer çiftleri), liste, küme (set), sıralı küme (sorted set), bitmap, HyperLogLog ve akış (stream) yapılarını doğrudan tutar.

Bu yapılar üzerindeki işlemler sunucu tarafında yapılır: bir listeye eleman eklemek ya da bir sayacı artırmak için verinin çekilip geri yazılması gerekmez — tek komut yeterlidir ve komut atomiktir, yani eşzamanlı iki istek birbirinin sonucunu bozmaz. Ayrıca her anahtara ömür (TTL) verilebilir; süresi geçen anahtar kendiliğinden silinir.

### Ne için kullanılır

- **Önbellek.** Pahalı bir sorgunun ya da dış API çağrısının sonucu bir anahtara TTL ile yazılır; sonraki istekler veritabanına hiç gitmez. En yaygın kullanım budur.
- **Oturum deposu.** Oturum bilgisi uygulamanın belleğinde değil ortak bir yerde tutulur; böylece uygulamanın hangi kopyasına düşerse düşsün istek aynı oturumu görür ve dağıtım sırasında oturumlar kaybolmaz.
- **Kuyruk ve iş listesi.** Liste ve akış yapıları, arka planda işlenecek işleri sıraya almak için kullanılır.
- **Sayaç ve oran sınırlama.** Atomik artırma ile TTL bir araya geldiğinde "bu IP'den dakikada kaç istek geldi" gibi sayaçlar tek komutla tutulabilir.
- **Pub/sub.** Bir kanala yayınlanan mesaj, o kanalı dinleyen tüm istemcilere iletilir; anlık bildirim ve önbellek geçersizleştirme gibi işler için kullanılır.

Bu kullanımların hangisi seçilirse seçilsin, bellek dolduğunda ne olacağı [tahliye politikasına](#tahliye-politikas) bağlıdır — ve doğru politika kullanıma göre değişir.

### Redis ile ilişkisi

Valkey, Redis'in 7.2.4 sürümünden Mart 2024'te ayrılmış bir çatallamadır (fork). Redis'in lisansı açık kaynak olmayan bir modele geçtiğinde, projenin uzun süreli geliştiricileri kod tabanını BSD 3-Clause lisansı altında sürdürmek üzere Linux Foundation çatısı altında Valkey'i kurdu; AWS, Google Cloud, Oracle, Ericsson ve Snap projeye destek veren kuruluşlar arasındadır.

Pratikte bunun karşılığı, protokolün ve komut kümesinin korunmuş olmasıdır. Mevcut Redis istemcileri, sürücüleri ve araçları — `redis-cli`, ioredis, `redis-py`, Spring Data Redis — Valkey'e bağlanmak için değişiklik gerektirmez; değişen tek şey bağlantı adresi ve şemasıdır (bkz. [Bağlanma](#balanma)).

---

## Nasıl Çalışır?

Bir instance oluşturulduğunda Komuta seçilen bölgede bir Valkey kümesi kurar, TLS'i açar, bağlantı kimlik bilgilerini üretir, günlük yedeği ve izleme kurallarını yerleştirir. Bu adımların hiçbiri ayrıca yapılandırma gerektirmez.

### Instance, plan ve bölge

Bir **instance**, kendi belleği, kendi kimlik bilgisi ve kendi yedek zinciriyle çalışan bağımsız bir Valkey sunucusudur. CPU, bellek, disk ve sunucu (replika) sayısı bağlı olduğu **plandan** gelir. Plandaki bellek yalnız bir fiyat farkı değil, verinin ne kadar yer bulacağını belirleyen değerdir (bkz. [Kullanılabilir Bellek](#kullanlabilir-bellek)).

**Bölge**, instance'ın fiziksel olarak çalıştığı yerdir ve oluşturulduktan sonra değiştirilemez. Başka bir bölgeye taşımanın yolu yeni bir instance kurmaktır.

**Sürüm**, instance'ın çalıştıracağı Valkey sürümüdür ve seçilen planın desteklediği sürümler arasından belirlenir. Sürüm sonradan değiştirilemez.

**Örnek adı** küçük harfle başlar, yalnız küçük harf, rakam ve tire içerir, en fazla 40 karakterdir; küme içindeki kaynak adı olarak kullanılır. Instance isteğe bağlı olarak bir **projeye** bağlanabilir, bağlanmadığında paylaşımlı sayılır.

### Verinin durduğu yer

Veri bellekte tutulur; diske snapshot olarak yazılır. Bu yüzden planın belleğinin bir kısmı veriye açılmaz — snapshot alınırken geçici bir kopya oluşur ve bu kopyanın sığacağı yer önceden ayrılmıştır.

### Instance durumları

| Durum | Anlamı |
|---|---|
| **Hazırlanıyor** | Kurulum sürüyor; bağlantı bilgisi henüz hazır değildir. |
| **Aktif** | Çalışıyor ve bağlantı kabul ediyor. |
| **Askıda** / **Tamamen Askıda** | Elle askıya alınmış, bağlantı kabul etmiyor. |
| **Yükseltiliyor** | Plan değişikliği uygulanıyor. |
| **Yükseltme Başarısız** | Plan değişikliği tamamlanamadı; instance eski planıyla **canlıdır** ve verisi yerindedir. |
| **Silinmeyi Bekliyor** | Silme planlanmış; süre dolmadan iptal edilebilir. |
| **Hata** | Kurulum tamamlanamamış instance. Bu durumdaki instance'lar platform tarafından otomatik temizlenir; yedekleri saklama süresi boyunca yerinde kalır. |

![Valkey listesi](https://cdn.komuta.io/docs/tr/images/valkey/home-page.png)

---

## Kullanılabilir Bellek

Veri için kullanılabilecek bellek, planın toplam belleğinin **%75'idir.** Kalan pay sunucunun kendi ihtiyaçları içindir: bellek parçalanması, replikasyon tamponu ve snapshot alınırken oluşan geçici kopya. Bu pay olmadan sunucu, snapshot alırken bellek sınırını aşıp kapanır.

| Plan belleği | Veri için kullanılabilir |
|---|---|
| 1 GB | ~768 MB |
| 2 GB | ~1,5 GB |
| 4 GB | ~3 GB |
| 8 GB | ~6 GB |

Sınıra ulaşıldığında ne olacağını instance'ın tahliye politikası belirler; iki davranıştan biri geçerlidir (bkz. [Tahliye Politikası](#tahliye-politikas)).

---

## Tahliye Politikası

Tahliye (eviction) politikası, bellek sınırına varıldığında Valkey'nin yeni yazmalarla ne yapacağını belirler. Politika instance oluşturulurken belirlenir; sonradan değiştirilebilen bir ayar değildir. İki davranış vardır ve sınıra ulaşıldığında sırasıyla şunları yaparlar.

### Tahliye yok (varsayılan)

Yeni yazma komutları hata döner ve **hiçbir anahtar silinmez.** Bellekteki veri olduğu gibi kalır, okuma çalışmaya devam eder; sınır aşılmadığı için sunucu da kapanmaz. Veri kaybetmemesi gereken kullanımlar — kuyruk, oturum deposu, iş listesi — için doğru davranış budur. Karşılığı, uygulamanın bu yazma hatasını ele almak zorunda olmasıdır: hata gelen yazma sessizce kaybolmaz, ama kendiliğinden de tekrar denenmez.

### Tahliye politikalı

En az kullanılan ya da süresi dolmak üzere olan anahtarlar silinir ve **yazma kesintisiz sürer.** Uygulama bellek sınırını hiç fark etmez; karşılığı, sınırdaki her yeni yazmanın eski bir anahtarın yerini almasıdır. Önbellek kullanımı için uygundur, çünkü silinen anahtar kaynağından tekrar üretilebilir.

> **İpucu:** Önbellek olarak kullanılacaksa politika baştan tahliyeli seçilmelidir. "Tahliye yok" ile başlayıp belleği doldurmanın sonucu, uygulamanın beklemediği bir anda yazma hatası almasıdır.

---

## Ağ Erişimi

Her instance üç erişim modundan biriyle çalışır. Mod değişikliği anında geçerli olur, instance yeniden başlatılmaz. Üç mod sırasıyla şunları yapar.

### Herkese açık

Genel bağlantı adresi internetten erişilebilir ve kaynak IP kısıtı uygulanmaz; kimlik bilgisine sahip herkes bağlanabilir. Yeni bir instance bu modla başlar.

### IP listesi ile kısıtlı

Genel adres açık kalır, ama yalnız listelenen kaynaklardan gelen bağlantılar kabul edilir; listede olmayan her kaynak ağ geçidinde reddedilir — bağlantı instance'a hiç ulaşmaz.

Liste kuralları:

- Her satıra bir IP ya da CIDR yazılır.
- Tek bir adres yazıldığında (`203.0.113.4`) otomatik olarak tek makineye (`/32`) genişletilir.
- Geçersiz bir ifade sessizce yok sayılmaz, hata verir.
- Liste boş bırakılamaz. Boş liste herkesi reddedeceği için istek geri çevrilir; erişimi tamamen kapatmanın yolu özel ağ modudur.

Bu mod her kümede kullanılamaz: bazı kümelerde ağ geçidi gerçek kaynak adresi göremediği için liste hiçbir şeyi kısıtlamaz — herkesi kabul eden bir "kısıtlama" bırakmamak için mod orada kapalı tutulur. Böyle bir durumda erişimi daraltmanın yolu özel ağ modudur.

### Yalnızca özel ağ

Genel bağlantı adresi tamamen kaldırılır; instance'a yalnız özel ağdaki kendi servislerinizden erişilir. Bu mod ancak uygulama kümeniz özel ağa dahil edilmişse seçilebilir — aksi hâlde instance hiçbir yerden erişilemez hâle geleceği için istek reddedilir.

> **Uyarı:** Özel ağ moduna geçildiği anda genel adres silinir. Kümelerinizin dışından bağlanan her istemci — bilgisayarınızdaki bir GUI istemcisi, bir CI işi, Komuta'da çalışmayan her şey — anında düşer ve yeniden bağlanamaz. Özel adresi kullanan servisler etkilenmez.

![Ağ erişimi kartı](https://cdn.komuta.io/docs/tr/images/valkey/network-access.png)

---

## Bağlanma

### Bağlantı adresi ve TLS

Her instance için bir hostname verilir ve bağlantılarda bu ad **olduğu gibi** kullanılmalıdır: trafiği doğru instance'a yönlendiren altyapı, TLS el sıkışmasının içindeki sunucu adına bakarak karar verir.

- Adresi IP'ye çevirip bağlanmak çalışmaz. IP ile kurulan bağlantı doğru instance'a ulaşmaz; genellikle bağlantı kurulmuş gibi görünüp hemen düşer.
- Sunucu adını (SNI) göndermeyen eski istemciler ve adresi IP'ye sabitleyen bağlantı havuzları aynı nedenle çalışmaz.

TLS açık olduğunda bağlantı `rediss://` şemasıyla kurulur; varsayılan port `6379`'dur. `redis-cli` gibi araçlarda TLS'in ayrıca açılması gerekir (`--tls`).

### Kimlik bilgileri

Bağlantı bilgisi host, port, kullanıcı adı ve paroladan oluşur; Redis uyumlu istemciler bu parolayla kimlik doğrular.

Parolalar platform tarafında şifrelenmiş saklanır ve yalnız bağlantı bilgisi görüntülendiğinde çözülür. **Her görüntüleme; instance, kullanıcı ve zaman bilgisiyle denetim kaydına yazılır.** Kimlik bilgisi paylaşmak yerine ekip üyelerine kendi hesaplarından erişim verilmesi önerilir.

Valkey'de kimlik bilgisi yenileme (rotate) işlemi bulunmaz; parolanın değişmesi gerekiyorsa destek talebi açılmalıdır.

### Özel ağ adresi

Özel ağ yolu olan instance'lar genel adresin yanında bir de özel adres alır. Bu adres kümedeki servislerden erişilir ve **erişim modundan bağımsızdır** — genel adres kapatıldığında ayakta kalan yol budur. Özel adres instance'ın kendi sertifikasını sunar; istemciden sunucu doğrulaması beklemez.

![Genel Bakış sekmesi](https://cdn.komuta.io/docs/tr/images/valkey/dashboard.png)

---

## Yedekleme

Günde bir kez tam yedek alınır ve nesne depolamasında tutulur. Yedek, alındığı andaki bellek içeriğinin kopyasıdır; iki yedek arasındaki değişiklikler kapsanmaz.

### Zamanlama ve saklama süresi

- Yedeğin saati beş alanlı bir cron ifadesiyle verilir — `0 3 * * *` her gün UTC 03:00 demektir.
- Saklama süresi gün olarak yazılır ve 1 ile 365 gün arasında olabilir; varsayılan 7 gündür.
- Planlı yedeğin dışında elle yedek alınabilir. Bu, günlük yedeğin yerine geçmez; ona ek bir kopya üretir. Riskli bir işlemden hemen önce bir koruma noktası isteniyorsa doğru yol budur.

Yedek saatini değiştirmek saklama süresini etkilemez; saklama süresi yalnız açıkça değiştirildiğinde güncellenir. Son yedeğin 26 saatten eski olması ayrıca işaretlenir — bu, günlük zamanlamanın geciktiği anlamına gelir.

> **Uyarı:** Saklama süresini kısaltmak geri alınamaz. Süre düşürüldüğünde yeni pencerenin dışında kalan yedekler bir sonraki bakımda kalıcı olarak silinir. Süreyi tekrar uzatmak silinenleri geri getirmez; koruma penceresinin yeniden dolması o kadar gün alır.

### Yedeklerin imhası

Yedekler yalnızca iki durumda kalıcı olarak silinir:

- Planlı silme işleminin geri alma süresi dolduğunda,
- Bir yöneticinin açık kalıcı silme talebiyle.

Bunların dışındaki hiçbir yol yedeklere dokunmaz. Instance hata durumuna düşüp otomatik temizlenirse ya da kurulum yarıda kalırsa yedekler saklama süresi boyunca yerinde kalır.

> **Not:** Yedeğin durması, geri yüklemenin arayüzden yapılabileceği anlamına gelmez. Bir Valkey yedeğine dönmek için destek talebi açılması gerekir.

---

## Plan Değişikliği

Plan değişikliği CPU, bellek, disk ve sunucu sayısını hedef planın değerlerine taşır. Bellek değiştiği için veri için kullanılabilir alan da aynı oranda değişir (bkz. [Kullanılabilir Bellek](#kullanlabilir-bellek)).

Hedef plan seçildiğinde bir etki önizlemesi hesaplanır: değişikliğin türü (yükseltme, düşürme, topoloji değişikliği), tahmini kesinti, kaynak değişiklikleri ve varsa uyarılar. Önizleme uygulanamaz bir değişiklik bildirdiğinde işlem başlatılamaz.

### Kısıtlar

- **Disk küçültülemez.** Valkey'de daha küçük diskli bir plana geçme isteği reddedilir. Bu bir platform kısıtı değil, blok depolamanın kendi kuralıdır.
- **Düşük plana geçiş mümkündür, ama bellek kontrolüne tabidir.** Bellekteki veri hedef planın veri payına sığmıyorsa değişiklik yapılmaz.
- **Kapasite düğüm bazında ölçülür.** Bir replikanın diski tek bir fiziksel düğümde büyümek zorundadır; kümenin toplam boş alanı yeterli görünse bile o düğümde yer yoksa değişiklik yapılamaz.
- **Önizleme bir garanti değildir.** Kapasite kontrolü onay anında yeniden çalışır; önizleme hesaplandıktan sonra beklenirse küme dolmuş olabilir ve aynı değişiklik reddedilebilir.

### Kesinti ve yüksek erişilebilirlik

Kaynak (CPU/RAM/disk) değişikliğinde sunucular sırayla yeniden başlatılır; beklenen kesinti yaklaşık 30 saniye – 2 dakikadır ve çok sunuculu kurulumlarda en az bir sunucu ayakta kalır. Sunucu sayısını değiştiren bir geçişte süre 2–5 dakikaya çıkar ve kısa bağlantı kesintileri olur.

Yüksek erişilebilirlikli planlarda replikalar farklı fiziksel düğümlere zorunlu olarak dağıtılır; tek bir düğümün kaybı hizmeti durdurmaz. Tek sunuculu planlarda böyle bir koruma yoktur ve bakım ile düğüm değişimlerinde kısa kesinti yaşanabilir. Her iki durumda da uygulamanın bağlantı kopmasına dayanıklı olması — yeniden bağlanma ve isteği tekrar deneme — gerekir.

### Yükseltme başarısız olursa

Plan değişikliği yarıda kalırsa instance eski planıyla çalışmaya devam eder ve **Yükseltme Başarısız** durumuna geçer. Bu durumdaki bir instance canlıdır, verisi yerindedir ve aynı değişiklik yeniden denenebilir. Sorun tekrarlıyorsa destek talebi açılmalıdır.

![Plan değiştir paneli](https://cdn.komuta.io/docs/tr/images/valkey/change-plan.png)

---

## Bakım ve Yaşam Döngüsü

### Yeniden başlatma

Sunucular sırayla yeniden başlatılır; replikası olan planlarda kesinti oluşmaz. Tek sunuculu bir planda instance kısa süre bağlantı kabul etmez.

### Askıya alma ve devam ettirme

İki askıya alma biçimi vardır ve aralarındaki fark geri dönüş süresidir:

- **Soft askıya alma** istemci bağlantılarını boşaltır, sunucuları çalışır tutar. Geri açma saniyeler sürer.
- **Hard askıya alma** sunucuları tamamen durdurur. Geri açma, küme yeniden zamanlama yaptığı için birkaç dakika sürer.

Her iki durumda da askıdaki instance bağlantı kabul etmez; uygulama bağlantı hatası alır.

### Silme

İki yol vardır ve sonuçları farklıdır:

- **Silmeyi planlamak** instance'ı 7 günlük bir bekleme süresinin sonunda silinmek üzere işaretler. Süre dolmadan iptal edilebilir; instance bu süre boyunca çalışmaya ve faturalanmaya devam eder.
- **Kalıcı silme** instance'ı ve **yedekler dahil tüm verisini** anında yok eder.

> **Uyarı:** Kalıcı silmenin geri dönüşü yoktur ve yedekler de silindiği için veri destek üzerinden de kurtarılamaz. Verinin bir süre daha durması isteniyorsa doğru seçim planlı silmedir.

---

## Metrikler ve Uyarılar

İzleme kuralları instance oluşturulurken otomatik kurulur; ayrıca yapılandırma gerekmez. Valkey'de kapsanan başlıklar: bellek kullanımı, kalıcılık (snapshot) hatası, istemci sayısı, CPU ve instance erişilebilirliği. Tetiklenen uyarılar uyarı geçmişinde listelenir.

Grafiklerde kararı değiştiren üç ölçü şunlardır:

| Ölçü | Ne anlatır |
|---|---|
| **Bellek kullanımı** | Veri payına ne kadar yaklaşıldığı. Sınıra varıldığında tahliye politikası devreye girer. |
| **Tahliye oranı** | Sıfırdan büyük olması, instance'ın bellek sınırında çalıştığı ve anahtarların silinmekte olduğu anlamına gelir. "Tahliye yok" politikasında bu değer artmaz; onun yerine uygulama yazma hatası alır. |
| **İsabet oranı** | Sorgulanan anahtarların ne kadarının bellekte bulunduğu. Düşmesi, verinin tahliye edildiğine ya da anahtar ömürlerinin kısa olduğuna işaret eder. |

---

## Sık Sorulan Sorular

### Redis istemcilerim Valkey ile çalışır mı?

Çalışır. Valkey, Redis ile uyumlu protokol konuşur; `redis-cli`, ioredis ve `redis-py` gibi istemciler değişiklik gerektirmez. TLS açık olduğu için bağlantı şeması `redis://` değil `rediss://` olur.

### Planın belleğinin tamamını veri için kullanabilir miyim?

Kullanamazsınız. Planın belleğinin %75'i veriye açılır; kalan pay bellek parçalanması, replikasyon tamponu ve snapshot kopyası içindir. 4 GB'lık bir planda veri için yaklaşık 3 GB yer vardır.

### Bellek dolduğunda veri kaybeder miyim?

Tahliye politikasına bağlıdır. Varsayılan "tahliye yok" davranışında hiçbir anahtar silinmez ve yazma komutları hata döner. Tahliye politikalı bir instance'ta en az kullanılan anahtarlar silinerek yazmaya devam edilir.

### Erişim modunu değiştirmek bağlantıları koparır mı?

Herkese açık ve IP listesi modları arasındaki geçişte instance yeniden başlatılmaz. Özel ağ moduna geçiş ise genel adresi sildiği için dışarıdan bağlanan tüm istemcileri anında düşürür; özel adresi kullanan servisler etkilenmez.

### Instance silinince yedekler de gider mi?

**Kalıcı silme** yedekleri de yok eder ve geri dönüşü yoktur. **Silmeyi planlamak** ise 7 günlük bekleme süresi boyunca hem instance'ı hem yedekleri yerinde bırakır ve iptal edilebilir.
