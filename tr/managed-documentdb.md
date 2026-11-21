# DocumentDB

DocumentDB, Komuta'nın kurup işlettiği MongoDB uyumlu bir doküman veritabanıdır. Şifreli bağlantı, günlük yedek, sürekli değişiklik arşivi ve izleme kuralları instance oluşturulurken hazır gelir; uygulamaya düşen tek iş, verilen `mongodb://` adresini MongoDB sürücüsüne vermektir.

---

## DocumentDB Nedir?

DocumentDB, veriyi tablo satırlarında değil **belgelerde** tutar. Belge, JSON'a benzeyen bir kayıttır (saklama biçimi BSON'dur); iç içe nesneler ve diziler ayrı tablolara bölünmeden aynı belgede durur ve doğrudan sorgulanır. Belgeler **koleksiyonlarda**, koleksiyonlar **veritabanlarında** toplanır. Bir koleksiyondaki belgelerin aynı alanlara sahip olması gerekmez; şema, istenirse doğrulama kurallarıyla sonradan sıkılaştırılır.

### Ne için kullanılır

- **Değişken yapılı kayıtlar.** Ürün katalogları, kullanıcı profilleri, form yanıtları gibi her kaydın biraz farklı alan taşıdığı veriler.
- **İçerik ve olay kayıtları.** Bir sipariş ve kalemleri, bir makale ve etiketleri gibi birlikte okunan verinin tek belgede tutulması.
- **Mevcut MongoDB uygulamaları.** Uygulama kodu MongoDB sürücüsüyle yazılmışsa çoğu durumda değişen tek şey bağlantı adresidir (bkz. [Uyumluluk ve Sınırlar](#uyumluluk-ve-sınırlar)).
- **Vektör arama.** Gömme (embedding) vektörleri belgelerle birlikte saklanır ve benzerlik aramasıyla sorgulanır.

İlişkisel veri, karmaşık birleştirmeler ve SQL gerektiren işler için [PostgreSQL](managed-postgresql.md) daha uygun bir seçimdir.

### Açık kaynak DocumentDB projesi

Servis, Linux Foundation çatısı altında MIT lisansıyla geliştirilen açık kaynak **DocumentDB** projesini çalıştırır. Proje MongoDB'nin kablo protokolünü konuşur: MongoDB sürücüleri, `mongosh` ve sürücü üzerine kurulu kütüphaneler (Mongoose, Spring Data MongoDB vb.) ayrı bir istemci gerektirmeden bağlanır. DocumentDB, MongoDB'nin kendisi değildir; komutların büyük kısmını destekler ama hepsini değil. Desteklenmeyenler [Uyumluluk ve Sınırlar](#uyumluluk-ve-sınırlar) bölümünde listelenir.

> **Not:** AWS'nin **Amazon DocumentDB** servisi ayrı bir üründür; bu sayfadaki servisle bir ilişkisi yoktur.

---

## Nasıl Çalışır?

Bir instance oluşturulduğunda Komuta seçilen bölgede bir DocumentDB sunucusu kurar, TLS'i açar, bağlantı kullanıcısını ve parolasını üretir, günlük yedeği ve sürekli değişiklik arşivini başlatır, izleme kurallarını yerleştirir. Bu adımların hiçbiri ayrıca yapılandırma gerektirmez.

### Instance, plan ve bölge

Bir **instance**, kendi diski, kendi kullanıcısı ve kendi yedek zinciriyle çalışan bağımsız bir veritabanı sunucusudur. CPU, bellek ve disk bağlı olduğu **plandan** gelir (bkz. [Planlar](#planlar)).

**Bölge**, instance'ın fiziksel olarak çalıştığı yerdir ve oluşturulduktan sonra değiştirilemez. Başka bir bölgeye taşımanın yolu yeni bir instance oluşturup veriyi aktarmaktır.

**Sürüm**, instance'ın çalıştıracağı DocumentDB sürümüdür; bugün `1.0` sunulur. Sürüm sonradan değiştirilemez.

**Örnek adı** küçük harfle başlar, yalnız küçük harf, rakam ve tire içerir, en fazla 40 karakterdir. Instance isteğe bağlı olarak bir **projeye** bağlanabilir.

Oluştururken veritabanı adı sorulmaz: veritabanları ve koleksiyonlar uygulama onlara ilk kez yazdığında kendiliğinden açılır.

### Instance oluşturma

Yeni bir instance, **Yönetilen Servisler** sayfasındaki DocumentDB sekmesinde **Yeni DocumentDB** butonuyla oluşturulur. Bölge, plan, ad ve sürüm seçilir; son adımda seçilenlerin özeti ve tahmini aylık maliyet görünür. **Oluştur** ile kurulum kuyruğa alınır; instance **Hazırlanıyor** durumunda listeye düşer ve kurulum tamamlanana kadar bağlantı bilgisi üretilmez.

### Instance durumları

| Durum | Anlamı |
|---|---|
| **Hazırlanıyor** | Kurulum sürüyor; bağlantı bilgisi henüz hazır değildir. |
| **Aktif** | Çalışıyor ve bağlantı kabul ediyor. |
| **Yükseltiliyor** | Plan değişikliği uygulanıyor. |
| **Yükseltme Başarısız** | Plan değişikliği tamamlanamadı; instance eski planıyla **canlıdır** ve verisi yerindedir. |
| **Geri Yükleniyor** | Bir geri yükleme işlemi sürüyor. |
| **Silinmeyi Bekliyor** | Silme planlanmış; bağlantı adresi kapalıdır ve yalnız disk faturalanır. Süre dolmadan iptal edilebilir. |
| **Hata** | Kurulum tamamlanamamış instance. Bu durumdaki instance'lar platform tarafından otomatik temizlenir; yedekleri saklama süresi boyunca yerinde kalır. |

DocumentDB'de askıya alma ve uyku yoktur; instance var olduğu sürece çalışır (bkz. [Bakım ve Yaşam Döngüsü](#bakım-ve-yaşam-döngüsü)).

---

## Planlar

| Plan | vCPU | Bellek | Disk |
|---|---|---|---|
| 1 GB | 0,5 | 1 GB | 10 GB |
| 2 GB | 0,5 | 2 GB | 40 GB |
| 4 GB (varsayılan) | 1 | 4 GB | 80 GB |
| 8 GB | 2 | 8 GB | 160 GB |
| 16 GB | 4 | 16 GB | 320 GB |
| 32 GB | 8 | 32 GB | 640 GB |

Güncel planlar ve fiyatları oluşturma sihirbazında listelenir.

- **Bütün planlar tek sunuculudur.** Yüksek erişilebilirlik için ikinci bir sunucu (replika) yoktur. Bakım, plan değişikliği ve düğüm değişimlerinde kısa kesinti yaşanabilir; uygulamanın kopan bağlantıdan sonra yeniden bağlanması gerekir.
- **1 GB planda organizasyon başına en fazla 5 instance** açılabilir. Silinmeyi bekleyenler dahil her durumdaki instance sayılır; PostgreSQL'in küçük planlarından ayrı sayılır. Daha büyük planlarda böyle bir sınır yoktur.

---

## Bağlanma

### Bağlantı adresi

**Genel Bakış** sekmesindeki **Bağlantı bilgisini göster** ile host, port, kullanıcı adı, parola ve hazır bir `mongodb://` bağlantı dizesi açılır. Bağlantı dizesi her MongoDB sürücüsünde ve `mongosh`'ta olduğu gibi çalışır:

```text
mongodb://<kullanıcı>:<parola>@<host>:10260/?tls=true&authMechanism=SCRAM-SHA-256&retryWrites=false&maxIdleTimeMS=1500000
```

- **Port `10260`'tır**, MongoDB'nin alışılmış `27017` portu değildir.
- **Bağlantı her zaman TLS'lidir.** Sertifikayı Komuta yönetir; genel olarak güvenilen bir sertifika olduğu için ayrıca bir CA dosyası gerekmez. Giriş SCRAM-SHA-256 ile yapılır.
- **Host adı olduğu gibi kullanılmalıdır.** Trafiği doğru instance'a yönlendiren altyapı, TLS el sıkışmasındaki sunucu adına göre karar verir. Adresi IP'ye çevirip bağlanmak çalışmaz; bağlantı kurulmuş gibi görünüp hemen düşer. Sunucu adını (SNI) göndermeyen eski istemciler de aynı nedenle çalışmaz.
- `mongodb+srv://` adresi ve replika kümesi adı (`replicaSet`) verilmez; gerekmez de.

Bağlantı dizesini elle kurmak yerine arayüzün verdiği dize kopyalanmalıdır; içindeki iki parametre özellikle korunmalıdır:

- **`retryWrites=false` kaldırılmamalıdır.** Sürücüler, bağlantı koptuğunda yazma işlemini varsayılan olarak bir kez daha gönderir. Sunucu böyle yeniden gönderilen bir yazmayı ikinci kez uygulayabilir: bir sayaç iki kez artar, tek kayıt silmesi iki kayıt siler. Bu parametre sürücünün otomatik tekrarını kapatır. Bunun karşılığı, bağlantı koptuğunda yazmanın hata olarak uygulamaya dönmesidir; yeniden denemeyi uygulama, işlemin uygulanıp uygulanmadığını kontrol ederek kendisi yapmalıdır.
- **`maxIdleTimeMS=1500000` kaldırılmamalıdır.** Genel adreste 30 dakika boyunca hiç trafik görmeyen bağlantılar kapatılır. Bu parametre sürücünün boştaki bağlantıyı 25 dakikada kendisinin kapatıp yenilemesini sağlar; yoksa uzun süre boşta kalan bir bağlantıdaki ilk istek hata alır.

Örnek kullanım (dize bir ortam değişkeninde tutularak):

```bash
mongosh "$MONGODB_URI"
```

```js
// Node.js (mongodb sürücüsü)
const { MongoClient } = require("mongodb");
const client = new MongoClient(process.env.MONGODB_URI);
```

```python
# Python (pymongo)
import os
from pymongo import MongoClient
client = MongoClient(os.environ["MONGODB_URI"])
```

### Size verilen kullanıcı

Bağlantı bilgisindeki kullanıcı, instance'taki **bütün veritabanlarında okuma ve yazma yetkisine** sahiptir (`readWriteAnyDatabase` ve `clusterAdmin` rolleri):

- Veritabanı ve koleksiyon oluşturmak, silmek, yeniden adlandırmak
- Her türlü belge işlemi, toplama (aggregation) ve transaction
- İndeks, view ve şema doğrulama kuralları oluşturmak

Kullanıcı yönetimi bu kullanıcıya açık değildir: `createUser` gibi komutlar yetki hatası verir, yani instance içinde ek kullanıcı açılamaz. Yalnız okuması gereken bir araç ya da kişi için konsoldan salt-okunur kullanıcı açılır (bkz. [Salt-okunur kullanıcılar](#salt-okunur-kullanıcılar)).

### Bağlantı bilgisinin görüntülenmesi

Parolalar platform tarafında şifrelenmiş saklanır ve yalnız **Bağlantı bilgisini göster** açıldığında çözülür. **Her görüntüleme; instance, kullanıcı ve zaman bilgisiyle kaydedilir.** Kimlik bilgisi paylaşmak yerine ekip üyelerine kendi hesaplarından erişim verilmesi önerilir.

Instance'ın özel ağ yolu varsa bağlantı dizelerinin sonunda **Bağlantı Dizesi (özel ağ)** ayrıca yer alır (bkz. [Özel adres](#özel-adres)).

### Salt-okunur kullanıcılar

**Genel Bakış** sekmesindeki **Salt-okunur erişim** kartı, bir AI aracına ya da yalnız okuması gereken birine ana parola yerine verilecek kullanıcıyı açar.

- **Kullanıcı aç** yeni bir kullanıcı ve bağlantı dizesi üretir. Parola **yalnız o anda bir kez** gösterilir; Komuta saklamaz. Kaybedilirse kullanıcı kapatılıp yenisi açılır.
- Bu kullanıcı instance'taki bütün veritabanlarını okur, hiçbir şeyi değiştiremez: yazma, silme ve kullanıcı yönetimi komutları reddedilir.
- Aynı anda en fazla **3** salt-okunur kullanıcı açık olabilir. Kullanıcı, kapatılana kadar açık kalır.
- **Kapat** kullanıcıyı hemen kapatır: o kullanıcıyla açılmış bağlantılar bir sonraki istekte reddedilir ve parola bir daha çalışmaz.
- Instance IP izin listesiyle kısıtlıysa aracın bağlandığı adres listeye eklenmelidir; **Yalnızca özel** moddaysa özel ağın dışında çalışan bir araç bağlanamaz.

> **Not:** Bir AI aracı bu kullanıcıyla bağlandığında, okuduğu veri o aracı sunan şirkete gider.

---

## Ağ Erişimi

Erişim modu **Genel Bakış** sekmesindeki **Ağ erişimi** kartından değiştirilir. Mod değişikliği anında geçerli olur; instance yeniden başlatılmaz.

### Erişim modları

| Mod | Anlamı |
|---|---|
| **Genel** | Genel adres üzerinden internetten erişilebilir, kaynak IP kısıtı yok. Varsayılan. |
| **Genel, IP izin listesi** | Genel adres yalnız belirtilen IP/CIDR aralıklarına açıktır. |
| **Yalnızca özel** | Genel adres tamamen kaldırılır; instance'a yalnız özel ağdan erişilir. |

Bir mod **Henüz kullanılamıyor** olarak işaretliyse gerekçesi kartın içinde yazar. IP izin listesi, kümenin bağlantının gerçek kaynak adresini göremediği durumlarda kapalı kalır (aksi hâlde liste sessizce herkesi kabul ederdi). **Yalnızca özel** ise ancak organizasyonun uygulama kümesi özel ağa dahil edildiğinde seçilebilir; aksi hâlde instance hiçbir yerden erişilemez hâle gelirdi ve istek reddedilir.

> **Uyarı:** **Yalnızca özel** moda geçmek genel bağlantı adresini siler. Kümelerin dışından bağlanan her istemci — dizüstü bilgisayar, bir GUI istemcisi, bir CI işi, Komuta üzerinde çalışmayan her şey — kopar ve yeniden bağlanamaz. Özel adresi kullanan servisler etkilenmez.

### IP izin listesi

İzin listesi her satıra bir CIDR ya da IP yazılarak doldurulur:

```text
203.0.113.0/24
198.51.100.7
```

- Tek başına yazılan bir adres otomatik olarak tek makineye (`/32`) genişletilir.
- Listede olmayan her kaynak instance'a ulaşmadan reddedilir.
- Liste boş bırakılamaz — boş liste tüm erişimi keseceği için istek reddedilir. Amaç tüm dış erişimi kapatmaksa doğru seçim **Yalnızca özel** moddur.
- Geçersiz bir ifade sessizce yok sayılmaz; hata verir.

### Özel adres

Organizasyonun özel ağa dahil kümelerindeki servisler instance'a özel adresiyle bağlanır. Bu adres bağlantı dizelerinde **Bağlantı Dizesi (özel ağ)** olarak yer alır ve **her modda çalışır** — genel modda da geçerlidir; genel adresin kaldırılması onu etkilemez. **Yalnızca özel** modda ana bağlantı dizesinin kendisi özel adrestir.

```text
mongodb://<kullanıcı>:<parola>@<özel-host>:10260/?authMechanism=SCRAM-SHA-256&retryWrites=false
```

Özel adres TLS kullanmaz, bu yüzden dizede `tls=true` yoktur ve hiçbir sertifika ayarı gerekmez. Giriş yine SCRAM-SHA-256 ile doğrulanır ve parola ağdan açık metin olarak geçmez; belgeler ve sorgular ise özel ağ içinde şifrelenmeden taşınır. `retryWrites=false` burada da aynı nedenle korunmalıdır.

---

## Yedekleme

### Yedek düzeni ve saklama süresi

Koruma iki parçadan oluşur ve ikisi birlikte çalışır:

- **Tam yedek** günde bir kez alınır. Yedeğin saati her instance için ayrı belirlenir.
- **Değişiklikler sürekli arşivlenir.** Belirli bir zamana geri dönüş (PITR) bu sayede mümkündür; koruma iki tam yedek arasındaki değişiklikleri de kapsar.

Varsayılan saklama süresi **7 gündür** (2–365 gün arasında değiştirilebilir) ve hem tam yedekleri hem değişiklik arşivini kapsar. Geri dönülebilecek en eski nokta da bu süreyle belirlenir.

Belirli bir işlemden önce ek bir kopya gerekiyorsa **İşlemler** sekmesindeki **Şimdi yedekle** ile elle bir yedek alınabilir. Bu günlük yedeğin yerine geçmez, ona ek bir kopya üretir.

### Backup zamanlaması

Günlük yedeğin saati ve saklama süresi **Yedekler** sekmesindeki **Backup zamanlaması** kartından değiştirilir. Saat beş alanlı bir cron ifadesiyle verilir (`0 3 * * *` → her gün UTC 03:00), saklama süresi gün olarak yazılır. Yedek saatini değiştirmek saklama süresini etkilemez.

> **Uyarı:** Saklama süresini kısaltmak geri alınamaz. Süre düşürüldüğünde yeni pencerenin dışında kalan yedekler bir sonraki bakımda kalıcı olarak silinir. Süreyi tekrar uzatmak silinenleri geri getirmez; koruma penceresinin yeniden dolması o kadar gün alır.

### Backup sağlığı

Yedeklerin sağlığı günde bir kez denetlenir ve **Yedekler** sekmesinde üç durumdan biri gösterilir:

| Durum | Anlamı | Ne yapmalı |
|---|---|---|
| **Sağlıklı** | Son yedek ve değişiklik arşivi 26 saatten daha yeni. | — |
| **Sorunlu** | Denetim çalıştı ama son yedek ya da arşiv beklenenden eski. | Disk doluluğunu kontrol edin; sürerse destek talebi açın. |
| **Bilinmiyor** | Denetim 48 saattir çalışamadı; yedeklerin durumu hakkında bir şey söylenemez. | Destek talebi açın. "Bilinmiyor" sağlıklı anlamına gelmez. |

Gösterge günlük denetime dayandığı için az önce alınmış bir yedek burada hemen görünmeyebilir.

### Yedeklerin imhası

Yedekler yalnızca iki durumda kalıcı olarak silinir:

- Planlı silmenin bekleme süresi dolduğunda,
- **Kalıcı sil** ile.

Bunların dışındaki hiçbir yol yedeklere dokunmaz. Instance hata durumuna düşüp otomatik temizlenirse, kurulum yarıda kalırsa ya da bir geri yükleme iptal edilirse yedekler saklama süresi boyunca yerinde kalır. Otomatik temizlenmiş bir instance'ın yedeğine ulaşmak için destek talebi açılması gerekir.

---

## Geri Yükleme

Geri yükleme **Yedekler** ya da **İşlemler** sekmesindeki **Geri yüklemeyi aç** ile başlatılır. İki seçenek vardır: son yedekten geri yükleme ve saklama süresi içindeki belirli bir ana geri dönüş.

### Geri dönülebilecek aralık

- **Geriye doğru:** en fazla saklama süresi kadar, ancak instance'ın ilk yedeğinden öncesine gidilemez. Yeni oluşturulmuş bir instance'ta bu aralık henüz birkaç saat olabilir.
- **İleriye doğru:** üst uç "şu an" değil, **en son arşivlenmiş değişikliktir.** Az yazma alan bir instance'ta son birkaç dakika henüz geri yüklenebilir olmayabilir.

Geçerli aralık **Restore noktaları** kartında **En eski** ve **En yeni** olarak yazar. Aralık dışında bir an seçildiğinde işlem başlamadan hata verir; **Hedefi doğrula** ile bir zaman damgası önceden denenebilir.

### Geri yükleme yeni bir instance oluşturur

Geri yükleme mevcut instance'ın üzerine yazmaz:

1. Seçilen andaki verilerle **yeni bir instance** oluşturulur.
2. Mevcut instance çalışmaya, hizmet vermeye ve **faturalanmaya devam eder** — iki instance bir süre yan yana durur ve ikisi de ücretlendirilir.
3. Uygulamanın bağlantı ayarlarını yeni instance'a çevirmek ve eskisini silmek kullanıcıya aittir.

> **Uyarı:** Geri yüklenen kopya ayrı bir instance'tır ve kendi bağlantı adresini alır — genel ve özel adresi kaynağınkinden farklıdır. Bağlantı dizesini yeni instance'ın **Genel Bakış** sekmesinden alın; kaynağa bağlı kalan servisler, ayarları elle değiştirilene kadar kaynağa bağlanmaya devam eder.

Sürmekte olan bir geri yükleme **İşlemler** sekmesindeki **Restore'u iptal et** ile durdurulabilir. Yarı kurulmuş kopya kaldırılır; kaynak instance ve yedekleri etkilenmez.

---

## Plan Değişikliği

Plan, instance başlığındaki **Plan değiştir** ile değiştirilir. Hedef plan seçildiğinde etki önizlemesi kendiliğinden çalışır: değişikliğin türü, tahmini kesinti, kaynak değişiklikleri ve varsa uyarılar görünür. Önizleme uygulanamaz bir değişiklik bildirdiğinde **Planı uygula** kapalı kalır.

- **Daha düşük bir plana geçilemez.** Düşük planlar listede görünse de değişiklik uygulanmaz.
- **Disk küçültülemez.** Bu bir platform kısıtı değil, blok depolamanın kendi kuralıdır.
- **Kapasite düğüm bazında ölçülür.** Diskin tek bir fiziksel düğümde büyümesi gerekir; kümenin toplam boş alanı yeterli görünse bile o düğümde yer yoksa değişiklik yapılamaz.
- **Önizleme bir garanti değildir.** Kapasite kontrolü onay anında yeniden çalışır; önizleme açıldıktan sonra beklenirse aynı değişiklik reddedilebilir.

Planlar tek sunuculu olduğu için plan değişikliği sırasında sunucu yeniden başlar ve kısa bir kesinti olur; tahmini süre önizlemede yazar. Değişiklik yarıda kalırsa instance eski planıyla çalışmaya devam eder ve **Yükseltme Başarısız** durumuna geçer; verisi yerindedir ve aynı değişiklik yeniden denenebilir. Sorun tekrarlıyorsa destek talebi açılmalıdır.

---

## Bakım ve Yaşam Döngüsü

Yeniden başlatma, parola yenileme ve silme **İşlemler** sekmesinde toplanır.

### Yeniden başlatma

**Yeniden başlat** sunucuyu yeniden başlatır. Tek sunuculu olduğu için instance kısa bir süre bağlantı kabul etmez; uygulamanın yeniden bağlanması gerekir.

### Askıya alma yok

DocumentDB'de askıya alma ve uyku yoktur. Instance var olduğu sürece çalışır ve planının tam fiyatıyla faturalanır; yalnız diskin faturalandığı tek durum, silmenin beklendiği haftadır.

### Parola yenileme

**Veritabanı parolasını rotate et**, bağlantı kullanıcısına yeni bir parola üretir ve **açık bütün istemci bağlantılarını koparır.** Uygulama yeni parolayla yeniden bağlanana kadar hata alır.

- Yalnız çalışır durumdaki bir instance üzerinde yapılabilir.
- Eski parola işlem tamamlandığı anda geçersiz olur.
- Bir sonraki yenileme tarihi 90 gün sonrası olarak işaretlenir; bu bir zorunluluk değil, hatırlatmadır.
- Salt-okunur kullanıcıların parolaları değişmez.

İşlem bakım penceresinde yapılmalı ve yeni parola dağıtılmaya hazır olunmalıdır.

### Silme

İki yol vardır ve sonuçları farklıdır:

- **Silme planla** instance'ı hemen erişime kapatır: genel adres kapanır ve instance özel ağdan da kaldırılır. Instance 7 günlük bekleme süresinin sonunda silinir; bu sürede yalnız disk faturalanır. Süre dolmadan **Planlı silmeyi iptal et** ile geri alınabilir: silme planlanırken hizmet veren instance önceki adresiyle yeniden hizmet verir. İptal için harcama limitinin planın tam fiyatını karşılaması gerekir, yoksa iptal reddedilir. Hata durumundaki bir instance iptalden sonra da hata durumunda kalır.
- **Kalıcı sil** instance'ı ve **yedekler dahil tüm verisini** anında yok eder.

Instance kurulurken, plan değiştirilirken ya da geri yüklenirken silme planlanamaz. Instance'ı kullanan servisler varsa silme reddedilir ve hata bu servislerin adını verir; önce bu servislerden instance'ın ortam değişkenleri kaldırılmalı ya da servisler silinmelidir.

> **Uyarı:** Kalıcı silmenin geri dönüşü yoktur ve yedekler de silindiği için veri destek üzerinden de kurtarılamaz. Verinin bir süre daha durması isteniyorsa doğru seçim planlı silmedir.

---

## Uyumluluk ve Sınırlar

Aşağıdaki liste, canlı bir instance üzerinde MongoDB protokolüyle yapılan kontrollere dayanır.

### Desteklenenler

- Belge ekleme, okuma, güncelleme, silme (CRUD); toplu yazma
- Güncelleme ve sorgu operatörleri
- Toplama hattı (aggregation), `$lookup`, `$facet`, `$graphLookup` ve `$setWindowFields` dahil
- Bütün indeks türleri; TTL indeksi (süresi dolan belgeler arka planda, kısa bir gecikmeyle silinir)
- Vektör arama. Vektör indeksi MongoDB Atlas'ın arama indeksi komutlarıyla değil, açık kaynak DocumentDB projesinin belgelediği biçimle oluşturulur: indeks anahtarında `"cosmosSearch"` ve yanında `cosmosSearchOptions` (indeks türü, boyut sayısı, benzerlik ölçüsü).
- View'lar ve şema doğrulama
- Çok belgeli transaction'lar
- 17 BSON veri tipi

### Desteklenmeyenler

| Özellik | Not |
|---|---|
| Change stream (`watch()`, `$changeStream`) | Değişiklikleri anlık dinlemek mümkün değildir. Yerine indeksli bir güncellenme zamanı alanı üzerinden düzenli sorgu yapılabilir. |
| Capped collection | Sabit boyutlu koleksiyon açılamaz; eski kayıtları silmek için TTL indeksi kullanılabilir. |
| Time series collection | Zaman serisi verisi normal bir koleksiyonda, zaman alanı indekslenerek tutulabilir. |
| `$where` | Sorgu içinde JavaScript çalıştırılamaz. |
| Collation | Dile duyarlı karşılaştırma ve sıralama yoktur. Büyük-küçük harf duyarsız arama için alanın normalleştirilmiş bir kopyası tutulabilir. |
| Yönetim ve tanı komutları | `serverStatus`, `top`, `rolesInfo`, `replSetGetStatus`, `listShards`, `fsync`, `logRotate`, `dbHash`, `reIndex`, `getParameter`. Bu komutlara dayanan izleme araçları çalışmaz; instance'ın durumu **İzleme** sekmesinden izlenir. |
| Kullanıcı yönetimi | `createUser` ve benzeri komutlara yetki yoktur (bkz. [Size verilen kullanıcı](#size-verilen-kullanıcı)). |

### Sıralı cursor'lar 60 saniye sessizlikte kapanır

Sıralı sonuç döndüren bir cursor — `$sort` içeren bir aggregate ya da `sort()` ile yapılan bir find — sürücü **60 saniye boyunca bir sonraki partiyi istemezse** sunucuda kapatılır ve sonraki istek `CursorNotFound` hatası alır. `noCursorTimeout` seçeneği yok sayılır. Sıralamasız cursor'lar bundan etkilenmez.

Kısa sayfalı sorgular (ör. `sort` + `limit` ile bir API sayfası) cursor'ı açık bırakmadığı için etkilenmez. Etkilenen kalıp, sıralı bir sonucu satır satır gezip her kayıt için yavaş bir iş yapan arka plan işleridir. Bu işler sonucu sayfalayarak okumalıdır: son okunan değerden devam eden sorgular (ör. `_id` ya da indeksli bir alan üzerinde `{ _id: { $gt: sonId } }` ile `limit`).

### Yüksek eşzamanlılık

Aynı anda çok yüksek sayıda istek gönderildiğinde istekler hata alabilir; durum birkaç dakika içinde kendiliğinden düzelir. Uygulamanın bütün kopyalarından aynı anda açık kalan istek sayısı makul tutulmalı, sürücünün bağlantı havuzu (`maxPoolSize`) gereğinden büyük ayarlanmamalıdır.

### Büyük yanıtlar

Her yanıt, uygulama okuyana kadar sunucu belleğinde tutulur. Aynı anda çok sayıda büyük yanıt (ör. binlerce belgelik partiler, çok büyük belgeler) belleği doldurabilir; bu durumda açık bağlantılar kopar ve uygulama yeniden bağlanmak zorunda kalır, veri etkilenmez. Projeksiyonla yalnız gereken alanları istemek, parti boyutunu küçük tutmak ve büyük okumaları sıraya koymak bunu önler. Tekrarlıyorsa daha büyük bir plana geçilmelidir.

### Disk dolunca salt-okunur

Disk doluluğu eşiğe ulaştığında instance salt-okunur moda geçer: **10 GB'lık diskte yaklaşık %85, 40 GB'ta %87, 80 GB'ta %93, 160 GB ve üstünde %95.** Okumalar çalışmaya devam eder, yazma istekleri hata alır. Yazma ancak yer açıldığında ya da daha büyük diskli bir plana geçildiğinde geri gelir. Disk uyarıları eşikten önce tetiklenir; bu yüzden bekletilmemelidir.

---

## İzleme ve Uyarılar

- **İzleme** sekmesi seçilen zaman aralığında bağlantı sayısı, işlem hızı ve disk kullanımı gibi grafikleri gösterir.
- **Loglar** sekmesi sunucu loglarında en fazla son 24 saat içinde arama yapar; seviye, zaman aralığı ve satır sınırı ayrıca seçilir. Bu ekran uzun süreli bir log arşivi değildir; daha uzun saklama gerekiyorsa loglar kendi tarafınıza aktarılmalıdır.
- İzleme kuralları instance oluşturulurken kendiliğinden kurulur. DocumentDB için kapsanan konular: disk doluluğu, bağlantı doygunluğu, instance'a ulaşılamaması, bağlantıları karşılayan katmanın durması ya da belleğinin dolması, sunucunun yeniden başlaması, eski ya da başarısız yedekler. Tetiklenen uyarılar **Uyarılar** sekmesinde listelenir.

---

## Sık Sorulan Sorular

### Mevcut MongoDB uygulamam değişiklik olmadan çalışır mı?

Çoğu durumda değişen tek şey bağlantı dizesidir. Uygulama change stream, capped ya da time series koleksiyon, `$where` veya collation kullanıyorsa bu kısımların değişmesi gerekir; ayrıntı [Uyumluluk ve Sınırlar](#uyumluluk-ve-sınırlar) bölümündedir.

### `retryWrites=false` neden var, kaldırabilir miyim?

Kaldırılmamalıdır. Sürücünün bağlantı kopunca yeniden gönderdiği bir yazma sunucuda ikinci kez uygulanabilir. Parametre bu otomatik tekrarı kapatır; tekrar gerekiyorsa uygulama işlemin sonucunu kontrol ederek kendisi dener.

### Yüksek erişilebilirlik var mı?

Hayır. Bütün planlar tek sunuculudur; bakım ve düğüm değişimlerinde kısa kesinti olabilir. Uygulama kopan bağlantıdan sonra yeniden bağlanacak şekilde yazılmalıdır.

### Kullanmadığım zamanlarda askıya alabilir miyim?

Hayır, DocumentDB'de askıya alma ve uyku yoktur. Instance var olduğu sürece planının tam fiyatıyla faturalanır.

### Yedekler instance ile birlikte silinir mi?

**Kalıcı sil** yedekleri de yok eder ve geri alınamaz. **Silme planla** ise yedekleri 7 günlük bekleme süresi boyunca yerinde bırakır ve iptal edilebilir.
