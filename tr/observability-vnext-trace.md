# Observability: Trace Ekranı

Trace ekranı, tekil bir isteğin servis(ler) içindeki yolculuğunu span'ler (adımlar) halinde gösterir. Arama listesinden bir trace seçtiğinizde, detayı aynı sayfada altta açılır — ayrı bir sayfaya gitmenize gerek yoktur.

---

## Trace arama

Son bir saatlik pencere içindeki trace'ler, varsayılan olarak en yeniden eskiye sıralı listelenir. Liste imleçli sayfalama kullanır; "Sonraki sayfayı yükle" ile daha fazla sonuç getirebilirsiniz.

Üstte üç bilgi görünür:

- **Anlık görüntü sonu (snapshot through)** — aramanın hangi ana kadar sınırlandırıldığı
- **Kaynak watermark** — kaynağın güvenilir biçimde tamamlandığı son an
- **Sorgu kimliği** — bir sorunu destek ekibiyle paylaşırken kullanabileceğiniz kimlik

Tablodaki her satır şunları gösterir:

| Sütun | Anlamı |
|---|---|
| **İşlem** | Kök span'in adı ve trace kimliği |
| **Durum** | OK / Hata / Belirsiz / Bilinmiyor |
| **Süre** | Milisaniye cinsinden toplam süre |
| **Trace şekli** | Span sayısı ve trace'e katılan servis sayısı |
| **Kanıt** | Örneklenmiş mi, hangi şema ve sorgu ile üretildiği |

### Kanıt rozetleri

| Rozet | Anlamı |
|---|---|
| **Doğrulanmış trace'ler** | Sonuç eksiksiz ve kesin |
| **Doğrulanmış boş** | Sorgu eksiksiz tamamlandı ve gerçekten hiç trace yok |
| **Örneklenmiş veya kısmi** | Sonuç var ama tam değil |
| **Gecikmeli** | Kaynak veri gönderiyor ama en güncel dakikalar henüz ulaşmadı |
| **Kullanılamıyor** | Sonuç doğrulanamadı |
| **Uyumsuz** | Kaynağın şeması güvenle yorumlanamıyor |

> **Not:** Trace toplama sırasında örnekleme uygulanabilir. Bu yüzden listedeki trace sayısını, o servisin aldığı gerçek istek sayısıyla bir tutmayın — [Genel Bakış](./observability-vnext-overview-screen.md) ekranındaki istek/hata sayıları için ayrı, örnekleme dışı bir özet kullanılır.

---

## Trace detayı: span şelalesi

Bir trace seçildiğinde, span'ler göreli bir zaman çizelgesinde (waterfall) listelenir. Girinti seviyesi span'in derinliğini gösterir; turuncu rota işareti, toplam süreyi belirleyen **kritik yol** üzerindeki span'leri işaretler.

Üstte altı özet bilgi bulunur: governed span sayısı, en yüksek derinlik, verinin hangi ana kadar kesinleştiği ("committed through"), trace'e katılan servis sayısı, kritik yoldaki toplam çalışma süresi ve saat uyuşmazlığı (clock skew) tespit edilen span sayısı.

Bir span'de eksik veya kısmi kanıt varsa, ya da trace kesilmiş/bozuk bir ağaç yapısındaysa ekranda ayrı bir uyarı görünür: *"Örnekleme, kaynak tazeliği ve ağaç kalitesi korunur. Eksik veya kısmi span'ler asla eksiksiz bir trace gibi gösterilmez."*

### Span detayı

Bir span'e tıkladığınızda sağ panelde şunları görürsünüz:

- Servis adı, span'in kendi çalışma süresi (self time), span kimliği, üst span kimliği (kök ise "Kök" yazar)
- Kanıt kaynağı, varsa HTTP metodu ve rotası
- **Governed öznitelikler**, **Kaynak öznitelikleri** ve **Span olayları** — bunlardan biri boşsa "döndürülmedi" olarak belirtilir
- Politika veya kaynak sınırları nedeniyle gösterilmeyen değer sayısı ("N değer politika veya kaynak sınırları nedeniyle gizlendi")

> **Uyarı:** Span öznitelikleri ve olayları boyut ve güvenlik politikalarıyla sınırlandırılır; bazı değerler bu yüzden hiç gösterilmeyebilir. Bu bir hata değildir.

---

## İlişkili loglar

Trace detayında **"İlişkili Loki logları"** butonuna bastığınızda, bu trace'in zaman penceresi ve katılan servisleriyle eşleşen log satırları getirilir. Her satırda zaman damgası, seviye, servis adı, mesaj ve namespace/pod/container bilgisi görünür.

Kanıt rozeti burada da aynı mantıkla çalışır: **"Kesin loglar"** eksiksiz bir sonucu, **"Nitelikli loglar"** bazı servislerin sorguya katılamadığını veya sonuç limitine takıldığını gösterir. Sonuç boşsa ve kanıt eksiksizse bu, "bu pencerede eşleşen log satırı yok" demektir — sorgunun başarısız olduğu anlamına gelmez.

---

## Benzer trace'lerle karşılaştırma

Kök span'de bir HTTP rotası ve metodu varsa, **"Benzer trace'lerle karşılaştır"** butonuyla aynı işlem üzerindeki en yavaş 10 trace ile karşılaştırma yapabilirsiniz. Ekran, incelediğiniz trace'in bu sınırlı küme içindeki sırasını gösterir (ör. "10 trace içinde 3. sırada").

> **Not:** Bu karşılaştırma yalnızca aynı anlık görüntü penceresindeki sınırlı bir küme üzerinden yapılır — tüm trafiğe göre bir yüzdelik dilim değildir. Kök span'de rota veya metot bilgisi yoksa bu karşılaştırma hiç sunulmaz, çünkü adil bir kıyaslama için ölçek belirlenemez.

---

## Sık sorulan sorular

**Trace sayısı, servisin aldığı gerçek istek sayısından az görünüyor, sorun mu var?**
Hayır. Trace toplama örnekleme uygulayabilir; toplam trafik için [Genel Bakış](./observability-vnext-overview-screen.md) ekranındaki istek/hata sayılarını kullanın.

**Bir span'de bazı öznitelikler eksik, veri mi kayboldu?**
Muhtemelen değil — öznitelik ve olay sayısı güvenlik politikaları ve kaynak sınırlarıyla kısıtlanır. Panelde kaç değerin bu sebeple gösterilmediği ayrıca belirtilir.

**"Benzer trace'lerle karşılaştır" butonu çıkmıyor, neden?**
Bu trace'in kök span'inde bir HTTP rotası veya metodu yoksa adil bir karşılaştırma kümesi oluşturulamaz, bu yüzden buton gösterilmez.

---

## İlgili

- [Observability: Menü ve Ekranlara Genel Bakış](./observability-vnext-guide.md)
- [Genel Bakış ekranı](./observability-vnext-overview-screen.md)
- [İncele ekranı](./observability-vnext-investigate.md)
