# Observability: İncele Ekranı

İncele, Komuta'nın bir servisteki sorunlar için otomatik olarak oluşturduğu **inceleme** kayıtlarını listeler ve her incelemenin ne olduğunu, olası nedenlerini ve bu nedenleri destekleyen ya da zayıflatan kanıtları tek sayfada gösterir. Bu ekranın amacı bir aday nedeni kesin kök neden gibi sunmak değil, hangi kanıtın var olduğunu, hangisinin eksik olduğunu ve dolayısıyla sonuca ne kadar güvenilebileceğini açıkça göstermektir.

---

## İnceleme listesi

Servise ait tüm incelemeler, en yeni tarihten başlayarak listelenir. Her satırda inceleme başlığı, aday neden sayısı, bulgu sayısı, açılış tarihi ve durum rozeti görünür.

| Durum | Anlamı |
|---|---|
| **Açık** | İnceleme sürüyor |
| **Doğrulanıyor** | Bir çözüm uygulandı, etkisi izleniyor |
| **Çözüldü** | İnceleme kapatıldı |
| **Terk edildi** | İnceleme bırakıldı |
| **Bilinmiyor** | Durum okunamadı |

> **Not:** Liste boşsa bu "servis sorunsuz" anlamına gelmez — sadece kimsenin bu servis için bir inceleme açmadığı anlamına gelir. Listelenen sayı toplam sayıdan azsa (ör. en son 50 kayıt), ekran bunu açıkça belirtir; kalan kayıtlar sessizce gizlenmez.

Bir incelemeye tıkladığınızda detay sayfası açılır.

---

## İnceleme detayı

### Üst bilgi

Başlık, kısa özet, durum rozeti ve dört tarih/kişi bilgisi:

- **İlk görülme** — incelemedeki tüm bulguların en erken başlangıç anı
- **Son görülme** — tüm bulguların en son görülme anı
- **Sahip** — incelemeyi üstlenen kişi, atanmamışsa "Atanmadı" olarak gösterilir
- **Değerlendirme anı** — bu incelemenin en son ne zaman güncellendiği

### İnceleme özeti

Yedi adımlık, her zaman aynı sırada görünen otomatik bir özettir. Sıra sabittir ve adımlar gizlenemez — özellikle beşinci adım (kanıt ve eksikler), diğer altı adıma ne kadar güvenileceğini gösterdiği için bilerek gizlenemez tutulmuştur:

1. **Ne oldu**
2. **Etki**
3. **Ne değişti**
4. **En güçlü aday neden** — henüz bir aday önerilmediyse bu açıkça belirtilir
5. **Destekleyici kanıt ve eksik kanıt** — destekleyen bulgular bir listede, karşı çıkan veya eksik olan kanıtlar ayrı, sarı renkli bir kutuda gösterilir
6. **Şimdi ne yapılmalı**
7. **Düzeldiğini nasıl doğrularız**

> **İpucu:** Beşinci adımdaki "eksik kanıt" kutusunu atlamayın — bir incelemenin ne kadar sağlam olduğunu asıl bu kutu gösterir.

### Aday nedenler

En fazla üç aday neden, sıralı biçimde listelenir. Ekranda kaç aday olduğu ve toplamda kaç aday değerlendirildiği birlikte gösterilir (ör. "3 / 3" ya da daha fazla aday varsa bu da görünür — dördüncü bir aday varsa bu gizlenmez, sadece kartta gösterilmez).

Her aday kartında iki ayrı ölçek bulunur ve bunlar **bilerek birleştirilmez**:

| Ölçek | Ne anlatır |
|---|---|
| **Gözlenen korelasyon** | Aday olayın, incelenen sorunla ne sıklıkla birlikte gözlendiği (Gözlenmedi / Zayıf / Güçlü) |
| **Nedensel durum** | Bu birlikte görülmenin bir "neden" olarak ileri sürülüp sürülemeyeceği (Değerlendirilmedi / Çürütüldü / Kanıtlarla tutarlı / Kanıtlanmış) |

> **Neden ayrı gösteriliyor?** Güçlü bir korelasyon, tek başına neden anlamına gelmez. İki ölçeği tek bir "güven skoru" gibi birleştirmek, güçlü bir korelasyonun kanıtlanmış bir neden gibi okunmasına yol açar — bu ekran bu hatayı bilerek engeller.

Bir aday **"Kanıtlanmış neden"** rozeti taşıyorsa, bu iddia sunucu tarafında doğrulanmış demektir. **"Çürütüldü"** rozeti taşıyan adaylar soluk renkte gösterilir.

Bir adayın karşı kanıtı hiç aranmamışsa kart üzerinde belirgin bir uyarı görünür: *"Hiç kimse bu adaya karşı kanıt aramadı, bu yüzden test edilmemiş sayılır."* Bu uyarı önemlidir çünkü test edilmemiş bir aday, kağıt üzerinde test edilmiş bir aday kadar güçlü görünebilir — bu uyarı ikisini birbirinden ayıran tek işarettir.

Her kartın altında destekleyici ve zayıflatıcı kanıt sayıları ile "atıf edilen kapsam" yüzdesi bulunur. Bu yüzde ölçülmemişse "Ölçülmedi" olarak gösterilir, %0 ile karıştırılmamalıdır.

### Kanıt zinciri

Aday nedenlerin dayandığı her okumayı, hangi adaya ait olduğunu belirterek tablo halinde listeler. Sütunlar: **Rol** (destekleyici / zayıflatıcı / sonuçsuz), **Veri kümesi ve sorgu**, **Kanıt durumu**, **Kapsam** ve **Kaynak watermark**.

> **İpucu:** Sorgu kimliği tabloda tam olarak gösterilir ve seçilebilir haldedir — bir kanıtı destek ekibiyle tartışırken bu kimliği kopyalayıp paylaşabilirsiniz.

Hiçbir aday kanıt göstermiyorsa tablo "henüz bir aday önerilmedi, bu yüzden atıf yapılan bir şey yok" der.

### İncelenmekte olan bulgular

İncelemeye dahil edilen her bulgu; başlığı, güven düzeyi (Doğrulandı / Olası / Varsayım / Bilinmiyor), kanıt kapsamı ve önem derecesi (Bilgi / Uyarı / Yüksek / Kritik) ile listelenir.

---

## İlgili

- [Observability: Menü ve Ekranlara Genel Bakış](./observability-vnext-guide.md)
- [Genel Bakış ekranı](./observability-vnext-overview-screen.md)
- [Trace ekranı](./observability-vnext-trace.md)
