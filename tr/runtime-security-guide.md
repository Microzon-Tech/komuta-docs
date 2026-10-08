# Çalışma Zamanı Güvenliği

Çalışma zamanı güvenliği, çalışan bir servisi değerlendirmenize yardımcı olur: kaydedilmiş davranışı, koruma için planlanan kısıtlamaları ve bunların iş yüküne ulaştığına dair kanıtı birlikte incelersiniz. Komuta bu bilgileri bir araya getirir; böylece normal uygulama davranışını gözden kaçırmadan şüpheli etkinliği araştırabilir ve hedefli değişiklik yapabilirsiniz.

Bu rehber koruma modelini açıklar. Ekran üzerinden kullanım için [Servis Güvenliği](service-security-guide.md) rehberini okuyun. Kuruluş genelindeki öncelikler ve müdahaleler için [Güvenlik Merkezi](security-center-guide.md) ile başlayın.

> **Üç bilgiyi ayrı tutun:** ne yapılandırıldı, neyin uygulandığı bildiriliyor ve gözlenen sonuç ne gösteriyor? Bu ayrım hem korumayı değerlendirirken hem değişiklik sonrasında uygulamayı normale döndürürken işe yarar.

> Görseller, 8 Ekim 2026 tarihinde Komuta arayüzündeki test ortamından alınmıştır. Gösterilen durumlar örnektir; kendi servisinizin güncel durumunu kontrol edin.

## Koruma katmanları ve yanıtladıkları sorular

| Katman | Müşterinin sorusu | İncelenecek kanıt |
|---|---|---|
| **Genel adres erişim koruması** | Bu servisin genel adresini kimler açabilir? | Erişim kuralları, koruma durumu ve ilgili erişim etkinliği |
| **Ağ politikaları** | Hangi gelen ve giden bağlantılara izin verilir? | Kural kapsamı, uygulama durumu, akışlar ve düşen bağlantılar |
| **İş yükü sıkılaştırması** | Uygulama hangi ayrıcalıklara ve yazılabilir alanlara ihtiyaç duyar? | Onaylı ayarlar, dağıtım sonucu ve canlı duruş |
| **Çalışma zamanı davranış koruması** | Hangi işlem, dosya veya desteklenen diğer davranış gözlendi; hangi kurallar geçerli? | Çalışma ortamı uygunluğu, koruma eylemi, gözlemler ve bulgu kanıtı |
| **Derleme ve imaj kanıtı** | Kontroller, sunulacak imaj hakkında ne raporladı? | İlgili derleme ve imaj için sunulan tarama sonuçları |

Bu katmanlar birbirini tamamlar. Ziyaretçinin erişim kontrolünden geçmesi, uygulama davranışının güvenli olduğunu göstermez. Derleme taraması, çalışan iş yükünün yapacağı her şeyi incelemez. Ağ izolasyonu, uygulamanın mevcut duruma nasıl geldiğini açıklamadan bağlantıları sınırlandırabilir.

Ziyaretçi kuralları için [Erişim koruması](service-access-protection.md), servis giriş noktaları için [Erişim ve portlar](services-ports.md) rehberini kullanın.

## Sonucu yorumlamadan önce kapsamı belirleyin

Seçilen kuruluş, servis, çalışma ortamı ve dağıtımla başlayın. Kanıtın değerlendirdiğiniz servise ve döneme ait olduğunu doğrulayın. Kuruluş özetleri önceliklendirmeyi, servis kanıtı ise belirli bir incelemenin ayrıntılarını destekler.

Şu koşulları birlikte kontrol edin:

- **Uygunluk:** bu katman seçilen çalışma ortamını ve servisi destekliyor mu?
- **Kullanılabilirlik:** mevcut görünüm ilgili kanıta ulaşabiliyor mu?
- **Güncellik:** kanıt, olay veya değişiklik için yeterince yeni mi?
- **İzin:** rolünüz bölümü okuyabiliyor veya önerilen işlemi yapabiliyor mu?

### Kata ve diğer uygunluk sınırları

**Kata** iş yükünde bazı çalışma zamanı davranış gözlemleri ve koruma işlemleri uygulanamaz. Her çalışma ortamının aynı yetenekleri sunduğunu varsaymak yerine servisin kapsam ve uygunluk bilgisini okuyun.

Ağ kurallarını, iş yükü sıkılaştırmasını, derleme kanıtını ve izolasyonu ayrı değerlendirin. Bir katmanın çalışma ortamı kısıtı, diğerinin desteğini veya etkili olup olmadığını belirlemez. Özellikle izolasyonun kendi uygunluk kontrolü vardır; uygun bir yalıtılmış çalışma ortamında kullanılabilir.

Kullanılamayan kart bulgu değildir; görünmeyen kart da tam koruma kanıtı değildir. Uygunluk bilinmiyorsa desteklenen kapsamı servis sorumlunuzla veya Komuta desteğiyle netleştirin.

![Admin test servisinde host runtime tespiti ve engelleme dahil desteklenen beş yetenek](https://raw.githubusercontent.com/Microzon-Tech/komuta-docs/main/img/security/admin-runtime-capabilities.jpg)

*Admin test ortamındaki komuta-test-app, host runtime tespiti ve engelleme dahil beş yeteneği destekliyor. Destekleniyor etiketi, tek başına korumanın etkinliğine ilişkin bir test sonucu değildir.*

## Çalışma zamanı modu ve koruma eylemi

### Çalışma zamanı modunu okuyun

Servisin **Koruma** sekmesi, uygun iş yüklerinde salt okunur **Çalışma zamanı koruma modu** kartını içerir. İstenen ve gözlenen durumu, uygulamanın beklemede veya başarısız olması dahil gösterir.

| Mod | Bu çalışma zamanı katmanının amaçlanan davranışı |
|---|---|
| **Kapalı** | Bu çalışma zamanı temeli kapalıdır. Diğer katmanlar kendi yapılandırma ve durumlarını korur. |
| **Gölge** | Modun desteklediği sıkılaştırmayı uygularken iş yükünün davranışını öğrenir; etkin davranış engellemesi olarak yorumlanmaz. |
| **Denetim** | Davranışı inceleme ve değerlendirme için gözler ve kaydeder. |
| **Zorlama** | Uygulama tamamlandığında desteklenen ve yapılandırılan kapsamda engelleyici koruma uygular. |

Her servisin aynı modda başladığını veya her modun bütün çalışma ortamlarında kullanılabildiğini varsaymayın. Kart durum bildirir; mod seçicisi değildir. Gerekli mod değişikliği için yetkili servis sorumlunuza veya Komuta desteğine başvurun.

### Denetim ve Engelle eylemlerini ayrı okuyun

Bir çalışma zamanı kuralında ayrıca **Denetim** veya **Engelle** eylemi bulunabilir. Eylem ilgili kurala veya koruma katmanına aittir; dört çalışma zamanı moduyla aynı durum alanı değildir.

| Eylem | Anlamı |
|---|---|
| **Denetim** | Kural, eşleşen davranışı kendisi engellemeden gözlemlemeyi ve kaydetmeyi amaçlar. |
| **Engelle** | Kural, desteklendiğinde ve etkili biçimde uygulandığında eşleşen davranışı reddetmeyi amaçlar. |

Denetim, ağ kısıtlamalarını veya başka kuralın engellemesini kaldırmaz. Engelle, bütün etkinliğin reddedileceği anlamına gelmez. Eşleşen işlem, hedef, kural kapsamı ve çalışma ortamı desteği, kuralın neyi etkileyebileceğini belirler.

Bir programın çalıştırılmasını kısıtlamak, bulunduğu dizine yazmayı kısıtlamaktan da farklıdır. Yola dayalı kuralları yorumlarken hangi işlemi ve kapsamı tanımladığını doğrulayın.

## Yapılandırma, uygulama ve gözlenen sonuç

Bir kontrolün durumunu üç noktada açıklayın:

| Kontrol noktası | Neyi gösterir? | Hâlâ ne doğrulanmalı? |
|---|---|---|
| **Yapılandırıldı** | Hedef için ayar veya kural kaydedildi | Çalışan iş yüküne ulaşıp ulaşmadığı |
| **Uygulandı / gözlendi** | Servis, yapılandırmayı veya modu uygulanmış olarak bildiriyor | İlgili koşullarda beklenen davranışa izin verilip verilmediği veya davranışın reddedilip reddedilmediği |
| **Gözlenen sonuç** | Belirli olay veya bağlantı bildirilen sonucu üretti | Bu kanıtın dışındaki yollar, iş yükleri ve koşullar |

Önizleme amaçlanan içeriği gösterir. Onay, akışın ilerlemesine izin verir. Sıradaki dağıtım, bekleyen işi gösterir. Bunların hiçbiri tek başına etkili engellemeyi kanıtlamaz.

İstenen ve gözlenen mod farklıysa geçiş ve dağıtım durumunu okuyun. Uygulama başarısız olduğunda kaydedilen istenen ayar görünmeye devam edebilir; önceki gözlenen durum hâlâ geçerli olabilir. Mod kullanılamıyor veya doğrulanmamışsa boşluğu varsayımla doldurmayın.

Kanıtı değerlendirdiğiniz yapılandırma ve dağıtımla eşleştirin. Önceki sürümdeki başarılı gözlem sonraki değişikliği doğrulamaz. Benzer biçimde, yeniden bağlama veya geri alma isteğinde de geri dönüşün tamamlandığını söylemeden önce güncel sonuç ve uygulama sağlığı kontrol edilmelidir.

## Temel güvenlik ve uygulama gereksinimleri

Temel güvenlik, beklenen güvenlik yapılandırmasını ifade eder. Canlı duruş, çalışan servisten gelen kullanılabilir bilgiyi bu beklentiyle karşılaştırır ve sapmayı ortaya çıkarabilir.

Desteklenen müşteri ayarları arasında dar kapsamlı **Linux capability** izinleri, **yazılabilir yollar** ve **root olarak çalışma izni** bulunur; her birinin yönetim izni ayrıdır. Amaç, diğer kısıtlamaları koruyarak belirlenmiş uygulama ihtiyacını karşılamaktır.

### Gereken en küçük değişikliği seçin

Başlangıç hatası veya reddedilen işlemde önce ilgili işlemi, yolu veya bağlantıyı belirleyin. İmaj gereksinimleri, dağıtım geçmişi ve mevcut ayarlarla karşılaştırın. Genel bir izin hatası, uygulamanın root veya geniş yazma erişimi gerektirdiğini tek başına göstermez.

Yetkili değişiklikte açık bir gerekçe kaydedin, onayı okuyun ve dağıtım sonucunu kontrol edin. Kök dosya sistemi kısıtlaması geçerliyse yazılabilir yol istisnası, root çalıştırma izninden farklı kapsama sahiptir. Uygulamanın içeriği koruması gerekiyorsa yazılabilir dizinin kalıcılığı ayrıca değerlendirilmelidir.

![Servisin çalışma zamanı gereksinimleri için Linux capability kartı](https://raw.githubusercontent.com/Microzon-Tech/komuta-docs/main/img/security/linux-capabilities.jpg)

*Servis Güvenliği → Koruma içindeki Linux capability kartı, mevcut ayrıcalıkları ve varsayılanları görünür kılar. İmajın ihtiyaçlarını bu listeyle karşılaştırın; test servisinin ayarlarını doğrudan kopyalamayın.*

### Sapmayı ve geri dönüşü doğrulayın

Canlı duruş, beklenen politikanın eksik olduğunu veya uyuşmazlık bulunduğunu bildiriyorsa kanıtın güncel, dağıtımın ise hedeflediğiniz dağıtım olduğunu doğrulayın. Duruş sorgusunun başarısız olması uygulama teşhisi değil, görünürlük eksikliğidir.

Değişiklikleri ilişkilendirmek için [dağıtım geçmişini](service-deployment-history.md) kullanın. Düzeltme veya geri dönüşten sonra hem başarısız olan uygulama davranışını hem ilk ayarı gerekli kılan koruma koşulunu kontrol edin. Tek başına başarılı yeniden başlatma iki soruyu birden yanıtlamaz.

## Gözlemler, bulgular ve politika kararları

Çalışma zamanı gözlemleri, uygun servislerde kaydedilmiş davranışı ve mevcut inceleme geçmişini gösterir. Bulgular işlem gerektiren güvenlik kayıtlarını düzenler; servis zaman çizelgesi bunları bağlama yerleştirir. İlgili bölümlerin izinleri ve veri zaman aralıkları farklı olabilir.

Bekleyen gözlem sayıları, gösterilen kapsamdaki incelemeleri anlatır. Saldırı sayısını ölçmez veya güvenlik duruşu testinin sonucunu kanıtlamaz. Sınıflandırmadan önce gerçek işlemi ve kanıtı okuyun.

**İzin ver** ve **Tehdit olarak işaretle**, gerekçe gerektiren bulgu kararlarıdır. Bulgunun nasıl değerlendirildiğini kaydeder; tek başına izin veya engelleme kuralı uygulamaz. **Göz ardı et** koruma politikası yerine bulgunun inceleme durumuyla ilgilidir.

İstisna isteği sunuluyorsa hedefi, gerekçeyi ve süreyi inceleyin. Onay bekleyen istek uygulanmış istisna değildir. Bir öneri veya müdahale ayrı uygulama adımı sunuyorsa devam etmeden önce önizlemeyi ve mevcut politikayı kontrol edin. [Güvenlik Merkezi](security-center-guide.md) rehberi bu akışları ve izinlerini açıklar.

## Kontrollü doğrulama

Koruma testi, kendi yetki ve etkisine sahip bilinçli bir işlemdir. Hedef servis için desteklenen, onaylı senaryo kullanın; incelemeyi üretim uygulamasında plansız teste dönüştürmeyin.

Başlamadan önce hedef, uygun çalışma ortamı, beklenen sinyal, izin verilen etki ve geri dönüş sorumlusu üzerinde anlaşın. Senaryonun **tespit**, **önleme** veya **geri dönüş** değerlendirmesi için mi olduğunu belirleyin. Bu sonuçlar farklı kanıtlar gerektirir.

1. Mevcut yapılandırmayı, gözlenen modu, uygulama sağlığını ve kanıt zamanını kaydedin.
2. Yalnız onaylı senaryoyu ve kapsamı, gereken izinlerle kullanın.
3. Sonuç kaydını beklenen kaynak, servis, işlem ve zaman aralığıyla eşleştirin.
4. Tespit için beklenen gözlem veya bulguyu doğrulayın; önleme için bildirilen reddi ve uygulama sonucunu da kontrol edin.
5. Kararlaştırılan geri dönüşü tamamlayın; normal davranışı ve koruma durumunu yeniden kontrol edin.

Sırf bulgu oluşturmak için hassas veya tuzak yollara erişmeyin. Eksik sinyal; kaynak, izin, filtre veya uygunluk sorununa işaret edebilir. Testi tekrarlamadan önce belirsizliği araştırın.

Tespit edilen senaryo, o senaryoda gözlenen davranışı gösterir. Bütün saldırıların önlendiğini veya her servisin kapsandığını kanıtlamaz.

### Örnek: test sonucunu Komuta kanıtıyla eşleştirin

8 Ekim 2026'da admin test ortamındaki **komuta-test-app** üzerinde **İkili bırak ve çalıştır** senaryosunu çalıştırdık. Senaryo, geçici dizine bir program kopyalayıp çalıştırmayı dener. Aşağıda test uygulamasının gerçek ret sonucu yer alır.

![komuta-test-app içinde İkili bırak ve çalıştır senaryosunun Engellendi sonucu](https://raw.githubusercontent.com/Microzon-Tech/komuta-docs/main/img/security/runtime-test-result.jpg)

*Test uygulaması `/tmp/komuta-dropped` için `permission denied` döndürdü. Üstteki “M2M ingest yapılandırıldı” bilgisi, kimlik doğrulamanın başarılı olduğunu göstermez; bu yerel test doğrudan sentetik veri gönderimine dayanmaz.*

Ardından **Servislerim → komuta-test-app → Güvenlik → Bulgular** yolundan aynı işlemin kaydını açtık. Bulgu, politika engelini ve test saatiyle eşleşen **Son görülme** bilgisini gösteriyor. Servis Audit modundayken de mevcut, açık bir Engelle kuralı ilgili işlemi reddedebilir.

![Komuta arayüzünde testle eşleşen politika engeli bulgusu](https://raw.githubusercontent.com/Microzon-Tech/komuta-docs/main/img/security/runtime-block-evidence.jpg)

*Test sonucu ile Komuta'daki kaynak, hedef işlem ve zamanın eşleşmesi bu örneğin engelleme kanıtıdır. İlk görülme ve tekrar sayısı önceki koşuları da içerir; bunları bu koşuda üretilen olay sayısı olarak okumayın.*

| Testte görülen sonuç | Nasıl yorumlanır? |
|---|---|
| **İzin verildi** | İşlem çalışabildi; Audit kapsamındaki bir davranış yine de bulgu üretebilir. |
| **Engellendi / permission denied** | Ret oluştu; hangi katmanın reddettiğini Komuta'daki eşleşen kanıtla doğrulayın. |
| **Bağlantı zaman aşımı** | Ulaşılabilirlik sorunu da olabilir; tek başına ağ politikası engeli saymayın. |
| **Çalıştırılmadı / invalid_client** | Sentetik veri gönderimi kimlik doğrulamada durdu; tespit veya engelleme başarıyla sınanmış değildir. |

Bu örnek için koruma ayarlarını değiştirmedik. Gerçek çalışma zamanı testleri ile doğrudan sentetik veri gönderimini ayrı değerlendirin; biri diğerinin toplama ve engelleme zincirini doğrulamaz.

## Sık karşılaşılan senaryolar

### “Mod Denetim görünüyor ama istek engellendi”

Reddin hangi katmandan geldiğini kontrol edin. Genel erişim kuralları, ağ politikaları, izolasyon veya başka uygun kural, çalışma zamanı modundan bağımsız olarak isteği kısıtlayabilir. İsteğin yolunu ve zamanını ilgili kanıtla karşılaştırın.

### “Yazılabilir yolu kaydettik ama hata sürüyor”

Kayıt sonucunu ve dağıtım durumunu okuyun; ardından çalışan uygulamanın gerçek yolunu kaydedilmiş dizinle karşılaştırın. Hatanın eksik bağımlılık veya başka başlangıç sorunu yerine yazma izniyle ilgili olduğunu doğrulayın. Yetkili en küçük düzeltmenin uygulanması tamamlandıktan sonra sonucu kontrol edin.

### “Bulguyu tehdit olarak işaretledik ama yeniden oldu”

İnceleme kararı değerlendirmeyi kaydeder. Ayrı müdahale veya politika işlemini ve uygulama durumunu inceleyin; ardından yeni kanıtla karşılaştırın. Tekrar, incelemeyi sürdürme nedenidir; kararın kaydedilemediğinin kanıtı değildir.

### “Servisi yeniden bağladık ama hâlâ sağlıklı değil”

İstenen serbest bırakmanın tamamlandığını kontrol edin; uygulama ve bağımlılık durumunu inceleyin. Ağ kısıtlamasının kaldırılması, ilk güvenlik ihlalini, uygulama hatasını veya ilgisiz erişim kurallarını düzeltmez.

## Sorun giderme

| Durum | Sonraki kontrol |
|---|---|
| **Mod yükleniyor veya kullanılamıyor** | Doğrulanmış sonucu bekleyin veya sunuluyorsa yeniden deneyin; belirsiz durumu etkin koruma saymayın. |
| **Gözlem veya bulgu yok** | Filtreleri, izinleri, uygunluğu, yüklenen kayıtları ve kanıt güncelliğini inceleyin. |
| **İstenen ve gözlenen durum farklı** | Uygulama durumunu ve son dağıtım sonucunu okuyun. |
| **Yapılandırma sapması** | Güncel duruşu onaylı ayarlar ve hedeflenen dağıtımla karşılaştırın. |
| **İzin reddedildi** | Göreviniz için gereken özel okuma veya yönetim iznini isteyin. |
| **Kata'da daha az çalışma zamanı kontrolü var** | Servisin uygunluk bilgisini kullanın; kalan her katmanı ayrı değerlendirin. |
| **Uygulama başarısız veya geri dönüş eksik** | Görünen hatayı koruyun ve başka değişiklikten önce yetkili sorumluya başvurun. |

Desteğe başvururken servisi, olay zamanını, görünen durumu, ilgili dağıtımı ve beklediğiniz sonucu belirtin. Yalnız bu inceleme için gerekli bilgiyi paylaşın.

## Bağlama uygun yardım ve isteğe bağlı AI

Maskot menüsü, geçerli güvenlik sayfası veya desteklenen servis ayarı için statik Türkçe ve İngilizce yardım içerir. AI veya dekoratif maskot kapalıyken de okunabilir. Maskot açık ve hazırken **Bu ekranı açıkla** rehberi yardım balonunda da gösterebilir.

Statik rehber servis verisi sorgulamaz, AI'a göndermez veya işlem yürütmez. İsteğe bağlı AI sohbeti ayrıdır. Sayfa bağlamı paylaşımı isteğe bağlıdır; açıldığında sayfa özeti ekleyebilir. Sohbeti yetkili kapsamınızda tutun. Öneri; kanıtın, sayfa izinlerinin veya işlemin onay ve sonuç kontrollerinin yerine geçmez.

## Sık sorulan sorular

### Sağlıklı özet korumayı doğrulamak için yeterli mi?

Önceliklendirmeye yardımcı olur; ancak sonuç kaynak kapsamına ve güncelliğe bağlıdır. Belirli koruma iddiası için hedef kuralı, uygulamayı ve ilgili sonuç kanıtını inceleyin.

### Denetim, hiç güvenlik olmamasıyla aynı mı?

Hayır. Denetim, ilgili çalışma zamanı modunun veya kuralın gözlem niyetini anlatır. Diğer sıkılaştırma ve erişim kısıtlamalarının kendi durumu vardır.

### Zorlama, bütün istenmeyen davranışların durdurulmasını garanti eder mi?

Uygun kapsam için amaçlanan modu tanımlar. Etkili koruma, uygulanmış kurallara ve desteklenen davranışa bağlıdır; ilgili kanıt üzerinden değerlendirin.

### Derleme taraması çalışma zamanı incelemesinin yerine geçer mi?

Hayır. Derleme kanıtı kontrol edilen imajı veya derlemeyi anlatır. Çalışma zamanı incelemesi, dağıtılmış uygulama çalışırken olanları ele alır.

### Bulgu kararı veya AI yanıtı korumayı otomatik değiştirir mi?

Kaydedilmiş bulgu kararı veya açıklama, koruma değişikliğinin kanıtı değildir. Sunuluyorsa açık ve yetkili uygulama veya müdahale akışını kullanın; sonucunu doğrulayın.

## İlgili rehberler

- [Servis Güvenliği](service-security-guide.md) — inceleme ve desteklenen servis ayarlarını yönetme.
- [Güvenlik Merkezi](security-center-guide.md) — kuruluş genelinde bulgular, kararlar ve müdahale.
- [Erişim koruması](service-access-protection.md) — genel servis erişimini koruma.
- [Erişim ve portlar](services-ports.md) — giriş noktalarını ve port ayarlarını inceleme.
- [Dağıtım geçmişi](service-deployment-history.md) — istenen değişikliği dağıtım sonucuyla karşılaştırma.
