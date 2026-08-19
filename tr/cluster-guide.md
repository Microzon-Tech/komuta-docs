# Cluster Yönetimi

Cluster yönetimi, Kubernetes cluster'larınızı ekleme, oluşturma ve yapılandırma işlemlerini tek bir panelden gerçekleştirmenizi sağlar.

---

## Cluster Nedir?

Kubernetes cluster, uygulamalarınızın çalıştığı sunucu altyapısıdır. DevOpsZon'da her proje ve servis bir cluster ile ilişkilendirilir. Çalışma modelinize göre farklı cluster seçenekleri mevcuttur.

---

## Cluster Ekleme Yöntemleri

### 1. Mevcut Cluster'ı Bağlama (Hibrit Mod)

Kendi Kubernetes cluster'ınız varsa, kubeconfig dosyası ile DevOpsZon'a bağlayabilirsiniz.

**Adımlar:**

1. Sol menüden **Cluster** sayfasına gidin
2. **Cluster Add** sekmesinde **Yeni Cluster Ekle** butonuna tıklayın
3. Cluster'ınıza bir ad verin
4. **kubeconfig** içeriğini yapıştırın
5. DevOpsZon, cluster'a bağlantıyı test edecektir
6. Bağlantı başarılıysa cluster listenize eklenecektir

> **Önemli:** Kubeconfig'inizin DevOpsZon'un çalışması için yeterli yetkilere sahip olduğundan emin olun. Cluster-admin veya eşdeğeri yetkiler gereklidir.

**Bağlantı sonrası otomatik yapılandırma:**

DevOpsZon, bağlanan cluster'a aşağıdaki bileşenleri otomatik olarak kurar:
- Komuta Pipeline (CI/CD pipeline)
- Komuta Rollout (deploy stratejileri)
- Komuta Metrics ve Komuta Alerts (izleme ve uyarılar)
- Komuta Logs (log toplama)
- Gerekli namespace ve erişim yetkileri

### 2. Yeni Cluster Oluşturma

DevOpsZon arayüzünden doğrudan yeni bir Kubernetes cluster oluşturabilirsiniz.

**Seçenekler:**

| Yöntem | Açıklama |
|--------|----------|
| **Managed Cluster** | Bulut sağlayıcı üzerinde otomatik kurulum |
| **Bare Metal** | Fiziksel sunuculara Kubernetes kurulumu |
| **BYOC (Bring Your Own Cloud)** | Kendi bulut hesabınızda kaynak oluşturma |

**Managed Cluster oluşturma sihirbazı:**

1. **Bulut sağlayıcı seçimi:** Mevcut durumda Hetzner Cloud desteklenmektedir
2. **Bölge seçimi:** Cluster'ın konumlandırılacağı veri merkezi bölgesi
3. **Node yapılandırması:** Sunucu tipi, sayısı ve kaynakları
4. **Kubernetes versiyonu:** Kurulacak K8s sürümü
5. **Ek ayarlar:** Ağ, güvenlik ve depolama yapılandırması

### 3. SaaS Modu

SaaS modunda cluster yönetimi DevOpsZon tarafından yapılır. Kullanıcı olarak cluster ekleme veya yönetme ihtiyacınız yoktur — projeleriniz otomatik olarak DevOpsZon altyapısında çalışır.

---

## Cluster Listesi

Cluster sayfasında bağlı tüm cluster'larınızın listesini görebilirsiniz:

| Bilgi | Açıklama |
|-------|----------|
| **Cluster Adı** | Cluster'a verdiğiniz ad |
| **Durum** | Bağlantı durumu (Connected, Disconnected, Error) |
| **Kubernetes Sürümü** | Cluster'daki K8s versiyonu |
| **Node Sayısı** | Worker node sayısı |
| **Topoloji** | Cluster mimarisi (single-master, multi-master vb.) |
| **Bölge** | Cluster'ın bulunduğu veri merkezi bölgesi |

---

## Cluster Detayı

Cluster listesinde bir satıra tıkladığınızda detay sayfası açılır:

### Özet Kartlar

- **Cluster tipi:** Managed, Imported, Bare Metal
- **Kubernetes sürümü**
- **Topoloji:** Master ve worker node dağılımı
- **Bölge:** Veri merkezi konumu

### Node Yönetimi

Cluster detayında node'ları (sunucuları) görebilir ve yönetebilirsiniz:

| İşlem | Açıklama |
|-------|----------|
| **Node ekleme** | Cluster'a yeni worker node ekleyin (Managed cluster'lar için) |
| **Node kaldırma** | Mevcut bir worker node'u güvenli şekilde çıkarın |
| **Node bilgileri** | CPU, bellek, disk kullanımı ve pod kapasitesi |

### Kubeconfig

Cluster'ın kubeconfig bilgisini görüntüleyebilir ve panoya kopyalayabilirsiniz. Bu, `kubectl` ile doğrudan erişim gerektiğinde kullanışlıdır.

---

## Addon'lar

Her cluster için kurulu olan ve kurulabilecek addon'ları görebilirsiniz.

### Kurulu Addon'lar

DevOpsZon'un çalışması için gerekli addon'lar cluster bağlandığında otomatik kurulur:

| Addon | Rol |
|-------|-----|
| **Komuta Pipeline** | CI/CD pipeline çalıştırma |
| **Komuta Rollout** | Gelişmiş deploy stratejileri |
| **Komuta Metrics** | Metrik toplama ve izleme |
| **Komuta Alerts** | Uyarı yönetimi ve yönlendirme |
| **Komuta Logs** | Log toplama ve sorgulama |
| **Komuta TLS** | Otomatik TLS sertifika yönetimi |
| **Komuta Gateway** | HTTP/HTTPS trafik yönlendirme |
| **Komuta Runtime Security** | KubeArmor ile container ve node runtime koruması — [detay](runtime-security-guide.md) |

### Addon Sağlık Durumu

Addon'ların çalışma durumunu izleyebilirsiniz:

- **Healthy (Sağlıklı):** Addon düzgün çalışıyor
- **Degraded (Düşük):** Addon çalışıyor ama bazı sorunlar var
- **Failed (Başarısız):** Addon çalışmıyor, müdahale gerekli

> **İpucu:** Addon sağlık durumu sorunlu görünüyorsa, adım yeniden çalıştırma (rerun) özelliğini kullanarak kurulum adımlarını tekrar deneyebilirsiniz.

---

## Cluster Ölçekleme

Managed cluster'lar için:

- **Scale Up:** Yeni worker node ekleyerek cluster kapasitesini artırın
- **Scale Down:** Kullanılmayan worker node'ları kaldırarak maliyeti azaltın

> **Uyarı:** Node kaldırma işlemi sırasında, üzerindeki pod'lar diğer node'lara taşınır. Bu süreçte kısa süreli performans düşüşü yaşanabilir.

---

## İpuçları

- **Çoklu cluster:** Farklı ortamlar (development, staging, production) için ayrı cluster'lar kullanın
- **Kapasite planlaması:** Servis sayısı ve kaynak ihtiyaçlarını göz önünde bulundurarak yeterli node kapasitesi sağlayın
- **Bölge seçimi:** Kullanıcılarınıza en yakın veri merkezi bölgesini seçerek gecikmeyi minimize edin
- **Yedek kubeconfig:** Cluster'ınızın kubeconfig'ini güvenli bir yerde yedekleyin
