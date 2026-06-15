# Servis Uyku Modu (Service Sleep / Hibernate)

Servis Uyku özelliği, DevOpsZon platformunda çalışan servislerinizin trafiği olmadığı sürelerde otomatik olarak uyku moduna geçmesini sağlayan bir maliyet optimizasyon mekanizmasıdır. Uyku modundaki bir servis **0 pod** çalıştırır — yani CPU ve RAM tüketmez — ama ilk HTTP isteği geldiğinde otomatik olarak ayağa kalkar ve isteği karşılar. Bu akış kullanıcı için neredeyse fark edilmezdir; servisi uyutup uyandırmak için elle müdahaleye gerek yoktur.

---

## Kimin için?

- **Geliştirme ve test ortamları** — hafta sonu ya da mesai sonrası boşta bekleyen staging servisleri.
- **Düşük trafikli projeler** — günün yalnız birkaç saatinde kullanılan dashboard, iç araç, demo uygulamaları.
- **Free plan kullanıcıları** — otomatik uyku varsayılan olarak açıktır; ücretsiz planın sürdürülebilirliğini sağlayan ana mekanizmadır.
- **Ücretli plan kullanıcıları** — dilerseniz planınızı yöneten admin üzerinden aktif edebilirsiniz. Sürekli trafik alan production servisleri için kapalı kalır.

---

## Nasıl çalışır? (Yüksek seviye)

1. **Ölçüm:** Platform, her servisin aldığı HTTP trafiğini arka planda düzenli aralıklarla (varsayılan 15 dakika) ölçer. Ölçüm kaynağı olarak Cilium Hubble'ın HTTP metriklerini kullanırız — bu veriler kendi trafiğinizden gelir, hiçbir zaman dışarı sızmaz.
2. **Eşik kontrolü:** Bir servisin son 3 saat (varsayılan) içerisinde hiç HTTP isteği almamış olduğu tespit edilirse, "uykuya uygun" aday olarak işaretlenir.
3. **Uyku operasyonu:** Servisin pod sayısı 0'a çekilir, HPA (otomatik ölçekleme) ayarları kaydedilir ve servisin giriş trafiği geçici olarak **Activator** adındaki küçük bir ara katmana yönlendirilir.
4. **Otomatik uyandırma:** Uyuyan bir servise bir kullanıcı istek gönderdiğinde:
   - Activator isteği yakalar,
   - Arka planda servisi uyandırır (pod sayısı eski haline getirilir),
   - Pod hazır olana kadar kullanıcının isteğini bekletir,
   - Pod ayağa kalktığında isteği yeni pod'a iletir ve cevabı kullanıcıya geri döner.
5. **Yeni isteklere hazır:** Servis artık normal şekilde çalışır. Trafik bittikten sonra tekrar 3 saat boşta kalırsa uyku döngüsü yeniden başlar.

Bu akışın **en önemli** kısmı şudur: uyandırma işlemi kullanıcıya iletilen isteğin bir parçasıdır, yani uyku moduna giden servisin adresi değişmez, istemcide bir düzenleme yapmanıza gerek kalmaz.

---

## Uyandırma ne kadar sürer? (Cold-start süresi)

Pod imajının boyutuna ve servis başlatma süresine bağlı olarak tipik olarak **2–15 saniye** arası bir gecikme yaşanır. Bu süre:
- Küçük Node.js / Go servislerinde 2–5 saniye,
- .NET / Java servislerinde 5–15 saniye,
- Ağır başlangıç adımları olan (DB migration, cache warm-up gibi) servislerde daha uzun

olabilir. Platform 20 saniyeyi geçen cold-start'ları "SLO ihlali" olarak izler ve dashboard'da uyarı olarak gösterir.

---

## Servisimin uyku durumunu nasıl görürüm?

### Servis Dashboard'u (Overview)

Servis detayının **Dashboard / Overview** sekmesinin üst kısmında kompakt bir **uyku banner'ı** görünür. Banner tek satır yüksekliğindedir ve topoloji görselini aşağı itmez. İçinde:

- Soldaki dairesel **ay (🌙) ikonu** ve "Servis uykuda" başlığı,
- Uyku nedenini açıklayan tek satırlık altyazı,
- **Uyuduğu zaman** bilgisi (örn: "15.04.2026 22:30"),
- Sağda birincil **"Uyandır"** düğmesi,
- Düğmenin altında "Servisim hep ayakta kalsın →" bağlantısı (planınızı yükseltmek için kısayol)

bulunur. Banner yalnızca servisiniz uyurken / uyuyorken / uyanırken görünür; aktif servislerde otomatik olarak kaybolur.

Geçiş durumlarında banner renk değiştirir:
- **Uyku modunda** → amber (altın sarısı) ton
- **Uyanıyor** → mavi ton (saat ikonu döner)
- **Sleep geçişi başarısız** → kırmızı ton (tekrar "Uyandır" denenebilir)

### Servis listeleri — kartlar

Projeler sayfasındaki servis kartlarında uyuyan servisler bir bakışta ayırt edilir:

- Kartın sağ üst köşesinde amber renkli **"UYUYOR"** etiketi (ay ikonu ile),
- Kart içeriği hafifçe soluklaştırılır (opacity düşer),
- Kartın kenarlığı amber tonuna döner ve hafif bir ışıltı efekti gösterir,
- Deploy (roket) düğmesi gizlenir — uyuyan bir servise deploy tetiklemek doğrudan gerekmez; deploy zaten servisi uyandırır.

Servis uykuya geçerken "UYUTULUYOR", uyanırken "UYANIYOR" etiketlerini görürsünüz.

### Anlık güncelleme

Komuta (yeni React arayüzü) servis uyku durumunu kısa aralıklarla arka planda yeniler; geçiş anlarında (uyutuluyor / uyanıyor) saniye başı güncelleme yapar. "Uyandır" düğmesine bastığınız an banner hemen durumunu günceller — API cevabını beklemeden hızlı geri bildirim alırsınız. Eski Angular arayüzünde aynı bilgi SignalR üzerinden canlı olarak da gelir.

---

## Servisimi elle uyandırmam gerekirse?

Üç yolla mümkündür:

1. **Dashboard'dan "Uyandır" düğmesi** — en hızlı yol. Bir kez tıklayın, pod hazır olduğunda durum kartı otomatik güncellenir.
2. **İlk HTTP isteği** — servisinize bir tarayıcıda veya `curl` ile istek atın, Activator servisi arka planda uyandırır, isteğiniz birkaç saniye gecikmeyle cevap alır.
3. **CI/CD pipeline deploy** — bir deploy tetiklediğinizde servis otomatik olarak uyandırılır (pipeline çalışan bir servisin uyuması zaten engellenir).

---

## Servisimi elle uyutabilir miyim?

Evet. Dashboard durum kartından "Hibernate" (Uykuya Al) seçeneğini seçerek servisinizi talep üzerine uyutabilirsiniz. Bu özellikle:

- Kısa süreli test ortamlarını kapatmak,
- Gece/hafta sonu kullanılmayacak demo servisleri kapatmak,
- Planlı bir kullanım penceresi (örn: "yalnız Pazartesi-Cuma 09:00–18:00") boyunca aktif tutmak

gibi senaryolarda kullanışlıdır.

---

## Uyku moduna girmesini istemediğim bir servisim var

Admin paneli üzerinden yöneticinize iki tür muafiyet ekleyebilirsiniz:

- **Asla Uyutma (NeverSleep):** Servis ne kadar süre boşta kalırsa kalsın uyutulmaz. Örneğin sık cron iş çalıştıran bir zamanlayıcı servisi, Webhook alıcısı veya kritik bir sağlık kontrolü servisi.
- **Zorunlu Uyut (ForceSleep):** Daha agresif uyku — trafik eşiğine bakılmaksızın politika çalıştığında uyutulur. Bu genellikle demo ortamlar için kullanılır.

Kendi hesabınızda bu muafiyetleri ekleyemezsiniz; DevOpsZon yönetici ekibi (veya organizasyonunuzdaki admin rolüne sahip kullanıcı) admin paneli üzerinden sizin için ekler. Talebinizi destek üzerinden veya yönetici paneline sahip ekip arkadaşınızdan isteyebilirsiniz.

---

## Planımda uyku özelliği açık mı?

| Plan türü | Varsayılan davranış |
|---|---|
| Free plan | **Açık** — servisleriniz 3 saat boşta kaldıktan sonra otomatik uyur |
| Ücretli planlar (Starter / Pro / Business) | **Kapalı** — sürekli çalışır; yönetici tarafınıza açabilir |
| Enterprise plan | **Kapalı** — özel talep üzerine açılır |

Plan yöneticisi her plan için bu davranışı override edebilir. Kendi planınızın güncel durumunu servis dashboard'un üst kısmındaki "Uyku Modu" kartında görebilirsiniz — kart görünmüyorsa planınızda uyku özelliği kapalı demektir.

---

## Sık sorulan sorular

**Soru: Uyuyan servisim veri kaybeder mi?**
Hayır. Uyku yalnızca pod sayısını 0'a çeker. Persistent Volume (disk), environment variable, config, secret gibi tüm kalıcı veriler aynen kalır. Servis uyandığında aynı veriyle kaldığı yerden devam eder.

**Soru: WebSocket bağlantılarım uykuya girerken ne olur?**
Uyuyan servislerde mevcut TCP bağlantıları (WebSocket dahil) pod kapanırken graceful shutdown ile kapatılır. İstemci tarafında yeniden bağlanma mantığınız varsa Activator yeni bağlantıyı yakalar ve servisi uyandırır. Sürekli açık bağlantı gereken servisler için admin'den **NeverSleep** muafiyeti isteyin.

**Soru: Uyandırma sırasında bir hata olursa?**
Activator tarafında 30 saniye içerisinde servis hazır olamazsa kullanıcıya HTTP 503 "Service is starting up, please retry" yanıtı döner. Retry-After header'ı gönderilir. Kısa bir bekleme sonrası tekrar istek atıldığında servis büyük ihtimalle hazır olur. Tekrarlayan hatalar dashboard'da kırmızı bir uyku durumu olarak görünür.

**Soru: Uyku özelliğinin platformu yavaşlatmasından endişe etmeliyim?**
Hayır. Uyku politikası, trafik ölçümünü Hubble'ın zaten topladığı metriklerden yapar — ekstra bir ağ trafiği yoktur. Servis uykudayken ise hiçbir CPU/RAM tüketilmez, yani platform üzerindeki kaynak kullanımını azaltır.

**Soru: Saldırgan bir istemci sürekli istek atarak servislerimi uyandırıp kaynak tüketebilir mi?**
Activator sadece tek bir wake tetikler: aynı servise aynı anda gelen 100 istek, 1 wake RPC'ye yol açar (coalescing). Aynı servise sürekli istek yağan bir senaryoda servis zaten uykuya gitmez — eşiği aşamaz. Bu nedenle kötü niyetli trafik normal trafik gibi davranacaktır. Dilerseniz üst seviyede bandwidth sınırı (Bandwidth Management) ile koruyabilirsiniz.

**Soru: Bir ay içerisinde bir kez kullandığım servis uyanıyor mu?**
Evet. Uyku durumu servisi ortadan kaldırmaz; yalnız pod'u kapatır. Bir ay sonra istek geldiğinde ilk cold-start süresi (3–15 saniye) beklendikten sonra normal şekilde cevap alırsınız.

**Soru: Uyku olayları geçmişte ne zaman oldu, nasıl görürüm?**
Servis dashboard'unun **Sleep History** alt sekmesi her uyku/uyanma olayını tarih, tetikleyici (otomatik sweep mi, elle mi, ingress activator mı) ve süre bilgisi ile listeler.

---

## Daha fazla bilgi

- [Platforma genel bakış](platform-overview.md)
- [Servis yönetimi](service-guide.md)
- [Planlar ve faturalandırma](billing-plans.md)
- [HPA / otomatik ölçekleme](service-guide.md#hpa)
