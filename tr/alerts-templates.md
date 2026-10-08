# Şablonlar, Metrikler ve Loglar

**Uyarılar → Şablonlar** sayfasında arama, kategori ve kapsam filtreleriyle başlayın. **Kullan** seçeneği, hedefi seçip parametreleri inceleyeceğiniz sihirbazı açar. Görünen seçenekler erişilebilir kaynaklarınıza ve iş yükünün uygunluğuna göre değişir.

Kendi cluster’ınızda metrik değerlendirmesi için Prometheus, log değerlendirmesi için Loki ve bildirim/susturma akışı için Alertmanager bileşenlerinin uygun biçimde kurulmuş ve çalışır olması gerekir. Eksik bileşen hatasını cluster yöneticinizle giderin. Komuta’da yalnız servis kullanan müşterinin bu bileşenlere ait erişim adreslerini yapılandırması beklenmez.

## Hangi koşul için hangi şablon?

| Gözlemek istediğiniz durum | Başlangıç seçimi | Seçim gerekçesi ve sınırı |
| --- | --- | --- |
| Süren kaynak baskısı | Servis CPU veya bellek yüzdesi | Tanımlı limite ne kadar yaklaşıldığını izler. Limit yoksa bu oranı güvenilir başlangıç kabul etmeyin. |
| Uygulamanın tekrar tekrar başlaması | Sık Pod Yeniden Başlatmaları | Tek bir dağıtım anından çok tekrarları araştırmak için uygundur. Son 15 dakikalık tahmini artışı kullanır. |
| Servisin çalışmaya hazır olmaması | Pod Hazır Değil | Kaynak kullanımından farklı olarak hazır olma durumunu izler. Tamamlanmış işleri sürekli çalışan servis gibi ele almaz. |
| Bildiğiniz tek bir hata mesajı | Log metin eşleşmesi | Kendi sabit metninizi ve kaynağı seçersiniz. Bir olay türünün beş dakikalık sayımı için uygundur. |
| Çok sayıda genel hata logu | Yüksek Hata Log Oranı | Şablonun tanıdığı kalıpların saniyelik hızını izler. Özel hata kodunuz kalıba uymuyorsa metin eşleşmesini seçin. |
| Kullanıcının gateway üzerinden yavaş yanıt alması | API Gateway p95 gecikme | Gateway istek gecikmesini izler. CPU yüksekliğini gecikmenin tek açıklaması olarak varsaymaz. |
| Gateway’de sunucu hatalarının artması | API Gateway 5xx hata oranı | Toplam trafiğe göre yüzdeyi izler; çok düşük trafikte koruyucu trafik koşulu vardır. |
| Node veya kalıcı disk kapasitesi | Node disk / PVC şablonu | Kendi cluster kapsamı ve ilgili kapasite ölçümleri gerekir; uygulama log boyutunu ölçmez. |

Bir servis için bütün şablonları açmak yerine, ekibin hangi durumda ne yapacağını bildiği birkaç koşulla başlayın. Her kurala bakacak kişiyi, hedef kanalı ve ilk kontrolü belirleyin. Örneğin bağlantı hatası mesajı için ilk kontrol servis logları ve veritabanı erişimidir; yalnız şiddeti yükseltmek bağlantıyı düzeltmez.

## Başlangıç değerini nasıl seçmeliyim?

Şablon varsayılanları başlangıç noktasıdır; her uygulama için önerilen evrensel sınırlar değildir. Aşağıdaki sayısal örnekler varsayımsaldır.

**CPU:** Normal yükte limitin `%35–60`’ı kullanılıyor ve kısa dağıtım sıçramaları görülüyorsa, varsayılan `%80 / 5m` süren baskıyı ayırmak için değerlendirilebilir. Normal yük zaten `%85` ise eşiği hemen artırmak yerine kapasiteyi ve limitleri araştırın. Metrik şablonundaki süre ile ölçümün kendi hesaplama penceresi farklıdır.

**Bellek:** Varsayılan `%85 / 10m`, limite yaklaşan sürekli kullanımı izler. Hızla bellek tüketip sonlanan bir uygulama için tek başına yeterli erken uyarı olmayabilir; OOM sonlanma şablonu farklı bir olguyu izler. Birini diğerinin eşdeğeri saymayın.

**Yeniden başlatma:** Servis varsayılanı, son 15 dakikadaki tahmini artışın `3` veya daha fazla olması ve koşulun `2m` sürmesidir. “Dakikada üç kez” anlamına gelmez. Bakım/dağıtım dönemlerini normal çalışma dönemiyle karşılaştırın.

**Log metni:** Nadir ve işlem gerektiren bir ifade için `> 0 / 1m` başlangıcını test edebilirsiniz. Zararsız tekil hatalar bekleniyorsa `> 10 / 1m`, son beş dakikada en az 11 satır ister. Aynı `10` değeri saniyelik hız şablonunda çok farklı yoğunluk demektir.

Ayarlama döngüsü: temsil edici bir normal dönem seçin → ölçümün mevcut olduğunu doğrulayın → tek eşiği veya süreyi değiştirin → kaydetme/yayını kontrol edin → benzer bir dönemde gerçek olayları karşılaştırın. Beklenen fayda görülmezse önceki değere dönün. [Kurallar rehberindeki örnek](alerts-rules.md) bu değişikliğin ekrandaki adımlarını gösterir.

## Servis metrik şablonları

| Şablon | İzlediği koşul | Başlangıç süresi |
| --- | --- | --- |
| Yüksek CPU Kullanımı (Servis) | CPU kullanımı, tanımlı pozitif CPU limitinin eşik yüzdesinden büyük. | 5 dakika |
| Yüksek Bellek Kullanımı (Servis) | Bellek kullanımı, tanımlı pozitif bellek limitinin eşik yüzdesinden büyük. | 10 dakika |
| Sık Pod Yeniden Başlatmaları (Servis) | Son 15 dakikadaki tahmini yeniden başlatma sayısı eşiğe eşit veya büyük. | 2 dakika |
| Pod Hazır Değil (Servis) | Çalışması beklenen pod hazır değil; başarılı veya başarısız olarak tamamlanmış pod’lar bu koşula dahil edilmez. | 5 dakika |
| Bellek Yetersizliğinden Sonlanan Container (Servis) | Son 10 dakika içinde bellek yetersizliğinden sonlandırılma kaydı var. | 1 dakika |
| Pod Zamanlanamıyor (Servis) | İş yükü çalışacağı yere yerleştirilemiyor. | 5 dakika |
| Container İmajı İndirilemiyor (Servis) | Çalıştırılacak imaj indirilemiyor. | 5 dakika |
| CrashLoopBackOff (Servis) | Container tekrar tekrar çöküp yeniden başlatılmayı bekliyor. | 3 dakika |
| Container Bellek Baskısı (Servis) | Pozitif bellek limitinin %80’inden fazla kullanım; eşik bu şablonda sabit. | 5 dakika |
| Rollout Kullanılabilir Replica Üretemiyor (Servis) | Uygun Rollout iş yükünde kullanılamayan replika var. | 10 dakika |

Başlangıç CPU eşiği `%80`, bellek eşiği `%85`, yeniden başlatma eşiği `3`’tür. Şablon formu hangi parametrenin değiştirilebilir olduğunu ve sınırlarını gösterir. CPU/bellek yüzdesi fiziksel makinenin tamamına göre değil, ilgili iş yükünün limitine göredir.

**Eksik veri sıfır değildir.** CPU veya bellek limiti yoksa bu yüzde şablonları için sonuç oluşmayabilir. İlgili metrik toplanmıyorsa kuraldan olay çıkmaması sağlıklı olduğu anlamına gelmez. Kullanılabilir veriyi servis ekranında kontrol edin.

## Kendi cluster’ınız için şablonlar

Cluster kapsamı, hesabınıza ait ve seçim listesinde sunulan cluster’lar içindir. CPU kullanımı, bellek kullanımı, sık yeniden başlatma ve hazır olmayan pod şablonlarının cluster sürümleri bulunur. Ayrıca:

| Şablon | Gerekli kaynak/veri |
| --- | --- |
| Deployment Replica Sayısı Uyuşmuyor (Cluster) | İstenen ve hazır Deployment replika sayıları; başlangıç süresi 10 dakika. |
| Node Hazır Değil | Node hazır olma durumu; 5 dakika. |
| Yüksek Node CPU / Bellek Kullanımı | Node kaynak metrikleri; 10 dakika. |
| Yüksek Disk Kullanımı (Node) | Node’un kök dosya sistemi kullanım verisi; 10 dakika. |
| PVC Neredeyse Dolu | Kalıcı diskin kullanılan ve toplam kapasite verisi; 10 dakika. |

Deployment ve Rollout farklı iş yükleridir. Servise özgü **Rollout Kullanılabilir Replica Üretemiyor** şablonu yalnız Rollout sahipliği doğrulandığında sunulur; her servis veya zamanlanmış iş için uygun değildir. Tamamlanması beklenen Job/CronJob işlerini sürekli çalışan servis gibi değerlendirmeyin.

## API Gateway şablonları

Yönetilen **API Gateway** servisi seçildiğinde p95 gecikme, 5xx hata oranı, 429 yanıt hızı ve yanıt önbelleğinin dolması şablonları kullanılabilir. Bunlar sıradan uygulama servislerine sunulmaz; ilgili gateway metriklerinin toplanması gerekir. İlk üç şablonun başlangıç süresi beş dakika, önbellek doluluğununki 15 dakikadır. Gecikme, yüzde ve saniyelik hız birimlerini formdaki açıklamaya göre okuyun.

### Gateway eşiklerini doğru birimle okuyun

| Koşul | Varsayılan eşik / süre | Doğru yorum |
| --- | --- | --- |
| p95 gecikme | `2000` milisaniye / `5m` | Beş dakikalık hızlardan hesaplanan gecikme dağılımının p95 değeri 2 saniyeyi aşar. Bütün isteklerin 2 saniyeyi aştığı anlamına gelmez. |
| 5xx hata oranı | `%5` / `5m` | Oran eşiği aşmalı ve toplam istek hızı `0,1 istek/s` üzerinde olmalı. Az trafikte tek hataya rağmen olay çıkmayabilir. |
| 429 yanıtları | `1 istek/s` / `5m` | 429 yanıtlarının saniyelik hızı eşiği aşar; yüzde değildir. |
| Yanıt önbelleği doluluğu | `%5` ayrılmamış alan / `15m` | Önbelleğin henüz ayrılmamış alanı bu oranın altına iner. Sıcak önbellekte düşük kalabilir; cache-miss veya eviction oranını ölçmez. |

p95 için formda `2` yazmak iki saniye değil **iki milisaniye** seçer. Önbellek uyarısını değerlendirirken yalnız bu değerden performans kaybı sonucu çıkarmayın; kullanım senaryosunu ve diğer gateway ölçümlerini birlikte inceleyin.

## Log şablonları

Log kuralları bir servise bağlıdır. Hazır şablonlar aşağıdaki metin kalıplarını uygulama loglarında arar; uygulamanızın aynı anlamı farklı metinle yazması eşleşme oluşturmayabilir.

| Şablon | Ölçüm | Başlangıç süresi |
| --- | --- | --- |
| Yüksek Hata Log Oranı | Son 5 dakika üzerinden error/fail kalıplarının saniyelik hızı. | 5 dakika |
| Veritabanı Bağlantı Hataları | Son 5 dakikadaki bağlantı hatası kalıplarının sayısı. | 1 dakika |
| OOM Killer Tespit Edildi | Son 5 dakikada bellek tükenmesi kalıbı var mı? | 1 dakika |
| Yavaş Veritabanı Sorguları | Son 5 dakikadaki yavaş sorgu kalıplarının sayısı. | 1 dakika |
| Yüksek Kimlik Doğrulama Hatası Oranı | Authentication/unauthorized/forbidden veya ilgili HTTP durum kalıplarının saniyelik hızı. | 5 dakika |
| Kritik İstisnalar | Fatal/critical/panic ve desteklenen exception kalıplarının saniyelik hızı. | 2 dakika |
| API Hız Sınırı Aşıldı | Rate-limit ve ilgili 429 kalıplarının saniyelik hızı. | 5 dakika |

Bu şablonlar varsayılan uygulama log seçicilerini kullanır; telemetri günlükleriniz için otomatik olarak aynı kapsamı varsaymayın. Telemetri kaynağını açıkça seçmek için [log metin eşleşmesi](alerts-quick-start.md) akışını kullanın. Bu akış sabit beş dakikalık **sayım** yapar; “saniyedeki hata sayısı” şablonuyla aynı ölçüm değildir.

Bir serviste en fazla **20 etkin log kuralı** bulunabilir. Sınır bütün kullanıcıların aynı servis için etkin kurallarını kapsar. Kapalı kurallar bu sayıya dahil değildir; yeniden etkinleştirme sırasında sınır tekrar kontrol edilir.

Aynı kapsamda aynı şablonun eşdeğer etkin kuralı zaten varsa ikinci etkin kural reddedilebilir. Yalnız adı değiştirmek bu kontrolü aşmaz; mevcut kuralı inceleyin. Bu kontrol bütün farklı özel sorgular için genel bir tekrar önleme garantisi değildir.

## Karşılaştırmayı ve veri penceresini okuyun

| Karşılaştırma | Örnek |
| --- | --- |
| `>` | Eşik 10 ise 10 yeterli değildir; değer 10’u aşmalıdır. |
| `>=` | Eşik 3 ise 3 de koşulu sağlar. Yeniden başlatma şablonu bu karşılaştırmayı kullanır. |
| `== 0` | Hazır olma gibi bir durum sıfırdır. Veri hiç yoksa bunun sıfır olduğu sonucuna varılmaz. |

Şablonlar karşılaştırmayı tanımlar; her şablonda ayrı bir karşılaştırma seçici yoktur. Gelişmiş metrik sorgularında PromQL, log sorgularında sayısal sonuç üreten LogQL kullanılır. Loki, log kuralını değerlendiren altyapıdır; sadece log satırı döndüren bir sorgu sayısal uyarı koşulu yerine geçmez. İlk kurulumda sorgu yazmanız gerekmez.

Eşik, ölçüm penceresi ve süreyi birlikte seçin. Örneğin eşik `%80` ve süre `5m` ise tek yüksek örnek yeterli değildir. Sorgu doğrulamasının başarılı olması da verinin bulunduğunu veya uyarının gerçekten tetiklendiğini kanıtlamaz.

Sonraki adım: [İlk uyarını oluştur](alerts-quick-start.md) · [Eksik seçenekleri araştır](alerts-troubleshooting.md)
