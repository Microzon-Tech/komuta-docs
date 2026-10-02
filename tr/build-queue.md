# Build Kuyruğu

Komuta'da her deploy önce bir build ile başlar. Build'ler paylaşılan derleme kapasitesinde çalışır; o an başlayamayan build kaybolmaz, **build kuyruğuna** girer ve sırası gelince kendiliğinden başlar. Kuyrukta bekleyen hiçbir build iptal edilmez.

---

## Sıra Nasıl Belirlenir

Bir build'in hemen başlayıp başlamayacağına şu kurallar karar verir:

- **Aynı servisin build'leri sırayla çalışır.** Bir servisin önceki build'i bitmeden yenisi başlamaz.
- **Hesap kademenizin eşzamanlı build sınırı vardır.** Hesabınızda aynı anda çalışabilen build sayısı hesap kademenize bağlıdır; sınır doluysa yeni build bir slotun boşalmasını bekler. Kademeniz ve bir üst kademeye nasıl geçeceğiniz **Hesap → Cüzdan** sayfasında görünür.
- **Kiracılar arasında adil sıra uygulanır.** Bir hesabın çok sayıda build'i kuyruktayken başka bir hesabın tek build'i öne geçebilir; hiçbir hesap kuyruğu tek başına tutamaz.
- **Derleme kapasitesi gerçek ölçüme göre açılır.** Build ancak onu taşıyabilecek boş kapasite olduğunda başlar.

| Hesap kademesi | Eşzamanlı build | Kuyrukta en fazla |
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
| Planınızın build slotu bekleniyor | Hesabınızın eşzamanlı build sınırı dolu. Bir build bitince başlar; daha fazla eşzamanlı build için hesap kademenizi yükseltebilirsiniz. |
| Build kapasitesinin boşalması bekleniyor | Platformdaki derleme kapasitesi o an dolu. Kapasite açıldığında başlar. |
| Boş bir build yuvası bekleniyor | Platform genelindeki eşzamanlı build sınırı dolu. |
| Build kapasitesi şu an okunamıyor, build'ler tek tek başlatılıyor | Kapasite geçici olarak ölçülemiyor; bu sürede build'ler tek tek başlatılır. |

Önceki build yalnızca temizlik adımındaysa ya da süre sınırını aştıysa servisi artık tutmaz; yeni build onu beklemeden başlayabilir.

---

## Bekleyen Build'e Gelen Push'lar

Bir build kuyrukta beklerken aynı branch'e yeni push'lar gelirse ayrı ayrı build açılmaz. Gelen push'lar bekleyen build'e katılır ve build sırası geldiğinde **en güncel commit'i** derler. Kuyruk kartında kaç push'un katıldığı `+N` olarak görünür.

---

## Tahmini Başlama Süresi

Kuyruktaki build'ler için Komuta, ne zaman başlayacağını tahmin eder ("Yaklaşık 5 dk. içinde başlaması bekleniyor"). Tahmin şunlara dayanır:

- Servisin son başarılı build'lerinin tipik süresi.
- Hesabınızda o an çalışan build'lerin kalan süresi.
- Hesap kademenizin eşzamanlı build sınırı ve kuyruktaki sıranız.

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
