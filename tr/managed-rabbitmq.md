# RabbitMQ

RabbitMQ, Komuta'nın kurup işlettiği bir mesaj kuyruğu servisidir. Sunucu kurulumu, TLS, yönetim arayüzü, günlük yedek ve izleme kuralları instance oluşturulurken hazır gelir; uygulamaya düşen tek iş verilen bağlantı adresini kullanmaktır.

---

## RabbitMQ Nedir?

RabbitMQ, iki uygulama parçasının birbirini beklemeden haberleşmesini sağlayan bir aracıdır (broker). Yayıncı mesajı brokera bırakır ve işine döner; mesaj kuyrukta bekler; tüketici hazır olduğunda alıp işler. Böylece yayıncının hızı tüketicinin hızına bağlı kalmaz ve tüketici bir süre ayakta olmasa bile mesaj kaybolmaz.

### Mesajın izlediği yol

Mesaj doğrudan kuyruğa değil, bir **exchange**'e yayınlanır. Exchange, mesajı hangi kuyruğa (ya da kuyruklara) koyacağını **bağlama** (binding) kurallarına ve mesajın yönlendirme anahtarına bakarak belirler. Kuyruk mesajları sırayla tutar, tüketiciler kuyruktan okur ve işlediği mesajı onaylar; onaylanmayan mesaj tüketici düştüğünde kuyruğa geri döner.

Bu ayrım, aynı mesajın tek bir tüketiciye iş olarak dağıtılmasını da birden fazla kuyruğa aynı anda kopyalanmasını da mümkün kılar — yayıncının kodu iki durumda da aynıdır, değişen tek şey bağlama kuralıdır.

### Ne için kullanılır

- **Arka plan işleri.** Video dönüştürme, rapor üretme, e-posta gönderme gibi uzun süren işler isteğin içinde değil kuyrukta işlenir; kullanıcı yanıtı beklemez.
- **Servisleri birbirinden ayırmak.** Bir servis olay yayınlar, onu kimin dinlediğini bilmez. Yeni bir tüketici eklemek yayıncı tarafında değişiklik gerektirmez.
- **Yük dengeleme.** Aynı kuyruğu birden fazla tüketici okuduğunda işler aralarında paylaşılır; tüketici sayısını artırmak işlem kapasitesini artırır.
- **Yük tepelerini düzleştirmek.** Ani gelen istek yığını kuyrukta birikir ve arkadaki sistem kendi hızında tüketir; veritabanı ya da dış API istek sağanağıyla karşılaşmaz.
- **Yeniden deneme.** Başarısız işler onaylanmadığı için kuyrukta kalır; tüketici yeniden ayağa kalktığında iş kaybolmamış olur.

RabbitMQ bir veri deposu değil, bir aktarım katmanıdır. Bu ayrım yedeklemede somutlaşır: yedek yalnız yapılandırmayı kapsar, kuyruklardaki mesajları kapsamaz (bkz. [Yedekleme](#yedekleme)).

---

## Nasıl Çalışır?

Bir instance oluşturulduğunda Komuta seçilen bölgede bir RabbitMQ kümesi kurar, TLS'i açar, uygulama ve yönetim kimlik bilgilerini üretir, yönetim arayüzünü yayınlar, günlük yedeği ve izleme kurallarını yerleştirir. Bu adımların hiçbiri ayrıca yapılandırma gerektirmez.

### Instance, plan ve bölge

Bir **instance**, kendi kimlik bilgisi ve yedek zinciriyle çalışan bağımsız bir RabbitMQ brokerıdır. CPU, bellek, disk ve düğüm sayısı bağlı olduğu **plandan** gelir; plandaki bellek aynı zamanda brokerın yayıncıları yavaşlatmaya başladığı eşiği belirler (bkz. [Broker Sınırları](#broker-snrlar)).

**Bölge**, instance'ın fiziksel olarak çalıştığı yerdir ve oluşturulduktan sonra değiştirilemez. Başka bir bölgeye taşımanın yolu yeni bir instance kurmaktır.

**Sürüm**, instance'ın çalıştıracağı RabbitMQ sürümüdür ve seçilen planın desteklediği sürümler arasından belirlenir. Sürüm sonradan değiştirilemez.

**Örnek adı** küçük harfle başlar, yalnız küçük harf, rakam ve tire içerir, en fazla 40 karakterdir; küme içindeki kaynak adı olarak kullanılır. Instance isteğe bağlı olarak bir **projeye** bağlanabilir, bağlanmadığında paylaşımlı sayılır.

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

---

## Bağlanma

### Bağlantı adresi ve TLS

Her instance için bir hostname verilir ve bağlantılarda bu ad **olduğu gibi** kullanılmalıdır. Adresin IP'ye çevrilmesi ya da bağlantı havuzunda bir IP'ye sabitlenmesi durumunda bağlantı kurulamaz.

Bağlantılar TLS ile kurulur: şema `amqps`, port `5671`'dir.

### Sanal sunucu

Kuyruklar, exchange'ler ve bağlamalar bir **sanal sunucu** (vhost) içinde yaşar; instance ile birlikte gelen sanal sunucu `/`'tır. Bağlantı adresi arayüzden kopyalandığında bu kısım doğru gelir, ama adres elle yazılıyorsa `/` karakterinin URI içinde `%2F` olarak kodlanması gerekir:

```text
amqps://app:<parola>@<host>:5671/%2F
```

> **Not:** Kodlamanın atlanması RabbitMQ'da en sık yapılan bağlantı hatasıdır. Kimlik bilgisi doğru olsa bile bağlantı yetkisiz sanal sunucu hatasıyla reddedilir.

### Kullanıcılar ve yetkileri

Her instance iki kullanıcıyla gelir ve ikisinin işi ayrıdır:

| Kullanıcı | Ne için | Yetkileri |
|---|---|---|
| **app** | Uygulama bağlantıları | `/` sanal sunucusunda tam okuma, yazma ve tanımlama yetkisi. Yönetim arayüzüne giriş yapamaz. |
| **admin** | Yönetim arayüzü ve operasyon | Tam yetki; yönetim arayüzüne giriş yapabilir. |

Uygulamalar `app` kullanıcısıyla bağlanır. `admin` hesabının uygulama yapılandırmasına konması, kuyruk ve politika silmeye yetkili bir kimlik bilgisini uygulama sunucularına dağıtmak anlamına gelir; yönetim arayüzü dışında kullanılmaması gerekir.

### Bağlantı bilgisinin görüntülenmesi

Parolalar platform tarafında şifrelenmiş saklanır ve yalnız bağlantı bilgisi görüntülendiğinde çözülür. **Her görüntüleme; instance, kullanıcı ve zaman bilgisiyle denetim kaydına yazılır.** Kimlik bilgisi paylaşmak yerine ekip üyelerine kendi hesaplarından erişim verilmesi önerilir.

RabbitMQ'da kimlik bilgisi yenileme (rotate) işlemi bulunmaz; parolanın değişmesi gerekiyorsa destek talebi açılmalıdır.

![Genel Bakış sekmesi — bağlantı bilgisi ve yönetim arayüzü bölümü](https://cdn.komuta.io/docs/tr/images/rabbitmq/dashboard.png)

---

## Yönetim Arayüzü

RabbitMQ'nun kendi yönetim arayüzü her instance ile birlikte yayınlanır ve veri bağlantısından **farklı bir adreste** durur: bağlantı adresinin ilk bölümünden sonra `.dash` eklenir. Veri adresi `rmq-a1b2c3d4.example.com` ise yönetim arayüzü şu adrestedir:

```text
https://rmq-a1b2c3d4.dash.example.com
```

Giriş `admin` kullanıcısıyla yapılır. Arayüz üzerinden kuyruklar, bağlantılar ve tüketiciler izlenir, kuyruk ve exchange tanımları oluşturulur, mevcut tanımlar dosya olarak dışa aktarılır.

Tanımların dışa aktarılması yedeklemenin bir parçası değildir, ama yapılandırmanın kendi elinizde bir kopyasının durması geri yükleme gerektiğinde destek beklemeden yeniden kurmayı mümkün kılar (bkz. [Yedekleme](#yedekleme)).

---

## Kuyruk Dayanıklılığı

Çok düğümlü bir plan seçmek kuyrukları kendiliğinden dayanıklı yapmaz. Bir kuyruk **quorum** tipinde tanımlanmadıysa yalnızca tek bir düğümde yaşar; o düğüm kaybedildiğinde kuyruk ve içindeki mesajlar da kaybolur. Plan yüksek erişilebilirlik sağlar, kuyruk tipini ise uygulama seçer.

### Quorum kuyruklar

Kalıcı olması gereken her kuyruk `x-queue-type=quorum` argümanıyla tanımlanır. Quorum kuyruk mesajlarını birden fazla düğümde çoğaltır; düğüm kaybında kuyruk ayakta kalır.

- Quorum kuyruklar `durable` olmak zorundadır; `exclusive` ya da `auto-delete` olamazlar.
- Mevcut bir kuyruğun tipi sonradan değiştirilemez. Klasik bir kuyruğu quorum'a taşımanın yolu yeni bir kuyruk oluşturup tüketicileri ona geçirmektir.
- Kısa ömürlü yanıt (RPC reply) kuyrukları klasik bırakılır; onların kalıcı olması beklenmez.

### Varsayılan tip neden değiştirilmiyor

Broker genelinde varsayılan kuyruk tipi quorum yapılmaz, çünkü bu ayar geçici (`exclusive` / `auto-delete`) kuyruk açan istemcileri kırar — MassTransit ve Celery gibi kütüphanelerin yanıt kuyrukları buna örnektir. Dayanıklılık bu yüzden broker genelinde değil, kuyruk bazında ve uygulama tarafından seçilir.

---

## Broker Sınırları

Broker kendini korumak için bellek ve disk eşikleri uygular. Eşiğe varıldığında broker çökmek yerine yayıncıyı bekletir; yani yayınlama işleminin bloklanması normal bir çalışma durumudur ve uygulamanın zaman aşımı ile yeniden deneme davranışının buna göre kurulması gerekir.

| Sınır | Değer | Aşıldığında |
|---|---|---|
| **Bellek eşiği** | Plan belleğinin %80'i | Broker yayıncıları yavaşlatır (flow control); tüketim hızlanana kadar yayınlama durur. |
| **Boş disk eşiği** | Bellek boyutunun 1,5 katı | Disk alarmı devreye girer ve yayınlama engellenir. |
| **Eşzamanlı bağlantı** | 5.000 | Yeni bağlantılar reddedilir. |

Bellek eşiğine varmanın olağan sebebi kuyrukların birikmesidir: tüketiciler yayınlama hızının gerisinde kaldığında bekleyen mesajlar bellekte yer tutar. Bu yüzden kuyruk birikmesi ve tüketicisi olmayan kuyruk uyarıları, bellek uyarısından önce gelen erken işaretlerdir.

Çok düğümlü kurulumlarda ağ bölünmesi yaşanırsa azınlıkta kalan düğümler kendini duraklatır (`pause_minority`); çoğunluk tarafı hizmet vermeye devam eder. Duraklamış düğüme bağlı istemciler bağlantı hatası alır ve yeniden bağlanmaları gerekir.

---

## Ağ Erişimi

RabbitMQ'da uygulama trafiği de dahil olmak üzere tüm bağlantılar genel erişim adresi üzerinden kurulur; erişimi daraltmanın yolu IP listesidir. İki mod kullanılabilir.

### Herkese açık

Genel bağlantı adresi internetten erişilebilir ve kaynak IP kısıtı uygulanmaz; kimlik bilgisine sahip herkes bağlanabilir. Yeni bir instance bu modla başlar. Mod değişikliği anında geçerli olur, instance yeniden başlatılmaz ve mevcut bağlantılar kesilmez.

### IP listesi ile kısıtlı

Genel adres açık kalır, ama yalnız listelenen kaynaklardan gelen bağlantılar kabul edilir; listede olmayan her kaynak ağ geçidinde reddedilir — bağlantı brokera hiç ulaşmaz. Uygulama sunucularının çıkış adreslerini listeye eklemek, diğer tüm kaynakları kapatmanın yoludur.

Liste kuralları:

- Her satıra bir IP ya da CIDR yazılır.
- Tek bir adres yazıldığında (`203.0.113.4`) otomatik olarak tek makineye (`/32`) genişletilir.
- Geçersiz bir ifade sessizce yok sayılmaz, hata verir.
- Liste boş bırakılamaz. Boş liste herkesi reddedeceği için istek geri çevrilir.

Bu mod her kümede kullanılamaz: bazı kümelerde ağ geçidi gerçek kaynak adresi göremediği için liste hiçbir şeyi kısıtlamaz — herkesi kabul eden bir "kısıtlama" bırakmamak için mod orada kapalı tutulur. İstek reddedilir, yarı uygulanmış bir kısıtlama bırakılmaz.

> **Not:** PostgreSQL ve Valkey'de bulunan yalnızca özel ağ modu RabbitMQ'da yoktur. RabbitMQ'nun özel adresi yalnız broker kümesinin içinde çözülür, uygulamaların çalıştığı yerde çözülmez; genel yolun kapatılması instance'ı hiçbir yerden erişilemez hâle getirirdi.

![Ağ erişimi kartı — IP listesi modu seçili](https://cdn.komuta.io/docs/tr/images/rabbitmq/network-access.png)

---

## Yedekleme

Günde bir kez yedek alınır ve nesne depolamasında tutulur. Yedek yalnız **yapılandırmayı** içerir: exchange'ler, kuyruk tanımları, bağlamalar, kullanıcılar ve politikalar.

> **Uyarı:** Kuyruklardaki mesajlar yedeğe dahil değildir ve bir felaket durumunda geri getirilemez. Yedek, brokerın yeniden kurulmasını sağlar; içindeki mesajların geri gelmesini sağlamaz.

### Mesajları kaybetmemek için

Mesaj dayanıklılığı yedeğin değil brokerın ve uygulamanın işidir. Kaybedilmemesi gereken mesajlar için dört şey birlikte gerekir:

- Kuyruklar quorum tipinde tanımlanır (bkz. [Kuyruk Dayanıklılığı](#kuyruk-dayankll)).
- Mesajlar `persistent` olarak yayınlanır.
- Yayıncı onayı (publisher confirm) kullanılır; onay gelmeyen mesaj yayınlanmış sayılmaz.
- İşlenmesi kritik olan mesajların kaydı uygulamanın kendi veritabanında da tutulur.

### Zamanlama ve saklama süresi

- Yedeğin saati beş alanlı bir cron ifadesiyle verilir — `0 3 * * *` her gün UTC 03:00 demektir.
- Saklama süresi gün olarak yazılır ve 1 ile 365 gün arasında olabilir; varsayılan 7 gündür.
- Planlı yedeğin dışında elle yedek alınabilir. Bu, günlük yedeğin yerine geçmez; ona ek bir kopya üretir. Kuyruk ve politika tanımlarında kapsamlı bir değişiklikten hemen önce bir koruma noktası isteniyorsa doğru yol budur.

Yedek saatini değiştirmek saklama süresini etkilemez; saklama süresi yalnız açıkça değiştirildiğinde güncellenir. Son yedeğin 26 saatten eski olması ayrıca işaretlenir — bu, günlük zamanlamanın geciktiği anlamına gelir.

> **Uyarı:** Saklama süresini kısaltmak geri alınamaz. Süre düşürüldüğünde yeni pencerenin dışında kalan yedekler bir sonraki bakımda kalıcı olarak silinir. Süreyi tekrar uzatmak silinenleri geri getirmez; koruma penceresinin yeniden dolması o kadar gün alır.

### Yedeklerin imhası ve geri yükleme

Yedekler yalnızca iki durumda kalıcı olarak silinir:

- Planlı silme işleminin geri alma süresi dolduğunda,
- Bir yöneticinin açık kalıcı silme talebiyle.

Bunların dışındaki hiçbir yol yedeklere dokunmaz. Instance hata durumuna düşüp otomatik temizlenirse ya da kurulum yarıda kalırsa yedekler saklama süresi boyunca yerinde kalır.

> **Not:** Yedeğin durması, geri yüklemenin arayüzden yapılabileceği anlamına gelmez. Tanımların geri yüklenmesi için destek talebi açılması gerekir. Yönetim arayüzünden dışa aktarılan tanım dosyası, bu bekleme olmadan yeniden kurmayı mümkün kılan kopyadır.

![Yedekler sekmesi — zamanlama ve saklama süresi](https://cdn.komuta.io/docs/tr/images/rabbitmq/backup-page.png)

---

## Plan Değişikliği

Plan değişikliği CPU, bellek, disk ve düğüm sayısını hedef planın değerlerine taşır. Bellek değiştiği için brokerın yayıncıları yavaşlattığı eşik de aynı oranda değişir (bkz. [Broker Sınırları](#broker-snrlar)).

Hedef plan seçildiğinde bir etki önizlemesi hesaplanır: değişikliğin türü (yükseltme, düşürme, topoloji değişikliği), tahmini kesinti, kaynak değişiklikleri ve varsa uyarılar. Önizleme uygulanamaz bir değişiklik bildirdiğinde işlem başlatılamaz.

### Kısıtlar

- **Disk küçültülemez.** Bu bir platform kısıtı değil, blok depolamanın kendi kuralıdır.
- **Düşük plana geçiş mümkündür.** Karşılığı, bellek eşiğinin düşmesi ve brokerın daha erken flow control'e girmesidir.
- **Kapasite düğüm bazında ölçülür.** Bir düğümün diski tek bir fiziksel makinede büyümek zorundadır; kümenin toplam boş alanı yeterli görünse bile o makinede yer yoksa değişiklik yapılamaz.
- **Önizleme bir garanti değildir.** Kapasite kontrolü onay anında yeniden çalışır; önizleme hesaplandıktan sonra beklenirse küme dolmuş olabilir ve aynı değişiklik reddedilebilir.

### Kesinti ve yüksek erişilebilirlik

Kaynak (CPU/RAM/disk) değişikliğinde düğümler sırayla yeniden başlatılır; beklenen kesinti yaklaşık 30 saniye – 2 dakikadır ve çok düğümlü kurulumlarda en az bir düğüm ayakta kalır. Düğüm sayısını değiştiren bir geçişte süre 2–5 dakikaya çıkar ve kısa bağlantı kesintileri olur.

Yüksek erişilebilirlikli planlarda düğümler farklı fiziksel makinelere zorunlu olarak dağıtılır; tek bir makinenin kaybı hizmeti durdurmaz. Bu korumanın kuyruklara yansıması için kuyrukların quorum tipinde olması gerekir. Tek düğümlü planlarda böyle bir koruma yoktur ve bakım ile düğüm değişimlerinde kısa kesinti yaşanabilir.

Her iki durumda da uygulamanın bağlantı kopmasına dayanıklı olması gerekir: yeniden bağlanma, kanalları yeniden açma ve onaylanmamış mesajları yeniden yayınlama.

### Yükseltme başarısız olursa

Plan değişikliği yarıda kalırsa instance eski planıyla çalışmaya devam eder ve **Yükseltme Başarısız** durumuna geçer. Bu durumdaki bir instance canlıdır, verisi yerindedir ve aynı değişiklik yeniden denenebilir. Sorun tekrarlıyorsa destek talebi açılmalıdır.

![Plan değiştir paneli — etki önizlemesi](https://cdn.komuta.io/docs/tr/images/rabbitmq/change-plan.png)

---

## Bakım ve Yaşam Döngüsü

### Yeniden başlatma

Düğümler sırayla yeniden başlatılır; çok düğümlü planlarda broker hizmet vermeye devam eder, ama yeniden başlatılan düğüme bağlı istemciler düşer ve yeniden bağlanır. Tek düğümlü bir planda instance kısa süre bağlantı kabul etmez.

### Askıya alma ve devam ettirme

İki askıya alma biçimi vardır ve aralarındaki fark geri dönüş süresidir:

- **Soft askıya alma** istemci bağlantılarını boşaltır, düğümleri çalışır tutar. Geri açma saniyeler sürer.
- **Hard askıya alma** düğümleri tamamen durdurur. Geri açma, küme yeniden zamanlama yaptığı için birkaç dakika sürer.

Her iki durumda da askıdaki instance bağlantı kabul etmez; uygulama bağlantı hatası alır. Yayıncılar bu süre boyunca mesaj bırakamaz — askıya almanın kuyruğu boşaltmadığını, yalnız erişimi kapattığını hesaba katmak gerekir.

### Silme

İki yol vardır ve sonuçları farklıdır:

- **Silmeyi planlamak** instance'ı 7 günlük bir bekleme süresinin sonunda silinmek üzere işaretler. Süre dolmadan iptal edilebilir; instance bu süre boyunca çalışmaya ve faturalanmaya devam eder.
- **Kalıcı silme** instance'ı, kuyruklarındaki mesajları ve **yedekler dahil tüm yapılandırmasını** anında yok eder.

> **Uyarı:** Kalıcı silmenin geri dönüşü yoktur ve yedekler de silindiği için tanımlar destek üzerinden de kurtarılamaz. Yapılandırmanın bir süre daha durması isteniyorsa doğru seçim planlı silmedir.

---

## Metrikler ve Uyarılar

İzleme kuralları instance oluşturulurken otomatik kurulur; ayrıca yapılandırma gerekmez. RabbitMQ'da kapsanan başlıklar: bellek kullanımı, kuyruk birikmesi, tüketicisi olmayan kuyruklar, disk alarmı, düğüm kaybı ve bağlantı sayısı. Tetiklenen uyarılar uyarı geçmişinde listelenir.

Grafiklerde kararı değiştiren ölçüler şunlardır:

| Ölçü | Ne anlatır |
|---|---|
| **Kuyruk derinliği** | Bekleyen mesaj sayısı. Sürekli artması, tüketim kapasitesinin yayınlama hızının gerisinde kaldığı anlamına gelir; bellek eşiğine giden yol da budur. |
| **Yayınlama ve iletim hızı** | Saniyedeki yayınlanan ve tüketiciye iletilen mesaj sayısı. İletim hızının yayınlama hızının altında kalması birikmenin sebebidir. |
| **Bağlantı ve kanal sayısı** | Açık istemci sayısı. Beklenmedik biçimde artması, bağlantıyı kapatmayan ya da her istekte yeni bağlantı açan bir istemciye işaret eder ve bağlantı sınırına götürür. |
| **Bellek kullanımı** | Bellek eşiğine ne kadar yaklaşıldığı. Eşiğe varıldığında yayınlama durur. |

![İzleme sekmesi — kuyruk derinliği ve yayınlama grafikleri](https://cdn.komuta.io/docs/tr/images/rabbitmq/monitoring.png)

---

## Sık Sorulan Sorular

### Çok düğümlü plan aldım, kuyruklarım dayanıklı mı?

Kuyruk quorum tipinde tanımlanmadıysa değil. Klasik kuyruk tek bir düğümde yaşar ve o düğüm kaybedildiğinde kuyruk da mesajları da kaybolur. Plan yüksek erişilebilirlik sağlar, kuyruk tipini uygulama seçer.

### Yedekten mesajlarımı geri alabilir miyim?

Alınamaz. Yedek yalnız exchange, kuyruk tanımı, bağlama, kullanıcı ve politikaları kapsar; kuyruklardaki mesajlar yedeğe girmez.

### Yayınlama neden aniden yavaşladı?

Bellek eşiğine (plan belleğinin %80'i) ya da boş disk eşiğine varıldığında broker yayıncıları bekletir. Bu bir hata değil, brokerın kendini koruma davranışıdır; tüketim hızlanıp kuyruklar boşaldığında yayınlama kendiliğinden normale döner.

### Bağlantı yetkisiz sanal sunucu hatası veriyor, kimlik bilgim yanlış mı?

Genellikle değil. Sanal sunucu `/`'tır ve bu karakterin bağlantı adresinde `%2F` olarak kodlanması gerekir. Adres arayüzden kopyalandığında doğru gelir; elle yazılan adreslerde en sık yapılan hata budur.

### Uygulamamı admin kullanıcısıyla bağlayabilir miyim?

Bağlanır, ama bağlanmaması gerekir. `admin`, yönetim arayüzü ve operasyon için tam yetkili hesaptır; uygulama yapılandırmasına konması kuyruk ve politika silme yetkisini uygulama sunucularına dağıtmak olur. Uygulamalar `app` kullanıcısıyla bağlanır.

### Instance silinince yedekler de gider mi?

**Kalıcı silme** yedekleri de yok eder ve geri dönüşü yoktur. **Silmeyi planlamak** ise 7 günlük bekleme süresi boyunca hem instance'ı hem yedekleri yerinde bırakır ve iptal edilebilir.
