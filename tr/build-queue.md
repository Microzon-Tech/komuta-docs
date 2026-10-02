# Build Kuyruğu

Komuta'da her deploy önce bir build ile başlar. Build'ler paylaşılan derleme kapasitesinde çalışır; o an başlayamayan build kaybolmaz, **build kuyruğuna** girer ve sırası gelince kendiliğinden başlar. Kuyrukta bekleyen bir build, siz iptal etmedikçe kuyruktan çıkarılmaz.

---

## Sıra Nasıl Belirlenir

Bir build'in hemen başlayıp başlamayacağına şu kurallar karar verir:

- **Aynı servisin build'leri aynı anda çalışmaz.** Servisin önceki build'i sürerken yeni build kuyrukta bekler ya da servisin ayarına göre eskisinin yerini alır (aşağıda "Yeni Bir Build Geldiğinde").
- **Hesap seviyenizin eşzamanlı build sınırı vardır.** Hesabınızda aynı anda çalışabilen build sayısı hesap seviyenize bağlıdır; sınır doluysa yeni build bir slotun boşalmasını bekler. Seviyeniz ve bir üst seviyeye nasıl geçeceğiniz **Hesap → Cüzdan** sayfasında görünür.
- **Kiracılar arasında adil sıra uygulanır.** Bir hesabın çok sayıda build'i kuyruktayken başka bir hesabın tek build'i öne geçebilir; hiçbir hesap kuyruğu tek başına tutamaz.
- **Derleme kapasitesi gerçek ölçüme göre açılır.** Build ancak onu taşıyabilecek boş kapasite olduğunda başlar.

| Hesap seviyesi | Eşzamanlı build | Kuyrukta en fazla |
|------|-----------------|-------------------|
| Starter | 1 | 5 |
| Verified | 1 | 10 |
| Basic | 2 | 20 |
| Pro | 4 | 30 |
| Scale | 8 | 50 |

---

## Bekleme Nedenleri

Kuyruktaki her build, neden beklediğini gösterir:

| Neden | Anlamı |
|-------|--------|
| Bu servisin önceki build'inin bitmesi bekleniyor | Aynı servisin bir build'i hâlâ çalışıyor. O bitince bu build başlar. |
| Planınızın build slotu bekleniyor | Hesabınızın eşzamanlı build sınırı dolu. Bir build bitince başlar; daha fazla eşzamanlı build için hesap seviyenizi yükseltebilirsiniz. |
| Build kapasitesinin boşalması bekleniyor | Platformdaki derleme kapasitesi o an dolu. Kapasite açıldığında başlar. |
| Boş bir build yuvası bekleniyor | Platform genelindeki eşzamanlı build sınırı dolu. |
| Build kapasitesi şu an okunamıyor, build'ler tek tek başlatılıyor | Kapasite geçici olarak ölçülemiyor; bu sürede build'ler tek tek başlatılır. |

Önceki build yalnızca temizlik adımındaysa ya da süre sınırını aştıysa servisi artık tutmaz; yeni build onu beklemeden başlayabilir.

---

## Yeni Bir Build Geldiğinde

Servisin bir build'i çalışırken aynı branch için yeni bir build gelirse ne olacağını servisin **Otomatik dağıtım** sayfasındaki **Yeni bir build geldiğinde** ayarı belirler. Ayar push, CI ve elle tetiklenen tüm build'ler için geçerlidir.

| Seçenek | Ne olur |
|---------|---------|
| **En yenisi kazanır** (varsayılan) | Çalışan eski build nazikçe iptal edilir ve yeni build hemen başlar. En son commit daha erken yayına çıkar; artık işe yaramayacak bir build için beklenmez. |
| **Sırayla** | Her build sonuna kadar çalışır; yeni build, öncekisi bitene kadar kuyrukta bekler. |

"En yenisi kazanır" yalnızca **aynı branch'teki, yeni build kuyruğa girmeden önce başlamış** build'leri iptal eder. Başka bir branch'in build'i ve hazır imaj dağıtımları etkilenmez. İptal edilen build, iptal anına kadar geçen süre kadar faturalanır ve denetim kaydına geçer. CI'dan tetiklenmiş bir deploy bu şekilde yerini yenisine bırakırsa durumu `superseded` olur (bkz. [CI'dan Deploy Tetikleme](ci-deploy.md)).

---

## Bekleyen Build'e Gelen Push'lar

Bir build kuyrukta beklerken aynı branch'e yeni push'lar gelirse ayrı ayrı build açılmaz. Gelen push'lar bekleyen build'e katılır ve build sırası geldiğinde **en güncel commit'i** derler. Kuyruk kartında kaç push'un katıldığı `+N` olarak görünür.

---

## Tahmini Başlama Süresi

Kuyruktaki build'ler için Komuta, ne zaman başlayacağını tahmin eder ("Yaklaşık 5 dk. içinde başlaması bekleniyor"). Tahmin şunlara dayanır:

- Servisin son başarılı build'lerinin tipik süresi.
- Hesabınızda o an çalışan build'lerin kalan süresi.
- Hesap seviyenizin eşzamanlı build sınırı ve kuyruktaki sıranız.

Build platform kapasitesini bekliyorsa tahmin gösterilmez; bu durumda başlama zamanı başka hesapların build'lerine bağlıdır ve güvenilir bir süre verilemez.

---

## Kuyruğu Nerede Görürsünüz

- **Pipeline'lar sekmesi:** "Sırada bekleyen build'ler" şeridi, şu an derlenen build'i ve sıradakileri soldan sağa gösterir. Yeni build'ler şeridin sağ ucuna eklenir. Bir karta tıklayınca bekleme nedeni, süre, tahmin ve iptal seçeneği açılır.
- **Servis listesi:** Kuyrukta build'i olan servisin durumunun altında **Kuyrukta #N** çipi görünür. Servisin bir build'i zaten çalışıyorsa çip **Sonraki build kuyrukta #N** olarak okunur.
- **Servis dashboard'u:** Durum rozeti kuyruktaki sırayı, bekleme nedenini ve tahmini başlama süresini gösterir.

Sıra numarası kendi hesabınızın kuyruğundaki sıradır; başka hesapların build'leri gösterilmez.

---

## Kuyruktaki Build'i İptal Etme

Servis üzerinde düzenleme yetkisi olan kullanıcılar kuyruktaki bir build'i iptal edebilir: **Pipeline'lar** sekmesinde kuyruk şeridindeki karta tıklayın ve **İptal et**'i seçin. İptal denetim kaydına geçer.

---

## Uzun Bekleme Bildirimi

Bir build 15 dakikadan uzun süre kuyrukta beklerse **Build kuyrukta uzun süre bekliyor** olayı oluşur. Bu olayı hangi kanala göndereceğinizi **Hesap → Bildirimler → Olay yönlendirme** ekranından seçersiniz. Her build için bildirim bir kez gönderilir.

---

## Faturalama

Kuyrukta geçen süre faturalanmaz. Build dakikası, derleme gerçekten başladığı andan bittiği ana kadar sayılır.

---

## İlgili Dokümanlar

- [Pipeline'lar](service-pipeline-guide.md) — Build'in aşamaları ve logları.
- [Otomatik Dağıtım](service-auto-deploy.md) — Push ile build tetikleme kuralları.
- [CI'dan Deploy Tetikleme](ci-deploy.md) — Kendi CI hattınızdan Komuta'yı çağırma.
