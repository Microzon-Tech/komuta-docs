# Sorun Giderme ve SSS

Bu sayfa Komuta'da en sık karşılaşılan sorunları bir arada toplar. Her başlıkta önce ne gördüğünüz (varsa konsoldaki mesajla birlikte), sonra olası neden, en sonda da ne yapmanız gerektiği yer alır. Ayrıntı gereken yerlerde ilgili sayfaya bağlantı verilir.

Buradaki adımlar sorunu çözmezse **Yardım Alma** bölümüne geçin.

| Gördüğünüz | Bölüm |
|---|---|
| Pipeline **Başarısız** oluyor ya da build başlamıyor | **Build Başarısız Oluyor** |
| Deploy bitti ama adres açılmıyor, hata kodu dönüyor ya da yanıtlar yavaş | **Deploy Başarılı Ama Servis Açılmıyor** |
| Servis uyuyor, geç açılıyor ya da uyanmıyor | **Servis Uyuyor ya da Uyanmıyor** |
| Bir şey oluştururken hata alıyorsunuz | **Bir Şey Oluşturamıyorum** |
| Repo listede yok ya da Git bağlantısı hata veriyor | **Git Bağlantısı** |
| Alan adı **Hazırlanıyor** ya da **İşlem Gerekli** durumunda kalıyor | **Özel Alan Adı** |
| Servis veritabanına bağlanamıyor | **Veritabanı Bağlantısı** |
| Log, metrik ya da trace ekranı boş | **Loglar, Metrikler ve Trace'ler Boş** |

---

## Build Başarısız Oluyor

### Pipeline "Başarısız" görünüyor

**Ne görürsünüz:** Servisin **Pipeline'lar** sekmesinde çalışma **Başarısız** durumundadır.

**Ne yapmalı:**

1. Çalışmayı açın. **Neden başarısız oldu?** bölümü hatalı görevin hata satırlarını gösterir; hata olarak işaretlenmiş satır yoksa son satırları listeler.
2. Otomatik onarım kaydında "Uygulamanın kendisi derlenmedi — bunu bir Dockerfile değişikliği çözemez." yazıyorsa hata kodunuzdadır; düzeltip push edin.
3. Hatayı yorumlatmak için **AI ile Çöz** menüsünü, Komuta MCP kuruluysa **MCP ile Çöz**'ü kullanın.
4. **Tekrar dene** dalın en son commit'ini derler. Aynı commit'te hata sürüyorsa önce düzeltmeyi push edin.

Ayrıntılar: [Pipeline'lar](service-pipeline-guide.md).

### Dockerfile pull request'i merge edilmedi

**Ne görürsünüz:** Servis panosunda **İşlem gerekli: Dockerfile pull request'ini merge edin** uyarısı. Deploy penceresinde ise: "Açık ve henüz merge edilmemiş bir AI Dockerfile PR'ı var. Önce branch'inize merge edin; aksi halde bu dağıtım başarısız olabilir."

**Neden:** Repoda Dockerfile yoktu ve yapay zekâ bir tane üretti ya da bozuk Dockerfile'ı onardı. Değişiklik ayrı bir dalda pull request olarak açılır. Servisin branch'ine merge edilene kadar sonraki build'ler eski hâliyle çalışır ve başarısız olur.

**Ne yapmalı:** Uyarıdaki **PR'ı Aç** ile pull request'i açın, inceleyip servisin branch'ine merge edin, sonra yeniden deploy edin. Deploy penceresinde **Dağıt**'a ikinci kez basmak uyarıyı geçer ama build büyük olasılıkla yine başarısız olur.

### Build kuyrukta bekliyor ya da kuyruğa girmiyor

**Ne görürsünüz:** Servis listesinde **Kuyrukta #N** çipi; ya da yeni build isteğinde "Planınızın build kuyruğu dolu. Bekleyen build'lerin başlamasını bekleyin veya bazılarını iptal edin; daha büyük kuyruk için planınızı yükseltebilirsiniz."

**Neden:** Aynı servisin önceki build'i sürüyordur, hesap seviyenizin eşzamanlı build sınırı doludur ya da derleme kapasitesi o an doludur. Kuyruktaki her build neden beklediğini gösterir.

**Ne yapmalı:** Bekleyin ya da gereksiz build'leri iptal edin. Seviyelere göre eşzamanlı build ve kuyruk sınırları [Build Kuyruğu](build-queue.md) sayfasındadır; hesap seviyeniz **Hesap → Cüzdan** sayfasında görünür.

> **Not:** Build dakikaları dolunca build'ler durmaz. Her fatura döneminde 2.000 dakika dahildir; aşan her dakika 0,008 USD olarak o dönemin ekstresine yansır. Kuyrukta geçen süre sayılmaz. Kullanımı **Hesap → Güncel Harcama** sayfasındaki **Build dakikaları** kartında görürsünüz.

---

## Deploy Başarılı Ama Servis Açılmıyor

Önce servis panosundaki durum kartına bakın; kartın başlığı sorunun türünü söyler.

| Kart başlığı | Anlamı | Ne yapmalı |
|---|---|---|
| **Dağıtım sürüyor** | Yeni sürüm hâlâ açılıyor. | Bitmesini bekleyin; kart kendiliğinden güncellenir. |
| **Yayında, ama URL yanıt vermiyor** | Pod'lar hazır ama genel adres 5xx döndürüyor ya da hiç yanıt vermiyor. | **Yanlış port** ve **Sağlık kontrolleri** başlıklarına bakın. |
| **Container ayağa kalkmıyor** | Uygulama açılırken çöküyor. | **Uygulama açılışta çöküyor** başlığına bakın. |
| **Yeni dağıtım başarısız oldu; kararlı sürüm yayında tutuldu** ya da **Arka arkaya restart nedeniyle servis otomatik durduruldu** | Restart koruması devreye girdi. | **Restart koruması** başlığına bakın. |
| **Dağıtım senkronizasyonu başarısız: değişiklikler cluster'a ulaşmadı** | Son değişiklik uygulanamadı. | **Yeniden senkronize et**'i deneyin; sorun sürerse karttaki mesaja göre düzeltip yeniden deploy edin. |
| **Uygulama uykuda** | Servis uyuyor. | **Servis Uyuyor ya da Uyanmıyor** bölümüne bakın. |
| **Bakımda** | Bakım modu açık; ziyaretçiler bakım sayfası görür (HTTP 503). | Hazır olduğunuzda **Bakımı bitir**'e basın. |

### Yanlış port

**Ne görürsünüz:** **Yayında, ama URL yanıt vermiyor**. Komuta, uygulamanın loglarından başka bir portu dinlediğini anlarsa kartta `expected port 8080, app is listening on 3000 — …` biçiminde bir satır gösterir.

**Neden:** Trafik, servisin birincil portunun **Hedef** değerine gönderilir. Uygulama başka bir portu dinlerse pod'lar hazır görünür ama istekler uygulamaya ulaşmaz.

**Ne yapmalı:**

- Uygulamanın `PORT` ortam değişkenini okumasını sağlayın. Komuta bu değişkeni birincil portun **Hedef** değeriyle doldurur; `PORT`'u kendiniz tanımladıysanız sizin değerinize dokunmaz.
- Ya da servisin **Erişim ve portlar** sayfasında birincil portun **Hedef** değerini uygulamanın gerçekten dinlediği porta eşitleyin ve yeniden deploy edin.
- Uygulama 1024'ün altındaki bir portu (ör. 80) dinlemeye çalışırsa açılamaz; kartta "Container 1024 altı bir port'a bind etmeye çalışıyor" ipucu çıkar. Uygulamayı 1024 veya üstü bir porta (ör. 8080) alın ya da ipucundaki **Capability ayarlarını aç** ile bu izni servise verin.
- Servisin genel bir adrese ihtiyacı yoksa (ör. arka plan işçisi) karttaki **Genel URL'yi yönet** ile adresi kapatabilirsiniz.

Komuta'nın adres kontrolü yalnızca 5xx yanıtlarını ve hiç yanıt alınamamasını hata sayar; `401`, `403` ya da `404` döndüren bir uygulama ayakta kabul edilir.

Ayrıntılar: [Erişim ve Portlar](services-ports.md), [Ortam Değişkenleri](services-environment-variables.md).

### Uygulama açılışta çöküyor

**Ne görürsünüz:** **Container ayağa kalkmıyor**. Kartın altında önceki container'ın son logu ve tanınan durumlar için bir ipucu yer alır.

Her yeniden başlatmanın nedeni **Yeniden başlatma ve çökme geçmişi** penceresinde tutulur. Pencereyi servis panosundaki "Son 7 günde N container yeniden başlatması" kartından ya da **Loglar** sayfasından açarsınız; **Bu anın çevresindeki logları aç** sizi o anın loglarına götürür.

| Konsoldaki işaret | Neden | Ne yapmalı |
|---|---|---|
| "Container bellek limitini aşınca öldürüldü" | Uygulama paketin belleğini aştı. | Bellek kullanımını azaltın ya da **Kaynak paketini değiştir** ile daha büyük bir pakete geçin. |
| "Uygulama kendiliğinden sonlandı (çıkış kodu 0); bir servis süreci çalışmaya devam etmelidir." | Ana süreç işini bitirip çıktı. | Ana süreci ön planda tutun; arka plana atıp çıkmayın. |
| "Uygulama hatayla sonlandı (çıkış kodu N)." | Uygulama açılışta hata verdi. | Son log satırlarındaki hatayı düzeltin. Eksik bir ortam değişkeni ya da gizli değer sık görülen nedendir. |
| "Container 1024 altı bir port'a bind etmeye çalışıyor" | Ayrıcalıklı port. | **Yanlış port** başlığındaki adımları uygulayın. |

> **Not:** Bir ortam değişkenini ekledikten ya da değiştirdikten sonra **Değişiklikleri Uygula**'ya basmadıkça değer çalışan servise ulaşmaz.

### Restart koruması

**Ne görürsünüz:** Bir pod art arda yeniden başladığında Komuta döngüyü durdurur ve iki karttan birini gösterir:

- **Yeni dağıtım başarısız oldu; kararlı sürüm yayında tutuldu** — Yalnızca yeni sürüm geri çekilir; önceki sağlıklı sürüm trafiğe hizmet etmeye devam eder.
- **Arka arkaya restart nedeniyle servis otomatik durduruldu** — Servis sıfır replikaya indirilir ve istekleri yanıtlamaz.

Kartta **Tespit edilen neden**, **Son çökme logu** ve **Durdurulma öncesi logları aç** bağlantısı yer alır.

**Ne yapmalı:** Tespit edilen hatayı düzeltin, değişikliği commit'leyin ve servisi yeniden deploy edin. Sağlıklı açılan yeni bir dağıtım korumayı kendiliğinden kaldırır.

> **Uyarı:** Aynı sürümü yeniden denemek servisi geri getirmez. Koruma ancak gerçek bir değişiklik (kod ya da yapılandırma) içeren yeni bir dağıtım sağlıklı açıldığında kalkar.

### Sağlık kontrolleri başarısız oluyor

**Ne görürsünüz:** Readiness kontrolünü geçemeyen pod trafik almaz. Liveness kontrolünü geçemeyen container yeniden başlatılır ve çökme geçmişinde "Container sağlık kontrolüne yanıt vermeyi bıraktı, bu yüzden Kubernetes onu yeniden başlattı." yazar. Art arda yeniden başlatmalar restart korumasını tetikleyebilir.

**Sık nedenler ve çözümleri:**

- **Kontrol yolu ya da portu yanlış.** Yeni açılan bir kontrol varsayılan olarak `/health` yolunu ve 80 numaralı portu sorgular. **Yol** ve **Port** alanlarını uygulamanıza göre değiştirin.
- **Uygulama geç açılıyor.** **Startup** kontrolü tanımlayın ya da **İlk gecikme (sn)** değerini artırın.
- **Uygulama yanıt veremeyecek kadar yavaşlıyor.** Konsolun ipucuna göre bunun nedeni kilitlenen bir iş parçacığı ya da yavaş bir bağımlılık olabilir. Zaman aşımını ve hata eşiğini gözden geçirin.

Ayrıntılar: [Sağlık Kontrolleri](services-health-probes.md).

### Adres 502, 503 ya da 504 döndürüyor

| Kod | Sık neden | Ne yapmalı |
|---|---|---|
| `503` | Trafiği alacak hazır pod yok: yeni sürüm açılamadı, restart koruması servisi durdurdu ya da readiness kontrolü geçmiyor. Bakım modu ile uyandırılamayan servislerin sayfaları da `503` döner. | Servis panosundaki durum kartından nedeni bulun. |
| `502` | Uygulama bağlantıyı kapattı ya da geçerli bir yanıt veremedi. | Aynı zamana ait servis loglarına bakın. |
| `504` | Uygulama zamanında yanıt vermedi. | Yavaş istekleri ve bağımlılıkları (veritabanı, dış API) inceleyin. |

> **Not:** `502` ve `504` yanıtlarında uygulamanızın döndürdüğü gövde çağırana ulaşmayabilir; çağıran onun yerine `error code: 502` ya da `error code: 504` gibi kısa bir düz metin görür. Hatanın ayrıntısını yanıt gövdesinde değil, servis loglarında arayın.

### Yanıtlar ya da indirmeler yavaş

**Neden:** Servisin bant genişliği planı her pod'un giden (çıkış) trafiğine bir hız sınırı koyar. Pod bu sınıra ulaştığında yanıtlar yavaşlar; büyük dosyalar ve yoğun trafik bunu ilk gösteren yerlerdir. Gelen trafik sınırlanmaz.

**Ne yapmalı:** Servisin **Bant Genişliği** sayfasında giden trafiğin **Çıkış (pod başına)** sınırına yaklaşıp yaklaşmadığına bakın. Sınıra dayanıyorsa daha yüksek bir plan seçin. Ayrıntılar: [Bant Genişliği Planları](bandwidth-plans.md).

### Ziyaretçi giriş sayfasına yönlendiriliyor ya da engelleniyor

**Ne görürsünüz:** Ziyaretçi **Devam etmek için giriş yapın** sayfasına yönlendirilir ya da giriş yaptıktan sonra **Erişiminiz yok** görür. Webhook ve CI çağrıları `401` ya da `403` alır.

**Neden:** Serviste erişim koruması açık ve istek kurallara uymuyor: ziyaretçi giriş yapmamış, servis onun hesabıyla paylaşılmamış ya da IP adresi veya ülkesi izin listesinde değil.

**Ne yapmalı:** Kişiyi paylaşıma ekleyin ya da kuralları gözden geçirin; makinelerden gelen çağrılar için servis token'ı kullanın. Ziyaretçiye dönen her yanıt [Erişim Koruması: Başvuru](access-protection-reference.md) sayfasında listelenir. Ayrıca bkz. [Erişim Koruması](service-access-protection.md) ve [Makineler ve Özel Ağ](access-protection-machines.md).

---

## Servis Uyuyor ya da Uyanmıyor

Ücretsiz paketteki bir servis 30 dakika trafik almazsa uyur ve ilk istekle uyanır; uyanması yaklaşık bir dakika sürebilir. Tarayıcıdan gelen ziyaretçi bu sürede **Bu servis uyanıyor** sayfasını görür; sayfa servis hazır olunca kendiliğinden yenilenir. Ücretli paketlerdeki servisler kendiliğinden uyumaz.

Servis uyanamıyorsa ziyaretçi bir bilgi sayfası (HTTP 503), siz de servis panosunda **Uygulama uykuda** kartını görürsünüz:

| Ziyaretçinin gördüğü | Konsoldaki açıklama | Ne yapmalı |
|---|---|---|
| **Bu servis bu ay için uykuda** | "Bu ayın ücretsiz saatleri doldu; bu ücretsiz plan uygulaması havuz sıfırlanana kadar uyur." | Ücretsiz servisler ayda toplam 450 uyanık saati paylaşır. Havuz dolunca bu servisler ayın 1'ine (UTC) kadar uyandırılamaz ve deploy edilemez. Beklemek istemiyorsanız servise **Her zaman açık eklentisi**ni ekleyin ya da servisi ücretli bir pakete taşıyın. |
| **Bu servis beklemede** | "Ücretsiz plan uygulamaları için organizasyona bir Git hesabı bağlanmalı. Bu uygulama, Hesap → Dış Girişler altından bir hesap bağlanana kadar uyur." | GitHub hesabınızı **Hesap → Dış Girişler** sayfasından bağlayın ya da kendi GitHub, GitLab veya Azure DevOps hesabınızı **Entegrasyonlar** sayfasından bağlayın. Ücretli paketler bu şartı aramaz. |
| **Bu servis kullanılamıyor** | "Hesap ödeme nedeniyle askıya alındığı için bu uygulama uyuyor. Hesap yeniden etkinleştirildiğinde uyanır." | Ödenmemiş ekstreyi **Hesap → Ekstreler** sayfasından ödeyin. |
| **Bu servis durduruldu** | "Bu uygulama elle uyutuldu; uyandırılana kadar replica sayısı sıfırda kalır." | Servis konsoldan uyutulmuştur ve gelen istekler onu uyandırmaz. Panodaki **Uyandır** ile açın. Açıklama "Bu uygulama platform ekibi tarafından uyutuldu" diyorsa destek talebi açın. |

**Uyanma denemesi başarısız olduysa** kartta **Sleep geçişi başarısız** görünür; **Uyandır** ile yeniden deneyin.

**Ücretsiz saatler hızlı tükeniyorsa** servise düzenli istek gelen bir şey (ör. bir izleme servisi ya da açık kalan bir tarayıcı sekmesi) onu uyanık tutuyor olabilir. Uyanık geçen her saat havuzdan düşer.

Ayrıntılar: [Uyku Modu](service-sleep.md).

---

## Bir Şey Oluşturamıyorum

| Gördüğünüz mesaj | Neden | Ne yapmalı |
|---|---|---|
| "Bu işlemle tahmini aylık harcamanız … USD olacak; mevcut harcama sınırınız … USD. …" | Yeni kaynak ya da plan yükseltmesi aylık tahmini harcama limitinin üstüne çıkarıyor. | Cüzdana bakiye yükleyin ya da kartınızı doğrulayın; limit bunlarla yükselir. **Hesap → Cüzdan** sayfasındaki **Limit artışı talep et** ile artış isteyebilirsiniz. |
| "Bir faturanız tahsil edilemediği için hesabınız ödemesiz dönemde. …" ya da "Ödenmemiş faturalarınız nedeniyle hesabınız askıya alındı. …" | Ödenmemiş bir ekstre var. | Ekstreyi **Hesap → Ekstreler** sayfasından ödeyin. |
| "Ücretsiz plan ile en fazla 1 servis yayınlayabilirsiniz. Daha fazla servis için ücretli bir plan seçin." | Organizasyonun ücretsiz servis hakkı dolu. | Yeni servis için ücretli bir paket seçin ya da mevcut ücretsiz servisi ücretli pakete taşıyın. |
| **Önce kişisel bir Git hesabı bağlayın** ya da **Bu Git hesabı başka bir organizasyonda kullanılıyor** | Ücretsiz plan kişisel bir Git hesabına bağlanır ve bir Git hesabı ücretsiz planı yalnızca tek bir organizasyonda kullanabilir. | **Git hesabı bağla** ile kişisel hesabınızı bağlayın, başka bir Git hesabı kullanın ya da ücretli bir paket seçin. |
| **Bu sayfaya erişiminiz yok** — "Rolünüzde … izni yok. Bir yöneticiden bu izni rolünüze tanımlamasını isteyin." | Rolünüzde gerekli izin yok. | Organizasyon yöneticisinden izni rolünüze eklemesini isteyin. |
| "Bu alan organizasyon yöneticileriniz tarafından yönetilir. Erişim gerekiyorsa onlardan birine başvurun." | Faturalandırma sayfaları yalnızca organizasyon yöneticilerine açıktır. | Bir yöneticiden yardım isteyin. |
| "Organizasyon oluşturmadan önce e-posta adresinizi doğrulayın. Gelen kutunuza gönderdiğimiz doğrulama bağlantısını açıp tekrar deneyin." | E-posta adresiniz doğrulanmamış. | Doğrulama e-postasındaki bağlantıyı açın; bağlantı 24 saat geçerlidir. E-posta gelmediyse doğrulama ekranındaki **Mail gelmedi mi? Tekrar gönder** ile yenisini isteyin. |
| "You have reached the maximum of 5 organizations per account." | Bir hesap en fazla 5 organizasyonda yer alabilir. | Bu sınır sabittir; mevcut organizasyonlarınızdan birini kullanın. |

> **Not:** Harcama limiti yalnızca yeni bir şey oluştururken ya da planı büyütürken kontrol edilir; zaten çalışan kaynaklar limit yüzünden durdurulmaz.

Ayrıntılar: [Faturalandırma](billing.md), [Organizasyonlar ve Ekip](organizations-team.md).

---

## Git Bağlantısı

| Gördüğünüz | Neden | Ne yapmalı |
|---|---|---|
| Sihirbazda "Bu hesap için depo döndürülmedi." ya da **Entegrasyonlar** sayfasında **Repo bulunamadı** | Bağlantının hiçbir repoya erişimi yok. GitHub App kurulumu **Seçili repolar** ile yapıldıysa yalnızca seçilen repolar görünür. | **GitHub App Kurulumları** listesindeki ayar düğmesiyle GitHub'da repoyu kuruluma ekleyin. GitLab token'ının `api` iznine, Azure DevOps PAT'ının Code (Read) ve Service Connections (Read, query, manage) yetkilerine sahip olduğunu kontrol edin. |
| **Repolar yüklenemedi** | Bağlantı kaldırılmış, askıya alınmış ya da kimlik bilgileri geçersiz olmuş olabilir. | Bağlantıyı yeniden bağlayın. |
| **Bu sağlayıcı henüz desteklenmiyor** | Bağlantı desteklenmeyen bir sağlayıcıya ait. | GitHub, GitLab ya da Azure DevOps bağlantısı kullanın. |
| "Bu kurulum GitHub'da kaldırıldı. Otomatik dağıtımı geri getirmek için yeniden bağlayın." | Komuta App GitHub tarafında kaldırıldı. | **Yeniden bağla**'ya basın. |
| "Bu kurulum GitHub'da askıya alındı. Erişimi geri getirmek için GitHub'da yeniden etkinleştirin." | Kurulum GitHub'da askıya alındı. | Kurulumu GitHub'da yeniden etkinleştirin. |
| **Token süresi doldu** | Azure DevOps PAT'ının süresi bitti; otomatik dağıtım build'leri klonlama sırasında başarısız olur. | **Token'ı güncelle** ile yeni bir PAT girin. GitLab bağlantılarında aynı işlem **Tokenı güncelle** ile yapılır. |

> **Uyarı:** Kullanımdaki bir Git bağlantısını kaldırmayın. Bağlantı kaldırılınca o repolardan oluşturulmuş servisler çalışmaya devam eder ama artık kaynak kodu çekemez ve bu bağlantıdaki otomatik dağıtım abonelikleri silinir.

Ayrıntılar: [Entegrasyonlar](integrations.md).

---

## Özel Alan Adı

**Ne görürsünüz:** **Alan Adları** sayfasında alan adı **Hazırlanıyor** ya da **İşlem Gerekli** durumunda kalır.

**Ne yapmalı:**

1. Alan adını açıp **Eklenecek DNS Kayıtları** listesine bakın. **Henüz tespit edilmedi**, **Farklı değer** ya da **DNS'te görünmüyor** yazan satırları DNS sağlayıcınızda ekleyin ya da düzeltin.
2. DNS değişikliğinin yayılmasını bekleyin; bu genellikle birkaç dakika, en fazla 24 saat sürer.
3. **Doğrulamayı Yenile**'ye basın.

**Sertifika verilmiyorsa** **SSL Durumu** satırına bakın:

| SSL Durumu | Anlamı ve yapılacak |
|---|---|
| **Doğrulama bekleniyor** | Sertifika doğrulama kaydı henüz bulunamadı. Kaydı ekleyin ya da düzeltin. |
| **Zaman aşımı** | Sağlayıcı beklemeyi bıraktı. DNS kayıtlarını kontrol edip doğrulamayı yenileyin. |
| **Süresi doldu** | Sertifika yenilenemedi. Sertifika doğrulama kaydını kontrol edip doğrulamayı yenileyin. |

Diğer durumlar:

- **Sağlayıcı Durumu** **DNS taşındı** gösteriyorsa DNS artık Komuta'yı göstermiyordur; trafik kaydını kontrol edin.
- **İncelemede** ya da **Engellendi** görüyorsanız destek talebi açın.
- Kök alan adında (ör. `example.com`) DNS sağlayıcınızın CNAME düzleştirmeyi (ALIAS ya da ANAME) desteklemesi gerekir. Bu durumda alan adı aktif olsa bile trafik yönlendirme satırı tespit edilmemiş görünebilir.

Ayrıntılar: [Özel Alan Adları](custom-domains.md).

---

## Veritabanı Bağlantısı

Servisiniz yönetilen bir PostgreSQL, Valkey ya da RabbitMQ örneğine bağlanamıyorsa sırayla şunları kontrol edin:

1. **Örnek hazır mı?** Örnek **Hazırlanıyor** durumundayken bağlantı bilgisi üretilmez; **Aktif** olmasını bekleyin.
2. **Bilgiler doğru mu?** Host, port, kullanıcı ve parolayı örneğin **Bağlantı bilgisini göster** alanından kopyalayın. PostgreSQL'de **Veritabanı parolasını rotate et** kullanıldıysa eski parola artık çalışmaz ve mevcut bağlantılar kopar.
3. **Değer servise ulaştı mı?** Bağlantı bilgisini ortam değişkeni olarak ekledikten sonra **Değişiklikleri Uygula**'ya basın.
4. **Doğru adresi mi kullanıyorsunuz?** Örneğin bir **Özel adres**i varsa (bağlantı dizelerinde **Bağlantı Dizesi (özel ağ)** olarak da görünür) Komuta'daki servislerinizden onu kullanın; bu adres her erişim modunda çalışır. **Ağ erişimi** **Yalnızca özel** ise genel adres yoktur ve Komuta dışından (bilgisayarınız, bir CI işi) bağlanılamaz. **Genel, IP izin listesi** modunda listede olmayan adresler reddedilir.
5. **Bağlantı sınırına mı takıldınız?** PostgreSQL'de eşzamanlı bağlantı sayısı plana göre sınırlıdır; uygulamanızın bağlantı havuzunu bu sınırın altında tutun.

Kendi uygulamalarınız arasındaki bağlantılar için servisin **Bağlantılar** sayfasındaki **Uygulama bağla**'yı kullanın; üretilen adresi **Ortam değişkeni olarak ekle** ile servise yazabilirsiniz.

Ayrıntılar: [PostgreSQL](managed-postgresql.md), [Bağlantılar](services-connections.md).

---

## Loglar, Metrikler ve Trace'ler Boş

**Loglar boşsa:**

- Loglar container çıktısından gelir. Uygulamanın standart çıktıya (stdout) ya da hata çıktısına (stderr) yazdığından emin olun.
- **Log bulunamadı** görüyorsanız zaman aralığını genişletin ve filtreleri temizleyin. Varsayılan aralık son 1 saattir.
- Uyuyan, bakımdaki ya da restart korumasıyla durdurulmuş bir servis yeni log üretmez.

**Metrik grafikleri boşsa** ("Seçili pencerede veri yok."): seçilen aralıkta çalışan bir pod olmayabilir. Servis uyuyorsa ya da yeni deploy edildiyse aralığı değiştirin.

**Trace'ler boşsa:**

- Otomatik izleme .NET, Java, Node.js ve Python'u destekler. Servisiniz başka bir dildeyse trace ekranındaki **OpenTelemetry kurulumu yap** ile OpenTelemetry SDK'sını uygulamanıza ekleyin.
- **Eşleşen trace dönmedi** sonucu hiç istek gelmediğini kanıtlamaz; örnekleme nedeniyle bazı istekler kaydedilmez. Zaman aralığını genişletin.
- "… yeni telemetriyi reddediyor" uyarısı servisin depolama sınırını aştığını gösterir; bu sürede gelen trace'ler saklanmaz. Sınırı **Hesap → Observability Depolama** sayfasından gözden geçirin.

Ayrıntılar: [Loglar](service-logs.md), [Gözlemlenebilirlik](observability.md).

---

## Yardım Alma

**Destek talebi.** Üst çubuktaki **Destek** simgesine (can simidi) tıklayın ya da [console.komuta.io/support](https://console.komuta.io/support) adresini açın ve **Ticket Oluştur** ile talep açın. Talebe şunları ekleyin:

- Servisin ya da kaynağın adı.
- Sorunun başladığı zaman (UTC).
- Ne yaptığınız ve gördüğünüz mesajın tam metni.
- Sunucu kaynaklı hatalarda bildirimde görünen **Destek referansı** (yanındaki **Kopyala** ile alabilirsiniz).

Ayrıntılar: [Destek](support-tickets.md).

**Platform durumu.** Komuta genelinde bir kesinti olup olmadığını [status.komuta.io](https://status.komuta.io) adresinden izleyebilirsiniz.

---

## İlgili Dokümanlar

- [Servis Dashboard](service-dashboard.md) — Servisin anlık durumu ve deploy işlemleri.
- [Pipeline'lar](service-pipeline-guide.md) — Build aşamaları, loglar ve otomatik Dockerfile onarımı.
- [Build Kuyruğu](build-queue.md) — Build'lerin neden beklediği.
- [Uyku Modu](service-sleep.md) — Ücretsiz servislerin uyuması ve uyanması.
- [Faturalandırma](billing.md) — Harcama limiti, cüzdan ve ekstreler.
- [Bant Genişliği Planları](bandwidth-plans.md) — Pod başına giden trafik hız sınırı.
- [Sınırlar ve Kotalar](limits.md) — Platform genelindeki sınırlar.
- [Destek](support-tickets.md) — Destek talebi açma ve takip.
