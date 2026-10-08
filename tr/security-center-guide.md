# Güvenlik Merkezi

Komuta Security Center; güvenlik bulgularını, servis korumasını, erişim etkinliğini ve inceleme kanıtlarını bir araya getirir. Organizasyonunuzda dikkat gerektiren konuları belirlemek, etkilenen servisi incelemek ve yetkili bir müdahaleyi sonucuna kadar takip etmek için kullanın.

Risk özetinden ilgili bulguya geçebilir, bulguyu servis trafiği ve koruma durumuyla karşılaştırabilir, politika önerilerini değerlendirebilir ve bir kararın arkasındaki kayıtları takip edebilirsiniz. Aynı çalışma alanı bu incelemeleri denetim geçmişi, derleme ve imaj kanıtları, kontrollü güvenlik senaryoları ve bildirim akışlarıyla ilişkilendirir.

> Görseller, 8 Ekim 2026 tarihinde Komuta arayüzündeki test ortamından alınmıştır. Gösterilen durumlar örnektir; kendi servisinizin güncel durumunu kontrol edin.

## Çalışma Zamanı Güvenliği: çalışan servislerinizi anlayın ve koruyun

Çalışma Zamanı Güvenliği, uygulamanız çalışırken gerçekleşen davranışları incelemenizi ve desteklenen koruma kontrollerini yönetmenizi sağlar. Komuta; işlem çalıştırma, dosya erişimi ve ağ bağlantılarıyla ilgili mevcut kanıtları bulgular, kurallar ve servis duruşuyla ilişkilendirir. Böylece şüpheli davranıştan ilgili servise, incelemeden hedefli müdahaleye ilerleyebilirsiniz.

| Yetenek | Size ne sağlar? |
|---|---|
| **Davranış gözlemleri ve bulgular** | Kaydedilmiş program çalıştırma, dosya yazma ve ağ girişimlerini inceleyin; önem derecesini, tekrarları ve olay zaman çizelgesini birlikte değerlendirin. |
| **Trafik görünürlüğü** | Bağlantı akışlarını, ağ olaylarını ve reddedilen bağlantıları inceleyerek servisin beklenen iletişimiyle karşılaştırın. |
| **Davranış kuralları ve öneriler** | Desteklenen işlem ve dosya davranışları için Denetim veya Engelle kuralları tanımlayın; önerileri dayandıkları kanıtla değerlendirin ve uygulama durumunu takip edin. |
| **Temel koruma ve canlı duruş** | Beklenen güvenlik ayarlarını çalışan servisin durumuyla karşılaştırın; izin verilen ayrıcalıkları ve yazılabilir alanları uygulamanın ihtiyacına göre dar kapsamda yönetin. |
| **Servis izolasyonu ve geri dönüş** | Uygun bir servis için yetkili müdahale akışından bağlantıları sınırlandırın; önizlemede etkiyi değerlendirin ve yeniden bağlama sonucunu takip edin. |
| **Kontrollü doğrulama** | Desteklenen güvenlik senaryolarıyla beklenen bulgunun oluşup oluşmadığını ve tespit zamanını inceleyin. |

**Güvenlik Merkezi**, bu yeteneklerin organizasyon genelindeki risk ve bulgu görünümüdür. Belirli bir servisin trafiği, koruma ayarları ve ayrıntılı kanıtları için **Servislerim → ilgili servis → Güvenlik** alanına geçin. Kullanılabilir kontroller servis ortamına ve yetkilerinize bağlıdır; etkili korumayı uygulama durumu ve ilgili sonuç kanıtıyla değerlendirin.

Ayrıntılı kullanım rehberleri **Servislerim → Servis Güvenliği** altında yer alır:

- [Çalışma Zamanı Güvenliği](runtime-security-guide.md): koruma katmanları, çalışma zamanı modları, kural eylemleri, uygunluk ve doğrulama adımları.
- [Servis Güvenliği](service-security-guide.md): Genel Bakış, Trafik, Koruma ve Bulgular sekmelerinde adım adım inceleme ve yetkili müdahale.

> **Doğru bağlamla başlayın:** organizasyonu, servisi, zaman aralığını ve kanıtın güncelliğini doğrulayın. Kaydedilmiş koruma ayarı amacı gösterir; uygulama durumu ve gözlenen sonuçlar ne olduğunu açıklar.

## Nereden başlamalısınız?

| Amacınız | Başlangıç noktası |
|---|---|
| Önce neyi inceleyeceğinize karar vermek | **Güvenlik → Genel Bakış**, ardından **Bulgular** |
| Tek bir uygulamanın davranışını incelemek | [Servis Güvenliği](service-security-guide.md) |
| İnternete açık uygulamaya kimlerin erişebileceğini belirlemek | **Güvenlik → Erişim koruması** ve [Erişim ve portlar](service-access-protection.md) |
| Bir koruma değişikliğini anlamak | **Politikalar**, **Engellemeler** ve [Çalışma Zamanı Güvenliği](runtime-security-guide.md) |
| İşlemleri veya oturum açmaları incelemek | **Denetim Kaydı** veya **Oturum Aktivitesi** |
| Bir imajı veya saklanan denetim kanıtını değerlendirmek | **Tedarik zinciri** veya **Denetim kaydı koruması** |

## İlk incelemeniz

1. **Organizasyonunuzu doğrulayın.** Security Center, geçerli organizasyonda hesabınızın erişebildiği kayıtlarla çalışır. Daha dar bir inceleme için ilgili servisi açın.
2. **Genel Bakış'ı okuyun.** Riski, açık bulguları, etkilenen servisleri, kapsamı ve veri güncelliğini birlikte değerlendirin. Sakin bir paneli sağlıklı sonuç olarak yorumlamadan önce eksik kanıtları araştırın.
3. **Bulgular'ı açın.** Servisi ve zaman aralığını seçin; gerektiğinde kaynak, önem derecesi, tür veya durumla daraltın.
4. **Ayrıntıyı okuyun.** Etkilenen işlemi, kaynağı, ilk ve son görülme zamanını, tekrarları ve mevcut kanıtları belirleyin. Trafik ve korumayı karşılaştırmak için servis bağlamına geçin.
5. **Sonraki adımı seçin.** İnceleme kararını kaydedin, bir playbook'a başvurun veya gerekli yetkiniz varsa müdahale hazırlayın.
6. **Sonucu doğrulayın.** Kaydedilen işlemi, uygulama durumunu ve ilgili davranışı ayrı ayrı kontrol edin. İncelemeyi ekip arkadaşınıza devrederken çözümlenmemiş kanıt eksiklerini görünür bırakın.

Bir sayfayı okumak veya yardımını açmak servis korumasını değiştirmez. Politika uygulamak, istisna oluşturmak, senaryo çalıştırmak ve engellemeyi kaldırmak ayrı işlemlerdir.

## Kapsam ve erişim

**Security Center**, hesabınızın erişebildiği servis ve kayıtların organizasyon görünümünü sunar. **Servis → Güvenlik**, açık olan servise odaklanır. **Hesap güvenliği** ise kendi hesabınızı, oturumlarınızı ve kişisel güvenlik geçmişinizi kapsar.

Okuma yetkisi; müdahale etme, politika düzenleme, istisna yönetme, güvenlik senaryosu çalıştırma, kayıt dışa aktarma veya arşiv güvencesini inceleme yetkisini kendiliğinden vermez. Bazı özellikler servisin çalışma ortamında destek de gerektirir. Bir işlemin görünmemesi yetki veya uygulanabilirlikten kaynaklanabilir; adresi değiştirmek erişim sağlamaz.

Organizasyonu, servisi veya filtreleri değiştirdikten sonra gösterilen bağlamı tekrar kontrol edin ve güncel görünümün yüklenmesini bekleyin. Bir bağlantı, sayaç veya eski tarayıcı sekmesi tüm organizasyon kayıtlarına erişebildiğinizin kanıtı değildir.

## Sayfaları tanıyın

| Grup | Sayfa | İnceleyebileceğiniz bilgiler |
|---|---|---|
| İzle | **Genel Bakış** | Riskler, etkilenen servisler, son bulgular ve kanıt kapsamı |
| İzle | **Bulgular** | Aranabilir güvenlik kayıtları, ayrıntılar, inceleme kararları ve yetkili müdahale |
| Kayıt | **Denetim Kaydı** | Kaydedilmiş işlemler, işlemi yapan kişi, zaman ve sonuç |
| Kayıt | **Oturum Aktivitesi** | Organizasyonun oturum açma etkinliği ve beklenmeyen sonuçlar |
| Kayıt | **Denetim kaydı koruması** | Organizasyonunuz için saklama süresi, hukuki koruma ve arşiv doğrulaması |
| Koru | **Erişim koruması** | İnternetten erişim durumu ve servis erişim ayarlarına bağlantılar |
| Koru | **Politikalar** | Uygulanabilir koruma kuralları ve politika önerileri |
| Koru | **Engellemeler** | Engelleme kayıtları, uygulama durumu ve yetkili geri alma |
| Koru | **Honey path'ler** | Desteklenen servislerde tuzak yol yapılandırması ve tespit kayıtları |
| Koru | **Olay Müdahale Playbook'ları** | İnceleme ve müdahale yönergeleri |
| Doğrula | **Sentetik saldırılar** | Kullanılabilir güvenlik senaryoları, çalışma geçmişi ve tespit sonuçları |
| Doğrula | **Tedarik zinciri** | Derleme ve imaj taramaları, çıktı kanıtları ve istisnalar |

Görünen sayfalar ve kontroller yetkilerinize ve özelliklerin kullanılabilirliğine bağlıdır. Bir yetenek kullanılamıyorsa sayfada gösterilen durumu ve açıklamayı esas alın.

## Genel Bakış: bağlamla önceliklendirin

Genel Bakış, **hangi servislerin neden dikkat gerektirdiğini** anlamanıza yardımcı olur. İnceleme seçmek için risk özetini, önem derecesi dağılımını, son bulguları ve ağ tehdidi bilgilerini kullanın. Kapsamı ve veri güncelliğini bu özetlerle birlikte okuyun.

Ana risk kartı, özetteki en yüksek riskli servisin anlık değerlendirmesini yansıtır; servis adını ve hesaplama zamanını okuyun. Risk puanı önceliklendirme aracıdır; uyumluluk sertifikası değildir. Güvenlik duruşu göstergesi değerlendirilen ayarları, gözlem sayısı kaydedilmiş davranışları anlatır. Bunların hiçbiri saldırı sayacı veya tüm koruma katmanlarının etkili olduğunun kanıtı değildir.

Bir özetten filtrelenmiş listeye geçtiğinizde hangi filtrelerin taşındığını kontrol edin. Özetleri ve ayrıntıları aynı kapsam ve zaman aralığında karşılaştırın; farklı kaynaklar farklı zamanlarda güncellenebilir.

![Güvenlik Merkezi genel bakışında öncelikli işler, risk ve kritik bulgular](https://raw.githubusercontent.com/Microzon-Tech/komuta-docs/main/img/security/security-center-overview.jpg)

*Genel bakış, inceleme bekleyen işleri risk ve kanıt güncelliğiyle birlikte gösterir. Bu örnekte düşük risk puanının yanında eski ve zayıflamış kanıt durumu da görünür.*

## Bulgular: sinyali incelemeye dönüştürün

Bulgular, bir güvenlik sinyalini servisi, önem derecesi, durumu ve kanıtlarıyla ilişkilendirir. Karar vermeden önce yoğun bir listeyi mevcut filtrelerle daraltın. Arama ve filtreler, görünen liste boş olsa bile kayıtları dışarıda bırakabilir. Zaman aralığı bulguları son görülme zamanına göre süzer; varsayılan olarak son yedi günle başlar.

### Kaydı anlayın

| Alan veya kavram | Nasıl kullanılır? |
|---|---|
| **Kaynak** | Kaydın hangi tür kanıttan üretildiğini anlayın: örneğin çalışma zamanı davranışı, ağ etkinliği, güvenlik duruşu veya derleme güvenliği. |
| **Önem derecesi ve güven düzeyi** | Öncelik belirleyin. Kaydı doğrulanmış olay saymadan önce etkilenen servisi ve kanıtı kontrol edin. |
| **İlk ve son görülme** | Davranışın ne zaman başladığını ve tekrarlanıp tekrarlanmadığını belirleyin. |
| **Tekrar** | Aynı mantıksal bulgunun tekrarlarını görün. Toplam sayı, her bir olayın kimliğini vermez. |
| **Etkilenen işlem** | İlgili programı, yolu, bağlantıyı veya çıktıyı uygulamanın beklenen davranışıyla karşılaştırın. |
| **Kanıt ve zaman çizelgesi** | Kaynağı, hedefi, zamanı ve sonucu ilişkilendirin. Kaydın sunduğu durumlarda ilişkili olayları veya süreç bağlamını kullanın. |

**Gözlem**, davranış veya durum kaydeder. **Bulgu**, buna bir inceleme yaşam döngüsü ekler. **Gözlem özeti**, hesaplandığı anda inceleme bekleyen gözlemleri toplar; özeti kapatmak alttaki kayıtları incelemez veya korumayı değiştirmez.

Kullanılamayan ayrıntı, eksik kaynak verisi veya eski zaman damgası değerlendirmenizin parçası olarak kalmalıdır. İki kayıt ilişkili görünüyorsa açıklamalarının yanında servislerini ve zamanlarını da eşleştirin.

### Kararlar ve müdahaleler ayrıdır

| Karar | Kaydettiği anlam |
|---|---|
| **Onayla** | Bulgu görülmüş ve ilk değerlendirmesi yapılmıştır; Açık kuyruğundan çıkar. |
| **İzin ver** | Gerekli gerekçeyle davranışı meşru kabul ediyorsunuz. |
| **Tehdit olarak işaretle** | Gerekli gerekçeyle davranışı tehdit olarak sınıflandırıyorsunuz. |
| **Yoksay** | Bulguyu geçersiz veya inceleme kapsamı dışında değerlendiriyorsunuz. |
| **Çöz** | İncelemeyi uygun açıklamayla sonuçlandırıyorsunuz. |

Bu kararlar ekibinizin inceleme ilerlemesini izlemesini sağlar. Tehdit olarak işaretlemek tek başına engelleme kuralı uygulamaz; İzin ver de kendiliğinden politika istisnası uygulamaz.

Kayıt bir müdahale sunuyorsa onay akışında hedefi, kapsamı, önerilen değişikliği ve gerekçeyi inceleyin. Gerçek engelleme, istisna ve servis izolasyonu kendi yetkilerini ve desteğini gerektirir. İzolasyon uygulamanın bağlantılarını kesebilir. Müdahale veya geri alma sonrasında olayı kapatmadan önce uygulama durumunu ve ilgili servis davranışını doğrulayın.

## Erişim koruması: uygulamaya kim ulaşabilir?

**Güvenlik → Erişim koruması**, erişebildiğiniz servislerin internete açıklık durumunu tek bir organizasyon görünümünde toplar. Durumlarını, koruma bitiş tarihlerini ve son 24 saatte izin verilen veya reddedilen istekleri karşılaştırın. Açık, korumalı, değişiklik sürecinde veya dikkat gerektiren servisleri belirleyin; ardından servisin **Yapılandırma → Erişim ve portlar** çalışma alanını açın.

Bu çalışma alanında **Genel Bakış, Kurallar, Kişiler, Makineler, Etkinlik, Ağ ve Ayarlar** ayrılır. Desteğe ve yetkilerinize göre oturum açma ve IP koşullarını, yola özel erişimi, paylaşımları, makine erişimini ve koruma bitiş tarihlerini yapılandırabilirsiniz. **Etkinlik**, izin verilen veya reddedilen erişimi incelemeye yardımcı olur ve ilgili yönetim yetkisini gerektirir.

Erişim koruması uygulamaya girişle ilgilidir. Çalışma zamanı bulguları çalışan servisin çevresindeki davranışlarla, ağ politikaları izin verilen bağlantılarla ilgilidir. Bir erişim sorununu incelerken bu katmanları karşılaştırın; durumları farklı sorulara yanıt verir.

Kurulum için [Erişim Koruması rehberini](service-access-protection.md), olayları yorumlamak için [Erişim Kaydı rehberini](access-protection-activity.md) izleyin. Yetkili değişiklik sonrasında uygulama ilerlemesini ve hedeflenen ziyaretçinin sonucunu kontrol edin; kaydedilmiş kural veya önizleme tek başına erişim testi değildir.

## Politikalar, öneriler ve istisnalar

### Koruma politikası hazırlayın

**Politikalar** sayfasında yenisini oluşturmadan önce mevcut kuralları ve hedef servislerini inceleyin. Politika sihirbazı, uygulanabilir servislerde program çalıştırma, dosya erişimi ve ilişkili davranışlar için kural hazırlamayı destekler. Kaydetmeden önce seçilen servisi, kural ayrıntılarını, Audit veya Block işlemini ve önizlemeyi kontrol edin.

**Audit**, eşleşen davranışı gözlemleme amacını belirtir. **Block**, desteklenen koruma katmanında eşleşen davranışı önleme amacını belirtir. Kurallar eşleşme koşulları ve çalışma ortamı desteğiyle sınırlıdır; Block seçmek tüm davranışları engellemez. Uygulanabilirlik ve doğrulama için [Çalışma Zamanı Güvenliği](runtime-security-guide.md) rehberine bakın.

Sihirbazda politika oluşturmak etkin bir politika kaydeder ve otomatik uygulamaya yol açabilir. Oluşturmayı etkisiz bir taslak olarak değil, koruma değişikliği olarak değerlendirin. Ayrı yayımlama kontrolü sunuluyorsa onun sonucunu da izleyin. Kaydedilmiş kayıt tek başına etkili koruma kanıtı değildir; gösterilen uygulama durumunu ve hataları kontrol edin. Kısıtlayıcı değişikliği değerlendirirken servisin normal başlangıcını, sağlık kontrollerini ve gerekli bağlantılarını kontrol edin.

### Öneriyi değerlendirin

Öneriler, politika değişikliklerini dayandıkları kanıtlarla ilişkilendirir. Kabul etmeden önce hedefi, güven düzeyini, gözlenen trafik veya davranışı ve önerilen kuralı inceleyin. Kullanılabilir yaşam döngüsü işlemlerini ilgili yetkiyle takip edin.

Kabul, başarılı uygulama ve etkili koruma ayrı sonuçlardır. Uygulama bekliyor veya başarısızsa bu durumu incelemede koruyun. Geri alma sunulduğunda önizlemesini inceleyin ve sonrasında oluşan durumu doğrulayın.

### İstisnayı dar tutun

Uygun bir bulgudaki istisna müdahalesi, gerekçe ve süreyle politikaya izin listesi istisnası talep edebilir. Onaylamadan önce hedefi ve önizlemeyi inceleyin. Bu işlem onay gerektirebilen bir talep açar; tek başına servisi yeniden dağıtmaz veya korumanın değiştiğini kanıtlamaz. Akış bunun yerine bulguyu bastırmayı sunuyorsa bu bir yoksayma kararı kaydeder ve politika istisnası oluşturmaz.

Meşru davranışın gerektirdiği en küçük kapsamı kullanın. Uygulama veya süre dolumu sonrasında hem hedeflenen erişimi hem kalan korumayı doğrulayın. Risk istisnası alttaki zayıflığın giderilmesi değildir.

## Engellemeler ve toparlanma

**Engellemeler**, etkin ve geri alınmış kayıtları, etkilenen servisleri, destekleyici kanıtları ve uygulama durumunu incelemenizi sağlar. Engelleme isteği kabul edilmiş olsa bile uygulanıyor, başarısız veya doğrulanmamış durumdaki kayıt ayrıca incelenmelidir.

Yetkili bir geri almada onaylamadan önce önizlemeyi okuyun. Geri alma yeni bir dağıtım gerektirebilir; kabul edilen istek tamamlanmış toparlanma değildir. Hedeflenen kısıtlamanın değiştiğini ve normal uygulama davranışının döndüğünü kontrol edin. Tek bir engellenen işlemi düzeltmek için ilgisiz korumaları kaldırmayın.

## Honey path'ler

Honey path'ler, beklenmedik erişimde dikkat çekmesi amaçlanan tuzak yollardır. Sayfa, desteklenen servislerde kaydedilmiş yapılandırmayı ve tespit geçmişini gösterir. Bu yapılandırmanın kullanımda olup olmadığını değerlendirirken son dağıtımı ayrıca kontrol edin.

Gerekli yönetim yetkisiyle uygun servisi seçerek yol ekleyebilir, yapılandırmasını düzenleyebilir, etkinlik durumunu değiştirebilir veya kaldırabilirsiniz. İşlem sonrasında kaydedilmiş girdiyi ve tespit geçmişini inceleyin. Yolun uygulamaya uygun olduğunu ve meşru etkinlikle çakışmayacağını doğrulayın. Etkin ayar, tespit edilmiş olay değildir. Kaydedilen erişimi servisi, zamanı ve kanıtlarıyla inceleyin; olay olarak sınıflandırmadan önce bağlamını değerlendirin.

Çalışıp çalışmadığını görmek için tuzak yolu açmayın veya değiştirmeyin. Açık hedefi ve beklenen sonucu olan, ayrıca onaylanmış bir senaryo kullanın.

## Denetim Kaydı, oturum açmalar ve kişisel güvenlik

**Denetim Kaydı**, kaydedilen etkinliği kaynak, işlemi yapan kişi, zaman ve sonuç üzerinden yeniden değerlendirmenizi sağlar. Gruplu görünüm tekrarlayan kayıtları özetler; ham görünüm tekil olayları incelemeye yardımcı olur. Sorunuza uygun görünümü seçin ve ne kadar verinin yüklendiğini kontrol edin.

**Oturum Aktivitesi**, organizasyondaki oturum açmalarla ilgilidir. Beklenmeyen bir oturum açmada etkilenen hesabı, zamanı, sonucu ve mevcut bağlamı inceleyin. Filtreler, sayaçlar ve CSV **yüklenmiş olayları** kapsar; kapsamı anlamak için yüklenen/toplam göstergesini ve sunulan eski olay yükleme işlemini kullanın.

Kişisel hesap güvenlik kaydınız, parola ayarlarınız ve etkin oturumlarınız **Hesap güvenliği** kapsamındadır. Servis ziyaretçilerinin erişimi **Erişim ve portlar → Etkinlik** altındadır. Komuta'da oturum açma ile korunan uygulamaya yapılan istek farklı olaylardır.

## Playbook'lar: müdahaleyi tekrarlanabilir hale getirin

**Olay Müdahale Playbook'ları**, sıralı inceleme ve müdahale yönergeleri sunar. Hazır playbook'ları okuyun; uygun yetkiyle ekibinizin süreci için bir kopyayı veya özel playbook'u yönetin.

Her adım için hedefi, önkoşulu, beklenen sonucu ve sorumlu kişiyi belirleyin. Playbook'u açmak yönergelerini çalıştırmaz. Desteklenen işlemleri yetkili sayfa kontrollerinden yürütün ve sonucu kaydedin. Devir sırasında aynı sırayı kullanarak ekip arkadaşınızın hangi konuların açık kaldığını görmesini sağlayın.

## Kontrollü güvenlik senaryoları

**Sentetik saldırılar**, yetkili kullanıcıların desteklenen senaryoları incelemesini, izin verilen çalışmaları başlatmasını ve sonuçları değerlendirmesini sağlar. Seçilen senaryonun tespit süresi içinde beklenen bulguyu üretip üretmediğini değerlendirmek için kullanın.

Çalıştırmadan önce hedef servisi, senaryonun kullanılabilirliğini, çalışma ortamı desteğini, beklenen sinyali, uygulamaya olası etkisini ve toparlanma planını doğrulayın. Listelenen senaryo veya geçmiş başarılı çalışma, güncel hazır oluşu göstermez.

Sonrasında çalışma durumunu, eşleşen bulguyu ve tespit süresini aynı hedef için karşılaştırın. Beklenen kanıt eksikse yalnız işlemin tamamlanması yeterli değildir. Sonuç test edilen senaryo ve zaman aralığı için geçerlidir; her saldırının önlendiğini veya her koruma katmanının etkili engelleme yaptığını göstermez.

![Sentetik saldırılar ekranında senaryo uygunluğu ve tatbikat geçmişi](https://raw.githubusercontent.com/Microzon-Tech/komuta-docs/main/img/security/scenario-applicability.jpg)

*Sentetik saldırılar ekranı senaryo kataloğunu ve tatbikat geçmişini birleştirir. Görüntüdeki çalışma zamanı için otomatik senaryolar uygulanamaz; bu durum başarılı bir saldırı testi sonucu değildir.*

## Tedarik zinciri: kanıtı imajla eşleştirin

**Tedarik zinciri**, derleme ve imaj güvenliği kanıtlarını incelemeye dahil eder. Tarama durumunu, zamanını, önem derecesini, etkilenen çıktıyı ve mevcut ayrıntıları değerlendirin. Sunulduğunda yazılım bileşen listesini (SBOM), zafiyet ve imza bilgisini tam olarak ilgili derleme veya imaj için inceleyin.

Kanıtı servisin gerçekten kullandığı çıktıyla eşleştirin. Yeni derleme, değişen imaj veya eski tarama yeniden değerlendirme gerektirebilir. Eksik veya başarısız kanıt temiz sonuç değildir; derleme taraması çalışan servisin tüm davranışlarını da incelemez.

Gerekli yetkiyle seçilen servis ve belirli bir imaj sürümü için süreli risk kabul edebilir veya mevcut istisnayı iptal edebilirsiniz. Gerekçesini, imajı, bitiş zamanını ve sonraki dağıtım denemesine etkisini inceleyin. Ayrı dağıtıma kabul durumunu kontrol edin: değerlendirilmemiş, doğrulanmış, engellenmiş, istisna kapsamında veya yalnız denetim olabilir. Risk kabulü zafiyeti ortadan kaldırmaz. İmza göstergesi her çıktının imzalandığını veya her imzanın bağımsız doğrulandığını göstermez.

## Denetim kaydı koruması

Bu salt okunur sayfa, organizasyonunuzun denetim kayıtlarının korunmasını değerlendirmenize yardımcı olur. **Arşiv korumasını**, **saklama süresini**, **hukuki korumayı**, gösterilen devralınmış varsayılanları ve **güncel yapılandırmanın doğrulamasını** birlikte okuyun.

| Sayfada görünen | Nasıl yorumlanır? |
|---|---|
| **Arşiv koruması yapılandırılmış** | Koruma yapılandırılmıştır; doğrulama sonucunu ayrıca kontrol edin. |
| Yakın tarihli kanıtla **Doğrulandı** | Güncel dış kopyanın denetim kayıtlarına karşı doğrulandığı bildirilmiştir. Gösterilen kapsam ve zaman içinde yorumlayın. |
| **Doğrulama bekliyor** | Güncel koruma henüz kanıtlanmamıştır. |
| **Koruma doğrulanmadı / kullanılamıyor** | Sorun çözülene kadar dış arşiv korumasına güvenmeyin. |
| **Hukuki koruma** | Kayıt saklama kilidinin uygulanıp uygulanmadığını gösterir; geçerli saklama süresiyle birlikte inceleyin. |

Korunan denetim geçmişi ile doğrulanmış dış arşiv koruması farklı güvencelerdir. Eski başarılı doğrulama, değişmiş yapılandırmayı doğrulamaz. Profil yoksa, yükleme başarısızsa veya doğrulama sonuçlanmıyorsa görünen durum ve zamanla desteğe başvurun. Bu sayfa uyumluluk sertifikası veya arşiv saklama süresini değiştirme kontrolü değildir.

## Uyarılar, kanallar ve bildirim tercihleri

Desteklenen uyarı akışları için **Uyarılar → Kurallar, Kanallar, Geçmiş, Sessizleştirmeler ve Şablonlar** alanlarını kullanın. Kurallar eşleşmeyi, kanallar hedefleri belirler; geçmiş kayıtlı sonuçları incelemeye yardımcı olur. Hesaptaki **Bildirimler ve Uyarılar** alanında kullanılabilir olay-kanal tercihleri ve kanal ayarları bulunur.

Kullanmak istediğiniz akışın olayını, hedefini ve yetkilerini doğrulayın. Tercihler önizleme olarak gösteriliyor veya kullanılamıyorsa düzenlemenin etkin olduğunu varsaymayın. Kaydedilmiş kanal, eşleşmiş kural veya yapılandırılmış tercih mesaj teslimini kanıtlamaz; mevcut teslim sonucunu ayrıca kontrol edin.

Uyarıyı sessizleştirmek ilgili bildirim akışını bastırır; bulguyu çözmez veya uyarıya yol açan koşulu kaldırmaz. Gelen kutusundaki öğeyi okumak veya kapatmak da sorunun giderildiğini doğrulamaz. Teslimi yalnız yetkili test işlemiyle ve kararlaştırılmış hedefe sınayın.

## Dışa aktarma ve inceleme devri

Yetkiniz varsa Bulgular, görünen tablo sayfasının ötesinde sunucu tarafındaki filtreler ve zaman aralığıyla CSV veya JSON dışa aktarımı sunar. Tek dışa aktarım **50.000 satırla** sınırlıdır; işleme sınırları sonucu daha da daraltabilir. Büyük bir sonuç kümesini incelemek için servis kapsamını veya zaman aralığını daraltın.

Denetim Kaydı ve Oturum Aktivitesi CSV'si, seçilen görünüme veya yüklenmiş kayıtlara bağlı farklı kapsama sahip olabilir. Dışa aktarımı eksiksiz saymadan önce sayfadaki açıklamayı, yüklenen/toplam göstergesini ve oluşan dosyayı kontrol edin.

Devir için organizasyonu ve servisi, filtreleri ve saat dilimini, bulgu referanslarını, kanıt zamanlarını, alınan kararları ve henüz doğrulanmamış sonuçları ekleyin. Kayıtları yalnız hedeflenen alıcılarla paylaşın. Dışa aktarılmış dosya, başka bir güvenlik sistemine otomatik teslim yapıldığını göstermez.

## Yardım ve isteğe bağlı AI

Komuta maskot menüsünü açarak geçerli sayfa için Türkçe veya İngilizce rehberi okuyun. Maskot kullanılabiliyorsa **Bu ekranı açıkla** açıklamayı konuşma balonunda da açar. İlgili servis güvenliği sekmelerinde, erişim korumasında ve yetkili erişim etkinliğinde de bağlama uygun yardım bulunur.

Bu statik rehber, AI sağlayıcısı etkinleştirilmeden çalışır. Dekoratif maskot kapalıyken rehber menü içinde okunabilir; konuşma balonu işlemi kullanılamaz. Amacı, sonraki adımları ve sınırları açıklar; güncel kayıtları incelemez, AI'a göndermez veya işlem yapmaz. Yüklenme ve yetki kontrolleri hangi açıklamanın kullanılabileceğini etkileyebilir.

AI yardımı kullanılabilir ve etkinse yetkili bağlam için isteğe bağlı destek olarak yararlanın. Sayfa bağlamı paylaşımı yeni oturumda kapalı başlar; açılması desteklenen sayfa özetini sorunuza ekleyebilir. İsteme hassas bilgi eklemekten kaçının; öneriyi uygulamadan önce geçerli organizasyonu, servisi ve kanıtı tekrar kontrol edin. AI açıklaması onay, doğrulanmış koruma sonucu veya doğru teşhis garantisi değildir. Değişiklikler kendi yetki ve onay akışlarını izler.

## Yaygın inceleme senaryoları

### Kritik bir bulgu oluştu

Servisi ve kanıt zamanını doğrulayın, etkilenen işlemi inceleyin ve Servis Güvenliği'nde trafik ile korumayı karşılaştırın. İlgili playbook'u kullanın, değerlendirmenizi kaydedin ve yalnız kanıta uygun müdahaleyi yapın. Sonucu kontrol ettikten sonra bulguyu kapatın veya neden ek inceleme gerektiğini kaydedin.

### Müşteri korunan uygulamayı açamıyor

Merkezi Erişim korumasından başlayın; ardından servisin Kurallar, Kişiler ve Etkinlik sekmelerine geçin. Hedeflenen ziyaretçiyi geçerli kuralla ve bildirilen erişim sonucuyla karşılaştırın. Kuralı değiştirmeden önce uygulama ilerlemesini kontrol edin; ilgili erişim yöntemini araştırmak için [Erişim Koruması rehberini](service-access-protection.md) kullanın.

### Koruma değişikliği uygulamayı aksattı

Son dağıtımı ilgili bulgu, istenen ayar ve gözlenen durumla karşılaştırın. İlgili en dar kuralı, yeteneği, yolu veya bağlantıyı belirleyin. Toparlanma planıyla yetkili düzeltme veya geri alma yapın; ardından uygulama sağlığını ve kalan korumayı doğrulayın.

### Panel sakin görünüyor

Önce organizasyonu, zaman aralığını ve filtreleri kontrol edin. Ardından kaynak güncelliğini, çalışma ortamına uygulanabilirliği ve hataları inceleyin. Kapsam eksikse kanıt boşluğunu kaydedin; boş liste etkinliğin hiç oluşmadığını mı, yoksa gözlenmediğini mi tek başına yanıtlayamaz.

## Sorun giderme

| Gördüğünüz durum | Sonraki adım |
|---|---|
| **Yükleniyor** | Sayaçları yorumlamadan önce güncel kapsamın ve yetkilerin belirlenmesini bekleyin. |
| **Boş sonuç** | Filtreleri, zaman aralığını, erişimi ve kanıtın kullanılabilirliğini kontrol edin. |
| **Yetki reddi veya eksik işlem** | Göreviniz için gereken belirli erişimi organizasyon yöneticinizden isteyin. |
| **Eski veya bilinmeyen kanıt** | Zamanları ve kaynak kapsamını karşılaştırın; güncellik düzelmiyorsa destek alın. |
| **Kullanılamıyor / uygulanamaz** | Ortam desteğini inceleyin ve diğer uygulanabilir koruma katmanlarını değerlendirin. |
| **Uygulama başarısız veya bekliyor** | Aynı isteği yeniden oluşturmadan önce geçerli isteği ve hatayı inceleyin. |
| **Bildirim gelmedi** | Kuralı, kanalı, tercihleri, sessizleştirmeleri ve kayıtlı teslim sonucunu karşılaştırın. |
| **Arşiv koruması doğrulanamıyor** | Dış korumayı doğrulanmamış kabul edin; durum ve zamanla desteğe başvurun. |

[Destek](support-tickets.md) isterken etkilenen sayfayı ve servisi, zaman aralığını, görünen durumu ve paylaşılması uygun kayıt referanslarını ekleyin. Parolaları, kimlik doğrulama bilgilerini ve gereksiz kişisel verileri dışarıda bırakın.

## Sık sorulan sorular

### Düşük risk puanı servisim güvenli demek mi?

Seçili kapsamdaki mevcut bulgu ve değerlendirmeleri önceliklendirmenize yardımcı olur. Kapsamı, güncelliği ve etkili koruma kanıtını da inceleyin.

### Bulguyu tehdit olarak işaretlemek kuralı etkinleştirir mi?

Karar, değerlendirmenizi kaydeder. Desteklenen müdahale veya politika işleminin kendi yetkisi, onayı, uygulama durumu ve sonuç doğrulaması vardır.

### İki serviste neden farklı koruma seçenekleri var?

Seçenekler yetkilere, etkin yeteneklere ve çalışma ortamı desteğine bağlıdır. Kata gibi yönetilen izole çalışma ortamları aynı davranış görünürlüğünü sunmaz. [Çalışma Zamanı Güvenliği](runtime-security-guide.md) rehberine bakın.

### Her boş sonuç başarılı kontrol müdür?

Hayır. Filtreler, tamamlanmamış yükleme, eksik yetkiler ve kullanılamayan kaynaklar görünürlüğü sınırlayabilir. Sonuca varmadan önce bu koşulları netleştirin.

### Başarılı senaryo veya arşiv rozeti sertifika olarak kullanılabilir mi?

Senaryo sonucu belirli bir testle, arşiv güvencesi gösterilen kayıt korumasıyla ilgilidir. Hiçbiri uygulamanızı veya organizasyonunuzu tek başına sertifikalandırmaz.

### Security Center için AI gerekli mi?

Hayır. İnceleme sayfaları, yetkili kontroller ve statik sayfa yardımı, kendi kullanılabilirlik ve yetki koşullarıyla isteğe bağlı AI yardımından bağımsız çalışır.

## Devam edin

- [Servis Güvenliği](service-security-guide.md) — tek uygulamada trafik, koruma ve bulguları inceleyin.
- [Çalışma Zamanı Güvenliği](runtime-security-guide.md) — uygulanabilirliği anlayın ve koruma sonuçlarını doğrulayın.
- [Erişim Koruması](service-access-protection.md) — uygulamanızı kimlerin açabileceğini yapılandırın.
- [Erişim Kaydı](access-protection-activity.md) — ziyaretçi erişimini inceleyin.
