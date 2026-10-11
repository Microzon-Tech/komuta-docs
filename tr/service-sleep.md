# Servis Uykusu ve Her Zaman Açık

Ücretsiz paketteki bir web servisi bir süre trafik almazsa uyutulur: replika sayısı sıfıra çekilir ve servis kaynak harcamaz. Genel URL'ine ilk istek geldiğinde servis kendiliğinden uyanır. Organizasyonunuzdaki ücretsiz servisler her ay ortak bir uyanık saat havuzunu paylaşır; havuz biterse ücretsiz servisler ayın 1'ine kadar uyur.

Bu sayfa hangi servislerin ne zaman uyuduğunu, uyanırken ziyaretçinin ne gördüğünü, servisi elle uyandırmayı ve uyutmayı, ücretsiz saat havuzunu, ücretsiz servisler için Git hesabı şartını ve servisi sürekli açık tutan **Her zaman açık** eklentisini anlatır.

---

## Hangi Servisler Uyur?

Otomatik uyku yalnızca ücretsiz pakete özeldir. Ücretli paketteki servisler boşta kaldıkları için hiçbir zaman uyutulmaz.

| Servis | Boşta kalınca uyur mu? |
|---|---|
| Ücretsiz pakette, **Genel URL**'i açık web servisi | Evet. 30 dakika boyunca trafik almazsa uyutulur. |
| Ücretsiz pakette, **Genel URL**'i kapalı ya da [Özel Ağ (Mesh)](services-ports.md) üzerinden diğer servislere açılmış servis | Hayır. Bu servisleri çağıranlar onları uyandıramayacağı için otomatik uyutulmazlar; ama uyanık kaldıkları her saat havuzdan düşer. |
| **Her zaman açık** eklentisi açık olan ücretsiz servis | Hayır. Havuzdan da saat harcamaz. |
| Ücretli paketteki servis | Hayır. |
| Job ve CronJob | Hayır. Uyku yalnızca sürekli çalışan servisler içindir. |

**Trafik olarak ne sayılır?** Yalnızca servisin genel URL'ine dışarıdan gelen HTTP istekleri. Diğer servislerinizden özel ağ üzerinden gelen çağrılar, uygulamanın kendi yaptığı dış bağlantılar ve Komuta'nın sağlık kontrolleri servisi uyanık tutmaz.

**Ne zaman uyutulmaz?**

- Yeni oluşturulan ya da yeni dağıtılan servis bir süre uyutulmaz; servise ilk trafiğin gelmesi için zaman tanınır.
- Build veya dağıtım sürerken servis uyutulmaz.
- Servis yeni uyandıysa bir süre tekrar uyutulmaz.
- Trafik bilgisi okunamadığında servis o tur uyutulmaz. Kontrol düzenli aralıklarla yapıldığı için uyku, 30 dakikalık sessizlikten biraz sonra gerçekleşebilir.

> **Not:** Ücretsiz paket tek replika ile ve otomatik ölçekleme olmadan çalışır. Ücretsiz paketin diğer kuralları ve fiyatlar [Fiyatlandırma](https://www.komuta.io/pricing) sayfasındadır.

---

## Uyanırken Ziyaretçi Ne Görür?

Uyuyan servisin genel URL'ine gelen ilk istek servisi uyandırır. Uyanma genellikle yaklaşık bir dakika sürer; süre uygulamanızın açılış hızına da bağlıdır.

**Tarayıcıyla gelen ziyaretçi** servis hemen yanıt veremezse **Bu servis Uyanıyor** sayfasını görür: "Birkaç saniye içinde açılacak. Hazır olduğunda sayfa kendiliğinden yenilenir." Sayfa birkaç saniyede bir servisi kontrol eder ve servis hazır olunca istenen sayfaya kendisi geçer. Ziyaretçinin bir şey yapması gerekmez. Tarayıcı dili Türkçe değilse sayfa İngilizce gösterilir.

**API istemcileri ve diğer tarayıcı dışı çağrılar** için istek, servis hazır olana kadar bekletilir ve ardından servise iletilir. Servis bu sürede hazır olmazsa `503` yanıtı ve `Retry-After` başlığı döner. Zaman aşımı kısa olan istemciler (örneğin webhook gönderen servisler) uyanma sırasındaki ilk istekte hata alabilir; bu istemcilere yeniden deneme ekleyin ya da servisi [Her zaman açık](#her-zaman-açık-eklentisi) tutun.

**Erişim koruması açık servislerde** uyuyan servis, ziyaretçi korumayı geçtikten sonra uyandırılır. Giriş yapmamış ya da izin listesinde olmayan ziyaretçiler servisi uyandıramaz. Ayrıntılar: [Servis Erişim Koruması](service-access-protection.md).

> **Uyarı:** Diğer servislerinizden özel ağ üzerinden gelen çağrılar uyuyan bir servisi uyandırmaz. Uyuyan servis bu çağrılara yanıt veremez; servisi konsoldan uyandırın.

### Servis istekle uyanamıyorsa

Bazı durumlarda istek servisi uyandıramaz. Ziyaretçi o zaman kendiliğinden yenilenmeyen bir bilgi sayfası görür. Sayfa, ziyaretçinin dil tercihine göre Türkçe ya da İngilizce gösterilir.

| Ziyaretçinin gördüğü başlık | Neden |
|---|---|
| **Bu servis bu ay için uykuda** | Organizasyonun bu ayki ücretsiz saatleri doldu. |
| **Bu servis beklemede** | Ücretsiz plan bir Git hesabına bağlı değil. |
| **Bu servis durduruldu** | Servis konsoldan **Gelen istekle otomatik uyansın** işaretlenmeden uyutuldu ya da platform ekibi tarafından durduruldu. |
| **Bu servis kullanılamıyor** | Hesap askıya alındı. Ziyaretçiye nedeni gösterilmez. |

---

## Uyku Durumunu İzleme

**Uygulamalar listesi.** Uyuyan servisler [Uygulamalar](https://console.komuta.io/services) listesinde **Uyuyor**, uyanmakta olanlar **Uyanıyor** etiketiyle görünür. Durum filtrelerindeki **Uyuyor** seçeneği ikisini birlikte listeler. Uyanma başarısız olursa servis **Başarısız** olarak görünür.

**Servis Dashboard'u.** Servis uyurken [Dashboard](service-dashboard.md)'daki durum kartında **Uyku durumu** bölümü öne çıkar. Bölümün başlığı servisin hâlini söyler:

| Başlık | Anlamı |
|---|---|
| **Uygulama uykuda** | Servis uyuyor. Alt satır neden uyutulduğunu söyler (bkz. [Uyku Nedenleri](#uyku-nedenleri)). |
| **Uygulama uykuya geçiyor** | Servis uyutuluyor; birkaç saniye içinde uykuda olur. |
| **Uygulama uyandırılıyor** | Servis açılıyor. Hazır olduğunda bölüm kendiliğinden güncellenir. |
| **Sleep geçişi başarısız** | Son uyutma ya da uyandırma denemesi tamamlanamadı. Hata metni **Hata:** satırında görünür. |

Uyku bölümünde servisin uykuya girdiği an **Uykuya girdiği zaman** satırında yazar. Servis boşta kaldığı için uyutulduysa, uyutulmadan önceki 30 dakikada kaç istek sayıldığı da gösterilir; örneğin **Probe boşta-kalma penceresinde 0 istek gözledi**. "Ama trafiğim vardı" diye düşünüyorsanız önce bu satıra bakın.

**Bildirimler.** Servis uyuduğunda ve uyandığında bildirim almak için **Hesap → Bildirimler ve Uyarılar** sayfasındaki olay-kanal matrisinde **Uygulama Yaşam Döngüsü** grubundaki **Uygulama uyku moduna alındı** ve **Uygulama uyandı** olaylarını bir kanala bağlayın. Kanalların hazırlanması: [Kanallar ve Bildirim Teslimi](notification-guide.md).

---

## Servisi Elle Uyandırma

1. Servisin **Dashboard** sayfasını açın.
2. Durum kartındaki **Uyku durumu** bölümünde **Uyandır**'a basın.
3. Düğme **Uyandırılıyor…** olarak değişir ve "Uygulama uyandırılıyor..." bildirimi çıkar. Servis hazır olunca bölüm kaybolur.

**Uyandır** düğmesi servis uyurken ve **Sleep geçişi başarısız** durumunda görünür; başarısız bir geçişi yeniden denemek için de kullanılır. Uyandırma engellenirse düğme bir hata mesajı gösterir (bkz. [Sorun Giderme](#sorun-giderme)).

Servisi uyandırmanın başka yolları da vardır:

- **Yeni sürüm dağıtmak.** **Dağıt** ile ya da push ile tetiklenen bir dağıtım, uyuyan servisi önce uyandırır. Ortam değişkeni değişikliklerini uygulamak da servisi uyandırır.
- **Ücretli pakete geçmek.** Paket değişikliğinden sonra uyuyan servis kendiliğinden uyandırılır (bkz. [Uykuyu Tamamen Kapatma](#uykuyu-tamamen-kapatma)).

---

## Servisi Elle Uyutma

Ücretsiz paketteki bir servisi beklemeden de uyutabilirsiniz. Örneğin bir süre kullanmayacağınız servisin havuzdan saat harcamasını durdurmak için.

1. Servis sayfasının üst kısmında **Uyut (Hibernate)** düğmesine basın.
2. **{servis} uyutulsun mu?** penceresinde nasıl uyanacağını seçin:
   - Varsayılan: "Servis konsoldan uyandırana kadar uykuda kalır; gelen istekler onu uyandırmaz." Ziyaretçiler **Bu servis durduruldu** sayfasını görür.
   - **Gelen istekle otomatik uyansın** işaretliyse servis şimdi uyur ve ilk istekle, boşta kalınca uyutulmuş bir servis gibi uyanır.
3. **Uyut (Hibernate)** ile onaylayın. "Uyku isteği işlendi. Sonucu doğrulamak için servis durumunu kontrol edin." bildirimi çıkar.

Gelen istekle uyanmayacak şekilde uyutulmuş bir servisi **Uyandır** ile, yeni bir dağıtımla ya da ücretli pakete geçerek uyandırırsınız.

> **Not:** **Uyut (Hibernate)** düğmesi yalnızca ücretsiz paketteki, uyanık ve **Her zaman açık** eklentisi kapalı servislerde görünür. Bakım modundaki servislerde de görünmez.

> **Not:** Servis kısa süre önce uyandıysa ya da o sırada bir build veya dağıtım sürüyorsa uyutma isteği uygulanmaz. Bu yüzden onaydan sonra servisin durumunu kontrol edin; servis hâlâ uyanıksa biraz sonra yeniden deneyin.

---

## Uyku Nedenleri

**Uygulama uykuda** başlığının altındaki satır, servisin neden uyuduğunu söyler. Uyanma yolu nedene göre değişir.

| Kartta görünen metin | Neden | Nasıl uyanır? |
|---|---|---|
| "Bu uygulama enerji tasarrufu için uyutuldu; trafiği olmadığında replica sayısı sıfıra çekildi." | 30 dakika trafik gelmedi. | İlk istekle, **Uyandır** ile ya da dağıtımla. |
| "Bu uygulama elle uyutuldu; uyandırılana kadar replica sayısı sıfırda kalır." | Organizasyonunuzdan biri **Uyut (Hibernate)** kullandı. | **Uyandır** ya da dağıtım ile. **Gelen istekle otomatik uyansın** seçildiyse ilk istekle de. |
| "Bu ayın ücretsiz saatleri doldu; bu ücretsiz plan uygulaması havuz sıfırlanana kadar uyur." | Organizasyonun aylık ücretsiz saat havuzu bitti. | Ayın 1'inde havuz sıfırlanınca. Hemen açmak için **Her zaman açık** ya da ücretli paket. |
| "Ücretsiz plan uygulamaları için organizasyona bir Git hesabı bağlanmalı. Bu uygulama, Hesap → Dış Girişler altından bir hesap bağlanana kadar uyur." | Ücretsiz plan bir Git hesabına bağlı değil. | Git hesabı bağlanınca ya da servis ücretli pakete geçince. |
| "Hesap ödeme nedeniyle askıya alındığı için bu uygulama uyuyor. Hesap yeniden etkinleştirildiğinde uyanır." | Hesap ödenmemiş ekstreler nedeniyle askıya alındı. Ücretli servisler de bu nedenle uyuyabilir. | Bakiye ödenip hesap yeniden etkinleşince. |
| "Bu uygulama platform ekibi tarafından uyutuldu; replica sayısı sıfıra çekildi." | Komuta platform ekibi servisi durdurdu. | Platform ekibi tutmayı kaldırınca. Gerekirse [destek talebi](support-tickets.md) açın. |
| **Bakımda** başlığı | Servis bakım moduna alındı. | **Bakımı bitir** ile. |

### Uyku geçmişi

Konsol, servisin şu anki uyku nedenini ve boşta kalma nedeniyle uyutulduysa son karardaki istek sayısını gösterir. Servisin tüm uyku ve uyanma kayıtları konsolda ayrı bir sayfada listelenmez; bu geçmişi MCP ile bağlı kod ajanınıza sorabilirsiniz (bkz. [MCP ile Yönetim](#mcp-ile-yönetim)).

---

## Aylık Ücretsiz Saat Havuzu

Organizasyonunuzdaki ücretsiz servisler her ay toplam **450 uyanık saati** paylaşır. Havuz organizasyona aittir ve takvim ayına göre işler:

- Ücretsiz paketteki bir servisin uyanık kaldığı her dakika havuzdan düşer. Uyuyan servis saat harcamaz.
- **Her zaman açık** eklentili servisler, ücretli paketteki servisler ve Job/CronJob'lar havuzdan saat harcamaz.
- Havuz her ayın 1'inde 00:00 UTC'de sıfırlanır. Kullanılmayan saatler sonraki aya devretmez.
- Ay içinde silinen servislerin o ay kullandığı saatler havuzdan düşmüş sayılır.

> **Not:** Havuz takvim ayına göre (ayın 1'i, UTC) sıfırlanır. Fatura dönemi ise ayın 6'sından 6'sına işler; ikisi farklı tarihlerdir.

### Kullanımı nerede görürsünüz?

| Yer | Ne gösterir? |
|---|---|
| **Uygulamalar** listesinin üstündeki **Bu ayki ücretsiz saatler** çubuğu | Kullanılan ve toplam saat, kalan saat. Tıklayınca servis başına dağılım, **Diğer servisler** (silinmiş servisler dahil), sıfırlanma zamanı ve kullanım bu hızla sürerse havuzun ne zaman biteceği (**Bu hızla … biter**) açılır. |
| Listede her ücretsiz servisin yanındaki etiket | **Bu ay …** (servisin bu ay kullandığı saat), **Her zaman açık** ya da havuz bittiyse **Saat doldu**. |
| Servis Dashboard'undaki durum kartında **Ücretsiz saatler** bölümü | **Ücretsiz saat havuzu** başlığı altında organizasyonun kullanımı ile **Bu servis**, **Kalan** ve **Sıfırlanma** bilgileri. |
| Durum kartında **Ücretsiz saat kullanımı** bölümü | Servis son 24 saatin dörtte üçünden fazlasında uyanık kaldıysa ya da kullanım bu hızla sürerse havuz ay bitmeden tükenecekse **Bu servis ücretsiz saatleri yoğun kullanıyor** uyarısı. |

Tarihler ve saatler bu ekranlarda UTC olarak gösterilir. Değerin yanında **Tahmini** etiketi varsa o ayın kullanımının bir kısmı tahmine dayanır.

> **İpucu:** Bir izleme servisi (uptime monitor) ya da açık kalan bir tarayıcı sekmesi servise düzenli istek gönderiyorsa servis hiç uyumaz ve havuzu hızla tüketir. Servisin sürekli açık kalması gerekiyorsa [Her zaman açık](#her-zaman-açık-eklentisi) eklentisini kullanın.

### Uyarılar

| Kullanım | Konsolda | E-posta |
|---|---|---|
| %80 | Sayfanın üstünde "Ücretsiz saat havuzunda kullanım %80 seviyesine ulaştı" uyarısı. Havuz bittiğinde ücretsiz servislerin sıfırlanmaya kadar uyuyacağını hatırlatır. | "Bu ayın ücretsiz saatlerinin %80'ini kullandınız" |
| %100 | "Bu ayki ücretsiz saatleriniz doldu" uyarısı. | "Bu ayın ücretsiz saatleri doldu" |

Uyarıda **Kullanımı gör** düğmesi Uygulamalar listesine götürür; kapatma düğmesi uyarıyı o tarayıcı oturumu boyunca gizler. E-postalar her ay her eşik için bir kez, organizasyon yöneticilerine ve ücretsiz planın Git hesabını bağlayan kişiye gönderilir.

### Havuz bitince ne olur?

Havuz bittiğinde organizasyondaki ücretsiz servisler (**Her zaman açık** olanlar hariç) uyutulur ve ayın 1'ine (UTC) kadar uykuda kalır:

- Servisler istekle uyanmaz; ziyaretçiler **Bu servis bu ay için uykuda** sayfasını görür.
- **Uyandır** ile uyandırılamaz.
- Dağıtım ve servis değişiklikleri (ortam değişkenleri, portlar, replika sayısı gibi) reddedilir. Git push'ları build başlatmaz.

Bir servisi ay bitmeden açmanın iki yolu var:

- O servis için [Her zaman açık](#her-zaman-açık-eklentisi) eklentisini açın. Eklenti etkinleşince servis uyanır.
- Servisi [ücretli bir pakete taşıyın](#uykuyu-tamamen-kapatma). Ücretli pakete geçiş havuz bitmişken de yapılabilir.

Ayın 1'inde havuz sıfırlanınca engel kalkar. Genel URL'i olan servis gelen ilk istekle uyanır; diğer ücretsiz servisleri Komuta kendisi uyandırır. Dilerseniz **Uyandır** ile beklemeden de açabilirsiniz.

---

## Ücretsiz Servisler için Git Hesabı

Ücretsiz plan, organizasyona bağlı kişisel bir Git hesabıyla kullanılır (GitHub, GitLab ya da Azure DevOps). Komuta, ücretsiz servisin reposunun sahibi olan Git hesabını çoğu zaman kendisi bağlar; genellikle bir şey yapmanız gerekmez.

Bağ kurulamazsa (repo sahibi repo adresinden okunamazsa ya da o hesap ücretsiz planı başka bir organizasyonda kullanıyorsa) konsolun üstünde **Ücretsiz servisleriniz bir Git hesabına bağlanamadı** uyarısı çıkar. Bu durumda organizasyondaki ücretsiz servisler uyutulur, istekle ya da **Uyandır** ile uyanmaz ve dağıtılamaz. Ziyaretçiler **Bu servis beklemede** sayfasını görür. Ücretli paketteki servisler etkilenmez.

Hesabı bağlamak için:

1. Uyarıdaki **Git hesabı bağla** düğmesine basın ya da **Hesap → Dış Girişler** sayfasını açın.
2. **Ücretsiz plan Git hesabı** kartında, **Git hesaplarınız** listesinden kişisel hesabınızın yanındaki **Bu hesapla bağla**'yı seçin.
3. Listede uygun hesap yoksa **GitHub'ı bağla** ile GitHub hesabınızı onaylayın ya da **Entegrasyonları aç** ile GitLab veya Azure DevOps hesabınızı bağlayıp bu sayfaya dönün. Git hesabı bağlama adımları: [Entegrasyonlar](integrations.md).

Hesap bağlanınca engel kalkar; servisler istekle ya da Komuta tarafından uyandırılır.

- Yalnızca kişisel hesaplar kullanılabilir. Organizasyon hesapları ve kendi sunucunuzda barındırdığınız Git sunucuları ücretsiz planı taşıyamaz.
- Bir Git hesabı ücretsiz planı yalnızca bir organizasyonda kullanabilir.
- Bağı yalnızca bağlayan kişi kaldırabilir. Kaldırılan hesap 30 gün boyunca ve en erken ayın sonuna kadar başka bir organizasyonun ücretsiz planı için kullanılamaz. Bağ kaldırılınca organizasyondaki **Her zaman açık** eklentileri de sona erer.

---

## Her Zaman Açık Eklentisi

**Her zaman açık**, ücretsiz paketteki tek bir servisi sürekli uyanık tutan ücretli bir eklentidir. Servis trafik olmasa da uyumaz ve ücretsiz saat havuzundan saat harcamaz. Havuz bitmişken açarsanız servis uyanır ve çalışmaya devam eder.

### Fiyat ve ücretlendirme

- Fatura dönemi başına **3 USD + vergi**. Fiyat, eklenti kartında ve onay penceresinde gösterilir. İleri bir tarihte fiyat değişecekse kart yeni fiyatı ve geçerlilik tarihini önceden gösterir.
- Ücretlendirme servis uyanıp çalışmaya başlayınca başlar. Her fatura döneminde yalnızca eklentinin açık kaldığı kısım kadar ödersiniz.
- Kupon ve kampanya indirimleri uygulanmaz.
- Eklentiyi kapattığınız an ücretlendirme durur.
- Eklentinin tutarı aylık harcama limitine dahildir. Limit kontrolü: [Faturalandırma](billing.md).

### Şartlar

Eklenti yalnızca ücretsiz paketteki web servisleri için alınabilir; ücretli paketteki servislerde, Job ve CronJob'larda kart görünmez. Bir şart eksikse kartta **Açmak için:** satırının altında yapılması gereken gösterilir:

| Kartta görünen | Ne yapmalı? |
|---|---|
| **Tek replika kullanın, otomatik ölçeklendirme kapalı** | Servis tek replikayla ve otomatik ölçekleme kapalı çalışmalı. Bkz. [Otomatik Ölçekleme](services-autoscaling.md). |
| **Doğrulanmış varsayılan kart ekleyin** | **Hesap → Ödeme Yöntemleri** sayfasından doğrulanmış bir kartı varsayılan yapın. |
| **Ücretsiz plana GitHub bağlayın** | Organizasyonun ücretsiz planı bir Git hesabına bağlı olmalı. Bkz. [Ücretsiz Servisler için Git Hesabı](#ücretsiz-servisler-için-git-hesabı). |
| **Askıyı kaldırmak için bakiyeyi ödeyin** | Hesap askıda. **Hesap → Ekstreler** sayfasından ödenmemiş ekstreleri ödeyin. |
| **Gecikmiş faturayı ödeyin** | Gecikmiş bir ödeme var. **Hesap → Ekstreler** sayfasından ödeyin. |
| **Harcama limitinizi yükseltin** | Eklenti bu ayki tahmini harcamayı limitin üstüne çıkarıyor. Limitiniz **Hesap → Cüzdan** sayfasındadır. |
| **Platform tutmasının kaldırılmasını bekleyin** | Servis platform ekibi tarafından uykuda tutuluyor. |

Eklentiyi organizasyon yöneticileri ve bu yetkiye sahip üyeler açıp kapatabilir. Yetkiniz yoksa kartta "Her zaman açık ayarını değiştirmek için organizasyonunuzdaki bir yöneticiye başvurun." yazar.

### Açma

1. Servisin **Dashboard** sayfasında durum kartının **Her zaman açık** bölümüne geçin. Servis uyuyorsa **Uyku durumu** bölümündeki **Uygulamam hep ayakta kalsın** bağlantısı da sizi buraya getirir.
2. Karttaki anahtarı açın.
3. **{servis} için Her zaman açık açılsın mı?** penceresinde fiyatı ve koşulları okuyun, **Aç** ile onaylayın.
4. Kartta **Açılıyor…** etiketi görünür: "Servis uyandırılıyor. Ücretlendirme servis çalışmaya başlayınca başlar."
5. Servis çalışmaya başlayınca etiket **Açık** olur ve kart eklentinin hangi tarihten beri açık olduğunu gösterir. Uygulamalar listesinde servisin etiketi **Her zaman açık** olur.

İşlem uzun sürerse kartta "Bu işlem her zamankinden uzun sürüyor. Arka planda denemeye devam ediyor; servis çalışmaya başlayana kadar ücret alınmaz." yazar; **Tekrar kontrol et** ile durumu yenileyebilirsiniz. Etkinleştirme tamamlanamazsa ücret alınmaz ve kartta "Son açma denemesi tamamlanamadı, bu yüzden ücret alınmadı. Yeniden deneyebilirsiniz." yazar.

### Kapatma

1. **Her zaman açık** bölümündeki anahtarı kapatın.
2. **{servis} için Her zaman açık kapatılsın mı?** penceresinde **Kapat**'a basın.

Ücretlendirme hemen durur. Servis normal uyku döngüsüne döner: boşta kalınca uyur ve yeniden havuzdan saat harcar. Bu ayın ücretsiz saatleri dolmuşsa servis hemen uyutulur.

### Eklenti ne zaman kendiliğinden sona erer?

- Servis ücretli bir pakete taşındığında (ücretli paketteki servis zaten uyumaz).
- Servis silindiğinde.
- Ücretsiz planın Git hesabı bağı kaldırıldığında.

> **Uyarı:** **Her zaman açık**, servisi hesabın askıya alınmasına ya da platform ekibinin durdurmasına karşı korumaz. Eklenti açıkken servis elle uyutulamaz ve bakım moduna alınamaz; önce eklentiyi kapatın.

---

## Uykuyu Tamamen Kapatma

Servisin hiç uyumamasını istiyorsanız ücretli bir pakete taşıyın. Ücretli paketteki servis boşta kaldığı için uyutulmaz ve ücretsiz saat havuzundan saat harcamaz.

1. Servisin **Dashboard** sayfasında **Plan / paket** kartındaki **Yükselt** düğmesine basın.
2. **Kaynak Planı** penceresinde ücretli bir paket seçin.
3. **Planı Uygula** ile onaylayın.

Servis uyuyorsa paket değişikliğinden sonra kendiliğinden uyandırılır. Serviste **Her zaman açık** eklentisi varsa sona erer. Paket fiyatları kullanılan CPU ve belleğe göre belirlenir; örneğin 0,5 vCPU / 1 GB paket ayda 9 USD'dir. Tüm fiyatlar [Fiyatlandırma](https://www.komuta.io/pricing) sayfasındadır.

> **Not:** Paket yükseltirken aylık tahmini harcama, organizasyonun harcama limitini aşacaksa işlem yapılmaz. Bkz. [Faturalandırma](billing.md).

**Her zaman açık mı, ücretli paket mi?** Ücretsiz paketin kaynakları yeterliyse ve yalnızca servisin uyumamasını istiyorsanız **Her zaman açık** daha ucuzdur. Daha fazla CPU, bellek, birden çok replika ya da otomatik ölçekleme gerekiyorsa ücretli pakete geçin.

---

## MCP ile Yönetim

MCP ile bağlı kod ajanınız servisin uyku durumunu ve uyku/uyanma geçmişini okuyabilir, servisi uyandırabilir ya da uyutabilir, organizasyonun ücretsiz saat kullanımını görebilir ve **Her zaman açık** eklentisini açıp kapatabilir. Uyutma ve eklentiyi açma gibi değişiklikler sizin onayınızla yapılır. Kurulum ve araçlar: [MCP Kurulumu](mcp-setup.md), [MCP Araçları](mcp-tools.md).

---

## Sorun Giderme

| Belirti | Olası neden | Ne yapmalı? |
|---|---|---|
| Ziyaretçi **Bu servis bu ay için uykuda** görüyor | Ücretsiz saat havuzu doldu. | Ayın 1'ini bekleyin ya da servise **Her zaman açık** ekleyin veya ücretli pakete taşıyın. |
| Ziyaretçi **Bu servis beklemede** görüyor | Ücretsiz plan bir Git hesabına bağlı değil. | **Hesap → Dış Girişler** sayfasından Git hesabı bağlayın. |
| Ziyaretçi **Bu servis durduruldu** görüyor | Servis istekle uyanmayacak şekilde elle uyutuldu. | Konsolda **Uyandır**'a basın. |
| **Bu servis Uyanıyor** sayfası uzun süre değişmiyor | Uygulama açılmıyor ya da sağlık kontrolünü geçemiyor. | Dashboard'da **Sleep geçişi başarısız** ve **Hata:** satırına, ardından [loglara](service-logs.md) bakın. |
| **Sleep geçişi başarısız** | Servis zamanında hazır olmadı ya da geçiş tamamlanamadı. | **Uyandır** ile yeniden deneyin. Sorun uygulamadaysa düzeltilmiş sürümü dağıtın; dağıtım, uyanma başarısız olsa da devam eder. |
| Diğer servislerden gelen çağrılar uyuyan servise ulaşmıyor | Özel ağ çağrıları uyuyan servisi uyandırmaz. | Servisi **Uyandır** ile açın. Sürekli açık kalması gerekiyorsa **Her zaman açık** ya da ücretli paket kullanın. |
| Servis hiç uyumuyor, havuz hızla bitiyor | Düzenli gelen istekler (izleme servisi, açık sekme) ya da servisin **Genel URL**'i kapalı veya özel ağa açık. | Durum kartındaki **Ücretsiz saat kullanımı** uyarısına ve listedeki servis etiketine bakın. İsteği gönderen aracı durdurun ya da **Her zaman açık** kullanın. |
| **Uyut (Hibernate)** sonrası servis uyumadı | Servis az önce uyanmıştı ya da build/dağıtım sürüyordu. | Biraz bekleyip yeniden deneyin. |
| Webhook ya da API istemcisi ilk istekte zaman aşımına düşüyor | Servis uyuyordu; uyanma yaklaşık bir dakika sürer. | İstemciye yeniden deneme ekleyin ya da **Her zaman açık** kullanın. |

**Uyandır'a basınca çıkan hata mesajları:**

- "Ücretsiz planın bu ayki uyanık saatleri bitti; bu servis gelecek ayın 1'ine (UTC) kadar uykuda kalır. Hemen uyandırmak için servise Her zaman açık eklentisini ekleyin ya da servisi ücretli bir plana taşıyın."
- Git hesabı eksikse mesaj, ücretsiz plandaki servisler için bağlı bir Git hesabı gerektiğini söyler ve şöyle biter: "Servisi uyandırmak için kişisel Git hesabınızı bağlayın ya da servisi ücretli bir plana taşıyın."
- "Hesabınız ödenmemiş ekstreler nedeniyle askıdayken bu servis uyandırılamaz. Bakiyenizi ödediğinizde servis yeniden uyanır."
- "Bu servis bir platform yöneticisi tarafından uykuda tutuluyor; tutma kaldırılana kadar uyandırılamaz."

**Havuz bitmişken dağıtım ya da ayar değişikliği yapınca:** "Ücretsiz planın bu ayki uyanık saatleri bitti; bu servisteki değişiklikler gelecek ayın 1'ine kadar engellendi. Hemen değişiklik yapmak için servise Her zaman açık eklentisini ekleyin ya da servisi ücretli bir plana taşıyın."

---

## İlgili Dokümanlar

- [Servis Dashboard](service-dashboard.md) — Durum kartı ve **Plan / paket** kartı.
- [Erişim ve Portlar](services-ports.md) — Genel URL ve özel ağ ayarları.
- [Servis Erişim Koruması](service-access-protection.md) — Korumalı servislerin uyandırılması.
- [Otomatik Ölçekleme](services-autoscaling.md) — Replika sayısı ve otomatik ölçekleme.
- [Faturalandırma](billing.md) — Fatura dönemi, ekstreler ve harcama limiti.
- [Entegrasyonlar](integrations.md) — Git hesabı bağlama.
- [Kanallar ve Bildirim Teslimi](notification-guide.md) — Uyku ve uyanma bildirimleri için kanal kurma.
- [MCP Araçları](mcp-tools.md) — Kod ajanıyla servis yönetimi.
