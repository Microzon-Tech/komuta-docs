# Kurallar ve Yayın Durumu

**Uyarılar → Kurallar** hesap kapsamındaki erişilebilir kuralları bir araya getirir. Servis detayındaki **Uyarılar** aynı işlemleri seçili servis bağlamında sunar.

## Kuralı bul ve incele

Arama ve mevcut kapsam, şiddet, etkinlik ve yayın filtrelerini kullanın. Listeyi daralttıysanız beklediğiniz kuralı görememeniz silindiği anlamına gelmez. Kuralın adı, servis/kapsamı, süresi ve etkinlik anahtarı ile yayın rozeti birlikte okunmalıdır.

Şiddet seviyeleri **Bilgi, Uyarı, Kritik ve Acil** şeklindedir. Şiddet, ekibinize öncelik verir ve kanalın şiddet filtresiyle ilişkilidir; otomatik müdahale veya eskalasyon taahhüdü değildir.

## Form alanlarını nasıl doldurmalıyım?

| Alan | İyi bir seçim | Formdaki sınır veya davranış |
| --- | --- | --- |
| Ad | `Orders API — bağlantı hatası` gibi servis ve koşulu anlatan ad | Log metin eşleşmesi ve düzenleme formunda zorunlu, en çok 128 karakter. |
| Özet | `Son 5 dakikada veritabanı bağlantı hatası görüldü; servis loglarını inceleyin.` | Aynı formlarda zorunlu, en çok 256 karakter. Parola veya müşteri verisi yazmayın. |
| Eşleşme metni | `connection refused` gibi sabit bir bölüm | Log metin formunda en çok 512 karakter; büyük/küçük harfe duyarlı düz metin. |
| Eşik | Log sayımında `0` ile en az bir eşleşme aramak | Metin formunda negatif olmayan tam sayı. Şablonlarda birim ve sınır şablona bağlıdır. |
| Süre | `30s`, `5m`, `1h` | Bir sayı ve birim; `60` veya `1m30s` yerine `60s` veya `90s`. `0s` beklemeyi kaldırır. |
| Bildirim aralığı | Süren bir durum için örneğin `15m` | Değer veriliyorsa en az 5 dakika. Boş bırakmak bildirimleri kapatma yöntemi değildir. |
| Şiddet | Ekipte belirlenmiş önem derecesi | Kanal şiddet filtresinin seçiminizi kabul ettiğini kontrol edin. |
| Kanallar | O kuralı takip eden ekibin aktif kanalı | Boş seçim uygun aktif kanallara yönlenir. Tek hedef istiyorsanız onu seçin. |
| Gelişmiş sorgu | Şablon yeterli değilse, izin verilen kapsamda sayısal koşul | Düzenleme formunda en çok 2.000 karakter. Sorgu kilitliyse değiştirmeye çalışmayın. |

Şablon oluşturma ekranındaki parametreler ile mevcut kuralın düzenleme alanları aynı olmak zorunda değildir. Örneğin paylaşılan altyapıdaki metrik sorgusunu düzenleme ekranından değiştiremezsiniz. Farklı eşik gerekiyorsa uygun şablonla yeni tanımı hazırlayın; yeni ve eski kuralların aynı anda etkin kalmasını bilinçli yönetin.

## Örnek: kısa CPU sıçramalarının bildirimini azaltın

Normal iş yükünü incelediniz ve beş dakikayı aşan, fakat on dakikaya ulaşmadan biten CPU yükselişlerinin müdahale gerektirmediğine karar verdiniz. Bu, varsayımsal bir ayarlama örneğidir; gerçek kapasite sorununu süreyi uzatarak gizlemeyin.

1. **Kurallar** içinde doğru servisin CPU kuralını bulun. Mevcut `%80` eşiğini, `5m` süreyi, şiddeti ve kanalları not edin.
2. **Düzenle** ile yalnız süreyi `10m` yapın. Böylece aynı karşılaştırmanın daha uzun devam etmesini istersiniz; CPU limitini veya veri penceresini değiştirmezsiniz.
3. Kaydedin, formu yeniden açarak `10m` değerinin saklandığını doğrulayın. Ardından yayın rozetinin kontrol zamanını ve varsa işlem sonucunu inceleyin.
4. Otomatik yayınlanan kuralda yayın akışını izleyin. Doğrudan yayın işlemi sunuluyorsa o akışı kullanın. **Tanım farklı** sürüyorsa yeni sürenin kurulu olduğunu varsaymayın.
5. Sonraki benzer yük döneminde servis ölçümlerini ve **Geçmiş** kayıtlarını karşılaştırın. Beklenen etki, kısa dalgalanmaların daha az olay üretmesidir; mevcut açık olayın anında çözülmesi değildir.
6. Sonuç uygun değilse önceki `5m` değerine dönün ve kaydetme/yayın kontrolünü tekrarlayın.

Bir ayarı denemek için **Çoğalt** kullanıyorsanız kopyanın kapalı başladığını unutmayın. İki kuralı açmak, tek kuralın iki sürümünü karşılaştıran özel bir deneme modu oluşturmaz; ikisi de bildirim üretebilir.

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
