# CI/CD Pipeline Yönetimi

DevOpsZon, **Komuta Pipeline** üzerinde Kubernetes-native CI/CD pipeline altyapısı sunar. Kaynak kodunuz otomatik olarak build edilir, Docker imajına dönüştürülür ve cluster'ınıza deploy edilir.

---

## Pipeline Nasıl Çalışır?

Bir deploy tetiklediğinizde, arka planda şu adımlar sırasıyla gerçekleşir:

```
Git Repo ──→ Kaynak Kodu Çekme ──→ Docker Build ──→ Image Push ──→ Deploy
   (1)            (2)                  (3)            (4)         (5)
```

| Adım | Açıklama |
|------|----------|
| **1. Tetikleme** | Deploy butonuna tıklama veya otomatik tetikleme |
| **2. Kaynak Çekme** | Git repository'den belirlenen branch'in kodu çekilir |
| **3. Docker Build** | Dockerfile kullanılarak container imajı oluşturulur |
| **4. Image Push** | Oluşturulan imaj, bağlı container registry'ye gönderilir |
| **5. Deploy** | Yeni imaj, Komuta Rollout ile cluster'a deploy edilir |

---

## Pipeline Tetikleme

### Manuel Tetikleme

1. **Projects** sayfasından servis satırındaki **Deploy** butonuna tıklayın
2. Veya **Service Management** → **Pipeline Summary** sekmesinden tetikleyin

### Toplu Tetikleme

Birden fazla servisi aynı anda deploy etmek için:
1. **Projects** sayfasında servisleri işaretleyin
2. **Toplu Deploy** aksiyonunu kullanın

### Kuyruk Yönetimi

Birden fazla pipeline aynı anda tetiklendiğinde, DevOpsZon akıllı bir kuyruk sistemi kullanır:
- Eşzamanlı çalışabilecek pipeline sayısı cluster kaynaklarına göre belirlenir
- Kuyrukta bekleyen pipeline'lar sırayla işlenir
- Her pipeline'ın kuyruk durumu proje listesinde gösterilir

---

## Pipeline İzleme

### Proje Listesinde

Proje listesindeki her servis satırında pipeline durumu rozetle gösterilir:

| Rozet | Anlamı |
|-------|--------|
| **Kuyrukta** | Pipeline kuyruğa alındı, sıra bekliyor |
| **Bekliyor** | Pipeline kaynakları hazırlanıyor |
| **Build** | Docker imajı oluşturuluyor |
| **Başarılı** | Pipeline başarıyla tamamlandı |
| **Hata** | Pipeline bir hata ile sonlandı |

### Pipeline Summary Sekmesi

**Service Management** → **Pipeline Summary** sekmesinde daha detaylı bilgiler:

- **Canlı pipeline:** Devam eden build'in her adımı gerçek zamanlı gösterilir
- **Görev listesi:** Her pipeline adımının adı, süresi ve durumu
- **Geçmiş:** Önceki tüm pipeline çalıştırmaları

### Pipeline Detayı

Bir pipeline çalıştırmasına tıkladığınızda detay görünümü açılır:

| Bilgi | Açıklama |
|-------|----------|
| **Görev tablosu** | Her adımın adı, başlangıç/bitiş zamanı, süresi |
| **Durum** | Her görevin başarı/hata durumu |
| **Loglar** | Her görev adımının detaylı logları |

---

## Pipeline Görevleri

Tipik bir DevOpsZon pipeline'ı şu görevlerden oluşur:

| Görev | Açıklama |
|-------|----------|
| **fetch-credentials** | Git ve registry erişim bilgilerini hazırlar |
| **git-clone** | Kaynak kodu repository'den çeker |
| **build-and-push** | Dockerfile ile imaj oluşturur ve registry'ye gönderir |
| **update-manifest** | Deployment manifest dosyasını yeni imaj etiketi ile günceller |
| **deploy** | Komuta Rollout üzerinden deploy'u başlatır |

---

## Pipeline Logları

Her pipeline görevinin loglarına erişmek için:

1. **Pipeline Summary** sekmesine gidin
2. İlgili pipeline çalıştırmasını tıklayın
3. Görev listesinde log görmek istediğiniz adıma tıklayın
4. Detaylı log çıktısı görüntülenir

### Log İçinde Arama

Pipeline loglarında hata ayıklama yaparken:
- Build hatalarını `error` veya `failed` anahtar kelimeleriyle arayın
- Dockerfile'daki sorunları `COPY`, `RUN` komutlarının çıktısından tespit edin
- Registry push hatalarını `push` loglarından kontrol edin

---

## Hata Durumları ve Çözümleri

| Hata | Olası Neden | Çözüm |
|------|------------|-------|
| **Build hatası** | Dockerfile'da sözdizimi hatası veya eksik bağımlılık | Dockerfile'ı kontrol edin; loglardan hata detayını inceleyin |
| **Push hatası** | Registry erişim bilgileri geçersiz veya süresi dolmuş | Integrations sayfasından registry bağlantısını güncelleyin |
| **Git clone hatası** | Repository erişim izni yok veya branch bulunamadı | Git bağlantınızı ve branch adını kontrol edin |
| **Kuyruk timeout** | Cluster kaynakları yetersiz | Daha az eşzamanlı pipeline çalıştırın veya kaynak artırın |

---

## Dockerfile Otomatik Düzeltme

DevOpsZon, build hatası oluştuğunda Dockerfile'ınızı otomatik analiz edebilir ve düzeltme önerisi sunabilir.

### Nasıl Çalışır?

1. Pipeline build aşamasında hata oluşur
2. Hata logları otomatik olarak analiz edilir
3. Düzeltme önerisi hazırlanır
4. Git repository'nize bir **fix branch** ve **Pull Request** oluşturulur
5. PR'ı inceleyip birleştirdikten sonra yeniden deploy edebilirsiniz

> **Not:** Otomatik düzeltme özelliği isteğe bağlıdır ve servis ayarlarından etkinleştirilir.

---

## İpuçları

- **Pipeline süresini kısaltmak için:** Docker multi-stage build kullanın ve gereksiz katmanları (layer) azaltın
- **Cache:** Bağımlılık kurulum katmanlarını ayrı tutarak Docker cache'inden yararlanın
- **Health check:** Deploy sonrası pod'ların readiness probe'unu geçmesini bekleyin
- **Paralel build:** Bağımsız servisler için birden fazla pipeline'ı paralel çalıştırabilirsiniz
