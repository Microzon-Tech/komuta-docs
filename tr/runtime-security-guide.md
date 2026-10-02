# Çalışma Zamanı Güvenliği

Çalışma zamanı güvenliği, çalışan iş yükünün davranışını ve korunma durumunu inceler. Komuta; ağ politikaları, iş yükü sıkılaştırması ve uygun ortamlardaki davranış sensörlerini birlikte kullanır. Katmanların kapsamları farklıdır; bir katmanın varlığı diğerlerinin etkili olduğunu kanıtlamaz.

Ekran kullanımı için [Güvenlik Merkezi](https://komuta.io/docs/services/security-center-guide) ve [Servis Güvenliği](https://komuta.io/docs/services/service-security-guide) rehberlerini okuyun.

## Koruma katmanları

| Katman | İncelediği veya sınırladığı alan | Kontrol edilmesi gereken |
|---|---|---|
| **Ağ politikaları** | İş yükleri arasındaki ve dış hedeflere trafik | Hedef, yön, uygulama durumu ve ilgili trafik sonucu |
| **İş yükü sıkılaştırması** | Root izni, Linux capability değerleri ve yazılabilir yollar | İstenen ayar, uygulanan dağıtım ve canlı duruş |
| **Host çalışma zamanı koruması** | Desteklenen iş yüklerinde process, dosya ve capability davranışları | Runtime uygunluğu, sensör durumu, kural ve olay kanıtı |
| **Çalışma zamanı izleme** | Desteklenen ortamda davranış telemetrisi ve ilişkili kanıt | Kaynak erişimi, veri güncelliği ve hedefle eşleşme |
| **Platform Host koruması** | Platformun node ve altyapı güvenliği | Operatör kapsamı, kaynak sağlığı ve platform kanıtı |

Build ve imaj taraması ise ayrı **tedarik zinciri** kanıtıdır. Temiz bir build taraması, canlı iş yükünün bütün davranışlarını denetlemez.

## Host runtime ve Kata uygunluğu

Host üzerinde çalışan sensör, aynı çekirdeği kullanan uygun container davranışını görebilir. Kata gibi ayrı misafir çekirdeği kullanan iş yüklerinde host sensörünün iç process ve dosya görünürlüğü aynı değildir.

Bu nedenle host davranış koruması için **uygulanamaz** veya **kullanılamaz** durumu görülebilir. Ağ politikası, build taraması ve iş yükü sıkılaştırmasını ayrı değerlendirin. Bir sensör adının listelenmesi, o sensörün seçilen servisin iç davranışını gördüğünü kanıtlamaz.

Çalışma ortamı veya kaynak desteği bilinmiyorsa bunu destekleniyor, korumasız ya da güvenli diye kesinleştirmeyin. Hedef servis ve kaynak kapsamını platform operatörüyle doğrulayın.

## Politika aksiyonu ile çalışma zamanı modu

Farklı koruma motorlarının ayarlarını tek bir mod gibi yorumlamayın.

| Ayar | Anlamı |
|---|---|
| **Audit politika aksiyonu** | Eşleşen davranışı gözlemleme ve kaydetme amacı taşır; bu kural için engelleme amacı yoktur. |
| **Block politika aksiyonu** | Eşleşen davranışı, desteklenen ve uygulanmış koruma katmanında reddetme amacı taşır. |
| **Off / Shadow / Audit / Enforce çalışma zamanı modu** | İlgili çalışma zamanı mekanizmasının istenen veya gözlenen modu. Aynı isimli politika aksiyonuyla eşdeğer değildir. |
| **Monitor / Enforce yönetim ayarı** | İlgili operatör koruma kontrolünün yapılandırılması. Hedefe ulaşma ve etkili davranış sonucu ayrıca doğrulanır. |

Audit seçmek diğer katmanların engellemelerini kaldırmaz. Block seçmek ise bütün process, dosya veya bağlantıların reddedilmesi anlamına gelmez; sonuç, kural eşleşmesi ve motorun desteğiyle sınırlıdır.

## Yapılandırma, uygulama ve sonuç

Koruma için üç ayrı soruyu cevaplayın:

1. **Ne istendi?** Doğru servis, yollar, programlar, yön ve davranış için kaydedilmiş kuralı inceleyin.
2. **Ne uygulandı?** Dağıtım sonucu, gözlenen mod ve varsa uygulama hatasını kontrol edin.
3. **Ne oldu?** İlgili hedef ve zaman aralığına ait olay, engelleme veya izin sonucu kanıtını inceleyin.

Bir kuyruğa alınmış dağıtım, başarılı API isteği veya sağlıklı kalp atışı üçüncü soruyu cevaplamaz. Eski yapılandırmanın kanıtı yeni bir kuralın çalıştığını doğrulamaz. Eksik veya başarısız ölçüm güvenli sonuç değildir.

## Temel güvenlik ve servis politikaları

Platform temel politikaları ve çalışma zamanı yönetimi, yetkili operatörlerin sorumluluğundadır. Müşteri Konsolunda iş yükünüze ait uygun politikaları ve servis koruma durumunu inceleyebilir; ayrı izinlerle desteklenen servis ayarlarını yönetebilirsiniz.

Politika sihirbazında Audit veya Block seçmeden önce servis hedefini, yolları ve YAML'ı gözden geçirin. Sihirbazın dosya yollarını kabul etmesi, bu yolların container içinde bulunduğunu veya kuralın etkin olduğunu kanıtlamaz. Geçici dizinden çalıştırmayı kısıtlamak, o dizine her yazmayı veya yorumlayıcı üzerinden tüm betikleri engellemekle aynı şey değildir.

Block; başlangıcı, sağlık kontrollerini, bakım veya zamanlanmış işleri aksatabilir. Uygun bir ortamda önce gözlem ve normal iş akışlarıyla değerlendirin; izinli değişikliği sınırlı kapsam ve geri dönüş planıyla uygulayın. Her servisin aynı başlangıç moduna sahip olduğunu varsaymayın.

## Gözlem ve bulgu inceleme

Servis Güvenliğindeki **Bulgular** sekmesi, uygun çalışma zamanı gözlemlerini ve mevcut karar geçmişini içerir. Kuruluş genelindeki Bulgular ekranı ise kayıtları servis, kaynak, önem ve zamanla önceliklendirir.

Bir gözlem özeti, hesap zamanındaki bekleyen incelemelerin sayısıdır. Bunu bir saldırı sayacı, güncel kuyruk boyutu veya duruş testi sonucu olarak sunmayın. Alt kayıtlar ve kaynak kanıtı ayrıca incelenmelidir.

Bulguya Allow veya Block kararı vermek, gerçek izin veya engelleme politikasının uygulandığı anlamına gelmez. Temel gözlem yönetimi, koruma yükseltme ve platform kontrolleri AdminUI'dadır. Gerçek müdahalenin ayrı izin ve sonuç kontrolü vardır.

## İstisnalar ve geri dönüş

İstisna veya öneri kabulünde hedefi, gerekçeyi, süreyi ve mevcut politikaya etkisini doğrulayın. Bekleyen onay ile uygulanmış istisna aynı değildir. Geri alma talebi de geri dönüşün tamamlandığını kanıtlamaz.

Bir koruma değişikliği uygulamayı bozduysa son dağıtım, gözlenen mod ve ilgili davranışın kanıtını karşılaştırın. İzinli sorumlu ile en dar kapsamlı düzeltmeyi seçin; genel korumayı kapatmak veya bütün bulgulara Allow vermek yerine gerekli davranışı değerlendirin.

## Kontrollü doğrulama

Tatbikat veya koruma testi, kendi başına ayrı bir operasyonel işlemdir. Onaylı hedef, uygun çalışma ortamı, beklenen sinyal, olası etki ve geri dönüş planı olmadan çalıştırmayın. Korunan veya tuzak dosyalara gelişigüzel erişmeyin.

Başarı değerlendirmesi, testin kaydı ile beklenen kaynak kanıtının aynı hedef ve zaman aralığında eşleşmesini gerektirir. Tespit edilen senaryo, bütün saldırıların önlendiği anlamına gelmez. Tespit kanıtı ile engelleme kanıtını ayrı tutun.

## Sorun giderme

| Durum | İnceleme adımı |
|---|---|
| **Permission denied veya uygulama başlangıç hatası** | İlgili process/yol, son koruma değişikliği ve dağıtımı karşılaştırın; yetkili dar kapsamlı düzeltme seçin. |
| **Mod değişti ama sonuç yok** | İstenen/gözlenen mod, dağıtım ve olay kanıtını ayrı kontrol edin. |
| **Hiç bulgu görünmüyor** | Zaman filtresi, izin, runtime uygunluğu ve kaynak güncelliğini doğrulayın. |
| **Politika uygulaması başarısız** | Bildirilen uygulama hatasını ve hedefi inceleyin; yinelenen kayıt oluşturmadan mevcut politikayı kontrol edin. |
| **Kata iş yükünde host kartı yok** | Katmanın uygunluğunu kontrol edin; diğer koruma katmanlarını kendi kanıtlarıyla değerlendirin. |

Kaynak sağlığı, platform Host veya cluster genelindeki ayarlar için operatör konsolu gerekir. Müşteri Güvenlik Merkezindeki boş sonuçtan altyapının sağlıklı olduğu sonucuna varmayın.

## Maskot rehberi

**Bu ekranı açıkla**, bulunduğunuz güvenlik sayfası veya servis sekmesi için statik Türkçe/İngilizce yardım sunar. AI sağlayıcısı kapalıyken de kullanılabilir; veri sorgulamaz veya koruma işlemi yürütmez.

Rehber ve ayrı yapay zekâ önerileri, güncel uygulama ve davranış kanıtının yerine geçmez. Değişiklikleri sayfanın kendi izin ve onay akışında yapıp sonuçlarını doğrulayın.

## İlgili dokümanlar

- [Güvenlik Merkezi](https://komuta.io/docs/services/security-center-guide)
- [Servis Güvenliği Çalışma Alanı](https://komuta.io/docs/services/service-security-guide)
- [Servis erişim koruması](https://komuta.io/docs/services/service-access-protection)
