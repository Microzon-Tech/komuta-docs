# Entegrasyonlar

DevOpsZon, kaynak kodlarınıza ve container imajlarınıza erişmek için çeşitli Git sağlayıcıları ve container registry'leri ile entegrasyon sağlar. Bu entegrasyonlar, proje oluşturma ve deploy süreçlerinin temelini oluşturur.

---

## Git Entegrasyonları

Git entegrasyonları, kaynak kodlarınızın bulunduğu repository'lere erişim sağlar. OAuth tabanlı yetkilendirme ile güvenli bağlantı kurulur.

### Desteklenen Git Sağlayıcıları

| Sağlayıcı | Bağlantı Yöntemi | Özellikler |
|-----------|------------------|-----------|
| **GitHub** | OAuth App | Repo listesi, branch görüntüleme, Dockerfile keşfi |
| **GitLab** | OAuth App | Repo listesi, branch görüntüleme, Dockerfile keşfi |
| **Bitbucket** | OAuth App | Repo listesi, branch görüntüleme, Dockerfile keşfi |
| **Azure DevOps** | OAuth App | Repo listesi, branch görüntüleme, Dockerfile keşfi |

### Git Bağlantısı Kurma

1. Sol menüden **Integrations** sayfasına gidin
2. **Git Connections** bölümünde bağlamak istediğiniz sağlayıcının **Bağlan** butonuna tıklayın
3. Sağlayıcının OAuth yetkilendirme sayfasına yönlendirilirsiniz
4. Gerekli izinleri onaylayın
5. DevOpsZon'a geri yönlendirilirsiniz ve bağlantı durumu **Connected** olarak görünecektir

### Bağlantı Sonrası

Bağlantı kurulduktan sonra:

- **Repo listesi:** Sağlayıcınızdaki tüm erişilebilir repository'leri görebilirsiniz
- **Branch görüntüleme:** Her repo'nun branch'lerini listeleyebilirsiniz
- **Dosya keşfi:** Repo içeriğini (Dockerfile, proje yapısı) inceleyebilirsiniz
- **Servis keşfi:** Dockerfile'ları otomatik tespit ederek deploy edilebilir servisleri keşfedebilirsiniz

### Bağlantı Durumları

| Durum | Anlamı |
|-------|--------|
| **Connected** | Bağlantı aktif, erişim sağlanıyor |
| **Expired** | Token süresi dolmuş, yeniden yetkilendirme gerekli |
| **Error** | Bağlantı hatası, kontrol gerekli |

> **İpucu:** Token süresi dolan bağlantıları **Yeniden Bağlan** butonuyla kolayca yenileyebilirsiniz.

---

## Container Registry Entegrasyonları

Container registry'ler, build edilen Docker imajlarınızın depolandığı ve deploy sırasında çekildiği kaynaklardır.

### Desteklenen Registry'ler

| Registry | Tür | Açıklama |
|----------|-----|----------|
| **Docker Hub** | Genel | Docker'ın resmi registry'si |
| **GitHub Container Registry (GHCR)** | Genel | GitHub ile entegre registry |
| **GitLab Container Registry** | Genel | GitLab ile entegre registry |
| **Azure Container Registry (ACR)** | Bulut | Azure'un yönetilen registry'si |
| **Amazon ECR** | Bulut | AWS'nin yönetilen registry'si |
| **Google Container Registry (GCR)** | Bulut | Google Cloud'un registry'si |
| **Özel (Custom)** | Özel | Kendi sunucunuzdaki registry |

### Registry Bağlantısı Kurma

1. **Integrations** sayfasında **Registry Connections** bölümüne gidin
2. **Yeni Registry Ekle** butonuna tıklayın
3. Registry tipini seçin
4. Erişim bilgilerini girin:

| Alan | Açıklama |
|------|----------|
| **Registry URL** | Registry adresi (ör: `ghcr.io`, `registry.example.com`) |
| **Kullanıcı Adı** | Registry kullanıcı adı veya service account |
| **Şifre / Token** | Erişim şifresi veya personal access token |

5. **Test** butonuyla bağlantıyı doğrulayın
6. **Kaydet** butonuyla bağlantıyı kaydedin

### Registry Kullanım Alanları

- **Build sırasında (Push):** Pipeline, oluşturduğu Docker imajını seçili registry'ye gönderir
- **Deploy sırasında (Pull):** Kubernetes, deploy edilen imajı registry'den çeker

Her projenin servis ayarlarında hangi registry'nin kullanılacağı belirtilir.

---

## Proje Oluşturma Akışındaki Rolü

Entegrasyonlar, proje oluşturma sihirbazının temelini oluşturur:

```
Git Bağlantısı → Repo Seçimi → Servis Keşfi → Registry Seçimi → Pipeline → Deploy
```

1. **Servis bağlantısı adımı:** Daha önce kurulan Git bağlantısından biri seçilir
2. **Repo seçimi adımı:** Seçilen Git bağlantısı üzerinden repo'lar listelenir
3. **Servis keşfi adımı:** Seçilen repo'lardaki Dockerfile'lar keşfedilir
4. **Pipeline yapılandırması:** Seçilen registry'ye imaj gönderilecek şekilde pipeline ayarlanır

---

## Bağlantı Yönetimi

### Bağlantı Güncelleme

Mevcut bir bağlantının erişim bilgilerini güncellemek için:
1. Bağlantı listesinde ilgili kaydın yanındaki **Düzenle** butonuna tıklayın
2. Yeni bilgileri girin
3. **Kaydet** butonuna tıklayın

### Bağlantı Silme

Kullanılmayan bağlantıları silebilirsiniz. Ancak dikkat edin:

> **Uyarı:** Aktif projeler tarafından kullanılan bir bağlantıyı silerseniz, o projelerin pipeline'ları başarısız olur. Silmeden önce bağlantıyı kullanan projeleri kontrol edin.

---

## İpuçları

- **Ayrı hesaplar:** Üretim ve geliştirme ortamları için farklı registry bağlantıları kullanın
- **Token yenileme:** Personal Access Token'larınızın süresini takip edin ve süresi dolmadan güncelleyin
- **Minimal yetki:** Git bağlantılarında yalnızca gerekli repo'lara erişim izni verin
- **Özel registry:** Gizlilik gereksinimleri olan projeler için özel registry kullanmayı tercih edin
