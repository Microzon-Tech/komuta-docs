# Faturalandırma ve Kaynak Planları

DevOpsZon'un faturalandırma sistemi, kullandığınız kaynaklara göre şeffaf bir fiyatlandırma sunar. Bu rehber, kaynak planları, fatura yönetimi ve ödeme süreçlerini açıklar.

---

## Faturalandırma Modeli

### SaaS Modu

SaaS modunda faturalandırma, seçtiğiniz kaynak planlarına göre **aylık bazda** yapılır:

- Her servis için bir kaynak planı seçilir
- Yönetilen servisler (PostgreSQL, RabbitMQ, Valkey) ayrı faturalandırılır
- Fatura dönemi sonunda otomatik fatura oluşturulur

### Hibrit Mod

Hibrit modda, bağlanan her Kubernetes cluster için **sabit bir aylık ücret** uygulanır:

- Kaç cluster bağladıysanız, her biri için ayrı faturalandırılırsınız
- Cluster üzerinde ne kadar kaynak kullanacağınız sizin tercihinizdir
- Yönetilen servisler (addon'lar) ek olarak faturalandırılır

---

## Kaynak Planları

Servisleriniz için kaynak planı seçerken, uygulamanızın ihtiyaçlarına göre CPU, bellek ve disk kapasitesini belirlersiniz.

### Servis Kaynak Paketleri

Proje oluşturma sırasında veya sonrasında her servis için bir kaynak paketi seçilir. Bu paket, Kubernetes'teki resource request ve limit değerlerini belirler.

### Addon Planları

Yönetilen servisler için özel planlar mevcuttur:

**PostgreSQL Planları:**

| Plan | vCPU | RAM | Disk | Bağlantı Limiti |
|------|:----:|:---:|:----:|:---------------:|
| postgres-2g | 0,5 | 2 GB | 40 GB | 250 |
| postgres-4g | 1 | 4 GB | 80 GB | 500 |
| postgres-8g | 2 | 8 GB | 160 GB | 1.000 |
| postgres-16g | 4 | 16 GB | 320 GB | 2.000 |
| postgres-32g | 8 | 32 GB | 640 GB | 4.000 |

HA planları ek maliyet ile Primary + Replica topolojisi sunar.

---

## Cüzdan ve Bakiye

DevOpsZon hesabınızda bir **cüzdan (wallet)** bulunur:

- Hesabınıza bakiye yükleyebilirsiniz
- Faturalar otomatik olarak cüzdandan düşülür
- Düşük bakiye durumunda bildirim alırsınız

### Bakiye Görüntüleme

**Account** sayfasından cüzdan bakiyenizi ve işlem geçmişinizi görebilirsiniz:

| Bilgi | Açıklama |
|-------|----------|
| **Mevcut Bakiye** | Cüzdanınızdaki mevcut tutar |
| **Aylık Tahmini Maliyet** | Mevcut kaynak kullanımınıza göre tahmini aylık fatura |
| **Son İşlemler** | Bakiye yükleme ve fatura düşüm geçmişi |

---

## Fatura Yönetimi

### Fatura Görüntüleme

Faturalarınıza **Account** → **Faturalar** bölümünden erişebilirsiniz:

| Bilgi | Açıklama |
|-------|----------|
| **Fatura Dönemi** | Faturanın kapsadığı tarih aralığı |
| **Toplam Tutar** | Fatura toplamı |
| **Durum** | Ödendi, Beklemede, Gecikmiş |
| **Kalemler** | Faturayı oluşturan hizmetlerin detayı |

### Fatura Kalemleri

Her fatura, şu kalem türlerini içerebilir:

| Kalem | Açıklama |
|-------|----------|
| **Servis kaynakları** | Her servis için seçilen kaynak paketinin maliyeti |
| **PostgreSQL** | Yönetilen veritabanı planı |
| **RabbitMQ** | Yönetilen mesaj kuyruğu planı |
| **Valkey** | Yönetilen in-memory depo planı |
| **Cluster (Hibrit)** | Bağlı cluster başına sabit ücret |

---

## Plan Değişikliği

Mevcut bir servisin veya addon'un planını istediğiniz zaman değiştirebilirsiniz:

### Servis Planı Değiştirme

1. Servis yönetimi panelinde **Resource Plan** sekmesine gidin (SaaS modu)
2. Mevcut planınız ve kullanılabilir planlar gösterilir
3. Yeni planı seçin ve onaylayın
4. Değişiklik bir sonraki faturalandırma döneminden itibaren geçerli olur

### Addon Planı Değiştirme

1. Addon detay panelinde **Plan Değiştir** butonuna tıklayın
2. Upgrade (üst plan) veya downgrade (alt plan) seçin
3. Etkileri gözden geçirin ve onaylayın

> **Not:** Downgrade işlemlerinde, mevcut veri miktarınız yeni planın disk kapasitesini aşıyorsa işlem yapılamaz.

---

## Ödeme Politikası

### Zamanında Ödeme

Faturalar, dönem sonunda otomatik olarak oluşturulur ve cüzdan bakiyesinden düşülür.

### Gecikmiş Ödeme

Fatura zamanında ödenmezse:

| Süre | İşlem |
|------|-------|
| **Fatura tarihi** | Bildirim gönderilir |
| **7 gün sonra** | Servis replica sayısı sıfıra düşürülür (servis durdurulur) |
| **14 gün sonra** | Kaynaklar tamamen silinir |

> **Uyarı:** Kaynaklar silindikten sonra geri getirilemez. Ödeme planınızı takip etmeniz önemlidir.

---

## İpuçları

- **Bakiye takibi:** Düşük bakiye bildirimlerini aktif tutun
- **Doğru plan seçimi:** Başlangıçta küçük planla başlayıp gerektiğinde büyütün
- **HA planları:** Üretim ortamları için HA planlarını tercih edin; ek maliyeti downtime riskiyle karşılaştırın
- **Fatura geçmişi:** Geçmiş faturaları düzenli olarak inceleyin ve beklenmedik maliyet artışlarını takip edin
