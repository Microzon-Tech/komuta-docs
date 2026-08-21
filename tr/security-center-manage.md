# Yönet: Bildirim ve Saklama

Güvenlik Merkezi'nin **Yönet** grubu, SOC'un nasıl haberdar edileceğini (bildirim kanalları ve yönlendirme kuralları), denetim kaydının nereye ve ne kadar süreyle tutulacağını (denetim depolama / WORM) ve platformun veri saklama politikasını (saklama süreleri, veri ikamet bölgesi, yasal tutma) yönettiğiniz üç sayfadan oluşur.

---

## Bildirimler

Bildirimler sayfası, güvenlik bulgularının hangi dış sistemlere (Slack, e-posta, PagerDuty vb.) iletileceğini tanımlayan **bildirim kanallarını** listeler. Bazı roller sadece kanal listesini ve durumlarını görebilir; **Yeni kanal**, **Düzenle** ve **Sil** aksiyonları güvenlik operatörü rolüne açıktır.

> **Not:** Kanal sırları (Slack webhook URL'si, PagerDuty yönlendirme anahtarı, webhook HMAC gizli anahtarı) salt-okunur kullanıcılara asla gösterilmez — sunucu tarafı bu alanları salt-okunur çağrılarda boş bir yapılandırmayla döndürür. Bir kanalı yönetme yetkiniz yoksa, kanalın var olduğunu ve etkin olup olmadığını görürsünüz ama içindeki sırları göremezsiniz.

### Kanal türleri

Bir kanal oluşturduğunuzda kanal türü seçilir; her tür kendi yapılandırma alanlarını ister:

| Kanal türü | Yapılandırma alanları | Not |
|---|---|---|
| Uygulama içi (SignalR) | Yok | Bağlı yönetici oturumlarına anlık bildirim gösterir; ek yapılandırma gerekmez. |
| E-posta | Alıcılar (satır başına bir e-posta, en fazla 32), konu ön eki (opsiyonel) | Boş satırlar sunucu tarafında otomatik atılır. |
| Slack | Webhook URL (`https://` ile başlamalı), kanal (örn. `#alerts-soc`) | |
| Microsoft Teams | Webhook URL (`https://` ile başlamalı) | |
| PagerDuty | Yönlendirme anahtarı (32 karakterlik hex) | Alan parola tipinde gösterilir (maskelenir). |
| Webhook | Uç nokta URL'si (`https://` ile başlamalı), HMAC gizli anahtarı (opsiyonel) | Platform her payload'ı bu anahtarla `X-Komuta-Signature` başlığında HMAC-SHA256 ile imzalar. |

Bir kanal oluşturmak için:

1. **Yeni kanal** düğmesine tıklayın.
2. **Ad**, isteğe bağlı bir **açıklama** ve **kanal türü** girin.
3. Seçilen türe göre görünen alanları (webhook URL'si, alıcılar, yönlendirme anahtarı vb.) doldurun.
4. **Etkin** anahtarını açık bırakın — kapalı kanallar dağıtıcı (dispatcher) tarafından atlanır.
5. **Kanal oluştur**'a tıklayın.

> **Uyarı:** Bir kanalı sildiğinizde dağıtıcı o kanala göndermeyi anında durdurur. Bu kanalı kullanan yönlendirme kurallarında ilgili kanal artık eşleşmez; kural, kalan kanallarla (varsa) çalışmaya devam eder.

Kanal türü, oluşturulduktan sonra değiştirilemez — bir kanalın türünü değiştirmeniz gerekiyorsa yeni bir kanal oluşturup eskisini silmeniz gerekir.

### Yönlendirme Kuralları'na erişim

Bildirimler sayfasının üst kısmındaki başlık kartında bir **Yönlendirme kuralları** bağlantısı bulunur. Yönlendirme Kuralları artık sol menüde ayrı bir öğe değildir; sadece bu bağlantı üzerinden, Bildirimler sayfasının bir alt sayfası olarak açılır.

### Kritik bulgu teslimat kapısı

Bildirimler ve Yönlendirme Kuralları sayfalarının her ikisinde de bir **kritik bulgu teslimat kapısı** kartı görünür. Bu kart, "Kritik önemdeki bir bulgu gerçekten birine ulaşabilir mi?" sorusuna platformun kendi değerlendirmesini gösterir ve üç durumdan birinde olur:

| Durum | Anlamı |
|---|---|
| Hazır | En az bir her zaman açık (7/24), filtresiz kural, etkin bir dış (uygulama içi olmayan) kanala bağlı. Kart, kaç kuralın kaç kanala ulaştığını ve başarısız teslimatların kaç dakika arayla, kaç saat boyunca yeniden denendiğini gösterir. |
| İstisna kabul edildi | Bir yönetici, eksik teslimat yolunu geçici olarak kabul etmiş. Kartta kabul eden kişi, kabul nedeni ve son geçerlilik zamanı görünür. |
| Kurulum gerekli | Kapsayan bir kanal veya kural yok. Kart, eksik olan koşulu (örn. "hiçbir dış kanal etkin değil" veya "her zaman açık, filtresiz bir kural yok") listeler. |

**Kurulum gerekli** durumundayken güvenlik operatörü rolündeki kullanıcılar doğrudan kart üzerinden **Kanal oluştur** / **Kural oluştur** kısayollarını kullanabilir, ya da riski geçici olarak kabul edebilir.

> **Uyarı:** "Riski geçici olarak kabul et" işlemi, kritik bulguların bu süre boyunca hiçbir operatöre ulaşmayabileceğini bilerek onaylamanız demektir. En az 12 karakterlik bir gerekçe girmeniz zorunludur (1 saat, 4 saat, 24 saat, 3 gün veya 7 gün süreyle geçerli olacak şekilde seçilebilir) ve bu karar kime ait olduğu, ne zaman verildiği ve nedeniyle birlikte denetim kaydına işlenir. İstisnayı **İstisnayı geri al** ile istediğiniz zaman erken sonlandırabilirsiniz; geri aldığınızda kapı, eksiksiz bir kritik yol kurulu değilse yeniden "Kurulum gerekli" durumuna döner.

---

## Yönlendirme Kuralları (Bildirimler sayfasındaki bağlantıdan açılır)

Kanallar ve yönlendirme kuralları aynı SOC yapılandırma yüzeyinin iki parçasıdır; ikisi de aynı role açıktır.

Bir yönlendirme kuralı, bir bulgunun hangi koşullarda hangi bildirim kanallarına gönderileceğini tanımlar. Bir bulgu, kuralın **tüm** filtrelerini geçmeden ve **şiddet eşiğini** karşılamadan kurala göre gönderim yapılmaz. Boş bırakılan filtreler "herhangi biri" anlamına gelir.

> **İpucu:** Yeni bir kural oluşturmak için en az bir bildirim kanalı olması gerekir. Hiç kanal yoksa **Yeni kural** düğmesi devre dışı kalır ve sayfa sizi önce bir kanal oluşturmaya yönlendirir.

### Kural alanları

| Alan | Açıklama |
|---|---|
| Ad / Açıklama | Kuralın kimliği ve amacı (açıklama isteğe bağlı, en fazla 512 karakter). |
| Şiddet eşiği | Bulgunun bu değere **eşit veya üzerinde** olması gerekir: Info, Low, Medium, High, Critical. |
| Kaynak filtresi | Belirli bir bulgu kaynağıyla (örn. Policy Guard, Runtime Sensor, Network Visibility, Posture, Identity) sınırlar; boş = herhangi bir kaynak. |
| Küme filtresi | Belirli bir kümeyle sınırlar (küme seçici sadece host yöneticileri için doldurulur); boş = herhangi bir küme. |
| Servis filtresi | Belirli bir servisle sınırlar; boş = herhangi bir servis. |
| Sahip takım filtresi | Belirli bir sahip takım etiketiyle sınırlar (en fazla 64 karakter); boş = herhangi bir takım. |
| İş saatleri | Açıksa kural sadece Pazartesi–Cuma, tenant yerel saatiyle 09:00–18:00 arasında tetiklenir. |
| Bildirim kanalları (birincil) | Kuralın ilk dalgada göndereceği kanallar; en az 1, en fazla 16 kanal seçilebilir. |
| Yükseltme (opsiyonel) | Belirli bir dakika sonra, birincil dalga onaylanmazsa ikinci bir kanal kümesine gönderim yapar. |
| Etkin | Kapalıysa dağıtıcı bu kuralı atlar. |

### Yükseltme (escalation) nasıl çalışır

Bir kuralda **Yükseltmeyi etkinleştir** açıldığında iki alan daha ortaya çıkar:

1. **Yükseltme gecikmesi (dakika):** 1 ile 1440 dakika (24 saat) arasında bir değer.
2. **Yükseltme kanalları:** Birincil kanallardan farklı olabilecek, ayrı bir kanal seçimi (en az 1 kanal zorunlu).

Birincil dalga tanımlanan süre içinde onaylanmazsa (acknowledge edilmezse), platform ikinci dalgayı yükseltme kanallarına gönderir. Yükseltme kapatılırsa, formda girilmiş olsa da gecikme ve kanal değerleri kaydedilmeden temizlenir.

### Bir kural oluşturmak veya düzenlemek

1. **Yeni kural** düğmesine tıklayın (veya var olan bir kuralın **Düzenle** simgesine).
2. Ad, açıklama ve şiddet eşiğini girin.
3. Gerekiyorsa kaynak, küme, servis ve sahip takım filtrelerini ayarlayın — hiçbirini seçmezseniz kural her bulguya uygulanır.
4. En az bir bildirim kanalı seçin.
5. İsterseniz yükseltmeyi açıp gecikme süresini ve yükseltme kanallarını belirleyin.
6. İsterseniz **İş saatleri** anahtarını açın.
7. **Kural oluştur** / **Değişiklikleri kaydet**'e tıklayın.

> **Uyarı:** Bir yönlendirme kuralını silmek anında etkilidir ve dağıtıcı o kural üzerinden gönderim yapmayı hemen durdurur. Özellikle kritik bulgu teslimat kapısını "Hazır" durumda tutan tek kapsayan kuralı siliyorsanız, kapı yeniden "Kurulum gerekli" durumuna dönebilir.

---

## Denetim Depolama

Bu sayfayı görüntülemek tenant yöneticilerine açıktır — kendi kiracınızın denetim koruma durumunu doğrulayabilirsiniz. Arka ucu değiştirme/yapılandırma ise sadece platform yöneticisine açıktır.

Bu sayfa, kurcalamaya karşı korumalı (tamper-evident) denetim kaydının sadece platform veritabanında mı, yoksa ayrıca değiştirilemez bir **WORM depolama** (Write Once Read Many — bir kez yazılır, çok kez okunur) arşivinde de mi tutulduğunu gösterir ve yapılandırır.

### Depolama arka ucu

| Arka uç | Açıklama |
|---|---|
| Yalnızca veritabanı | Denetim girdileri sadece platform veritabanında tutulur. Bu, dış değişmezlik (external immutability) şartını karşılamaz ve **korumasız** kabul edilir. |
| Nesne kilitli arşiv (Object-Lock) | Her denetim girdisi ayrıca değişmez bir nesne kilitli kovaya (bucket) yazılır. Saklama süresi dolmadan veya yasal tutma kaldırılmadan hiçbir kullanıcı bu girdileri silemez. |

Bir tenant'ın profili host'tan devralınmışsa (kendi override'ı yoksa) sayfada **Devralınan platform varsayılanı** etiketi görünür.

Arka ucu değiştirmek veya yapılandırmak için (yalnızca host yöneticileri):

1. **Yapılandırmayı değiştir** düğmesine tıklayın.
2. Arka uç olarak **Yalnızca veritabanı** veya **Nesne kilitli arşiv**'i seçin.
3. Nesne kilitli arşiv seçtiyseniz şu alanları doldurun: **Arşiv kovası (bucket) adı**, **Bölge**, **Saklama süresi (gün)** (varsayılan ≈2555 gün / ~7 yıl), isteğe bağlı **Anahtar öneki** ve isteğe bağlı **Özel uç nokta** (AWS dışında S3 API uyumlu bir servise işaret ediyorsanız).
4. **Kaydet**'e tıklayın.

> **Uyarı:** Nesne kilitli arşive geçiş, yalnızca bundan sonraki denetim girdilerini yansıtmaya başlar. Geçmişteki girdiler platform veritabanında kalır ve **geriye dönük olarak arşive kopyalanmaz.**

### Arşiv doğrulama durumu

Platform, arşive yazılan girdileri periyodik olarak geri okuyarak kanonik içerikle eşleştiğini, COMPLIANCE saklama modunun uygulandığını ve doğrulamanın güncel olduğunu kanıtlar. Bu doğrulama şu durumlardan birinde olabilir:

| Durum | Anlamı |
|---|---|
| Doğrulandı | Güncel yapılandırma başarıyla kanıtlandı; harici kopya sağlıklı. |
| Doğrulama bekliyor / Yetişiyor | Platform güncel yapılandırmayı kanıtlıyor veya arşiv birikimini kapatıyor; kaç girdinin kaldığı gösterilir. |
| Korumasız | Güncel bir geri okuma kanıtı yok; son hata kodu ve ayrıntısı kartta gösterilir. |

Kart ayrıca son kanıt zamanını, "şu tarihe kadar saklanır" bilgisini ve bitişik doğrulanmış sıra numarası / gözlenen son sıra numarası kontrol noktasını gösterir. Eksik bir arşiv nesnesi otomatik olarak onarılıp yeniden doğrulanmışsa "Arşiv boşluğu onarıldı" bildirimi görünür.

### Yasal tutma (legal hold)

Yasal tutma, yalnızca **Nesne kilitli arşiv** arka ucu seçiliyken kullanılabilir. Etkinleştirildiğinde, tutma aralığında oluşan (geciken arşiv art dolgusu dahil) her denetim girdisi silinmeye karşı sabitlenir.

> **Uyarı:** Yasal tutmayı etkinleştirmek geri alınamaz bir "tutma" işlemidir ve girdiği **gerekçe metni denetim zincirine olduğu gibi (verbatim) yazılır.** Yasal tutmayı yönetmek platform yöneticisine özeldir; ayrıca profilin host'tan devralınmamış olması (kendi tenant override'ınız olması) gerekir.

> **Uyarı:** Yasal tutmayı **kaldırmak (release)** sadece bundan sonra arşive yazılacak girdiler için tutmayı durdurur. Tutma etkinken zaten yazılmış nesneler tutulu kalır ve serbest bırakılmaları için arşive özgü (native) bir serbest bırakma iş akışı gerekir; COMPLIANCE saklama süresi bu nesneler üzerinde uygulanmaya devam eder.

Yasal tutma etkinken sayfanın üstünde kalıcı bir "Yasal tutma etkin" uyarı şeridi görünür; bu şerit yasal tutmanın tutma aralığındaki tüm yeni arşiv girdilerini, yapılandırılan saklama süresinden bağımsız olarak koruduğunu hatırlatır.

---

## Saklama Politikası

Bu sayfa Yönet grubunun en yüksek yetki gerektiren sayfasıdır: diğer iki sayfadan farklı olarak **sayfayı görüntülemek için bile** platform yöneticisi rolü gerekir — bir güvenlik operatörünün görüntüleme yetkisi burada yeterli değildir.

Bu sayfa; ham/normalize edilmiş olay, bulgu, agregasyon ve denetim verilerinin ne kadar süreyle saklanacağını, verinin hangi bölgede tutulacağını ve tenant için yasal tutmanın açık olup olmadığını yönetir. Politika her zaman etkin (ambient) tenant'a bağlı olarak okunur ve güncellenir.

> **Not:** Host bağlamındaysanız (bir tenant'a geçiş yapmadıysanız) düzenleme devre dışıdır — sayfa "Bir tenant'ın saklama politikasını düzenlemek için o tenant'a geçin" ipucunu gösterir. Host kapsamının kendine ait bir politikası yoktur.

### Profil ön ayarı

| Profil | Açıklama |
|---|---|
| Varsayılan (Default) | API'nin şu anda döndürdüğü platform varsayılanını kullanır. |
| Düzenlenmiş (Regulated) | API'nin şu anda döndürdüğü düzenlenmiş (regulated) ön ayarı kullanır. |
| Özel (Custom) | Alan bazında saklama süresi geçersiz kılmalarına izin verir; sadece düzenleyici/regülatör kaynaklı özel bir pencere şartınız varsa kullanın. |

Profil **Özel** olarak seçilmediği sürece aşağıdaki saklama süresi alanları kilitlenir ve düzenlenemez; gerçek değerler etkin dataset sözleşmesi tablosundan okunur.

### Saklama pencereleri

| Alan | Birim |
|---|---|
| Ham olay günü | Gün |
| Normalize edilmiş olay günü | Gün |
| Bulgu günü | Gün |
| Agregasyon ayı | Ay |
| Denetim yılı | Yıl |

### Veri ikamet bölgesi ve yasal tutma

- **Veri ikamet bölgesi:** Tenant güvenlik verisinin hangi bölgede tutulacağını sabitler (örn. EU, EU-DE, TR, US, GLOBAL). Boş bırakılırsa platform tarafında bölge zorlaması uygulanmaz.
- **Yasal tutma:** Aşağıdaki "Etkin dataset sözleşmesi" tablosunda tutma kapsamında (hold-covered) işaretlenmiş, tenant'a özgü dataset'ler için silmeyi (purge) askıya alır.

> **Uyarı:** Bu sayfadaki yasal tutma anahtarı, **tüm** veri kümelerini korumaz. Global TTL'e bağlı dataset'ler (örneğin ham ağ akışı) bu tutmanın kapsamı dışındadır — sayfa bunu açıkça "Global ham ağ akışı TTL'i bu tutma kapsamında değildir" şeklinde belirtir. Hangi dataset'in tutma kapsamında olduğunu görmek için aşağıdaki tabloyu kullanın.

### Etkin dataset sözleşmesi

Sayfanın alt kısmındaki tablo, canlı API yapılandırmasından gelen gerçek (etkin) değerleri gösterir — profil alanlarını fiziksel bir saklama garantisi olarak kullanmayın:

| Sütun | Anlamı |
|---|---|
| Dataset | Veri kümesinin adı (ör. ham çalışma zamanı olayları, normalize edilmiş güvenlik olayları, analitik bulgular, denetim karar kaydı). |
| Depo | Verinin hangi tür veri deposunda tutulduğu. |
| Etkin pencere | Şu anda uygulanan saklama süresi ve varsa host tavan değeri. |
| Kapsam | Global mi tenant'a özgü mü. |
| Uygulama durumu | Bu ayarın şu anda ne şekilde uygulandığını gösterir — örneğin yapılandırıldığı gibi çalışıyor, küresel bir varsayılan süreye tabi, yasal tutma nedeniyle sabitlenmiş, platformun belirlediği bir üst sınır nedeniyle sınırlandırılmış, geçici olarak devre dışı, veya henüz gerçek etkisi olmayan deneme modunda. "Fiziksel doğrulama gerekli" etiketi görünüyorsa bu satırın süresi henüz fiziksel olarak doğrulanmamıştır. |
| Yasal tutma | Bu dataset'in tenant yasal tutmasından etkilenip etkilenmediği (Evet/Hayır). |

> **Not:** Eğer bu tablo hiç satır döndürmüyorsa sayfa açık bir uyarı gösterir: "Hiçbir dataset seviyesi sözleşme döndürülmedi — profil alanlarını fiziksel bir saklama garantisi olarak kullanmayın."

---

## Neden böyle?

Yönet grubundaki üç sayfa kasıtlı olarak farklı yetki katmanlarında durur:

- **Bildirimler ve Yönlendirme Kuralları**'nı bir güvenlik operatörü veya tenant yöneticisi kendi başına kurabilir, platform yöneticisine bağımlı kalmaz.
- **Denetim Depolama**'da görüntüleme tenant yöneticisine açıktır — kendi verinizin hangi koruma altında olduğunu doğrulayabilmeniz gerekir. Ancak arka ucu **yönetmek** platform yöneticisine özeldir, çünkü bir dış arşiv kovası ve saklama süresi seçmek platform genelinde altyapısal bir karardır.
- **Saklama Politikası**, sayfayı sadece görüntülemek için bile platform yöneticisi rolü gerektirir. Bunun nedeni, buradaki her değişikliğin (saklama süresi, veri ikamet bölgesi, yasal tutma) doğrudan yasal/regülatif bir garantiyi değiştirmesidir; bu yüzden platform bu yüzeyi en dar yetki grubuna kilitler.

---

## SSS

**Bir bildirim kanalını silersem yönlendirme kurallarım ne olur?**
Kural, silinen kanalın kimliğini listesinde tutar ama bu kimlik artık hiçbir kanala karşılık gelmediği için o kanala gönderim yapılamaz; arayüzde ilgili satır "Kanal kaldırıldı" notuyla işaretlenir. Kural, kalan (varsa) kanallarla çalışmaya devam eder. Silinen kanal bir kuralın *tek* kanalıysa, o kural artık hiçbir yere gönderim yapamaz.

**Yasal tutmayı kaldırdım ama arşivdeki nesneler hâlâ silinemiyor, neden?**
Denetim Depolama sayfasındaki "tutmayı kaldır" işlemi sadece bundan sonra yazılacak yeni girdiler için geçerlidir. Tutma etkinken zaten yazılmış nesneler, arşive özgü serbest bırakma iş akışı çalıştırılana kadar tutulu kalır.

**Saklama Politikası sayfasında alanlar neden gri (kilitli) görünüyor?**
Profil ön ayarı **Özel (Custom)** olarak seçilmediği sürece saklama süresi alanları düzenlenemez; bunlar seçili profilin (Varsayılan/Düzenlenmiş) API'den gelen değerlerini yansıtır.

**Kritik bulgu teslimat kapısı "Kurulum gerekli" diyor ama bende hem kanal hem kural var, neden?**
Kapı sadece **her zaman açık (iş saatleri kısıtlaması olmayan), filtresiz** bir kuralın **etkin, uygulama içi olmayan** bir kanala bağlı olmasını "Hazır" sayar. İş saatleri kısıtlı bir kural veya sadece uygulama içi (SignalR) kanala giden bir kural bu koşulu karşılamaz.

---

## Kimler Kullanır?

| Sayfa / işlev | İlgilenen rol |
|---|---|
| Bildirimler, Yönlendirme Kuralları (görüntüleme) | Tüm geliştirici ekip |
| Bildirimler, Yönlendirme Kuralları — kanal/kural oluşturma, düzenleme, silme | Güvenlik operatörü |
| Denetim Depolama (görüntüleme) | Tenant yöneticisi |
| Denetim Depolama — arka uç yapılandırma, yasal tutma | Platform yöneticisi |
| Saklama Politikası (görüntüleme dahil) | Platform yöneticisi |

---

## İlgili

- [Güvenlik Merkezi: Genel Bakış](./security-center-overview.md)
- [İzle: Genel Bakış ve Bulgular](./security-center-monitor.md)
- [Kayıt: Denetim ve Giriş Aktivitesi](./security-center-record.md)
- [Koru: Politika ve Müdahale](./security-center-protect.md)
- [Doğrula: Uyumluluk ve Sağlık](./security-center-verify.md)
