# Hızlı Başlangıç Rehberi

DevOpsZon'a hoş geldiniz! Bu rehber, platformu ilk kez kullanacak olanlar için adım adım başlangıç sürecini anlatır.

---

## DevOpsZon Nedir?

DevOpsZon, uygulamalarınızı Kubernetes üzerinde kolayca yayınlamanızı, yönetmenizi ve izlemenizi sağlayan bir DevOps platformudur. Git reponuzu bağlayın, servisleri keşfedin ve tek tıkla canlıya alın — altyapı karmaşıklığıyla uğraşmadan.

---

## İlk Adımlar

### 1. Hesap Oluşturun

DevOpsZon Console'a erişmek için [console.devopszon.com](https://console.devopszon.com) adresinden hesabınızı oluşturun. Kaydolduktan sonra e-posta doğrulamanızı tamamlayarak panele giriş yapabilirsiniz.

### 2. Git Sağlayıcınızı Bağlayın

Projelerinizi yayınlayabilmek için öncelikle kaynak kodlarınızın bulunduğu Git sağlayıcısını bağlamanız gerekir.

- Sol menüden **Integrations** sayfasına gidin
- **Git Connection** bölümünden sağlayıcınızı seçin: **GitHub**, **GitLab**, **Bitbucket** veya **Azure DevOps**
- OAuth yetkilendirmesini tamamlayın
- Bağlantı durumunuz yeşil olarak görünecektir

### 3. Container Registry Bağlayın

Build edilen Docker imajlarınızın depolanacağı bir container registry tanımlayın.

- Aynı **Integrations** sayfasında **Registry Connection** bölümüne geçin
- Desteklenen registry'ler: **Docker Hub**, **GitHub Container Registry (GHCR)**, **GitLab Container Registry**, **Azure Container Registry (ACR)**, **Amazon ECR**, **Google GCR** veya **özel registry**
- Erişim bilgilerinizi girerek bağlantıyı kaydedin

### 4. Cluster Belirleyin

Uygulamalarınızın çalışacağı Kubernetes cluster'ını belirleyin. İki seçeneğiniz var:

- **SaaS Modu:** DevOpsZon'un yönetilen altyapısını kullanın — ek bir cluster kurulumu gerekmez
- **Hibrit Mod:** Kendi Kubernetes cluster'ınızı **kubeconfig** dosyası ile bağlayın veya DevOpsZon arayüzünden yeni bir cluster oluşturun

### 5. İlk Projenizi Oluşturun

- Üst başlıkta **"+"** butonuna veya **Yeni Proje** seçeneğine tıklayın
- Proje oluşturma sihirbazı açılacaktır:

| Adım | Açıklama |
|------|----------|
| **Temel Bilgiler** | Proje adı ve açıklaması |
| **Servis Bağlantısı** | Git sağlayıcınızı ve registry'nizi seçin |
| **Repo Seçimi** | Yayınlamak istediğiniz repository'leri işaretleyin |
| **Servis Keşfi** | Repo'lardaki Dockerfile'lar otomatik tespit edilir ve servisler önerilir |
| **Kaynak & Cluster** | Servislerin çalışacağı cluster ve kaynak paketini belirleyin |
| **Ortam Değişkenleri** | Gerekli environment variable'ları tanımlayın |
| **Özet & Onay** | Tüm ayarları gözden geçirin ve projeyi oluşturun |

### 6. İlk Deploy'unuzu Yapın

Proje oluşturulduktan sonra, proje listesinde servislerinizi göreceksiniz. Bir servisi deploy etmek için:

- Servis satırındaki **Deploy** butonuna tıklayın
- İmaj etiketi (tag) seçin veya yeni bir build tetikleyin
- Pipeline otomatik olarak başlayacak ve ilerlemeyi takip edebileceksiniz
- Build ve deploy tamamlandığında servisinize atanan hostname üzerinden erişim sağlayabilirsiniz

---

## Platform Yapısı

DevOpsZon Console, sol menüdeki ana bölümlerle organize edilmiştir:

| Menü | Amaç |
|------|------|
| **Projects** | Projelerinizi ve servislerinizi yönetin |
| **Services** | Seçili servisin detaylı yönetim paneli |
| **Cluster** | Kubernetes cluster'larınızı yönetin |
| **Addons** | Yönetilen servisler (PostgreSQL, RabbitMQ, Valkey) |
| **Integrations** | Git ve container registry bağlantıları |
| **Alerts** | Uyarı kuralları ve bildirimler |
| **Notifications** | Bildirim kanalları ve ayarları |
| **Domains** | Özel domain yönetimi |
| **Access Control** | Kullanıcı yetkileri ve erişim kontrolü |

---

## Sonraki Adımlar

- Projelerinizi detaylı yönetmek için **Proje Yönetimi** rehberine göz atın
- Deployment stratejilerini (Canary, Blue/Green) öğrenmek için **Deploy Stratejileri** bölümüne bakın
- Yönetilen veritabanı veya mesaj kuyruğu için **Yönetilen Servisler** rehberine gidin
- Uyarı ve bildirim kurmak için **Uyarı Yönetimi** bölümünü inceleyin
