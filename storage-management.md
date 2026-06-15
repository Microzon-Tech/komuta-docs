# Kalıcı Depolama (Storage)

DevOpsZon servisleriniz varsayılan olarak **stateless** çalışır — pod'lar yeniden başladığında tüm yerel dosyalar silinir. Veritabanı, dosya sunucusu, yükleme klasörü veya cache gibi **kalıcı veri** tutan uygulamalar için servisinize bir **disk planı** bağlamanız gerekir. DevOpsZon, Longhorn üzerinde çalışan dağıtık blok depolama altyapısı ile bu ihtiyacı karşılar.

Bu rehber, servis yönetimi panelindeki **Storage** sekmesini kullanarak PersistentVolumeClaim (PVC) bağlamayı, plan seçmeyi, mount path yapılandırmayı ve güvenlik kurallarını açıklar.

---

## Depolama Nasıl Çalışır?

DevOpsZon'da depolama üç bileşenden oluşur:

| Bileşen | Açıklama |
|---------|----------|
| **Longhorn** | Kubernetes üzerinde çalışan dağıtık blok depolama motoru. Tüm worker node'lar arasında veri kopyalarını (replica) senkron tutarak yüksek erişilebilirlik sağlar. |
| **Disk Planı** | Satın aldığınız depolama paketi. Boyut, replika sayısı, yedekleme politikası ve aylık ücreti belirler. |
| **Mount Preset** | Container'ın hangi dizine volume bağlanacağını tanımlayan katalog girişi. Platform tarafından hazırlanmış güvenli yollardan birini seçersiniz. |

Bir servise depolama bağladığınızda:

1. Seçtiğiniz plana göre bir PVC (PersistentVolumeClaim) oluşturulur
2. PVC, Longhorn tarafından plan parametrelerinde belirtilen replika sayısı ile provision edilir
3. Deploy sırasında pod'un spec'ine otomatik olarak `volumes` ve `volumeMounts` eklenir
4. Container'ınız, seçtiğiniz mount path üzerinden volume'a okuma/yazma yapabilir
5. Pod yeniden başlatıldığında, silindiğinde veya başka bir node'a taşındığında veri korunur

> **Not:** Longhorn tüm replikaları farklı node'larda tutmaya çalışır. Tek node'lu cluster'larda 1 replika ile çalışır; çok node'lu cluster'larda 3 replika önerilir.

---

## Disk Planları

DevOpsZon, farklı iş yüklerine yönelik beş disk planı sunar. **Her plan aynı StorageClass'ı (`longhorn-static-rwx`) kullanır** — fark sadece boyut ve aylık ücrette. Bu tasarımın iki pratik sonucu var:

1. **Tüm planlar `ReadWriteMany` (RWX) erişim modunu destekler** → hangi plan seçilirse seçilsin Canary / BlueGreen / AutoPromote rollout stratejileri, HPA ve çoklu replika topolojileri sorunsuz çalışır.
2. **Free → Paid upgrade veri kaybı olmadan yapılır** → Kubernetes PVC'nin `storageClassName` alanını immutable (değiştirilemez) kabul eder. Her plan aynı class'ta olduğu için upgrade sırasında sadece `SizeGb` artar, Longhorn volume online expand edilir, veri aynı disk üzerinde kalır.

| Plan | Boyut | StorageClass | Access Mode | Reclaim | Replika | Aylık Ücret | Uygun Olduğu İş Yükü |
|------|:-----:|:------------:|:-----------:|:-------:|:-------:|:-----------:|----------------------|
| **Free** | 1 GB | `longhorn-static-rwx` | **RWX** | Delete | 3 | **0.00 ₺** | Hobi projeleri, deneme ortamları, prototipler |
| **Small** | 10 GB | `longhorn-static-rwx` | **RWX** | Delete | 3 | **1.99 ₺** | Küçük production workload'ları, içerik depolama |
| **Medium** | 50 GB | `longhorn-static-rwx` | **RWX** | Delete | 3 | **7.99 ₺** | Orta ölçekli DB, dosya/medya depolama |
| **Large** | 200 GB | `longhorn-static-rwx` | **RWX** | Delete | 3 | **24.99 ₺** | Büyük DB'ler, dosya sunucuları, yoğun okuma/yazma |
| **XLarge** | 1 TB | `longhorn-static-rwx` | **RWX** | Delete | 3 | **89.99 ₺** | Kurumsal veri platformları, log arşivleme |

> 💡 **Neden her şey RWX?** DevOpsZon, Canary / BlueGreen / AutoPromote rollout stratejilerinin üçünü de kullanıcıya açar. Bu stratejiler rollout sırasında eski ve yeni pod'ları **aynı anda** ayakta tutar, ikisi de aynı PVC'yi mount eder. Bu da volume'un `ReadWriteMany` olmasını zorunlu kılar. "Her zaman RWX" mimarisi, tier değişimi veya HPA etkinleştirme sırasında "Multi-Attach error" ile karşılaşma riskini tamamen ortadan kaldırır. Detaylar için [Storage RWX Mimarisi](../storage-rwx-architecture.md) dokümanına bakın.

> 🔄 **Plan Upgrade'i veri kaybı olmadan**: Free plan'dan herhangi bir ücretli plana geçiş yaptığınızda (veya Small → Medium gibi yukarı doğru geçişte), platform sadece PVC'nin boyut talebini artırır. Longhorn volume'u **çevrimiçi (online) olarak genişletilir** — pod restart bile gerekmez. Verileriniz aynı disk üzerinde kalır, migration yoktur.

### Reclaim Politikası Ne Anlama Geliyor?

DevOpsZon'un kullandığı **tüm StorageClass'lar `reclaimPolicy: Delete`** ile konfigüre edilir:

- `longhorn-rwx` → Free plan, 2 replika, Delete
- `longhorn-static-rwx` → Ücretli plan'lar (Small/Medium/Large/XLarge), 3 replika, Delete

Bu platform seviyesinde bilinçli bir karardır: **bir silme, gerçekten silmedir.** Bir depolama eki veya servis silindiğinde PVC, altındaki PV, Longhorn volume, snapshot'lar ve S3 yedek arşivi — hepsi geri alınamaz şekilde kaldırılır. Plan katmanı fark etmez: Free ile Ücretli plan'ın davranışı aynıdır.

Platform ayrıca ek bir güvenlik katmanı olarak `IServiceStoragePurgeService` çalıştırır. Bu servis silmenin parçası olarak:

1. PV reclaim politikasını Delete'e patch'ler (yukarıda anlatıldığı gibi zaten Delete, bu sadece import edilmiş harici PV'leri yakalamak için bir savunma katmanı)
2. `longhorn-system` namespace'indeki Longhorn Volume, Snapshot, Backup ve BackupVolume custom resource'larını temizler
3. PVC'yi cluster'dan açıkça siler (attachment silme akışında) — böylece ArgoCD prune ile yarış durumu oluşmaz

> ⚠️ **Önemli:** Platform silmeyi "geri alınamaz" olarak tasarladı. Bir ek veya servis silindikten sonra verinin admin tarafından geri yüklenmesi gibi bir kurtarma yolu **yoktur**. Kritik veriniz için silmeden önce manuel yedek aldığınızdan emin olun.

### Free Plan Sınırları

Free plan her kullanıcıya **ücretsiz** olarak sunulur, ancak bazı sınırları vardır:

- Sadece 1 GB alan
- **En fazla 1 adet depolama eki** — aynı serviste ikinci bir free attachment oluşturulamaz (billing kotası)
- 2 replika (tek node kaybında servis çalışmaya devam eder, iki node kaybında durur)
- Reclaim politikası `Delete` — volume silinirse veri geri gelmez
- `longhorn-rwx` StorageClass ile RWX destekli — hobi iş yüklerinin Canary / BlueGreen / HPA deneyebilmesi için
- Production workload'ları için önerilmez

### Plan Katmanı (Tier) Kuralı

Bir servisin tüm depolama ekleri **tek bir fiyat katmanında** olmalıdır: ya hepsi ücretsiz ya da hepsi ücretli. Aynı serviste ücretsiz ve ücretli attachment **karıştırılamaz**.

Bu teknik bir kısıtlama değildir — artık her plan RWX'tir, teknik olarak karıştırılabilirlerdi. Ancak platform, ücretsiz ve ücretli planları ayrı faturalama döngülerinde işler. Aynı servise karma attachment eklemek, fatura dağılımını bozacağı için platform bunu reddeder.

Katmanlar arası geçiş yapmak için:
1. Servisin tüm eklerini düzenle → yeni plan tier'ına taşı (tek ek varsa düz upgrade çalışır)
2. Birden fazla ek varsa: mevcut ekleri silip yeni tier ile tekrar ekleyin

UI plan picker'ı serviste bulunan eklerin tier'ına göre karşı tier'daki plan kartlarını otomatik olarak devre dışı bırakır ve tooltip ile sebebi gösterir.

---

## Mount Preset Kataloğu

DevOpsZon, container'ınızın image ailesine göre en uygun mount path'i otomatik olarak önerir. Bu sayede `/etc` veya `/proc` gibi hassas host dizinlerine volume bağlanması önlenir.

### Kategoriler

| Kategori | Örnek Preset | Mount Path | Hedef Image |
|----------|--------------|------------|-------------|
| **Database** | PostgreSQL data | `/var/lib/postgresql/data` | postgres, timescaledb, citus |
| **Database** | MySQL data | `/var/lib/mysql` | mysql, mariadb, percona |
| **Database** | MongoDB data | `/data/db` | mongo |
| **Database** | Elasticsearch data | `/usr/share/elasticsearch/data` | elasticsearch, opensearch |
| **Cache** | Redis data | `/data` | redis |
| **Cache** | Valkey data | `/bitnami/valkey/data` | valkey |
| **Message Broker** | RabbitMQ data | `/var/lib/rabbitmq` | rabbitmq |
| **Message Broker** | Kafka data | `/var/lib/kafka/data` | kafka, confluentinc/cp-kafka |
| **Web** | NGINX web root | `/usr/share/nginx/html` | nginx |
| **Web** | Apache document root | `/var/www/html` | httpd, apache, php, wordpress |
| **Web** | WordPress uploads | `/var/www/html/wp-content` | wordpress |
| **Logs** | Application logs | `/var/log/app` | (generic) |
| **Observability** | Prometheus data | `/prometheus` | prom/prometheus |
| **Observability** | Grafana data | `/var/lib/grafana` | grafana |
| **Generic** | General data | `/data` | (her image) |
| **Generic** | Application data | `/app/data` | (her image) |
| **Generic** | Application storage | `/app/storage` | (her image) |
| **Generic** | Uploads | `/app/uploads` | (her image) |

### Image Ailesine Göre Öneriler

Servisinizin container image'ına göre sistem otomatik olarak ilgili preset'leri **"Image için önerilen"** etiketiyle üste çıkarır. Örneğin:

- `postgres:16` için → PostgreSQL data preset'i en üstte gösterilir
- `nginx:alpine` için → NGINX web root üstte gösterilir
- `my-custom-app:1.0` için → sadece Generic kategorisindeki preset'ler gösterilir

Generic preset'ler her zaman listede yer alır; özel preset bulunmayan iş yükleri bu seçenekleri kullanabilir.

---

## Servis Yönetim Panelinde Storage Sekmesi

Depolama eklemek veya mevcut ekleri yönetmek için:

1. **Projects** menüsünden servisinizi açın
2. Servis yönetim panelinin sol kenar çubuğundan **Storage** sekmesini seçin
3. Mevcut ekler kart görünümünde listelenir; sağ üstteki **Depolama Ekle** butonu ile yeni ek oluşturabilirsiniz

### Storage Kartlarında Gösterilen Bilgiler

Her depolama kartı şu bilgileri içerir:

- **Volume adı** (PVC kaynağında kullanılan DNS-1123 label)
- **Plan adı** ve boyutu
- **StorageClass** (longhorn veya longhorn-static)
- **Container mount path**
- **Sub-path** (opsiyonel)
- **Preset adı ve kategorisi**
- **Aylık maliyet**
- **Salt okunur** (read-only) işareti
- **Düzenle** ve **Sil** aksiyonları

---

## Yeni Depolama Ekleme

**Depolama Ekle** butonuna tıkladığınızda açılan dialog dört bölümden oluşur:

### 1. Plan Seçimi

Beş plan kart olarak listelenir. Varsayılan olarak **Free** plan seçilidir ve **Default** rozeti ile işaretlenir. Karta tıklayarak plan seçin. Her kartta plan adı, boyut, replika sayısı, reclaim politikası ve aylık ücret gösterilir.

### 2. Mount Path Seçimi

Mount path preset'leri kategorilere ayrılmış pill butonlar olarak gösterilir. Container image'ınıza uygun preset'ler üstte yeşil çerçeve ile **"Image için önerilen"** etiketiyle öne çıkar. Diğer preset'ler altta listelenir.

Bir preset'e tıkladığınızda:

- Mount path otomatik olarak doldurulur
- Varsa `DefaultSubPath` sub-path alanına aktarılır

> **Güvenlik notu:** Kullanıcılar serbest metin olarak mount path giremez. Platform sadece katalogdaki güvenli yolları kabul eder. Bu sayede `/etc`, `/proc`, `/sys` gibi hassas host dizinlerine volume bağlanması engellenir.

### 3. Volume Adı ve Sub-Path

- **Volume adı:** DNS-1123 label formatında benzersiz bir isim girin. Küçük harf, rakam ve tire kullanılabilir (örn. `db-data`, `uploads`, `redis-persist`). Bu isim, Kubernetes'te oluşturulacak PVC kaynağının adının bir parçası olacaktır.
- **Alt yol (sub-path):** Opsiyonel. PVC içindeki göreli bir dizini mount etmek istiyorsanız (örn. `db/prod`) bu alanı kullanın. Birden fazla servis aynı PVC'yi paylaşırken alt klasörleri birbirinden izole etmek için yararlıdır.

### 4. Salt Okunur Mount

Volume'u container içinde `readOnly: true` olarak mount etmek isterseniz kutucuğu işaretleyin. Bu, container'ın volume'a yazmasını engeller — mevcut veriyi değiştirmeyen ve yalnızca okuyan iş yükleri için kullanışlıdır.

**Kaydet** butonuna bastıktan sonra:

1. Backend DB kaydını oluşturur
2. **Otomatik deploy kuyruğa eklenir** (servisinizin mevcut image tag'i ile, yeni build tetiklenmez)
3. Pipeline `base/pvc.yaml` manifest'ini üretir ve Git'e commit eder
4. ArgoCD bu manifest'i sync ederek PVC kaynağını cluster'a uygular
5. Longhorn, plan parametrelerine göre volume'u provision eder
6. Pod yeniden oluşturulur ve volume mount edilir

Sağ üstten **yeşil bildirim** görünecek: _"Depolama kaydedildi — Deploy otomatik tetiklendi, PVC ~30 saniye içinde hazır olacak"_. İlerlemeyi **Pipeline Summary** sekmesinden gerçek zamanlı izleyebilirsiniz.

> 💡 Detay için bu sayfadaki [Otomatik Deploy](#otomatik-deploy) bölümüne bakın.

---

## Mevcut Bir Eki Düzenleme

Kartın altındaki **Düzenle** butonu ile bir eki güncelleyebilirsiniz. Düzenleme dialog'unda şunları değiştirebilirsiniz:

- **Plan:** Aynı StorageClass ve reclaim politikası içinde daha büyük bir plana yükseltebilirsiniz (ör. Small → Medium → Large)
- **Sub-path:** Değiştirilebilir
- **Salt okunur:** Açılıp kapatılabilir

**Kaydet** basıldığında Create flow'undaki gibi **otomatik deploy tetiklenir**. Plan yükseltmesi (ör. 10 GB → 50 GB) Longhorn volume expansion ile uygulanır — pod restart gerekebilir.

### Düzenlenemeyenler

- **Mount path:** Kubernetes, mevcut bir PVC'nin bağlandığı path'i mount etme stratejisinin immutable bir parçası olarak görür. Değiştirmek için eski eki silip yeni bir ek oluşturmanız gerekir.
- **Volume adı:** PVC kaynağının adının parçasıdır, değişmez.
- **StorageClass geçişi** (örn. Free → Medium): Kubernetes, var olan bir PVC'nin StorageClass'ını değiştirmeye izin vermez. Platform bu tür plan değişimlerini engeller. Geçiş yapmak istiyorsanız **önce eski eki silin, sonra yeni planla yeni bir ek oluşturun**.
- **Boyut küçültme:** Longhorn, volume'u expand eder ama shrink edemez. Plan boyutunu düşürmeye çalışırsanız hata alırsınız.

---

## Bir Eki Silme

Kartın altındaki **Sil** butonu ile bir eki kaldırabilirsiniz. **Silme işlemi geri alınamaz.** Plan tier'ı, StorageClass veya reclaim politikası fark etmez — platform tarafından yapılan her depolama silme işlemi tam temizlik anlamına gelir.

### Silme Sırasında Ne Olur?

Onay verdiğiniz anda backend şu adımları sırayla uygular:

1. **PVC ↔ PV çözümlemesi** — Silinen ekin Longhorn volume adı (CSI volume handle) cluster'dan okunur.
2. **Reclaim policy override** — PV'nin `persistentVolumeReclaimPolicy` alanı anında `Delete`'e patch'lenir. Retain policy ile seed'lenmiş ücretli plan PV'leri bile bu adımdan sonra silineceklerdir.
3. **Longhorn snapshot temizliği** — `snapshots.longhorn.io` içinde bu volume'a ait kayıtlar bulunur ve silinir.
4. **Longhorn backup temizliği** — `backups.longhorn.io` ve `backupvolumes.longhorn.io` içindeki S3 yedek referansları silinir; bu işlem aynı zamanda S3 arşivindeki yedek dosyalarını da kaldırır.
5. **Longhorn volume delete** — `volumes.longhorn.io` kaynağı explicit silinir.
6. **PVC delete** — Cluster'dan PVC kaldırılır; PV artık Delete policy'li olduğu için underlying disk de geri alınır.
7. **DB kaydı** — Attachment DB'den silinir.
8. **Auto-deploy** — Servis manifest'i regenerate edilir (silinen PVC artık dahil değil) ve Git'e commit edilir. ArgoCD zaten temizlenmiş kaynak için no-op yapar.

Tüm bu adımlar atomik değildir ama **best-effort** çalıştırılır: herhangi bir adımda Longhorn API geçici olarak erişilemezse, backend uyarı log'u yazar ve sonraki adımlara devam eder. Bu sayede geçici bir cluster arızası silme işlemini sonsuza kadar bloke edemez.

### Onay Dialog'u

Sil butonuna bastığınızda aşağıdaki uyarı ile karşılaşırsınız:

> **"'<volume-adı>' depolama eki kalıcı olarak silinecek. PVC, disk üzerindeki tüm veriler, Longhorn snapshot'ları ve S3 yedekleri geri alınamaz şekilde yok edilecek. Devam etmek istiyor musunuz?"**

> **Önemli:** Silme işlemi sadece dialog'da **Sil** butonuna bastığınızda yürütülür. Platform, onay alınmadığı sürece backend'e silme isteği göndermez. Kritik verileriniz için silmeden önce manuel bir yedek aldığınızdan emin olun — silme sonrası **geri dönüş yoktur**.

---

## Otomatik Deploy

DevOpsZon, storage ekinizde yapılan her değişikliği (ekleme / güncelleme / silme) **otomatik olarak bir sonraki deploy'a dönüştürür**. Yani kullanıcı olarak artık "Storage → Save" yapıp sonra ayrıca **Dashboard → Deploy** tıklamak zorunda değilsiniz — tek adım yeterli.

### Nasıl çalışır?

1. Save butonuna basıldığında backend DB kaydını yazıyor
2. Kayıt başarılı olur olmaz backend **servisinizin mevcut image tag'i** ile `IDeploymentTriggerService.EnqueueDeploymentAsync` çağırıyor
3. MassTransit / RabbitMQ üzerinden `IDeploymentQueueMessage` yayınlanıyor
4. `DeploymentQueueConsumer` mesajı alıyor → pipeline çalışıyor → PVC manifest'i üretiliyor → Git'e commit → ArgoCD sync → Longhorn PV oluşturuyor

> **Not:** Auto-deploy servisin **aynı image'ını** yeniden kullanır. Yeni bir build tetiklenmez. Sadece YAML manifest'ler regenerate edilir ve storage attachment değişikliği uygulanır.

### Üç farklı bildirim senaryosu

| Durum | Bildirim rengi | Mesaj |
|---|---|---|
| ✅ Kayıt başarılı + deploy kuyruğa eklendi | **Yeşil (başarı)** | "Deploy otomatik tetiklendi — PVC ~30 saniye içinde hazır olacak" |
| ⚠️ Kayıt başarılı + deploy kuyruğa eklenemedi (RabbitMQ kapalı, vb.) | **Sarı (uyarı)** | "Kayıt başarılı ama deploy tetiklenemedi: {neden}. Dashboard'dan manuel tetikleyin" |
| ℹ️ Platform tarafında auto-deploy kapatılmış | **Mavi (bilgi)** | "Kayıt başarılı. Platform tarafında auto-deploy kapalı — Dashboard'dan manuel tetikleyin" |

### Deploy ile rollout çakışması

Servisiniz aktif bir Canary / BlueGreen rollout'un ortasındayken storage değiştirirseniz:

- Platform servis bazlı bir **deployment kilidi** kullanır (Redis üzerinden)
- Storage change deploy mesajı kuyruğa düşer ama aktif rollout bitene kadar **bekler**
- Aktif rollout tamamlandığında (veya abort edildiğinde) yeni deploy sıraya girer
- Storage değişiklikleri **kaybolmaz** — sadece uygulaması gecikir

### Platform yöneticileri için

Platform yöneticileri `appsettings.json` üzerinden auto-deploy davranışını kapatabilir:

```json
{
  "Storage": {
    "autoDeployOnAttachmentChange": false
  }
}
```

`false` olduğunda kullanıcı her storage değişikliğinden sonra deploy'u manuel başlatmak zorunda kalır. Dev/test ortamlarında veya manuel deploy disiplini istenen organizasyonlarda kullanılabilir. **Default değeri `true`** ve üretimde açık kalması önerilir.

### İlerlemeyi izleme

Auto-deploy tetiklendikten sonra ilerlemeyi iki yerden izleyebilirsiniz:

1. **Pipeline Summary sekmesi**: Canlı step-by-step progress (SignalR üzerinden), her step için yeşil/kırmızı tick
2. **Dashboard sekmesi**: Aktif rollout kartı + pod durumu

İlerleme `RolloutStatusHub` SignalR hub'ı üzerinden gerçek zamanlı gönderilir; sayfa yenilemek gerekmez.

---

## Yedekleme ve Geri Yükleme

DevOpsZon, Longhorn üzerinden iki otomatik yedekleme mekanizması sağlar:

### Günlük Snapshot

Her gün **02:00**'de tüm volume'ların snapshot'ı alınır. Son **7 gün**'lük snapshot saklanır. Snapshot'lar cluster içinde tutulur; bir node veya pod kaybı durumunda hızlı geri alma için kullanılabilirler.

### Haftalık S3 Backup

Her **Pazar 03:00**'te tüm volume'ların tam yedeği platform yöneticisinin yapılandırdığı S3-uyumlu depolama hedefine gönderilir. Son **4 hafta**'lık yedek saklanır. Snapshot'lar cluster içindeyken S3 yedekleri cluster dışında tutulduğu için cluster çöküşü durumunda bile verinizi geri alabilirsiniz.

> **Not:** S3 yedek hedefi yalnızca platform yöneticisi tarafından yapılandırıldığında etkindir. Kendi cluster'ınızı bağladıysanız, admin bölümünden "Longhorn" purpose'lu bir ManagedObjectStorage kaydı oluşturmanız gerekir.

### Bir Snapshot'tan Geri Yükleme

Şu anda UI üzerinden snapshot geri yükleme desteklenmemektedir. Veri kaybı yaşarsanız platform destek ekibine başvurun — snapshot'tan geri yükleme platform admin'i tarafından manuel olarak yapılabilir.

---

## Pipeline Entegrasyonu

Bir depolama eki eklediğinizde (veya güncelleyip silindiğinizde) platform, **otomatik tetiklenen** bir deploy sırasında aşağıdaki adımları gerçekleştirir:

0. **AppService auto-deploy enqueue** — Attachment DB'ye yazılır yazılmaz `IDeploymentTriggerService.EnqueueDeploymentAsync` çağrılır (servis mevcut image tag'i ile)
1. **AcquireDeploymentLockStep** — Servis bazlı Redis kilidi alınır (aktif rollout varsa bekler)
2. **DeploymentContextFactory** — Attachment + plan bilgisi snapshot'a yüklenir
3. **PvcContributor** — Servisin aktif eklerini tarayarak `base/pvc.yaml` dosyasını oluşturur (her ek için bir `ReadWriteMany` PersistentVolumeClaim)
4. **RolloutContributor** — Pod template'ine `volumes` (`persistentVolumeClaim`) ve container spec'ine `volumeMounts` ekler
5. **KustomizationContributor** — `base/kustomization.yaml`'daki resource listesine `pvc.yaml`'ı dahil eder
6. **ValidateDeploymentStep** — Cluster'daki mevcut PVC'lerin `accessModes` ve `storageClassName` alanlarını hedef manifest ile karşılaştırır; uyumsuzluk varsa (örn. legacy RWO PVC) deploy'u durdurur ve kullanıcıya "servisi silip yeniden oluştur" mesajı verir
7. **PushToGitStep** — Manifest'leri servis repo'suna commit + push eder
8. **SyncArgoCdStep** — ArgoCD'ye sync tetikler (veya auto-sync açıksa ArgoCD polling ile yakalar)
9. **Longhorn Provisioner** — PVC'yi karşılayacak PV'yi dinamik olarak oluşturur
6. **Pod Restart** — Yeni pod volume'u mount edilmiş olarak ayağa kalkar

Manifest'ler servis repo'nuza commit edilir; ArgoCD bunları declarative olarak uygular. Bu sayede her zaman "ne çalışıyor" sorusunun cevabı Git'teki manifest'tir.

---

## Sık Sorulan Sorular

### Aynı servise birden fazla depolama ekleyebilir miyim?

Evet. Örneğin bir web uygulamanın `uploads`, `cache` ve `logs` için üç ayrı ek oluşturabilirsiniz. Her ek bağımsız bir PVC olarak provision edilir. Aynı volume adını veya aynı mount path'i iki ekte kullanamazsınız — platform DB seviyesinde bunu engeller.

### Aynı disk'i iki farklı servis paylaşabilir mi?

Hayır — bir PVC tek bir servise aittir. Ama **aynı servisin birden fazla pod'u** (canary + stable, replica 1 + replica 2 gibi) aynı PVC'ye bağlanabilir. Platform bunu otomatik halleder: DevOpsZon pipeline'ı **her PVC'yi `ReadWriteMany` (RWX) olarak oluşturur** — tier, strateji veya replika sayısı ne olursa olsun.

Bu karar, "Always RWX" mimarisi kapsamında bilinçli olarak platform seviyesinde alınmıştır. Rollout stratejilerinin üçü de (Canary / BlueGreen / AutoPromote) rollout sırasında eski ve yeni pod'ları aynı anda ayakta tutar, ikisi de aynı PVC'yi mount eder; bu yüzden RWX kaçınılmazdır. Tek replika + tek strategy olan senaryolar bile gelecekteki HPA veya replika artırma işlemlerinden etkilenmesin diye RWX olarak provision edilir.

> ℹ️ **Performans notu**: Longhorn RWX volume'lar, volume başına bir share-manager (NFS) pod'u üzerinden çalıştığı için saf RWO block device'a kıyasla latency ve IOPS açısından ~2-3x daha yavaştır. Çok yoğun random I/O gerektiren veritabanları için bu fark hissedilir olabilir; böyle durumlarda replika sayısını 1'de tutarak veya `fsync` tuning yaparak bottleneck'i azaltabilirsiniz. Hot-path veritabanları için managed addon (PostgreSQL, Valkey, RabbitMQ) tercih etmek daha iyi bir pratiktir — onlar kendi optimize edilmiş block-device storage'larını kullanırlar.

### Mevcut servisim RWO volume ile oluşturulmuştu, şimdi ne olur?

"Always RWX" refactor'undan önce oluşturulmuş servislerin PVC'leri `ReadWriteOnce` (RWO) modundaydı. Kubernetes, mevcut bir PVC'nin `spec.accessModes` alanını değiştirmeye izin vermez. Pipeline artık her PVC'yi RWX olarak üretmeye çalıştığı için bu servislerin **bir sonraki deploy'unda** platform "StorageAccessMode uyumsuzluğu" hatası verecek ve deploy'u durduracak.

Çözüm: **Servisi silip yeniden oluşturun.** Yeni servis otomatik olarak RWX PVC alacak ve tüm rollout stratejileri / HPA / multi-replica özellikleri sorunsuz çalışacaktır. Veri migrasyonu yapmanız gerekiyorsa önce eski servisin PVC'sini snapshot'a alın, sonra yeni servise manuel olarak geri yükleyin (destek ekibinden yardım isteyebilirsiniz).

### Volume'um dolunca ne olur?

Uygulamanız disk dolu hatası (`ENOSPC`) alır. Büyütmek için servis yönetim panelinden mevcut eki **Düzenle** diyalogundan daha büyük bir plana geçin — platform Longhorn'un volume expansion özelliğini kullanarak volume'u büyütür (pod restart gerekebilir).

### Yanlışlıkla sildiğim bir eki geri getirebilir miyim?

- **Retain planlarda:** Underlying PV hâlâ cluster'da durur. Platform admin'i `kubectl` ile yeni bir PVC oluşturup PV'ye bind edebilir. Destek ekibine başvurun.
- **Delete planlarda (Free):** Geri getirme mümkün değildir. Veri kalıcı olarak silinir.

### Production için Free plan yeterli mi?

Hayır. Free plan yalnızca 1 GB alan, 2 replika ve `Delete` reclaim politikası sunar. Production iş yükleri için **en az Small** planını, kritik veritabanları için **Medium ve üstü** planları öneririz.

### Storage ekledim ama deploy kuyruğa düşmedi — ne yapayım?

Sarı uyarı bildirimi "Kayıt başarılı ama deploy tetiklenemedi" şeklinde göründüyse genelde RabbitMQ veya mesaj kuyruğu geçici olarak erişilemez demektir. Attachment DB'de kayıtlı olduğu için **veri kaybı yok** — **Dashboard → Deploy** butonuna tıklayarak manuel deploy tetikleyin. Pipeline çalışırken `DeploymentContextFactory` attachment'ınızı otomatik dahil edecek.

### Sub-path'i ne zaman kullanmalıyım?

Sub-path'i iki durumda kullanabilirsiniz:

1. **Aynı PVC'de birden fazla alt klasörü farklı container'lara mount ederken** (pod içinde çok container'lı senaryo — nadir)
2. **Büyük bir paylaşımlı dizini mantıksal alt bölümlere ayırmak istediğinizde** — örneğin bir WordPress sitesinde `wp-content/uploads` yerine sadece `wp-content` alt klasörünü mount etmek

Basit tek-container servisler için sub-path genellikle gerekmez.

---

## Güvenlik ve İzolasyon

- **Multi-tenant isolation:** Her attachment `TenantId` ile ilişkilendirilir. Tenant'ınızın dışındaki başka bir servis sizin PVC'lerinize erişemez.
- **Mount path whitelist:** Platform sadece katalogdaki güvenli mount path'lerini kabul eder. Serbest path girişi yalnızca platform admin'ine açıktır ve `/data`, `/app`, `/var/lib`, `/var/log`, `/home`, `/mnt`, `/opt`, `/usr/share`, `/prometheus`, `/bitnami` prefix'leri ile sınırlıdır.
- **Encryption at rest:** Longhorn volume'ları, cluster yöneticisinin yapılandırdığı şifreleme ayarlarına göre disk üzerinde şifrelenir.
- **Backup credential isolation:** S3 backup credential'ları cluster'da ayrı bir Secret'ta tutulur. Container'ınız bu Secret'a erişemez.

---

## İlgili Rehberler

- [Servis Yönetimi](service-guide.md) — Storage sekmesinin içinde yer aldığı servis yönetim paneli
- [Faturalandırma ve Kaynak Planları](billing-plans.md) — Disk planlarının genel ücretlendirme modelindeki yeri
- [Yönetilen Servisler (Managed Addon'lar)](managed-addons.md) — PostgreSQL, RabbitMQ, Valkey gibi yönetilen servislerin kendi depolama yönetimi (ayrı bir paradigma)
