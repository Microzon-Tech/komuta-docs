# Sessizlikler: Planlı Susturma

**Uyarılar → Sessizlikler**, seçtiğiniz kuralların bildirimlerini bir zaman penceresinde susturmak içindir. İzlemeyi kapatma, sorunu giderme veya geçmiş kaydını çözme işlemi değildir. Servis detayındaki Uyarılar sayfasından da servis bağlamında susturma oluşturabilirsiniz.

## Bir bakım penceresi oluşturun

1. **Sessizlik Oluştur** seçeneğini açın; örneğin **Planlı bakım** adını verin.
2. Yalnız bakımın etkileyeceği **uyarı kurallarını** seçin. Servis adını ve kuralın kapsamını kontrol edin.
3. **Başlangıç** ve **Bitiş** zamanlarını ayarlayın. Form tarayıcınızın yerel saatini kullanır; farklı saat dilimindeki ekiplerle pencereyi açıkça paylaşın.
4. Bakımın nedenini açıklayan kısa bir yorum ekleyin.
5. Oluşturun ve susturmanın yayın/eşitleme durumunu kontrol edin.

En az bir kural seçilmelidir. Seçili kurallar tek bir cluster kapsamında olmalıdır; farklı cluster’lardaki kurallar için ayrı pencereler oluşturun. Bitiş, başlangıçtan sonra olmalıdır. Önceden doldurulan iki saatlik pencereyi ihtiyacınıza göre kısaltın veya değiştirin.

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
