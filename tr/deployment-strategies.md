# Dağıtım Stratejileri

Dağıtım stratejisi, bir servisin yeni sürümünün canlı trafiğe nasıl alınacağını belirler: yeni sürüm hazır olunca trafik bir anda mı geçsin, siz onaylayınca mı geçsin, yoksa kademeli mi geçsin. Komuta'da üç strateji vardır: **Blue-Green**, **Auto Promote** ve **Canary**.

Strateji her servis için ayrı seçilir. Konsoldan oluşturduğunuz yeni servisler **Auto Promote** ile başlar; organizasyon genelinde varsayılan strateji belirleyen bir ayar yoktur. Stratejiler sürekli çalışan servisler içindir; job ve cron job'lar bir strateji kullanmaz.

Bu sayfa her stratejinin nasıl ilerlediğini, rollout sırasında kullanabileceğiniz düğmeleri ve yeni sürüm sorun çıkardığında Komuta'nın ne yaptığını anlatır.

---

## Stratejiler Bir Bakışta

| Strateji | Yeni sürüm nasıl canlıya geçer | Sizin yapmanız gereken | Önizleme adresi |
|---|---|---|---|
| **Blue-Green** | Yeni sürüm eskisinin yanında ayağa kalkar; siz onaylayınca trafiğin tamamı bir anda yeni sürüme geçer. | **Yükselt** / **Canlıya al** ile onaylamak | Var |
| **Auto Promote** | Blue-Green ile aynı, ama yeni sürümün tüm pod'ları hazır olunca geçiş kendiliğinden yapılır. | Bir şey yapmanız gerekmez | Yok |
| **Canary** | Trafik yeni sürüme adım adım aktarılır: %20, %40, %60, %100. | İki adımda devam onayı vermek ya da tek seferde canlıya almak | Yok |

Üç stratejide de yeni sürüm hazırlanırken eski sürüm trafiği karşılamaya devam eder; bir süre iki sürüm yan yana çalışır. Bir pod'un "hazır" sayılması için servisin **Readiness** kontrolünü geçmesi gerekir. Bu kontrolü [Sağlık Kontrolleri](services-health-probes.md) sayfasından ayarlarsınız.

> **Not:** Komuta'da pod'ları tek tek değiştiren klasik "rolling update" stratejisi seçilemez. Seçenekler yalnızca bu üçüdür.

**Genel Bakış** sayfasındaki **Rollout durumu** kartı ve **Rollout Konsolu** stratejileri biraz farklı adlarla gösterir: Blue-Green için **Blue/Green**, Auto Promote için **Otomatik Yükseltme**.

---

## Blue-Green

1. Yeni sürüm, canlı sürümün yanında ayrı bir önizleme ortamında başlatılır. Canlı trafik eski sürümde kalır.
2. Yeni sürümün pod'ları hazır olunca rollout onayınızı bekler. **Genel Bakış** sayfasındaki **Rollout Konsolu** bu aşamayı **Hazır** ya da **Duraklatıldı** olarak gösterir; "Önizleme hazır. Aktif hale getirmek için yükseltin." uyarısı ve **Yükselt** düğmesi görünür.
3. Yeni sürümü önizleme adresinden deneyin.
4. **Yükselt** düğmesine basın. Trafiğin tamamı bir anda yeni sürüme geçer.
5. Eski sürümün pod'ları, süren isteklerin tamamlanması için yaklaşık bir dakika daha çalışır, sonra kaldırılır.

Siz onaylamadıkça yeni sürüm canlıya geçmez; rollout bu noktada bekler. Onay bekleyen bir rollout, [Rollout Bekçisi](#rollout-bekçisi) tarafından takılmış sayılmaz.

> **Not:** Onay adımı yalnızca kod değişiklikleri için değildir. Ortam değişkenlerini **Değişiklikleri Uygula** ile uyguladığınızda da yeni bir sürüm oluşur ve canlıya geçmek için onayınızı bekler.

**Önizleme adresi:** Servisin genel adresi açıksa Komuta, Blue-Green servisler için ayrı bir önizleme adresi oluşturur. Bu adres, servis adresinin ilk bölümüne `-preview` eklenerek elde edilir:

| | Adres |
|---|---|
| Servis adresi | `https://panel-3a2331e6.edge-1.komuta.app` |
| Önizleme adresi | `https://panel-3a2331e6-preview.edge-1.komuta.app` |

Konsol önizleme adresini ayrıca göstermez; servisin adresinden yukarıdaki gibi türetirsiniz. Önizleme adresi yalnızca Komuta adresi için oluşturulur, özel alan adlarınızın önizleme karşılığı yoktur. Servise [erişim koruması](service-access-protection.md) açtıysanız önizleme adresi de aynı kurallarla korunur.

Yeni sürümün pod'ları sağlıksızsa Komuta canlıya almayı reddeder; hatalı sürüm trafiğe geçmez. Bu durumda konsol önizlemede sağlıksız pod olduğunu söyler ve **İptal Et** ile önizlemeyi temizlemenizi önerir.

---

## Auto Promote

Auto Promote, onay adımı olmayan Blue-Green'dir:

1. Yeni sürüm canlı sürümün yanında başlatılır.
2. Yeni sürümün tüm pod'ları **Readiness** kontrolünü geçip en az 10 saniye hazır kalınca trafik kendiliğinden yeni sürüme geçer.
3. Eski sürümün pod'ları yaklaşık bir dakika sonra kaldırılır.

Geçiş için tek koşul pod'ların hazır olmasıdır. Hata oranı ya da yanıt süresi gibi bir eşik tanımlanmaz ve kontrol edilmez. Bu nedenle **Readiness** kontrolünüz uygulamanın gerçekten istek karşılayabildiğini göstermelidir; yalnızca sürecin açık olduğunu gösteren bir kontrol, hazır olmayan bir sürümün canlıya geçmesine yol açabilir.

Yeni sürümün pod'ları hiç hazır olmazsa geçiş yapılmaz ve eski sürüm trafiği karşılamaya devam eder. Ne olacağı için [Yeni Sürüm Başarısız Olursa](#yeni-sürüm-başarısız-olursa) bölümüne bakın.

Auto Promote için önizleme adresi oluşturulmaz.

---

## Canary

Canary, trafiği yeni sürüme sabit adımlarla aktarır. Adımlar bütün Canary servislerde aynıdır ve değiştirilemez:

| Adım | Yeni sürümün trafik payı | Sonra ne olur |
|---|---|---|
| 1 | %20 | Siz devam edene kadar bekler. |
| 2 | %40 | 5 dakika bekler, sonra kendiliğinden devam eder. |
| 3 | %60 | Siz devam edene kadar bekler. |
| 4 | %100 | Yeni sürüm tüm trafiği alır, rollout tamamlanır. |

Bir adımın ilerlemesi için yeni sürümün pod'larının hazır olması ve en az 10 saniye hazır kalması gerekir.

- **Devam Et** rollout'u bir sonraki adıma geçirir. Düğmenin üzerinde bir sonraki trafik payı yazar, örneğin `%40`.
- **Yükselt** kalan bütün adımları atlar ve yeni sürüme trafiğin tamamını verir.
- **İptal Et** trafiği tamamen eski sürüme döndürür.

Rollout sürerken **Rollout Konsolu** hangi adımda olduğunuzu ve yeni sürümün trafik payını gösterir. **Dağıtım → Geçmiş** sayfasında da **Canary ilerleyişi** ve iki sürümün (**Stable** ve **Canary**) payları görünür.

Adımlar yeni kod sürümleri için uygulanır. Ortam değişkenlerini **Değişiklikleri Uygula** ile uyguladığınızda Komuta değişikliği adımları beklemeden doğrudan canlıya alır.

Canary için ayrı bir önizleme adresi yoktur; yeni sürüm, servisin normal adresine gelen trafiğin bir kısmını alır. Yeni sürüm hata durumundaysa Komuta onu tek seferde canlıya almayı reddeder; önce **İptal Et** ile geri dönmeniz gerekir.

> **Uyarı:** Canary ağırlıkları ve Blue-Green geçişi, servisin genel adreslerine (Komuta adresi ve eklediğiniz alan adları) gelen trafik için geçerlidir. Komuta API Gateway üzerinden gelen istekler bu ağırlıklara uymayabilir ve henüz canlıya alınmamış yeni sürümün pod'larına da ulaşabilir. Aynısı **Bağlantılar** ile özel ağdan gelen istekler için de geçerli olabilir. Servisinizi [API Gateway](api-gateway.md) arkasında yayınlıyorsanız gateway trafiğinin yeni sürüme kademeli geçtiğine güvenmeyin.

---

## Stratejiyi Seçme ve Değiştirme

1. Servisin **Genel Bakış** sayfasında **Rollout durumu** kartındaki **Stratejiyi değiştir** simgesine tıklayın. Aynı pencereyi **Dağıtım → Geçmiş** sayfasında, **Mevcut sürüm** bölümündeki **Strateji değiştir** ile de açabilirsiniz.
2. **Rollout stratejisini değiştir** penceresinde **Blue-Green**, **Auto Promote** ya da **Canary**'yi seçin. Servisin şu anki stratejisi **Mevcut** olarak işaretlidir.
3. **Stratejiyi uygula**'ya tıklayın.

"Rollout stratejisi güncellendi. Bir sonraki deploy'da geçerli olacak." mesajını görürsünüz. Yeni strateji, servisin bir sonraki dağıtımında kullanılır. O an devam eden bir rollout varsa etkilenmez, mevcut stratejisiyle tamamlanır.

Stratejiyi değiştirmek için servisi düzenleme yetkiniz olmalıdır. Yetkiniz yoksa simge görünmez.

Servisi bir stack ile oluşturuyorsanız stratejiyi manifestte servisin `rollout.strategy` alanıyla seçersiniz: `auto-promote`, `blue-green` ya da `canary`. Alan yazılmazsa `auto-promote` kullanılır. Ayrıntılar için [Stack Manifest Referansı](stack-manifest-reference.md) sayfasına bakın.

---

## Rollout Konsolu ve İşlemler

Bir rollout sürerken **Genel Bakış** sayfasındaki durum kartında **Rollout Konsolu** görünür. Konsol stratejinin adını ve rollout'un nerede olduğunu gösterir:

| Strateji | Konsolda görünen aşamalar |
|---|---|
| **Blue-Green** | **Oluşturuluyor** → **Hazır** (ya da **Duraklatıldı**) → **Yükseltiliyor** → **Aktif** |
| **Auto Promote** | **Oluşturuluyor** → **Hazır** → **Aktif** |
| **Canary** | %0'dan %100'e adım adım ilerleyen trafik çubuğu (%20, %40, %60, %100) |

Aynı işlemler iki yerden yapılır: **Genel Bakış** sayfasındaki **Rollout Konsolu** ve **Dağıtım → Geçmiş** sayfasındaki işlem çubuğu. Düğme adları iki yerde biraz farklıdır:

| Rollout Konsolu | Geçmiş sayfası | Ne yapar | Ne zaman kullanılır |
|---|---|---|---|
| **Yükselt** | **Canlıya al** | Yeni sürüme trafiğin tamamını verir. Blue-Green'de önizlemeyi canlıya alır, Canary'de kalan adımları atlar. | Blue-Green ve Canary'de rollout sürerken. Auto Promote'ta konsolda görünmez, geçiş kendiliğinden yapılır. |
| **Devam Et** | **Devam** | Duraklatılmış rollout'u kaldığı yerden sürdürür. Canary'de bir sonraki adıma geçer. | Rollout duraklatılmışken. Konsolda yalnızca Canary'de görünür. |
| — | **Duraklat** | Rollout'u olduğu yerde bekletir. | Rollout sürerken. Yalnızca Geçmiş sayfasında vardır. |
| **İptal Et** | **Durdur** | Rollout'u iptal eder ve yeni sürümün pod'larını kaldırır. Trafik eski sürümde kalır ya da eski sürüme döner. | Rollout tamamlanmadan önce. |
| — | **Yeniden dene** | İptal edilmiş rollout'u aynı sürümle yeniden başlatır. | Yalnızca iptal edilmiş bir rollout varken. |
| **Yeniden Başlat** | **Yeniden başlat** | Çalışan sürümün pod'larını yeniden oluşturur. Sürüm değişmez. | Konsolda rollout sürerken; Geçmiş sayfasında her zaman. |

- **Rollout Konsolu** düğmeleri yalnızca servisi düzenleme yetkisi olan kullanıcılara görünür. **İptal Et** ve **Yeniden Başlat** önce onay ister: "Rollout iptal edilsin mi?" ve "Uygulama pod'ları yeniden başlatılsın mı?".
- **Yeniden dene** sürümü değiştirmez. Rollout yeni sürümdeki bir hata yüzünden iptal edildiyse aynı sonuçla karşılaşırsınız; önce hatayı düzeltip yeni bir sürüm dağıtın.
- Yeniden başlatma sırasında yeni pod'lar hazır olana kadar canlı trafik kısa süreli hata alabilir.

> **Uyarı:** **Dağıtım → Geçmiş** sayfasındaki işlem çubuğu onay sormaz; **Durdur** ve **Yeniden başlat** tıkladığınız anda çalışır.

Önceki bir sürüme dönmek için **Dağıtım → Geçmiş** sayfasındaki dağıtım kayıtlarında bulunan **Geri al** işlemini kullanın. Geri alma, seçtiğiniz sürümü yeniden dağıtır ve onay beklemeden canlıya alır; devam eden bir rollout varsa iptal edilir. Ayrıntılar için [Dağıtım Geçmişi](service-deployment-history.md) sayfasına bakın.

Bu işlemlerin hepsini (duraklatma, devam, canlıya alma, iptal, yeniden deneme, geri alma, yeniden başlatma) kod ajanınızdan Komuta MCP ile de yapabilirsiniz. Bkz. [Komuta MCP](mcp-tools.md) ve [MCP Kurulumu](mcp-setup.md).

---

## Yeni Sürüm Başarısız Olursa

Komuta, canlıya alınmış bir sürümü kendiliğinden önceki sürüme geri almaz. Yeni sürüm canlıya geçmeden önce sorun çıkarırsa ise eski sürüm trafiği karşılamaya devam eder.

| Durum | Ne olur | Ne yapmalısınız |
|---|---|---|
| Blue-Green ya da Auto Promote'ta yeni sürümün pod'ları hazır olmuyor | Trafik yeni sürüme geçmez, eski sürüm çalışmaya devam eder. Komuta sağlıksız önizlemeyi canlıya almayı reddeder. | Hatayı düzeltip yeni sürümü dağıtın ya da **İptal Et** ile önizlemeyi temizleyin. |
| Canary'de yeni sürüm hata durumunda | Komuta yeni sürümü tek seferde canlıya almayı reddeder. | **İptal Et** ile trafiği eski sürüme döndürün. |
| Yeni sürümün pod'ları sürekli çöküyor, eski sürüm sağlıklı | [Çökme koruması](#çökme-koruması) yeni sürümü iptal eder; eski sürüm yayında kalır. | Hatayı düzeltip yeniden dağıtın. |
| Rollout zaman aşımı içinde tamamlanmıyor | [Rollout Bekçisi](#rollout-bekçisi) bildirim gönderir ve ayarınıza göre rollout'u iptal edebilir. | Bildirimdeki servisi açıp rollout'u inceleyin. |
| Canlıya alınmış sürüm sürekli çöküyor ve servisin hiçbir pod'u hazır değil | [Çökme koruması](#çökme-koruması) servisi sıfır replikaya indirir. | Hatayı düzeltip yeni sürümü dağıtın ya da **Geri al** ile önceki bir sürüme dönün. |

---

## Rollout Bekçisi

Rollout Bekçisi, tamamlanamadan takılı kalan dağıtımlarda Komuta'nın ne yapacağını belirleyen organizasyon ayarıdır. Organizasyondaki tüm servislere uygulanır.

Ayara **Hesap → Organizasyon → Rollout Bekçisi** yolundan ulaşırsınız: `https://console.komuta.io/account/rollout-watchdog`

| Alan | Ne belirler |
|---|---|
| **Özel politika** | Kapalıyken platform varsayılanı uygulanır: "Kapalı = platform varsayılanı (10 dakika sonra yalnızca bildir)." Açınca aşağıdaki iki alanı siz belirlersiniz. |
| **Takılma zaman aşımı (saniye)** | 60 ile 86400 saniye (24 saat) arasında bir değer. Bu süre içinde tamamlanmayan rollout takılı sayılır. |
| **Müdahale eylemi** | **Yalnızca bildir**: Dağıtım zaman aşımına uğramış olarak işaretlenir ve bildirim gönderilir, servise dokunulmaz. **Rollout'u iptal et**: Takılı rollout otomatik iptal edilir, eski sürüm trafiği karşılamaya devam eder; Canary'de trafik tamamen eski sürüme döner. |

Değişiklikleri **Değişiklikleri Kaydet** ile kaydedin.

Bekçi şu kurallarla çalışır:

- Takılı sayılan rollout, ne tamamlanmış ne de onayınızı bekleyen rollout'tur. Blue-Green'in canlıya alma onayı ya da Canary'nin bir adımda beklemesi takılma sayılmaz.
- Takılan her dağıtım için tek bir bildirim gönderilir. **Bildirimler ve Uyarılar** sayfasındaki olay listesinde bu olayın adı **Dağıtım takıldı**'dır; bkz. [Kanallar ve Bildirim Teslimi](notification-guide.md).
- Otomatik iptal yalnızca eski bir sürüm hâlâ çalışıyorsa yapılır. Servisin ilk dağıtımında geri dönülecek bir sürüm olmadığı için Komuta iptal etmez, yalnızca bildirir.
- Aynı dağıtım en fazla bir kez otomatik iptal edilir.

Yeni sürüm uzun süre hiç hazır olamazsa Komuta dağıtımı başarısız sayar ve aynı ayarı uygular. Bu durumda tek fark şudur: Organizasyonunuzda bu ayar hiç kaydedilmediyse ve eski sürüm hâlâ çalışıyorsa Komuta yeni sürümü otomatik iptal eder. **Özel politika** kapalı olarak kaydedildiyse yalnızca bildirim gönderilir.

> **Not:** Sayfayı görmek için Rollout Bekçisi'ni görüntüleme izniniz olmalıdır. Değiştirme izniniz yoksa sayfa salt okunur açılır ve "Salt okunur — bu politikayı değiştirmek için Rollout Müdahalesini Yönet iznine ihtiyacınız var." yazar.

---

## Çökme Koruması

Bir servisin uygulama container'ı arka arkaya çöküp yeniden başladığında Komuta, belirli sayıda yeniden başlatmadan sonra devreye girer. Yalnızca uygulamanızın kendi container'ındaki çökmeler sayılır; Komuta'nın pod'a eklediği hazırlık adımlarındaki hatalar bu korumayı tetiklemez.

Komuta iki durumu birbirinden ayırır:

| Durum | Komuta ne yapar | Durum kartında görünen |
|---|---|---|
| Yeni sürümün pod'ları çöküyor, eski sürüm sağlıklı çalışıyor | Yalnızca yeni sürümü iptal eder. Eski sürüm trafiği karşılamaya devam eder. | **Yeni dağıtım başarısız oldu; kararlı sürüm yayında tutuldu** |
| Servisin hiçbir pod'u hazır değil ve pod'lar sürekli çöküyor | Servisi sıfır replikaya indirir. Servis, düzeltilmiş bir dağıtım yapılana kadar istek karşılamaz. | **Arka arkaya restart nedeniyle servis otomatik durduruldu** |

Bu kart **Genel Bakış** sayfasındaki durum kartında görünür ve şunları içerir:

- Çöken pod'un adı, hata nedeni ve yeniden başlatma sayısı,
- **Tespit edilen neden** ve **Son çökme logu**,
- Durdurulmadan önceki dönemin loglarını açan **Durdurulma öncesi logları aç** bağlantısı,
- Ne yapmanız gerektiği ve korumanın ne zaman devreye girdiği.

Koruma devreye girdiğinde bildirim de gönderilir. **Bildirimler ve Uyarılar** sayfasındaki olay listesinde adı **Tekrarlanan çökmeler sonrası servis durduruldu**'dur. Olay ayrıca durum kartındaki **Yeniden başlatma geçmişi**ne yazılır.

**Servisi geri getirmek için:**

1. **Son çökme logu**nda ya da **Durdurulma öncesi logları aç** ile açılan loglarda hatanın nedenini bulun.
2. Hatayı düzeltin ve değişikliği kaydedin.
3. Servisin yeni sürümünü dağıtın.

Yeni sürüm dağıtıldığında koruma kendiliğinden kalkar ve **Yeniden başlatma geçmişi**nde "Çökme koruması kaldırıldı — yeni bir sürüm dağıtıldı" kaydı görünür. Yeni sürüm de sürekli çökerse koruma yeniden devreye girer.

> **Uyarı:** Aynı sürümü yeniden denemek, korumayı kaldırmanın güvenilir bir yolu değildir. Hata sürümün kendisindeyse aynı sürüm yine çöker ve koruma yeniden devreye girer; değişmemiş bir sürümü yeniden dağıtmak da servisi durdurulmuş halde bırakabilir. Servisi geri getirmek için hatayı düzeltip yeni bir sürüm dağıtın.

---

## İlgili Dokümanlar

- [Dağıtım Geçmişi](service-deployment-history.md) — Yayına alınan sürümler, rollout işlemleri ve geri alma.
- [Servis Dashboard](service-dashboard.md) — Rollout durumu kartı, durum kartı ve topoloji.
- [Sağlık Kontrolleri](services-health-probes.md) — Yeni sürümün ne zaman "hazır" sayılacağını belirleyen kontroller.
- [Pipeline'lar](service-pipeline-guide.md) — Yeni sürümü üreten derleme ve yayınlama süreci.
- [API Gateway](api-gateway.md) — Gateway arkasındaki servisler.
- [Kanallar ve Bildirim Teslimi](notification-guide.md) — Takılan dağıtım ve çökme koruması bildirimlerinin gideceği yerler.
- [Komuta MCP](mcp-tools.md) — Rollout işlemlerini kod ajanınızdan yapmak.
