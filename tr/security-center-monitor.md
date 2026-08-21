# İzle: Genel Bakış ve Bulgular

Güvenlik Merkezi'nin "İzle" grubu, kiracınızdaki güvenlik telemetrisini canlı olarak izlemenizi ve tekil bulgular (finding) üzerinde inceleme/aksiyon almanızı sağlayan iki sayfadan oluşur: **Genel Bakış** ve **Bulgular**.

---

## Genel Bakış

Genel Bakış sayfası, kiracınızın güvenlik durumunu tek ekranda özetler: risk skoru, telemetri kapsamı (coverage), açık bulgular, koleksiyon (collector) sağlığı ve "bugün yapılması gerekenler" listesi.

### Sayfa düzeni

Sayfa, kiracınızın çalışma zamanı (host-runtime) telemetrisine ne kadar erişebildiğine göre iki farklı düzende render edilir:

| Kiracı tipi | Davranış |
|---|---|
| Normal (host-runtime tespiti etkin veya kısmi kümelerde) | Kapsam kartı → Bugünün işleri → KPI satırı (Skor, İzle, Tespit Et, Engelle) → "Engelle" bölümü → "Tespit Et" bölümü → Hızlı aksiyonlar |
| Yalıtılmış çalışma zamanı (host-runtime tespiti uygulanamaz, en az 1 küme var) | Bugünün işleri → KPI satırı (Skor, İzle — Tespit Et/Engelle gizli) → Bulgular bölümü → Ağ tehditleri → Risk odağı → Kapsam kartı → Uyumluluk kısayolları |

Yalıtılmış çalışma zamanı, iş yüklerinizin paylaşımlı bir platform kümesinde çalıştığı ve platformun host-runtime sensörünün bu kümeyi gözlemleyemediği anlamına gelir. Bu durumda "Tespit Et" ve "Engelle" KPI kutuları ve ilgili host-runtime kartları (Tespit Performansı, Çalışma Zamanı Engelleme) hiç gösterilmez — sıfır değerli, kapsam boşluğunu ima eden kutucuklar göstermek yerine bu bölümler tamamen kaldırılır.

### Telemetri kapsamı kartı

Bu kart, platformun üç güvenlik katmanını (pillar) ne ölçüde kapsadığını dürüst biçimde özetler:

| Katman | Kapsam durumu |
|---|---|
| Ağ tespiti (Network Detection) | Her iş yükü kiracısı için her zaman etkin |
| İş yükü yalıtımı (Workload Isolation) | Her iş yükü kiracısı için her zaman etkin |
| Host-runtime tespiti | Kiracının kümesine göre değişir: **Etkin**, **Kısmi** veya **Uygulanamaz** |

Host-runtime tespit rozeti üç durumdan birini gösterir:

| Rozet | Anlamı |
|---|---|
| Etkin | Kiracının tüm kümeleri host tarafından gözlemlenebiliyor |
| Kısmi | Kümelerin bir kısmı gözlemlenebiliyor, bir kısmı değil |
| Uygulanamaz | Kiracı yalıtılmış (paylaşımlı) bir platformda çalışıyor — bu katman bu kiracı için mimari olarak geçerli değil |

> **Not:** "Uygulanamaz" durumu bir hata veya eksik yapılandırma değildir. Kiracının çalışma modeliyle ilgilidir ve host-runtime tespitinin bu kiracı için hiçbir zaman devreye girmeyeceği anlamına gelir; ağ tespiti ve iş yükü yalıtımı katmanları yine de etkindir.

### KPI satırı

**Risk skoru** kartı, kiracınızdaki en riskli servislerin risk skorlarının ortalamasını gösterir (0 en düşük risk, 100 en yüksek risk). Skor **açıklanabilirdir** — kara kutu değildir; skorun üzerine gelindiğinde hangi etkenlerin skoru nasıl etkilediği ayrı ayrı gösterilir:

- **Bulgu şiddeti:** Kritik bulgular skoru en çok yukarı çeker, Bilgi düzeyindeki bulgular en az etkiler.
- **Kaynak güvenilirliği:** Çekirdek (kernel) seviyesindeki gözlemler, duruş (posture) kontrollerine göre daha yüksek güvenle değerlendirilir.
- **Tazelik:** Eski bulguların skora etkisi zamanla azalır.
- **Etki alanı:** Servisin kritiklik seviyesi, internete açık olması ve ayrıcalıklı erişim kullanımı skoru yukarı çeker.
- **Tekrar faktörü:** Aynı bulgunun tekrar sayısı skora ek katkı yapar; bu katkının etkisi sınırlıdır, tek başına skoru sürükleyemez.

Skor yükseldikçe kart daha çok dikkat çekecek şekilde renklenir, böylece hangi servislerin öncelikli ilgi gerektirdiği bir bakışta görülür.

**İzle / Tespit Et / Engelle** kutucukları, izleme döngüsünün üç aşamasını gösterir:

- **İzle:** kaç kaynağın (collector) canlı olduğu (örn. 3/4 sağlıklı) ve en son sinyalin ne kadar önce geldiği.
- **Tespit Et** (yalnızca host-runtime kapsamı olan kiracılarda): açık kritik bulgu sayısı ve tespit oranı (detection rate).
- **Engelle** (yalnızca host-runtime kapsamı olan kiracılarda): kaç ihlalin bugüne kadar engellendiği ve kaç bulgunun "engellemeye hazır" (block-ready) olduğu — yani kanıtından doğrudan bir çalışma zamanı engeli türetilebilecek, henüz uygulanmamış açık bulgu sayısı.

> **Not:** "Engellemeye hazır" sayacı, servis bazında çalışma zamanı engelleme kontrolünün açık olup olmadığına bakmaz — kontrol kapalı olsa da bulgu sayılır; aksiyonun kendisi (Engelle) bu durumu ayrıca ve dürüstçe bildirir.

### Bugünün işleri (Today's actions)

KPI satırının üzerinde, kiracı genelinde önceliklendirilmiş bir "bugün ne yapmalıyım" listesi yer alır. Bu liste şu sinyallerden, önem sırasına göre en fazla 5 satır üretir:

| Öncelik | Sinyal | Şiddet |
|---|---|---|
| 1 | Tetiklenmiş bir tuzak yol (honey path) *(yakında)* | Kritik |
| 2 | Açık kritik bulgu var | Kritik |
| 3 | Bir koleksiyon kaynağı tamamen sağlıksız/görünmez | Yüksek |
| 4 | Bir tatbikat (sentetik atak) tespit edilemedi veya hata verdi | Yüksek |
| 5 | Bir koleksiyon kaynağı bozulmuş (degraded) durumda | Dikkat |
| 6 | Kullanılmayan/etkin olmayan bir tatbikat senaryosu var | Bilgi |

Her satırda ilgili sayfaya giden bir aksiyon düğmesi bulunur. Hiçbir sinyal tetiklenmediğinde "Bugün dikkat gerektiren bir şey yok" boş durumu gösterilir.

> **Not:** 1. sıradaki tuzak yolu sinyali önceliklendirme mantığında yerini almış olsa da şu an devrede değildir — tuzak yolu tetiklenmelerinin kiracı geneli bir uç noktası henüz bulunmadığından bu sinyal bugün hiçbir zaman listede görünmez. Kiracı geneli veri eklendiğinde devreye alınacaktır.

> **İpucu:** Bir sinyal kaynağı (örneğin koleksiyon sağlığı) henüz yüklenmemişse veya sizin bu veriyi görme yetkiniz yoksa, ilgili satır listede **hiç görünmez** — sahte bir "her şey normal" mesajı asla gösterilmez. Bu yüzden liste boşsa, gerçekten görüntülenen tüm sinyaller açısından temiz olduğunuz anlamına gelir.

### Bulgular ve risk odağı bölümleri

> **Not:** Bu iki bölümün başlığı kiracı tipine göre değişir: host-runtime kapsamı olan (Normal) kiracılarda KPI kutularıyla aynı adı taşır — **"Engelle"** ve **"Tespit Et"**; yalıtılmış çalışma zamanı kiracılarında ise **"Bulgular"** ve **"Risk odağı"** olarak görünür. İçerikleri aşağıda aynı şekilde açıklanmıştır.

- **Kritik bulgular akışı:** açık/kabul edilmiş (Acknowledged) Kritik veya Yüksek şiddetli bulguların bir özeti. Kiracıda hiç açık Kritik/Yüksek bulgu yoksa bu kart tamamen gizlenir (boş kart göstermek yerine).
- **Çalışma zamanı engelleme kartı** (host-runtime kapsamı olan kiracılarda): ihlal özetini ve "engellemeye hazır" bulguların örnek bir listesini gösterir.
- **Tespit performansı kartı** (host-runtime kapsamı olan kiracılarda): tatbikat (sentetik atak) tespit oranını gösterir.
- **En riskli servisler kartı:** risk skoruna göre sıralı servis listesi; risk listesi hesaplanamadığında liste boş yerine "geçici olarak kullanılamıyor" durumunu gösterir.

### Ağ tehditleri kartı

Bu kart yalnızca yalıtılmış çalışma zamanı kiracılarında görünür (Normal kiracılarda gösterilmez) ve ağ katmanındaki iki farklı sinyali bir arada sunar:

- **Engellenen akış bulguları:** ağ katmanında düşürülen/engellenen trafiğin bulgu sayısı; tıklandığında bu kaynağa süzülmüş Bulgular sayfasına gider.
- **Açık ağ olayları (incident) listesi:** DDoS/anomali tespiti etkinse, son 4 açık olayın kısa listesi.

> **Not:** Yalıtılmış çalışma zamanı kiracılarının sahip olduğu bir küme bulunmadığından, bu kartta "Ağ merkezi"ne giden bağlantı ve sayısal "açık olay" kutucuğu gösterilmez — sadece son olayların kısa listesi görünür. DDoS/ağ olayı tespiti kiracınız için etkin değilse, liste yerine bunun neden gösterilmediğini açıklayan bir not gösterilir; engellenen akış bulguları bu durumda da toplanmaya devam eder, sadece bunlardan bir "olay" (incident) türetilmez.

### Neden böyle?

- **"Sıfır ihlal" hiçbir zaman tek başına yeşile boyanmaz.** Bir kaynak sağlıksızsa veya güven düzeyi yetersizse, "0 ihlal" durumu yeşil değil, nötr/bilinmeyen olarak gösterilir — aksi halde bir izleme kesintisi "her şey güvenli" gibi yanlış yorumlanabilir.
- **"Bilinmeyen" (Unknown) hiçbir zaman "sağlıklı" ile aynı gösterilmez.** Veri henüz gelmemişse bu, veri geldi ve sorun yoksa denenle asla karıştırılmaz.
- **Yalıtılmış çalışma zamanı tenant'larında host-runtime kartları tamamen kaldırılır**, sıfırlanmış olarak gösterilmez — çünkü bu platform mimarisinin kapatamayacağı bir boşluk değil, bilinçli bir tasarım kararıdır (paylaşımlı platform kümesi).

---

## Bulgular

Bulgular sayfası, kiracınızdaki **tüm kaynaklardan** (çalışma zamanı sensörleri, politika koruması, ağ görünürlüğü, duruş kontrolleri, kimlik olayları, saldırı zinciri motoru vb.) gelen güvenlik bulgularının tek, filtrelenebilir listesidir. Satırları seçip toplu yaşam döngüsü kararı (Onayla / İzin Ver / Tehdit Olarak İşaretle / Reddet / Çöz) verebilir, ya da tek bir bulgu için **Yanıtla** düğmesiyle daha ayrıntılı bir müdahale (çalışma zamanı engeli, istisna, yalıtım vb.) başlatabilirsiniz.

### Filtreler ve arama

| Filtre | Açıklama |
|---|---|
| Arama | Serbest metin arama (başlık/özet/nedene göre) |
| Zaman aralığı | Son 24 saat, son 7 gün (varsayılan), son 30 gün, tüm zamanlar |
| Kaynak | Bulgunun hangi tespit alanından geldiğine göre süzer (örn. çalışma zamanı, ağ, politika, kimlik, saldırı zinciri) |
| Servis | Kiracıdaki servislerden biri |
| Tür | Bulgu türü (örn. şüpheli çalıştırma, kimlik bilgisi sızıntısı) |
| Şiddet | Info / Low / Medium / High / Critical |
| Durum | Open / Acknowledged / Allowed / Blocked / Dismissed / Resolved |
| Engellenebilirlik | Engellenebilir / Uygulanmış / Engellenemez |

Üstteki istatistik kutucukları (Toplam / Açık / Kritik / Yüksek) aynı zamanda birer filtre kısayoludur — bir kutucuğa tıklamak ilgili filtreyi açar/kapatır.

> **İpucu:** Varsayılan zaman aralığı son 7 gündür. Bunun nedeni, sürekli açık kalan binlerce eski bulgunun listeyi doldurup güncel ve üzerinde çalışılabilir olanları gömmesini önlemektir. Daha eski bulguları görmek için aralığı "Tüm zamanlar" yaparak genişletebilirsiniz.

> **Not:** Tür ve engellenebilirlik filtreleri (eski sürüm sunucularda) yalnızca o an yüklü sayfayı süzer — sonraki sayfalarda eşleşen satırlar olabilir. Bu durumda arayüz "eşleşen sonuç bu sayfada yok" uyarısı ve "sonraki sayfayı dene" kısayolu gösterir.

### Tablo sütunları

| Sütun | İçerik |
|---|---|
| Yanıt | Seçim kutusu + **Yanıtla** düğmesi |
| Şiddet | Severity rozeti |
| Kaynak | Source rozeti |
| Servis *(bazı görünümlerde)* | Servis adına bağlantı |
| Tür | Bulgu türü rozeti + kısa başlık/özet |
| Güven | Confidence rozeti |
| Engel | Çalışma zamanı engelleme durumu |
| Sayaç | Bu bulgunun kaç kez gözlemlendiği |
| Durum | Yaşam döngüsü durumu |

### Şiddet seviyeleri

| Seviye | Anlamı |
|---|---|
| Critical | Hemen aksiyon al — aktif saldırı veya doğrudan ele geçirme riski işareti (bilinen saldırı aracı, kimlik bilgisi erişimi, tetiklenmiş tuzak, saldırı zinciri) |
| High | Ciddi etki — ayrıcalık yükseltme, hassas yollara yazma, konteyner kaçışı denemesi veya engellenen politika ihlalleri |
| Medium | Dikkate değer ama acil değil — doğrudan zarar kanıtı olmayan şüpheli davranış |
| Low | Düşük etkili gözlem — genelde bilgilendirme amaçlı; bir örüntüye dönüşürse önemli |
| Info | Denetim izi için kaydedilen bağlam — aksiyon beklenmez |

> **Not:** Şiddet, tespit kaynağının kurallarınca atanır ve **tekrar sayısı şiddeti asla yükseltmez**; operatör de şiddeti değiştiremez. Tespitin ne kadar güvenilir olduğu ayrı bir eksen olan Güven (Confidence) ile ölçülür.

### Güven (Confidence)

Güven düzeyi, bulgunun kendi şiddetinden ve kaç kez tekrarlandığından türetilir — sabit bir "bu kaynak her zaman güvenilirdir" eşlemesi kullanılmaz, çünkü bu tek seferlik bir olayı gerçekte olduğundan daha güvenilir gösterebilirdi.

| Seviye | Anlamı |
|---|---|
| Yüksek | Bu tespit büyük olasılıkla gerçek bir olay, yanlış alarm değil — doğru kabul edilebilir. Etkinliğin *güvenli* olduğu anlamına gelmez |
| Orta | Sinyal makul ama sınırlı destekleyici kanıta sahip — büyük olasılıkla gerçek, aksiyon almadan önce doğrulanmalı |
| Düşük | Zayıf veya tek seferlik bir sinyal — yanlış alarm olabilir. Aksiyon almadan önce araştırın; bu düzeydeki bildirimler bir kademe düşürülür |

> **Not:** "Yüksek güven" ifadesi, etkinliğin *güvenli* olduğu anlamına gelmez — tam tersini kastediyor: platformun tespitin *gerçek* olduğundan eminliğidir.

### Durum yaşam döngüsü

| Durum | Anlamı |
|---|---|
| Open | Henüz karar verilmedi — incelemeyi bekliyor |
| Acknowledged | Bir operatör bunu görüp inceliyor — henüz sonuçlanmadı |
| Allowed | Bir operatör bu davranışı güvenli/beklenen olarak işaretledi — kapalı; tekrarlar yeniden açmaz |
| Blocked | Bir operatör bunu tehdit olarak işaretledi; kanıt uygun olduğunda bir çalışma zamanı engeli de uygulanmış olabilir — kapalı |
| Dismissed | Gürültü veya ilgisiz olarak bir kenara bırakıldı — güvenlik kararı verilmedi — kapalı |
| Resolved | Sonuçlandırıldı — bu bulgu üzerinde başka aksiyon alınamaz; gerekiyorsa kanıtını dışa aktarın |

### Karar (decision) türleri

Toplu karar çubuğundan veya Yanıtla sihirbazından uygulanan beş karar vardır:

| Karar | Buton etiketi | Sebep gerekli mi? | Etkisi |
|---|---|---|---|
| Acknowledge | **Onayla** | Hayır (isteğe bağlı not) | Görüldü/inceleniyor olarak işaretler; politika etkisi yok |
| Allow | **İzin Ver** | Evet (4+ karakter) | Davranışı meşru olarak işaretler, kapatır |
| Block | **Tehdit Olarak İşaretle** | Evet (4+ karakter) | Onaylanmış tehdit olarak kaydeder, kapatır |
| Dismiss | **Reddet** | Hayır (isteğe bağlı kategori + not) | Gürültü olarak susturur, kapatır |
| Resolve | **Çöz** | Hayır (isteğe bağlı not) | Düzeltmenin tamamlandığını kaydeder, kapatır |

> **Uyarı:** "Tehdit Olarak İşaretle" (Block) kararı **sadece bulgunun durumunu değiştirir** — çekirdek düzeyinde hiçbir şeyi engellemez. Gerçek bir çalışma zamanı engeli uygulamak istiyorsanız **Yanıtla** sihirbazındaki ayrı "Çalışma zamanında engelle" aksiyonunu kullanmanız gerekir.

> **Uyarı:** Kararlar, değiştirilemez (tamper-evident/WORM) bir denetim izine yazılır ve **daha sonra düzenlenemez**. "İzin Ver" veya "Tehdit Olarak İşaretle" kararı verirken girdiğiniz sebep, kalıcı olarak kaydedilir.

Toplu karar çubuğu her zaman görünür değildir: sadece "İzin Ver" / "Tehdit Olarak İşaretle" / "Çöz" kararları, bulguları yönetme yetkisinin yanında ayrı bir "izin ver/engelle" yetkisi gerektirir. Bu yetkiye sahip olmayan roller (örn. geliştirici rolü) yalnızca Onayla / Reddet uygulayabilir; arayüz bunu satırın altında açık bir uyarı metniyle bildirir.

### Engel (çalışma zamanı engelleme) sütunu

Bu sütun, "bu bulguyu şu an çekirdek seviyesinde durdurabilir miyim?" sorusunu tek bakışta yanıtlar:

| Simge/durum | Anlamı |
|---|---|
| Yeşil kalkan | Bu bulgunun tam eşleşen çalışma zamanı engel kuralı zaten servis için yapılandırılmış |
| Kırmızı kalkan (buton) | Bir çalışma zamanı engeli uygulanabilir — tıklamak Yanıtla sihirbazını açar |
| Soluk kalkan | Engelleme desteklenmiyor; üzerine gelindiğinde nedeni gösterilir |
| Tire (—) | Bulgu, engellenebilirlik bilgisi taşımayan eski bir satır |

### Yanıtla sihirbazı

Bir bulguya **Yanıtla** düğmesiyle girildiğinde üç adımlı bir panel açılır:

1. **Bağlam:** bulgunun kimliği, kanıtı, uygulanabilir yanıt planı (aşağıya bakın), eşleşen müdahale kitapçıkları (playbook) ve bu bulgu için daha önce tetiklenmiş bildirim denemeleri (salt okunur).
2. **Yanıt:** bulgunun desteklediği somut çalışma zamanı aksiyonları + yaşam döngüsü kararı seçimi.
3. **Onayla:** özetin gözden geçirilip kararın uygulanması; yeni durum gösterilir.

> **Not:** Eşleşen müdahale kitapçıkları listesi sadece bilgilendirme amaçlıdır — buradan hiçbir şey "çalıştırılmaz"; bu sayfada bir "Çalıştır" düğmesi yoktur.

**Aksiyonlar listesi** yalnızca bu bulgu için gerçekten desteklendiği bildirilen aksiyonlardan türetilir — arayüz hiçbir aksiyonu kendiliğinden varsaymaz veya öngörmez:

| Aksiyon | Ne yapar |
|---|---|
| Çalışma zamanında engelle | Bulgudan uygulanan bir çalışma zamanı engel kuralı türetir ve servisi yeniden dağıtır. Engellenen yol çekirdek seviyesinde reddedilir — davranış meşruysa iş yükünü bozabilir |
| İstisna ekle | Bu davranışı beklenen olarak kaydeder, bir daha bulgu olarak görünmemesini sağlar; onay öncesi tam şekli (bastırma / izin listesi / kanıt filtresi) gösterilir |
| Önerilen politikayı onayla | Gözlemlenen davranıştan öğrenilmiş bir politika önerisini kabul edip etkin kurala yükseltir |
| Koruma modunu değiştir | İş yükünün koruma modunu (kapalı / gölge / denetim / uygula) değiştirir; uygula modu ihlal eden davranışı reddeder |
| Uygulama modunu değiştir | Çalışma zamanı sensörünü yalnızca gözlem modundan, kötü niyetli süreçleri etkin biçimde sonlandırma moduna geçirir; sadece desteklenen kümelerde |
| İş yükünü yalıtla | Servisi ağdan keser — erişilemez ve dışarı erişemez hale gelir. Kesinti, onayladığınız anda değil, ağ politikası kümeye yansıdığında etkin olur; onaylamadan önce tam olarak neyin kesileceğini görürsünüz ve dilediğiniz zaman geri alabilirsiniz |
| Bulguyu bastır | Bu bulgunun gürültü olarak görünmesini durdurur; hiçbir uygulama değişikliği yapmaz, sadece konsolun gösterdiğini etkiler |
| Kanıtı dışa aktar | SIEM/adli inceleme için maskelenmiş kanıt paketini dışa aktarır; salt okunur, bulguda hiçbir şey değişmez |

Her aksiyonun panelde üç eksende (Güven, Hazırlık, Etki riski) düşük/orta/yüksek olarak bir seviyesi gösterilir. Etki riski yüksek veya orta olan aksiyonlarda, onaydan önce ayrıca bir uyarı bandı gösterilir.

> **Uyarı:** "Çalışma zamanında engelle", "İş yükünü yalıtla" gibi uygulayıcı (enforcing) aksiyonlar canlı iş yükünü kesintiye uğratabilir. Panel her zaman "bu ne yapacak" önizlemesini gösterir ve — desteklendiği durumlarda — bir **Geri al** seçeneği sunar; ancak uygulanana kadar önizlemeyi dikkatle okuyun.

**Bulgu türüne göre önerilen karar:** Sihirbaz, bulguya eklenen rehberlik verisine göre bir karar önerisi ile açılır (örneğin kimlik bilgisi sızıntısında "Tehdit Olarak İşaretle" önerilir); operatör bunu değiştirebilir, sihirbaz hiçbir zaman otomatik uygulama yapmaz.

### Uygulanabilir yanıt planı

Her bulgu için sihirbazın Bağlam adımında beş alanlı bir rehber kutusu gösterilir:

| Alan | Cevapladığı soru |
|---|---|
| Sorumlu sahip | Bu bulguyla kim ilgilenmeli? (Servis sahibi / Güvenlik operatörü / Platform güvenlik ekibi / Platform operatörü / Olay komutanı) |
| Sıradaki aksiyon | Ne yapılmalı? |
| Göz ardı edilirse risk | Bu düzeltilmezse ne olabilir? |
| Doğrulanacak kanıt | Karar vermeden önce ne kontrol edilmeli? |
| Güvenli düzeltme yolu | Hangi sıra/yöntemle ilerlenmeli? |

Bu veri sistemden geldiğinde doğrudan kullanılır; bazı durumlarda arayüz, güvenli tarafta kalan genel bir metin gösterir.

### Saldırı zinciri (correlation) ve süreç ağacı görünümleri

Birden fazla sinyalin tek bir tehdit örüntüsüne bağlandığı **saldırı zinciri** bulgularında (kaynağı "Attack Chain" olan bulgular), Bağlam adımında:

- eşleşen kuralın kimliği ve güven skoru,
- zincire katkı yapan kaynaklar,
- zincirin zaman penceresi ve olay sayısı,
- (yetki varsa) zincirdeki tekil olayların çözülmüş listesi

gösterilir. Saldırı zinciri olmayan, süreç tabanlı bulgularda ise gözlemlenen tek süreç (ikili yol + argümanlar) bir "süreç ağacı" olarak gösterilir; bugünkü kanıt modeli düz olduğundan (üst-alt süreç zinciri henüz kaydedilmiyor) görünüm bunu açıkça bir bilgi notuyla belirtir.

Saldırı zinciri bulgularında ayrıca "Zinciri engelle" akışı kullanılabilir: önce bir önizleme gösterilir — kaç üyenin engelleneceği, kaçının zaten engellenmiş olduğu ve engellenemeyenlerin nedeni dürüstçe listelenir — ardından tek bir yeniden dağıtımla tüm engellenebilir üyeler birlikte uygulanır. Ağ katmanına ait zincir üyeleri bu akışın dışında tutulur; onlar için ayrı bir ağ politikası önerisi hazırlanır.

### Dışa aktarma (Export)

Yetkiniz varsa, sayfanın üstündeki dışa aktar düğmesi CSV veya JSON formatında bir kanıt paketi üretir. Dışa aktarma, ekrandaki **tüm filtrelenmiş sonuç kümesini** kapsar (tek sayfa değil) — arama, kaynak, tür, engellenebilirlik, zaman aralığı, şiddet, durum ve servis filtreleri dahil.

### Neden böyle?

- **Yaşam döngüsü kararı ile çalışma zamanı engeli kasıtlı olarak ayrı tutulur.** "Tehdit Olarak İşaretle" bir triyaj sonucu kaydeder; "Çalışma zamanında engelle" ise gerçek bir kernel kuralı uygular. Bu ayrım olmadan operatörler bir durum değişikliğini yanlışlıkla gerçek bir engelleme sanabilirdi.
- **Allow / Block için sebep zorunludur, Acknowledge / Dismiss / Resolve için değildir.** Denetim izinin (audit trail) anlamlı kalması için kalıcı, tersine döndürülemez kararların bir gerekçesi olmalı; "görüldü" veya "gürültü" gibi daha hafif kararlarda bu zorunluluk gereksiz sürtünme yaratırdı.
- **Güven skoru, sabit bir kaynak listesi değil bulgunun kendisinden türetilir.** Aksi halde tek seferlik bir olay, sırf belirli bir kaynaktan geldiği için yapay olarak "yüksek güvenilir" gösterilebilirdi.
- **"Engellemeye hazır" sayacı servisin engelleme kontrolünün açık olup olmadığına bakmaz.** Sayaç, "kanıttan bir engel türetilebilir mi" sorusuna dürüst bir cevap verir; kontrolün kapalı olması ayrı bir gerçektir ve aksiyonun kendisi bunu ayrıca bildirir.

### SSS

**Bir bulguyu yanlışlıkla "Tehdit Olarak İşaretle" ile kapattım, geri alabilir miyim?**
Bu bir yaşam döngüsü kararıdır ve denetim izine yazılır; durum değişikliği geri alınamaz şekilde kayıtlıdır. Ancak bu karar tek başına hiçbir çalışma zamanı engeli uygulamaz — eğer ayrıca bir çalışma zamanı engeli de uygulamışsanız (Yanıtla sihirbazından), o engel için **Geri al** seçeneği panelde ayrıca sunulur.

**"İzin Ver" dedikten sonra bulgu bir daha hiç görünmeyecek mi?**
Evet — İzin Ver kararı bulguyu kapatır ve aynı davranışın tekrarları bulguyu yeniden açmaz.

**Neden bazı bulgularda "Engel" sütununda tire (—) görüyorum?**
Bu, bulgunun engellenebilirlik yeteneği (capability) bilgisi taşımayan eski bir kayıt olduğu anlamına gelir; bu satırlar için engelleme durumu bilinmiyor.

**Kaynak filtresinde gördüğüm "Attack Chain" nedir?**
Bu, birden fazla sinyalin tek bir tehdit örüntüsüne bağlandığı, saldırı zinciri motoru tarafından türetilmiş bulgulardır.

---

## Kimler Kullanır?

| Sayfa / aksiyon | İlgilenen rol |
|---|---|
| Genel Bakış'ı görüntüleme | Tüm geliştirici ekip |
| Bulgular listesini görüntüleme | Tüm geliştirici ekip |
| Bulgularda Onayla / Reddet kararları | Geliştirici |
| Bulgularda İzin Ver / Tehdit Olarak İşaretle / Çöz kararları | Güvenlik operatörü |
| Çalışma zamanında engelleme, iş yükü izolasyonu, istisna ekleme ve önerilen politika onayı gibi müdahale aksiyonları | Güvenlik operatörü |

En iyi uygulama: geliştiricilere görüntüleme ve Onayla/Reddet yetkisi verin; İzin Ver/Tehdit Olarak İşaretle/Çöz gibi kalıcı kararları ve çalışma zamanı müdahalelerini güvenlik operatörleriyle sınırlayın.

---

## İlgili

- [Güvenlik Merkezi: Genel Bakış](./security-center-overview.md)
- [Kayıt: Denetim ve Giriş Aktivitesi](./security-center-record.md)
- [Koru: Politika ve Müdahale](./security-center-protect.md)
- [Doğrula: Uyumluluk ve Sağlık](./security-center-verify.md)
- [Yönet: Bildirim ve Saklama](./security-center-manage.md)
