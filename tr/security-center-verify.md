# Doğrula: Uyumluluk ve Sağlık

Doğrula grubu, güvenlik kontrollerinin gerçekten çalıştığını kanıtlamaya yarayan dört sayfayı içerir: algılama yeteneğini tatbikatlarla sınayan Sentetik Ataklar, build hattının SBOM/tarama/imza kanıtlarını gösteren Tedarik Zinciri, çerçeve bazlı kontrol kapsamını raporlayan Uyumluluk ve host altyapısının sağlık probu bulgularını listeleyen Küme Sağlığı.

---

## Sentetik Ataklar

Sentetik Ataklar sayfası, planlı veya elle tetiklenen tatbikatlarla algılama hattının kendi kendini nasıl sınadığını gösterir. Bir tatbikat, bilinen-kötü bir eylemi gerçekten tetikler; değerlendirici arka plan görevi bu eylemi, senaryonun SLA penceresi içinde ilgili bir güvenlik bulgusuyla eşleştirip çalıştırmayı sonuçlandırır.

> **Uyarı:** Bir tatbikatı tetiklemek gerçek bir eylemi başlatır — sayfa "bilinen-kötü bir eylem tetiklenir" şeklinde açıkça belirtir. Bu, izole bir demo değildir; hedef servis veya kümede gerçek bir etki oluşturan bir adımdır. Üretim iş yükleri üzerinde tetiklemeden önce hedef servis/küme seçimini ve zamanlamayı buna göre planlayın.

### Algılama SLA panosu

Sayfanın üstünde bir pano penceresi seçici bulunur: **Son 24 saat**, **Son 3 gün**, **Son 7 gün**, **Son 14 gün**, **Son 30 gün**. Seçilen pencereye göre aşağıdaki kutucuklar güncellenir:

| Kutucuk | Ne gösterir |
|---|---|
| Toplam tatbikat | Pencere içindeki tüm çalıştırma sayısı; bekleyen veya hatalı çalıştırma varsa ipucu satırında belirtilir |
| Algılama oranı | Sonuçlanan çalıştırmalar içinde "Algılandı" oranı |
| SLA karşılanma oranı | Algılanan çalıştırmalar içinde SLA süresi içinde kalanların oranı |
| Ortalama algılama süresi (TTD) | Algılanan çalıştırmaların ortalama algılanma süresi |

Panonun üzerinde ayrıca **Planlı tatbikat hazırlığı** kartı yer alır; bu kart planlı (otomatik) çalıştırmaların sağlık durumunu **Healthy**, **Partial** veya bilinmeyen bir durum olarak özetler, son başarılı planlı tatbikatın zamanını gösterir ve varsa nedenleri rozet olarak listeler.

Panonun altında bir **kategori ısı haritası** bulunur; her kategori için (aşağıdaki tablo) algılama oranı ve çalıştırma sayısı gösterilir:

| Kategori | Açıklama |
|---|---|
| Runtime process | Çalışma zamanı süreç tabanlı saldırı senaryoları |
| Runtime network | Çalışma zamanı ağ tabanlı saldırı senaryoları |
| Runtime file | Çalışma zamanı dosya sistemi tabanlı saldırı senaryoları |
| Identity / RBAC | Kimlik ve RBAC yetki yükseltme senaryoları |
| Policy bypass | Politika atlatma senaryoları |

### Senaryo kataloğu

**Senaryo kataloğu** sekmesi, kullanılabilir tüm tatbikat senaryolarını listeler. Her senaryo kartında şu bilgiler yer alır:

- Kategori rozeti (yukarıdaki tablodaki kategorilerden biri)
- Durum rozeti: **Aktif**, **Devre dışı** veya **Projektör bekliyor** (bu senaryonun algılama kaynağı henüz devreye alınmamış; etkinleştirilemez)
- Otomasyon rozeti: **Planlı** (otomatik çalışır) veya **Sadece manuel**
- Beklenen algılama kaynağı, beklenen bulgu türü ve SLA süresi (saniye)
- İzole çalışma zamanı ortamındaysanız ve senaryo host çalışma zamanı kaynağına dayanıyorsa: **Bu platformda algılanamaz** rozeti

Senaryoyu kataloğunda etkinleştirme/devre dışı bırakma anahtarı sadece **senaryo kataloğunu yönetme** yetkisi olan kullanıcılara görünür; bu yetki host-yöneticileriyle sınırlıdır (kiracı yöneticileri bu anahtarı göremez). Bir senaryo "Projektör bekliyor" durumundaysa ve zaten kapalıysa, anahtar açma yönünde kilitlenir — kapatma her zaman mümkündür.

> **Not:** Katalog düzeyindeki etkinleştirme/devre dışı bırakma yetkisi kasıtlı olarak host-yöneticilerine sınırlıdır: bir kiracı yöneticisinin bir tatbikatı platform genelinde susturabilmesi, güvenlik operasyon merkezi için görünmez bir kör nokta oluşturur.

### Tatbikat tetikleme

**Tatbikat başlat** düğmesi (yalnızca tatbikat tetikleme yetkisi olan kullanıcılara görünür) bir diyalog açar:

1. **Senaryo** seçin — sadece etkin, otomatikleştirilebilir ve (izole ortamdaysanız) bu platformda algılanabilir senaryolar listede görünür.
2. **Hedef servis kimliği** girin (boş bırakılırsa küme genelinde tatbikat çalışır).
3. **Hedef küme kimliği** girin (boş bırakılırsa kümeler arası kapsam kullanılır).
4. **Tatbikat başlat** düğmesine tıklayın.

Gönderim sonrası sistem "Bekliyor" durumunda bir çalıştırma kaydı oluşturur; değerlendirici arka plan görevi eşleşen bir bulgu geldiğinde (veya SLA'nın iki katı süre geçtiğinde bulgu gelmezse) çalıştırmayı sonuçlandırır.

Tatbikat tetikleme yetkisi kiracı düzeyinde verilebilir (host ve kiracı yöneticileri tarafından). İzole çalışma zamanı ortamında hiçbir algılanabilir senaryo kalmadıysa **Tatbikat başlat** düğmesi devre dışı görünür ve nedeni açıklayan bir not eklenir.

### Çalıştırma geçmişi

**Çalıştırma geçmişi** sekmesi geçmiş tatbikatları listeler. Her satırda kategori, sonuç ve zaman bilgisi bulunur:

| Sonuç | Anlamı |
|---|---|
| Bekliyor | Değerlendirici henüz sonuçlandırmadı |
| Algılandı | Beklenen bulgu SLA penceresi içinde geldi |
| Algılanmadı | Beklenen bulgu gelmedi |
| Hata | Tatbikat tamamlanamadı (zaman aşımı, devre dışı algılayıcı, yetki sorunu veya algılama hattının hazır olmaması gibi nedenlerle) |

Algılanan çalıştırmalarda ayrıca **SLA karşılandı** / **SLA aşıldı** rozeti ve gerçek algılama süresi (TTD) görünür.

---

## Tedarik Zinciri

Tedarik Zinciri sayfası, her build/pipeline çalıştırması için üç kanıt parçasını bir arada gösterir: build sürecinin ürettiği SBOM (yazılım envanteri) referansı, pushlanan imajın güvenlik açığı taraması ve şeffaflık günlüğüne kaydedilen imza. Amaç, "ne yayınlandı, taraması yapıldı mı, imzalandı mı?" sorusunu listeden çıkmadan yanıtlayabilmektir.

Tedarik Zinciri sayfası host çalışma zamanı gözlemine bağlı değildir; izole çalışma zamanı ortamında olsanız bile bu sayfa tam olarak çalışır, çünkü kanıtlar (SBOM, tarama, imza) build hattından gelir, çalışan iş yükünden değil.

### Filtreler ve durum makinesi

Üstteki **Durum** filtresi, bir build/pipeline çalıştırmasının kanıt sürecindeki aşamasına göre süzer: kanıt henüz gelmedi, SBOM üretildi, güvenlik açığı taraması tamamlandı, imza tamamlandı, tüm kanıtlar doğrulandı, ya da süreç (tarama veya imzalama aşamasında) başarısız oldu.

Servis filtresiyle listeyi tek bir servise indirebilir; ayrıca bir pipeline çalıştırma adına `?run=` bağlantısıyla derin bağlantı verilebilir — bu durumda liste yerine o tek çalıştırmanın kanıt kaydı gösterilir.

### Satır görünümü

Her satırda durum rozeti, önem derecesi sayaçları (Kritik/Yüksek/Orta/Düşük), üç kanıt etiketi ve bir dağıtım kabul rozeti yer alır:

- **SBOM** etiketi: SBOM mevcutsa yeşil, yoksa gri "SBOM yok".
- **İmza** etiketi: Rekor şeffaflık günlüğüne kayıtlı bir imza varsa yeşil "İmzalı · Rekor"; imzalama başarılı olduğu halde Rekor girdisi çözülemediyse turuncu "İmza doğrulanamadı"; imza hiç yoksa gri "İmzasız".
- **Önem derecesi sayaçları**: taranmadıysa "Taranmadı"; sayaçlar tüm sıfır ve rapor da temizse yeşil "Güvenlik açığı yok"; sayaçlar bilinmiyorsa gri "Tarandı" (detay için satırı açmak gerekir); iz bulunan ama sayaçlara yansımamış bulgular varsa turuncu "Güvenlik açığı bulundu".
- **Dağıtım kabul rozeti** (aşağıya bakın).

Bir satırı tıklayarak açtığınızda imza & şeffaflık günlüğü detayları, SBOM belgesi referansı ve varsa satır içi bir güvenlik açığı tarama tablosu (önem derecesine göre sıralı, en fazla 50 satır) görünür. Rapor 30 KB'ı aştığında satır içi gösterilmez, yalnızca harici depoya işaret eden bir referans gösterilir.

### Dağıtım kabul kararı

Her build artefaktının bir **dağıtım kabul kararı** vardır — bu, imajın dağıtım zamanı imza/köken doğrulama kapısından geçip geçmediğini gösterir:

| Karar | Anlamı |
|---|---|
| Admission not evaluated | Henüz dağıtım kapısına ulaşmadı |
| Admission verified | Tüm kriptografik kanıtlar geçti |
| Deploy blocked | Dağıtım engellendi |
| Exception applied | Sınırlı bir istisna uygulanmış |
| Audit-only cohort | Sadece denetim modunda değerlendirilen bir kohort |

Kartı açtığınızda üç kanıt satırının ayrı ayrı durumu görünür: **SBOM tasdiki** ve **SLSA kökeni** için doğrulandı / kayıtlı ama kriptografik doğrulama bekliyor / eksik; **tarama kapısı** için ise farklı bir durum kümesiyle engellendi / geçti / eksik.

Bir imaj **Deploy blocked** durumundaysa ve istisna yönetme yetkiniz varsa, **Bu değişmez imaj riskini kabul et** düğmesiyle sınırlı bir istisna kaydedebilirsiniz:

1. Riski, telafi edici kontrolü ve düzeltme sahibini açıklayan en az 16 karakterlik bir **risk kabul nedeni** yazın.
2. **İstisna süresini** seçin (1, 4, 8 veya 24 saat).
3. **Sınırlı istisnayı kaydet**'e tıklayın.

> **Not:** Bu istisna dağıtım kabulünü genel olarak devre dışı bırakmaz — yalnızca bu tek servisi ve bu değişmez imaj digest'ini, seçilen süre sona erene kadar geçirir. Kayıt; aktör, neden ve politika parmak iziyle birlikte WORM denetim zincirine eklenir.

Aktif bir istisnayı, istisna yönetme yetkiniz varsa **İstisnayı iptal et** ile geri alabilirsiniz; bu işlem de WORM denetim zincirine eklenir ve bir sonraki dağıtım denemesi tüm kanıtlar geçmediği sürece yeniden engellenir.

---

## Uyumluluk

Uyumluluk sayfası, "çerçeve → kontrol → kanıt" hiyerarşisinde bir uyumluluk kapsama panosu sunar. Desteklenen çerçeveler:

| Çerçeve |
|---|
| CIS Kubernetes |
| NIST SP 800-53 |
| SOC 2 |
| ISO/IEC 27001 |
| PCI DSS |

Her kontrol satırı aşağıdaki kapsam durumlarından birini taşır:

| Kapsam durumu | Anlamı |
|---|---|
| Met | Kontrol tam olarak karşılanıyor |
| Partially met | Kontrol kısmen karşılanıyor |
| Missing signal | Gerekli kanıt sinyali eksik |
| Not applicable | Kontrol bu ortam için geçerli değil |
| Not evaluated | Kontrol henüz değerlendirilmedi |

Bu sayfanın erişim yetkisi, diğer Security Center sayfalarından **tamamen bağımsızdır**.

> **Not:** Bu yetkinin ayrı tutulmasının nedeni, bir uyumluluk denetçisi rolünün Security Center'ın diğer hiçbir yetkisine (bulgular, tatbikatlar, tedarik zinciri vb.) sahip olmadan da yalnızca bu sayfayı görebilmesini sağlamaktır — denetçi erişimini en az yetki ilkesiyle sınırlamak için bilinçli bir tasarım kararıdır.

### Kapsam kayması ve izole çalışma zamanı

İş yükleriniz host'un gözlemleyemediği izole bir sanal makine çalışma zamanında çalışıyorsa, sadece host çalışma zamanı telemetrisiyle kanıtlanabilecek kontroller başarısız olarak değil **Not applicable** olarak gösterilir; ağ veya denetim kanıtına dayanan kontroller normal şekilde değerlendirilir. Bu şekilde yeniden sınıflandırılan satırlarda **İzole çalışma zamanı** rozeti görünür.

### Panonun bölümleri

- **Özet kutucukları**: Toplam, Karşılanan, Kısmi, Eksik sinyal, N/A, Bekleyen — aktif filtreye göre güncellenir.
- **Çerçeve kartları**: her çerçeve için bir renkli kapsam çubuğu ve kontrol tablosu (ID, kontrol başlığı, durum, kanıt kaynakları, son değerlendirme zamanı).
- **Kanıt kaynakları** rozetleri şu etiketlerle gösterilir: Policy suggestions (öğrenilmiş politika önerileri), Policy exceptions (politika istisnaları), Image scan (imaj taraması), Audit trail (denetim izi).

### Yeniden değerlendirme ve dışa aktarma

- **Re-evaluate Framework** düğmesi (yalnızca değerlendirme tetikleme yetkisi olan kullanıcılara görünür), açılır menüden bir çerçeve seçerek uyumluluk motorunu o çerçeve için elle çalıştırır. Büyük katalog setlerinde bu işlem uzun sürebilir; işlem sürerken düğme bir yükleniyor göstergesiyle bekler.
- **Export Evidence** düğmesi (yalnızca dışa aktarma yetkisi olan kullanıcılara görünür) aktif çerçeve filtresine göre **JSON** veya **CSV** formatında bir kanıt dosyası indirir.

Hiç değerlendirme çalıştırılmamışsa panoda "Henüz uyumluluk eşlemesi değerlendirilmedi" boş durumu görünür; yetkiniz varsa buradan doğrudan CIS Kubernetes değerlendirmesini tetikleyebilirsiniz.

---

## Küme Sağlığı

Küme Sağlığı sayfası, kümedeki günlük sağlık probu işinin ürettiği sentetik bulguları listeler. Bu prob günde bir kez çalışır ve disk kullanımı, dosya sistemi doldurma oranı veya veri alım hızı gibi altyapı sağlık göstergelerinden biri güvenli sınırın dışına çıktığında bir bulgu kaydı yazar. Her şey sınırlar dahilindeyse iş sessiz kalır — bu yüzden **boş sayfa, kararlı/sağlıklı durumun kendisidir.**

Bu sayfa sol menüde **sadece host/platform yöneticilerine görünür** — kiracı operatörlerinden gizlenir.

> **Not:** Bu sayfadaki bulgular kiracıya özgü değildir (kiracı kimliği taşımaz), kümenin alt yapısını izler — bir iş yükünü değil. Bu nedenle menü girişi kasıtlı olarak host-yöneticileriyle sınırlıdır ve bulgu tablosunda anlamsız kalacağı için **Servis** sütunu gösterilmez.

### Sayfa davranışı

- **Kaynak** filtresi bu sayfada sabittir ve değiştirilemez — sadece küme sağlığı bulgularını gösterir, bir operatörün yanlışlıkla görünümü kiracı bulgularına genişletmesini engeller.
- Önem derecesi ve durum filtreleri, arama kutusu ve sayfalama, standart Bulgular tablosuyla aynı şekilde çalışır.
- Üstteki istatistik çipleri (Toplam, Açık, Kritik, Yüksek) filtrelenmiş görünümdeki bulgu sayılarını özetler.

---

## SSS

**Bir sentetik atak tatbikatı üretim ortamımda gerçek bir etki yaratır mı?**
Evet. Tatbikat, bilinen-kötü bir eylemi gerçekten tetikler; bu izole bir demo değildir. Hedef servis/küme seçimini buna göre yapın ve tetiklemeden önce zamanlamayı planlayın.

**Uyumluluk sayfasını görebiliyorum ama diğer Security Center sayfalarına erişemiyorum, bu normal mi?**
Evet, normaldir. Uyumluluk sayfasının erişim yetkisi diğer tüm Security Center yetkilerinden bağımsızdır; bir denetçi rolü sadece bu yetkiyle sınırlı erişime sahip olabilir.

**Küme Sağlığı menüde neden görünmüyor?**
Bu sayfa sadece host/platform yöneticilerine gösterilir. Kiracı hesabınızla oturum açtıysanız menüde görünmemesi beklenen davranıştır.

---

## Kimler Kullanır?

| Sayfa / işlev | İlgilenen rol |
|---|---|
| Sentetik Ataklar — görüntüleme, çalıştırma geçmişi | Tüm geliştirici ekip |
| Sentetik Ataklar — tatbikat tetikleme | Güvenlik operatörü |
| Sentetik Ataklar — senaryo kataloğunu açma/kapatma | Platform yöneticisi |
| Tedarik Zinciri (görüntüleme, kanıt inceleme) | Tüm geliştirici ekip |
| Tedarik Zinciri — risk kabul istisnası kaydetme/iptal etme | Güvenlik operatörü |
| Uyumluluk (görüntüleme, kanıt dışa aktarma) | Denetçi (auditor) rolü |
| Uyumluluk — yeniden değerlendirme tetikleme | Güvenlik operatörü / Denetçi |
| Küme Sağlığı | Platform yöneticisi |

---

## İlgili

- [Güvenlik Merkezi: Genel Bakış](./security-center-overview.md)
- [İzle: Genel Bakış ve Bulgular](./security-center-monitor.md)
- [Kayıt: Denetim ve Giriş Aktivitesi](./security-center-record.md)
- [Koru: Politika ve Müdahale](./security-center-protect.md)
- [Yönet: Bildirim ve Saklama](./security-center-manage.md)
