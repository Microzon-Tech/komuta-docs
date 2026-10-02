# Servis Güvenliği Çalışma Alanı

Servis detayındaki **Güvenlik** bölümü, açık olan iş yüküne ait trafik, koruma, bulgu ve kanıtları bir araya getirir. Kuruluş genelinde önceliklendirme için [Güvenlik Merkezi](https://komuta.io/docs/services/security-center-guide) kullanın; bir kaydı araştırırken ilgili servise dönün.

## Başlamadan önce

Servis adını ve güncel kuruluşu doğrulayın. Ekranda gösterilen çalışma ortamı, kaynak uygunluğu ve verinin zamanını kontrol edin. Dağıtılmamış servis, desteklenmeyen kaynak, yükleme hatası veya eski veri, temiz güvenlik sonucu değildir.

Her bölümün okuma ve yönetim izinleri ayrıdır. Bir bölümü okuyabilmek, bulgu kararı, izolasyon, capability değişikliği veya koruma modu yönetimi yapma yetkisi sağlamaz.

## Sekmeler ve kapsam

| Sekme | İçerik | İlk kontrol |
|---|---|---|
| **Genel bakış** | Servisin güvenlik duruşu, kaynak kapsamı, öne çıkan bulguları ve önerilen inceleme adımları | Doğru servis ve güncel kanıt |
| **Trafik** | Akışlar, ağ olayları, düşen trafik, DNS ve sunulan trafik ayrıntıları | Zaman aralığı ve kaynağın kullanılabilirliği |
| **Koruma** | Ağ politikaları, duruş ve sapma, çalışma zamanı modu ve iş yükü sıkılaştırması | Kural hedefi ve uygulama durumu |
| **Bulgular** | Servise ait bulgular, uygun çalışma zamanı gözlemleri ve olay zaman çizelgesi | Kaynak, olay zamanı ve ilgili işlem |

İleri kontrollerin bağlantıları, aynı servis bağlamında capability, yazılabilir yollar, root izni veya çalışma zamanı gözlem incelemesini açabilir. Eski Gözlemler bağlantısı, uygun Bulgular alt bölümüne yönlendirilir.

## Genel bakış

Özet ve sonraki adım önerilerini, kaynak kapsamıyla birlikte okuyun. İnceleme bekleyen gözlem sayısı, başarısız duruş kontrolü veya yeni saldırı sayısı değildir. Düşük risk ve boş bulgu listesi, bütün sensörlerin sağlıklı veya korumanın etkili olduğunu kanıtlamaz.

Bir öneriden başka sekmeye geçtiğinizde servisin ve zaman aralığının aynı araştırmayı temsil ettiğini kontrol edin. Kaynak bilinmiyor, eski veya eksikse platform operatörüyle uygunluk ve veri akışını araştırın.

## Trafik

İlgili alt bölümü ve zaman aralığını seçin. Bağlantı kaynağı, hedefi, yönü, portu ve bildirilen sonucu, servis ağ politikalarıyla karşılaştırın.

Düşen trafik kaydı ilgili bağlantının bildirilen sonucudur; tüm bağlantıların engellendiği anlamına gelmez. DNS veya uygulama katmanı ayrıntıları yalnız ilgili kaynak ve hizmet kapsamı destekleniyorsa bulunur. Eksik telemetriyi sıfır trafik olarak değerlendirmeyin.

## Koruma

### Ağ politikaları ve canlı duruş

Gelen ve giden kuralları servisin ihtiyaçlarıyla karşılaştırın. Duruş kartında beklenen temel güvenlik durumu ile çalışan iş yükünün bildirilen durumu arasındaki sapmayı inceleyin.

Sapma veya sorgu hatasında uygulama durumu, mevcut dağıtım ve kaynak zamanı birlikte değerlendirilmelidir. Yeniden dağıtımın sıraya alınması, temel güvenlik durumunun düzeldiğini kanıtlamaz.

### Çalışma zamanı modu

Desteklenen ortamda **Çalışma zamanı koruma modu** kartı, **istenen modu**, **gözlenen modu** ve dağıtım durumunu ayrı gösterir. Off, Shadow, Audit ve Enforce bir çalışma zamanı modunun değerleridir; başka bir koruma motorundaki Audit/Block aksiyonuyla aynı durum alanı değildir.

- **Bekliyor / sıraya alındı**: değişiklik henüz hedefe ulaşmamış olabilir.
- **Başarısız**: istenen ayar kaydedilmiş olsa da uygulama doğrulanmamıştır.
- **Gözlenen mod**: bildirilen mevcut durumdur; ilgili davranışın etkili biçimde engellendiği ayrıca kanıtlanmalıdır.
- **Bilinmiyor / kullanılamıyor**: aktif yaptırım iddiası kurulamaz.

Müşteri görünümünden koruma durumunu okuyun. Temel güvenlik değerlendirme, sıfırlama, Block/Enforce'a yükseltme ve platform genelindeki koruma yönetimi, ilgili yetkilerle **AdminUI → Servis temel güvenlik yönetimi / Çalışma zamanı koruma yönetimi** ekranlarında yapılır. [Çalışma Zamanı Güvenliği](https://komuta.io/docs/services/runtime-security-guide) rehberi katmanların farkını açıklar.

### Linux capability değerleri

Varsayılan capability sıkılaştırmasını ve izinli dar listeyi inceleyin. İmajın gerektirdiği yeteneği ekleme veya çıkarma, ayrı capability yönetimi yetkisi ister. Yalnız uygulamanın ihtiyaç duyduğu değerleri kullanın.

Kaydedilen değer, çalışan pod'un değerinin değiştiğini tek başına kanıtlamaz. Uygulanan dağıtımı ve uygulama sağlığını kontrol edin.

### Yazılabilir yollar

Salt okunur kök dosya sistemine ek olarak, uygulamanın gerçekten yazması gereken dizinleri tanımlayın. Yol ve depolama kurallarını, framework'ün ihtiyaçlarını ve kalıcılık beklentisini kontrol edin. Hassas sistem yolları için sunucunun kısıtları uygulanır.

Yol eklemek ayrı yönetim yetkisi gerektirir. Kaydetme sonrasında dağıtım sonucunu doğrulayın. Korunan dosyaları okuyarak veya yazarak deneme yapmayın; uygun bir doğrulama planı kullanın.

### Root olarak çalıştırma izni

Bu geçersiz kılmayı yalnız imaj gerçekten root izni gerektiriyorsa değerlendirin. İzin vermek, canlı process'in hangi kullanıcıyla çalıştığına dair kanıt değildir. Ayrı yönetim yetkisi, uygulanan dağıtım ve çalışma zamanı duruşu kontrolü gerekir.

### Önizleme

Sunulan politika veya temel güvenlik önizlemesinde hedefi, kuralları ve değişiklikleri inceleyin. Önizleme, planlanan içeriği gösterir; kaydetme, uygulama veya etkili koruma sonucu değildir.

## Bulgular ve kanıt

Servis bulguları ve olay zaman çizelgesi farklı okuma izinlerine bağlı olabilir. Birinin kullanılamaması diğerinin boş veya temiz olduğu anlamına gelmez.

Bulgu ayrıntısında kaynağı, önem derecesini, ilk ve son görülmeyi, tekrarı ve ilgili işlemi inceleyin. Zaman çizelgesini araştırdığınız olayın öncesi ve sonrasıyla karşılaştırın. Yüklenen kayıt miktarı ve zaman filtresini dikkate alın.

### Çalışma zamanı gözlem incelemesi

Uygun servislerde, Bulgular içindeki çalışma zamanı gözlem bölümü kaydedilmiş davranışları ve mevcut karar geçmişini gösterir. Bekleyen, izinli, engelli veya kapatılmış gözlemleri filtreleyebilirsiniz.

Gözlem, otomatik bir saldırı veya engelleme politikası değildir. Müşteri görünümü kayıtları incelemek içindir; temel güvenlik gözlem yönetimi operatör ekranındadır. İşlem gerektiren güvenlik bulgusuna, Bulgular bölümündeki yetkili yanıt akışından karşılık verin.

### Karar ve müdahale

Acknowledge, Allow, Block, Dismiss ve Resolve kararları ile gerçek politika uygulama veya izolasyon ayrı işlemlerdir. Allow kaydı doğrudan izin kuralı, Block kaydı doğrudan kernel engellemesi değildir. Ayrıntılar için [Güvenlik Merkezi](https://komuta.io/docs/services/security-center-guide) rehberini kullanın.

## İş yükü izolasyonu

İzolasyon, iş yükünün ağ erişimini sınırlayan bir müdahaledir. Uygun runtime'da ve ayrı izolasyon yetkisiyle kullanılabilir. Hedefi, gerekçeyi, beklenen etkiyi ve geri açma planını doğrulamadan işlem yapmayın.

İzolasyon isteğinin kabul edilmesi, tam ağ kesintisinin ölçüldüğü anlamına gelmez. Aynı şekilde geri açma isteği, uygulamanın sağlıklı döndüğünü kanıtlamaz. İşlem sonrası bildirilen durumu ve ilgili ağ/uygulama kanıtını kontrol edin.

## Çalışma ortamı uygunluğu

Host sensörü, Kata gibi ayrı misafir çekirdeği kullanan bir iş yükünün içindeki tüm davranışı göremez. Bazı host çalışma zamanı kartları bu nedenle uygulanamaz veya görünmez olabilir. Bu durum iş yükünün güvensiz ya da tamamen korunmuş olduğu şeklinde tek başına yorumlanamaz.

Ağ, iş yükü sıkılaştırma, build taraması ve izolasyon yeteneklerini kendi uygunluklarına göre değerlendirin. Bilinmeyen kaynak durumunu destekleniyor veya sağlıklı saymayın.

## Maskot ile yardım

İlgili sekmede maskot menüsünden **Bu ekranı açıkla** seçeneğini kullanın. Rehber Genel bakış, Trafik, Koruma, Bulgular ve desteklenen ileri bölümler için bağlama uygun başlangıç adımını açıklar.

Statik yardım Türkçe ve İngilizcedir; yapay zekâ sağlayıcısı veya dekoratif maskot açık olmak zorunda değildir. Yardımı okumak servis kaydı sorgulamaz, yapay zekâya veri göndermez veya değişiklik yapmaz. Gerçek işlemler sayfanın kendi yetki ve onay akışında yapılır.

## Pratik akışlar

### Şüpheli bir bulguyu araştırma

1. Doğru servis ve kuruluşta olduğunuzu kontrol edin.
2. Bulgunun kaynağı, zamanı ve kanıtını okuyun.
3. Trafik ve olay zaman çizelgesini aynı zaman aralığıyla karşılaştırın.
4. İlgili playbook'u okuyun; yalnız izinli müdahaleyi uygulayın.
5. Kaydedilmiş karar ile gerçek müdahale sonucunu ayrı doğrulayın.

### Uygulama bir koruma değişikliğinden sonra çalışmıyor

1. Son dağıtımı, istenen/gözlenen modu ve hata kaydını karşılaştırın.
2. İlgili process, yol veya bağlantıya ait kanıtı inceleyin.
3. Gerekli capability veya yazılabilir yol değişikliğini en dar kapsamla değerlendirin.
4. Temel güvenlik modu ya da istisna gerekiyorsa yetkili sorumluya başvurun; her bulguya Allow vermeyi çözüm saymayın.
5. İzinli değişiklikten sonra uygulama sağlığını ve koruma sonucunu doğrulayın.

### Koruma kanıtı yok

Çalışma ortamı desteğini, kaynak erişimini, verinin zamanını ve sayfa izinlerini kontrol edin. Boş listeyi güvenli sonuç olarak raporlamayın. Kaynak sağlığı veya platform ayarı sorunu için operatöre başvurun.

## İlgili dokümanlar

- [Güvenlik Merkezi](https://komuta.io/docs/services/security-center-guide)
- [Çalışma Zamanı Güvenliği](https://komuta.io/docs/services/runtime-security-guide)
- [Servis erişim koruması](https://komuta.io/docs/services/service-access-protection)
