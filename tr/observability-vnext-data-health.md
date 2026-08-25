# Observability: Veri Sağlığı Ekranı

Veri Sağlığı, "servis sağlıklı mı?" sorusunu değil, **"Observability ekranlarındaki veriye şu an güvenebilir miyim?"** sorusunu yanıtlar. Diğer tüm ekranlardaki (Genel Bakış, İncele, Trace) sayı ve grafiklerin arkasındaki veri toplama hattının durumunu gösterir.

Ekran her 60 saniyede bir otomatik olarak yenilenir.

---

## Genel durum

Servisin verisini toplayan her hücre (cell) için ayrı bir kart gösterilir. Kartın üstünde ağ geçidi, protokol ve şema sürümü gibi izlenebilirlik bilgileri; sağ üstte ise genel durum rozeti bulunur:

| Rozet | Anlamı |
|---|---|
| **Sağlıklı** | Kaynak tüm kontrollerden geçti |
| **Gecikmeli** | Kaynak veri gönderiyor ama en güncel dakikalar henüz ulaşmadı |
| **Reddedildi** | Gelen verinin bir kısmı kabul edilmedi |
| **Uyumsuz** | Kaynağın şeması veya politikası güvenle yorumlanamıyor |
| **Kullanılamıyor** | Kaynağın durumu doğrulanamadı |
| **Bilinmiyor** | Durum bilgisi okunamadı |

Kartın üstünde ayrıca ekranın tamamı için bir kanıt rozeti bulunur: **Doğrulanmış veri**, **Doğrulanmış boş**, **Gecikmeli**, **Kısmi kanıt**, **Kullanılamıyor** veya **Uyumsuz**. Bu, diğer tüm Observability ekranlarıyla ortak olan aynı kanıt sözlüğünü kullanır — bkz. [Observability: Menü ve Ekranlara Genel Bakış](./observability-vnext-guide.md).

> **Not:** "Doğrulanmış boş" rozeti, bu kiracı için hiç bir hücre kaydı bulunmadığı ve bunun eksiksiz biçimde doğrulandığı anlamına gelir — bu bir sağlık sinyali değil, doğrulanmış bir "hiçbir şey yok" sonucudur.

---

## Hazırlık göstergeleri

Her hücre için dört ayrı hazırlık durumu **Hazır** / **Hazır değil** olarak gösterilir:

- **Sinyal deposu** — verinin saklandığı depo hazır mı
- **Alım (Ingest)** — alım hattı hazır mı
- **Sinyal kabul ediyor** — kaynak şu anda yeni veri kabul ediyor mu
- **Sorgu yolu** — bu hücreye sorgu gönderilebiliyor mu

Dördü de hazır değilse, o hücreden gelen sonuçlara temkinli yaklaşın; ilgili diğer ekranlardaki eksik veya kısmi sonuçların nedeni büyük olasılıkla buradadır.

---

## Kaynak bazlı sayaçlar

Her sinyal türü (ör. trace, log, ağ akışı) için ayrı bir kutu gösterilir:

| Alan | Anlamı |
|---|---|
| **Gözlemlenen** | Kaynağa ulaşan toplam kayıt sayısı |
| **Kabul edilen** | Başarıyla işlenen kayıt sayısı |
| **Reddedilen** | Politika veya doğrulama nedeniyle reddedilen kayıt sayısı |
| **Kayıp** | İşlenemeden kaybolan kayıt sayısı |

Kutunun sağ üstünde **"Uzlaştırıldı"** veya **"Hesabı tutmuyor"** rozeti bulunur.

> **Neden önemli?** "Hesabı tutmuyor" rozeti, gözlemlenen kayıtların bir kısmının ne kabul edilen, ne reddedilen, ne de kayıp olarak sayılmadığı anlamına gelir — yani sayaçlar birbirini tam olarak açıklamıyor demektir. Bu durumda o kaynaktan gelen sonuçlara daha temkinli yaklaşın; "Uzlaştırıldı" ise tüm gözlemlenen kayıtların nereye gittiği tam olarak hesaba katılmış demektir.

Bir hücrenin gerekçe kodları varsa (ör. bir eşiğin aşıldığını gösteren kısa bir kod), kartın altında ayrı rozetler olarak listelenir. Bunlar teknik görünümlü kısa kodlardır; bir sorunu destek ekibiyle paylaşırken referans olarak kullanabilirsiniz.

---

## İlgili

- [Observability: Menü ve Ekranlara Genel Bakış](./observability-vnext-guide.md)
- [Genel Bakış ekranı](./observability-vnext-overview-screen.md)
- [İncele ekranı](./observability-vnext-investigate.md)
- [Trace ekranı](./observability-vnext-trace.md)
