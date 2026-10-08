# İlk Uyarını Oluştur

Bu rehberin sonunda tek bir test logunu izleyen kuralınız, açıkça seçilmiş bir bildirim kanalınız ve tetiklenmeden çözülmeye kadar kontrol edebileceğiniz bir akışınız olacak. Sorgu yazmanız veya kendi cluster’ınızın olması gerekmez.

Örnekteki `orders-api-demo` servis adı, `DOCS_ALERT_SAMPLE` metni ve **Docs test** kanal adı temsildir. Kendi test servisinizi ve erişebildiğiniz bir bildirim hedefini kullanın.

## 1. Hazırlığı tamamlayın

Servisin **Loglar** sayfasını açın. Yakın zamanda oluşmuş bir uygulama çıktı günlüğü görebiliyor musunuz? Göremiyorsanız önce zaman aralığını, filtreleri ve günlük kaynağını düzeltin. Henüz ulaşmayan veriye kural eklemek bu sorunu çözmez.

Ardından [Kanallar ve Bildirim Teslimi](notification-guide.md) rehberindeki adımlarla **Docs test** adında bir kanal hazırlayın. **Test gönder** işleminin sonucunu hem Komuta’da hem gerçek hedefte doğrulayın. Bu iki kontrol tamamlanınca kural oluşturmaya geçin.

Uyarı oluşturma izni, servisleri okuma ve hedef servise erişim gerekir. Kanal seçimi için bildirimleri okuma izni de gerekir. Düğme görünmüyorsa [izin tablosuyla](alert-guide.md) rolünüzü kontrol ettirin.

**Hazır olduğunuzun işareti:** Servisin yeni loglarını görebiliyor ve kanal testini gerçek hedefte bulabiliyorsunuz.

## 2. Log metin eşleşmesi formunu açın

**Uyarılar → Kurallar → Yeni Kural** yolunu izleyin. **Uygulama Düzeyi** seçin, test servisinizi bulun ve **Günlük metni eşleştir** seçeneğine geçin. Bu yol, önce bir log satırı seçmeden kuralı hazırlamanızı sağlar.

Servis loglarında uygun bir satır zaten varsa, satırın işlem menüsündeki **Bu logdan alarm oluştur** aynı amaçla kullanılabilir. Bu durumda kaynak ve metin satırdan doldurulur; değişken zaman damgasını ve istek numarasını eşleşme metninden çıkarın.

## 3. Örnek kuralı doldurun

| Alan | Bu örnekte girilecek değer | Neden? |
| --- | --- | --- |
| Günlük kaynağı | Uygulama çıktı günlükleri | Test satırını uygulamanın standart çıktısında/hata çıktısında arayacağız. |
| Ad | `Docs sample alert` | Kurallar ve Geçmiş içinde kolayca arayabilirsiniz. |
| Eşleşme metni | `DOCS_ALERT_SAMPLE` | Yalnız bu denemeye ait, değişmeyen bir ifade. |
| Eşik | `0` | Son beş dakikada en az bir eşleşme yeterlidir; karşılaştırma `> 0` olur. |
| Süre | `1m` | Koşulun bir dakika boyunca devam etmesini isteriz. |
| Bildirim aralığı | `15m` | Aynı durum sürerse tekrar bildirimlerinin aralığıyla ilgilidir. İlk tetiklenmeyi 15 dakika bekletmez. |
| Şiddet | Uyarı | Bu örnek, ekibin gerçek bir acil durum akışına karışmasın. |
| Özet | `DOCS_ALERT_SAMPLE detected in the demo service` | Alıcının neyi araştıracağını açıklar. |
| Bildirim kanalları | **Docs test** | Hedefi bilinçli seçeriz; boş seçim bildirimleri kapatmaz. |

![Örnek verilerle gerçek log uyarısı formu: 1 kaynak, 2 eşleşme metni, 3 eşik ve süre, 4 bildirim hedefi.](https://raw.githubusercontent.com/Microzon-Tech/komuta-docs/main/img/alerts/tr/log-form.png)

*Görsel gerçek Komuta form bileşeninin örnek verilerle yerel gösterimidir; canlı hesapta kural oluşturulmamıştır. Numaralar yukarıdaki alanları bulmanıza yardımcı olur.*

Formdaki koşul önizlemesini okuyun. Seçtiğiniz servis ve kaynak, `5m` penceresi, `0` eşik ve `1m` süre görünüyor olmalı. Sonra oluşturun. Başarı mesajındaki **Alarmlara git** bağlantısını kullanın; kuralın doğru serviste ve etkin olarak listelendiğini kontrol edin.

**Form ilerlemiyorsa:** Boş ad/özet, negatif veya ondalıklı eşik ve birimsiz süreyi düzeltin. Bu formda eşik sıfır veya daha büyük tam sayıdır; `60` yerine `60s` veya `1m` yazın. Servis başına 20 etkin log kuralı sınırı doluysa önce mevcut kuralları gözden geçirin.

## 4. Kontrollü bir eşleşme üretin

Kural kaydedildikten sonra test uygulamanızın mevcut test işlemiyle yalnız bir kez `DOCS_ALERT_SAMPLE` içeren bir log satırı üretin. Aşağıdaki ifadeler, kendi test uygulamanızın uygun kod noktasında kullanılabilecek örneklerdir:

```javascript
console.error('DOCS_ALERT_SAMPLE');
```

```python
print('DOCS_ALERT_SAMPLE', flush=True)
```

Bu kodu kendi bilgisayarınızın terminalinde çalıştırmak Komuta’daki servise log eklemez. Satır, izlediğiniz uygulamadan çıkmalı ve onun **Loglar** ekranında görünmelidir. Üretim hatası yaratmanız gerekmez; uygulamanın normal test yolunu kullanın.

Loglar’da doğru kaynağı seçin ve `DOCS_ALERT_SAMPLE` metnini arayın. Satır görünüyorsa veri girişini doğruladınız. Görünmüyorsa henüz Geçmiş veya bildirim tarafını araştırmayın; [veri adımını](alerts-troubleshooting.md) çözün.

Eşleşme **büyük/küçük harfe duyarlı düz metin içerme** işlemidir. `docs_alert_sample`, seçtiğimiz büyük harfli metinle aynı değildir. Bu alan düzenli ifade alanı değildir; örneğin `.*` yazmak “her şey” anlamına gelmez.

## 5. Pencereyi, süreyi ve bildirimi ayırın

Aşağıdaki saatler yalnız anlatım içindir. Değerlendirme ve iletim sıklığı nedeniyle gerçek kayıtlarda farklı zamanlar görebilirsiniz.

<ol class="docs-alert-timeline" aria-label="Tek log satırının örnek yaşam döngüsü" role="list">
<li><strong>12:00 · Satır oluşur</strong><p>DOCS_ALERT_SAMPLE seçili servis akışına ulaşır. Son beş dakikanın eşleşme sayısı artık sıfırdan büyüktür.</p></li>
<li><strong>12:00:20 · İlk olumlu değerlendirme</strong><p>Bu örnekte koşul ilk kez görülür. Bir dakikalık bekleme bu değerlendirmeden itibaren ilerler.</p></li>
<li><strong>12:01:20 veya sonrası · Tetiklenme</strong><p>Koşul aralıksız devam ettiyse kural tetiklenmeye uygun hâle gelir. Olayın Komuta’ya, bildirimin hedefe ulaşması ayrıca zaman alabilir.</p></li>
<li><strong>12:05 civarı ve sonrası · Çözülme yolu</strong><p>Başka eşleşme yoksa ilk satır pencereden çıkar. Sonraki değerlendirme ve çözülme olayı alındığında Geçmiş güncellenir.</p></li>
</ol>

Tek bir satır beş dakika boyunca pencerede kaldığı için `1m` süresini karşılayabilir. Süre, “her dakika yeni bir satır gelsin” şartı değildir. Eşik `10` olsaydı aynı pencerede en az **11** eşleşme gerekirdi. Her yeni eşleşme koşulun devam etmesine katkı sağlayabilir.

`0s` yalnız bekleme süresini kaldırır. `15m` bildirim aralığı ise tekrar gönderimleriyle ilgilidir. Bunları log sayım penceresi veya kesin teslim zamanı olarak kullanmayın.

## 6. Başarıyı dört yerde kontrol edin

| Nerede? | Beklenen sonuç | Farklıysa sonraki adım |
| --- | --- | --- |
| Servis → Loglar | Test metnini içeren satır, doğru kaynakta görünür. | Kaynak, zaman aralığı ve uygulamanın gerçekten log üretmesini kontrol edin. |
| Uyarılar → Kurallar | `Docs sample alert`, doğru servis ve ayarlarla etkindir. | Kaydetme hatasını veya yanlış kapsamı düzeltin. Log kuralında **Doğrulanmadı** rozeti tek başına hata değildir. |
| Uyarılar → Geçmiş | Aynı kurala ve servise ait yeni olay vardır. | Eşik, süre ve mevcut veri üzerinden [tetiklenmeme yolunu](alerts-troubleshooting.md) izleyin. |
| Uyarılar → Kanallar ve gerçek hedef | İlgili gönderim kaydı ve hedefte mesaj bulunur. | Olay var, mesaj yoksa [bildirim yolunu](alerts-troubleshooting.md) izleyin. |

E-postadaki başarılı gönderim kaydını, alıcının posta kutusunu ayrıca kontrol ederek tamamlayın. Slack veya Teams’te doğru hedef kanala bakın. **Test / YAML**, bu dört kontrolün yerine geçen bir tetikleme düğmesi değildir.

## 7. Çözülmeyi ve temizliği tamamlayın

Yeni test satırı üretmeyi durdurun. Eski satırların beş dakikalık pencereden çıkmasına izin verin. **Geçmiş** içindeki aynı olayda çözülme bilgisini arayın; çözülme bildirimi gönderilmişse **Kanallar** içinde onu da tetiklenme mesajından ayrı kontrol edin.

Koşul artık sağlanmadığı hâlde çözülme kaydı gelmiyorsa [çözülmeme adımlarına](alerts-troubleshooting.md) geçin. İncelemeyi bitirdikten sonra test kuralını kapatın veya silin. Deneme için ayrı bir kanal oluşturduysanız başka kuralın onu kullanmadığını kontrol ederek kaldırın.

**Tamamlanma ölçütü:** Kuralı bulabiliyor, test satırını gösterebiliyor, ilgili olayı ve gerçek hedefteki mesajı eşleştirebiliyor, çözülme durumunu okuyabiliyor ve test ayarlarını temizleyebiliyorsunuz.

## Sonraki senaryo: CPU şablonu

Log örneğini tamamladıktan sonra servisin **Uyarılar → Yeni Kural → Şablon kullan** yolundan **Yüksek CPU Kullanımı (Servis)** seçebilirsiniz. `%80` eşiği ve `5m` süre başlangıç değerleridir; uygulamanızın normal yükünü ve tanımlı CPU limitini inceleyerek seçin. Kanalları belirleyin, özeti kontrol edin ve oluşturun.

CPU normal seyrederken olay oluşmaması beklenir. Veri ve pozitif CPU limiti mevcut olmalı; deneme amacıyla servise yük bindirmek gerekmez. [Şablon seçimi ve ayarlama örnekleri](alerts-templates.md), hangi koşulda hangi kuralın daha anlamlı olduğunu açıklar.

[Kuralları yönet](alerts-rules.md) · [Olay kayıtlarını oku](alerts-history.md) · [Sorun gider](alerts-troubleshooting.md)
