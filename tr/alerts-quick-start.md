# İlk Uyarını Oluştur

İlk kural için tek bir servis, anlaşılır bir koşul ve test edilmiş bir bildirim kanalı seçin. Aşağıdaki “Örnek uygulama” adı ve log metni yalnızca örnektir.

## Başlamadan önce

- Doğru hesapta olduğunuzu ve hedef servise erişebildiğinizi kontrol edin.
- Uyarı oluşturma izniniz olsun. Servis seçimi için servisleri, kanal seçimi için bildirimleri okuma izni gerekir.
- Servisin ilgili metrik veya log verisini ürettiğini kontrol edin. Şablon seçmek eksik veriyi oluşturmaz.
- [Kanallar](notification-guide.md) rehberine göre bir kanal hazırlayın. Kanalı boş bırakmak bildirimleri kapatmaz: uygun aktif kanallar kullanılabilir.

## Yol 1: Servis detaylarından bir şablon

1. Servisinizi açın ve **Uyarılar** sayfasına gidin. **Yeni Kural** ile sihirbazı açın.
2. Uygun bir şablon seçin. Örneğin **Yüksek CPU Kullanımı (Servis)**, CPU limiti tanımlı ve metrikleri alınan bir servis için uygundur.
3. Şablonun değiştirilebilir alanlarını gözden geçirin. Örneğin `%80` eşiği, CPU kullanımını tanımlı CPU limitine göre karşılaştırır. Servis kapsamı otomatik doldurulur.
4. Bildirim kanallarını ve tekrar aralığını seçin; özet ekranında adı, koşulu ve hedefi inceleyip oluşturun.
5. Kural listesine dönün. Etkinlik ve [yayın durumunu](alerts-rules.md) kontrol edin. CPU’nun normal olması halinde olay oluşmaması beklenir; gerçek servise yük bindirmek zorunda değilsiniz.

Şablonun bekleme süresi ve şiddeti başlangıç değerleridir. Gerekirse oluşturduktan sonra **Düzenle** üzerinden izin verilen alanları değiştirin. Paylaşılan altyapıdaki metrik kuralının sorgusu korunur.

## Yol 2: Bir log satırından

1. Servisin **Loglar** sayfasında izlemek istediğiniz satırın işlem menüsünü açın; **Bu logdan alarm oluştur** seçeneğini seçin.
2. **Eşleşme metni** alanını inceleyin. Zaman damgası, istek numarası veya kişisel veri yerine, mesajın sabit ve güvenli bir bölümünü kullanın; örneğin `example-operation-failed`.
3. **Eşik** için `0`, **Süre** için `1m` seçerseniz, son beş dakikadaki eşleşme sayısının bir dakika boyunca sıfırdan büyük kalması gerekir. Tek bir kayıt bu koşulu sağlayabilir.
4. Şiddeti, adı, özeti ve kanalları gözden geçirin. Oluşturun; başarı mesajındaki **Alarmlara git** bağlantısıyla kurala dönün.

> **Sayım penceresi ve süre farklıdır.** Bu formda sayım penceresi beş dakikadır. Eşik `10` ise koşul `10’dan fazla`, yani en az 11 eşleşmedir. `0s` bekleme süresini kaldırır; toplama, değerlendirme ve bildirim gecikmelerini ortadan kaldırmaz.

Log satırından açıldığında kaynak satırdan belirlenir. Uygulama çıktı günlükleri ile telemetri günlükleri ayrı akışlardır. Birleşik görünümde bir kayıt görmek, kuralın iki akışı birlikte saydığı anlamına gelmez. Kaynak belirlenemiyorsa doğrulanabilir bir servis veya satır seçin.

## Yol 3: Genel Uyarılar ekranından

1. **Uyarılar → Genel Bakış** veya **Kurallar → Yeni Kural** yolunu açın.
2. **Uygulama Düzeyi** seçip servisinizi bulun. Kendi cluster’ınız varsa cluster kapsamı da sunulur.
3. **Şablon kullan** ile aynı şablon akışına ulaşın. Servis seçtikten sonra **Günlük metni eşleştir** ile log satırı seçmeden aynı metin eşleşmesi formunu da açabilirsiniz.
4. Metin eşleşmesinde günlük kaynağını seçin; şablonda parametreleri doldurun. Kanalları ve son özeti kontrol edip oluşturun.

**Şablonlar → Kullan** da aynı sihirbazı, şablon seçili olarak açar. Şablonun kapsamı sabit kalır; yalnızca ona uygun servis veya cluster seçilebilir. **Manuel (Gelişmiş)** seçeneği ise ayrıntılı sorgu formuna geçer; ilk kurulum için zorunlu değildir.

## Çalıştığını güvenli biçimde doğrulayın

| Kontrol | Ne beklemelisiniz? |
| --- | --- |
| Kaydetme | Kural doğru serviste, amaçladığınız ayarlarla listelenir. |
| Yayın | Metrik kuralı için güncel yayın gözlemini okuyun. Log kurallarında bu doğrulama mevcut olmayabilir; **Doğrulanmadı** tek başına başarısızlık değildir. |
| Koşul | İzlediğiniz veride seçtiğiniz koşulun gerçekten oluştuğunu ve gerekli süreyi sağladığını kontrol edin. |
| Olay | **Geçmiş** içinde ilgili yeni kaydı bulun. |
| Bildirim | **Kanallar** kaydını kontrol edin; sonra mesajı alıcının posta kutusunda veya hedef kanalda doğrulayın. |
| Çözülme | Koşul sona erdikten ve ilgili veri penceresi geçtikten sonra çözülme kaydını kontrol edin. |

Uygun bir test ortamında uygulamanızın güvenli bir test logu üretmesini sağlayabilirsiniz. Gerçek hata yaratmayın, üretim servisini bozmayın; hassas bilgi içermeyen bir metin kullanın. Test için eklenen kuralı işi bitince kapatın veya silin. **Test / YAML** kural doğrulamasıdır; **Test gönder** kanal denemesidir. İkisi de bütün akışın çalıştığını tek başına kanıtlamaz.

Sonraki adım: [Kuralları yönet](alerts-rules.md) · [Sorun gider](alerts-troubleshooting.md)
