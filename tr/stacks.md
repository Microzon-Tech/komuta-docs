# Stack'ler

Stack, bir uygulamayı oluşturan servisleri, yönetilen servisleri (PostgreSQL, Valkey, RabbitMQ) ve alan adlarını tek bir `komuta.yaml` dosyasında tanımlar. Komuta dosyayı mevcut kaynaklarla karşılaştırır, neyin oluşturulacağını ve tahmini aylık maliyeti bir planda gösterir; siz onaylayınca planı uygular.

Birden fazla servisi ve veritabanını birlikte, tekrarlanabilir şekilde kurmak istediğinizde Stack kullanın. Dosya kaynak kodunuzun yanında durur ve sürümlenir. Dosyanın tüm alanları [komuta.yaml Manifest Başvurusu](stack-manifest-reference.md) sayfasındadır.

---

## Nasıl Çalışır

1. `komuta.yaml` dosyasını hazırlayın: AI asistanınızla, elle ya da reponuzdan.
2. **Plan oluştur** ile dosyayı doğrulatın ve planı görün.
3. Şifre ve anahtar gibi gizli değerleri tek kullanımlık güvenli bağlantıdan girin.
4. **Planı onayla ve uygula** ile planı başlatın ve çalıştırmayı izleyin.

Stack'in kimliği dosyadaki `name` alanıdır. Aynı adla yeni bir plan oluşturduğunuzda Komuta aynı Stack'i günceller; farklı bir ad yeni bir Stack demektir. Stack'in oluşturduğu ya da devraldığı kaynaklar o Stack'e aittir ve bir kaynak yalnızca bir Stack'e ait olabilir.

Bir Stack, ilk planı oluşturulduğu anda [**Stack'ler**](https://console.komuta.io/stacks) listesinde **Uygulanmadı** etiketiyle görünür.

### Gereken izinler

| İzin | Ne sağlar |
|---|---|
| **Stack ve çalıştırmaları görüntüle** | Stack listesi ve Stack sayfası |
| **Stack manifesti hazırla ve doğrula** | **Yeni Stack** sayfası |
| **Stack planı oluştur** | **Plan oluştur** ve son planı görüntüleme |
| **Stack planını uygula** | Planı uygulama ve güvenli secret formu |
| **Başarısız Stack çalıştırmasını sürdür** | **Başarısız adımları sürdür** |
| **Stack manifestini dışa aktar** | **Dışa aktar** |

Plan ayrıca dokunacağı her kaynak için ilgili izni ister; örneğin servis oluşturmak için servis oluşturma, bir servisi devralmak ya da güncellemek için servis düzenleme izni. Eksik izin planda bir sorun olarak listelenir.

---

## Stack Oluşturma

**Stack'ler → Yeni Stack** ile [Stack başlat](https://console.komuta.io/stacks/new) sayfası açılır. Burada iki yol vardır.

### AI ile oluştur

**Önerilen** yoldur. Projeniz, Komuta'ya bağlı bir AI istemcisinde açık olmalıdır.

1. **Yerel projeyi AI istemcinizde açın.**
2. **Komuta MCP bağlantısını doğrulayın.** Kurulu değilse **Komuta MCP'yi kur** ile [Komuta MCP Sunucusunun Kurulumu](mcp-setup.md) sayfasına gidin.
3. **Yayınlama promptunu yapıştırın.** Promptu **Promptu kopyala** ile alın.

Asistan kodu inceler, kalan iş kararlarını tek mesajda sorar, `komuta.yaml` dosyasını kaynak kodunuzun yanında hazırlar, doğrular ve son maliyet ile risk özetini gösterir. Yayınlamadan önce onayınızı bekler. Gizli değerler sohbette istenmez; tek kullanımlık güvenli bağlantıdan girilir.

> **Not:** Komuta yerel klasörünüzü tarayıcıdan bağlayamaz. Sohbeti, projenizin zaten açık olduğu AI istemcisinde başlatın.

**Profil.** Asistanınız size nasıl ilerlemek istediğinizi sorabilir ve seçiminizi kaydeder: yönlendirmeli (`guided`, varsayılan), dengeli (`balanced`) ya da uzman (`expert`). Profil yalnızca asistanın kaç kararı açıklayıp sorduğunu belirler; maliyet, internete açılma, kalıcı veri ve güvenlik gevşetmeleri her profilde sizin onayınızı ister. Konsolda uzman profil, plan incelemesinde her kaynağın **Yönetilen alanlar** listesini de gösterir.

### Mevcut YAML'ı incele

**Gelişmiş** yoldur; elinizdeki bir `komuta.yaml` dosyasını incelemek içindir.

- **komuta.yaml seç** ile dosyayı yükleyin. Yalnızca `.yaml` veya `.yml` uzantılı ve en fazla 1 MiB dosya kabul edilir.
- **Mevcut YAML metnini yapıştır** ile boş düzenleyiciyi açın ve içeriği yapıştırın. Boş düzenleyicide **İskeletle başla**, doldurmanız gereken yer tutucularla bir başlangıç dosyası yükler; değerleri kendi kaynaklarınızla değiştirin ve alanları [başvuru](stack-manifest-reference.md) sayfasıyla karşılaştırın.

Düzenleyicide satır sayısı ve kopyalama düğmesi bulunur. **YAML'ı indir** taslağı `komuta.yaml` olarak indirir; dosyayı reponuza koyun ki doğruluk kaynağı repo olsun. **Plan oluştur** düğmesine basana kadar dosya tarayıcıdan çıkmaz.

Taslak uygulanmadan sayfadan ayrılmak isterseniz **YAML incelemesinden çıkılsın mı?** uyarısı çıkar; taslak ve planı silinir. **Başka kaynak seç** de aynı şekilde taslağı silip başlangıç ekranına döner.

### Repodaki komuta.yaml dosyasından

[**Uygulamalar → Yeni Uygulama**](new-service-guide.md) sihirbazında seçtiğiniz reponun **kökünde** `komuta.yaml` dosyası varsa sihirbaz Dockerfile aramaz. Dosya Stack sayfasına **Yayın önerisini incele** başlığıyla açılır ve plan kendiliğinden oluşturulur.

Dosyada `source.repository` yazmayan servislere sihirbazda seçtiğiniz repo, `source.branch` yazmayanlara seçtiğiniz branch eklenir. Başka bir repoyu gösteren servislere dokunulmaz.

> **Not:** Yalnızca repo kökündeki ve tam olarak `komuta.yaml` adındaki dosya algılanır.

### .NET Aspire ve çok servisli projeler

Eski Aspire içe aktarma adresi artık **Yeni Stack** sayfasını açar; ayrı bir Aspire içe aktarma ekranı yoktur. Aspire ya da docker-compose gibi çok servisli bir projeyi yayınlamak için **AI ile oluştur** yolunu kullanın ya da `komuta.yaml` dosyasını kendiniz yazın.

---

## Doğrulama ve Plan

Ayrı bir doğrulama düğmesi yoktur: **Plan oluştur** önce dosyayı doğrular. Dosyada hata varsa "Manifestin düzeltilmesi gerekiyor." uyarısı çıkar ve **Doğrulama** kartı her sorunu kodu, dosyadaki yolu (ör. `$.services[0].compute.plan`) ve mesajıyla listeler. Uyarılar planı engellemez ama yine listelenir. Mesajların tamamı [başvuru](stack-manifest-reference.md#doğrulama-mesajları) sayfasındadır.

Dosya geçerliyse Komuta planı oluşturur. Plan sırasında şunlar da denetlenir:

- hedef projede aynı adla mevcut kaynaklar (devralma gerekir mi),
- her servisin branch'i ve Dockerfile'ı: Dockerfile **push edilmiş** branch'te bulunmalıdır; plan o anki commit'i ve Dockerfile'ı sabitler,
- paket, yönetilen servis, depolama ve bant genişliği planlarının adları ve fiyatları,
- izinleriniz ve ücretsiz paket kuralları.

> **Uyarı:** Plan, eksik bir Dockerfile'ı sizin için üretmez. Hazır imaj kullanmayan (`build.strategy: image` olmayan) bir serviste Dockerfile branch'e push edilmeden plan engellenir: `Dockerfile '…' does not exist in pushed branch '…'. Create, test, commit and push it before planning.`

### Planda ne görürsünüz

| Kutucuk | Anlamı |
|---|---|
| **Planlanan işlemler** | Plandaki kaynak işlemi sayısı. |
| **Tahmini aylık maliyet** | Planlanan durumun aylık tahmini; altında son uygulanan sürümün tahmini ve fark (**Mevcut … · değişim …**). |
| **Eksik secret** | Henüz girilmemiş zorunlu gizli değer sayısı. |
| **Değişmez plan geçerliliği** | Planın kalan süresi. Plan 30 dakika geçerlidir. |

Süre dolunca **Planın süresi doldu** görünür ve plan uygulanamaz. **Yeni plan oluştur** ile Komuta yetkileri, fiyatları ve canlı kaynak durumunu yeniden denetler.

Planda riskli bir şey varsa üç kart çıkar: **İnternete açık kaynaklar** (genel adresi olan servisler, özel erişimli olmayan yönetilen servisler ve alan adları), **Kalıcı veri** (kalıcı depolama alanları) ve **Güvenlik gevşetmeleri** (root olarak çalışma, ek Linux yetenekleri, yazılabilir yollar).

**Plan incelemesi** kartı **Uygulamaya hazır** ya da **İşlem gerekiyor** etiketini taşır. İşlemleri adla arayabilir, **Tüm işlemler**, **Yalnız değişiklikler**, **İlgi gerektirenler** ve **Değişiklik yok** ile süzebilirsiniz. Her satırda kaynağın adı, Komuta'nın gerekçesi, varsa **Değişen alanlar** ve işlem başka Stack'leri de etkiliyorsa "… stack daha etkilenir" uyarısıyla bu Stack'lerin adları bulunur. Plandaki sorunlar kartın altında `KOD · yol — mesaj` biçiminde listelenir. Gerekçe ve sorun mesajları İngilizce gösterilir.

| İşlem | Anlamı |
|---|---|
| **Oluştur** | Kaynak yok; oluşturulacak. |
| **Güncelle** | Stack'in kaynağı; dosyada değişen alanlar var. |
| **Devral** | Aynı adla bir kaynak zaten var; onayınızla Stack'e geçer. |
| **Değişiklik yok** | Dosya son uygulanan sürümle aynı. |
| **Sahipsiz** | Kaynak artık dosyada yok; silinmez, çalışmaya devam eder. |
| **Kullan** | Mevcut bir yönetilen servis yönetilmeden kullanılır (`uses.addons`). |

| Etki | Nerede görünür |
|---|---|
| **Rollout** | Oluşturulan ya da güncellenen servisler: yeni sürüm yayına alınır. |
| **Yeniden oluşturma** | Yönetilen servisler; yeni oluşturulanlar da bu etiketle listelenir. |
| **Etki yok** | Proje, alan adları, devralınan, değişmeyen, sahipsiz ve kullanılan kaynaklar. |

**İlgi gerektirenler** süzgeci; devralma onayı bekleyen, **Yeniden oluşturma** etkili, **Sahipsiz** ve başka Stack'leri etkileyen işlemleri gösterir.

### Tahmini maliyet

Tahmin şunları toplar:

- her servisin paket fiyatı × replika sayısı (`fixed` modunda `count`, otomatik ölçeklemede `min`; yazılmazsa 1),
- kalıcı depolama planları,
- bant genişliği planları,
- Stack'in oluşturduğu yönetilen servislerin aylık plan fiyatı.

Tahmine girmeyenler: `uses.addons` ile kullanılan mevcut yönetilen servisler, alan adları, ücretsiz kotayı aşan build dakikaları, gözlemlenebilirlik depolaması ve vergi. **Mevcut**, son uygulanan sürümün tahminidir (yeni bir Stack'te 0); **değişim** ikisinin farkıdır. Fiyatlar USD cinsindendir. Güncel birim fiyatlar [Fiyatlandırma](https://www.komuta.io/pricing) sayfasındadır.

Bir fiyat bulunamazsa plan engellenir ve "Fiyat bilgisi alınamadı. Onaylamadan önce planı yenileyin." görünür.

> **Not:** Harcama limiti planda denetlenmez; Komuta her servisi ve yönetilen servisi oluştururken denetler. Limit aşılırsa o adım başarısız olur. Limit için [Faturalandırma](billing.md) sayfasına bakın.

### Mevcut kaynakları devralma

Hedef projede aynı adla bir servis, yönetilen servis ya da alan adı varsa plan **Devral** işlemini ve **Mevcut kaynakların devralınmasını onaylayın** kartını gösterir.

1. Devralmak istediğiniz her kaynağı işaretleyin.
2. **Onaylanan devralmalarla yeniden planla** düğmesine basın.

Devralınan kaynak değiştirilmez; Stack yalnızca sahipliğini üstlenir. Yönetilen bir servisi devralmak için dosyada türü, planı ve ağ erişimi mevcut haliyle aynı olmalıdır. Hiçbir projeye bağlı olmayan ortak bir yönetilen servis devralınınca hedef projeye taşınır; proje dışındaki servisler onu kullanıyorsa devralma engellenir, bu durumda `uses.addons` kullanın. Başka bir Stack'e ait kaynak devralınamaz.

---

## Gizli Değerler

Şifre, anahtar ve token gibi değerler `komuta.yaml` dosyasına yazılamaz. Dosyada `secrets` altında bir ad tanımlarsınız, ortam değişkeni bu ada başvurur. Değerin kaynağı üç türlüdür:

| Kaynak | Değer nereden gelir |
|---|---|
| `prompt` (varsayılan) | Siz tek kullanımlık güvenli bağlantıdan girersiniz. |
| `generate` | Komuta plan uygulanırken rastgele üretir. |
| `existing` | Bu Stack'te daha önce başka bir adla saklanmış değer kullanılır (`existingName`). |

Değerler Stack'e ait şifreli bir depoda saklanır ve bir daha gösterilmez. Aynı Stack'in sonraki planları kayıtlı değeri kullanır; yeniden sormaz.

### Tek kullanımlık güvenli bağlantı

1. Planda **Güvenli secret formunu aç** düğmesine basın. Form yeni sekmede açılır. Açılmazsa plan sayfasındaki "Güvenli form açılmadıysa bu tek kullanımlık bağlantıyı kullanın" uyarısında yer alan bağlantıyı kullanın.
2. **Güvenli secret aktarımı** sayfasında her değerin, planın oluşturduğu hangi servise ve hangi ortam değişkenine yazılacağı görünür.
3. Değerleri tek tek girin ya da **.env veya JSON'dan yapıştır** ile yapıştırıp **Alanları doldur** deyin. Satırlar ortam değişkeni adına ya da secret adına göre eşlenir; diğerleri atlanır. Satır sonu içeren değerler (ör. PEM anahtarları) bu yolla doldurulmaz.
4. **Güvenle kaydet** deyin. **Secret değerleri güvenle kaydedildi** görününce sekmeyi kapatın.

Plan sayfası kendiliğinden yenilenir ve "Secret değerleri güvenli biçimde alındı. Plan incelemeye hazır." bildirimi çıkar.

| Kural | Ayrıntı |
|---|---|
| Süre | Bağlantı 15 dakika geçerlidir. |
| Kullanım | Tek kullanımlıktır; kaydedince geçersiz olur. |
| Kişi | Yalnızca bağlantıyı açan kişi, **Stack planını uygula** izniyle kullanabilir. |
| Kapsam | Yalnızca o planın eksik `prompt` değerlerini ister ve hepsinin birlikte girilmesini bekler. |

Süresi dolmuş ya da kullanılmış bağlantıda "Güvenli bağlantı geçersiz, süresi dolmuş veya daha önce kullanılmış." görünür; plandan yeni bir bağlantı açın.

> **Not:** `required: false` olan bir `prompt` değeri formda istenmez. Bir ortam değişkeni ona başvuruyorsa ilgili adım uygulama sırasında `SECRET_NOT_RESOLVED` ile durur.

---

## Planı Uygulama

1. Plan **Uygulamaya hazır** olduğunda **Planı onayla ve uygula** düğmesine basın. Sayfa uzunsa **Planı onayla** düğmesi sizi bu düğmeye götürür.
2. **Bu plan aynen uygulansın mı?** penceresinde işlem sayısını ve tahmini maliyeti okuyun, **Onaylanan planı uygula** deyin.

Düğme; planda hata, eksik secret, onaylanmamış devralma, bilinmeyen fiyat ya da süresi dolmuş plan varken kapalıdır. Planı yalnızca onu oluşturan kişi uygulayabilir. Uygulama anında Komuta bazı koşulları yeniden denetler:

| Mesaj | Ne yapmalı |
|---|---|
| Planın süresi doldu; yeni bir plan oluşturun. | Yeni plan oluşturun. |
| Planlamadan sonra yetkiler değişti; yeni bir plan oluşturun. | Yeni plan oluşturun. |
| Gerekli secret değerleri tamamlanmadı. | Güvenli formu tamamlayın. |
| Planlamadan sonra kaynak durumu değişti. Yeni bir plan oluşturun. | Yeni plan oluşturun. |
| Planlamadan sonra uzak kaynak revision'ı değişti; yeni bir plan oluşturun. | Branch'e yeni commit gelmiştir; yeni plan oluşturun. |
| Uzak kaynak revision'ı doğrulanamadı; planı değiştirmeden apply işlemini yeniden deneyin. | Biraz sonra yeniden uygulayın. |
| Kaynak fiyatları değişti veya fiyat bilgisi alınamadı. Onaylamadan önce yeni bir plan oluşturun. | Yeni plan oluşturun. |
| Bu proje için başka bir Stack çalışması devam ediyor. | Bir projede aynı anda tek çalışma olur; bitmesini bekleyin. |

---

## Çalıştırmayı İzleme

Uygulama başlayınca **Canlı uygulama zaman çizelgesi** her kaynak için bir adım gösterir. **Değişiklik yok**, **Sahipsiz** ve **Kullan** işlemleri adım oluşturmaz.

Adımlar bağımlılık sırasıyla çalışır. Proje oluşturulacaksa her şeyden önce oluşturulur. Bir servis, değer okuduğu yönetilen servislerden ve başvurduğu servislerden sonra; bir alan adı, bağlı olduğu servisten sonra çalışır. Aynı anda en fazla `apply.maxParallel` adım çalışır (varsayılan 3).

| Çalışma durumu | Anlamı |
|---|---|
| **Kuyrukta** | Çalışma başlamayı bekliyor. |
| **Çalışıyor** | Adımlar yürütülüyor. |
| **Girdi bekleniyor** | Komuta süren adımların bitmesini ya da bir kaynağın hazır olmasını bekliyor (ör. kurulan veritabanı ya da servisin ilk dağıtımı). Sizden bir şey beklenmez. |
| **Başarılı** | Bütün adımlar tamamlandı; bu sürüm Stack'in etkin sürümü oldu. |
| **Kısmen başarısız** | Bazı adımlar tamamlandı, bazıları başarısız oldu. |
| **Başarısız** | Hiçbir adım tamamlanamadı. |

| Adım durumu | Anlamı |
|---|---|
| **Beklemede** | Sırasını ya da yeniden denemesini bekliyor. |
| **Çalışıyor** | Yürütülüyor. |
| **Bekliyor** | Kaynağın hazır olmasını bekliyor. |
| **Başarılı** | Tamamlandı. |
| **Başarısız** | Durdu; hata kodu ve mesajı adımın altında görünür. |
| **Engellendi** | Bağlı olduğu adım başarısız olduğu için çalıştırılmadı. |

Geçici hatalar artan aralıklarla en fazla 5 deneme yapılarak kendiliğinden yeniden denenir. Kalıcı hatalar adımı hemen durdurur; o adıma bağlı adımlar engellenir, diğerleri devam eder. Sık görülen adım hataları:

| Kod | Anlamı |
|---|---|
| `REGION_NOT_FOUND` | `target.region` yazılmamış ya da etkin bir bölge değil. |
| `GIT_CONNECTION_NOT_FOUND` / `REGISTRY_CONNECTION_NOT_FOUND` | `target.connections` altındaki bağlantı adı bulunamadı. |
| `COMPUTE_PLAN_NOT_FOUND` / `ADDON_PLAN_NOT_FOUND` / `ADDON_PLAN_INACTIVE` | Paket ya da yönetilen servis planı yok veya satışta değil. |
| `ROUTE_HOST_NOT_OWNED` | Bir rota, `domains` altında o servise bağlanmamış bir alan adını kullanıyor. |
| `SECRET_NOT_RESOLVED` | Başvurulan gizli değer kayıtlı değil. |
| `OWNERSHIP_CONFLICT` | Kaynak başka bir Stack'e ait. |
| `OWNED_RESOURCE_NOT_FOUND` | Stack'in kaynağı konsoldan silinmiş. |

### Başarısız adımları sürdürme

**Başarısız** ya da **Kısmen başarısız** bir çalışmada **Başarısız adımları sürdür** düğmesi çıkar.

1. **Başarısız adımları sürdür** deyin.
2. **Bu Stack çalışması sürdürülsün mü?** penceresinde **Çalışmayı sürdür** deyin.

Başarılı adımlara dokunulmaz; başarısız ve engellenmiş adımlar yeniden değerlendirilir. Sürdürme yalnızca planı oluşturan kişi tarafından ve izinleri değişmemişken yapılabilir. Plan, branch'in commit'ini sabitlediği için branch ilerlediyse servis adımı yeniden başarısız olur; bu durumda yeni plan oluşturun.

---

## Stack Sayfası

[**Stack'ler**](https://console.komuta.io/stacks) listesinde her kartta Stack'in adı, projesi, son çalışma durumu (hiç uygulanmadıysa **Uygulanmadı**), **Dışa aktar** ve **Operasyonları aç** bulunur.

Stack sayfasının üstünde **Hedef proje**, **Son çalışma**, **Sahip olunan kaynak** ve **Sapma bulunan kaynak** kutucukları yer alır. Altında **Sahiplik ve sapma** paneli, son çalışmanın planı (salt okunur; **Stack planı oluştur** izni gerekir) ve **Canlı uygulama zaman çizelgesi** görünür.

### Sahiplik ve sapma

Panel, Stack'e ait her kaynağı **Yönetilen alanlar** listesiyle ve **Senkron** ya da **Stack dışında değiştirildi** durumuyla gösterir. Başlıktaki etiket **İstenen durumla eşleşiyor** ya da **Sapma bulundu** olur.

> **Not:** Bu karşılaştırma bugün yalnızca Komuta'nın Stack üzerinden yaptığı değişiklikleri kaydeder. Konsoldan yapılan bir değişiklik bu panelde sapma olarak görünmez. Böyle bir değişiklik, Stack aynı ayarı bir sonraki değiştirişinde ortaya çıkar: ortam değişkeni güncellemesi `ENV_UPDATE_DRIFT`, erişim koruması güncellemesi `ACCESS_DRIFT` ile durur.

---

## Dışa Aktarma

**Dışa aktar**, son **başarılı** çalışmada uygulanan dosyayı `komuta.yaml` olarak indirir. Dosya yüklendiği haliyle gelir; yorumlar da korunur. Gizli değerler dosyada hiç bulunmadığı için dışa aktarımda da yoktur.

Hiç başarılı çalışması olmayan bir Stack dışa aktarılamaz: "Bu Stack'in dışa aktarılabilecek başarılı bir revision'ı yok."

Dışa aktarma, konsoldan kurduğunuz kaynaklardan dosya üretmez. Mevcut kaynakları bir Stack'e almak için onları dosyada aynı adlarla tanımlayın ve plan sırasında devralın.

---

## Değiştirme ve Yeniden Uygulama

Stack sayfasında düzenleme düğmesi yoktur. Değişiklik dosya üzerinden yapılır:

1. `komuta.yaml` dosyasını reponuzda ya da **Dışa aktar** ile aldığınız kopyada düzenleyin. `name` alanını değiştirmeyin.
2. **Yeni Stack → Mevcut YAML'ı incele** ile dosyayı yükleyin.
3. **Plan oluştur** deyin. Plan dosyayı son uygulanan sürümle karşılaştırır; her kaynak **Değişiklik yok**, **Güncelle**, **Oluştur** ya da **Sahipsiz** olarak listelenir.

Bugün yerinde uygulanabilen değişiklikler:

- Yeni servis, yönetilen servis ve alan adı eklemek.
- Rollout stratejisi `auto-promote` olan bir serviste düz ortam değişkenlerini eklemek, değiştirmek veya silmek. Değişken gizli olmamalı, build sırasında kullanılmamalı, `valueFrom` içermemeli ve değeri boş olmamalıdır. Komuta değerleri yazar, servisi aynı imajla yeniden dağıtır ve sağlıklı olmasını bekler. Güncellenen değişken artık bir ortam değişkeni grubunu izlemez.
- `access` bloğunu tek başına değiştirmek ([Stack Manifestinde Erişim Koruması](stack-manifest-access.md)).

Diğer değişiklikler (paket, port, imaj, yönetilen servis planı, alan adı gibi) planı engeller: `In-place updates for Stack-managed … resources are not enabled on this target.` Bunları kaynağın kendi sayfasından yapın.

> **Not:** Plan dosyayı canlı kaynakla değil, son uygulanan dosyayla karşılaştırır. Konsoldan değiştirdiğiniz bir ayarı dosyaya da yazarsanız plan bunu değişiklik olarak görür ve engeller.

---

## Silme

Stack'i silmek için konsolda bir işlem yoktur. Bir kaynağı dosyadan çıkarmak onu silmez: kaynak **Sahipsiz** olarak listelenir, çalışmaya ve ücretlendirilmeye devam eder.

Bir kaynağı kaldırmak için kendi sayfasından silin: servisler için **Uygulamalar**, veritabanları ve kuyruklar için **Yönetilen Servisler**, alan adları için **Alan Adları**. Silinen kaynak Stack'in sahiplik listesinde kalır; Stack o kaynağı değiştirmeye çalışırsa adım `OWNED_RESOURCE_NOT_FOUND` ile durur.

---

## İlgili Dokümanlar

- [komuta.yaml Manifest Başvurusu](stack-manifest-reference.md) — dosyanın tüm alanları, varsayılanlar ve hata mesajları.
- [Stack Manifestinde Erişim Koruması](stack-manifest-access.md) — `access` bloğu.
- [Komuta MCP Sunucusunun Kurulumu](mcp-setup.md) — AI istemcinizi Komuta'ya bağlama.
- [Yeni Servis Oluşturma](new-service-guide.md) — uygulama sihirbazı ve repodaki `komuta.yaml`.
- [Ortam Değişkenleri](services-environment-variables.md) — gizli ve build türündeki değişkenler.
- [Özel Alan Adları](custom-domains.md) — Stack'in eklediği alan adlarının doğrulanması.
- [Faturalandırma](billing.md) — harcama limiti ve ücretlendirme.
