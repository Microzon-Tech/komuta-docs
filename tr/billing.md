# Faturalandırma ve Cüzdan

Komuta'da faturalandırma organizasyon düzeyinde yürür: servisleriniz, yönetilen servisleriniz ve diğer kaynaklarınız saatlik ölçülür, her ayın 6'sında bir önceki dönemin ekstresi kesilir ve tutar varsayılan kartınızdan otomatik çekilir. Bu sayfa neyin faturalandığını, ekstreleri nasıl okuyup ödeyeceğinizi, cüzdanı, ödeme yöntemlerini, fatura profilini, harcama limitini ve ödenmemiş bir ekstre olduğunda ne olduğunu anlatır.

---

## Faturalandırma Sayfaları

Sağ üstteki kullanıcı menüsünden **Hesabım**'ı açın. Sol menüdeki **Faturalandırma** grubunda şu sayfalar bulunur:

| Sayfa | Adres | Ne işe yarar |
|---|---|---|
| **Fatura Profili** | [console.komuta.io/account/billing/profile](https://console.komuta.io/account/billing/profile) | Faturada yer alan yasal kimlik, adres, vergi numarası ve fatura e-postaları. |
| **Güncel Harcama** | [console.komuta.io/account/billing/current](https://console.komuta.io/account/billing/current) | İçinde bulunduğunuz dönemin o ana kadarki maliyeti ve build dakikaları. |
| **Observability Depolama** | [console.komuta.io/account/billing/observability-storage](https://console.komuta.io/account/billing/observability-storage) | Her servisin sakladığı telemetri miktarı ve aylık fiyatı. |
| **Cüzdan** | [console.komuta.io/account/billing/wallet](https://console.komuta.io/account/billing/wallet) | Bakiye, bakiye yükleme, harcama limiti, hesap seviyesi ve kuponlar. |
| **Ekstreler** | [console.komuta.io/account/billing/invoices](https://console.komuta.io/account/billing/invoices) | Kesilen ekstreler, dökümleri ve ödeme. |
| **Ödeme Yöntemleri** | [console.komuta.io/account/billing/payment-methods](https://console.komuta.io/account/billing/payment-methods) | Kayıtlı kartlar ve varsayılan kart. |

Faturalandırma sayfaları yalnızca **organizasyon yöneticilerine** açıktır. Üyeler bu sayfaları menüde görmez; ödenmemiş bir ekstre olduğunda üst banttaki uyarıda "Bakiyeyi kapatması için organizasyon yöneticinize başvurun — faturalandırma sayfaları yalnızca yöneticilere açıktır." yazar. Yöneticiler kullanıcı menüsünde bakiyeyi, hesap seviyesini ve **Bakiye yükle** düğmesini de görür.

> **Not:** Her organizasyonun cüzdanı, ekstreleri, kartları ve harcama limiti ayrıdır. Birden fazla organizasyona üyeyseniz faturalandırma sayfaları o an seçili organizasyonu gösterir.

---

## Nasıl Faturalandırılırsınız

- **Para birimi:** USD. Fiyatlara vergi dahil değildir; vergi, fatura profilinizdeki adrese göre hesaplanır ve ekstrede ayrı satırda görünür.
- **Ölçüm:** Kullanım saatlik ölçülür. Aylık fiyatlar 720 saat üzerinden hesaplanır; ay ortasında açtığınız ya da sildiğiniz bir kaynak yalnızca çalıştığı saatler kadar faturalanır.
- **Fiyatı önceden görürsünüz:** Servis sihirbazında paket fiyatları, yönetilen servis sihirbazında ve Stack planında **Tahmini aylık maliyet** gösterilir.

| Ne faturalanır | Fiyat |
|---|---|
| Servisler ve işler | Seçtiğiniz kaynak paketine göre; fiyat oluşturma sihirbazında gösterilir. Servis paketleri: 0,5 vCPU / 1 GB 9 USD, 1 vCPU / 2 GB 18 USD, 2 vCPU / 4 GB 36 USD, 4 vCPU / 8 GB 72 USD (aylık). Birden fazla replikayla çalışan servisin her replikası ayrı ölçülür. |
| Yönetilen servisler (PostgreSQL, Valkey, RabbitMQ) | Planın CPU limiti, bellek limiti ve diski üzerinden: vCPU başına aylık 12,85 USD, GiB bellek başına aylık 5,14 USD, GiB disk başına aylık 0,1447 USD. |
| API Gateway | Aylık 9 USD (saatlik 0,0125 USD) + gateway'in kullandığı işlem kaynakları. |
| Build dakikaları | Her fatura döneminde 2.000 dakika ücretsiz; sonrası dakika başına 0,008 USD. Kuyrukta beklenen süre faturalanmaz. |
| Bant genişliği planları | Starter ücretsiz (varsayılan), Growth 4,99 USD, Business 14,99 USD, Enterprise 49,99 USD, Scale 149,99 USD (aylık). Plan ücreti trafik olmasa da saatlik işler. |
| Observability depolama | GB-saat başına 0,000342 USD (GB başına aylık yaklaşık 0,25 USD). |
| Her zaman açık eklentisi | Fatura dönemi başına 3 USD + vergi; dönemin yalnızca açık kaldığı kısmı kadar. |

Her organizasyon bir servisi ücretsiz pakette çalıştırabilir; bu servis faturalanmaz. Ücretsiz planın kuralları ve **Her zaman açık** eklentisinin koşulları [Servis Uykusu](service-sleep.md) sayfasındadır. Güncel fiyat listesinin tamamı [Fiyatlandırma](https://www.komuta.io/pricing) sayfasındadır.

---

## Fatura Dönemi ve Ekstreler

Fatura dönemi her ayın 6'sında 00:00'da (UTC) başlar ve bir sonraki ayın 6'sında biter. Dönem kapandıktan sonra, ayın 6'sında o dönemin ekstresi kesilir.

- Dönem içinde faturalanacak kullanım yoksa ekstre kesilmez.
- Toplam 0,50 USD'nin altındaysa ekstre kesilmez; kullanım bir sonraki döneme devreder.
- Ölçümü gecikmeli tamamlanan kullanım bir sonraki ekstreye eklenir. Bir ekstrede birden fazla dönemin kullanımı varsa satır açıklamasında dönemin ayı yazar (örneğin `[2026-09]`).

Ekstre kesildiğinde tutar şu sırayla kapatılır:

1. Kupon veya kampanya indiriminiz varsa düşülür.
2. Cüzdanınızdaki **Kredi** (kupon, kampanya ve hediye kredisi) vergi eklenmeden önce düşülür.
3. Kalan tutara vergi eklenir.
4. Cüzdanınızdaki **Yüklenen bakiye** vergi sonrası tutara uygulanır.
5. Geriye kalan tutar varsayılan kartınızdan otomatik çekilir.

Ekstrenin **Son ödeme** tarihi kesildiği gündür; ayrıca bir ödeme süresi yoktur.

### Ekstreler sayfası

**Hesap → Ekstreler** sayfası her ekstreyi bir satırda gösterir: **Fatura**, **Dönem**, **Durum**, **Tutar** ve **Tarih**. **Fatura** sütununda fatura numarası görünür; fatura henüz düzenlenmediyse **Henüz kesilmedi** yazar. İlk ekstreniz, ilk fatura döneminiz kapandığında burada görünür.

Bir satıra tıkladığınızda ekstre açılır:

- **Toplam**, **Ara toplam**, **Vergi**, **Son ödeme** ve ödendiyse **Ödendi** tarihi. Vergi hesaplanana kadar **Vergi** alanında "—" görünür.
- Maliyetin kategorilere dağılımı: **Yönetilen Servisler**, **Servisler ve İşler**, **API Gateway**, **Ağ**, **Gözlemlenebilirlik**.
- Her kategoride kaynaklar; bir kaynağa tıklayınca sayaçları (**Sayaç**, **Adet**, **Birim Fiyat**, **Tutar**).
- Kupon veya kampanya indirimi varsa, ekstrenin tamamından düşen ayrı bir eksi satır.

Toplamın yanındaki bilgi simgesi bu ekranın bir kullanım dökümü olduğunu hatırlatır: vergisi, numarası ve PDF'i olan fatura ödeme sağlayıcımız tarafından kesilir ve **Faturayı görüntüle** ile açılır.

### Ekstre durumları

| Durum | Anlamı |
|---|---|
| **Taslak** | Ekstre hazırlanıyor. Kesilmiş ekstrelerde normalde görmezsiniz. |
| **Düzenlendi** | Ekstre kesildi ve tahsilata gönderildi. Varsayılan kartınızdan çekim yapılır; başarısız olursa çekim otomatik olarak birkaç gün boyunca yeniden denenir. |
| **Ödendi** | Tutar tahsil edildi. |
| **Tahsil edilemedi** | Yeniden denemeler bitti ve tutar tahsil edilemedi. Organizasyon ödemesiz döneme girer; ekstreyi hâlâ ödeyebilirsiniz (bkz. **Ödenmemiş Ekstreler** bölümü). |
| **İptal Edildi** | Ekstre Komuta tarafından iptal edildi; ödenecek bir şey yoktur. |

### Ekstreyi elle ödemek

Açık bir ekstre (**Düzenlendi** veya **Tahsil edilemedi**) satırı açıldığında duruma göre şu düğmeler çıkar:

| Düğme | Ne zaman görünür | Ne yapar |
|---|---|---|
| **Faturayı görüntüle ve öde** | Fatura kesilmişse | Faturayı yeni sekmede açar. Kartla, gerekirse 3-D Secure doğrulamasıyla ödeyebilir, PDF'i indirebilirsiniz. Ödenmiş ekstrelerde düğmenin adı **Faturayı görüntüle**'dir. |
| **Bakiyeden öde ({amount})** | Fatura kesilmişse ve **Yüklenen bakiye**niz varsa | Faturayı bakiyeniz düşülerek yeniden düzenler. Bakiye tutarın tamamını karşılarsa "Bakiyenizden ödendi. Bu ekstrede ödenecek bir şey kalmadı." mesajı çıkar. |
| **Şimdi öde** | Ekstre borçlu ama faturası henüz kesilmemişse | Faturayı keser; bakiyeniz varsa otomatik uygulanır. Ardından "Faturanız hazır. Ödemek için "Faturayı görüntüle ve öde" düğmesini kullanın." mesajı çıkar. |

> **Not:** **Şimdi öde** fatura profilinizde adres eksikse çalışmaz; konsol sizi **Fatura Profili** sayfasına götürür.

> **Uyarı:** Kesilmiş bir faturası olan ekstreyi, cüzdana sonradan yüklediğiniz bakiye kendiliğinden kapatmaz. Böyle bir ekstreyi **Bakiyeden öde** ya da **Faturayı görüntüle ve öde** ile ödeyin. Açık faturanız varken bakiye yüklerseniz yükleme penceresi de bunu hatırlatır.

---

## Güncel Harcama

**Hesap → Güncel Harcama** içinde bulunduğunuz fatura döneminin o ana kadarki maliyetini ürün bazında gösterir.

- **Mevcut Döngü Toplamı** — dönemin şu ana kadarki tutarı. **Faturalama Dönemi** dönemin başlangıç ve bitişini, **Takip Bitişi** ölçümün hangi saate kadar tamamlandığını gösterir. İçinde bulunduğunuz saat henüz eklenmez.
- **Build dakikaları** — dönemde kullanılan dakikalar ve ücretsiz kotadan kalan ya da aşan dakika. Süre build başladığı andan bittiği ana kadar sayılır; kuyrukta beklenen süre faturalanmaz, iptal edilen build yalnızca iptal edildiği ana kadar faturalanır. Ücretsiz kotayı aşan dakikalar bu dönemin ekstresine yansır.
- **Maliyet Dağılımı** ve **Geçmiş Dönemler** — dönem harcamasının ürünlere dağılımı ve son dönemlerin toplamları.
- **Tüm Kullanımlar** — toplamı oluşturan her ölçülen kalem: **Ürün**, **Plan / Kaynak**, **Sayaç**, **Miktar**, **Birim Fiyat**, **Tutar**. Aynı kaynağın birden fazla sayacı tek satırda toplanır; satıra tıklayınca açılır.

Bu sayfadaki tutar vergi, kredi ve indirim uygulanmadan önceki kullanım tutarıdır.

---

## Cüzdan

**Hesap → Cüzdan** sayfası bakiyenizi, harcama limitinizi, hesap seviyenizi, kuponlarınızı ve tüm cüzdan hareketlerini bir arada gösterir.

### Bakiye: yüklenen bakiye ve kredi

**Bakiye** iki ayrı havuzdan oluşur. Bilgi simgesine tıklayınca **Bakiye dağılımı** açılır.

| | **Yüklenen bakiye** | **Kredi** |
|---|---|---|
| Kaynağı | Kartınızla yüklediğiniz tutar | Kayıt bonusu, kupon ve kampanya kredisi |
| Süresi | Dolmaz | Promosyon kredilerinin son kullanma tarihi vardır |
| Faturada kullanımı | Vergi eklendikten sonra | Önce bu kullanılır, vergi eklenmeden düşülür |

- Krediniz varsa bakiyenin altında iki havuzun tutarı ayrı gösterilir. Süresi dolacak bir kredi varsa "{amount} son kullanım {date}" etiketi çıkar.
- Süresi dolan ve harcanmamış promosyon kredisi bakiyeden düşer; **İşlemler** listesinde **Promosyon kredisi sona erdi** olarak görünür.
- Kayıtla açılan ilk organizasyona 30 gün geçerli 10 USD hoş geldin kredisi yüklenir (**Kayıt bonusu**).

### Bakiye yükleme

Bakiye yüklemeden önce **Fatura Profili** sayfasındaki adresin eksiksiz olması gerekir.

1. **Hesap → Cüzdan** sayfasında **Bakiye yükle**'ye tıklayın.
2. Hazır tutarlardan (10, 25, 50, 100, 300 USD) birini seçin ya da **Özel tutar** alanına bir tutar yazın. Tek seferde en az 5, en çok 1.000 USD yükleyebilirsiniz.
3. Kayıtlı kartınız varsa **Ödeme yöntemi** altında kartı seçin; başka bir kartla ödemek için **Yeni kart kullan**'ı seçin.
4. **Ödemeye devam et**'e tıklayın.
5. Yeni kart seçtiyseniz kart bilgilerini girin, ardından **{amount} öde**'ye tıklayın. Bankanız isterse 3-D Secure doğrulamasını tamamlayın.
6. Ödeme tamamlanınca "**{amount} yüklendi**" ve **Yeni bakiye** görünür. Ödemenin onaylanması uzun sürerse **Ödeme alındı** mesajı çıkar; bakiye ödeme tamamlandığında kendiliğinden görünür, sizden başka bir işlem beklenmez.

Yükleme işleminin kendisinden ayrıca vergi alınmaz; yüklenen tutar gelecek ekstrelerinizde vergi sonrası tutara uygulanır. Yüklemeler hesap seviyenizi de yükseltir (bkz. **Harcama Limiti ve Hesap Seviyeleri** bölümü).

"Önce bir ödeme yöntemi ekleyin: bakiye yükleme için fatura hesabı gerekir ve bu hesap, Hesabım → Faturalandırma bölümünde ilk kartınızı kaydettiğinizde oluşturulur." hatasını görürseniz önce **Hesap → Ödeme Yöntemleri** sayfasından bir kart ekleyin.

### İşlemler

**İşlemler** listesi cüzdana giren ve çıkan her hareketi gösterir. **Tümü**, **Giren** ve **Çıkan** sekmeleriyle süzebilir, tarihe, tutara ya da son kullanma tarihine göre sıralayabilirsiniz. Promosyon kredileri **Promosyon** etiketi taşır. Bir satıra tıklayınca **İşlem No**, varsa **Ödeme No** ve **Fatura No** görünür; bir ekstreye bağlı hareketlerde **Faturayı görüntüle** sizi o ekstreye götürür.

| Hareket | Anlamı |
|---|---|
| **Bakiye yükleme** | Kartınızla yüklediğiniz tutar. |
| **Kayıt bonusu** | Kayıtta verilen hoş geldin kredisi. |
| **Kupon kredisi** / **Kampanya kredisi** | Bir kupon, promosyon kodu ya da kampanyadan gelen kredi. |
| **Fatura ödemesi** | Kredinin bir ekstreye uygulanması. |
| **Bakiye faturaya uygulandı** | Yüklenen bakiyenin bir faturaya uygulanması. |
| **Promosyon kredisi sona erdi** | Süresi dolan, harcanmamış kredi. |
| **İade** | Size iade edilen tutar. |
| **Manuel düzeltme** | Komuta ekibinin yaptığı bakiye düzeltmesi. |

---

## Ödeme Yöntemleri

**Hesap → Ödeme Yöntemleri** fatura ödemeleri ve bakiye yüklemeleri için kayıtlı kartlarınızı gösterir. Ekstreler her zaman **varsayılan** karttan çekilir.

Kart eklemek için:

1. **Hesap → Fatura Profili** sayfasında ülke, adres, şehir ve posta kodunun dolu olduğundan emin olun. Değilse kart formu açılmaz ve "Kart kaydetmeden veya bakiye yüklemeden önce fatura adresinizi girin. …" hatası çıkar.
2. **Kart ekle**'ye tıklayın.
3. Kart bilgilerini girin ve **Kartı kaydet**'e tıklayın. Bankanız isterse 3-D Secure doğrulamasını tamamlayın.

Kart numaranızın tamamı Komuta'da saklanmaz; kart ödeme sağlayıcımızda güvenli şekilde tutulur.

- **Doğrulama:** Kart eklenirken 1 USD'lik geçici bir provizyon alınır ve hemen iptal edilir; kartınızdan para çekilmez. Provizyon başarılı olursa kart **Doğrulandı** etiketi alır; hesabınız **Starter** seviyesindeyse **Verified** seviyesine yükselir. Doğrulanamayan kart yine de kaydedilir.
- **Varsayılan kart:** Yeni eklediğiniz kart varsayılan olur ve **Varsayılan** etiketi taşır. Başka bir kartı varsayılan yapmak için o kartta **Varsayılan Yap**'a tıklayın.
- **Kart kaldırma:** Kartın üzerine gelip çarpı simgesine tıklayın ve **Kartı kaldır** ile onaylayın. Varsayılan kartı kaldırırsanız en son eklenen diğer kartınız varsayılan olur.

> **Uyarı:** Tek kartınızı kaldırırsanız otomatik tahsilat durur: bir sonraki ekstreniz ödenmez ve yeni bir kart ekleyene kadar hizmetleriniz kısıtlanabilir.

---

## Fatura Profili

**Hesap → Fatura Profili**, faturalarınızda yer alan yasal kimlik bilgilerinizi ve fatura e-postalarınızı tutar. Faturaların muhasebe açısından geçerli olması için bilgileri güncel tutun.

**Yasal kimlik** bölümü:

| Alan | Açıklama |
|---|---|
| **Yasal unvan** | Kayıtlı şirket veya kişi adı. |
| **Müşteri tipi** | **Kurumsal** veya **Bireysel**. |
| **Vergi kimlik no** | Yalnızca **Kurumsal** seçiliyken görünür; ülke `TR` ise adı **Vergi Kimlik No (VKN)** olur. |
| **Vergi numarası türü** | Yalnızca **Kurumsal** seçiliyken görünür; vergi numarası girdiyseniz zorunludur: **UK VAT**, **EU VAT** veya **Türkiye VKN / TIN**. |
| **Adres satırı 1**, **Adres satırı 2**, **Şehir**, **Posta kodu** | Fatura adresi. |
| **Ülke** | 2 harfli ISO ülke kodu (örneğin `TR`, `DE`, `GB`). |
| **Faturalandırma dili** | **Türkçe** veya **İngilizce**; faturanın dili. |

Kaydetmek için **Değişiklikleri Kaydet**'e tıklayın.

**Ülke**, **Adres satırı 1**, **Şehir** ve **Posta kodu** zorunludur (posta kodu sistemi olmayan ülkelerde posta kodu istenmez). Vergi bu adrese göre hesaplandığı için adres eksikken:

- Kart ekleyemez, bakiye yükleyemez ve **Şimdi öde** ile ekstre ödeyemezsiniz.
- Kullanımınız ölçülmeye devam eder ama faturası kesilemez; ekstre **Ekstreler** sayfasında **Henüz kesilmedi** olarak kalır. Üst bantta **Fatura adresiniz eksik** uyarısı ve **Fatura adresi ekle** düğmesi görünür; profil sayfasında **Faturalandırma durduruldu — adresiniz eksik** uyarısı çıkar.
- Faturası kesilemeyen ekstre, oluşturulduğu tarihten 10 gün sonra organizasyonu ödemesiz döneme alır (bkz. **Ödenmemiş Ekstreler** bölümü).

**Yasal unvan** boşsa ekstreler yine kesilir, ancak geçici kimlik bilgileriyle düzenlenir.

**Kurumsal** müşteriyseniz vergi numaranızı ve türünü girin. Ödeme sağlayıcımız numarayı doğrular; numara doğrulanamazsa reverse charge uygulanabilecek durumlarda bile KDV alınır.

### Fatura e-postaları

**Fatura e-postaları** bölümüne fatura ve ödeme bildirimlerinin gideceği en fazla üç adres girebilirsiniz: **Birincil e-posta** ve iki **Ek e-posta**. Organizasyon açıldığında birincil adres, organizasyonu açan kişinin e-posta adresidir. Bu adreslere örneğin şu bildirimler gider:

- Kart çekimi başarısız olduğunda, ödeme bağlantısıyla birlikte.
- Bakiye yüklendiğinde.
- Organizasyonun ödeme durumu değiştiğinde (ödemesiz dönem, askıya alma, yeniden etkinleşme).
- Hesap seviyeniz yükseldiğinde ve yeni bir işlem tahmini aylık harcamanızı limitin %80'ine ya da üstüne taşıdığında.
- Adres eksik olduğu için ekstre kesilemediğinde.

---

## Harcama Limiti ve Hesap Seviyeleri

Her organizasyonun bir **Aylık kaynak maliyeti tahmin limiti** vardır. Yeni bir kaynak oluşturduğunuzda ya da mevcut bir kaynağı büyüttüğünüzde (paket, replika, otomatik ölçeklendirme üst sınırı, yönetilen servis planı, bant genişliği planı gibi) Komuta, organizasyonun tahmini aylık maliyetini bu değişiklikle birlikte hesaplar. Tahmin limiti aşacaksa işlem yapılmaz ve şu mesaj çıkar:

> Bu işlemle tahmini aylık harcamanız ‹yeni tahmin› USD olacak; mevcut harcama sınırınız ‹limit› USD. Devam edebilmek için sınırınızın en az ‹fark› USD daha yüksek olması gerekiyor. Sınırınız cüzdanınıza yükleme yaptıkça ve kartınızı doğruladıkça yükselir; Hesabım → Faturalandırma bölümünden yükleme yapabilirsiniz.

Limit yalnızca oluşturma ve büyütme anında kontrol edilir. Zaten çalışan kaynaklar durdurulmaz; build dakikaları gibi ölçülen kullanım limiti aşabilir.

**Hesap → Cüzdan** sayfasındaki halka, bu dönemde kaydedilen harcamanın (**Güncel Dönem Toplamı**) limitinize oranını gösterir. Halkanın altında seviyenizin adı ve bir sonraki seviyeye geçmek için **{tier} seviyesine yüksel** bağlantısı bulunur; bağlantı sonraki seviyenin limitini ve koşullarını gösterir.

### Hesap seviyeleri

Her yeni organizasyon **Starter** seviyesinde, aylık 20 USD limitle başlar. Seviyeler aşağıdaki koşullardan **herhangi biri** sağlandığında kendiliğinden açılır:

| Seviye | Aylık limit | Nasıl açılır | Eşzamanlı build (kuyrukta en fazla) |
|---|---|---|---|
| **Starter** | 20 USD | Başlangıç seviyesi | 1 (5) |
| **Verified** | 50 USD | Doğrulanmış bir kart ekleyin | 1 (10) |
| **Basic** | 100 USD | Toplam 10 USD bakiye yükleyin **veya** toplam 10 USD ekstre ödeyin | 2 (20) |
| **Pro** | 300 USD | Toplam 50 USD bakiye yükleyin **veya** toplam 50 USD ekstre ödeyin | 4 (30) |
| **Scale** | 1.000 USD | Toplam 300 USD bakiye yükleyin **veya** toplam 300 USD ekstre ödeyin | 8 (50) |

- Bakiye yükleme toplamı yüklediğiniz tutarların toplamıdır; bakiyeyi harcamanız seviyenizi düşürmez. İade edilen bir yükleme toplamdan düşer.
- Seviyeniz yükseldiğinde fatura e-postalarınıza bildirim gider.
- Limiti kendiniz ayarlayamazsınız. Scale üstüne çıkmak ya da limitinizi daha erken artırmak için **Hesap → Cüzdan** sayfasında **Limit artışı talep et**'e tıklayın. Mevcut limitinizin yazılı olduğu bir [destek talebi](support-tickets.md) formu açılır; talep ettiğiniz limiti ve kullanım planınızı yazıp gönderin. Ekibimiz talebi inceler.

> **Not:** Ödemesiz dönemdeki ya da askıya alınmış bir organizasyonda harcamayı artıran işlemler, limit ne olursa olsun yapılamaz. Ayrıntılar **Ödenmemiş Ekstreler** bölümündedir.

---

## Kuponlar ve Promosyonlar

Kupon ya da promosyon kodunu kayıt sırasında veya sonradan **Hesap → Cüzdan** sayfasındaki **Kupon ve promosyonlar** kartında kullanabilirsiniz.

1. **Hesap → Cüzdan** sayfasında **Kod kullan**'a tıklayın (ya da doğrudan **Kupon ve promosyonlar** kartına inin).
2. **Kupon veya promosyon kodu** alanına kodu yazın. Kod yazarken kontrol edilir ve ne kazandıracağı gösterilir.
3. **Kodu uygula**'ya tıklayın. "{code} kodu uygulandı" mesajı çıkar ve kod **Aktif teklifler** listesine eklenir.

Bir kod şunlardan birini ya da birkaçını sağlayabilir:

- **Kredi** — cüzdanınızın **Kredi** havuzuna eklenir, kendi bakiyenizden önce harcanır ve genellikle bir son kullanma tarihi vardır. Bazı kredilerde "e-postanı doğruladığında eklenir" yazar; kredi e-posta adresinizi doğruladığınızda yüklenir.
- **Yüzde indirim** — belirli bir süre ya da tarihe kadar ekstrelerinizden yüzde indirim; ekstrede ayrı bir eksi satır olarak görünür.
- **Ek ücretsiz servis** — ücretsiz pakette çalıştırabileceğiniz servis sayısını artırır.

Kurallar:

- Her kod bir organizasyonda yalnızca bir kez kullanılabilir.
- Bir organizasyonda en fazla bir partner kuponu olabilir ve partner kuponu ilk ödenmiş faturadan önce uygulanmalıdır. Promosyon kodları istediğiniz zaman eklenebilir.
- Yalnızca yeni organizasyonlar için çıkarılan kodlar kayıtta kullanılmadıysa, **Cüzdan** sayfasından kayıttan sonraki 30 gün içinde, yalnızca kayıtla açılan organizasyonda ve ilk ödenmiş ekstreden önce uygulanabilir.
- Kampanyalar kod gerektirmez; organizasyonunuza otomatik uygulanır ve listede "Otomatik uygulandı — kod gerekmez" olarak görünür.
- **Her zaman açık** eklentisine kupon ve kampanya indirimleri uygulanmaz.

Kayıtta girdiğiniz kod uygulanamadıysa kartın üstünde "Kayıtta girdiğiniz {code} kodu uygulanamadı" uyarısı çıkar; başka bir kodunuz varsa aynı karttan uygulayabilirsiniz.

Sık görülen ret mesajları:

| Mesaj | Anlamı |
|---|---|
| Bu kupon kodunu tanımıyoruz. | Kod yanlış yazılmış ya da yok. |
| Kupon kodu 6–32 karakter olmalı: harf, rakam ve tire. | Kodun biçimi geçersiz. |
| Bu kuponun süresi dolmuş. / Bu kupon henüz geçerli değil. | Kod geçerlilik tarihleri dışında. |
| Bu kuponun kullanım limiti dolmuş. | Kodun toplam kullanım hakkı bitti. |
| Bu kod bu organizasyonda daha önce kullanıldı. | Aynı kod ikinci kez kullanılamaz. |
| Bu organizasyonda zaten bir partner kuponu var. Promosyon kodları yine de eklenebilir. | Partner kuponu hakkı dolu. |
| Partner kuponu yalnızca ilk ödenmiş faturanızdan önce uygulanabilir. | Organizasyon daha önce ekstre ödedi. |

---

## Ödenmemiş Ekstreler

Bir ekstre ödenmediğinde organizasyon önce **ödemesiz döneme** girer, ödenmezse **askıya alınır**.

### Ödemesiz döneme giriş

1. Ekstre kesildiğinde varsayılan kartınızdan çekim yapılır. Çekim başarısız olursa fatura e-postalarınıza ödeme bağlantısını içeren bir bildirim gider ve çekim otomatik olarak birkaç gün boyunca yeniden denenir.
2. Yeniden denemeler biterse ekstrenin durumu **Tahsil edilemedi** olur ve organizasyon ödemesiz döneme girer.

Fatura adresiniz eksik olduğu için faturası kesilemeyen bir ekstre de, oluşturulduğu tarihten 10 gün sonra organizasyonu ödemesiz döneme alır.

### Ödemesiz dönemde

- Çalışan servisleriniz ve yönetilen servisleriniz çalışmaya devam eder.
- Aylık harcamayı artıran işlemler (ölçeklendirme, paket veya plan yükseltme, yeni ücretli kaynak) yapılamaz. Şu mesaj çıkar: "Bir faturanız tahsil edilemediği için hesabınız ödemesiz dönemde. Aylık harcamanızı artıran işlemler, bakiye kapatılana kadar durdurulmuştur. Devam etmek için Hesabım → Faturalandırma bölümünden faturanızı ödeyin."
- Konsolun üstünde **Ödenmemiş bir fatura nedeniyle hesabınız ödemesiz dönemde** uyarısı görünür. Uyarı ödenmemiş fatura sayısını, toplam tutarı ve hesabın askıya alınacağı tarihi yazar; yöneticiler için **Faturalandırmaya git** düğmesi vardır.

### Askıya alma

Ödemesiz dönem **14 gün** sürer. Bu sürenin sonunda gecikmiş ekstre hâlâ ödenmemişse organizasyon askıya alınır:

- Tüm servisleriniz uyku moduna alınır ve PostgreSQL veritabanlarınız durdurulur.
- Konsolda değişiklik yapılamaz; her şeyi görüntüleyebilirsiniz. Ekstre ödeme, kart ekleme ve fatura profilini düzenleme açık kalır.
- Konsolun üstünde **Ödenmemiş faturalar nedeniyle hesabınız askıya alındı** uyarısı görünür.

### Hesabı geri açmak

1. **Hesap → Ekstreler** sayfasında **Tahsil edilemedi** durumundaki ekstreyi açın.
2. Kartınız geçersizse önce **Hesap → Ödeme Yöntemleri** sayfasından yeni bir kart ekleyin.
3. **Faturayı görüntüle ve öde** ile kartla ödeyin ya da yüklenmiş bakiyeniz varsa **Bakiyeden öde**'yi kullanın.

Gecikmiş ekstre kalmadığında organizasyon kendiliğinden normale döner. Askıya alma sırasında uyutulan servisler uyandırılır ve durdurulan PostgreSQL veritabanları yeniden başlatılır. Askıdan önce sizin uyuttuğunuz servisler ya da ücretsiz saatleri dolan ücretsiz servisler uykuda kalır.

> **Uyarı:** Bakiye yüklemek, faturası kesilmiş bir ekstreyi kendiliğinden kapatmaz ve hesabı askıdan çıkarmaz. Ekstreyi yukarıdaki adımlarla ödeyin.

---

## Servis Bazında Maliyet

Bir servisin ne kadar tuttuğunu görmek için servis sayfasında sol menüden **Fatura** sekmesini açın:

- **Bu dönem ücretleri** — servisin içinde bulunduğunuz dönemdeki tutarı. Yanında **Faturalama Dönemi** ve **Takip Bitişi** yazar.
- **Bu dönem dökümü** — servise ait kalemler: **Kalem**, **Kullanım**, **Oran**, **Tutar**. Servisin kaynak kullanımı ve bant genişliği planı burada görünür.
- **Ekstre geçmişi** — servisin yer aldığı geçmiş ekstreler, durumları ve servisin her ekstredeki payı. Satıra tıklayınca servisin o ekstredeki kalemleri açılır.

Yönetilen servisler, API Gateway ve build dakikaları bu sekmede değil, **Hesap → Güncel Harcama** ve **Hesap → Ekstreler** sayfalarında görünür.

---

## İlgili Dokümanlar

- [Hızlı Başlangıç](quick-start.md) — Hesap, organizasyon, ücretsiz başlangıç ve maliyetin özeti.
- [Servis Uykusu](service-sleep.md) — Ücretsiz plan, ücretsiz saat havuzu ve **Her zaman açık** eklentisi.
- [Bant Genişliği Planları](bandwidth-plans.md) — Planlar, hız sınırları ve aylık ücretler.
- [Observability](observability.md) — Telemetri depolama planları ve ücretleri.
- [Build Kuyruğu](build-queue.md) — Seviyeye göre eşzamanlı build sınırları.
- [Limitler](limits.md) — Platformdaki tüm sınırlar tek yerde.
- [Organizasyonlar ve Ekip](organizations-team.md) — Organizasyon yöneticisi rolü ve üyeler.
- [Destek Talepleri](support-tickets.md) — Limit artışı ve fatura soruları için destek talebi açma.
