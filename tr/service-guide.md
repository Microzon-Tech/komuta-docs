# Servis Yönetimi

Servis yönetimi, DevOpsZon'un en kapsamlı bölümüdür. Bir servisi seçtiğinizde, o servise ait tüm operasyonları — deploy'dan izlemeye, log'dan uyarılara — tek bir panelden yönetirsiniz.

---

## Servis Yönetimi Paneli

Servis yönetimi paneline erişmek için proje listesinden bir servisin adına tıklayın veya sol menüdeki **Services** bölümünden servis seçin. Panel, üst kısımdaki sekme çubuğu ile farklı yönetim alanlarına geçiş yapmanızı sağlar.

### Sekme Yapısı

| Sekme | Amaç |
|-------|------|
| **Dashboard** | Servisin anlık durumu, trafik akışı, pod bilgileri |
| **Environment Variable** | Ortam değişkenlerinin yönetimi |
| **Ingress Management** | Giriş kuralları, hostname ve trafik yapılandırması |
| **HPA** | Otomatik ölçekleme (Horizontal Pod Autoscaler) ayarları |
| **Resources** | CPU ve bellek kaynak limitleri |
| **Ports** | Servis port yapılandırması |
| **Storage** | Longhorn destekli kalıcı disk (PVC) yönetimi |
| **Health Probes** | Sağlık kontrol (Liveness/Readiness) ayarları |
| **Alert Management** | Servise özel uyarı kuralları |
| **Pipeline Summary** | CI/CD pipeline çalıştırma geçmişi |
| **Logs** | Gerçek zamanlı log görüntüleme |
| **History & Rollback** | Deployment geçmişi ve geri alma |

> **Not:** SaaS modunda ek olarak **Resource Plan** ve **Invoice** sekmeleri görünür; bazı operasyonel sekmeler farklılık gösterebilir.

---

## Dashboard

Dashboard, servisinizin anlık durumunu gösteren ana görünümdür. Tüm bilgiler **SignalR** üzerinden gerçek zamanlı güncellenir — sayfa yenilemesine gerek yoktur.

### Canlı Trafik Görselleştirmesi

Dashboard'un merkezinde **Canlı Trafik Analizi** bileşeni yer alır. Bu interaktif görselleştirme, servisinize gelen trafiğin ingress'ten pod'lara kadar olan akışını canlı olarak gösterir:

- **HTTPRoute / Ingress noktası:** Gelen trafiğin giriş noktası (üstte)
- **Servis kartları:** Active ve (varsa) Preview/Canary servis grupları, her biri üzerinde trafik yüzdesi ve ilerleme çubuğu
- **Pod düğümleri:** Her pod'un faz durumu (Running, Pending, Failed), imaj etiketi ve sağlık bilgisi (altta)
- **Trafik okları:** Ingress'ten servislere, servislerden pod'lara akan trafiğin yönü ve yoğunluğu

Görselleştirme, kullandığınız **rollout stratejisine** göre farklı bilgiler sunar. Stratejilerin detaylı açıklaması aşağıdadır.

### Durum Bilgileri

Dashboard'da servisinizle ilgili özet bilgiler görüntülenir:

- **Pod sayısı ve durumu:** Running, Pending, Failed
- **Son deployment revizyonu:** Aktif sürüm bilgisi
- **HPA durumu:** Otomatik ölçekleme aktifse mevcut/minimum/maksimum replica sayısı
- **Kaynak tüketimi:** CPU ve bellek kullanımı
- **Alert sayısı:** Aktif uyarı sayısı
- **Pipeline durumu:** Son build'in task dot'ları ile adım adım durumu (rollout-status görevi için özel vurgu)

---

## Rollout Stratejileri

DevOpsZon, **Argo Rollouts** altyapısını kullanarak üç farklı deployment stratejisi destekler. Her strateji, yeni sürümünüzü farklı bir yaklaşımla canlıya almanızı sağlar.

### Strateji Karşılaştırması

| Özellik | Canary | Blue/Green | Auto-Promote |
|---------|:------:|:----------:|:------------:|
| **Trafik kontrolü** | Kademeli (yüzde ile) | Anlık geçiş | Otomatik geçiş |
| **Preview ortamı** | Yok (tek hostname) | Var (ayrı hostname) | Var (ayrı hostname) |
| **Manuel onay gerekli** | Evet (her adımda) | Evet (promote) | Hayır (otomatik) |
| **Trafik yüzdesi ayarı** | Evet (adım adım artış) | Hayır (%100 kesme) | Hayır |
| **Hostname sayısı** | 1 (paylaşımlı) | 2 (active + preview) | 2 (active + preview) |
| **Anında geri dönüş** | Evet (abort) | Evet (abort) | Evet (abort) |
| **Önerilen kullanım** | Yüksek trafikli servisler | Kritik servisler | Güvenli otomatik dağıtım |

---

### Canary Stratejisi

Canary stratejisi, yeni sürümünüzü canlı trafiğin küçük bir yüzdesine yönlendirerek gerçek kullanıcılarla test etmenizi sağlar. Sorun yoksa trafik yüzdesini kademeli olarak artırırsınız.

#### Canary Nasıl Çalışır?

```
Adım 1: %5 trafik yeni sürüme → İzle → Sorun yok mu?
Adım 2: %25 trafik yeni sürüme → İzle → Sorun yok mu?
Adım 3: %50 trafik yeni sürüme → İzle → Sorun yok mu?
Adım 4: %100 trafik yeni sürüme → Tamamlandı ✓
```

1. Yeni sürüm deploy edildiğinde, trafiğin yalnızca küçük bir yüzdesi (ör: %5) yeni pod'lara yönlendirilir
2. Eski (stable) pod'lar kalan trafiği almaya devam eder
3. Her adımda rollout **duraklar (Paused)** ve sizden onay bekler
4. Metrikleri, logları ve uyarıları kontrol ettikten sonra bir sonraki adıma geçersiniz
5. Son adımda tüm trafik yeni sürüme aktarılır ve eski pod'lar temizlenir

#### Dashboard'da Canary Görünümü

Canary stratejisi aktifken dashboard şu şekilde görünür:

**Üst Banner:**
- **"Yeni sürüm hazırlanıyor — Full Promote gerekli"** bilgi mesajı
- **Paused** rozeti: Canary duraklatılmış, sonraki adım için onay bekleniyor
- Strateji rozeti: **Canary** ikonu

**Canlı Trafik Analizi:**
- **Tek hostname** gösterilir (Canary'de ayrı preview hostname yoktur)
- **Stable servis kartı:** Eski sürüm, ör: **%75 trafik** — yeşil ilerleme çubuğu
- **Canary servis kartı:** Yeni sürüm, ör: **%25 trafik** — mavi ilerleme çubuğu
- **Pod grupları:** Stable pod'lar ve Canary pod'lar ayrı gruplar halinde gösterilir
- Her pod üzerinde imaj etiketi ve durum bilgisi

**Trafik yüzdeleri**, Gateway API üzerindeki ağırlık kuralları (weight) ile belirlenir ve gerçek zamanlı olarak güncellenir.

#### Canary Aksiyonları

| Aksiyon | Nerede | Açıklama |
|---------|--------|----------|
| **Resume (Sonraki Adım)** | Trafik Dağılımı diyaloğu | Bir sonraki trafik adımına ilerler (ör: %5 → %25) |
| **Full Promote** | Banner + Trafik Dağılımı diyaloğu | Tüm trafiği anında yeni sürüme yönlendirir (%100) |
| **Abort** | Banner | Canary'yi iptal eder, tüm trafiği eski sürüme döndürür |

**Trafik Dağılımı Diyaloğu:**

Banner'daki **"Full Promote"** linkine veya trafik kartlarına tıkladığınızda açılan diyalogda:
- **Mevcut trafik dağılımı** gösterilir (ör: Stable %75, Canary %25)
- **Sonraki adım** bilgisi gösterilir (ör: %50)
- **Resume** butonu ile sonraki adıma geçebilirsiniz
- **Full Promote** butonu ile tüm trafiği yeni sürüme alabilirsiniz

> **Önemli:** Canary'de trafik yüzdesi yalnızca **artırılabilir**. Trafik yüzdesini düşürmek istiyorsanız **Abort** ile rollout'u iptal edip önceki sürüme dönmeniz gerekir.

#### Canary Trafik Adımları

Trafik adımları, servisinizin rollout manifest'inde tanımlanır. DevOpsZon, hedef ağırlığa ulaşmak için gerekli sayıda `promote` komutunu otomatik olarak çalıştırır. Örnek bir adım yapısı:

| Adım | Canary Ağırlığı | Stable Ağırlığı | Durum |
|:----:|:---------------:|:---------------:|-------|
| 1 | %5 | %95 | Duraklatılır — onay beklenir |
| 2 | %25 | %75 | Duraklatılır — onay beklenir |
| 3 | %50 | %50 | Duraklatılır — onay beklenir |
| 4 | %100 | %0 | Tamamlandı — eski pod'lar temizlenir |

#### Canary Ne Zaman Kullanmalı?

- Yüksek trafikli servislerde riski minimize etmek istediğinizde
- Yeni sürümün performansını gerçek trafik ile ölçmek istediğinizde
- A/B testi benzeri senaryolarda
- Kademeli ve kontrollü geçiş istediğinizde

---

### Blue/Green Stratejisi

Blue/Green stratejisi, yeni sürümü tamamen ayrı bir ortamda (Preview) hazırlayıp test ettikten sonra tek bir adımda canlıya almanızı sağlar. İki ortam paralel çalışır — biri canlı trafiği alırken diğeri test edilir.

#### Blue/Green Nasıl Çalışır?

```
┌─────────────────────────────────────────────────┐
│  Active (Blue)        Preview (Green)            │
│  ┌──────────┐         ┌──────────┐              │
│  │ Eski     │ ←trafik │ Yeni     │ ←test        │
│  │ Sürüm    │  %100   │ Sürüm    │  (preview)   │
│  └──────────┘         └──────────┘              │
│                                                  │
│  ────── Promote ──────→                          │
│                                                  │
│  ┌──────────┐         ┌──────────┐              │
│  │ (temiz-  │         │ Yeni     │ ←trafik      │
│  │  lenir)  │         │ Sürüm    │  %100        │
│  └──────────┘         └──────────┘              │
└─────────────────────────────────────────────────┘
```

1. Yeni sürüm deploy edildiğinde, **Preview** ortamında ayrı pod'lar oluşturulur
2. Canlı trafik tamamen **Active** (eski sürüm) pod'larından akmaya devam eder
3. Preview hostname üzerinden yeni sürümü test edebilirsiniz
4. Rollout **duraklar (Paused)** ve sizden onay bekler
5. **Promote** ile tüm trafik Preview'a (yeni sürüm) yönlendirilir — geçiş anlıktır
6. Eski Active pod'ları temizlenir

#### Dashboard'da Blue/Green Görünümü

Blue/Green stratejisi aktifken dashboard şu şekilde görünür:

**Üst Banner:**
- **"Yeni sürüm preview ortamında hazırlanıyor"** bilgi mesajı
- Yeni ve eski sürüm bilgileri (revizyon numaraları)
- **Full Promote** linki (preview sağlıklıysa)
- **Preview Link** butonu: Preview hostname'ine doğrudan erişim
- **Paused** rozeti
- Strateji rozeti: **BlueGreen** ikonu

**Canlı Trafik Analizi:**
- **İki hostname** gösterilir:
  - **Aktif:** `my-api-a1b2c3d4.devopszon.com` — canlı trafik
  - **Önizleme:** `my-api-a1b2c3d4-preview.devopszon.com` — test trafiği
- **Active servis kartı:** Eski sürüm pod'ları — %100 canlı trafik
- **Preview servis kartı:** Yeni sürüm pod'ları — yalnızca preview trafiği
- Her iki grup için ayrı pod düğümleri

**Sağlıksız Preview Durumu:**

Eğer preview ortamındaki pod'lar sağlıksızsa (crash, readiness probe başarısız vb.), dashboard şunu gösterir:
- **Uyarı mesajı:** "Preview ortamı sağlıksız — Full Promote önerilmez"
- **Abort** butonu ön plana çıkar
- Full Promote linki gizlenir veya uyarılı olarak gösterilir

#### Blue/Green Aksiyonları

| Aksiyon | Nerede | Açıklama |
|---------|--------|----------|
| **Full Promote** | Banner + Trafik Dağılımı diyaloğu | Tüm trafiği yeni sürüme (Preview → Active) yönlendirir |
| **Abort** | Banner | Preview'ı iptal eder, eski sürüm çalışmaya devam eder |
| **Preview Link** | Banner + Hostname listesi | Preview hostname'ine doğrudan erişim |

**Trafik Dağılımı Diyaloğu:**

Blue/Green'de trafik yüzdesi ayarlanamaz — geçiş **tek adımda %100** olarak gerçekleşir. Diyalogda:
- Mevcut durum bilgisi gösterilir
- **Full Promote** butonu ile geçiş onaylanır
- Açıklama: "Tüm trafik yeni sürüme tek adımda yönlendirilecektir"

#### Blue/Green Hostname'leri

Blue/Green stratejisinde her servise iki hostname atanır:

| Tür | Format | Kullanım |
|-----|--------|----------|
| **Active** | `{servis-adı}-{id}.devopszon.com` | Canlı trafik — kullanıcılarınız bu adresi kullanır |
| **Preview** | `{servis-adı}-{id}-preview.devopszon.com` | Test trafiği — yeni sürümü doğrulamak için kullanın |

Promote sonrası Preview hostname geçerliliğini yitirir ve rollout tamamlandığında görünümden kaldırılır.

#### Blue/Green Ne Zaman Kullanmalı?

- Kritik servislerde sıfır riskli geçiş istediğinizde
- Yeni sürümü canlıya almadan önce kapsamlı test yapmanız gerektiğinde
- Anlık geri dönüş (rollback) yeteneği şart olduğunda
- Kademeli trafik yönetimi gereksiz olduğunda

---

### Auto-Promote Stratejisi

Auto-Promote, Blue/Green stratejisinin otomatikleştirilmiş versiyonudur. Preview ortamı hazırlandıktan sonra, belirli koşullar sağlandığında trafik geçişi **otomatik olarak** gerçekleşir — manuel onay gerekmez.

#### Auto-Promote Nasıl Çalışır?

1. Yeni sürüm Preview ortamında oluşturulur (Blue/Green ile aynı)
2. Preview pod'ları sağlıklı hale gelir
3. Sistem, belirlenen bekleme süresi boyunca preview'ı izler
4. Sorun tespit edilmezse **otomatik olarak promote** gerçekleşir
5. Tüm trafik yeni sürüme yönlendirilir

#### Dashboard'da Auto-Promote Görünümü

Auto-Promote stratejisi aktifken dashboard şu şekilde görünür:

**Bekleme Durumu:**
- Ekranda **"Yeni versiyon hazırlanıyor"** overlay'i gösterilir
- Roket animasyonu ile görsel bekleme göstergesi
- Manuel promote butonları **gizlenir** — süreç otomatiktir
- Kullanıcıdan ek aksiyon beklenmez

**Tamamlanma:**
- Otomatik promote gerçekleştikten sonra overlay kaybolur
- **"Deployment başarılı"** bildirimi gösterilir
- Dashboard normal görünümüne döner

**Hata Durumu:**
- Preview pod'ları sağlıksızsa **Abort** banner'ı gösterilir
- Uyarı mesajı: "Preview ortamı sağlıksız — otomatik geçiş yapılamıyor"
- Kullanıcı yalnızca **Abort** aksiyonu alabilir

#### Auto-Promote Aksiyonları

| Aksiyon | Nerede | Açıklama |
|---------|--------|----------|
| **Abort** | Banner (yalnızca hata durumunda) | Preview sağlıksızsa rollout'u iptal eder |

> **Not:** Auto-Promote stratejisinde kullanıcı müdahalesi minimum seviyededir. Promote otomatik olarak gerçekleşir; kullanıcı yalnızca sorun durumunda Abort ile müdahale edebilir.

#### Auto-Promote Ne Zaman Kullanmalı?

- CI/CD sürecinize tam güvendiğinizde
- Hızlı ve otomatik dağıtım istediğinizde
- Manuel onay darboğazını ortadan kaldırmak istediğinizde
- Geliştirme ve staging ortamlarında

---

### Stale Rollout Politikası

Canary veya Blue/Green stratejisinde, bir rollout uzun süre **Paused** durumda kalabilir (kullanıcı promote etmeyi unutmuşsa veya ertelemişse). DevOpsZon, bu "stale" (bayatlamış) rollout'ları otomatik olarak yönetmek için platform düzeyinde bir politika sunar.

#### Nasıl Çalışır?

- Platform yöneticisi, **maksimum bekleme süresi** (ör: 60 dakika) tanımlar
- Bir rollout bu süreyi aşacak şekilde Paused kalırsa:
  1. Preview pod'larının sağlık durumu kontrol edilir
  2. Sağlıklıysa → **otomatik Full Promote** gerçekleşir
  3. Sağlıksızsa → **otomatik Abort** gerçekleşir
- Belirli servisler bu politikadan **hariç tutulabilir**

> **Not:** Stale rollout politikası, Canary ve Blue/Green stratejileri için geçerlidir. Auto-Promote stratejisi zaten kendi otomatik geçişini yönettiğinden bu politikanın kapsamı dışındadır.

---

### Rollout Durumları

Deploy sürecinde servisiniz aşağıdaki durumlardan geçer. Bu durumlar dashboard'daki banner, rozetler ve trafik görselleştirmesinde anlık olarak gösterilir:

| Durum | Görsel | Anlamı |
|-------|--------|--------|
| **Progressing** | Animasyonlu ilerleme | Deploy devam ediyor, yeni pod'lar oluşturuluyor |
| **Paused** | Sarı rozet | Kullanıcı müdahalesi bekleniyor (Canary adımı veya Blue/Green promote) |
| **Healthy / Completed** | Yeşil rozet | Deploy başarıyla tamamlandı, tüm pod'lar sağlıklı |
| **Degraded** | Kırmızı rozet | Bazı pod'lar sağlıksız; müdahale gerekebilir |

Tüm bu durum geçişleri **SignalR** üzerinden gerçek zamanlı bildirilir. Ayrıca pipeline'daki **rollout-status** task dot'u da (başlık çubuğundaki küçük durum göstergeleri) rollout'un durumuna göre renklenir.

### Rollout Sırasında Deploy Kilidi

Bir rollout **Paused** durumdayken (Canary adımı beklerken veya Blue/Green promote beklerken), aynı servise **yeni bir deploy tetiklenemez**. Önce mevcut rollout'u Promote veya Abort etmeniz gerekir.

Bu mekanizma, çakışan deploy'ların önüne geçerek tutarlılığı korur

---

## Ortam Değişkenleri (Environment Variables)

Servislerinizin çalışma zamanında ihtiyaç duyduğu konfigürasyon değerlerini yönetin.

### Değişken Ekleme

1. **Environment Variable** sekmesine gidin
2. **Ekle** butonuna tıklayın
3. Anahtar (Key) ve değer (Value) girin
4. Gizli değerler için **Secret** seçeneğini işaretleyin
5. **Kaydet** butonuna tıklayın

> **Önemli:** Ortam değişkeni değişiklikleri bir sonraki deploy'da aktif olur. Mevcut pod'lara anında uygulanmaz.

### Secret Değişkenler

Secret olarak işaretlenen değişkenler:
- Kubernetes Secret olarak saklanır
- Panelde maskelenmiş olarak gösterilir (`••••••••`)
- Sadece yetkili kullanıcılar tarafından görüntülenebilir

---

## Ingress Yönetimi

Servisinize dışarıdan nasıl erişileceğini yapılandırın.

### Hostname

Her servis oluşturulduğunda otomatik bir hostname atanır:

```
{servis-adı}-{benzersiz-id}.devopszon.com
```

Blue/Green stratejisinde ek olarak preview hostname'i oluşturulur:

```
{servis-adı}-{benzersiz-id}-preview.devopszon.com
```

### Özel Domain

Kendi alan adınızı bir servise bağlamak için **Domains** sayfasını kullanın. Detaylar için **Ingress ve Domain Yönetimi** rehberine bakın.

### Trafik Yapılandırması

Ingress Management sekmesinde şu ayarları yapabilirsiniz:

| Ayar | Açıklama |
|------|----------|
| **Host kuralları** | Hangi hostname'lerin bu servise yönlendirileceği |
| **Path kuralları** | URL path bazlı yönlendirme |
| **TLS** | SSL/TLS sertifika yapılandırması |
| **Backend port** | Servisin dinlediği port |

---

## Otomatik Ölçekleme (HPA)

Horizontal Pod Autoscaler, servisinizin yük altında otomatik olarak ölçeklenmesini sağlar.

### HPA Yapılandırması

| Parametre | Açıklama |
|-----------|----------|
| **Minimum Replica** | En az çalışacak pod sayısı |
| **Maksimum Replica** | En fazla ölçeklenecek pod sayısı |
| **CPU Hedefi (%)** | Hedef CPU kullanım yüzdesi |
| **Bellek Hedefi (%)** | Hedef bellek kullanım yüzdesi |

Kubernetes, belirlenen metriklere göre pod sayısını otomatik artırır veya azaltır.

---

## Kaynak Yönetimi (Resources)

Her servis için CPU ve bellek limitleri belirleyin.

| Parametre | Açıklama |
|-----------|----------|
| **CPU Request** | Servisin garanti ettiği minimum CPU miktarı |
| **CPU Limit** | Servisin kullanabileceği maksimum CPU miktarı |
| **Memory Request** | Servisin garanti ettiği minimum bellek miktarı |
| **Memory Limit** | Servisin kullanabileceği maksimum bellek miktarı |

> **İpucu:** Request değerlerini uygulamanızın normal yük altındaki tüketimine, limit değerlerini ise tepe yükü kaldıracak şekilde ayarlayın.

---

## Port Yapılandırması

Servisinizin dinlediği portları yapılandırın. Çoğu web uygulaması için varsayılan port yeterlidir, ancak birden fazla port dinleyen servisler için ek portlar tanımlayabilirsiniz.

---

## Storage (Kalıcı Depolama)

**Storage** sekmesi, servisinize Longhorn destekli kalıcı diskler bağlamanızı sağlar. Veritabanı, dosya sunucusu, yükleme klasörü, cache veya log arşivi gibi pod restart sonrası korunması gereken veriler için bir **disk planı** seçerek PersistentVolumeClaim (PVC) oluşturabilirsiniz.

### Kısa Özet

- **5 disk planı:** Free (1 GB, ücretsiz), Small (10 GB), Medium (50 GB), Large (200 GB), XLarge (1 TB). Ücretli planlar `Retain` reclaim politikası ile production güvenliği sağlar.
- **25+ mount path preset:** PostgreSQL, MySQL, MongoDB, Redis, Valkey, RabbitMQ, NGINX, Apache, Prometheus, Grafana gibi image ailelerine özel önceden tanımlı güvenli mount path'leri.
- **Image-family highlighting:** Container image'ınıza uygun preset'ler "Image için önerilen" etiketi ile öne çıkar.
- **Güvenli path whitelist:** Serbest metin path girişi yok — platform sadece katalogdaki güvenli yolları kabul eder. `/etc`, `/proc`, `/sys` gibi hassas host dizinlerine volume bağlanamaz.
- **Otomatik yedekleme:** Günlük snapshot + haftalık S3 backup (platform yöneticisi tarafından yapılandırıldıysa).
- **İki aşamalı silme onayı:** Veri kaybı riskine karşı hem UI confirmation hem backend acknowledgment flag'i.
- **Otomatik deploy:** Her storage değişikliği (ekleme/güncelleme/silme) otomatik olarak bir deploy tetikler — manuel Deploy tıklamaya gerek yok.

### Temel Kullanım Akışı

1. Servis yönetim panelinden **Storage** sekmesini açın
2. **Depolama Ekle** butonuna tıklayın
3. Dialog'dan plan seçin (varsayılan: Free)
4. Image ailesine uygun mount path preset'i seçin (örn. `postgres:16` için PostgreSQL data)
5. Benzersiz bir volume adı girin (örn. `db-data`)
6. Opsiyonel olarak sub-path ve read-only bayrağını ayarlayın
7. **Kaydet** → **otomatik deploy** tetiklenir, PVC ~30 saniye içinde cluster'da hazır olur
8. Yeşil toast: _"Deploy otomatik tetiklendi"_ → ilerlemeyi **Pipeline Summary** sekmesinden canlı izleyebilirsiniz

### Detaylı Rehber

Storage yönetimi, plan seçim kriterleri, mount path kataloğu, backup/restore ve pipeline entegrasyonu hakkında kapsamlı bilgi için ayrı bir rehber hazırlanmıştır:

➡️ **[Kalıcı Depolama (Storage) Rehberi](storage-management.md)**

Rehberde ayrıca şunları bulabilirsiniz:

- Her plan için detaylı kullanım senaryoları
- Retain vs Delete reclaim politikası karşılaştırması
- Plan yükseltme/değiştirme kuralları (ne değiştirilebilir, ne değiştirilemez)
- Silme işleminin geri alınamazlığı ve veri kaybı uyarıları
- Çok volume'lu servisler ve sub-path kullanım senaryoları
- Güvenlik ve multi-tenant izolasyon detayları
- Sık sorulan sorular

---

## Sağlık Kontrolleri (Health Probes)

Kubernetes'in servisinizin sağlık durumunu kontrol etmesi için probe'lar tanımlayın.

### Liveness Probe

Servisin çalışıp çalışmadığını kontrol eder. Başarısız olursa pod yeniden başlatılır.

| Parametre | Açıklama |
|-----------|----------|
| **HTTP Path** | Sağlık kontrol endpoint'i (ör: `/health`) |
| **Port** | Kontrol edilen port |
| **Initial Delay** | İlk kontrol öncesi bekleme süresi (saniye) |
| **Period** | Kontrol aralığı (saniye) |
| **Failure Threshold** | Kaç başarısız kontrolden sonra unhealthy sayılacağı |

### Readiness Probe

Servisin trafik almaya hazır olup olmadığını kontrol eder. Başarısız olursa pod trafik havuzundan çıkarılır ama yeniden başlatılmaz.

> **İpucu:** Veritabanı bağlantısı, önbellek ısınması gibi ön koşulları readiness probe'da kontrol edin.

---

## Pipeline Özeti

Bu sekmede servisinize ait tüm CI/CD pipeline çalıştırmalarını görebilirsiniz.

- **Canlı pipeline:** Devam eden build'in adım adım ilerlemesini izleyin
- **Geçmiş:** Tamamlanmış pipeline'ların listesi, süre ve durum bilgisi
- **Görev detayı:** Her pipeline adımının loglarına erişin

Detaylı bilgi için **CI/CD Pipeline** rehberine bakın.

---

## Log Görüntüleme

**Logs** sekmesinden servisinizin uygulama loglarını gerçek zamanlı olarak görüntüleyin.

- Tarih aralığı filtresi ile belirli bir zaman dilimini sorgulayın
- Metin araması ile loglar içinde anahtar kelime arayın
- Pod bazlı filtreleme ile belirli bir pod'un loglarını izleyin

---

## Deployment Geçmişi ve Rollback

**History & Rollback** sekmesinde servisinizin tüm deployment geçmişini görebilirsiniz.

### Revizyon Listesi

Her revizyon için şu bilgiler gösterilir:
- Revizyon numarası
- Deploy tarihi ve saati
- İmaj etiketi
- Durum (Active, Stable, Revision)

### Rollback (Geri Alma)

Sorunlu bir deploy'dan sonra önceki bir sürüme dönmek için:

1. Geri dönmek istediğiniz revizyonu bulun
2. **Rollback** butonuna tıklayın
3. Onaylayın

Rollback işlemi yeni bir revizyon oluşturarak seçtiğiniz sürümü yeniden aktif eder.

> **Uyarı:** Rollback, uygulamanın kodunu eski sürüme döndürür ancak ortam değişkenleri ve konfigürasyon değişikliklerini geri almaz.
