# Güvenlik Merkezi (Security Center)

Komuta Güvenlik Merkezi, kuruluşunuzun iş yüklerine ait güvenlik bulgularını, koruma durumunu ve olay kanıtlarını bir arada incelemenizi sağlar. **Güvenlik** menüsünden başlayın; tek bir iş yükünü araştırırken servisin **Güvenlik** çalışma alanına geçin.

Gördüğünüz sayfalar ve kullanabileceğiniz işlemler hesabınızın izinlerine, seçili kuruluşa ve iş yükünün çalışma ortamına bağlıdır. Önce kapsamı, veri kaynağını ve verinin zamanını kontrol edin. Boş sonuç veya düşük risk skoru, tek başına güvenli olduğunuz anlamına gelmez.

## Hangi ekran hangi kapsam için çalışır?

| Ekran | Kapsam | Ne için kullanılır? |
|---|---|---|
| Müşteri Konsolu → Güvenlik | Güncel kuruluşunuzun, hesabınıza açık iş yükleri ve kayıtları | Kuruluş genelinde önceliklendirme, bulgu inceleme ve izinli müdahale |
| Servis → Güvenlik | Açık olan servis | Trafiği, korumayı, bulguları ve olay zaman çizelgesini aynı servis bağlamında inceleme |
| AdminUI → Güvenlik | Yetkili platform operatörünün erişebildiği, ekranda belirtilen kapsam | Platform işletimi, kuruluşlar arası inceleme ve altyapı güvenliği yönetimi |

AdminUI, platform operatör konsoludur. Müşteri kuruluşunda yönetici olmak, platform operatörü olmakla aynı erişimi sağlamaz. AdminUI'da **platform Host kapsamı** ile **seçili kuruluş kapsamı** farklıdır; bir sayfadaki hedef seçimi başka sayfanın hedefini otomatik belirlemez.

## Müşteri Konsolundaki sayfalar

**Güvenlik** menüsü dört iş grubuna ayrılır. İzin veya çalışma ortamı nedeniyle bazı sayfalar görünmeyebilir.

| Grup | Sayfa | Başlangıç noktası |
|---|---|---|
| İzle | **Genel Bakış** (`/security`) | Kapsam, risk, açık bulgular ve kanıt durumunu kontrol edin. |
| İzle | **Bulgular** (`/security/findings`) | Servis, kaynak, önem ve zaman filtreleriyle araştırın. |
| Kayıt | **Denetim Kaydı** (`/security/audit`) | Kaydedilmiş işlemleri, aktörü ve sonucu karşılaştırın. |
| Kayıt | **Oturum Aktivitesi** (`/security/login-activity`) | Kuruluşun giriş faaliyetlerini ve beklenmedik sonuçları inceleyin. |
| Kayıt | **Denetim kaydı koruması** (`/security/audit-storage`) | Kendi kapsamınızın arşiv güvencesini ve güncel doğrulamasını okuyun. |
| Koru | **Politikalar** (`/security/policies`) | İş yükü politikalarını ve önerileri hedefleriyle birlikte inceleyin. |
| Koru | **Engellemeler** (`/security/blocks`) | Mevcut engelleri, etkiledikleri servisleri ve kanıtlarını inceleyin. |
| Koru | **Honey path'ler** (`/security/honey-paths`) | Desteklenen servislerin tuzak yollarını ve tespit kayıtlarını inceleyin. |
| Koru | **Olay Müdahale Playbook'ları** (`/security/playbooks`) | İnceleme ve müdahale adımlarını izleyin. |
| Doğrula | **Sentetik saldırılar** (`/security/synthetic-attacks`) | Uygun senaryoları, izinli tatbikatları ve tespit sonuçlarını inceleyin. |
| Doğrula | **Tedarik zinciri** (`/security/supply-chain`) | Build ve imaj taramalarını, çıktı kanıtlarını ve istisnaları inceleyin. |

## Gözlem, bulgu ve kanıt arasındaki fark

| Kavram | Anlamı |
|---|---|
| **Gözlem** | Bir kaynağın kaydettiği davranış veya durum. Henüz bir saldırı ya da kalıcı engelleme kararı olduğu anlamına gelmez. |
| **İhlal** | Bir koruma kuralıyla ilişkili kayıtlı davranış. Kaydın aksiyonu Audit ise gözlemleme, Block ise bildirilen engelleme sonucudur. |
| **Bulgu** | İncelenen ve yaşam döngüsü yönetilen güvenlik kaydı. Kaynağı, önem derecesi, servis bağlamı ve kanıtıyla değerlendirilir. |
| **Tekrar** | Aynı mantıksal bulgunun yeniden görülmesi. Tekrar sayısı ve son görülme zamanı, ayrı bir olay kimliğinin yerine geçmez. |
| **Gözlem özeti** | Hesap anında inceleme bekleyen gözlemlerin toplu sayısı. Yeni saldırı, başarısız duruş kontrolü veya güncel kuyruk boyutu olarak okunmamalıdır. |
| **Kanıt** | Bulguyu veya işlemi açıklayan kaynak, zaman, hedef ve sonuç bilgisi. Güncelliği ve ilgili kapsamla eşleşmesi gerekir. |

Duruş puanı, bekleyen gözlem sayısı ve olay tekrar sayısı farklı ölçümlerdir. Bir gözlem özetini kapatmak, alttaki gözlemleri değerlendirmez veya korumayı değiştirmez.

## Koruma durumunu nasıl yorumlamalı?

| Gösterilen durum | Ne anlatır? | Sonraki kontrol |
|---|---|---|
| **Yapılandırıldı / istenen mod** | Bir kural veya ayar kaydedildi. | Doğru servise ve kapsama ait mi? |
| **Bekliyor / sıraya alındı** | Uygulama süreci henüz tamamlanmadı. | Dağıtım sonucu ve gözlenen mod nedir? |
| **Uygulandı / gözlenen mod** | Platform, hedefteki uygulama durumunu bildiriyor. | Veri güncel mi; ilgili davranışta beklenen sonuç var mı? |
| **Etkisi doğrulandı** | İlgili hedef, davranış ve zaman aralığı için sonuç kanıtı mevcut. | Kanıt, güncel yapılandırma ve çalışma ortamıyla eşleşiyor mu? |
| **Bilinmiyor / kullanılamıyor / eski** | Sonuç doğrulanamıyor. | Kaynak erişimi, uygunluk ve veri güncelliğini araştırın. |

Kaydedilmiş Block veya Enforce ayarı, uygulama kuyruğuna alınmış değişiklik ya da sağlıklı bir sensör kalp atışı, tek başına iş yükünde etkili engelleme kanıtı değildir. Aynı şekilde bulgu bulunmaması, kaynakların tüm davranışları gördüğünü kanıtlamaz.

Host üzerinde çalışan sensörler ile Kata gibi ayrı misafir çekirdeği kullanan çalışma ortamlarının görünürlüğü farklıdır. Host çalışma zamanı korumasının desteklenmediği ortamda bunun için **uygulanamaz** veya **kullanılamaz** durumu beklenebilir; ağ, duruş ve tedarik zinciri yeteneklerini kendi uygunluklarına göre değerlendirin. Ayrıntılar için [Çalışma Zamanı Güvenliği](https://komuta.io/docs/services/runtime-security-guide) rehberini okuyun.

## Genel Bakış

Genel Bakış, kuruluşunuzdaki araştırılacak riskleri ve iş yüklerini önceliklendirir. Risk kartını açık bulgularla birlikte okuyun; kaynak kapsamını, zaman aralığını, kanıtın güncelliğini ve hizmete ait koruma durumunu karşılaştırın.

Bir özet kartından Bulgulara veya ilgili servise geçerken filtreleri kontrol edin. Risk skoru ve sertleştirme göstergeleri operasyonel önceliklendirmeye yardım eder; uyumluluk sertifikası veya tüm saldırıların önlendiğine dair garanti değildir.

## Bulgular (Findings)

1. Zaman aralığını, servisi, kaynağı, önem derecesini ve durumu daraltın.
2. Bulgu ayrıntısında ilk ve son görülmeyi, tekrarları, etkilenen işlemi ve kanıtı okuyun.
3. İlgili servis çalışma alanında trafiği, koruma durumunu ve olay zaman çizelgesini karşılaştırın.
4. Yalnız hesabınızın izin verdiği kararı veya müdahale işlemini seçin; istenen gerekçeyi ve hedefi doğrulayın.
5. İşlemden sonra kaydedilen kararı ve varsa dağıtım veya müdahale sonucunu ayrı kontrol edin.

### Bulgu kararları ve gerçek müdahale

| Karar | Anlamı |
|---|---|
| **Acknowledge** | Bulgu incelemeye alındı. |
| **Allow** | Davranış meşru kabul edildi; gerekli gerekçeyi kaydedin. |
| **Block** | Davranışın engellenmesi gerektiği kararı kaydedildi. |
| **Dismiss** | Bulgu geçersiz veya kapsam dışı olarak değerlendirildi. |
| **Resolve** | İnceleme sonuçlandırıldı; kapanış nedenini doğrulayın. |

Bu kararlar ile **çalışma zamanı engelleme politikası oluşturma**, **politika istisnası uygulama** ve **iş yükünü izole etme** ayrı işlemlerdir. Bir Block kararı kernel engellemesini, Allow kararı ise otomatik izin kuralını kanıtlamaz. Gerçek müdahalede ek izinler, çalışma ortamı uygunluğu, açık onay ve sonuç kontrolü gerekir.

İzolasyon ağ erişimini etkileyebilir. Uygulamayı geri açmadan önce doğru hedefi, kaydedilmiş izolasyon durumunu ve geri dönüş sonucunu kontrol edin.

## Politikalar

**Politikalar** ekranında iş yükünüze ait koruma kuralları ile **Öneriler** bölümünü ayrı değerlendirin. Host çalışma zamanı kuralları ile ağ politikaları farklı kaynaklara ve uygunluk koşullarına bağlıdır. Platform ve küme genelindeki yönetim AdminUI'dadır.

Koruma politikası oluşturma sihirbazı, uygun servisi seçip kabuk çalıştırma, hassas dosya erişimi, belirli programlar veya geçici dizinlerden çalıştırma amaçları için kural hazırlamanıza yardımcı olur. Cluster ve namespace seçilen servisten alınır. Oluşturmadan önce hedefi, yolları, Audit/Block davranışını ve YAML önizlemesini inceleyin.

Sihirbazın uygun servis göstermesi, canlı sensör sağlığı veya etkili koruma kontrolü değildir. Oluşturulan kayıt platform tarafından uygulanabilir; sonrasında uygulama durumu ve yetkili bir testin kanıtı kontrol edilmelidir. Block, başlangıç, sağlık kontrolü veya bakım işlemlerini aksatabilir.

### Önerilen Politikalar (Suggested Policies)

Önerinin hedefini, dayandığı gözlemleri, güven düzeyini ve kural farkını inceleyin. Kabul, uygulama ve geri alma sonuçlarını ayrı takip edin. Başarısız uygulama veya bekleyen dağıtım, etkin koruma olarak gösterilmemelidir. Hesabınızda sunulan işlemler için ayrı öneri yönetimi yetkisi gerekir.

### Politika İstisnaları (Policy Exceptions)

İstisna, belirli bir gerekçe ve süreyle koruma davranışını gevşetebilir. Kapsamı, onay durumunu, son kullanma zamanını ve mevcut politikaya etkisini inceleyin. Bir istisna talebinin oluşturulması, onaylanması veya bulguya Allow kararı verilmesi, istisnanın iş yüküne etkili biçimde uygulandığı anlamına gelmez.

## Engellemeler ve honey path'ler

**Engellemeler** ekranında etkilenen servisi ve kaydın dayandığı kanıtı kontrol edin. Engel kaldırmak ayrı izinli işlemdir; talebin kabul edilmesiyle iş yükünün sağlıklı çalışmaya dönmesini birbirinden ayırın.

**Honey path'ler**, meşru uygulama davranışının dokunmaması gereken servis yollarının izlenmesi içindir. Hedef servis, çalışma ortamı desteği ve kurulum durumu doğrulanmalıdır. Ayarın açık olması tespit kanıtı değildir. Tuzak dosyayı okuyarak, yazarak veya yoklayarak deneme yapmayın; yalnız açıkça onaylanmış tatbikat ve hedef kullanın.

## Denetim Kaydı ve Oturum Aktivitesi

**Denetim Kaydı**, hesabınıza açık güvenlik faaliyetlerini kaynak, zaman, aktör ve sonuç bağlamında araştırmak içindir. Toplu görünüm tekrarlanan olayları özetler; ham görünüm tekil kayıtları incelemeye yardımcı olur. CSV'nin seçilen ham veya toplu görünüm ve yüklenen kayıtlarla kapsamını kontrol edin.

**Oturum Aktivitesi**, kuruluşun giriş faaliyetlerini gösterir. Kişisel hesabınızın güvenlik günlüğü ve etkin oturumları ayrı hesap ekranlarıdır. Beklenmedik girişte ilgili hesabı, zamanı, sonucu ve mevcut kanıtı doğrulayın.

Oturum Aktivitesindeki filtreler, sayaçlar ve CSV **yüklenmiş olayları** kapsar. Ekrandaki yüklenen/toplam gösterimini kontrol edin; daha eski olayları yüklemek inceleme kapsamını genişletir. Yükleme hatası, hiç faaliyet olmadığı anlamına gelmez.

## Olay Müdahale Playbook'ları (IR Playbooks)

Playbook'lar araştırma ve müdahale için sıralı yönergeler sunar. Hazır rehberleri inceleyebilir, yetkiniz varsa kendi sürecinize uygun kopya veya özel rehber yönetebilirsiniz.

Her adımın ön koşulunu ve beklenen kanıtını kontrol edin. Rehberi açmak, otomatik izolasyon, kimlik bilgisi değiştirme veya bildirim gönderme işlemi başlatmaz. Desteklenen müdahaleyi yalnız sayfanın ayrı, yetkili kontrolleriyle uygulayın ve sonucunu doğrulayın.

## Sentetik saldırılar

Tatbikatlar, seçilen hedef ve senaryoya ait tespit hattını değerlendirmeye yardımcı olur. Senaryonun görünmesi veya etkin olması, çalıştırıldığı anlamına gelmez. Çalıştırma ayrı izin ve açık işlem gerektirir.

Önce hedef servisi, çalışma ortamını, beklenen sinyal kaynağını, senaryonun etkilerini ve geri dönüş planını doğrulayın. Sonuçta çalışma kaydını, beklenen bulguyu ve tespit zamanını karşılaştırın. Bir senaryonun tespit edilmesi, tüm saldırı sınıflarının önlendiğini veya engellemenin etkili olduğunu kanıtlamaz. Platform genelindeki senaryo kullanılabilirliği AdminUI kataloğundan yönetilir.

## Tedarik zinciri

Build ve imaj güvenliğini değerlendirirken taranan çıktıyı, tarama zamanını, önem derecesini ve sunulan kanıtları inceleyin. SBOM, zafiyet taraması veya imza bilgisi bulunuyorsa bunu ilgili build ve imajla eşleştirin; başka bir çıktının kanıtını kullanmayın.

Eksik, eski veya başarısız taramayı temiz sonuç saymayın. İstisna varsa gerekçesi, kapsamı ve süresini inceleyin. Bir riski kabul etmek zafiyeti gidermez; imza kaydı bulunması da tüm build'lerin imzalandığını veya imzanın bağımsız doğrulandığını kanıtlamaz.

## Denetim kaydı koruması

Müşteri görünümü, **kendi kapsamınızın arşiv koruma güvencesini** okumak içindir. Saklama süresi, hukuki saklama (legal hold), platformdan devralınan ayarlar, harici değişmez depolama durumu ve güncel doğrulama sonucunu birlikte inceleyin.

Yapılandırılmış arşiv ile **mevcut yapılandırmanın doğrulanmış olması** ayrıdır. Bekleyen, geriden gelen, başarısız, bilinmeyen veya kullanılamayan doğrulama durumunu güncel değişmezlik kanıtı olarak sunmayın. Son başarılı doğrulamanın zamanını kontrol edin.

Eklenebilir denetim kaydı ve kriptografik zincir, harici **WORM / Object Lock** saklama ile aynı güvence değildir. Depolama değişmezliği ve varsa imza doğrulaması, gerçek yapılandırma ve ilgili doğrulama kanıtlarıyla değerlendirilir. Bucket, arşiv hedefi, hukuki saklama yönetimi ve kuruluşlar arası seçim gibi operasyonel kontroller AdminUI'dadır.

## Alarmlar, bildirimler ve dışa aktarım

Güvenlik bildirimlerini **Alarmlar → Kurallar, Kanallar, Geçmiş, Susturmalar ve Şablonlar** üzerinden takip edin. Kural eşleşmesi, kanal yapılandırması ve gerçek mesaj teslimi farklı sonuçlardır. Bir alarmın susturulması, bulgunun çözülmesi veya koruma riskinin giderilmesi değildir.

Bulguların izinli **CSV/JSON dışa aktarımı**, yalnız ekrandaki sayfayla sınırlı kalmadan filtre ve zaman aralığına göre sunucuda hazırlanır. Bir dosya en fazla **50.000 satır** içerir; tarama sınırları da sonucu daraltabilir. Sınıra ulaşan çıktıyı bütün eşleşmelerin eksiksiz dökümü saymayın; zaman veya servis kapsamını daraltın. Denetim Kaydı ve Oturum Aktivitesi CSV’lerinde yüklenmiş kayıt kapsamını ayrıca kontrol edin. Dosyanın kapsamını, zamanını ve kayıtlarını inceleme amacınıza göre doğrulayın.

Dışa aktarılan dosya, SIEM sistemine otomatik aktarım veya bildirim teslimi kanıtı değildir. Kuruluşunuz bir entegrasyon kullanıyorsa hedef, şema, erişim ve teslim kanıtlarını ayrıca doğrulayın. Bu rehber, her kuruluşa otomatik SIEM gönderimi veya belirli bir teslim süresi vaat etmez.

## AdminUI'da operatöre ait sayfalar

Platform operatörleri, aşağıdaki ekranları **Güvenlik** başlığı altında, ilgili izin ve kapsam koşullarıyla kullanır. Müşteri Konsolunun kuruluş görünümü bu yönetim ekranlarının yerine geçmez.

| Sayfa | Operatörün görevi |
|---|---|
| **Güvenlik Bulguları** | İzinli sekmelerden kuruluşlar arası bulgular, adli inceleme ve telemetriyi değerlendirme |
| **Gözlemler** | Kuruluşlar arası temel güvenlik gözlemlerini inceleme |
| **Ağ Olayları** | İzinli kuruluşlar arası ağ olaylarını inceleme |
| **Denetim Kayıtları** | Kuruluşlar arası API denetim özetlerini inceleme |
| **Güvenlik sinyali kapsamı** | Gösterilen kapsamdaki kaynakların çerçeve kontrolleriyle eşlemesini değerlendirme |
| **Servis temel güvenlik yönetimi** | Açıkça seçilen servisin temel güvenlik gözlemleri, koruma modu ve yönetim işlemleri |
| **Altyapı sağlık bulguları** | Platform Host kapsamındaki altyapı kaynaklı bulguları inceleme |
| **Güvenlik saklama politikası** | Gösterilen kapsamın süre ve hukuki saklama politikasını yönetme |
| **Çalışma zamanı koruma yönetimi** | Yapılandırılmış yaptırım, uygunluk ve platform kontrollerini yönetme |
| **Denetim depolama yönetimi** | Seçili kapsamın arşiv yapılandırması ve doğrulamasını yönetme |
| **Güvenlik senaryo kataloğu** | Platform genelindeki senaryo kullanılabilirliğini ve kaynak hazırlığını yönetme |

## Komuta maskotu ile sayfa yardımı

Müşteri Konsolunda maskot menüsünden **Bu ekranı açıkla** seçeneğini açın. Rehber, bulunduğunuz güvenlik sayfasının veya servis sekmesinin amacını, nereden başlayacağınızı ve dikkat etmeniz gereken sınırları Türkçe veya İngilizce anlatır. İlgili alarm ve hesap güvenliği ekranlarında da bağlama uygun açıklamalar bulunur.

Bu açıklama için yapay zekâ sağlayıcısının açık olması gerekmez. Dekoratif maskot kapalıyken de sayfa rehberi kullanılabilir. Statik açıklama güvenlik kaydı okumaz, yapay zekâya veri göndermez ve işlem yürütmez. Erişim kontrolü sürüyorsa veya sayfaya izniniz doğrulanmadıysa rehber bu durumu açıklar.

AdminUI'da maskot simgeli **Komuta sayfa rehberi**, desteklenen güvenlik ekranının amacı, ön koşulları, başlangıç adımı ve sınırlarını anlatır. Bu operatör rehberi, otomatik işlem yapan bir sohbet veya animasyonlu müdahale sistemi değildir.

Ayrı yapay zekâ sohbetini kullanırken gizli değerleri paylaşmayın. Yapay zekâ açıklaması veya önerisi, kaydedilmiş işlem sonucu, güncel yetki, etkili koruma veya uyumluluk kanıtının yerine geçmez. Güvenlik değişiklikleri sayfanın kendi izin, onay ve sonuç kontrolüne bağlıdır.

## Pratik inceleme akışı

1. Güncel kuruluşu ve araştırdığınız servisi doğrulayın.
2. Genel Bakışta kaynak kapsamını ve veri güncelliğini kontrol edin.
3. Bulgular listesini servis, kaynak ve zamanla daraltın.
4. Servis Güvenliğinde trafik, koruma ve kanıtı karşılaştırın.
5. Gerekirse playbook'u ve **Bu ekranı açıkla** rehberini kullanın.
6. Yalnız yetkili müdahaleyi açıkça uygulayın; işlem kaydı ve gerçek sonucu doğrulayın.
7. Altyapı, kaynak sağlığı, saklama veya arşiv sorunu için platform operatörüne başvurun.

## İlgili dokümanlar

- [Servis Güvenliği Çalışma Alanı](https://komuta.io/docs/services/service-security-guide)
- [Çalışma Zamanı Güvenliği](https://komuta.io/docs/services/runtime-security-guide)
- [Servis erişim koruması](https://komuta.io/docs/services/service-access-protection)
