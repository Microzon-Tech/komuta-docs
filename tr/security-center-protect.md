# Koru: Politika ve Müdahale

Güvenlik Merkezi'nin **Koru** grubu, aktif korumayı (politikalar, çalışma zamanı uygulaması) ve bir tehdit tespit edildiğinde alınacak aksiyonu (bloklar, tuzak dosyalar, olay müdahale playbook'ları) tek bir çalışma alanında toplar. Beş sayfadan oluşur: **Politikalar**, **Çalışma Zamanı Uygulama**, **Bloklar**, **Honey Paths** ve **Playbook'lar**.

---

## Politikalar

Politikalar sayfası iki sekmeden oluşur: canlı politikaların listelendiği **Politikalar** sekmesi ve öğrenilmiş kural önerilerinin incelendiği **Öneriler** sekmesi. Hangi sekmenin görüneceği, ekibinizdeki rolünüze göre değişebilir — bazı roller sadece Politikalar'ı, bazıları sadece Öneriler'i görür (ayrıntı için sayfanın sonundaki "Kimler Kullanır?" bölümüne bakın).

### Politikalar sekmesi

Bu sekme, ana çalışma zamanı koruma motoru olan **Policy Guard**'ın süreç, dosya ve ağ kurallarını tek bir tabloda listeler. Her satır iki kaynaktan birinden gelir:

| Kaynak | Açıklama | Düzenlenebilir mi? |
|---|---|---|
| **Platform** | Dağıtım hattı tarafından servisin güvenlik temeline göre otomatik üretilmiştir. | Hayır — salt okunur; değiştirmek için servisin güvenlik ayarlarına gidilir. |
| **Manuel** | Bu sayfadan bir operatör tarafından oluşturulmuştur. | Evet |

Her satırın durumu bir renkli nokta ile gösterilir:

| Durum | Anlamı |
|---|---|
| **Zorunlu kılınıyor (Enforcing)** | Politika kümede etkin ve uygulanıyor. |
| **Sapma (Drift)** | Politika veritabanında etkin olarak kayıtlı ama kümedeki hali beklenenden farklı. |
| **Eksik (Missing)** | Politika etkin olarak işaretli ama kümede bulunamıyor. |
| **Devre dışı (Disabled)** | Operatör politikayı kapatmış; küme ile senkronize edilmiyor. |
| **Burada etkisiz (Not effective here)** | Politika teknik olarak var ama bu kümede hiçbir etkisi yok (aşağıdaki "Neden böyle?" bölümüne bakın). |

Kind (Tür) filtresinde üç seçenek görünür: **Policy Guard** (bugün gerçek satırları olan tek tür — süreç/dosya/ağ kuralları), **Network Policy** ve **Runtime Sensor** (ikisi de "yakında" etiketiyle listelenir; şimdilik seçildiklerinde tablo boş gelir ve bunun bir arıza değil, henüz teslim edilmemiş bir özellik olduğunu açıklayan bir bilgi kutusu gösterilir).

Kaynak, Tür, Aksiyon (İzin ver / Engelle / Denetle) ve Durum filtreleri serbestçe birleştirilebilir; arama kutusu ad, servis ve ad alanına (namespace) göre eşleşir.

Satır başına aksiyonlar (yalnızca **Manuel** kaynaklı satırlarda etkindir):

1. Satırdaki anahtarla politikayı **etkinleştirin veya devre dışı bırakın** — devre dışı bırakılan bir politika veritabanında kalır ama kümeye hiç gönderilmez.
2. Üç nokta menüsünden **Konuşlandır (Deploy)** veya **Konuşlandırmayı kaldır (Undeploy)** seçin.
3. **Düzenle** ile açılan panelde ad, açıklama, kapsam (Workload / Host / Cluster), aksiyon, önem derecesi (1–10), ad alanı, küme, servis ve ham YAML gövdesini güncelleyin.
4. **Sil** ile politikayı hem kümeden hem veritabanından kaldırın.

> **Uyarı:** Bir politikayı silmek onu hem kümeden hem veritabanından kaldırır ve geri alınamaz.

**Yeni politika** düğmesi aynı düzenleme panelini boş bir şablonla açar; kaydetmeden önce YAML gövdesi görülebilir ve elle düzenlenebilir.

Sayfanın üstündeki **Tümünü senkronize et (Reconcile all)** düğmesi, tüm kiracılardaki tüm yönetilen servisleri dağıtım kuyruğuna yeniden ekler; her servisin manifestleri (rollout, çalışma zamanı koruma politikası, kustomization) güncel koda göre yeniden üretilip dakikalar içinde canlı kümelere senkronize edilir. Bu, dağıtım hattında bir değişiklik yapıldıktan sonra bu değişikliğin canlı kümelere yayılmasını sağlamak için kullanılır.

> **Uyarı:** "Tümünü senkronize et" her kiracıdaki her servisi yeniden dağıtım kuyruğuna alır — kapsamı tek bir servis veya kiracı değil, platformun tamamıdır. Yalnızca bir dağıtım hattı değişikliğinin canlı kümelere yayılması gerektiğinde kullanın.

> **Not:** İzole (sanal makine tabanlı) bir çalışma zamanında bu sekmenin üstünde bir bilgi kutusu görünür: süreç ve dosya kuralları burada hiç etkili olmaz, çünkü platform bu iş yükleri için ana makine çekirdeğine erişemez. Bu durumda **Yeni politika** ve **Tümünü senkronize et** düğmeleri devre dışı kalır; ağ segmentasyonu kuralları ise etkilenmeden çalışmaya devam eder.

### Öneriler sekmesi

Bu sekme, gözlenen davranıştan veya bir müdahale aksiyonundan öğrenilmiş **bekleyen kural önerilerini** listeler — örneğin bir ağ olayından hazırlanmış bir "reddet" kuralı veya bir zincir müdahalesinden hazırlanmış bir iş yükü kuralı. Bazı roller öneriyi sadece inceleyebilir, bazıları ayrıca kabul/reddedebilir.

Her satır şu bilgileri taşır:

| Alan | Açıklama |
|---|---|
| **Tür** | Kuralın etki ettiği eksen: İş yükü kuralı, Ağ kuralı veya Çalışma zamanı kuralı. |
| **Güven düzeyi** | Düşük / Orta / Yüksek — öneri motorunun bu kuralı ne kadar güvenle çıkardığı. |
| **Hedef** | Belirli bir servis, belirli bir küme veya "Herhangi biri". Hedef bir servise bağlıysa ama adı çözülemiyorsa bu açıkça "adı bulunamadı" olarak gösterilir — asla sessizce "Herhangi biri"ne düşürülmez. |
| **Önerildi** | Önerinin üretildiği zaman. |

Bir öneriyi **İncele** ile açtığınızda değişiklik önizlemesi (kümedeki gerçek kaynakla karşılaştırmalı fark) yüklenir. Önizleme henüz yüklenmemişse, yüklenirken hata almışsa veya kümedeki karşılığı bulunamamışsa **Kabul et ve uygula** düğmesi devre dışı bırakılır — bir operatörün ne değişeceğini görmeden bir öneriyi onaylaması engellenir.

> **Not:** Bir öneri uygulanmaya çalışılıp başarısız olursa "Uygulama başarısız" durumuna geçer; bu durumda satır listede kalır ve **Tekrar dene** düğmesi belirir. Bir öneriyi reddederken isteğe bağlı bir gerekçe girilebilir.

> **İpucu:** "N önerinin M'si gösteriliyor" uyarısı çıkarsa, kalan öneriler kuyruk boşaldıkça (mevcutlar kabul/reddedildikçe) otomatik olarak görünür — sayfa hiçbir öneriyi sessizce gizlemez.

### Alt not: İhlaller sayfası

"İhlaller" artık ayrı bir menü öğesi değil; daha önce bu sayfaya kaydedilmiş eski bağlantılar ve üst menü çubuğundaki güvenlik göstergesi (kalkan simgesi), aynı bilgiyi gösteren Bulgular sayfasına sizi otomatik olarak yönlendirir.

---

## Çalışma Zamanı Uygulama

Çalışma Zamanı Uygulama sayfası, bir kiracının çalışma zamanı tehdit tespitine nasıl tepki vereceğini belirler: sadece gözlemleyip bulgu mu üretsin, yoksa eşleşen şüpheli süreçleri doğrudan sonlandırsın mı? Modu değiştirmek platform yöneticisine özel bir işlemdir; diğer roller güncel modu görebilir ama değiştiremez. Sayfa, çalışma zamanı sensörü kurulu kümelerde kullanılabilir.

| Mod | Ne yapar | Kim değiştirebilir |
|---|---|---|
| **Sadece gözlem (Monitor)** | Eşleşen tehditler bulgu üretir, hiçbir süreç sonlandırılmaz. | — |
| **Aktif sonlandırma (Enforce)** | Kural motoruyla eşleşen her süreç çekirdek tarafından sonlandırılır; bulgular yine üretilir. | Sadece host yöneticisi |

Modu değiştirmek bir onay penceresi açar ve **aktif sonlandırma moduna geçerken bir gerekçe metni zorunludur** — bu metin denetim zincirine olduğu gibi kaydedilir. Gözlem moduna dönerken gerekçe isteğe bağlıdır.

> **Uyarı:** Aktif sonlandırma modu, kural motoruyla eşleşen HER süreci sonlandırır — kurallar bu kiracı için ayarlanmamışsa meşru bir iş yükü de yanlışlıkla öldürülebilir. Bu yüzden yalnızca kural kümesi bu kiracı için ayarlandıktan sonra etkinleştirilmesi önerilir.

Aktif sonlandırma moduna terfi isteği, arka planda her kümenin gerekli çalışma zamanı kurallarına sahip olup olmadığını kontrol eden bir ön kontrolden geçer. Herhangi bir küme bu kontrolü geçemezse terfi engellenir ve hangi kümenin hangi nedenle hazır olmadığı liste halinde gösterilir; operatör kümeyi düzeltip tekrar deneyebilir veya iptal edip gözlem modunda kalabilir.

> **Not:** Kiracı aktif sonlandırma modundayken sayfanın üstünde kalıcı bir uyarı şeridi görünür: "Aktif sonlandırma modu — bu kiracıdaki eşleşen tehditler çekirdek tarafından sonlandırılır." Meşru bir iş yükü öldürülüyorsa gözlem moduna dönülmesi önerilir.

### Paylaşılan küme uygulaması

Aynı sayfanın altında, sadece host yöneticisinin gördüğü bir **Paylaşılan küme uygulaması** kartı bulunur. Bu, birden fazla kiracının aynı düğümü paylaştığı bir kümede servis bazlı çalışma zamanı bloklarının önünü açan iki kademeli bir kilittir:

1. Bu karttaki genel ana anahtar (**Kilitle / Kilidi aç**) — kapalıyken hiçbir paylaşılan kümede servis bazlı blok uygulanamaz.
2. Her hedef servisin kendi güvenlik sayfasındaki katılım tercihi (bu doküman kapsamı dışında).

Bu iki kademe de açık olmadan, ve bir operatör bir bulguya blok uygulamadan, hiçbir engelleme gerçekleşmez — anahtarın kendisi hiçbir şeyi doğrudan engellemez, sadece platform çapındaki kilidi kaldırır.

> **Uyarı:** Anahtarı kilitlemek yeni paylaşılan-küme bloklarının uygulanmasını durdurur ama önceden uygulanmış bloklar tek tek geri alınmadıkça yerinde kalır.

---

## Bloklar

Bloklar sayfası, aktif olarak zorunlu kılınan her çalışma zamanı bloğunu — tek bir bulgudan doğan bloklar ve bir saldırı zinciri müdahalesinden doğan toplu bloklar dahil — tek bir kiracı çapında listede birleştirir. Sayfa, çalışma zamanı sensörü kurulu kümelerde kullanılabilir.

Sayfanın üstünde bir **hazırlık kartı** bulunur; bu kart yanıt kontrolünün o an kullanılabilir olup olmadığını özetler:

| Durum | Anlamı |
|---|---|
| **Kontrol ediliyor** | Veriler henüz yükleniyor. |
| **Yanıt durumu bilinmiyor** | Veri yüklenemedi veya okuma yetkisi yok. |
| **Çalışma zamanı kanıtı hazır değil** | Yanıt kontrolüne erişim var ama alttaki kanıt sağlığı yetersiz — yeni bir blok uygulamak güvenilir olmayabilir. |
| **Yanıt kontrolü kullanılabilir** | Her şey hazır; blok uygulanabilir ve geri alınabilir. |
| **Salt okunur yanıt erişimi** | Görüntüleme var ama uygulama/geri alma yetkisi yok. |

> **Not:** Sıfır aktif blok görmek asla "sıfır tehdit var" anlamına gelmez — bloklar bir tespit kapsamı göstergesi değil, bir yanıt aksiyonudur. Her bulgu için destek ve risk ayrı ayrı değerlendirilir.

Liste **Durum** filtresiyle **Aktif**, **Geri alınmış** veya **Tümü** olarak süzülebilir; bir servise tıklandığında listeyi o servise indirger.

Her satırda blok türü (**Saldırı zinciri** veya **Tek bulgu**), canlı durum ve etki özeti görünür:

| Durum rozeti | Anlamı |
|---|---|
| **Aktif** | Blok kümede etkin ve uygulanıyor. |
| **Uygulanıyor** | Blok kaydedildi, servisin yeniden dağıtımı hâlâ sürüyor. |
| **Yeniden dağıtım başarısız** | Blok kaydedildi ama kümeye yansıtacak yeniden dağıtım başarısız oldu — durum takip gerektirir. |
| **Geri alınmış** | Blok kaldırıldı, servis eski haline döndü. |

Bir bloğu geri almak iki adımlıdır:

1. **Geri al** düğmesine basıldığında bir önizleme başlar ve kaç girdinin kaldırılacağı, kaç girdinin başka bloklarla paylaşıldığı için korunacağı gösterilir.
2. Önizleme geldiğinde **Geri almayı onayla** ile işlem gerçek olarak uygulanır — servis bir kez yeniden dağıtılır.

> **Uyarı:** Bir bloğu geri almak, o bulgu için sağlanan korumayı kaldırır. Sunucu geri alma isteğini "başarılı" olarak işaretlese de blok hâlâ etkinse (yazma veya yeniden dağıtım kuyruğa alma başarısız olduysa) sayfa bunu asla başarı olarak göstermez — bu durumda tekrar denemeniz istenir.

> **Not:** Geri alma geçmişi sadece saldırı-zinciri bloklarını kapsar; geri alınmış bir tek-bulgu bloğu ayrı bir kayıt defteri satırı bırakmaz.

---

## Honey Paths

Honey Paths sayfası, bir servisin pod'larına tuzak (canary) dosyalar yerleştirmenizi sağlar. Listelenen bir yolun çalışma zamanında **okunması**, kritik önem derecesinde bir bulgu üretir — meşru bir iş yükünün bu dosyalara asla dokunmaması beklendiğinden bu, yorum gerektirmeyen bir ihlal sinyalidir. Sayfa, çalışma zamanı sensörü kurulu kümelerde kullanılabilir.

Kullanım akışı:

1. Üstteki seçiciden bir **servis** seçin — arama kutusuyla filtrelenebilir.
2. **Yeni honey path** ile bir tuzak dosya ekleyin: **mutlak yol** (`/` ile başlamalı), isteğe bağlı açıklama ve **Etkin** anahtarı.
3. Tabloda her satırın **Etkin** anahtarını doğrudan değiştirebilir, **Düzenle** veya **Sil** ile yönetebilirsiniz.

Değişiklikler canlı çalışma zamanı politikasına yaklaşık bir dakika içinde yansır. Bir servis seçildiğinde üstte bir **hazırlık kartı** görünür:

| Durum | Anlamı |
|---|---|
| **Servis seçin** | Henüz bir servis seçilmedi. |
| **Kontrol ediliyor** | Servis seçildi, veriler yükleniyor. |
| **Hazırlık bilgisi yok** | Veri yüklenemedi. |
| **Yapılandırılmadı** | Servis için henüz etkin bir tuzak yolu yok. |
| **Yapılandırıldı** | En az bir etkin tuzak yolu var ve çalışma zamanı kanıtı sağlıklı. |
| **Yapılandırıldı; çalışma zamanı kanıtı doğrulanamadı** | Etkin tuzak yolu var ama alttaki kanıt sağlığı henüz doğrulanamadı. |

Her satırda tetiklenme sayısı ve son tetiklenme zamanı görünür; tetiklenme sayısı sıfırdan büyükse satır belirgin bir rozetle işaretlenir.

> **İpucu:** İlk tuzak olarak kimlik bilgisi görünümlü bir yol (örn. `/opt/komuta/canary/credentials`) ile başlayıp etkin tutun, ardından çalışma zamanı politikasının senkronize olduğunu doğrulayın.

> **Uyarı:** Bir honey path'i silmek, çalışma zamanı bileşeninin bu yolu bir dakika içinde izlemeyi durdurmasına yol açar. Geçmiş tetiklenme kayıtları saklanır ama bundan sonra bu yolun okunması artık bir bulgu üretmez.

---

## Playbook'lar

Playbook'lar sayfası, olay müdahale (incident response) için **salt okunur bir katalog** sunar. Bir bulgu bir playbook'un tetikleme koşuluyla eşleştiğinde, ekibin izlemesi önerilen standart yanıt adımlarını gösterir.

Katalog **Yerleşik playbook'lar** ve **Özel playbook'lar** olarak iki gruba ayrılır — özel playbook yazma özelliği henüz yayınlanmadığından bugün her kiracı aynı temel yerleşik seti görür.

Her kart şu bilgileri gösterir:

| Alan | Açıklama |
|---|---|
| **Önem derecesi eşiği** | Playbook'un tetiklenmesi için bulgunun en az hangi önem derecesinde olması gerektiği (Bilgi / Düşük / Orta / Yüksek / Kritik). |
| **Kaynak** | Tetikleyici sinyalin geldiği alan — örn. Çalışma zamanı sensörü veya Ağ görünürlüğü — ya da "herhangi biri". |
| **Tür** | Tetikleyici bulgu türü, ya da "herhangi biri". |
| **Aksiyon sayısı** | Playbook'un önerdiği adım sayısı. |

Bir karta tıklandığında detay görünümü açılır ve önerilen yanıt adımları numaralı bir liste halinde gösterilir — örneğin bir olay açmak, servis sahibini aramak, pod'u izole etmeye önermek, bir kimlik bilgisini döndürmek veya bir kanıt paketi hazırlamak.

> **Not:** Bu katalog tamamen tavsiye niteliğindedir — hiçbir playbook adımı otomatik olarak çalıştırılmaz. Bildirim gönderimi ve otomatik aksiyon yürütme gelecekteki bir sürümde eklenecektir; şu an playbook'lar SOC ekibinin ne yapması gerektiğini anlatır, Komuta'nın otomatik olarak ne yapacağını değil.

---

## Neden böyle?

**Süreç/dosya politikaları neden bazı kümelerde hiçbir işe yaramıyor?**
Bu politikalar ana makine çekirdeği tarafından zorunlu kılınır. Platformunuzun iş yükleri izole bir sanal makine çalışma zamanında çalışıyorsa, ana makine bu iş yüklerinin içine giremez — dolayısıyla süreç ve dosya kuralları teknik olarak var olsa da hiçbir etkisi olmaz. Bu durumda Politikalar, Çalışma Zamanı Uygulama, Bloklar ve Honey Paths sayfaları bunu açıkça bir bilgi kutusuyla belirtir; ağ segmentasyonu ve iş yükü izolasyonu gibi ağ katmanı korumaları ise etkilenmeden çalışır.

**Aktif sonlandırma moduna geçerken neden gerekçe zorunlu?**
Bu mod gerçek süreçleri sonlandırabilen, geri dönüşü olmayan bir aksiyondur. Gerekçe metni denetim zincirine olduğu gibi kaydedilerek "bu değişiklik neden şu anda yapıldı" sorusuna her zaman yanıt verilebilir hale gelir; gözlem moduna dönüşte ise zaten koruyucu bir geri çekilme olduğundan gerekçe isteğe bağlı bırakılmıştır.

**Paylaşılan küme uygulaması neden iki kademeli?**
Bir bloğu paylaşılan bir düğümde uygulamak, o düğümü kullanan başka kiracıları da dolaylı olarak etkileyebilir. Bu yüzden hiçbir kiracı kendi başına bu yetkiyi açamaz: önce platform genelinde bir kilit host yöneticisi tarafından kaldırılmalı, sonra her servis kendi güvenlik sayfasından ayrı ayrı katılım göstermelidir. İki kademe de sağlanmadan hiçbir blok uygulanamaz.

**Öneri kabul etmeden önce neden değişiklik önizlemesi zorunlu?**
Bir öneriyi kabul etmek gerçek bir kuralı canlıya alır. Önizleme yüklenemediğinde veya kümedeki karşılığı bulunamadığında **Kabul et** düğmesi kilitlenir — aksi halde bir operatör ne değiştiğini hiç görmeden bir kuralı onaylayabilirdi.

---

## SSS

**Bir politikayı devre dışı bırakırsam silinir mi?**
Hayır. Devre dışı bırakılan bir politika veritabanında saklanır ama küme ile senkronize edilmez; istediğiniz zaman yeniden etkinleştirebilirsiniz.

**Platform kaynaklı bir politikayı neden düzenleyemiyorum?**
Bu satırlar dağıtım hattı tarafından servisin güvenlik temeline göre otomatik üretilir. Değiştirmek için servisin kendi güvenlik ayarlarına gidilmesi gerekir — bu sayfa sadece görüntüleme sunar.

**Bir bloğu geri aldıktan sonra tekrar uygulayabilir miyim?**
Geri alma o bloğu kaldırır; aynı korumayı yeniden istiyorsanız ilgili bulguya tekrar bir blok uygulamanız gerekir.

---

## Kimler Kullanır?

| Sayfa / işlev | İlgilenen rol |
|---|---|
| Politikalar sekmesi — görüntüleme **ve** oluşturma/düzenleme/silme/senkronize etme | Tüm geliştirici ekip |
| Öneriler sekmesi (görüntüleme) | Tüm geliştirici ekip |
| Öneriler — kabul etme, reddetme | Güvenlik operatörü |
| Çalışma Zamanı Uygulama (görüntüleme) | Tüm geliştirici ekip |
| Çalışma Zamanı Uygulama — mod değiştirme, paylaşılan küme kilidi | Platform yöneticisi |
| Bloklar (görüntüleme, geri alma) | Güvenlik operatörü |
| Honey Paths (görüntüleme) | Tüm geliştirici ekip |
| Honey Paths — oluşturma, düzenleme, silme | Güvenlik operatörü |
| Playbook'lar | Tüm geliştirici ekip |

> **Not:** Öneriler, Honey Paths ve Bloklar sekmelerinde görüntüleme ve yönetme ayrı yetkilerle kontrol edilir. Politikalar sekmesinde ise bugün tek bir görüntüleme yetkisi, hem sekmeyi göstermeye hem de tüm yönetim aksiyonlarını (Yeni politika, Düzenle, Sil, Konuşlandır/Konuşlandırmayı kaldır, Tümünü senkronize et) açmaya yeterlidir — ayrı bir "yönetme" kontrolü yoktur. Bu sekmeyi görebilen herkes bugün itibarıyla politikaları da yönetebilir.

---

## İlgili

- [Güvenlik Merkezi: Genel Bakış](./security-center-overview.md)
- [İzle: Genel Bakış ve Bulgular](./security-center-monitor.md)
- [Kayıt: Denetim ve Giriş Aktivitesi](./security-center-record.md)
- [Doğrula: Uyumluluk ve Sağlık](./security-center-verify.md)
- [Yönet: Bildirim ve Saklama](./security-center-manage.md)
