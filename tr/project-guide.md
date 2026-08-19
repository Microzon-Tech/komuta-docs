# Proje Yönetimi

Projeler, DevOpsZon'daki en üst seviye organizasyon birimidir. Her proje altında bir veya birden fazla Git repository'si ve bu repository'lerden keşfedilen servisler yer alır.

---

## Proje Kavramı

DevOpsZon'da bir proje şu hiyerarşiyi temsil eder:

```
Proje
├── Repository 1
│   ├── Servis A (Dockerfile ile tanımlı)
│   └── Servis B (Dockerfile ile tanımlı)
├── Repository 2
│   └── Servis C
└── Repository 3
    └── Servis D
```

Her proje bir **cluster** ve **registry** ile ilişkilendirilir. Projeye bağlı tüm servisler, belirlenen cluster'da çalışır ve build edilen imajlar seçilen registry'ye gönderilir.

---

## Proje Listesi

Console'a giriş yaptığınızda karşınıza **Project Management** sayfası gelir. Bu sayfada:

- **Üst başlıkta proje seçici:** Mevcut projeleriniz arasında geçiş yapabilirsiniz
- **Favori projeler:** Sık kullandığınız projeleri yıldızlayarak hızlı erişim sağlayın
- **Servis tablosu:** Seçili projeye ait tüm servislerin listesi

### Servis Tablosunda Görünen Bilgiler

| Sütun | Açıklama |
|-------|----------|
| **Servis Adı** | Keşfedilen servisin adı (Dockerfile'dan türetilir) |
| **Branch** | Aktif Git branch'i |
| **Namespace** | Kubernetes namespace'i |
| **Cluster** | Servisin çalıştığı cluster |
| **Durum** | Servisin mevcut durumu (Running, Degraded, Progressing vb.) |
| **Pipeline** | Son pipeline çalıştırmasının durumu |
| **İşlemler** | Deploy, detay, entegrasyon, silme gibi aksiyonlar |

---

## Yeni Proje Oluşturma

Yeni bir proje oluşturmak için üst menüdeki **"+"** butonuna tıklayın veya overflow menüsünden **Yeni Proje** seçin. 7 adımlı sihirbaz sizi yönlendirecektir:

### Adım 1: Temel Bilgiler

Projenize bir ad ve isteğe bağlı açıklama verin. Proje adı, platform genelinde benzersiz olmalıdır.

### Adım 2: Servis Bağlantısı

Daha önce **Integrations** sayfasında tanımladığınız Git sağlayıcısını ve container registry'yi seçin. Bu bağlantılar, repo'larınıza erişim ve build imajlarının depolanması için kullanılacaktır.

> **İpucu:** Henüz bir Git veya registry bağlantınız yoksa, bu adımdan entegrasyon sayfasına yönlendirilirsiniz.

### Adım 3: Repo Seçimi

Bağlı Git sağlayıcınızdaki repository'ler listelenir. Projeye eklemek istediğiniz repo'ları işaretleyin. Bir projeye birden fazla repo bağlayabilirsiniz.

### Adım 4: Servis Keşfi

DevOpsZon, seçtiğiniz repo'ları tarayarak **Dockerfile** içeren dizinleri otomatik tespit eder. Her Dockerfile, deploy edilebilir bir servis olarak önerilir.

- Keşfedilen servisler listesini inceleyin
- İstemediğiniz servislerin işaretini kaldırın
- Her servis için branch seçimi yapın

> **Not:** Servis keşfi yalnızca Dockerfile tabanlıdır. Dockerfile içermeyen uygulamalar otomatik keşfedilemez.

### Adım 5: Kaynak ve Cluster

- Servislerin çalışacağı **Kubernetes cluster'ını** seçin
- **Kaynak paketi** belirleyin (CPU, bellek, disk)
- SaaS modunda kaynak paketleri fiyatlandırmaya doğrudan etki eder

### Adım 6: Ortam Değişkenleri

Servislerin çalışması için gerekli **environment variable'ları** tanımlayın. Her servis için ayrı ayrı veya toplu olarak değişken ekleyebilirsiniz.

| Alan | Açıklama |
|------|----------|
| **Key** | Değişken adı (ör: `DATABASE_URL`) |
| **Value** | Değişken değeri |
| **Gizli (Secret)** | İşaretlendiğinde değer Kubernetes Secret olarak saklanır |

### Adım 7: Özet ve Onay

Tüm ayarlarınızı gözden geçirin. Onayladığınızda DevOpsZon şunları otomatik olarak yapar:

1. Projeyi oluşturur
2. Repo'ları bağlar
3. Servisleri tanımlar
4. Gerekli Kubernetes kaynaklarını hazırlar
5. İlk pipeline'ı tetikler (isteğe bağlı)

---

## Projeye Repo Ekleme

Mevcut bir projeye yeni repository eklemek için:

1. Proje sayfasındaki overflow menüsünden **Repo Ekle** seçin
2. Git sağlayıcınızdaki repo listesinden yeni repo'yu seçin
3. Servis keşfini onaylayın
4. Yeni servisler proje listesine eklenecektir

---

## Yeni Servis Keşfi

Repo'larınıza yeni bir Dockerfile eklediğinizde, bu servisi keşfetmek için:

1. Overflow menüsünden **Yeni Servis Keşfet** seçin
2. DevOpsZon repo'larınızı yeniden tarar
3. Yeni bulunan servisler önizleme olarak gösterilir
4. Onayladığınız servisler projeye eklenir

---

## Proje İşlemleri

### Servis Deploy Etme

Proje listesindeki herhangi bir servisin yanındaki **Deploy** butonuna tıklayarak yeni bir deploy başlatabilirsiniz. Deploy dialog'unda:

- Mevcut imaj etiketlerinden birini seçin veya
- Yeni bir build tetikleyin

### Proje Silme

Bir projeyi silmek, o projeye bağlı **tüm servisleri, pipeline geçmişlerini ve Kubernetes kaynaklarını** kalıcı olarak siler. Bu işlem geri alınamaz.

> **Uyarı:** Proje silme işlemi, ilişkili tüm Kubernetes kaynaklarını (deployment, service, ingress vb.) da temizler.

### Cluster'a Bağlama

Overflow menüsündeki **Cluster'a Bağla** seçeneği ile projenizi farklı bir cluster'a taşıyabilir veya ek cluster bağlantısı kurabilirsiniz.

---

## İpuçları

- **Favori projeler:** Sık kullandığınız projeleri yıldızlayarak üst başlıktaki seçiciden hızlıca erişin
- **Toplu deploy:** Birden fazla servisi aynı anda deploy edebilirsiniz
- **Proje başına izleme:** Her projenin sağlık durumunu proje listesindeki durum göstergelerinden takip edin
