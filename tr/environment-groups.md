# Ortam Değişkeni Grupları

Ortam değişkeni grubu, birden fazla servisin kullandığı değişkenleri tek yerde tutar: ortak bir veritabanı bağlantısı, bir API anahtarı, tüm servislerde aynı olan bir log ayarı gibi. Değeri bir kez gruba yazarsınız, sonra istediğiniz servise aktarırsınız.

Grup canlı bir bağ değildir, bir kaynaktır. Aktarma sırasında değerler servise kopyalanır ve servis o andan itibaren kendi kopyasıyla çalışır. Grupta bir değeri değiştirmek hiçbir servisi kendiliğinden değiştirmez; güncellemeyi servislere siz gönderirsiniz.

Gruplar sol menüde **Yapılandırma → Ortam Değişkeni Grupları** altında, [console.komuta.io/env-groups](https://console.komuta.io/env-groups) adresindedir. Gruplar organizasyonun tamamında ortaktır; her projedeki servis aynı gruptan aktarabilir.

---

## Grup mu, Servis Değişkeni mi?

| Durum | Kullanın |
|---|---|
| Değer yalnızca bir servise ait | Servisin kendi [ortam değişkenleri](services-environment-variables.md) |
| Aynı değer birden fazla serviste kullanılıyor ve birlikte güncellenmeli | Ortam değişkeni grubu |
| Değer çoğu serviste aynı, birinde farklı olmalı | Gruptan aktarın, farklı olması gereken serviste değeri düzenleyin. Düzenlenen değişken gruptan kopar. |

---

## Grup Oluşturma

1. **Ortam Değişkeni Grupları** sayfasını açın. Henüz grup yoksa **İlk grubunuzu oluşturun** düğmesine, varsa liste başlığındaki **+** (**Yeni grup**) düğmesine tıklayın.
2. **Yeni ortam değişkeni grubu** penceresinde **Ad** alanını doldurun (ör. `shared-postgres`). **Açıklama (isteğe bağlı)** alanına grubun ne için olduğunu yazabilirsiniz.
3. **Grubu oluştur**'a tıklayın.

Yeni grup açılır ve iki sekme gösterir: **Değişkenler** ve **Servisler**. Soldaki listede her grubun değişken sayısı, kaç tanesinin hassas olduğu (anahtar simgesi) ve kaç serviste kullanıldığı (sunucu simgesi) görünür. **Grup ara...** kutusu ada ve açıklamaya göre arar.

Adı ya da açıklamayı sonradan değiştirmek için grubun başlığındaki **Düzenle**'yi kullanın ve **Kaydet**'e tıklayın. Grubun adını değiştirmek servislerdeki kopyaların bağını koparmaz.

---

## Değişken Ekleme

1. **Değişkenler** sekmesinde **Değişken ekle**'ye tıklayın.
2. **Gruba değişken ekle** penceresinde **Anahtar** ve **Değer** alanlarını doldurun.
3. Gerekiyorsa aşağıdaki seçenekleri açın ve **Ekle**'ye tıklayın.

| Seçenek | Ne yapar |
|---|---|
| **Hassas** | Değer şifreli saklanır ve bir daha gösterilmez. Servise aktarıldığında servisin gizli değerleri arasına yazılır ve servis sayfasındaki **Gizli** sekmesinde listelenir. |
| **Build sırasında erişilebilir** | Değer yalnızca çalışan uygulamaya değil, derleme aşamasına da verilir. `NEXT_PUBLIC_`, `VITE_`, `REACT_APP_` gibi önekli bir anahtar yazdığınızda kendiliğinden açılır. |

Kimlik bilgisi taşıdığı belli olan anahtarlar, **Hassas** kapalı olsa bile her zaman hassas saklanır. Büyük/küçük harf ve `_`, `-`, `.` işaretleri dikkate alınmadan adı `PASSWORD`, `SECRET`, `SECRET_KEY`, `API_KEY`, `PRIVATE_KEY`, `ACCESS_TOKEN`, `REFRESH_TOKEN`, `AUTH_TOKEN`, `CREDENTIAL`, `CREDENTIALS`, `CONNECTION_STRING`, `APPLICATION_KEY`, `SIGNING_KEY`, `ENCRYPTION_KEY`, `WEBHOOK_KEY`, `DATABASE_URL`, `REDIS_URL`, `RABBITMQ_URL` veya `AMQP_URL` ile biten ya da `ConnectionStrings` ile başlayan anahtarlar bu kapsamdadır.

Listede düz değerler açıkça, hassas değerler kilit simgesi ve noktalarla gösterilir. Hassas değişkenlerin yanında turuncu, build değişkenlerinin yanında mor bir nokta ve **Build** etiketi bulunur.

> **Uyarı:** Hassas olarak işaretlenmeyen değerleri, grupları görüntüleme yetkisi olan herkes okuyabilir. **Build sırasında erişilebilir** işaretli bir değer imaja, public önekli ise tarayıcı paketine de gömülür; gizli kalması gereken bilgileri bu şekilde tanımlamayın.

### Değişkeni düzenleme

Satırın üzerine geldiğinizde **Düzenle** ve **Sil** düğmeleri görünür. **Grup değişkenini düzenle** penceresinde:

- Hassas bir değeri değiştirmek istemiyorsanız **Değer** alanını boş bırakın; saklanan değer korunur.
- Hassas bir değişkeni düz değere çevirmek için değeri yeniden yazmanız gerekir; saklanan değer hiçbir zaman gösterilmez.
- Anahtarın adını değiştirirseniz servislerdeki eski adlı kopyaların gruba bağı kopar. Bu kopyalar servislerde kendi değişkenleri olarak kalır.

---

## .env Yapıştırarak Toplu Ekleme

1. **Değişkenler** sekmesinde **.env yapıştır**'a tıklayın.
2. Açılan pencereye bir `.env` dosyasının ya da JSON'un içeriğini yapıştırın ve **Önizle**'ye tıklayın.
3. Önizlemede her satırın durumunu kontrol edin. Gerekirse satırdaki **Hassas** anahtarını açıp kapatın.
4. **… değişkeni kaydet**'e tıklayın. Sonuç "*X eklendi, Y değiştirildi.*" biçiminde bildirilir.

| Etiket | Anlamı |
|---|---|
| **Yeni** | Anahtar grupta yok; eklenir. |
| **Değiştirir** | Anahtar grupta zaten var; değeri yapıştırılan değerle değiştirilir. |
| **Atlandı** | Değeri boş; kaydedilmez. |

Yapıştırma kuralları:

- `.env` içinde `#` ile başlayan satırlar atlanır, `export ` öneki kabul edilir, değeri saran tırnaklar kaldırılır.
- İç içe JSON düzleştirilir; seviyeler `__` ile birleşir (`{"Logging": {"Level": "info"}}` → `Logging__Level`).
- Aynı anahtar birden fazla kez geçiyorsa son değer kullanılır.
- Geçersiz anahtar adı taşıyan satırlar önizlemeye hiç gelmez.
- Adında `SECRET`, `PASSWORD`, `TOKEN`, `CREDENTIAL`, `PRIVATE`, `API_KEY`, `PWD`, `AUTH` gibi ifadeler geçen satırlarda ve grupta zaten hassas olan anahtarlarda **Hassas** açık gelir.
- Bu pencerede build seçeneği yoktur: yeni anahtarlarda önekten belirlenir, var olanlarda değişmez. Değiştirmek için değişkeni tek tek düzenleyin.

Toplam değişken sayısı sınırı aşacaksa "*Bir grupta en fazla 500 değişken olabilir.*" uyarısı görünür ve kaydetme kapanır. Metni düzeltmek için **Geri**'ye tıklayın.

---

## Grubu Bir Servise Aktarma

Aktarma servisin sayfasından yapılır:

1. Servisi açın ve **Ortam değişkenleri** sayfasına geçin.
2. Araç çubuğundaki **Gruptan** düğmesine tıklayın (kutular simgesi; adı üzerine gelince görünür). Aynı ekrana **Toplu İçe Aktar** penceresindeki **Gruptan** sekmesinden de ulaşabilirsiniz. Servisin hiç değişkeni yoksa **Gruptan** düğmesi boş ekranın ortasında görünür.
3. **Ortam değişkeni grubu** listesinden grubu seçin.
4. Gruptaki her değişken, servisteki değerle karşılaştırılmış olarak listelenir. Aktarmak istemediklerinizin işaretini kaldırın.
5. **… değişkeni aktar**'a tıklayın. Servisteki bir değer değişecekse düğme "*… değişkeni aktar (… tanesi değişir)*" olarak görünür.
6. Servis sayfasında **Değişiklikleri Uygula**'ya tıklayın. Aktarılan değerler kaydedilir, ancak uygulayana kadar çalışan servise ulaşmaz.

| Etiket | Anlamı |
|---|---|
| **Yeni** | Anahtar serviste yok; eklenir. |
| **Değiştirir** | Serviste farklı bir değer ya da farklı bir **Hassas**/build ayarı var; grubun değeri yazılır. Düz değerlerde eski ve yeni değer yan yana gösterilir. |
| **Aynı değer** | Serviste aynı değer ve ayarlar var; değer yeniden yazılmaz, yalnızca gruba bağlanır. |

Aktarma bitince "*X eklendi, Y değiştirildi, Z zaten aynıydı. Devreye almak için değişiklikleri uygulayın.*" mesajı görünür. Aktarılan her değişkenin yanında grubun adını taşıyan bir etiket bulunur; etikete tıklamak grubu açar.

> **Not:** Serviste hassas olarak tutulan bir değişkenin gruptaki karşılığı düz değerse satırda "*(serviste hassas olarak tutuluyor; aktarılırsa düz değere dönüşür)*" uyarısı çıkar ve satır işaretsiz gelir. Bu satırı yalnızca bilerek seçin.

Bir değişken aynı anda yalnızca bir gruba bağlı olabilir. Serviste başka bir gruptan gelmiş bir anahtar "*(şu an … grubundan)*" notuyla gösterilir; aktarırsanız artık bu gruba bağlanır.

---

## Grup Değişince Servislere Ne Olur?

Hiçbir şey, siz göndermedikçe. Grubun başlığında da bu hatırlatılır: "*Servisler bu değerleri içe aktardığında bir kopyasını alır. Buradaki bir değeri değiştirmek, Servisler sekmesinden gönderene kadar hiçbir servisi değiştirmez.*"

**Servisler** sekmesi, gruptan kopya alan servisleri proje adlarıyla listeler. Her satırda servisin bu gruptan kaç değişken aldığı ve bir durum etiketi bulunur:

| Etiket | Anlamı |
|---|---|
| **Güncel** | Servisteki tüm kopyalar grupla aynı. |
| **Güncel değil** | En az bir kopyanın değeri, **Hassas** ya da build ayarı gruptakinden farklı. Farklı olan anahtarlar satırda listelenir. |

Güncel olmayan kopya varsa sekme adının yanında turuncu bir nokta görünür.

### Servislere gönderme

1. **Servisler** sekmesini açın. Güncel olmayan kopyası olan ve düzenleyebildiğiniz servisler seçili gelir; istemediklerinizin işaretini kaldırın.
2. **… servise gönder**'e tıklayın.
3. **Grup değerlerini gönder** penceresinde hangi serviste hangi anahtarların yazılacağını kontrol edin.
4. **Pod'ları şimdi yeniden başlat** seçeneğine karar verin (varsayılan olarak açıktır) ve **Gönder ve yeniden başlat** ya da **Değerleri gönder**'e tıklayın.
5. Sonuçları okuyup **Kapat**'a tıklayın.

| Sonuç | Anlamı |
|---|---|
| **… değişken güncellendi · devreye alınıyor** | Değerler servise yazıldı; servis yeni değerlerle yeniden başlatılıyor. |
| **… değişken güncellendi · bir sonraki deploy'u bekliyor** | Değerler servise yazıldı; çalışan servis bir sonraki deploy'a ya da servis sayfasında **Değişiklikleri Uygula**'ya kadar eski değerlerle çalışır. |
| **Zaten güncel** | Yazılacak bir şey yoktu. |
| Kırmızı hata metni | Servise yazılamadı; nedeni satırda gösterilir. |

Gönderme yalnızca hâlâ gruba bağlı kopyaların üzerine yazar. Serviste elle düzenlenmiş değerler gruptan kopmuştur ve olduğu gibi kalır.

> **Uyarı:** Uygulamalar değişkenlerini başlarken okur. **Pod'ları şimdi yeniden başlat** kapalıyken hassas bir değeri gönderirseniz, bu arada kendiliğinden yeniden başlayan pod'lar yeni değeri, diğerleri eski değeri kullanır. Birlikte değişmesi gereken değerler (ör. kullanıcı adı ve şifre) için seçeneği açık bırakın.

- **Pod'ları şimdi yeniden başlat** açıkken bir servisteki değerlerin bir kısmı yazılamazsa o servis yeniden başlatılmaz ve "*Devreye alınmadı: bazı değerler yazılamadı. Sorun giderilince yeniden gönderin.*" mesajı görünür. Yazılamayan anahtarlar **Güncel değil** olarak kalır; yeniden gönderdiğinizde eksikler tamamlanır ve servis yeniden başlatılır.
- Değerler yazıldığı hâlde yeniden başlatma başarısız olursa değerler serviste kayıtlıdır; servis sayfasında **Değişiklikleri Uygula** ile devreye alabilirsiniz.
- **Build sırasında erişilebilir** işaretli değerler imaja derleme sırasında girer; pod'ları yeniden başlatmak imajı yeniden derlemez. Böyle bir değerin uygulamaya ulaşması için servisi yeniden derleyip dağıtın ([Pipeline'lar](service-pipeline-guide.md)).

---

## Hangi Değer Geçerli Olur?

Çalışan servis yalnızca kendi değişkenlerini kullanır; grup, deploy sırasında servisin değerleriyle birleştirilmez. Aynı anahtar hem grupta hem serviste varsa, siz aktarana ya da gönderene kadar servisin değeri geçerlidir.

| Durum | Geçerli olan değer |
|---|---|
| Anahtar yalnızca grupta | Servise aktarılana kadar serviste yoktur. |
| Anahtar hem grupta hem serviste, servis bu anahtarı gruptan almamış | Servisin değeri. Aktarırsanız grubun değeri yazılır (**Değiştirir**). |
| Servisteki kopya gruba bağlı, grupta değer değişti | Gönderene kadar servisin eski değeri (**Güncel değil**). |
| Servisteki kopya serviste düzenlendi | Servisin değeri. Bağ kopmuştur; gönderme bu değere dokunmaz. |

Servisteki bir kopyanın gruba bağı şu durumlarda kopar:

- Servis sayfasında değişkenin değerini, anahtarını ya da **Hassas** ayarını değiştirdiğinizde.
- **Toplu İçe Aktar** ile farklı bir değer yapıştırdığınızda.
- Bir [Stack](stacks.md) aynı değişkene değer yazdığında.
- Grupta değişkeni sildiğinizde, adını değiştirdiğinizde ya da grubu sildiğinizde.

Bağı kopan değişkenin yanındaki grup etiketi kaybolur. Yeniden gruba bağlamak için gruptan tekrar aktarın.

> **Not:** Serviste yalnızca build ayarını değiştirmek bağı koparmaz. Bu durumda kopya **Güncel değil** görünür ve bir sonraki gönderimde grubun ayarına döner.

---

## Değişken ve Grup Silme

**Değişken silme:** Satırdaki **Sil** düğmesine tıklayın ve **Değişkeni sil** penceresinde onaylayın. **Grup silme:** Grubun başlığındaki çöp kutusu simgesine (**Grubu sil**) tıklayın ve onaylayın.

İki işlem de geri alınamaz, ancak servislere dokunmaz. Kopya alan servis varsa onay penceresi bunu belirtir: "*… serviste bir kopya var. Kopyalar servislerde kendi değişkenleri olarak kalır; hiçbir şey silinmez ya da yeniden deploy edilmez.*" Grup silindiğinde "*Grup silindi. Servisler kopyalarını koruyor.*" mesajı görünür.

Bir değişkeni servisten tamamen kaldırmak istiyorsanız servisin **Ortam değişkenleri** sayfasında silin ve **Değişiklikleri Uygula**'ya tıklayın. Bir servisi gruptan ayırmak için ayrı bir düğme yoktur; değişkeni serviste düzenlemek ya da silmek yeterlidir.

---

## Kimler Yönetebilir?

Organizasyon yöneticileri, üyelerin bu sayfada neler yapabileceğini **Erişim Yönetimi** sayfasında üyeyi seçip **Ortam Değişkeni Grupları** kartından belirler.

| Yetki | İzin verdiği |
|---|---|
| **Ortam değişkeni gruplarını görüntüle** | Sayfayı ve grupları görmek, düz değerleri okumak, **Servisler** sekmesini görmek. Servisi düzenleme yetkisiyle birlikte: servise **Gruptan** aktarmak ve grubu o servise göndermek. |
| **Ortam değişkeni grubu oluştur** | Yeni grup oluşturmak. |
| **Grupları ve değişkenlerini düzenle** | Ad ve açıklamayı değiştirmek; değişken eklemek, düzenlemek, silmek; **.env yapıştır**. |
| **Ortam değişkeni gruplarını sil** | Grubu silmek. |

Aktarma ve gönderme ayrıca hedef servisi düzenleme yetkisi ister; böylece bir üye grubun değerlerini yalnızca zaten düzenleyebildiği servislere yazabilir. **Servisler** sekmesi yalnızca görebildiğiniz servisleri listeler. Düzenleyemediğiniz bir serviste "*Bu servisi düzenleme yetkiniz yok, bu yüzden buradan güncellenemez.*" yazar ve servis seçilemez.

---

## Sınırlar ve Doğrulama

| Alan | Kural |
|---|---|
| Grup adı | Zorunlu, en fazla 128 karakter. Organizasyonda benzersiz olmalı; büyük/küçük harf fark etmez. |
| Açıklama | İsteğe bağlı, en fazla 512 karakter. |
| Değişken sayısı | Grup başına en fazla 500. |
| Anahtar | En fazla 256 karakter; yalnızca harf, rakam, `-`, `_` ve `.`. Grupta benzersiz olmalı; büyük/küçük harfe duyarlıdır (`db_url` ile `DB_URL` farklıdır). |
| Değer | Zorunlu. Düz değerlerin başındaki ve sonundaki boşluklar kırpılır; hassas değerler olduğu gibi saklanır. |

Karşılaşabileceğiniz mesajlar:

| Mesaj | Ne zaman |
|---|---|
| "*'…' adında bir ortam değişkeni grubu zaten var.*" | Organizasyonda aynı adlı bir grup var. |
| "*Bu anahtara sahip bir değişken zaten var*" / "*Bu grupta '…' adında bir değişken zaten var.*" | Grupta aynı anahtar var. |
| "*'…' geçerli bir değişken adı değil. Yalnızca harf, rakam, '-', '_' ve '.' kullanın.*" | Anahtarda izin verilmeyen bir karakter var. |
| "*'…' için bir değer girin.*" | Değer boş; ya da hassas bir değişkeni düz değere çevirirken değeri yeniden yazmadınız. |
| "*Bir grupta en fazla 500 değişken olabilir.*" | Ekleme sınırı aşıyor. |
| "*Grupta artık şu değişkenler yok: …. Sayfayı yenileyip tekrar deneyin.*" | Aktarma sırasında seçtiğiniz bir değişken gruptan silinmiş ya da adı değişmiş. |
| "*Devreye alınmadı: bazı değerler yazılamadı. Sorun giderilince yeniden gönderin.*" | Gönderimde bir servisin değerlerinin bir kısmı yazılamadı. |

---

## Yapay Zekâ Asistanıyla

Komuta MCP bağlıysa asistanınız grupları listeleyebilir ve gösterebilir, grup oluşturabilir, gruba değişken ekleyip güncelleyebilir, bir servise aktarmadan önce karşılaştırmayı gösterebilir, aktarabilir ve grubu servislere gönderebilir. Değişiklik yapan işlemlerden önce onayınızı ister; hassas değerleri hiçbir zaman okuyamaz. Kurulum için [MCP Kurulumu](mcp-setup.md), araçların genel listesi için [Komuta MCP](mcp-tools.md) sayfasına bakın.

---

## İlgili Dokümanlar

- [Ortam Değişkenleri](services-environment-variables.md) — Servisin kendi değişkenleri, türleri ve **Değişiklikleri Uygula**.
- [Pipeline'lar](service-pipeline-guide.md) — Build değişkenlerinin kullanıldığı derleme aşaması.
- [Stack'ler](stacks.md) — Servis değişkenlerini Stack dosyasıyla yönetmek.
- [Organizasyon ve Ekip](organizations-team.md) — Üyeler, roller ve yetkiler.
- [Komuta MCP](mcp-tools.md) — Asistanla yapılabilen işlemler.
