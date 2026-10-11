# Gözlemlenebilirlik

Gözlemlenebilirlik ekranları bir servisin canlıda nasıl davrandığını gösterir: kaç istek aldığını, hangi uç noktaların yavaşladığını ya da hata verdiğini, tek bir isteğin servisler arasında nerede zaman harcadığını ve başarı hedeflerinizin ne durumda olduğunu. Bir sorunun ne zaman ve hangi uç noktada başladığını bulmak, yavaş bir isteği adım adım incelemek ya da bir deploy'un etkisini görmek için buradan başlarsınız.

Log arama ayrı bir ekrandır: [Loglar](service-logs.md). Gözlemlenebilirlik ekranları loglara bağlantı verir ama onların yerini almaz.

---

## Ekranlara Ulaşma

İki giriş noktası vardır:

- **Servis sayfasından.** Servisi açın ve sol menüdeki **Gözlemlenebilirlik** sekmesine tıklayın. O servisin **Genel Bakış** ekranı açılır ve sol menü Gözlemlenebilirlik menüsüne döner.
- **Ana menüden.** **İzleme** başlığı altındaki [**Gözlemlenebilirlik**](https://console.komuta.io/observability) sayfasında **Servislere göz at** ile servis kataloğunu açın, servisin satırındaki **Observability'yi aç** ile o servisin ekranlarına geçin.

Servisin Gözlemlenebilirlik menüsü:

| Menü | Ne gösterir | Gözlemlenebilirlik açık olmalı mı? |
|---|---|---|
| **Genel Bakış** | İstek sayısı, hata oranı, yanıt süresi, değişiklikler ve en çok zaman alan uç noktalar | Evet |
| **Trace** | Tek tek isteklerin izleri ve ayrıntıları | Evet |
| **Metrikler** | Ölçülen CPU ve bellek kullanımı | Hayır |
| **Bağımlılıklar / L7** | Servise gelen ve servisten giden ağ akışları | Evet |
| **Uç Noktalar** | Uç nokta bazında istek, hata ve gecikme | Evet |
| **Güvenilirlik → SLO ve hata bütçesi** | Başarı hedefleri ve hata bütçesi | Evet |
| **Güvenilirlik → Anomaliler ve uyarılar** | Taban çizgisinden istatistiksel sapmalar | Evet |
| **Güvenilirlik → Uyarılar** | Servisin uyarı kuralları ve sessizlikleri | Hayır |
| **Güvenilirlik → Sürüm etkisi** | Deploy öncesi ve sonrası karşılaştırması | Evet |
| **Optimize Et → Maliyet** | Maliyet dağılımı ve telemetri verimliliği | Evet |
| **Optimize Et → Talep tahmini** | Geçmiş isteklere göre talep tahmini | Evet |
| **Optimize Et → Telemetri saklama** | Saklanan telemetri, kota ve maliyet | Evet |
| **Araçlar → Panolar** | Kendi oluşturduğunuz servis panoları | Evet |
| **Araçlar → Doğal dille sorgu** | Telemetriye doğal dille soru sorma | Evet |

Sayfanın üstünde servis ve ortam bilgisi, **Uyarılar** kısayolu ve abonelik açıksa **Aboneliği iptal et** düğmesi bulunur.

---

## Gözlemlenebilirliği Açma

Gözlemlenebilirlik servis başına açılır. Açık olmayan bir serviste ücretli ekranların yerine **Gelişmiş gözlemlenebilirlik — ücretli plan** tanıtımı görünür; **Metrikler** ve **Uyarılar** her zaman kullanılabilir.

1. Servisin ücretli Gözlemlenebilirlik ekranlarından birini açın ve **Yükselt ve aç** düğmesine tıklayın.
2. **Observability'yi aç** penceresinde servisin telemetrisinden ne kadarının saklanacağını seçin (aşağıda **Saklama ve Maliyet** bölümüne bakın). Pencere seçiminize göre tahmini aylık maliyeti hesaplar.
3. **Observability'yi aç** ile onaylayın.

Onayladığınızda servis bir kez yeniden dağıtılır ve izleme uygulamanıza eklenir. Servis henüz hiç dağıtılmamışsa izleme ilk dağıtımda eklenir. Veriler, yeniden dağıtılan sürüm istek almaya başladıktan sonra görünür.

**Kimler kullanabilir?**

- Gözlemlenebilirliği yalnızca organizasyon yöneticileri açabilir. Diğer üyeler düğme yerine "Yükseltme için organizasyonunuzdaki bir yöneticiye başvurun." mesajını görür.
- Ücretli ekranları görmek ayrı bir yetki ister. Yöneticilerde bu yetki vardır; bir üye ekranı açtığında **Service Insights izni gerekli** uyarısını görüyorsa, yönetici **Erişim Yönetimi** sayfasında üyeyi seçip **Modül yetkileri** kartındaki **Gelişmiş gözlemlenebilirlik** satırından **Gözlemlenebilirliği görüntüle** yetkisini verir. Aynı satırda **SLO düzenle**, **Anomalileri görüntüle**, **Doğal dil sorgusu** ve **Özel pano düzenle** yetkileri de bulunur. Ayrıntılar: [Organizasyonlar ve Ekip Üyeleri](organizations-team.md).

**Kapatma.** Abonelik açıkken sayfanın üstündeki **Aboneliği iptal et** düğmesini kullanın. Onayladığınızda trace, uç nokta analitiği ve SLO toplanması durur, ekranlar ücretli plan önizlemesine döner ve izlemenin kaldırılması için servis yeniden dağıtılır. Toplanmış veriler süresi dolana kadar kalır ve depolaması artık faturalanmaz. SLO'larınız ve panolarınız silinmez; yeniden açtığınızda geri gelir.

---

## Veri Nasıl Gelir

**Otomatik izleme.** Gözlemlenebilirlik açık bir serviste Komuta izlemeyi uygulamanıza kendisi ekler; kodda değişiklik ya da ortam değişkeni gerekmez. Dil, Dockerfile'daki çalışma imajından algılanır:

| Dil | Otomatik izleme |
|---|---|
| .NET (ASP.NET dahil) | Var. Çalışma imajı resmi Microsoft .NET imajı, Debian, Ubuntu, distroless ya da Alpine tabanlı olmalıdır; Komuta'nın tanımadığı bir taban imajda izleme eklenmez. |
| Java (Spring Boot, Quarkus, Micronaut dahil) | Var |
| Node.js (Next.js, NestJS, Express, Fastify, Koa dahil) | Var |
| Python (FastAPI, Django, Flask dahil) | Var |
| Go, Ruby, PHP, Rust ve diğerleri | Yok; aşağıdaki elle kurulumu kullanın |

İzleme sürekli çalışan servislere eklenir; Job ve CronJob'lar için trace toplanmaz.

**Elle kurulum (desteklenmeyen diller).** Desteklenmeyen bir dilde **Trace** ekranı "Go için otomatik izleme kullanılamıyor" gibi bir mesaj ve **OpenTelemetry kurulumu yap** düğmesi gösterir. Açılan **Bu servisi OpenTelemetry ile bağlayın** sihirbazı üç adımdan oluşur:

1. **Genel bakış** — Kod tarafında neden kurulum gerektiği.
2. **Kodu hazırla** — OpenTelemetry API, SDK, OTLP exporter ve web framework'ünüzün enstrümantasyon paketini kurun. SDK'yı uygulama başlangıcında olabildiğince erken başlatın; sunucu, dış HTTP, veritabanı ve mesajlaşma enstrümantasyonlarını açın; gelen ve giden isteklerde W3C trace context'i koruyun.
3. **Dağıt** — "OpenTelemetry SDK'sını ve gerekli enstrümantasyon kütüphanelerini uygulama koduna ekledim." kutusunu işaretleyip **Entegre et ve dağıt** deyin.

Telemetri adresi, protokol, servis kimliği ve örnekleme ayarlarını siz girmezsiniz; Komuta bunları ekleyip servisi yeniden dağıtır. Bu kurulum yalnızca trace gönderir, metrik ya da log göndermez. Son adımı uygulamak için servis düzenleme yetkisi gerekir.

**Trace'leri loglarla eşleştirme.** Bir trace'in içinde ilişkili logları görmek için uygulamanın logları stdout'a, trace kimliği taşıyacak şekilde yazması gerekir. Otomatik eşleştirme .NET, Java, Python ve Node.js'i kapsar:

| Dil | Ne yapmalı |
|---|---|
| .NET | Logları Microsoft.Extensions.Logging (ILogger) üzerinden yazın. Yalnızca Serilog kullanıyorsanız olayları ILogger'a da iletin ya da `trace_id` ve `span_id` alanları üretin. |
| Java | Logları SLF4J ile Logback veya Log4j2 üzerinden yazın. |
| Node.js | Yapılandırılmış logları Pino, Winston veya Bunyan ile yazın. |
| Python | Standart `logging` modülünü kullanın. Loguru veya structlog kullanıyorsanız standart logging'e bağlayın ya da `trace_id` ve `span_id` alanları üretin. |
| Diğer diller | W3C trace context'i iletin ve loglara yapılandırılmış `trace_id` ve `span_id` alanları yazın. |

Logger'ı değiştirdikten sonra servisi yeniden dağıtın.

**Otomatik izlemeyi kapatma.** Otomatik izleme bir uygulamayla uyumsuzsa servis için kapatılabilir: Stack dosyasında servisin `security.autoInstrumentation.disabled` alanını `true` yapın. Ayrıntılar: [komuta.yaml Manifest Başvurusu](stack-manifest-reference.md).

---

## Genel Bakış

**Genel Bakış**, "servis çalışıyor mu, neye göre ve zaman nereye gidiyor?" sorusunu tek ekranda yanıtlar.

**Zaman aralığı.** **1 saat**, **6 saat**, **24 saat** veya **7 gün** seçin; varsayılan 1 saattir. 1 ve 6 saatlik aralıklar her dakika yenilenir; 24 saat ve 7 gün, okunduğu andaki durumu gösterir. **Dünle karşılaştır** anahtarı her değeri bir gün öncesinin aynı aralığıyla karşılaştırır; 7 gün seçiliyken anahtar **Geçen haftayla karşılaştır** olur. Aralık ve karşılaştırma adres çubuğunda tutulduğu için bağlantıyı paylaştığınız kişi aynı görünümü açar.

**Durum satırı.** Ekran tek cümlelik bir durumla açılır:

| Durum | Anlamı |
|---|---|
| **Sağlıklı** | Sunucu hataları %1'in altında ve açık uyarı yok |
| **Dikkat gerekiyor** | Sunucu hataları (5xx) %1 veya üstünde ya da açık bir uyarı var |
| **Sorun var** | Sunucu hataları %5 veya üstünde ya da açık kritik bir uyarı var |
| **Bu aralıkta istek yok** | Servise istek gelmedi ya da servis için izleme açık değil |
| **Durum belirsiz: verinin bir kısmı gecikmeli** | Aralığın verisinin bir kısmı henüz gelmedi; rakamlar değişebilir |
| **Veriler okunamadı** | Telemetri okunamadı; bu, servisin sağlığı hakkında bir şey söylemez |
| **Durum belirlenemedi** | Kanıt, servisin çalışıp çalışmadığını söylemeye yetmiyor |

İstemci hataları (4xx) durumu hiçbir zaman değiştirmez. Durumun yanında **Kısmi veri** etiketi varsa aralığın bir kısmının verisi eksiktir; neyin eksik olduğu sayfanın altındaki **Veri kaynağı** bölümünde yazar.

**Göstergeler.** **İstek / dk**, **Sunucu hataları (5xx)**, **İstemci hataları (4xx)** ve **Yanıt süresi (p95)**. Karşılaştırma açıksa her birinin altında düne ya da geçen haftaya göre değişim görünür. Yanıt süresi kutusu p50 ve p99 değerlerini de verir.

**Grafikler.** **Duruma göre istekler** istekleri 2xx, 3xx, 4xx ve 5xx olarak; **Yanıt süresi** gecikmeyi zaman içinde gösterir. Deploy ve yeniden başlatma gibi değişiklikler grafiklerde işaretlenir.

**Değişiklikler ve uyarılar.** Aralıktaki deploy'lar, başarısız deploy'lar, yeniden başlatmalar (bellek taşması, sağlık kontrolü başarısız, başlatılamadı gibi nedenleriyle), çökme döngüleri ve açılan ya da kapanan uyarılar.

**Zaman nereye gidiyor.** Toplam sürenin en büyük payını alan uç noktalar. **Tüm uç noktalar** ile **Uç Noktalar** ekranına geçersiniz.

**Kaynaklar (şu an).** Servisin anlık CPU ve bellek kullanımı ve limite oranı; ayrıntısı **Metrikler** ekranındadır.

Alttaki bağlantılar **Trace'leri aç**, **Logları aç**, **Tüm uç noktalar** ve **Bağımlılıklar / L7** ekranlarına aynı aralıkla götürür. Log sayfası en fazla 12 saat okuduğu için 24 saat ve 7 günde bağlantı **Logları aç (son 12 saat)** olur.

---

## Uç Noktalar

**Uç Noktalar**, servisin her rotası için istek, hata ve gecikmeyi ayrı bir satırda gösterir. Genel Bakış ile aynı veriden okunduğu için iki ekran birbiriyle çelişmez.

Sayım aralığı **1 saat**, **6 saat** veya **24 saat** olabilir. Genel Bakış'tan geldiyseniz oradaki aralıkla açılır; 7 gün 24 saate iner.

| Sütun | Ne gösterir |
|---|---|
| **Uç nokta** | Metot ve rota şablonu (ör. `GET /api/orders/{id}`) |
| **Eğilim** | Aralık içindeki istek eğilimi |
| **İstek** | Toplam istek sayısı |
| **Arıza** | Sunucu hatası (5xx) oranı; parantez içinde sayısı |
| **Reddedilen** | İstemci hatası (4xx) oranı; parantez içinde sayısı |
| **En kötü zaman diliminin p95'i** | Aralığa giren 5 dakikalık dilimlerin en kötüsündeki p95 gecikmesi |

> **Not:** Gecikme sütunu aralığın tamamının p95'i değil, en kötü 5 dakikalık dilimin p95'idir. Gecikme özeti olmayan satırlarda değer yerine **sketch yok** yazar; bu satırlar süre eşiğiyle filtrelediğinizde hızlı sayılmaz, listeden çıkar.

**Filtre ve sıralama.** Rota veya metoda göre arayın. Sağlık filtresi **Tümü**, **Hata verenler** (5xx alan), **Hatasız** (hiç 4xx ya da 5xx almayan) ve **Ölçülmemiş** (oranlanacak isteği olmayan) değerlerini alır. **Eşikler** altında **En az bu kadar istek** ve **En kötü zaman dilimi p95'i en az (ms)** girilir. Sıralama **En çok trafik**, **En çok hata**, **En yavaş zaman dilimi** veya **Rota** olabilir.

**Özel satırlar ve etiketler.**

| Satır / etiket | Anlamı |
|---|---|
| **Rotası olmayan istekler** | Hiçbir rotanın cevap vermediği istekler; çoğu 404'tür. Tarayıcıların denediği her adres ayrı uç nokta olmasın diye yollarıyla adlandırılmaz. Trace'lerini zamana ya da duruma göre arayın. |
| **Diğer istekler (kardinalite sınırı)** | Servis ayırt edilebilen uç nokta sınırını aştığında kalan istekler bu satırda toplanır. Trace'leri gerçek rotalarını taşır. |
| **Sağlık kontrolü** | Satırdaki isteklerin bir kısmı platformun sağlık kontrolleridir; oranı ipucunda yazar. Bu istekler toplamlara dahildir. |
| **Framework rotası** | Rota, uygulamanın framework'ünün bildirdiği şablondur. |
| **Çıkarılmış rota** | Framework rota bildirmediği için şablon, gözlenen istek yolundan çıkarılmıştır. |

Komuta istek yollarındaki değişken parçaları (sayı, UUID, uzun onaltılık değer, token, e-posta, IP) yer tutucuya çevirir; böylece `/orders/123` ve `/orders/456` aynı satırda toplanır.

**Uç nokta detayı.** Bir satıra tıkladığınızda dört sekme açılır:

- **Zaman içinde trafik** — İstek, hata ve p95; **5 dakika** (son 6 saat), **1 saat** (son 7 gün) ve **1 gün** (son 30 gün) dilimleriyle. Eksen saklanan dilimler bittiği yerde durur; bu, trafiğin bittiği anlamına gelmez.
- **Durum kodları** — Her HTTP durum kodunun sayısı, payı ve p95'i. Bir koda tıklayınca o yanıtı veren trace'ler açılır.
- **Trace'ler** — Bu uç noktanın trace'leri.
- **Loglar** — Uç noktanın en yeni trace'leri, trace kimliği üzerinden servisin log satırlarıyla eşleştirilir. **Bu satırları indir** ile satırları `.ndjson` ya da `.log` olarak indirebilirsiniz.

---

## Trace'ler

Trace, tek bir isteğin ya da arka plan işinin baştan sona izidir. Her adım bir *span*'dir: gelen istek, veritabanı sorgusu, başka bir servise yapılan çağrı. **Trace** ekranı trace'leri listeler; birini seçtiğinizde ayrıntısı listenin yanında açılır.

### Arama

1. Zaman aralığını seçin: **Son 15 dakika**, **Son 1 saat**, **Son 6 saat** veya **Son 24 saat**. Varsayılan 1 saattir.
2. Gerekirse daraltın: **Yalnızca hatalar**, **Süre (ms)** için en az ve en çok değer, **Metot**, **Rota** (rota kataloğundan seçilir), arka plan işleri için **İş adı**, kuyruk tüketicileri için **Kuyruk veya konu**.
3. Sıralamayı seçin: **En yeni**, **En uzun işlem** ya da **Önce hatalar**. Sayfa başına 5 veya 20 trace gösterilir.

Belirli bir trace'i arıyorsanız (ör. bir log satırındaki `trace_id`), kimliği **Trace ID ile açın (32 onaltılık karakter)** kutusuna yazıp **Trace'i aç** deyin. Doğrudan arama yalnızca seçili zaman aralığının içinde arar; trace daha eskiyse aralığı genişletin.

> **Not:** Açılışta seçili aralıkta hiç trace yoksa ekran aralığı adım adım genişletir (1 saat, 6 saat, 24 saat) ve bunu açıkça yazar: "Son 1 saat aralığında eşleşen trace dönmedi; pencere Son 6 saat olarak genişletildi." Aralığı kendiniz seçtiğinizde bu davranış durur.

**Canlı izleme.** Liste açıkken pencere her dakika ilerler: **En yeni** sıralamasında **En yeniler izleniyor**, diğer sıralamalarda **Canlı pencere** etiketi görünür. Sonraki sayfaya geçtiğinizde ya da ID ile bir trace açtığınızda pencere sabitlenir (**Duraklatıldı**); **Yeniden en yenileri izle** ile devam ettirirsiniz.

**Açıklanmışlar anahtarı.** Varsayılan olarak liste yalnızca nedeni belli olan trace'leri gösterir: gelen bir istek ya da adı belli bir arka plan işi. Arkasında ne istek ne iş olan, tek span'lik giden çağrılar gizlenir. Hepsini görmek için anahtarı kapatın (**Açıklanmamışlar dahil**); bu satırlar **Açıklanmamış** etiketini taşır.

**Satırları okuma.** Her satır kök işlemi (metot ve rota ya da iş adı), durumu (**Başarılı**, **Hata**, **Bildirilmedi**), süreyi ve span sayısını gösterir. Süre, duvar saatiyle geçen süredir; uzun ömürlü bağlantılar (**Uzun ömürlü bağlantı**) ve zamanlanmış işler (**Zamanlanmış iş**) ayrıca etiketlenir. **En uzun işlem** sıralaması kök işlemin süresine göredir, CPU süresine ya da span sürelerinin toplamına göre değil.

---

## Trace Detayı

Listeden bir trace seçtiğinizde ayrıntısı iki görünümle açılır:

- **Span şelalesi** — Span'ler zaman ekseninde, çağrı hiyerarşisiyle. Her satırda span'in süresi ve kendi kodunda geçen payı (**kendi süresi**) yazar. Üst bilgide kritik yol bulunur: işlemin bitişini belirleyen span zinciri ve bunun ne kadarının dış çağrılarda, ne kadarının servisin kendi kodunda geçtiği. Hata varsa **ilk hata** bağlantısı ve **Önceki hata** / **Sonraki hata** düğmeleri hatalı span'lere götürür. Art arda tekrarlanan satırlar katlanır.
- **Servis haritası** — Trace'e katılan servisler ve aralarındaki çağrılar; her düğümde çağrı sayısı, hata sayısı ve kendi süresi.

Bir span'e tıkladığınızda **Span kanıtı** paneli açılır: servis, span türü, başlangıç, süre, HTTP metodu ve rotası, durum mesajı, **Yönetilen nitelikler**, **Kaynak nitelikleri** ve **Span olayları**.

**İlişkili loglar.** Trace'in kimliğini taşıyan log satırları trace'in altında listelenir. Etiket, eşleşmenin nasıl yapıldığını söyler: **Trace id ile eşleşti** kesin eşleşmedir; **Zaman penceresiyle eşleşti** yaklaşıktır ve bazı satırlar başka isteklere ait olabilir. Bir satırı log sayfasında görmek için **Bu satırı log sayfasında aç** bağlantısını kullanın.

**Kanıt raporunu indir.** Trace'in kimliğini, kanıt durumunu ve span'lerini bir Markdown dosyası olarak indirir; destek talebine ya da olay kaydına eklemek için kullanışlıdır.

Detayda şu uyarılardan birini görebilirsiniz:

| Uyarı | Anlamı |
|---|---|
| **Kısmi span detayı: … span'in … tanesi gösteriliyor** | Trace güvenlik sınırını aşan sayıda span içeriyor (en fazla 2.000 span gösterilir). Şelale tam yürütme yolu değildir. |
| **… span'in parent'ı eksik** | Bazı span'lerin üst span'i gelmedi; bu satırlar hiyerarşinin dışında gösterilir. |
| **Saat uyumsuzluğu** | Bazı span'lerin saati kök span'inkiyle uyuşmuyor; konumları yaklaşıktır, süreler doğrudur. |
| **Saklanmış trace span'i bulunamadı** | Seçilen aralıkta bu trace için span saklanmamış. Normal ve hızlı trace'ler örnekleme nedeniyle saklanmayabilir. |

### Trace'leri Karşılaştırma

- **Benzer trace'lerle karşılaştır** — Aynı işlemin (aynı metot ve rota) aynı zaman penceresindeki en yavaş 10 trace'ini listeler ve bu trace'in bu küme içindeki sırasını yazar. Bu sıra yalnızca bu küme içindir, tüm trafiğin yüzdeliği değildir.
- **Genişletilmiş sunumda aç** — Trace'i geniş bir görünümde açar. Buradaki **Başka bir trace ile karşılaştır** ile ikinci bir trace'i (aynı işlemin en yavaşlarından ya da listede yüklü olanlardan) kendi şelalesiyle yanına açarsınız. Hızlı ve yavaş bir isteği yan yana koymanın yolu budur.

Kök span'inde rota ya da metot olmayan trace'ler için benzer küme oluşturulamaz; ekran bunu söyler ve ikinci trace'i yüklü listeden seçmenizi ister.

---

## Örnekleme ve Kanıt Etiketleri

Ekranlardaki sayılar iki farklı kaynaktan gelir; sonuçları doğru okumak için farkı bilmek gerekir.

**Sayımlar örneklenmez.** Genel Bakış ve Uç Noktalar'daki istek, hata ve gecikme değerleri her isteğin sayıldığı sayaçlardan gelir. Genel Bakış'ın altındaki **Veri kaynağı** bölümünde **Kayıt** satırı bunu belirtir: **Örnekleme öncesi span sayaçları — her istek sayılır**.

**Trace'ler örneklenir.** Her isteğin trace'i saklanmaz. Hatalı ve yavaş trace'lerin ayrı saklama kuralları vardır ve daha yüksek oranda saklanır; normal ve hızlı trace'lerin yalnızca bir kısmı saklanır. Bu oran Trace ekranının üstünde **Normal trace saklama oranı** olarak yazar. Bu yüzden trace listesi isteklerin kesin toplamı değildir; bir isteğin trace'ini bulamamanız o isteğin gerçekleşmediği anlamına gelmez.

Ekranlarda sık göreceğiniz etiketler:

| Etiket | Anlamı |
|---|---|
| **Kesin kanıt** | Sorgu tüm kaynaklardan eksiksiz okundu; sonuç kesindir. |
| **Nitelikli kanıt** | Sonuç gerçek veridir ama eksik olabilir (örnekleme, gecikmeli ya da okunamayan bir kaynak). |
| **Kanıt kapsamı** | İstenen verinin ne kadarının okunabildiği. Ölçülemediyse **Ölçülmedi** yazar. |
| **Kaynak watermark değeri** | Verinin hangi ana kadar eksiksiz geldiği. Daha yeni kayıtlar henüz gelmemiş olabilir. |
| **Kısmi veri** | Genel Bakış'taki durumun aralığın tamamına değil, bir kısmına dayandığını gösterir. |
| **Son yenileme reddedildi** | Son okuma kendi içinde tutarsız olduğu için kabul edilmedi; ekranda son geçerli okuma duruyor ve ilerlemiyor. |

Trace listesinin üstünde sonucun genel durumu da yazar:

| Durum | Anlamı |
|---|---|
| **Gerçek kayıtlar · kısmi trafik** | Gösterilen trace'ler gerçektir; ancak normal trafiğin yalnızca bir kısmı saklandığı için liste tüm isteklerin kesin toplamı değildir. Örneklenen trace'lerde olağan durum budur. |
| **Doğrulanmış boş sonuç** | Sorgu eksiksiz tamamlandı ve aralıkta gerçekten trace yok. |
| **Gecikmeli** | Kaynak verisi geride; güncel trace'ler eksik olabilir. |
| **Kullanılamıyor** / **Uyumsuz** | Sorgu tamamlanamadı; kesin ya da "sağlıklı" bir sonuç gösterilmez. |

> **Not:** Gözlemlenebilirlik ekranları okunamayan veriyi "sağlıklı" ya da "sıfır" olarak göstermez. "… okunamadı" başlıklı bir kart servisin durumu hakkında değil, okumanın kendisi hakkındadır. Biraz sonra tekrar deneyin ya da aralığı daraltın.

---

## SLO ve Hata Bütçesi

SLO, bir uç noktanın ya da servisin tamamının belirli bir süre boyunca tutturması gereken başarı oranıdır. Hata bütçesi, hedefi bozmadan izin verilen hatalı istek sayısıdır.

1. **Güvenilirlik → SLO ve hata bütçesi** ekranında **Yeni SLO** düğmesine tıklayın.
2. **Görünen ad** girin.
3. **Method** ve **Path** alanlarını doldurun (ör. `GET` ve `/api/checkout`). İkisini de boş bırakırsanız SLO servisin tamamı için geçerli olur (**Servis geneli SLO**).
4. **Hedef (%)** seçin: %99, %99,5, %99,9, %99,95 veya %99,99. Varsayılan %99,9'dur.
5. **Pencere** seçin: **7 gün** veya **30 gün**. Varsayılan 30 gündür.
6. **SLO oluştur** ile kaydedin.

Hangi hedefi seçeceğinizden emin değilseniz **Öner** düğmesi, uç noktanın son 30 günlük trafiğine bakarak yapay zekâ ile bir hedef ve pencere önerir; kabul ettiğiniz değerler forma yazılır.

Her SLO kartı hedefi, şu anki başarı oranını, harcanan hata bütçesini ve durumu gösterir:

| Durum | Anlamı |
|---|---|
| **Bütçede** | Hata bütçesinin %25'inden azı harcandı |
| **İzlemede** | Bütçenin %25–75'i harcandı |
| **Hızla tükeniyor** | Bütçenin %75–100'ü harcandı |
| **Tükenmiş** | Bütçe bitti; hedef bu pencerede tutturulamıyor |
| **Pencere hazırlanıyor** | Pencerenin tamamı için henüz veri yok; değerler yalnızca verisi olan günleri kapsar |
| **Bilinmiyor** | Durum hesaplanamadı |

**SLO bildirimleri.** Komuta SLO'ları saatte bir değerlendirir ve durum kötüleştiğinde organizasyonun bildirim kanallarına bildirim gönderir: **Tükenmiş** kritik, **Hızla tükeniyor** uyarı, **İzlemede** bilgi düzeyindedir. Bütçe harcanmaya devam ettikçe hatırlatma, SLO düzeldiğinde de bir kez bildirim gelir. Pencere hazırlanırken bildirimler duraklatılır. Kanalların ayarı için: [Kanallar ve Bildirim Teslimi](notification-guide.md).

---

## Diğer Ekranlar

| Ekran | Ne işe yarar |
|---|---|
| **Metrikler** | Servisin ölçülen CPU ve bellek kullanımı. Eksik bir seri sıfır değil, bilinmiyor olarak gösterilir. Gözlemlenebilirlik açık olmadan da kullanılır. |
| **Bağımlılıklar / L7** | Servise gelen (**Gelen**) ve servisten giden (**Giden**) ağ akışları; HTTP görülen bağlantılarda HTTP hata oranı ve p95. İstek değil akış sayılır: uzun ömürlü tek bir bağlantı tek akıştır. |
| **Anomaliler ve uyarılar** | Uç noktaların gecikme, hata oranı ve istek hacmini son 7 günün taban çizgisiyle karşılaştırır; yalnızca istatistiksel olarak anlamlı sapmaları gösterir. Yeni bir uç nokta taban çizgisi oluşana kadar değerlendirilmez. |
| **Uyarılar** | Servisin uyarı kuralları, tetiklenme geçmişi ve sessizlikleri. Ayrıntılar: [Uyarılar: Genel Bakış](alert-guide.md). |
| **Sürüm etkisi** | Her deploy'dan sonraki saati öncesiyle karşılaştırır; kanıt eksikse neden-sonuç iddiasında bulunmaz. |
| **Maliyet** | Servisin maliyet dağılımı, telemetri hacmi ve yapay zekâ destekli tasarruf önerileri. |
| **Talep tahmini** | Geçmiş istek hacminden önümüzdeki günlerin talep tahmini. Kaynak yeterliliği bu ekrandan çıkarılmaz. |
| **Panolar** | Hazır widget kataloğundan kendi servis panolarınızı oluşturup kaydedersiniz. |
| **Doğal dille sorgu** | Servisin telemetrisine Türkçe ya da İngilizce soru sorarsınız; dönen tabloyu ve çalıştırılan salt okunur sorguyu görürsünüz. Üretilen açıklama kanıt değil, yol göstericidir. Organizasyon başına saatte 30 sorgu sınırı vardır. |

---

## Saklama ve Maliyet

Gözlemlenebilirlik açık bir servisin trace'leri, ağ akışları ve çalışma zamanı güvenlik olayları saklanır ve saklanan miktar kadar ücretlendirilir. Loglar bu hesaba girmez. Boyut, telemetri geldiği haliyle (sıkıştırılmadan) ölçülür; fatura da aynı sayıyı kullanır.

**Fiyat.** GB-saat başına 0,000342 USD (aylık GB başına yaklaşık 0,25 USD), vergi hariç. Kullanım saatlik ölçülür ve ekstrede **Gözlemlenebilirlik** kategorisinde görünür. Bir serviste Gözlemlenebilirlik açıldığında o anki koşullar (dahil alan, varsa ücretsiz dönem ve fiyat) servise kilitlenir; saklama seçimini sonradan değiştirmek bu koşulları korur. Güncel koşullar açma penceresinde ve **Hesap → Observability Depolama** sayfasında yazar.

**Saklama seçimi.** Açarken ve sonradan **Optimize Et → Telemetri saklama** ekranındaki **Ne kadar saklansın** bölümünde iki seçenekten birini seçersiniz:

| Seçenek | Ne olur |
|---|---|
| **… içinde kal** | Veri dahil alanı geçince yer açmak için en eski veri silinir; servis dahil alan boyutunda kalır. Dahil alanın üzerinde hiçbir zaman ödeme yapmazsınız. Varsayılan seçenek budur. |
| **… üzerine büyü** | Eski veri daha uzun saklanır; servisin sakladığı her GB için ödersiniz. **Şu boyutta büyümeyi durdur:** ile bir üst sınır koyabilirsiniz; sınıra ulaşınca en eski veri silinir. Sınır koymazsanız servis platformun servis başına tavanına kadar büyür. |

Ücretsiz dönem varsa, bu dönemde dahil alan ücretsizdir; büyüme seçeneğinde dahil alanın üstü baştan ücretlendirilir. Dönem bitince servisin sakladığı her GB ücretlendirilir (dahil alanda kalma seçeneğinde en fazla dahil alan kadar).

**Maliyet hesaplayıcı** servisin ölçülen günlük ortalamasını (son 7 tam gün) ya da sizin girdiğiniz değeri kullanarak servisin hangi boyutta dengeye oturacağını, ne zaman dolacağını ve aylık maliyeti gösterir. Seçimi küçültmek servisin o an sakladığı verinin bir kısmını hemen siler; hesaplayıcı ne kadarının silineceğini yazar.

> **Uyarı:** Silinen telemetri geri getirilemez. Seçimi değiştirmeden önce hesaplayıcının uyarısını okuyun.

**Telemetri saklama ekranı** servisin durumunu, saklanan boyutu (**Trace'ler**, **Ağ akışları** ve **Runtime güvenlik olayları** olarak ayrı ayrı), **Saklanan en eski veri** ve **Son ölçüm** zamanını gösterir. Saklama seçimini **Seçimi kaydet** ile değiştirirsiniz; bunu yalnızca organizasyon yöneticileri yapabilir.

| Durum | Anlamı |
|---|---|
| **Henüz ölçülmedi** | Servis telemetri gönderdikten birkaç dakika sonra ölçülür. |
| **Kota içinde** | Saklanan veri kotanın altında. |
| **%… dolu** | Kotanın %80'i ya da daha fazlası dolu. |
| **Kota aşıldı** | Kota aşıldı; yer açmak için en eski veri siliniyor. |
| **Yeni veriyi reddediyor** | Saklanan veri kota ile toleransın toplamını aştı; yeni trace, ağ akışı ve güvenlik olayları saklanmıyor. |
| **Saklanmıyor** | Serviste Gözlemlenebilirlik açık değil. |

Bir servis yeni telemetriyi reddettiğinde konsolun üstünde "… yeni telemetriyi reddediyor" uyarısı çıkar. Üç seçeneğiniz vardır: büyüme seçeneğine geçmek, büyüme sınırını yükseltmek ya da hiçbir şeyi değiştirmemek. Değiştirmezseniz en eski veri silinir ve servis tekrar sınırın altına indiğinde yeni veri yeniden kabul edilir; arada gelen veri kaybolur. Kota dolmaya yaklaştığında, aşıldığında ve veri reddedilmeye başladığında organizasyonun fatura e-posta adreslerine de bildirim gider.

**Hesap → Observability Depolama.** Tüm servislerin kullanımını, durumunu ve aylık tahminini tek tabloda; **Tahmini aylık toplam**, **GB başına aylık fiyat** ve varsa ücretsiz dönemi gösterir. **Dönem bütçesi** kartında fatura dönemi başına bir USD tutarı girebilirsiniz. Bütçe bir uyarı eşiğidir, harcama sınırı değildir: dönem sonu tahmini ya da hesaplanan ücret bütçeye ulaştığında kart bunu gösterir, ama veri toplama durmaz.

**Ne kadar süre saklanır?** Kota daha önce silmediği sürece trace'ler en fazla 14 gün, ağ akışları 7 gün, çalışma zamanı güvenlik olayları 30 gün saklanır. Genel Bakış ve Uç Noktalar'daki sayımlar ayrı tutulur; uç nokta trafik grafiği 30 güne kadar geriye gider.

---

## Gizlilik

**Ne toplanır?** Her trace için istek yolu ve rota şablonu, HTTP metodu ve durum kodu, süreler, servisler arası çağrılar ve uygulamanın kütüphanelerinin span'lere eklediği nitelikler (ör. veritabanı sorgu metni); ağ akışlarında bağlantının kaynağı ve hedefi, HTTP yolu ve DNS sorguları. Loglar ayrı toplanır ve [Loglar](service-logs.md) ekranındadır.

**Maskeleme.** Komuta trace verisini saklarken ve gösterirken hassas bilgileri ayıklar:

- `authorization`, `cookie`, `set-cookie` ve bağlantı dizesi gibi nitelikler ile adında token, secret, password, apikey ya da credential geçen başlıklar tamamen çıkarılır.
- Bearer ve Basic kimlik bilgileri, `password=`, `token=`, `api_key=` gibi atamalar, JWT'ler, özel anahtarlar ve URL içindeki kullanıcı adı ile parola `REDACTED` ile değiştirilir.
- URL'lerdeki hassas sorgu parametreleri (key, secret, password, signature, code, token ve benzerleri) maskelenir.
- Bir değer en fazla 4.096 karakter gösterilir; span başına en fazla 48 nitelik ve 32 olay görünür.

Çıkarılan değerler span panelinde "Politika veya kaynak sınırları nedeniyle … değer çıkarıldı", "Hassas içerik telemetri maskeleme politikası tarafından kaldırıldı." ya da "Hassas SQL parçaları veritabanı ifadesi maskeleme politikası tarafından kaldırıldı." notlarıyla belirtilir.

> **Uyarı:** Veritabanı sorgu metinleri yalnızca içlerindeki gizli parçalar ayıklanarak saklanır. E-posta adresi, IP, ad gibi kişisel veriler trace'lerde maskelenmez. Kişisel verileri URL yollarına, sorgu parametrelerine ve SQL metnine gömmekten kaçının.

**Erişim.** Veriler yalnızca organizasyonunuz içinde, ilgili yetkiye sahip kullanıcılara gösterilir; her sorgu tek bir servise ve ortamına kapsanır.

**Yapay zekâ özellikleri.** **Doğal dille sorgu** kullandığınızda sorunuz ve sorgunun döndürdüğü en fazla 200 satır, yanıtı üretmek için bir yapay zekâ modeline gönderilir. SLO **Öner** düğmesi ve **Maliyet** ekranındaki öneriler de uç noktaların toplu istatistiklerini aynı amaçla kullanır.

---

## MCP ile Kullanım

Kod ajanınızı Komuta'ya bağladıysanız ajan, konsolda gördüğünüz aynı saklanmış trace'leri arayabilir, bir trace'in ayrıntısını açabilir ve servisin loglarını okuyabilir; örneğin "son bir saatteki en yavaş 5 isteği bul ve ilkinin şelalesini özetle" diyebilirsiniz. Kurulum: [Komuta MCP Sunucusunun Kurulumu](mcp-setup.md). Araç listesi: [Komuta MCP](mcp-tools.md).

---

## Veri Görünmüyorsa

| Belirti | Olası neden | Ne yapmalı |
|---|---|---|
| Ekranda **Gelişmiş gözlemlenebilirlik — ücretli plan** tanıtımı var | Serviste Gözlemlenebilirlik açık değil | Bir organizasyon yöneticisi **Yükselt ve aç** ile açar. |
| **Service Insights izni gerekli** | Kullanıcının görüntüleme yetkisi yok | Yönetici **Erişim Yönetimi** sayfasında **Modül yetkileri** → **Gelişmiş gözlemlenebilirlik** altından **Gözlemlenebilirliği görüntüle** yetkisini verir. |
| Açtıktan sonra **Bu aralıkta istek yok** | Servis henüz yeniden dağıtılmadı ya da istek almadı | Dağıtımın bittiğini [Dağıtım Geçmişi](service-deployment-history.md) sayfasından kontrol edin, servise istek gönderin. |
| **Trace** ekranında "… için otomatik izleme kullanılamıyor" | Uygulamanın dili otomatik izlemede yok | **OpenTelemetry kurulumu yap** sihirbazını izleyin. |
| .NET servisinde hiç trace yok | Çalışma imajının tabanı tanınmadı | Dockerfile'da resmi .NET, Debian, Ubuntu, distroless ya da Alpine tabanlı bir çalışma imajı kullanıp yeniden dağıtın. |
| Job ya da CronJob için trace yok | Bu iş türlerine izleme eklenmez | Logları kullanın. |
| Bilinen bir isteğin trace'i bulunamıyor | Trace örneklendi, seçili aralığın dışında ya da saklama süresini geçti | Aralığı genişletin; hatalı ve yavaş istekler daha yüksek oranda saklanır. Logdan gelindiyse **Saklanmış trace span'i bulunamadı** bu durumu anlatır. |
| Trace'te **İlişkili loglar** boş | Logger desteklenmiyor ya da loglar stdout'a gitmiyor | **Veri Nasıl Gelir** bölümündeki dil tablosuna göre logger'ı ayarlayıp yeniden dağıtın. |
| Yeni trace'ler gelmiyor, durum **Yeni veriyi reddediyor** | Servis saklama sınırını aştı | **Telemetri saklama** ekranında büyüme seçeneğine geçin ya da sınırı yükseltin. |
| Bir kart "… okunamadı" diyor | Okuma tamamlanamadı | Birkaç dakika sonra tekrar deneyin ya da daha dar bir aralık seçin. |

---

## İlgili Dokümanlar

- [Loglar](service-logs.md) — Servis loglarında arama; log satırından trace'e geçiş.
- [Servis Dashboard](service-dashboard.md) — Servisin anlık durumu ve Servis Doktoru.
- [Uyarılar: Genel Bakış](alert-guide.md) — Uyarı kuralları, geçmiş ve sessizlikler.
- [Kanallar ve Bildirim Teslimi](notification-guide.md) — SLO ve uyarı bildirimlerinin gideceği kanallar.
- [Organizasyonlar ve Ekip Üyeleri](organizations-team.md) — Üyelere gözlemlenebilirlik yetkisi verme.
- [komuta.yaml Manifest Başvurusu](stack-manifest-reference.md) — Otomatik izlemeyi Stack dosyasından kapatma.
- [Faturalandırma ve Cüzdan](billing.md) — Ekstreler, cüzdan ve harcama limiti.
- [Komuta MCP](mcp-tools.md) — Kod ajanınızla trace ve log sorgulama.
- [Fiyatlandırma](https://www.komuta.io/pricing) — Güncel fiyatlar.
