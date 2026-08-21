# Kayıt: Denetim ve Giriş Aktivitesi

Kayıt grubu, tenant üzerinde olan bitenin değişmez bir kaydını tutar: birleşik bir denetim zaman çizelgesi ve kimlik doğrulama (giriş/çıkış) etkinliği. Bu grup aksiyon almaya değil, izlemeye ve kayıt tutmaya odaklıdır — bulgu çözme, politika uygulama veya engelleme gibi aksiyonlar Koru grubunda yer alır.

Grup 2 sayfadan oluşur: **Denetim Günlüğü** ve **Giriş Etkinliği**.

---

## Denetim Günlüğü

### Genel bakış

Denetim Günlüğü, tenant'taki güvenlikle ilgili tüm olayları tek bir kronolojik akışta birleştirir: çalışma zamanı algılama sinyalleri (Policy Guard, Runtime Sensor, Network Visibility), kimlik/giriş olayları ve platform API denetim kayıtları. Sayfa üstünde **Hash-chained · append-only** rozeti bulunur; bu, alttaki denetim kayıtlarının hash-zincirlenmiş ve sadece-ekleme (WORM) sözleşmesiyle tutulduğunu belirtir — kayıtlar sonradan değiştirilemez veya silinemez. Zincirin gerçekte doğrulanması periyodik olarak arka planda çalışan bir doğrulayıcı tarafından yapılır; rozet bu sözleşmeyi gösterir, anlık bir doğrulama sonucu değildir.

Zaman çizelgesi üstünde şu özet sayaçlar gösterilir:

| Sayaç | Anlamı |
|---|---|
| Total | Yüklenen (filtrelenmemiş) toplam olay sayısı |
| Failed | Başarısız / reddedilmiş / kilitli / geçersiz / hatalı olarak sınıflanan **veya** şiddeti Critical/High olan olay sayısı |
| (Kaynak başına 4 kutu) | Yüklenen olaylar içinde görülen kaynaklardan alfabetik olarak ilk 4'ünün sayısı (en sık görülen 4 kaynak değil) |

> **Not:** Sayfa açıldığında, URL'de açık bir tarih parametresi yoksa varsayılan olarak yalnızca **son 24 saatlik** pencere gösterilir. Bu, tek bir gürültülü kaynağın (örneğin çok sık tekrar eden bir ağ olayı) sayfanın ilk sayfasını tamamen doldurup giriş/kimlik olaylarını görünmez kılmasını önlemek içindir. Bu 24 saatlik varsayım, aktif bir filtre olarak gösterilmez ve "filtreleri temizle" ile kaldırılmaz — sadece kullanıcı bir adli inceleme bağlantısıyla (aşağıya bakın) veya URL'de açık bir tarih aralığıyla gelirse değişir.

### Kaynak türleri

Zaman çizelgesindeki her satır, olayın hangi katmandan geldiğini gösteren renkli bir kaynak etiketi taşır:

| Görünen ad | Ne izler |
|---|---|
| Policy Guard | İş yüklerinin çalışma zamanı korumasını (dosya, süreç, yetenek) kuralları karşısında izler, her ihlali bildirir |
| Network Visibility | Servisler arası trafiği izler; engellenen bağlantıları ve olağan dışı trafik örüntülerini bildirir |
| Runtime Sensor | Konteynerler içinde çalışan süreçleri gözlemler; beklenmeyen komut çalıştırma, dosya erişimi ve yetki kullanımını yakalar |
| Identity | Şüpheli oturum açma ve hesap etkinliği — başarısız girişler, kilitlenmeler, taklit (impersonation) |
| Audit | Platform denetim kaydından türetilen kayıtlar — hangi API'nin kim tarafından ve ne zaman çağrıldığı |

> **Not:** Daha önce paylaşılmış bir sayfa yer imi farklı bir kaynak adı gösteriyorsa yine de doğru satırlara ulaşır — eski adlandırmalar otomatik olarak güncel karşılıklarına eşlenir.

### Filtreler

| Filtre | Ne yapar |
|---|---|
| Arama kutusu | Olay tipi, mesaj, kaynak, aktör ve istemci IP'si üzerinde serbest metin araması |
| Kaynak çipleri (üstte) | Birden çok doğal kaynağı aynı anda işaretleyip sunucu tarafında listeyi bu kaynaklarla sınırlar |
| Kaynak açılır menüsü | Tek bir kaynağı seçer; istemci tarafında ek bir daraltma uygular |
| Sonuç | Tümü / Başarılı / Başarısız |
| Aktör rozeti | Aktif bir aktör filtresi varsa (genelde bir adli inceleme bağlantısından gelir) görünür; **×** ile temizlenir |
| Tarih penceresi rozeti | Açık bir tarih aralığı uygulanmışsa görünür; **×** ile temizlenir |

Kaynak çipleri ile kaynak açılır menüsü birbirini tamamlar ve birlikte kullanılabilir; çipler birden çok kaynağı aynı anda işaretlemenizi sağlar, açılır menü ise tek bir kaynağa odaklanmak için kullanılır.

> **İpucu:** Aktif bir filtre sonucu boş bir liste veriyorsa, zaman çizelgesinin altında **Filtreleri temizle** düğmesi belirir. Bu düğme kaynak, aktör, sonuç ve arama filtrelerini sıfırlar; ancak tarih penceresi ve önem derecesi eşiği (bunlar bir adli inceleme bağlantısından geldiği için) kasıtlı olarak korunur.

### Zaman çizelgesi satırları

Her satır şunları gösterir: saat (seçili saat diliminde) ve "X dakika/saat önce" bilgisi, kaynak etiketi, küme etiketi (olay bir kümeye bağlıysa), olay tipi, Başarılı (OK) / Başarısız (FAIL) rozeti, varsa aktör (kullanıcı adı) ve istemci IP'si, ve ham olay mesajı.

Aynı mantıksal olayın kısa sürede tekrar tekrar tetiklenmesi (örneğin aynı süreç yürütmesinin defalarca yakalanması), satırın yanında bir **×N** rozetiyle tek satıra toplanır. Bu rozete tıklamak, o grubun içindeki her tekil oluşumu (saat, aktör, IP farklarıyla) açar. Toplanan grup satırın en son görülen zamanını gösterir; ilk görülme zamanı rozetin üzerine gelindiğinde görünür.

Satırlar gün başlıklarına ("Bugün", "Dün" veya tarih) göre gruplanır; bu gruplama, seçtiğiniz saat dilimine göre hesaplanır.

### Yan panel: içgörüler

Zaman çizelgesinin yanında filtrelenmiş sonuçlara göre üç kart bulunur:

| Kart | İçerik |
|---|---|
| Top users | En sık görülen ilk 5 aktör |
| Top IPs | En sık görülen ilk 5 istemci IP'si |
| Frequent actions | En sık görülen ilk 5 olay tipi |

### Adli inceleme bağlantısı

Giriş Etkinliği sayfasındaki her satır ve giriş riski panelindeki her sinyal, "Forensics" / "Investigate evidence" bağlantısıyla Denetim Günlüğü'ne atlayabilir. Bu bağlantı, ilgili olayın aktörünü ve zamanının **±30 dakikalık** bir penceresini otomatik uygular, böylece o olayın çevresindeki tüm ilişkili aktiviteyi tek ekranda görebilirsiniz.

### CSV dışa aktarma

Sağ üstteki **Export CSV** düğmesi iki mod sunar:

| Mod | İçerik |
|---|---|
| Grouped (deduped) | Ekranda görüldüğü gibi toplanmış satırlar — her satırda son görülme zamanı, ilk görülme zamanı, oluşum sayısı, kaynak, olay tipi, önem derecesi, aktör, IP ve mesaj |
| Raw (one row per event) | Toplamadan, her olay için tek satır — zaman, kaynak, olay tipi, önem derecesi, aktör, IP, mesaj |

Dışa aktarma her zaman o anda **filtrelenmiş** sonuç kümesi üzerinden çalışır; filtreler sonucu sıfıra indiriyorsa dışa aktarma bir bilgi mesajıyla iptal edilir.

---

## Giriş Etkinliği

### Genel bakış

Giriş Etkinliği sayfası, tenant'taki oturum geçmişini ve kimlik doğrulama olaylarını listeler: kim, ne zaman, hangi IP ve tarayıcıdan, hangi uygulama üzerinden oturum açtı/kapattı, parola değiştirdi, hesabı kilitlendi ya da iki faktörlü kimlik doğrulamayı açtı/kapattı.

Sayfa üstünde şu özet sayaçlar bulunur: Total events, Successful logins, Failed attempts, Unique users, Unique IPs.

> **Not:** Bu sayfa en fazla son 300 kaydı tek seferde yükler; Denetim Günlüğü'ndeki gibi imleçli "Load more" (daha fazla yükle) mekanizması yoktur. Daha eski bir olayı aramanız gerekiyorsa Denetim Günlüğü'nü kullanın veya arama/filtre kombinasyonuyla bu 300 kayıt içinde daraltın.

### Kayıt türleri

| Olay | Anlamı | Sonuç |
|---|---|---|
| Signed in | Başarılı oturum açma | Başarılı |
| Signed out | Oturum kapatma | Başarılı |
| Sign-in failed | Başarısız oturum açma denemesi | Başarısız |
| Account locked | Hesap kilitlendi | Başarısız |
| Sign-in not allowed | Oturum açmaya izin verilmedi | Başarısız |
| Invalid username | Geçersiz kullanıcı adı | Başarısız |
| Invalid password | Geçersiz parola | Başarısız |
| Password changed | Parola değiştirildi | Başarılı |
| Reset code sent | Parola sıfırlama kodu gönderildi | Başarılı |
| Password reset | Parola sıfırlandı | Başarılı |
| 2FA enabled / disabled | İki faktörlü kimlik doğrulama açıldı/kapatıldı | Başarılı |

Listede tanınmayan bir olay tipi görünürse (yukarıdaki tabloda olmayan), sayfa onu ham haliyle gösterir ve adında "fail / denied / locked / invalid" gibi bir ifade varsa otomatik olarak Başarısız sayar.

### Giriş geçmişi tablosu

Masaüstünde tablo şu sütunları gösterir: **Time** (saat), **Event** (olay + Başarılı/Başarısız rozeti), **User** (kullanıcı adı), **IP address**, **Device** (tarayıcı + işletim sistemi), **Application**. Her satırın sağında, o olayı Denetim Günlüğü'nde ±30 dakikalık pencereyle açan bir **Forensics** bağlantısı bulunur.

### Filtreler

| Filtre | Ne yapar |
|---|---|
| Arama kutusu | Kullanıcı adı, kimlik, olay, IP, tarayıcı bilgisi ve uygulama adı üzerinde arama yapar |
| Result | Tümü / Success / Failed |
| Action | Belirli bir olay tipine daraltır (yalnızca yüklenen kayıtlarda görülen tipler listelenir) |

Filtreler sonucu sıfıra indirirse **Clear filters** düğmesi belirir.

### CSV dışa aktarma

**Export CSV** düğmesi, o anda filtrelenmiş listeyi tek bir mod olarak indirir: zaman, olay, kullanıcı, istemci IP'si, tarayıcı, uygulama adı sütunlarıyla.

### Giriş tehdit değerlendirmesi paneli

Giriş geçmişinin üstünde, tenant'ın son dönem giriş etkinliğini otomatik olarak analiz eden bir **kimlik tehdit tespiti** paneli bulunur. Panel her 30 saniyede bir kendini tazeler.

Panel üç kapsam kutusu gösterir:

| Kutu | Ne anlatır |
|---|---|
| Historical baseline (Geçmiş temel çizgi) | Hesap başına normal başarısızlık oranının ne kadar veriyle öğrenildiğini gösterir — "Ready" (hazır) veya "Insufficient data" (yetersiz veri) |
| Impossible-travel coverage (İmkansız-seyahat kapsamı) | IP coğrafi konum çözümlemesinin ne kadarının yapılabildiğini gösterir — yüzde olarak, ya da "Limited" / "Not configured" |
| Trusted-network context (Güvenilen ağ bağlamı) | Kuruluşun güvenilen ağ/IP aralıklarının tanımlanıp tanımlanmadığını gösterir |

> **Not:** Güvenilen bir ağdan gelen etkinlik risk değerlendirmesinde asla tamamen gizlenmez, sadece önem derecesi düşürülür (downgrade). Sinyal listesinde "Trusted network" rozetiyle işaretlenerek görünür kalır — böylece bir operatör, güvenilen bir kaynaktan gelen gerçek bir tehdidi kaçırmaz.

Bu üç kapsam alanından biri eksikse panel bunu bir **Detection coverage gap** (algılama kapsam boşluğu) uyarısı olarak gösterir. Olası boşluk nedenleri:

| Görünen mesaj | Anlamı |
|---|---|
| Baseline scan truncated | Temel çizgi taraması kesildi (çok fazla veri) |
| Baseline needs more history | Temel çizgi için daha fazla geçmiş veri gerekiyor |
| Trusted networks not configured | Güvenilen ağlar tanımlanmamış |
| IP geolocation not configured | IP coğrafi konum çözümlemesi yapılandırılmamış |
| IP geolocation coverage below 80% | IP coğrafi konum kapsamı %80'in altında |

Tespit edilen risk sinyalleri şu türlerden biri olabilir:

| Sinyal | Açıklama | Önem derecesi kaynağı |
|---|---|---|
| Account brute-force pattern | Başarısız denemeler hem sabit eşiği hem de o hesabın temel çizgisini aştı | Değişken |
| Password-spray pattern | Tek bir kaynak birden çok hesabı hedef aldı | Değişken |
| Impossible-travel pattern | Art arda gelen başarılı girişler, tanımlı seyahat hızı sınırını aştı | Değişken |
| Possible account takeover | Başarılı bir giriş, yoğun bir başarısızlık patlamasının hemen ardından geldi | Değişken |

Her sinyal kartında ilgili kullanıcı adı, IP adresi, başarısızlık sayısı, etkilenen hesap/IP sayısı, temel çizgiye göre kat sayısı (varsa) ve seyahat mesafesi/hızı (imkansız-seyahat sinyalleri için) gösterilir. Her sinyalin yanındaki **Investigate evidence** bağlantısı, o sinyalin kanıtını Denetim Günlüğü'nde açar.

Hiçbir sinyal tespit edilmediğinde panel, kapsam tam ise "No deterministic login threat detected" (kesin bir giriş tehdidi tespit edilmedi), kapsamda boşluk varsa "No confirmed signal · coverage limited" (onaylı sinyal yok, kapsam sınırlı) mesajını gösterir — bu iki durum kasıtlı olarak birbirinden ayrılır, çünkü "temiz" ile "göremiyoruz" aynı şey değildir.

> **Not:** Tehdit değerlendirmesi servisi geçici olarak ulaşılamaz durumda olursa panel bunu açıkça bildirir ("Login threat assessment is unavailable") ve ham giriş olaylarının aşağıda görünmeye devam ettiğini belirtir — böylece bir servis kesintisi, "hiçbir tehdit yok" şeklinde yanlış yorumlanmaz.

---

## Neden böyle?

**Varsayılan 24 saatlik pencere.** Denetim Günlüğü'nün varsayılan olarak son 24 saati göstermesi rastgele bir tercih değildir: tek bir gürültülü kaynağın (örneğin sürekli tekrarlanan bir ağ kuralı) sayfanın ilk yüklenen sayfasını tamamen doldurup rutin giriş ve kimlik aktivitesini görünmez kılmasını önler.

**Hash-zincir rozeti bir sözleşmeyi belirtir, canlı bir doğrulama değil.** Rozetin metni kasıtlı olarak geçmiş zaman ("verified" / doğrulandı) kullanmaz, çünkü istemci taraf bir doğrulama çalıştırmaz — bu doğrulama arka planda periyodik olarak çalışan bir sunucu görevi tarafından yapılır. Rozet, verinin bu sözleşme altında tutulduğunu belirtmek içindir.

**Güvenilen ağ, gizlemek için değil düşürmek içindir.** Giriş riski değerlendirmesinde güvenilen bir ağdan gelen etkinlik asla listeden kaybolmaz, sadece önem derecesi düşer. Bir saldırganın güvenilen bir IP aralığını ele geçirmesi ihtimaline karşı, "güvenilen" etiketi bir tehdidi görünmez kılmak için kullanılmaz.

**Tekrarlayan olaylar toplanır, kaybolmaz.** Aynı mantıksal olayın (örn. bir sürecin defalarca çalıştırılması) her tekrarı ayrı bir satır olarak listelenseydi, gerçek sinyal gürültü içinde kaybolurdu. Bunun yerine tekrarlar bir sayaçla tek satıra toplanır; ayrıntı istenirse genişletilebilir.

---

## SSS

**Bir filtre uyguladım ama zaman çizelgesi boş görünüyor, neden?**
Denetim Günlüğü'nde bir kaynak/aktör/sonuç/arama filtresi aktifken eşleşen bir satır henüz yüklenen sayfada yoksa, sayfa arka planda otomatik olarak ek sayfalar getirmeye çalışır (en fazla 10 sayfa, ardından durur). Bu süre boyunca liste boş görünebilir; eşleşen satır bulunduğunda veya sınırlara ulaşıldığında liste güncellenir. Sonuç yine boşsa **Filtreleri temizle** ile daraltmayı kaldırabilirsiniz.

**Giriş Etkinliği sayfasında neden tarih aralığı seçemiyorum?**
Sayfa doğrudan bir tarih aralığı seçicisi sunmaz; en son 300 kaydı gösterir. Belirli bir zaman aralığını incelemek için bir satırın veya risk sinyalinin **Forensics** bağlantısıyla Denetim Günlüğü'ne geçip oradaki tarih penceresini kullanabilirsiniz.

---

## Kimler Kullanır?

| Sayfa | İlgilenen rol |
|---|---|
| Denetim Günlüğü, Giriş Etkinliği (görüntüleme) | Tüm geliştirici ekip |
| Adli inceleme bağlantısı üzerinden olay araştırması | Güvenlik operatörü / Denetçi (auditor) rolü |

---

## İlgili

- [Güvenlik Merkezi: Genel Bakış](./security-center-overview.md)
- [İzle: Genel Bakış ve Bulgular](./security-center-monitor.md)
- [Koru: Politika ve Müdahale](./security-center-protect.md)
- [Doğrula: Uyumluluk ve Sağlık](./security-center-verify.md)
- [Yönet: Bildirim ve Saklama](./security-center-manage.md)
