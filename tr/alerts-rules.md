# Kurallar ve Yayın Durumu

**Uyarılar → Kurallar** hesap kapsamındaki erişilebilir kuralları bir araya getirir. Servis detayındaki **Uyarılar** aynı işlemleri seçili servis bağlamında sunar.

## Kuralı bul ve incele

Arama ve mevcut kapsam, şiddet, etkinlik ve yayın filtrelerini kullanın. Listeyi daralttıysanız beklediğiniz kuralı görememeniz silindiği anlamına gelmez. Kuralın adı, servis/kapsamı, süresi ve etkinlik anahtarı ile yayın rozeti birlikte okunmalıdır.

Şiddet seviyeleri **Bilgi, Uyarı, Kritik ve Acil** şeklindedir. Şiddet, ekibinize öncelik verir ve kanalın şiddet filtresiyle ilişkilidir; otomatik müdahale veya eskalasyon taahhüdü değildir.

## Yapılabilen işlemler

| İşlem | Etkisi ve dikkat edilecek nokta |
| --- | --- |
| **Yeni Kural** | Şablon, servis log metin eşleşmesi veya uygun kapsamda gelişmiş sorgu ile oluşturur. Etkin kural oluşturulması yayın akışını da başlatır. |
| **Düzenle** | İzin verilen ad, sorgu, süre, şiddet, özet ve bildirim alanlarını değiştirir. Paylaşılan cluster’daki metrik sorgusu düzenlemeye kapalıdır. |
| **Etkinleştir / Devre dışı bırak** | Kuralın etkinliğini ve ilgili yayın tanımını değiştirir. Geçmişteki açık kaydı otomatik olarak “çözüldü” kanıtına dönüştürmez. |
| **Çoğalt** | Ayarları yeni bir kurala kopyalar. Kopya başlangıçta kapalıdır; adı ve kanalları kontrol edip gerektiğinde etkinleştirin. |
| **Test / YAML** | Kural doğrulaması ve varsa yapılandırma önizlemesini gösterir. Gerçek tetiklenme veya bildirim gönderme denemesi değildir. |
| **Yayınla / Yayından kaldır** | Ekranda sunulan, doğrudan yayınlanabilen kurallar için kullanılır. Otomatik yayın akışına bağlı kurallarda bu işlemler sunulmaz. |
| **Sil** | Onay sonrasında kuralı ve ilgili yayın tanımını kaldırma akışını başlatır. Geçmiş kayıtlarını temizleme işlemi değildir. |

Birden fazla kullanıcı kuralı seçerek etkinleştirme, kapatma, uygun kuralları yayınlama/yayından kaldırma, şiddet değiştirme ve silme yapabilirsiniz. Toplu sonuçta başarı ve başarısızlık sayısını kontrol edin; bir kuralın başarıyla değişmesi bütün seçimin başarılı olduğu anlamına gelmez.

## Yönetilen kurallar

Komuta’nın otomatik oluşturduğu kurallar, sizin oluşturduğunuz kurallardan farklıdır. **Kurallar** ekranında bu yönetilen satırlar düzenleme, silme, açma/kapatma, çoğaltma ve toplu değişiklik için sunulmaz. Ayrıntıları ve geçmişi inceleyebilirsiniz. **Test / YAML** seçeneği, bant genişliği ajanının yönettiği kurallarda testin desteklenmediğini bildirebilir.

Yayın akışı Komuta tarafından yönetilen her kural salt okunur değildir: sizin oluşturduğunuz bir servis metrik kuralı otomatik yayınlanırken izin verilen alanları hâlâ düzenleyebilirsiniz. Düğmelerin kullanılabilirliğini esas alın. Bildirimleri geçici olarak azaltmanız gerekiyorsa uygun kurallar için [Sessizlikler](alerts-silences.md) kullanın.

## Yayın isteği ile yayın gözlemi

Kaydetme veya yayınlama yanıtı, işlemin sonucudur. Yayın rozeti ise desteklenen kurallarda son kontrolde kurulu tanımın kayıtlı tanımla karşılaştırılmasını gösterir.

| Rozet | Anlamı | Nasıl ilerlenir? |
| --- | --- | --- |
| **Yayınlandı** | Kurulu tanım, kontrol anında kayıtlı tanımla eşleşmiştir. | Değerlendirme, olay ve teslimi ayrıca kontrol edin. |
| **Tanım farklı** | Kurulu tanım kayıtlı tanımdan farklıdır. Eski tanım değerlendirilmeye devam edebilir. | Son değişiklik ve yayın işlemini inceleyin. |
| **Kurulu değil** | Son başarılı kontrol, kuralı yetkili kaynağında bulamamıştır. | Etkinliği ve yayın yolunu kontrol edin. |
| **Doğrulanmadı** | Güncel yayın için yeterli gözlem yoktur. | Eksik kanıtı başarısız yayınla eşitlemeyin; hata varsa giderin. |

Rozetin kontrol zamanına bakın. Eski veya alınamayan gözlem güncel doğrulama sayılmaz. Mevcut yayın gözlemi log kuralları ve bant genişliği ajanının yönettiği kurallar için kurulum doğrulaması sağlamaz; bu kurallar **Doğrulanmadı** görünebilir.

Bir yayın isteği bekliyor, işleniyor veya başarısız olabilir. Başarısız doğrudan yayın için **Tekrar dene** sunuluyorsa, önce hatayı kontrol edin. Otomatik yayın yolunda manuel yayın düğmesi beklemeyin. Sırf bekliyor diye aynı kuralı yeniden oluşturmak mükerrer uyarıya yol açabilir.

Servisin dağıtımının sağlıklı/eşitlenmiş olması, uyuması veya uyanması tek başına uyarı kuralının kurulu, durmuş ya da değerlendirilmekte olduğunu kanıtlamaz. Kesin yayın veya teslim süresi varsaymayın.

## Süre ve bildirim tekrar aralığı

- **Süre**, koşulun tetiklenmeden önce ne kadar devam etmesi gerektiğidir; örneğin `2m` veya `30s`.
- **Bildirim aralığı**, tekrar bildirimlerinin sıklığıyla ilgilidir. Girilen değer en az beş dakika olmalıdır; tek başına teslim takvimi garantisi değildir.
- Sorgunun **veri penceresi** ayrı bir ayardır. Beş dakikalık geçmişe bakan koşul, yeni hata kesildikten sonra bir süre daha doğru kalabilir.

Kısa dalgalanmalar için süreyi, sürekli gereksiz tekrarlar için eşik ve tekrar aralığını gözden geçirin. [Şablonlar](alerts-templates.md) karşılaştırmanın ve eksik verinin nasıl yorumlanacağını açıklar.

Sonraki adım: [Geçmişi oku](alerts-history.md) · [Yayın sorunlarını çöz](alerts-troubleshooting.md)
