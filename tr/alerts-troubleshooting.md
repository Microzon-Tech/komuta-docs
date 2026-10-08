# Uyarılarda Sorun Giderme

İncelemeyi eksik olan ilk aşamadan başlatın: **veri → etkin kural → yayın/değerlendirme → olay → bildirim**. Her adımda doğru hesap, servis ve zaman aralığını kullanın. Bir kontrolün sonucunu not ettikten sonra sonraki adıma geçmek, aynı ayarları tekrar tekrar değiştirmeyi önler.

## Hangi yoldan ilerlemeliyim?

| Gördüğünüz belirti | Başlangıç |
| --- | --- |
| Beklenen log veya metrik görünmüyor | Aşağıdaki **1. Kaynak verisi var mı?** adımı |
| Veri var ama Geçmişte olay yok | **2. Kural koşulu gerçekten sağlanıyor mu?** |
| Geçmişte olay var ama mesaj yok | **Olay var, bildirim yok** bölümü |
| Test mesajı da gelmiyor | **Kanal testinin sonucuna göre ilerleyin** bölümü |
| Olay çözülme bekliyor | **Kayıt çözülmüyor** bölümü |
| Kaydetme/yayın işlemi hata veriyor | **Yayın bekliyor veya başarısız** bölümü |
| Form veya seçenek eksik | **Şablon, kaynak veya işlem görünmüyor** bölümü |

## Olay oluşmuyor: üç aşamalı kontrol

### 1. Kaynak verisi var mı?

**Nerede:** İlgili servis → **Loglar** veya metrik ekranı. Zaman aralığını beklenen olay saatini kapsayacak şekilde ayarlayın.

**Log için beklenen:** Kuralda seçilen kaynakta, eşleşme metnini içeren yeni bir satır. [İlk örnekte](alerts-quick-start.md) bu metin `DOCS_ALERT_SAMPLE` olur. Arama filtrelerini temizleyip uygulama çıktısı/telemetri seçimini karşılaştırın. Satır farklı kaynakta görünüyorsa kural doğru akışı izlemiyordur.

**Metrik için beklenen:** İlgili ölçümün güncel değeri ve şablonun ihtiyaç duyduğu bilgi. CPU/bellek yüzdesi için pozitif kaynak limiti gerekir. Grafik yoksa veya limit tanımlı değilse “değer sıfır, kural çalışıyor” sonucuna varmayın.

**Veri yoksa:** Uygulamanın gerçekten kayıt/ölçüm ürettiğini, doğru kaynağı ve erişimi kontrol edin. Kendi cluster’ınızda toplama/değerlendirme bileşeni hatası varsa yöneticinizle giderin. Bu aşama düzelmeden eşiği değiştirmeyin.

**Veri varsa:** Aynı servis ve kaynağı not edip ikinci adıma geçin.

### 2. Kural koşulu gerçekten sağlanıyor mu?

**Nerede:** **Uyarılar → Kurallar**, kural adıyla arama → ayrıntı veya **Düzenle**.

**Beklenen:** Doğru kapsam, etkin kural, amaçladığınız karşılaştırma ve süre. Metin eşleşmesinde harf büyüklüğü dahil sabit metin bulunmalıdır; `.*` gibi ifadeler özel desen değildir. Zaman damgası ve istek numarası değişiyorsa eşleşme metnini sabit kısma daraltın.

**Koşul sağlanmıyorsa:** Tek nedeni düzeltin. `>10` için aynı beş dakikalık pencerede 11 satır gerekir; tam 10 yeterli değildir. `1m` süre, koşulun bir dakika devam etmesini ister. Sayım ile saniyelik hız şablonlarını aynı eşikle karşılaştırmayın. Gateway 5xx şablonunda düşük trafik koruması bulunduğunu [şablon tablosundan](alerts-templates.md) kontrol edin.

**Koşul sağlanıyorsa:** Son değişiklik zamanını not edip üçüncü adıma geçin. Kuralı tekrar oluşturmanız gerekmez.

### 3. Yayın ve Geçmiş ne gösteriyor?

**Nerede:** Aynı kuralın yayın rozeti ve kontrol zamanı; ardından **Uyarılar → Geçmiş**.

**Beklenen:** Desteklenen metrik kurallarında güncel yayın eşleşmesi; ardından koşul ve süre karşılandıysa ilgili olay. **Tanım farklı**, **Kurulu değil** veya işlem hatası varsa aşağıdaki yayın yolunu izleyin. Log ve ajan yönetimli kurallarda **Doğrulanmadı**, tek başına yayın hatası değildir.

Geçmiş dönemini test zamanını içerecek şekilde genişletin ve durum filtresini temizleyin. Satır bulunursa veri/kural incelemesinden **bildirim** incelemesine geçin. Veri, koşul ve desteklenen yayın kontrolü doğru olduğu hâlde olay alınmıyorsa zamanları ve gördüğünüz durumları destek talebine ekleyin. Sabit bir teslim süresi varsayarak arka arkaya yeni kurallar açmayın.

**Test / YAML** doğrulamasının geçmesi bu üç aşamanın tamamlandığını göstermez; gerçek veri veya olay üretmez.

## Olay var, bildirim yok

1. **Geçmiş** satırını açıp kuralı, doğrulanmış kapsamı, olay türünü ve saatini belirleyin. Tetiklenme mesajı ile çözülme mesajını ayrı arayın.
2. **Kurallar → Düzenle** içinde kanal seçimine bakın. Hedef silindiyse seçimi güncelleyin. Boş seçim, uygun aktif kanalları kullanır; tek hedef istiyorsanız açıkça seçin.
3. **Kanallar** listesinde hedefin aktif olduğunu ve varsa şiddet filtresinin kuralı kabul ettiğini doğrulayın. Yanlış hedef veya filtre varsa onu düzeltip sonraki kontrollü olayı izleyin.
4. **Sessizlikler** içinde aynı kuralı kapsayan aktif pencere ve eşitleme durumunu kontrol edin. Bakım penceresi beklenen susturmayı açıklıyorsa kuralı değiştirmeyin. Birden fazla pencere varsa hepsini inceleyin.
5. **Kanallar** teslim kayıtlarında olay saatini kapsayan dönem, kanal ve durum filtreleriyle arayın. Tekrar aralığı ile bildirim sınırlarını da değerlendirin.

| Bulduğunuz sonuç | Sonraki adım |
| --- | --- |
| Başarısız gönderim | Görünen hatayı okuyun; aşağıdaki kanal testini yapın. Geçersiz hedef/erişim sorunu varsa kanal ayarını düzeltin. |
| Bastırılmış gönderim | Görünen kısıtı ve bildirim sınırlarını inceleyin. Tekrar tekrar test göndermek sınırı çözmez. |
| Bekleyen/işlenen gönderim | Henüz tamamlanmış saymayın; daha sonraki durumunu kontrol edin. Süren hata veya takılma için zamanları kaydedin. |
| Başarılı gönderim, hedefte mesaj yok | Doğru alıcı/kanal, posta filtreleri veya Teams workflow çalıştırma sonucunu inceleyin. E-postada başarılı kayıt sağlayıcı kabulüdür. |
| Filtreler temizken kayıt yok | Kural kanalı, aktiflik, şiddet, sessizlik ve tekrar aralığını birlikte kontrol edin. Olay bilgisiyle destek isteyin. |

Platform bildirim abonelik matrisi, uyarı kuralının kanal seçimi değildir. Başka türde bildirim almanız bu kuralın yönlendirmesini doğrulamaz.

## Kanal testinin sonucuna göre ilerleyin

**Nerede:** **Kanallar → Bildirim ayarlarında yönet**, ilgili kanal → **Test gönder**. Test gönderme izni gerekir. Bir testin sonucunu değerlendirmeden art arda yeni testler başlatmayın.

| Sonuç / hedef | Kontrol ve beklenen sonuç |
| --- | --- |
| Test düğmesi yok veya kullanılamıyor | Bildirimleri düzenleme iznini ve izinlerin yüklenme durumunu kontrol ettirin. Yalnız okuma yeterli değildir. |
| Test gönderilmedi | Kanalın aktifliği, gerekli alanları ve görünen kısıtı kontrol edin. Bu sonuç, hedefe yapılmış başarılı bir deneme değildir. |
| E-posta | Adreslerin alıcı etiketleri olarak kaydedildiğini doğrulayın. Gerçek posta kutusu, spam/karantina ve kurumsal filtrelere bakın; çok alıcılı kanalda gerekli her alıcıyı kontrol edin. |
| Slack | Kayıtlı adresin Incoming Webhook olduğundan, doğru hedefe bağlı bulunduğundan ve uygulama erişiminin sürdüğünden emin olun. Başarıyı hedef kanalda mesajla doğrulayın. |
| Teams | Teams kanal bağlantısı yerine Workflows webhook adresi kullanın. Workflow etkinliği, sahipliği, kimlik doğrulama seçeneği ve çalıştırma geçmişini inceleyin; başarılı çalışmada mesaj doğru kanalda görünmelidir. |

**Test ulaşıyorsa ama gerçek olay ulaşmıyorsa:** Yukarıdaki kural yönlendirmesine dönün. **Test de ulaşmıyorsa:** [Kanal kurulumunu](notification-guide.md) düzeltin; daha sonra tek yeni testle sonucu doğrulayın. Kanal testi kuralı tetiklemez ve normal olay geçmişinde aynı tür satır oluşturmasını beklemeyin.

## Kayıt çözülmüyor

**Nerede:** Açık olayın ayrıntısı ve ilgili servisin güncel log/metrik ekranı.

1. Aynı koşulu hâlâ sağlayan yeni veri var mı? Varsa çözülme beklemeyin; temel sorunu araştırın.
2. Yeni eşleşme yoksa son eşleşmenin zamanını ve sorgu penceresini karşılaştırın. Metin örneğinde satırlar beş dakika pencerede kalır; yeniden başlatma şablonunda 15 dakikalık geçmiş kullanılır. Son yeni eşleşme bu pencereyi uzatabilir.
3. Veri koşulu artık sağlamıyorsa sonraki değerlendirme ve çözülme olayının alınmasını izleyin. **Geçmiş**, canlı sorgu sonucunu değil son alınan durumu gösterir.
4. Durum değişmiyorsa son eşleşme, tetiklenme, yaptığınız kontrol ve varsa son kural değişikliği zamanını kaydedip destek isteyin.

Kuralı kapatma/silme veya servisi uyutma, mevcut kaydı çözülmüş olarak doğrulama yöntemi değildir. Çözülme kaydı var ama mesajı yoksa bildirim yolunu bu kez çözülme türü için izleyin.

## Yayın bekliyor veya başarısız

**Nerede:** **Kurallar**, ilgili satırın yayın durumu ve varsa işlem ayrıntısı.

- **Tanım farklı:** Kuralı yeniden açıp kaydedilmiş ayarı kontrol edin. Son değişiklik zamanı ile yayın kontrol zamanını karşılaştırın. Eski tanım hâlâ değerlendirilebilir.
- **Kurulu değil:** Kuralın etkin olması bekleniyor mu? Devre dışıysa yokluğu bununla uyumlu olabilir. Etkinse doğrudan/otomatik yayın yolundaki sonucu inceleyin.
- **Doğrulanmadı:** Önce kural türünü okuyun. Log veya ajan yönetimli kural için bu rozet mevcut gözlem sınırını yansıtabilir. Desteklenen metrik kuralında yükleme hatası ya da eski kontrol zamanı varsa güncel sonucu yeniden kontrol edin.
- **İşlem başarısız:** Görünen hatayı giderin. Doğrudan yayında **Tekrar dene** sunuluyorsa kullanın; otomatik yayınlı kural için manuel düğme aramayın.

Sonuç değişmiyorsa ayar, işlem ve kontrol zamanlarını destek talebine ekleyin. Servisin dağıtımının sağlıklı olması kural yayını yerine kanıt olarak kullanılmaz. [Rozetlerin ayrıntılı anlamları](alerts-rules.md).

## Şablon, kaynak veya işlem görünmüyor

| Nerede / belirti | Yapılacak kontrol | Nasıl devam edilir? |
| --- | --- | --- |
| Kendi cluster listesi boş | Hesabınızda kendi cluster’ınız var mı, liste hatasız yüklendi mi? | Yalnız servisiniz varsa **Uygulama Düzeyi** şablonlarıyla devam edin. Yükleme hatasıysa tekrar deneyin. |
| Özel metrik sorgusu kapalı | Paylaşılan altyapı mı? | Servis şablonunu kullanın; gelişmiş metrik sorgusu uygun kendi cluster kapsamına bağlıdır. |
| API Gateway / Rollout şablonu yok | Seçili servis türü ve doğrulanmış iş yükü uygun mu? | Gateway için yönetilen API Gateway; Rollout için doğrulanmış Rollout sahipliği gerekir. |
| Oluştur/düzenle düğmesi yok | [İzin tablosu](alert-guide.md) ve yönetilen kural durumu | Okuma, oluşturma ve düzenleme ayrı yetkilerdir; otomatik yönetilen kurallar değiştirilemez. |
| Log kuralı oluşturulamıyor veya açılmıyor | Kaynak/kapsam, form hatası, etkin log kuralı sayısı | Servis başına bütün kullanıcıları kapsayan 20 etkin log kuralı sınırını kontrol edin. |
| Aynı şablon yeniden oluşturulamıyor | Eşdeğer etkin kural zaten var mı? | Mevcut kuralı inceleyin; yalnız adını değiştirerek yeniden denemeyin. |

## Susturma oluşturulamıyor veya etkisiz

**Oluşturma hatası:** En az bir geçerli kural, tek cluster kapsamı ve bitişi başlangıçtan sonraki pencere seçin. Silinmiş kuralı seçimden çıkarın. Eşitleme hatası varsa başarılı susturma kaydı oluştuğunu varsaymayın.

**Oluştu ama mesaj geliyor:** Mesajın kuralını seçili kurallarla, olay/gönderim saatini pencereyle ve yerel saat dilimiyle karşılaştırın. Pencere henüz başlamamış veya bitmiş olabilir. Eşitleme durumunu kontrol edin; başka kuralın mesajı bu pencerenin kapsamına girmez. [Bakım örneği](alerts-silences.md) önce/sıra/sonra kontrollerini gösterir.

## Kapsam belirsiz veya çok fazla bildirim var

**Hesap bant genişliği** tek servis belirtmez. **Kapsam doğrulanamadı** durumunda kural adından tahmin yürütmeyin. [Paket kaybı türlerini](alerts-history.md) ayırın; hepsi normal kota sınırlaması değildir.

Çok mesaj için önce **Geçmiş**te farklı olaylar mı, aynı olayın tekrarları mı olduğunu ayırın. Sonra aynı koşulu izleyen mükerrer kuralları, kanallardaki ortak alıcıları, süreyi ve tekrar aralığını inceleyin. Yalnız görüntüde kayıtları katlamak gelecekteki gönderimleri birleştirmez. Geçici bakım gürültüsü için bitişi belli, dar kapsamlı sessizlik kullanın; sürekli sorunun nedenini araştırın.

## Destek talebini sonuçlarla doldurun

[Destek talebinde](support-tickets.md) aşağıdaki örneği kendi gözlemlerinizle doldurun. Örnek değerler temsildir; “var/yok” yazarken hangi ekranda baktığınızı da belirtin.

```text
İşlem: Servis için log metin eşleşmesi kuralı
Beklenen: Bir test satırından sonra olay ve seçili kanalda mesaj
Zaman / saat dilimi: [tarih, saat, saat dilimi]
Kapsam: [ekranda doğrulanan servis / cluster / hesap / doğrulanamadı]
Kural: Etkin; eşik 0; süre 1m; tekrar aralığı 15m
Veri kontrolü: [Loglar ekranında seçili kaynak ve son eşleşme zamanı]
Yayın: [görünen rozet, kontrol zamanı ve varsa hata]
Geçmiş: [olay var/yok, tetiklenme/çözülme zamanı]
Kanal: [tür, test sonucu, teslim kaydının durumu]
Sessizlik: [aynı kuralı kapsayan pencere var/yok]
Son değişiklik ve denenen adımlar: [...]
```

Kişisel ve erişim bilgileri kapatılmış ekran görüntüsü ekleyebilirsiniz. Webhook adresi, parola, token, ham log içindeki kişisel bilgi, altyapı erişim adresi veya özel kaynak kimliklerini paylaşmayın. Gerekiyorsa servis/kural adlarını anonimleştirin; destek ekibinin istediği ek bilgiyi güvenli destek kanalıyla iletin.

[Genel Bakışa dön](alert-guide.md) · [Kontrollü ilk örneği uygula](alerts-quick-start.md)
