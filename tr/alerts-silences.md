# Sessizlikler: Planlı Susturma

**Uyarılar → Sessizlikler**, seçtiğiniz kuralların bildirimlerini bir zaman penceresinde susturmak içindir. İzlemeyi kapatma, sorunu giderme veya geçmiş kaydını çözme işlemi değildir. Servis detayındaki Uyarılar sayfasından da servis bağlamında susturma oluşturabilirsiniz.

## Bir bakım penceresi oluşturun

1. **Sessizlik Oluştur** seçeneğini açın; örneğin **Planlı bakım** adını verin.
2. Yalnız bakımın etkileyeceği **uyarı kurallarını** seçin. Servis adını ve kuralın kapsamını kontrol edin.
3. **Başlangıç** ve **Bitiş** zamanlarını ayarlayın. Form tarayıcınızın yerel saatini kullanır; farklı saat dilimindeki ekiplerle pencereyi açıkça paylaşın.
4. Bakımın nedenini açıklayan kısa bir yorum ekleyin.
5. Oluşturun ve susturmanın yayın/eşitleme durumunu kontrol edin.

En az bir kural seçilmelidir. Seçili kurallar tek bir cluster kapsamında olmalıdır; farklı cluster’lardaki kurallar için ayrı pencereler oluşturun. Bitiş, başlangıçtan sonra olmalıdır. Önceden doldurulan iki saatlik pencereyi ihtiyacınıza göre kısaltın veya değiştirin.

## Tam örnek: 14:00–14:30 arasında servis bakımı

Bu senaryoda aynı cluster’daki `orders-api-demo` servisi için oluşturduğunuz **CPU yüksek** ve **Pod hazır değil** kurallarının bakım sırasında mesaj göndermesini istemiyorsunuz. Adlar örnektir; yalnız hesabınızda seçilebilir ve bakımın etkilediği kuralları kullanın.

| Form alanı | Örnek seçim |
| --- | --- |
| Ad | `Orders demo — planlı bakım` |
| Kurallar | Doğru servise ait CPU yüksek ve Pod hazır değil |
| Başlangıç | Planladığınız bakım günü, yerel saat 14:00 |
| Bitiş | Aynı gün, yerel saat 14:30 |
| Yorum | `Planlı sürüm geçişi. Bakım sonunda servis sağlığı ve bildirimler kontrol edilecek.` |

**Bakım öncesinde:** Pencereyi oluşturun. Seçili iki kuralı, tarihi, saat dilimini, yaklaşan durumunu ve eşitleme sonucunu kontrol edin. Ekip başka bir saat dilimindeyse saat dilimini de bakım duyurusuna ekleyin. Oluşturma hata verdiyse bakımı susturulmuş kabul etmeyin.

**Bakım sırasında:** Başlangıç geçince pencerenin aktif olduğunu ve beklenen kuralları kapsadığını kontrol edin. Servis metriklerini ve olay geçmişini izlemeye devam edin; sessizlik koşulu düzeltmez. Mesaj gelirse mesajın olay zamanı, kuralı ve gönderim kaydını karşılaştırın: daha önce gönderilmiş bir mesaj, başka kural veya başka kapsam bu pencerenin etkisiz olduğunu tek başına göstermez.

**Bakım bittiğinde:** Servisin sağlığını kontrol edin. Bakım erken bittiyse **Süresi sona ersin** işlemini uygulayıp sonucu doğrulayın; aksi hâlde planlanan bitişi takip edin. Sonraki gerçek olaya ait bildirim yönlendirmesini ve hedefi kontrol edin. Yeni olay yokken hemen mesaj gelmemesi bir hata değildir.

**Bakım uzarsa:** Yeni bitişi kapsayan ayrı bir pencere oluşturup eşitlemesini doğrulayın; gerekli kapsama ulaşmadan mevcut pencereyi erken sonlandırmayın. Örtüşen pencereler varsa yalnız birini sonlandırmak diğerinin etkisini kaldırmaz. Listeyi aynı kurallar açısından gözden geçirip yalnız artık gerekmeyen pencereleri kapatın.

Tamamlandığında kural tanımları yerinde kalmalı, bakım penceresi bitmiş olmalı ve servis sağlığı ayrıca doğrulanmış olmalıdır.

## Durumları yorumlayın

Ekranda devam eden, yaklaşan ve süresi dolan pencereler ayrılır. Bir pencerenin zaman olarak aktif olması, susturmanın hedefe ulaştığını tek başına göstermez. Yayın doğrulanmamışsa bildirimlerin sustuğuna güvenmeyin. Buradaki **Alertmanager**, susturmayı uygulayan bildirim bileşenidir; müşterinin ayrıca bağlantı adresi girmesi beklenmez.

Oluşturma sırasında hedefe eşitleme başarısız olursa işlem hata verir; başarılı bir susturma kaydı oluşmuş gibi devam etmeyin. Kural silinmişse, hedef çözülemiyorsa veya farklı cluster’lar seçildiyse seçimi düzeltin.

**Süresi sona ersin** ile devam eden pencereyi erken sonlandırabilir, **Sil** ile uygun kaydı kaldırabilirsiniz. İşlem sonrasında sonucu kontrol edin. Mevcut arayüzde pencereyi düzenleme formu sunulmaz; farklı bir pencere gerekiyorsa eski pencerenin durumunu gözden geçirip yenisini oluşturun.

## Kapsamı gereksiz genişletmeyin

Bir hesap bant genişliği kuralını seçerseniz, susturma tek bir servisi değil o kuralın hesap kapsamını etkiler. “Tüm kuralları seçip sonra daraltırım” yaklaşımı yerine bakım kapsamındaki az sayıdaki kuralı seçin. Servis seçimi ile kural seçimini birbirine karıştırmayın.

Kural aramasını temizledikten sonra seçili etiketleri tekrar kontrol edin; arama değiştirmek önceki seçimi kaldırmaz. Yalnız okuma yetkisi olan kullanıcı pencereleri görebilir; oluşturma, erken sonlandırma ve silme için uyarıları düzenleme izni gerekir.

## Bildirim yoğunluğunu azaltın

- Aynı servis ve koşul için mevcut kuralı arayın; çoğaltılmış kapalı bir kuralı açmadan önce karşılaştırın.
- Geçici sıçramalar için bekleme süresini artırın. Sürekli eşik aşımlarında önce gerçek nedeni araştırın.
- Bildirim kanallarını açıkça seçin; boş seçimin uygun aktif kanallara yönlenebileceğini unutmayın.
- Tekrar aralığını ekibinizin müdahale ritmine göre ayarlayın; çok kısa aralık daha hızlı tespit garantisi vermez.
- Bakım için bitişi belli, dar kapsamlı susturma kullanın; bakım bitince sonucu kontrol edin.

Kanallar ekranındaki aynı kayıtları katlayan görünüm, kuralları birleştirme veya gelecekteki bildirimleri tekilleştirme ayarı değildir. Bu bölümde otomatik eskalasyon veya kullanıcı tarafından ayarlanabilir olay gruplama akışı sunulmaz.

Sonraki adım: [Kuralları ayarla](alerts-rules.md) · [Susturma sorunlarını incele](alerts-troubleshooting.md)
