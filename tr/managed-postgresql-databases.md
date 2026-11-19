# PostgreSQL'de Birden Çok Veritabanı

Bir PostgreSQL instance'ının içinde, instance oluşturulurken açılan ana veritabanının yanında başka veritabanları da açılabilir. Her ek veritabanının kendi kullanıcısı ve parolası vardır ve bu kullanıcı yalnız kendi veritabanına girebilir. Ek veritabanları instance'ın planını, yedeklerini ve bağlantılarını paylaşır; birbirinden ayrı tutulması gereken uygulamalara, her biri için yeni bir instance açmadan ayrı bir veritabanı verilebilir.

> **Not:** Ek veritabanları her organizasyona açıktır. **Veritabanları** kartı "Organizasyonunuz için yeni veritabanı oluşturma açık değil. Var olanlar çalışmaya devam eder." diyorsa organizasyonunuzda yeni veritabanı açma kapatılmıştır: açılmış veritabanları çalışmaya devam eder, bilgi almak için [destek talebi](support-tickets.md) açabilirsiniz.

---

## Nasıl Çalışır?

Her ek veritabanı aynı adı taşıyan bir kullanıcıyla birlikte açılır ve veritabanının sahibi bu kullanıcıdır: `raporlar` adıyla açılan veritabanının kullanıcı adı da `raporlar` olur. Parolayı Komuta üretir.

Veritabanları birbirine kapalıdır:

- Ek bir veritabanının kullanıcısı yalnız kendi veritabanına bağlanabilir; ana veritabanına da öteki ek veritabanlarına da bağlanamaz.
- Ana veritabanının kullanıcısı ek veritabanlarına bağlanamaz.

Ek veritabanının kullanıcısı kendi veritabanının sahibidir: şema, tablo, index, view, fonksiyon ve trigger oluşturabilir, her türlü veri işlemini yapabilir ve hazır kurulu eklentileri kullanabilir. Yeni rol ya da veritabanı oluşturamaz ve sunucunun yönetici hesabı değildir. [Hazır kurulu eklentiler](managed-postgresql.md#hazır-kurulu-eklentiler) ek veritabanlarında da kurulu gelir.

### Ayrı olanlar, ortak olanlar

- **Her veritabanının kendine ait:** verisi, kullanıcı adı ve parolası, salt-okunur kullanıcıları ve **İçgörü** sekmesindeki sorgu istatistikleri.
- **Instance'ta ortak:** plan (CPU, bellek ve disk), bağlantı adresi ve port, ağ erişimi (erişim modu ve IP izin listesi), sunucu bağlantıları, yedekler ve geri yükleme, sorgu sınırları, izleme ve uyarılar.

Disk ortak olduğu için disk doluluğu eşiğe ulaştığında bütün veritabanları birlikte salt-okunur moda geçer (bkz. [Disk düzeni ve dolma durumu](managed-postgresql.md#disk-düzeni-ve-dolma-durumu)).

Ek veritabanları ayrıca ücretlendirilmez; instance'ın planı ve ücreti değişmez.

### Kaç veritabanı açılabilir?

Bir instance ana veritabanı dahil en fazla **5** veritabanı taşır. **Veritabanları** kartının başlığındaki sayaç (ör. `2 / 5`) açık veritabanı sayısını ve sınırı gösterir. Sınır plandan hesaplanır ve bugünkü planların hepsinde 5'tir. Sınıra ulaşıldığında yeni veritabanı açmak için önce birini silmek gerekir.

### Bağlantıların paylaşımı

Uygulamalar veritabanına bir bağlantı havuzu üzerinden bağlanır (bkz. [Bağlantı ve bellek sınırları](managed-postgresql.md#bağlantı-ve-bellek-sınırları)). Tek veritabanı varken havuz bölünmez ve ana veritabanı havuzun tamamını kullanır. İkinci veritabanı açıldığında Komuta havuzun sunucu bağlantılarını veritabanları arasında eşit böler ve her veritabanına aynı üst sınırı koyar. Yoğun bir veritabanı böylece ötekilerin bağlantılarını tüketemez: sınırına ulaşan veritabanının yeni işlemleri, kendi bağlantılarından biri boşalana kadar havuzda bekler.

Her veritabanının payı veritabanı açıldıkça küçülür, silindikçe büyür. Kart her veritabanının şu anki payını, veritabanı açma penceresi de şu anki payı ve yeni veritabanıyla birlikte payın kaça düşeceğini gösterir. Tek sunuculu planlarda bir veritabanının aynı anda kullanabileceği en fazla sunucu bağlantısı:

| Plan RAM | 1 veritabanı | 2 veritabanı | 3 veritabanı | 4 veritabanı | 5 veritabanı |
|---|---|---|---|---|---|
| 512 MB | 20 | 18 | 12 | 9 | 7 |
| 1 GB | 30 | 23 | 15 | 11 | 9 |
| 2 GB ve üstü | 55–86 | 43 | 28 | 21 | 17 |

2 GB ve üstünde tek veritabanının payı planın vCPU sayısına göre 55 ile 86 arasındadır. Bu sayılar [Bağlantı ve bellek sınırları](managed-postgresql.md#bağlantı-ve-bellek-sınırları) tablosundaki veritabanı bağlantısı sınırından küçüktür, çünkü o sınırın bir kısmı yönetici hesabına ve platformun kendi bağlantılarına ayrılır. Yüksek erişilebilirlikli planlarda havuz iki sunucuya bölündüğü için değerler farklıdır (iki ve daha fazla veritabanında en fazla iki bağlantı daha düşük); kesin sayıyı kart ve veritabanı açma penceresi gösterir.

Uygulamaların açabileceği istemci bağlantısı sınırı bölünmez; bütün veritabanlarının istemcileri bu sınırı birlikte kullanır. Plan değiştiğinde paylar yeni planın bağlantılarına göre yeniden hesaplanır.

> **Uyarı:** İlk ek veritabanı açıldığı anda ana veritabanının payı da küçülür; örneğin 2 GB'lık tek sunuculu bir planda 55–86'dan 43 sunucu bağlantısına iner. Veritabanı açma penceresi açmadan önce iki sayıyı da gösterir. Ana veritabanı aynı anda çok sayıda işlem çalıştırıyorsa ilk ek veritabanını açmadan önce bunu hesaba katın.

---

## Veritabanı Açma

1. Instance'ın **Genel Bakış** sekmesindeki **Veritabanları** kartında **Veritabanı oluştur** deyin.
2. **Ad** alanına veritabanının adını yazın. Aynı ad veritabanının kullanıcısına da verilir ve sonradan değiştirilemez.
3. Pencerenin altındaki not, bu veritabanıyla birlikte her veritabanının aynı anda en fazla kaç bağlantı kullanabileceğini gösterir. **Oluştur** deyin.

Veritabanı açılınca bağlantı bilgileri bir pencerede gösterilir (bkz. [Bağlanma](#bağlanma)).

Veritabanı açmak için:

- Instance **Aktif** olmalıdır. Değilse kart "Yeni veritabanı, servis çalışır durumdayken oluşturulabilir." der.
- Instance'ta bir dump içe aktarma sürmemelidir.
- Instance üzerinde **Düzenleme** izniniz olmalıdır. Listeyi ve veritabanlarının boyutunu **Okuma** izni olan herkes görür; bağlantı bilgisini görmek, parolayı sıfırlamak ve silmek de **Düzenleme** ister (bkz. [Organizasyonlar ve Ekip Üyeleri](organizations-team.md)).

### Ad kuralları

- 2–31 karakterdir; yalnız küçük harf, rakam ve alt çizgi (`_`) içerir.
- İlk karakter harf, son karakter harf ya da rakam olmalıdır.
- Instance'ta aynı adla bir veritabanı ya da kullanıcı varsa ad kullanılamaz; ana veritabanının adı ve kullanıcı adı da buna dahildir. Listedeki bir veritabanının adını yazarsanız pencere "Bu serviste bu adla bir veritabanı zaten var." der ve **Oluştur** açılmaz. Ad bir kullanıcıya aitse (ör. ana veritabanının kullanıcısı) **Oluştur**'dan sonra "Bu instance'ta bu adla bir veritabanı ya da kullanıcı zaten var. Başka bir ad seçin." çıkar.
- PostgreSQL'in ve platformun kendi adları kullanılamaz: `postgres`, `template0`, `template1`, `app`, `public`, `all`, `user`, `none`, `replication`, `samerole`, `samegroup`, `sameuser`, `current_user`, `current_role`, `session_user`, `streaming_replica`; `pg_`, `cnpg_`, `komuta_` ve `documentdb` gibi platform önekleriyle başlayan adlar da. Pencere bu durumda "Bu ad platforma ayrılmış. Başka bir ad seçin." der.

Bu kurallar ana veritabanının ad kuralından dardır (bkz. [Ad, Sürüm ve Veritabanı](managed-postgresql.md#2-ad-sürüm-ve-veritabanı)).

---

## Bağlanma

Veritabanı açıldığında ya da satırındaki **Bağlantı** düğmesine basıldığında host, port, veritabanı adı, kullanıcı adı ve parola ile hazır bağlantı dizeleri gösterilir. Host ve port ana veritabanınınkiyle aynıdır; veritabanı adı ve kullanıcı adı ek veritabanının adıdır. Instance'ın özel ağ yolu varsa bağlantı dizeleri arasında özel ağ adresi de yer alır.

Ek veritabanının parolası saklanır; bağlantı bilgileri listeden istendiği zaman yeniden görülebilir. Bağlantı adresi ve şifreleme kuralları ana veritabanıyla aynıdır (bkz. [Bağlantı adresi](managed-postgresql.md#bağlantı-adresi)).

---

## Parolayı Sıfırlama

Veritabanının satırındaki **Parolayı sıfırla**, kullanıcıya yeni bir parola üretir ve yeni bağlantı bilgilerini gösterir.

- Eski parolayla bağlanan uygulamalar yeni parolayı alana kadar yeni bağlantı açamaz. Açık bağlantılar hemen kopmaz; yine de uygulamaları hemen yeni parolaya geçirin.
- Bir Stack'in servisleri bu veritabanını kullanıyorsa sıfırlama reddedilir (bkz. [Stack'ten bağlanma](#stackten-bağlanma)).
- Instance'ta dump içe aktarma sürerken sıfırlama yapılamaz.

Ana veritabanının **Veritabanı parolasını rotate et** işlemi ek veritabanlarının parolalarını değiştirmez; her ek veritabanının parolası kendi satırından sıfırlanır. Ancak bu işlem bağlantı havuzunu yeniden başlattığı için ek veritabanlarının açık bağlantıları da kopar; bu veritabanlarına bağlanan uygulamalar parolaları değişmediği için kendi parolalarıyla yeniden bağlanabilir (bkz. [Parola yenileme](managed-postgresql.md#parola-yenileme)). Bir Stack servisi instance'ın herhangi bir veritabanına bağlıyken ana parolanın yenilenmesi de reddedilir.

---

## Silme

Veritabanının satırındaki **Sil**'e basın, açılan pencereye veritabanının adını yazın ve **Veritabanını sil** deyin. Veritabanı, verisi, kullanıcısı ve salt-okunur kullanıcıları silinir; bu veritabanına bağlı her istemcinin bağlantısı hemen kesilir. **İçgörü** sekmesinde bu veritabanı için toplanan sorgu istatistikleri de silinir ve veritabanının bağlantı payı kalan veritabanlarına dağıtılır.

Silmek için:

- Instance'ın son 26 saat içinde tamamlanmış bir yedeği olmalıdır. Silinen veritabanını geri getirmenin tek yolu, silmeden önce alınmış bir yedekten geri yüklemektir. Yedek yoksa "Son bir günde tamamlanmış bir yedek yok; bu veritabanı silindikten sonra geri getirilemez. Yedek alıp yeniden deneyin." mesajı çıkar: **İşlemler** sekmesindeki **Şimdi yedekle** ile bir yedek alın, yedek tamamlanınca yeniden deneyin.
- Bir Stack'in servisleri bu veritabanını kullanıyor olmamalıdır (bkz. [Stack'ten bağlanma](#stackten-bağlanma)).
- Instance **Aktif**, **Askıda** ya da **Yükseltme Başarısız** durumunda olmalıdır. Plan değişikliği, geri yükleme ya da kurulum sürerken ve dump içe aktarma sürerken silme yapılamaz.

Ana veritabanı tek başına silinemez; yalnız instance'la birlikte silinir (bkz. [Silme](managed-postgresql.md#silme)).

> **Uyarı:** Silme geri alınamaz. Silinen veritabanı ancak instance silmeden önceki bir ana geri yüklenerek geri gelir ve geri yükleme yeni bir instance oluşturur (bkz. [Yedekleme ve Geri Yükleme](#yedekleme-ve-geri-yükleme)).

---

## Yedekleme ve Geri Yükleme

Yedekler instance düzeyindedir: günlük tam yedek ve sürekli işlem günlüğü arşivi bütün veritabanlarını birlikte kapsar (bkz. [Yedekleme](managed-postgresql.md#yedekleme)). Tek bir veritabanı ayrı yedeklenmez ve tek başına geri yüklenmez.

[Geri yükleme](managed-postgresql.md#geri-yükleme) yeni bir instance oluşturur ve seçilen andaki bütün veritabanlarını verileri ve kullanıcılarıyla birlikte getirir. Kopyanın adresi kaynağınkinden farklıdır; ek veritabanlarının bağlantı bilgisini kopyanın kendi **Veritabanları** kartından alın.

Ek veritabanlarının parolaları kopyaya kaynaktan taşınır. Kaynakta artık bulunmayan, yani sonradan silinmiş bir veritabanının ya da parolası geri dönülen andan sonra sıfırlanmış bir veritabanının parolası kopyada bilinmez. Bu veritabanının bağlantı bilgisi "Bu veritabanının şifresi kayıtlı değil (örneğin bir geri yüklemeden sonra). Yeni şifre almak için şifreyi sıfırlayın." hatasını verir; parola sıfırlanana kadar bu veritabanında konsolun yazma modu, **İçgörü** sekmesindeki sorgu planı ve içe aktarma da aynı hatayla durur. Kopyada **Parolayı sıfırla** ile yeni parola alın.

---

## Veritabanı Seçimi

Instance'ta ek veritabanı olduğunda aşağıdaki ekranlarda bir **Veritabanı** seçicisi çıkar. Seçicide ana veritabanı "(ana)" ekiyle görünür ve başlangıçta seçilidir.

| Ekran | Seçimin etkisi |
|---|---|
| **Sorgu** sekmesi | Sorgular ve şema listesi seçilen veritabanında çalışır. Konsol yine varsayılan olarak salt-okunurdur; **Yazma modu** açıkken ifadeler seçilen veritabanının kendi kullanıcısıyla çalışır (bkz. [SQL Konsolu](managed-postgresql.md#sql-konsolu)). |
| **İçgörü** sekmesi | Sağlık özeti, yavaş sorgular, **İstatistikleri sıfırla** ve sorgu planı seçilen veritabanına aittir. **Aktif sorgular** listesi bütün instance'ı gösterir; **Veritabanı** sütunu her sorgunun hangi veritabanında çalıştığını yazar. |
| **Genel Bakış** → **Salt-okunur erişim** | **Kullanıcı aç** ile açılan kullanıcı yalnız seçilen veritabanını okur. Instance'ta aynı anda en fazla 3 salt-okunur kullanıcı açık olabilir; bu sayı bütün veritabanları için toplamdır. |
| **İşlemler** → **URL'den içeri al** | Dump seçilen veritabanına aktarılır (bkz. [Ek veritabanına içe aktarma](#ek-veritabanına-içe-aktarma)). |

### Ek veritabanına içe aktarma

[Dump dosyasından içe aktarma](managed-postgresql.md#dump-dosyasından-içe-aktarma) ek veritabanlarında da kullanılabilir:

- Seçilen veritabanı düşürülür ve dump'tan yeniden oluşturulur; mevcut verisi kalıcı olarak silinir. Öteki veritabanlarına dokunulmaz.
- İçe aktarılan nesnelerin sahibi o veritabanının kendi kullanıcısı olur.

Instance'ta ek veritabanı varken yapılan her içe aktarmada, hedef ana veritabanı olsa da:

- İçe aktarma sürerken işlem günlüğü arşivlemesi bütün instance için durur; hiçbir veritabanı bu aralıktaki bir ana geri yüklenemez. İçe aktarma bittiğinde, başarısız olsa da, Komuta yeni bir tam yedek başlatır; yedek tamamlandıktan sonraki her ana yeniden geri dönülebilir.
- İçe aktarma sürerken veritabanı açılamaz, silinemez ve parolası sıfırlanamaz.

---

## Stack'ten Bağlanma

Stack manifestindeki bir servis, ek bir veritabanına `valueFrom.addon` altındaki `database` alanıyla bağlanır:

```yaml
services:
  - name: rapor-api
    env:
      - name: DATABASE_URL
        sensitive: true
        valueFrom:
          addon: { name: shop-db, property: connectionString, database: raporlar }
```

- `database` yazılmazsa ya da ana veritabanının adı yazılırsa ana veritabanı bağlanır.
- Ek bir veritabanının adı yazıldığında `database`, `user`, `password` ve `connectionString` o veritabanından ve onun kullanıcısından gelir; `host` ve `port` instance'ınkidir.
- Yalnız PostgreSQL'de kullanılır. Yönetilen servis `addons` ya da `uses.addons` altında olabilir.
- Stack veritabanı açmaz: veritabanı önceden konsoldan açılmış olmalıdır. Aynı planda oluşturulan bir yönetilen servisin ek veritabanına bağlanılamaz; plan bunu `ADDON_DATABASE_NOT_OPENED` ile reddeder. Önce yönetilen servisi uygulayın, veritabanını konsoldan açın, sonra servisi ekleyin.
- Stack'in zaten çalıştırdığı bir serviste bu bağlantıyı eklemek, değiştirmek ya da kaldırmak yerinde uygulanamaz ve planı engeller; veritabanı servis ilk oluşturulurken seçilir.

Bir Stack servisi ek bir veritabanına bağlandıktan sonra o veritabanı, hiçbir Stack servisi onu kullanmayana kadar silinemez ve parolası sıfırlanamaz; konsol hangi servislerin kullandığını söyler: "Bu veritabanını bir Stack üzerinden şu servisler kullanıyor: …. Veritabanını silmek ya da şifresini sıfırlamak bağlantılarını koparır; önce bu servislerden ilgili ortam değişkenlerini kaldırın ya da servisleri silin. Hiçbir şey değiştirilmedi." Bu süre boyunca ana veritabanının parolası da yenilenemez. Koruma, servisin adımı bittiğinde başlar; uygulama sürerken bağlanılan veritabanını silmeyin ve parolasını sıfırlamayın.

Uygulama sırasında servisin adımı şu hatalarla durabilir:

| Kod | Anlamı ve yapılacak |
|---|---|
| `ADDON_DATABASE_NOT_FOUND` | Yönetilen serviste bu adla bir veritabanı yok. Veritabanını konsoldan açın, sonra yeniden uygulayın. |
| `ADDON_DATABASE_INVALID` | Ad ne ana veritabanının adı ne de geçerli bir ek veritabanı adı. Manifestteki adı düzeltin. |
| `ADDON_DATABASE_PASSWORD_UNKNOWN` | Veritabanının parolası kayıtlı değil (örneğin bir geri yüklemeden sonra). Konsoldan parolayı sıfırlayın, sonra yeniden uygulayın. Başka Stack servisleri bu veritabanını kullanıyorsa sıfırlama reddedilir. |
| `ADDON_DATABASE_CHANGED` | Servis oluşturulurken veritabanı silindi ya da parolası sıfırlandı; servis başlatılmadı ve elindeki değerler çalışmaz. Servisi silin, sonra yeniden uygulayın. |
| `ADDON_DATABASE_NOT_READABLE` | Veritabanları geçici olarak okunamadı. Adım kendiliğinden birkaç kez yeniden denenir. |
| `ADDON_DATABASE_NOT_AVAILABLE` | Yönetilen servisin veritabanları okunamıyor. Destek talebi açın. |

Manifest alanlarının ayrıntısı ve doğrulama mesajları için [komuta.yaml Manifest Başvurusu](stack-manifest-reference.md) sayfasına bakın.

---

## Sık Sorulan Sorular

### Ek veritabanı ayrıca ücretlendirilir mi?

Hayır. Ek veritabanları instance'ın planını paylaşır ve instance'ın ücreti değişmez. Daha fazla CPU, bellek ya da disk gerekiyorsa instance'ın planı yükseltilir (bkz. [Plan Değişikliği](managed-postgresql.md#plan-değişikliği)).

### Ek veritabanının kullanıcısı ana veritabanını okuyabilir mi?

Okuyamaz. Ek veritabanının kullanıcısı yalnız kendi veritabanına bağlanabilir; ana veritabanının kullanıcısı da ek veritabanlarına bağlanamaz. Bir aracın ya da kişinin yalnız okuması gerekiyorsa **Salt-okunur erişim** kartından o veritabanı için salt-okunur bir kullanıcı açın.

### Tek bir veritabanını geri yükleyebilir miyim?

Tek başına geri yükleyemezsiniz. Yedekler ve geri yükleme instance düzeyindedir: geri yükleme, bütün veritabanlarını seçilen andaki hâliyle içeren yeni bir instance oluşturur.

### Veritabanının adı değiştirilebilir mi?

Değiştirilemez; ad aynı zamanda veritabanının kullanıcı adıdır. Başka bir ad gerekiyorsa yeni adla bir veritabanı açılır, veri ona aktarılır ve eskisi silinir.

### İkinci veritabanını açmak ana veritabanını etkiler mi?

Veritabanı açmak PostgreSQL'i yeniden başlatmaz ve ana veritabanının verisine dokunmaz. Değişen bağlantı payıdır: ana veritabanının aynı anda kullanabileceği sunucu bağlantısı paylaşıma göre azalır (bkz. [Bağlantıların paylaşımı](#bağlantıların-paylaşımı)). Bağlantı havuzunun ayarı değiştiği için havuz bu sırada bir kez yenilenebilir; kopan bir bağlantıyı yeniden açabilen uygulamalar bundan etkilenmez.
