# Şablonlar, Metrikler ve Loglar

**Uyarılar → Şablonlar** sayfasında arama, kategori ve kapsam filtreleriyle başlayın. **Kullan** seçeneği, hedefi seçip parametreleri inceleyeceğiniz sihirbazı açar. Görünen seçenekler erişilebilir kaynaklarınıza ve iş yükünün uygunluğuna göre değişir.

Kendi cluster’ınızda metrik değerlendirmesi için Prometheus, log değerlendirmesi için Loki ve bildirim/susturma akışı için Alertmanager bileşenlerinin uygun biçimde kurulmuş ve çalışır olması gerekir. Eksik bileşen hatasını cluster yöneticinizle giderin. Komuta’da yalnız servis kullanan müşterinin bu bileşenlere ait erişim adreslerini yapılandırması beklenmez.

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
