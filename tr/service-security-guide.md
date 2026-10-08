# Servis Güvenliği

Tek bir serviste neler olduğunu anlayın, kanıtları inceleyin ve etkisini bilerek müdahale seçin. **Servis detayı → Güvenlik**, servisin bulgularını, trafiğini ve koruma durumunu aynı çalışma alanında birleştirir.

Kuruluş genelinde önceliklendirme için [Güvenlik Merkezi](security-center-guide.md) kullanın; ardından davranışı incelemek için ilgili servisi açın. Servisin genel adresini kimlerin açabileceğini yönetmek için **Erişim ve portlar** üzerindeki [Erişim koruması](service-access-protection.md) ayarlarını kullanın.

> **İyi bir inceleme üç adımdan oluşur:** servis ve kanıt kapsamını kontrol edin, ilgili kayıtları karşılaştırın, ardından yetkili değişikliğin sonucunu doğrulayın. Kaydedilen ayar niyeti gösterir; güncel uygulama durumu ve davranış kanıtı ne olduğunu ortaya koyar.

> Görseller, 8 Ekim 2026 tarihinde Komuta arayüzündeki test ortamından alınmıştır. Gösterilen durumlar örnektir; kendi servisinizin güncel durumunu kontrol edin.

## İlk kullanım

1. Doğru kuruluşu seçin ve incelemek istediğiniz servisi açın.
2. **Güvenlik → Genel bakış** bölümüne geçin. Özeti yorumlamadan önce çalışma ortamını, kapsamı ve veri güncelliğini kontrol edin.
3. Acil bir bulgudan, ağ olayından veya düşen bağlantı özetinden ilgili inceleme bölümüne geçin.
4. Olayı içeren bir zaman aralığı seçin. Etkin filtreleri kontrol edin ve sunuluyorsa ek kayıtları yükleyin.
5. Kanıtın bir kural değişikliği, servis ayarı veya olay müdahalesi gerektirip gerektirmediğine karar vermeden önce **Koruma** bölümünü inceleyin.

Çalışma zamanı ve trafik sonuçları için servisin uygun bir dağıtımı ve kullanılabilir kanıt kaynakları olmalıdır. Henüz dağıtılmamış bir servis, çalışan bir servisle aynı kanıtı üretmeye hazır değildir.

Rolünüz de önemlidir. Bulguları okumak, olay zaman çizelgesini görmek, güvenlik duruşunu incelemek, trafiği görüntülemek ve değişiklik yapmak farklı izinler gerektirebilir. Salt okunur bir kart rolünüz için normal olabilir. Göreviniz daha fazla erişim gerektiriyorsa kuruluş yöneticinizden ekranda belirtilen ilgili izni isteyin.

## Doğru sekmeyi bulun

| Sekme | Kullanım amacı | Başlangıç noktası |
|---|---|---|
| **Genel bakış** | Servisin acil bulgularını, ağ olaylarını ve kanıt eksiklerini önceliklendirmek | Servis kimliği, kapsam ve veri güncelliği |
| **Trafik** | Akışları, olayları, düşen bağlantıları ve sunulan DNS ayrıntılarını incelemek | Kanıt zaman aralığı ve bağlantı filtreleri |
| **Koruma** | Ağ politikalarını, canlı duruşu, çalışma zamanı modunu ve desteklenen sıkılaştırma ayarlarını okumak | Kural kapsamı ve bildirilen mevcut durum |
| **Bulgular** | Bulguları, çalışma zamanı gözlemlerini ve kanıt zaman çizelgesini araştırmak | Kaynak, olay zamanı ve ilgili davranış |

Olay müdahalesi kartı çalışma alanında sekmeler arasında da görünür; inceleme yaparken izolasyon durumunu kontrol edebilirsiniz.

![komuta-test-app servisinin güvenlik özeti ve dört güvenlik sekmesi](https://raw.githubusercontent.com/Microzon-Tech/komuta-docs/main/img/security/service-overview.jpg)

*komuta-test-app üzerindeki Güvenlik ekranı: Genel bakış, Trafik, Koruma ve Bulgular sekmeleri aynı servis bağlamında çalışır. Üst kart mevcut duruşu, olay müdahalesi kartı ise izolasyon durumunu gösterir.*

## Genel bakış: önceliği belirleyin

Genel bakış, yüksek öncelikli bulguları ağ olayları ve düşen trafik özetleriyle birleştirir. İncelemeye **Bulgular** veya **Trafik** bölümünde devam etmek için ilgili özeti açın.

Bu özetleri kapsam bilgisiyle birlikte okuyun. Kapsam, seçilen servis için hangi kanıtın uygulanabilir ve kullanılabilir olduğunu gösterir. Kullanılamayan kaynak, eski veri veya başarısız sorgu, listede satır bulunmasa bile sonucu belirsiz bırakabilir.

Acil görünümü yalnız açık Kritik ve Yüksek önem dereceli bulguları gösterebilir. Daha geniş kayıt için **Tüm bulguları göster** seçeneğini kullanın. Sayaçlar gösterilen kapsamı anlatır; inceleme bekleyen gözlemler saldırı sayacı değildir ve bulgu bulunmaması tam bir güvenlik değerlendirmesi sayılmaz.

## Trafik: belirtiyi bağlantıyla eşleştirin

### Akışlar ve DNS

Kanıt zaman aralığını seçin; bağlantı akışını gerektiğinde protokol, sonuç ve bağlantı kaynağı veya hedefi filtreleriyle daraltın. Kaynak, hedef, yön ve portu uygulamanın beklenen davranışıyla karşılaştırın.

Akış görünümü güvenlikle ilgili bağlantıları ve sağlıklı trafiğin temsili bir örneklemini içerir. Kesin toplam veya her bağlantının eksiksiz kaydı olarak değil, inceleme için kullanın.

Servis ve kanıt kaynağı destekliyorsa DNS ve uygulama düzeyi ayrıntılar hedefi anlamanıza yardımcı olur. Bu ayrıntıların bulunmaması, ad çözümleme veya isteğin hiç gerçekleşmediğini göstermez. Genel adres ve port ayarları için [Erişim ve portlar](services-ports.md) bölümünü kullanın.

### Ağ olayları

Olaylar, ilişkili ağ anormalliklerini inceleme için bir araya getirir. Davranışın beklenen bir durum mu yoksa müdahale gerektiren bir sorun mu olduğuna karar vermeden önce etkilenen servisi, destekleyici kayıtları ve güncel yaşam döngüsünü okuyun.

Akışlar ve düşen bağlantılar seçilen kanıt zaman aralığını kullanır; olaylar ise güncel yaşam döngüsünü gösterir. Trafik zaman aralığını değiştirmek, olay listesini geçmişteki olay durumlarının anlık görüntüsüne dönüştürmez.

### Düşen bağlantılar

Hangi bağlantıların reddedildiğini ve bildirilen nedeni görmek için düşen trafik bölümünü kullanın. Bağlantıyı **Koruma** altındaki kurallarla ve uygulamanın belirtileriyle karşılaştırın.

Reddedilen tek bir bağlantı, o bağlantıya ait kanıttır. Bütün yolların kapalı olduğunu, servisin tamamen izole edildiğini veya benzer her girişimin aynı sonucu vereceğini kanıtlamaz.

## Koruma: mevcut güvenlik duruşunu anlayın

### Ağ politikaları

Servisin gelen ve giden trafik kurallarını gerekli bağımlılıklarıyla karşılaştırın. Müşteri isteği, sağlık kontrolü ve dışarıya yapılan bağımlılık bağlantısı farklı izinler gerektirebilir.

Rolünüz bir değişiklik veya politika akışı sunuyorsa uygulamadan önce hedefi ve önizlemeyi kontrol edin. Ardından bildirilen uygulama durumunu ve ilgili trafik sonucunu doğrulayın. Genel adreste oturum açma ve IP kısıtlamaları ayrı [Erişim koruması](service-access-protection.md) ayarlarıdır; ağ politikası ile ziyaretçi erişim kuralı farklı soruları yanıtlar.

### Temel güvenlik ve canlı duruş

Temel güvenlik, servisten beklenen güvenlik yapılandırmasıdır. **Canlı duruş**, çalışan iş yükü için sunulan kontrolleri raporlar; beklenen bir politikanın bulunmaması veya root kullanımının onaylı ayarla uyuşmaması gibi yapılandırma sapmalarını vurgular.

Sapmayı son dağıtım ve onaylı servis ayarlarıyla karşılaştırın. Sorgu hatası, duruşun belirlenemediği anlamına gelir; başarısız uygulama testi veya temiz sonuç değildir. Gösterilen temel durum, önizleme ve güncel çalışma zamanı kanıtı incelemenin farklı sorularını yanıtlar.

### Çalışma zamanı koruma modu

Uygun servislerde **Çalışma zamanı koruma modu**, salt okunur bir durum kartıdır. İstenen modu, gözlenen modu ve dağıtım durumunu ayrı gösterir. Modlar **Kapalı**, **Gölge**, **Denetim** ve **Zorlama** olarak görünür; anlamları için [Çalışma Zamanı Güvenliği](runtime-security-guide.md) rehberini okuyun.

| Durum | Nasıl yorumlanır? |
|---|---|
| **Bekliyor / sırada / uygulanıyor** | İstenen değişikliğin uygulandığı henüz doğrulanmamıştır; gözlenen modu inceleyin. |
| **Başarısız** | Kaydedilen isteğin uygulaması tamamlanmamıştır. Yeni işlemden önce bildirilen hatayı okuyun. |
| **Gözlenen mod** | Bildirilen güncel moddur. Etkili korumayı doğrulamak için ilgili davranış ayrıca incelenmelidir. |
| **Kullanılamıyor veya doğrulanmamış** | Bu görünümden güncel mod belirlenememektedir. |

**Denetim / Engelle** koruma eylemi, bu çalışma zamanı modlarından ayrı bir alandır. Engelle etiketi, ilgili kurallar için engelleme niyetini belirtir; her işlem için toplu bir başarı sonucu değildir.

### Linux capability değerleri

Capability değerleri uygulamaya belirli ayrıcalıklar verir. Kart, kullanılabilir izin listesini, mevcut eklemeleri ve varsayılanları gösterir. İlgisiz bir başlangıç hatasını gidermek için ayrıcalık eklemek yerine listeyi imajın ihtiyaç duyduğu ölçüde dar tutun.

Capability yönetimi izniyle gereken ekleme veya çıkarmaları inceleyin, gerekçe girin ve kaydedin. Ayrıcalık eklemek onay gerektirir. Değişiklik sonraki dağıtımda etkili olur; kayıt sonucunu okuyun ve o dağıtımdan sonra uygulama sağlığı ile canlı duruşu doğrulayın.

![Koruma sekmesindeki Linux capability izin listesi](https://raw.githubusercontent.com/Microzon-Tech/komuta-docs/main/img/security/linux-capabilities.jpg)

*Koruma sekmesindeki Linux capability kartında mevcut eklemeleri ve platform varsayılanlarını karşılaştırabilirsiniz. Görüntüdeki izinler test servisine aittir; uygulamanız için önerilen bir izin listesi değildir.*

### Yazılabilir yollar

**Yazılabilir yollar** kartı, kök dosya sistemi salt okunur olduğunda uygulamanın yazması gereken dizinleri tanımlar. Tanınan uygulama çatısı için otomatik sağlanan yollar varsa bunları da gösterir; bu yollar bu karttan düzenlenemez.

Yazılabilir yol yönetimi izniyle yalnız gerekli mutlak uygulama dizinlerini ekleyin, gerekçe girin ve onayı inceleyin. İzin verilen yol kuralları geçerliliğini korur. Mevcut ayarlar yüklenemiyorsa düzenlemeden önce başarıyla yenileyin.

Yazılabilir dizin, kendiliğinden kalıcı depolama anlamına gelmez. Kalıcılık seçeneği sunuluyorsa kapsamını okuyun ve uygulamanın dağıtımlar arasında neyi koruması gerektiğini değerlendirin. Her yazılabilir yolun kalıcı olduğunu varsaymadan servisin sonucunu doğrulayın.

![Koruma sekmesindeki yazılabilir yollar ve gerekçe alanı](https://raw.githubusercontent.com/Microzon-Tech/komuta-docs/main/img/security/writable-paths.jpg)

*Yazılabilir yollar kartı, ek dizinleri ve değişiklik gerekçesini aynı yerde gösterir. Ekrandaki /app/App_Data bir giriş örneğidir; kaydedilmiş bir yol değildir.*

### Root olarak çalışma izni

**Root olarak çalışmaya izin ver**, buna ihtiyaç duyan imajlar için ayrı ve ayrıcalıklı bir ayardır. Onay, bu gevşetmenin salt okunur kök dosya sistemi gerekliliğini de etkilediğini açıklar. Seçmeden önce imajın gerçek ihtiyacını kontrol edin; dar kapsamlı bir capability veya yazılabilir dizin, farklı bir gereksinimi daha az erişimle karşılayabilir.

Bu ayarı değiştirmek ayrı izin ve gerekçe gerektirir. Kaydedilmiş istisna, o anda çalışan işlemin kullanıcı kimliğine dair kanıt değildir. Sonuçlanan dağıtımı ve canlı duruşu inceleyin.

### Önizleme ve değişiklik doğrulama

Sunuluyorsa **YAML önizlemesi** ile planlanan temel güvenlik içeriğini inceleyin. Önizleme, değişikliği kaydetmez veya uygulamaz.

Desteklenen servis ayarlarında şu sırayı izleyin:

1. Mevcut ayarı, ilgili belirtiyi ve beklenen iyileşmeyi kaydedin.
2. Gereken en dar kapsamı değiştirin ve anlamlı bir gerekçe yazın.
3. Dağıtım uyarıları dahil onayı ve kayıt sonucunu okuyun.
4. Sonuçlanan [dağıtım geçmişini](service-deployment-history.md) ve güncel duruşu kontrol edin.
5. İstenen uygulama davranışının çalıştığını ve ilgili korumanın hâlâ destekleyici kanıta sahip olduğunu doğrulayın.

Ayar kaydedilmiş ancak uygulama başarısız olmuş veya sırada kalmışsa olay notlarınızda bu ayrımı koruyun. Belirsiz durumu tamamlanmış gibi göstermek için aynı değişikliği tekrar tekrar kaydetmeyin.

## Bulgular: kanıttan karara

### Bulguyu araştırın

Kaynak, önem derecesi, etkilenen işlem, ilk ve son görülme bilgisi ile sunulan kanıtı okumak için bulguyu açın. Ayrıntıları incelediğiniz servis ve zaman aralığıyla karşılaştırın. Tekrar bilgisi, tek seferlik olay ile yinelenen davranışı ayırt etmeye yardımcı olur.

Acil filtresi veya kayıtların yalnız bir bölümünün yüklenmiş olması listeyi daraltabilir. İnceleme gerektiriyorsa zaman aralığını genişletin veya daha fazla kayıt yükleyin. Eksik satırın nedeni filtre, izin veya kaynak kullanılabilirliği olabilir.

### Çalışma zamanı gözlem incelemesi

Uygun servislerde kaydedilmiş dosya yazmaları, işlem çalıştırmaları ve ağ girişimleri, mevcut inceleme geçmişiyle birlikte görünür. İncelemeyi daraltmak için **Tümü**, **Beklemede**, **İzinli**, **Engelli** veya **Gözardı** filtrelerini kullanın.

Bu bölüm, gözlemleri ve kaydedilmiş kararları okumak içindir. İşlem gerektiren güvenlik bulgusuna yetkili Bulgular akışından karşılık verin. Gözlem otomatik olarak saldırı değildir; İzinli veya Engelli inceleme durumu, ilgili çalışma zamanı kuralının etkili olduğunu kanıtlamaz.

### Kanıt zaman çizelgesi

İncelediğiniz davranışın öncesi ve sonrasındaki olayları karşılaştırmak için zaman çizelgesini kullanın. Bulgu ve zaman çizelgesi erişimleri bağımsızdır; birini görme izniniz varken diğerini göremeyebilirsiniz. Sonuç çıkarırken zaman aralığını ve kayıt yükleme sınırlarını dikkate alın.

### Kaydedilmiş karar ve gerçek müdahale

Bulguyu gördüğünüzü belirtmek, izin vermek, tehdit olarak işaretlemek, göz ardı etmek veya çözülmüş saymak bir inceleme kararı kaydeder. Tek başına karar, trafik kuralı uygulamaz, bir işlemin reddedildiğini kanıtlamaz veya servisi izole etmez.

Kullanılabilir müdahale eyleminde önizlemeyi, gerekli izni, kapsamı ve beklenen etkiyi inceleyin. Uygulanmayı ve sonucu ayrıca doğrulayın. [Güvenlik Merkezi](security-center-guide.md), bulguları, müdahale rehberlerini, önerileri ve yanıt akışlarını birlikte açıklar.

## İş yükü izolasyonu ve geri dönüş

İzolasyon, seçilen iş yükünün ağ bağlantılarını sınırlar; normal hizmeti ve bağımlılık erişimini kesintiye uğratabilir. Uygun iş yükü ve ayrı yetki gerektirir. Önizleme, onaydan önce önerilen kapsamı ve etkiyi gösterir.

| Kapsam | Amaçlanan etki |
|---|---|
| **Yalnız gelen trafik** | İş yükünü gelen istekleri karşılamaktan çıkarır; giden bağlantılar açık kalır. |
| **Gelen ve giden trafik** | Bağımlılıklara erişim dahil her iki yönü sınırlar. |

Yalnız gelen trafiğin kesilmesi tam karantina değildir. Kapsamı yalnız en kısa kesintiye göre değil, olayın ihtiyacına göre seçin.

1. Servisi, olay gerekçesini, beklenen etkiyi ve geri dönüş sorumlusunu doğrulayın.
2. **Bu iş yükünü izole et…** seçeneğini açın ve önizlemeyi okuyun. Kullanılamayan veya reddedilen önizleme, başarılı kontrol sayılmaz.
3. Yalnız yetkili olduğunuz, desteklenen kapsamı onaylayın ve gerekli gerekçeyi girin.
4. Bildirilen sonucu inceleyin. Bekliyor durumu istenen kısıtlamanın henüz doğrulanmadığını, başarısız durumu ise izolasyon varsayılmaması gerektiğini gösterir.
5. İlgili trafik kanıtını ve uygulama davranışını amaçlanan kısıtlamayla karşılaştırın.

Müdahale tamamlandığında yetkili kullanıcı **Bu iş yükünü yeniden bağla** seçeneğini açabilir, gerekçe girip onaylayabilir. Hem kısıtlamanın kaldırıldığını hem uygulamanın sağlıklı döndüğünü doğrulayın. Yeniden bağlama da bir güvenlik kararıdır; isteğin kabulü sağlıklı geri dönüşü kanıtlamaz.

## Çalışma ortamı uygunluğu

Sunulan kanıt ve işlemler seçilen servisin çalışma ortamına bağlıdır. **Kata** iş yükünde bazı davranış gözlemleri ve çalışma zamanı koruma işlemleri uygulanamaz. Ağ, derleme kanıtı, sıkılaştırma ve izolasyonun her biri kendi desteği ve bildirilen durumuna göre değerlendirilmelidir.

| Gösterilen koşul | İnceleme için anlamı |
|---|---|
| **Uygun ve güncel kanıt mevcut** | Kayıtları belirtilen kapsam ve zaman aralığında değerlendirebilirsiniz. |
| **Uygulanamaz** | Bu katman seçilen çalışma ortamı için geçerli değildir; diğer uygun kanıtları kullanın. |
| **Bilinmiyor / kullanılamıyor** | Destek veya güncel durum belirlenememektedir; gerekiyorsa açıklama isteyin. |
| **Eski veri** | Kanıt, mevcut durumu değerlendirmek için gerekenden eskidir. |

Sırf bir kart sağlıklı görünsün diye çalışma ortamını değiştirmeyin. Uyumluluğu ve uygulama gereksinimlerini servis sorumlunuzla veya Komuta desteğiyle doğrulayın.

## Pratik incelemeler

### Servis bir bağımlılığa ulaşamıyor

Olayın zaman aralığında **Trafik** bölümünü açın. İlgili hedefi ve portu bulun; sonucu ve varsa düşme nedenini inceleyin. **Koruma** içindeki giden trafik kurallarıyla ve izolasyon durumuyla karşılaştırın. Değişiklik gerekiyorsa izinli en dar kuralı veya düzeltmeyi kullanın; ardından gereken bağlantının çalıştığını ve ilgisiz kısıtlamaların korunduğunu doğrulayın.

### Yeni imaj dağıtımdan sonra çalışmıyor

Hata zamanını dağıtım geçmişi, canlı duruş ve ilgili gözlemlerle karşılaştırın. Uygulamanın belirli bir yazılabilir dizin, capability veya root izni gerektirip gerektirmediğini belirleyin. Ayarı gevşetmeden önce imaj ve hata kanıtını kullanın. Yetkili düzeltmeden sonra hem uygulama sağlığını hem sonuçlanan güvenlik durumunu kontrol edin.

### Şüpheli davranış tekrarlanıyor

Bulgunun tekrar bilgisi ile ilk ve son görülme zamanlarını kontrol edin; zaman çizelgesi ve trafikle karşılaştırın. Uygun müdahale rehberini okuyun ve izinli yanıtı belirleyin. Çözülmüş inceleme kaydı, yeni olayı engellemez; davranışın veya erişim koşullarının gerçekten değişip değişmediğini araştırın.

## Sorun giderme

| Görülen durum | Sonraki adım |
|---|---|
| **Yükleniyor** | Bölümün yüklenmesini bekleyin; yer tutucular sıfır sayım değildir. |
| **Kayıt yok** | Gözlenen olay olmadığı sonucundan önce filtreleri, yüklenen kayıtları, kapsamı ve güncelliği kontrol edin. |
| **Eski kanıt** | Yenileyin ve son bildirilen zamanı olay aralığıyla karşılaştırın. Süren eksikliği desteğe iletin. |
| **Kaynak kullanılamıyor / sorgu başarısız** | Sunuluyorsa yeniden deneme kontrolünü kullanın. Sorun sürerse servis, zaman ve görünen hatayı desteğe iletin. |
| **İzin reddedildi / salt okunur** | Görevinizin gerektirdiği özel okuma veya yönetim iznini isteyin. |
| **Ayar kaydedildi, dağıtım başarısız** | Başarısız dağıtımı inceleyin ve yetkili sorumluyla uygulamanın normal geri dönüş sürecini izleyin. |
| **İstenen ve gözlenen mod farklı** | Geçiş ve dağıtım durumunu kontrol edin; istenen modu aktif olarak raporlamayın. |
| **İzolasyon bekliyor veya başarısız** | Karantinayı doğrulanmamış kabul edin; sonraki müdahaleden önce işlem sonucunu inceleyin. |

## Bulunduğunuz ekranda yardım

Geçerli sekmeye veya desteklenen ayara uygun rehber için maskot menüsünü açın. Statik açıklamayı menü içinde okuyabilirsiniz; maskot açık ve hazırken **Bu ekranı açıkla** seçeneği açıklamayı yardım balonunda da açar. Statik yardım Türkçe ve İngilizcedir, AI kapalıyken de kullanılabilir. Okumak servis kaydı sorgulamaz, AI'a veri göndermez veya korumayı değiştirmez.

AI sohbeti kullanılabilir ve etkinse izinli servis veya kuruluş kapsamınızda isteğe bağlı yardım olarak kullanın. Sayfa bağlamı ayrı ve isteğe bağlı bir ayardır; açılması geçerli sayfanın özetini sohbete ekleyebilir. Açıklama, işlem yetkisi vermez veya korumayı doğrulamaz. Sayfanın kendi izin ve onay akışını kullanın; sonrasında gerçek sonucu değerlendirin.

## Sık sorulan sorular

### Boş Bulgular sekmesi servisin güvenli olduğu anlamına mı gelir?

Eşleşen kayıt gösterilmediği anlamına gelir. Sonucun kapsamını anlamak için kaynak kapsamını, güncelliği, filtreleri ve izinleri kontrol edin.

### Bulguları okuyabiliyorum; neden zaman çizelgesini veya ayarları göremiyorum?

Bu bölümlerin izinleri ayrıdır. Birine erişim, bütün kanıtlara veya yönetim kontrollerine erişim sağlamaz.

### Çalışma zamanı koruma modunu buradan değiştirebilir miyim?

Mod kartı durum bildirir. Desteklenen servis ayarları için rolünüze sunulan düzenleme kontrollerini kullanın; gerekli mod değişikliği için servis sorumlunuza veya Komuta desteğine başvurun.

### Tehdit olarak işaretlemek davranışı hemen durdurur mu?

Bulgu kararı ile uygulanmış koruma kuralı farklı kayıtlardır. Gerçek müdahale eylemini, uygulama durumunu ve ilgili gözlenen sonucu inceleyin.

### Erişim koruması aynı akışın parçası mı?

Genel servis adresine erişimi denetleyerek servis güvenliğini tamamlar. **Erişim ve portlar** altında yapılandırın; ziyaretçi erişimini değerlendirmek için kendi etkinlik ve durum bilgisini kullanın.

## İlgili rehberler

- [Güvenlik Merkezi](security-center-guide.md) — kuruluş genelinde önceliklendirme ve müdahale.
- [Çalışma Zamanı Güvenliği](runtime-security-guide.md) — koruma katmanları, modlar ve doğrulama.
- [Erişim koruması](service-access-protection.md) — genel servis erişim kuralları.
- [Erişim ve portlar](services-ports.md) — servis giriş noktaları ve port ayarları.
- [Dağıtım geçmişi](service-deployment-history.md) — değişikliği dağıtım sonucuyla karşılaştırma.
