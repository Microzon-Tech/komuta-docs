# Özel Alan Adları ve Servis Adresleri

HTTP trafiği sunan her servis, deploy edildiğinde Komuta'dan HTTPS bir adres alır; bunun için bir şey ayarlamanız gerekmez. Ziyaretçilerin servise kendi alan adınızdan (ör. `api.sirketim.com`) ulaşmasını istiyorsanız alan adını **Alan Adları** sayfasından bağlar, DNS sağlayıcınızda birkaç kayıt eklersiniz. Bu sayfa ikisini de anlatır: Komuta'nın verdiği adresi ve kendi alan adınızı bağlamayı, kontrol etmeyi, kaldırmayı.

---

## Servisinizin Komuta Adresi

Komuta adresi, servisin ilk başarılı deploy'u sırasında oluşturur. O zamana kadar servis listesinde adres yerine **URL oluşuyor** ya da **URL bekleniyor** görünür ("Public URL deploy sırasında oluşturulur. Bir sonraki başarılı deploy'dan sonra burada görünecek.").

Adres şu biçimdedir:

```
siparis-api-3a2331e6.edge-1.komuta.app
```

- İlk bölüm servisin adı ve ardından servis kimliğinin ilk 8 karakteridir. Ad küçük harfe çevrilir, Türkçe ve aksanlı harfler düz Latin harflerine dönüştürülür (`ş` → `s`, `ğ` → `g` gibi), boşluk, `_` ve `.` `-` olur, diğer karakterler atılır; addan en fazla 40 karakter kullanılır.
- Adresin geri kalanı servisin çalıştığı bölgeye aittir ve her zaman `.komuta.app` ile biter.
- Adres hemen HTTPS ile çalışır; sertifikasını Komuta yönetir. Düz HTTP istekleri `301` ile HTTPS'e yönlendirilir.
- İstekler servisin birincil portuna gider. Bkz. [Erişim ve Portlar](services-ports.md).

Kendi alan adınızı bağladıktan sonra da bu adres çalışmaya devam eder; ikisi de aynı servise gider.

### Adresleri nerede görürsünüz

**Servis Detay → Yapılandırma → Erişim ve portlar** sayfasını açıp **Ağ** sekmesine geçin. **Genel adresler** kartı servisin bütün adreslerini listeler:

| Etiket | Anlamı |
|---|---|
| **Komuta alan adı** | Komuta'nın oluşturduğu adres. |
| **Özel alan adı** | Sizin bağladığınız alan adı. Özel alan adları Komuta adresinden önce listelenir. |
| **Beklemede** | Özel alan adı henüz aktif değil. |
| **{host} adresine yönleniyor** | Özel alan adı ziyaretçilerini servisin başka bir alan adına gönderiyor (bkz. [Bir alan adını diğerine yönlendirme](#bir-alan-adını-diğerine-yönlendirme)). |

Her adresin yanındaki kopyalama düğmesi tam URL'i kopyalar; **Alan adlarını yönet** bağlantısı **Alan Adları** sayfasını açar.

### Genel URL

Aynı karttaki **Genel URL** anahtarı, servise internetten ulaşılıp ulaşılamayacağını belirler. Kapalıyken kartta "Yayında değil — servise internetten ulaşılamaz." yazar. Kapatmak istediğinizde **Genel URL kapatılsın mı?** sorulur ("Servisin rotaları kaldırılır ve alan adları yanıt vermez."): Komuta adresi de bütün özel alan adları da yanıt vermez olur.

Job ve CronJob'lar HTTP sunmadığı için adres almaz ve onlara özel alan adı bağlanamaz.

### Blue-Green önizleme adresi

Blue-Green stratejisiyle deploy edilen bir servis, yeni sürümü canlıya almadan önce deneyebileceğiniz bir önizleme adresinden de yanıt verir. Bu adres, Komuta adresinin ilk bölümüne `-preview` eklenmiş halidir, ör. `siparis-api-3a2331e6-preview.edge-1.komuta.app`. Özel alan adlarının önizleme adresi yoktur. Bkz. [Dağıtım Stratejileri](deployment-strategies.md).

### Edge önbelleğini temizleme

**Genel adresler** kartındaki **Edge önbelleğini temizle**, servisin dosyalarının bütün adreslerdeki önbellek kopyasını temizler. Komuta bunu her deploy'dan sonra da yapar; yeni deploy yapmadan bir değişikliğin görünmesini istediğinizde kullanın. Dakikada bir kez kullanılabilir.

---

## Alan Adı Bağlamadan Önce

- **Genel URL'i açık bir servis.** Alan adı, HTTP sunan ve **Genel URL**'i açık olan bir servise bağlanır. **Hedef Uygulama** listesinde yalnızca bu servisler görünür.
- **Alan adının DNS'ine erişim.** Alan adınızın DNS'inin yönetildiği yerde üç kayıt oluşturacaksınız.
- **Alt alan adıyla başlayın.** `www.sirketim.com` ya da `api.sirketim.com` en kolay durumdur. Kök alan adı (`sirketim.com`) için DNS sağlayıcınızın CNAME düzleştirmeyi desteklemesi gerekir; bkz. [Kök alan adları](#kök-alan-adları).
- **Her adres ayrı bir alan adıdır.** `sirketim.com` ile `www.sirketim.com` ayrı ayrı eklenir. Bir servise birden fazla özel alan adı bağlanabilir.
- **Wildcard yok.** `*.sirketim.com` kabul edilmez; her alt alan adını ayrı ekleyin.
- **Türkçe karakterli alan adları** `xn--` biçimiyle girilir (ör. `şirket.com` yerine `xn--irket-idb.com`).
- **Bir alan adı, bir servis.** Bir alan adı Komuta'da aynı anda yalnızca bir servise bağlı olabilir; bu, bütün organizasyonlar için geçerlidir.
- **Yetki.** Alan adı eklemek için alan adı ekleme yetkisi ve hedef serviste düzenleme yetkisi gerekir. Alan adını kontrol etmek ve kaldırmak için alan adı düzenleme yetkisi gerekir.

---

## Özel Alan Adı Bağlama

### 1. Alan adını ekleyin

1. Sol menüde **Yapılandırma** altındaki [**Alan Adları**](https://console.komuta.io/domains) sayfasını açın.
2. **+** (**Alan adı ekle**) düğmesine tıklayın. Henüz hiç alan adınız yoksa **İlk Alan Adınızı Ekleyin**'e tıklayın.
3. **Özel Alan Adı Ekle** penceresinde **Alan adı** alanına alan adını yazın, ör. `api.sirketim.com`.
4. **Hedef Uygulama**'dan servisi seçin.
5. **Alan Adı Ekle**'ye tıklayın.

"Alan adı eklendi. Gösterilen DNS kayıtlarını DNS sağlayıcınıza ekleyin, ardından durumu görmek için Doğrulamayı Yenile'yi kullanın." mesajı görünür ve yeni alan adı açılır. Komuta servisin rotalarını arka planda günceller; deploy yapmanız gerekmez.

### 2. DNS kayıtlarını oluşturun

**Eklenecek DNS Kayıtları** bölümü Komuta'nın ihtiyaç duyduğu kayıtları listeler. Her birini DNS sağlayıcınızda gösterilen tür, **Ad** ve **Beklenen** değerle birebir oluşturun. Yazım hatası olmaması için değerlerin yanındaki kopyalama düğmelerini kullanın.

`api.sirketim.com` için kayıtlar şöyle görünür:

| Amaç | Tür | Ad | Değer |
|---|---|---|---|
| **Trafik Yönlendirme** | CNAME | `api.sirketim.com` | `origin.komuta.app` |
| **Sahiplik Doğrulaması** | TXT | `_cf-custom-hostname.api.sirketim.com` | Konsolda gösterilen kod |
| **Sertifika Doğrulaması** | CNAME | `_acme-challenge.api.sirketim.com` | Konsolda gösterilen `api.sirketim.com.<kod>.dcv.cloudflare.com` |

- Sahiplik kodu ve sertifika doğrulama değeri alan adınıza özeldir. Bunları her zaman konsoldan alın.
- TTL'i **Auto** ya da sağlayıcınızın izin verdiği en düşük değere ayarlayın.
- Birçok DNS paneli ada alan adınızı kendiliğinden ekler. Sizinki de ekliyorsa yalnızca önündeki kısmı yazın (ör. `api` ya da `_acme-challenge.api`).
- Sertifika doğrulama kaydını bir kez eklersiniz. Sertifikalar bundan sonra otomatik olarak verilir ve yenilenir.
- Alan adı aktif olduktan sonra kayıtları silmeyin. Ziyaretçileri trafik kaydı taşır; sertifika yenilemesi de sertifika doğrulama kaydına bağlıdır.
- Daha önce bağlanan alan adları `origin.edge-1.komuta.app` gibi bir adrese yönleniyor olabilir. Bunlar çalışmaya devam eder; değiştirmeniz gerekmez.

Bölümdeki **?** düğmesi aynı adımları içeren **DNS kayıtları nasıl eklenir** penceresini açar.

> **Not:** Konsol sertifika doğrulama CNAME'i yerine `_acme-challenge` **TXT** kayıtları listeliyorsa bunların değerleri her sertifikada değişir ve birkaçı aynı adı paylaşabilir: hepsini ekleyin.

### 3. Etkinleşmesini bekleyin

Kayıtların üstünde ne kadar ilerlediğiniz yazar, ör. "3 kayıttan 2 tanesi yapılandırıldı". Her kayıt kendi durumunu ve en son ne zaman kontrol edildiğini gösterir. Henüz aktif olmayan bir alan adı açıkken sayfa her 30 saniyede bir kendini yeniler.

Komuta sahipliği doğrulayıp sertifika verildiğinde alan adının durumu **Aktif** olur ve servis `https://api.sirketim.com` adresinden yanıt verir.

- DNS değişiklikleri genellikle birkaç dakikada görünür; 24 saate kadar sürebilir.
- Hemen kontrol etmek için **Doğrulamayı Yenile**'ye tıklayın. "Doğrulama yenilendi. DNS'i yeni değiştirdiyseniz birkaç dakika sonra tekrar yenileyin." mesajı görünür. Düğme, alan adı aktif değilken ya da ilgilenmeniz gerekirken görünür. Yenileme, alan adını servisin rotalarına yeniden de uygular.
- Komuta alan adının durumunu arka planda da kontrol eder; sayfayı kapatsanız da durum güncellenir.

> **Uyarı:** Eklendikten sonra 14 gün içinde aktif olmayan bir alan adı otomatik olarak kaldırılır. İstediğiniz zaman yeniden ekleyebilirsiniz.

---

## Durumlar

### Alan adı durumu

Listede her alan adının yanındaki etiket ve sayfanın üstündeki sayaçlar şu durumları kullanır:

| Durum | Anlamı |
|---|---|
| **Aktif** | Sahiplik doğrulandı ve sertifika kullanımda. Ziyaretçiler servise ulaşıyor. |
| **Hazırlanıyor** | Komuta DNS kayıtlarını ya da sertifikayı bekliyor. Henüz bir sorun yok. |
| **Güncelleniyor** | Çalışan bir alan adının sertifikası yeniden dağıtılıyor. İşlem gerekmiyor. |
| **İşlem Gerekli** | Alan adı sizi bekliyor; ör. bir hata bildirildi, DNS artık Komuta'yı göstermiyor, sertifikanın süresi doldu ya da kontrol zaman aşımına uğradı. Neyi düzeltmeniz gerektiğini **Sorunlar Tespit Edildi** ve **Senkronizasyon Durumu** kartları söyler. |

### Senkronizasyon Durumu

**Senkronizasyon Durumu** kartı, altlarında birer satır açıklamayla iki ayrı durum gösterir; ayrıca **Son Senkronizasyon** ve varsa **Son Hata** yazar.

**Sağlayıcı Durumu** alan adının kendisiyle ilgilidir:

| Durum | Konsoldaki açıklama |
|---|---|
| **Aktif** | Bu alan adının trafiği Komuta'ya ulaşıyor. |
| **Bekliyor** | Aşağıdaki DNS kayıtlarının bulunması bekleniyor. |
| **Güncelleniyor**, **Dağıtılıyor**, **Taşınıyor** | İşlem gerekmiyor. |
| **İncelemede** | Sağlayıcı bu alan adını inceliyor. Durum düzelmezse destek ile iletişime geçin. |
| **DNS taşındı** | DNS artık Komuta'yı göstermiyor. Trafik kaydını kontrol edin. |
| **Engellendi** | Sağlayıcı bu alan adını engelledi. Destek ile iletişime geçin. |
| **Silindi** | Bu alan adı sağlayıcıda artık yok. Buradan kaldırıp yeniden ekleyin. |
| **Henüz bildirilmedi** | Sağlayıcı bu durumu henüz bildirmedi. |

**SSL Durumu** sertifikayla ilgilidir:

| Durum | Konsoldaki açıklama |
|---|---|
| **Aktif** | Sertifika verildi ve kullanımda. |
| **Başlatılıyor**, **Sertifika veriliyor**, **Dağıtılıyor** | Sertifika hazırlanıyor. İşlem gerekmiyor. |
| **Doğrulama bekleniyor** | Sertifika doğrulama kaydının bulunması bekleniyor. |
| **Süresi doluyor** | Sertifika henüz yenilenemedi. Sertifika doğrulama kaydını kontrol edin. |
| **Süresi doldu** | Sertifikanın süresi doldu. Sertifika doğrulama kaydını kontrol edip yenileyin. |
| **Zaman aşımı** | Sağlayıcı beklemeyi bıraktı. DNS kayıtlarını kontrol edip yenileyin. |

### DNS kaydı durumları

| Durum | Anlamı |
|---|---|
| **Yapılandırıldı** | Kayıt beklenen değerle bulundu. |
| **Bekliyor** | Kayıt doğrulanmayı bekliyor. |
| **Farklı değer** | DNS başka bir değer döndürüyor; döndürdüğü değer **Gözlemlenen** satırında görünür. Kaydı düzeltin. |
| **Tespit edilmedi** | "Genel DNS'ten yanıt yok: kayıt eksik ya da kontrol tamamlanamadı." |
| **Henüz tespit edilmedi** | Kayıt henüz kontrol edilmedi. |

Üzerinde çalışılması gereken kayıtlar **Aksiyon gerekli ({count})**, tamamlananlar **Yapılandırıldı ({count})** başlığı altında toplanır. Alan adı aktif olduktan sonra eşleşmeyen kayıtlar **Bilgi amaçlı ({count})** başlığına taşınır ve işlem gerektirmez.

---

## Kök Alan Adları

`sirketim.com` gibi bir kök alan adı, detay panelinde **Kök alan adı** olarak işaretlenir; diğerleri **Alt alan adı** olarak görünür. `sirketim.com.tr` ya da `example.co.uk` gibi adlar da kök alan adı sayılır.

DNS, bir alan adının kökünde normal CNAME kaydına izin vermez. Konsol orada CNAME istiyorsa DNS sağlayıcınızın CNAME düzleştirmeyi (ALIAS ya da ANAME olarak da bilinir) desteklemesi gerekir. Konsolun da belirttiği gibi düzleştirilmiş kayıt IP adresleriyle yanıt verdiği için **Trafik Yönlendirme** satırı alan adı aktif olduktan sonra da tespit edilmemiş görünebilir. Bu beklenen bir durumdur.

Hem `sirketim.com` hem `www.sirketim.com` adresinden yayın yapmak için ikisini de aynı servise ekleyin. Ardından birini diğerine yönlendirebilirsiniz.

---

## TLS Sertifikaları

Sertifika yüklemeniz ya da yenilemeniz gerekmez.

- **Komuta adresi:** HTTPS ilk deploy'dan itibaren çalışır.
- **Özel alan adı:** Sertifika, sertifika doğrulama kaydı bulunduktan sonra verilir ve kayıt yerinde kaldığı sürece otomatik yenilenir. İlerleme **SSL Durumu** satırında görünür. **Süresi doluyor** ya da **Süresi doldu** görürseniz sertifika doğrulama kaydını kontrol edip **Doğrulamayı Yenile**'ye tıklayın.

---

## Bir Alan Adını Diğerine Yönlendirme

Alan adının detay panelindeki **Yönlendirme** kartı, o alan adının ziyaretçilerini aynı servisin başka bir alan adına kalıcı (`301`) yönlendirmeyle gönderir. Path ve query korunur; servis uyurken de yönlendirme yanıt verir.

1. **Yönlendirme** kartında hedef alan adını seçin. **Yönlendirme yok — servisi sun** yönlendirmeyi kapatır.
2. **{source}, {target} adresine yönlendirilsin mi?** penceresini onaylayın.

"Yönlendirme kaydedildi. Rota güncellemesiyle, genellikle bir iki dakika içinde yayına girer." mesajı görünür ve alan adında **{host} adresine yönleniyor** etiketi çıkar.

- Hedef yalnızca aynı servisin **Aktif** olan başka bir özel alan adı olabilir. Böyle bir alan adı yoksa kartta "Bu alan adını yönlendirmek için aynı servise başka bir alan adı ekleyip doğrulayın." yazar.
- Yönlendirmeler zincirlenmez. Başka alan adlarının yönlendiği bir alan adı kendisi yönlendirilemez.
- Tarayıcılar kalıcı yönlendirmeleri hatırlar; yönlendirmeyi kapattıktan sonra da onu görmüş ziyaretçiler bir süre yönlendirilmeye devam edebilir.

---

## Alan Adını Kaldırma

1. **Alan Adları** sayfasında alan adını seçip **Kaldır**'a tıklayın.
2. Açılan pencere alan adının kalıcı olarak kaldırılacağını söyler. **Alan Adını Kaldır**'a tıklayın.

"Alan adı silindi." mesajı görünür. Komuta alan adını arka planda servisin rotalarından çıkarır ve alan adı artık servisinize gitmez. Bu alan adına yönlenen alan adları yeniden servisi sunar.

- Bir servisi silmek, ona bağlı özel alan adlarını da kaldırır.
- Alan adını kaldırdıktan sonra artık kullanmadığınız DNS kayıtlarını DNS sağlayıcınızdan silin.
- Aynı alan adını sonradan aynı ya da başka bir servise yeniden ekleyebilirsiniz.

---

## Erişim Koruması ve Özel Alan Adları

Serviste [erişim koruması](service-access-protection.md) açıksa, özel alan adı aktif olduğunda koruma onu da kapsar.

- Ziyaretçiler servisin her adresi için ayrı giriş yapar. Bkz. [Giriş ve Paylaşım](access-protection-sign-in-sharing.md).
- Alan adının DNS'i kendi Cloudflare hesabınızdaysa ve kayıt proxy'leniyorsa koruma onu kapsayamaz ve konsol şunu söyler: "Bir özel domain kendi Cloudflare hesabınız üzerinden proxy'leniyor. Komuta'nın koruyabilmesi için DNS kaydını 'DNS only' (gri bulut) yapın."
- Korunan bir servisin Komuta adresi dahil en fazla 50 adresi olabilir. Daha fazlasında konsol "Bu servisin domain sayısı tek bir koruma politikasının kapsayabileceğinden fazla. Artık ihtiyacınız olmayan özel domainleri kaldırın." der.

---

## Sınırlar

| Sınır | Değer |
|---|---|
| Alan adı uzunluğu | En fazla 253 karakter, her bölüm en fazla 63 |
| Wildcard alan adları | Desteklenmez |
| Bir alan adının bağlanabileceği servis | 1, bütün organizasyonlar genelinde |
| Servis başına alan adı | Sabit bir sınır yok (erişim korumasıyla 50 adres) |
| Alan adı ekleme | Organizasyon başına günlük (UTC günü) ve ağ başına saatlik bir sınır vardır. Alan adını kaldırmak sayacı geri vermez. |
| Etkinleşme süresi | 14 gün; sonra alan adı otomatik olarak kaldırılır |

Ayrıca bkz. [Sınırlar](limits.md).

---

## Sık Karşılaşılan Sorunlar

**"Geçerli bir alan adı girin (örn. www.ornek.com)"**
Form yalnızca harf, rakam, `-` ve `.` kabul eder ve ad bir alan adı uzantısıyla bitmelidir. `*`, `http://`, sondaki `.` ya da path'i kaldırın; Türkçe karakterli alan adlarını `xn--` biçimiyle girin.

**Servis Hedef Uygulama listesinde yok ya da "Eşlenecek uygulama yok." yazıyor**
Listede yalnızca HTTP uç noktası olan ve **Genel URL**'i açık servisler görünür. **Erişim ve portlar → Ağ** altında **Genel URL**'i açın. Job ve CronJob'lar hiçbir zaman seçilemez. Organizasyonunuzda böyle bir servis yoksa servisin bulunduğu organizasyona geçin ya da önce bir servis oluşturun.

**"'{ServiceName}' dışarıya HTTP trafiği sunmadığı için ona özel alan adı bağlanamaz. Önce servisin genel URL'sini açın; job ve cron job'lara özel alan adı bağlanamaz."**
Nedeni yukarıdakiyle aynıdır; ör. genel URL bu arada kapatılmıştır.

**Alan adı eklenirken hata alınıyor**
- Alan adı sizin ya da başka bir organizasyonun Komuta servisine zaten bağlı. Önce oradan kaldırın.
- Organizasyonunuz günlük alan adı ekleme sınırına ulaştı. Ertesi gün (UTC) tekrar deneyin.

**Bir kayıt Tespit edilmedi ya da Farklı değer durumunda kalıyor**
**Gözlemlenen** ile **Beklenen** değeri karşılaştırın. En sık nedenler eksik kayıt, yazım hatası, alan adının iki kez eklenmesi (`api.sirketim.com.sirketim.com`) ya da DNS değişikliğinin henüz yayılmamasıdır. Kaydı düzeltin, birkaç dakika bekleyip **Doğrulamayı Yenile**'ye tıklayın.

**"Bu alan adı için henüz DNS kaydı dönmedi. Tekrar kontrol etmek için Doğrulamayı Yenile'yi kullanın; sorun sürerse destek ile iletişime geçin."**
**Doğrulamayı Yenile**'ye tıklayın. Liste boş kalırsa bir [destek talebi](support-tickets.md) açın.

**Kök alan adının Trafik Yönlendirme satırı tespit edilmedi görünüyor ama alan adı Aktif**
CNAME düzleştirmede bu beklenen bir durumdur. İşlem gerekmez.

**Aktif bir alan adı DNS taşındı durumuyla İşlem Gerekli'ye geçti**
Trafik kaydı artık Komuta'yı göstermiyor. Konsolda gösterilen CNAME kaydını geri ekleyin.

**Sağlayıcı durumu Silindi**
Alan adını **Alan Adları** sayfasından kaldırıp yeniden ekleyin, ardından DNS kayıtlarını kontrol edin.

**Sağlayıcı durumu İncelemede ya da Engellendi**
Bir [destek talebi](support-tickets.md) açın.

**Değişiklik hesap ya da ücretsiz plan nedeniyle reddediliyor**
Alan adı değişiklikleri, servisteki diğer değişikliklerle aynı kurallara tabidir. "Hesabınız ödenmemiş ekstreler nedeniyle askıda; bu servisteki değişiklikler engellendi. Yeniden değişiklik yapabilmek için bakiyenizi ödeyin." ya da ücretsiz bir serviste "Ücretsiz planın bu ayki uyanık saatleri bitti; bu servisteki değişiklikler gelecek ayın 1'ine kadar engellendi. Hemen değişiklik yapmak için servise Her zaman açık eklentisini ekleyin ya da servisi ücretli bir plana taşıyın." mesajını görebilirsiniz. Bkz. [Servis Uyku Modu](service-sleep.md).

---

## İlgili Dokümanlar

- [Erişim ve Portlar](services-ports.md) — Portlar, genel URL ve servisin adresleri.
- [Erişim Koruması](service-access-protection.md) — Servisin adreslerini giriş ve IP kurallarıyla korumak.
- [Dağıtım Stratejileri](deployment-strategies.md) — Blue-Green ve önizleme adresi.
- [API Gateway](api-gateway.md) — Gateway'e özel alan adı aynı yolla bağlanır: **Alan Adları** sayfasından ekleyip gateway'in servisine bağlayın.
- [Stack Manifest Referansı](stack-manifest-reference.md) — Alan adlarını Stack'te tanımlamak.
- [Destek](support-tickets.md) — Alan adı aktif olmadığında yardım almak.
