# Yönetilen Servisler (Managed Addon'lar)

DevOpsZon, uygulamalarınızın ihtiyaç duyduğu veritabanı, mesaj kuyruğu ve önbellek servislerini tam yönetimli olarak sunar. Bu servisler otomatik olarak kurulur, yapılandırılır, yedeklenir ve izlenir.

---

## Desteklenen Servisler

| Servis | Tür | Kullanım Alanı |
|--------|-----|----------------|
| **PostgreSQL** | İlişkisel Veritabanı | Yapılandırılmış veri depolama, ACID uyumlu işlemler |
| **RabbitMQ** | Mesaj Kuyruğu | Asenkron mesajlaşma, olay odaklı mimari, görev kuyruğu |
| **Valkey** | In-Memory Veri Deposu | Önbellek, oturum yönetimi, gerçek zamanlı veri |

---

## PostgreSQL — Yönetilen Veritabanı

### Özellikler

| Özellik | Detay |
|---------|-------|
| **Versiyon** | PostgreSQL 16 ve 17 |
| **Topoloji** | Single Instance ve HA (Primary + Replica) |
| **Connection Pooling** | Dahili PgBouncer (transaction mode) |
| **Otomatik Yedekleme** | Günlük yedekleme, S3/B2 depolama |
| **TLS** | Otomatik sertifika ile şifreli bağlantı |
| **Erişim** | SNI routing ile benzersiz hostname (ör: `pg-a1b2c3d4.devopszon.app`) |
| **Performans Ayarı** | Plan bazında otomatik PostgreSQL tuning |

### Planlar

#### Single Instance

| Plan | vCPU | RAM | Disk | Max Connection |
|------|:----:|:---:|:----:|:--------------:|
| **postgres-2g** | 0,5 | 2 GB | 40 GB | 250 |
| **postgres-4g** | 1 | 4 GB | 80 GB | 500 |
| **postgres-8g** | 2 | 8 GB | 160 GB | 1.000 |
| **postgres-16g** | 4 | 16 GB | 320 GB | 2.000 |
| **postgres-32g** | 8 | 32 GB | 640 GB | 4.000 |

#### HA (Primary + Replica)

| Plan | vCPU (per instance) | RAM (per instance) | Disk (per instance) |
|------|:-------------------:|:------------------:|:-------------------:|
| **postgres-8g-ha** | 2 | 8 GB | 160 GB |
| **postgres-16g-ha** | 4 | 16 GB | 320 GB |
| **postgres-32g-ha** | 8 | 32 GB | 640 GB |

> HA planlarında otomatik failover bulunur. Primary sunucu devre dışı kaldığında Replica otomatik olarak Primary rolünü üstlenir.

### PostgreSQL Oluşturma

1. Sol menüden **Addons** sayfasına gidin
2. **PostgreSQL** sekmesini seçin
3. **Yeni Instance Oluştur** butonuna tıklayın
4. Plan ve topoloji seçin
5. Oluşturma süreci başlayacak ve adım adım ilerleme gösterilecektir

**Kurulum adımları (otomatik):**

| Adım | Açıklama |
|------|----------|
| Namespace oluşturma | İzole çalışma alanı |
| Veritabanı cluster kurulumu | CloudNativePG operatörü ile |
| PgBouncer dağıtımı | Connection pooling |
| TLS sertifika üretimi | Let's Encrypt ile |
| DNS kaydı oluşturma | SNI hostname |
| Sağlık kontrolü | Bağlantı doğrulama |
| Yedekleme yapılandırması | Otomatik günlük yedek |

### Bağlantı Bilgileri

Instance oluşturulduktan sonra bağlantı bilgilerinize addon detay panelinden erişebilirsiniz:

- **Hostname:** `pg-{id}.devopszon.app`
- **Port:** `30930`
- **Veritabanı adı:** Otomatik oluşturulur
- **Kullanıcı adı ve şifre:** Güvenli olarak saklanır ve panelden kopyalanabilir

**Bağlantı string örnekleri:**

```
postgresql://user:password@pg-a1b2c3d4.devopszon.app:30930/mydb?sslmode=require
```

### Plan Değişikliği

Mevcut PostgreSQL instance'ınızın planını artırabilir veya azaltabilirsiniz:

1. Addon detay panelinde **Plan Değiştir** butonuna tıklayın
2. Yeni planı seçin
3. Onaylayın

> **Not:** Plan değişikliği sırasında kısa süreli kesinti yaşanabilir. HA planlarda bu süre minimize edilir.

---

## RabbitMQ — Yönetilen Mesaj Kuyruğu

### Özellikler

| Özellik | Detay |
|---------|-------|
| **Topoloji** | Single Broker ve HA Quorum Cluster (3 node) |
| **Protokol** | AMQPS (5671) — TLS ile şifreli |
| **Management UI** | Web tabanlı yönetim arayüzü (port 15672) |
| **Otomatik Yedekleme** | Günlük definition ve message snapshot |
| **SNI Routing** | Benzersiz hostname (ör: `rmq-{id}.devopszon.app`) |
| **Guardrails** | MaxMemory, MaxConnections, MaxQueues, MaxChannels limitleri |

### RabbitMQ Oluşturma

1. **Addons** sayfasında **RabbitMQ** sekmesini seçin
2. **Yeni Instance Oluştur** butonuna tıklayın
3. Topoloji ve plan seçin
4. 8 adımlı kurulum süreci SignalR üzerinden gerçek zamanlı izlenebilir

### Bağlantı Bilgileri

- **AMQPS Hostname:** `rmq-{id}.devopszon.app:5671`
- **Management UI:** `https://rmq-{id}.devopszon.app:15672`
- **Kullanıcı adı ve şifre:** Panelden kopyalanabilir

### Plan Değişikliği

CPU, RAM ve disk kaynaklarını canlı olarak değiştirebilirsiniz. HA topolojilerde zero-downtime plan değişikliği desteklenir.

---

## Valkey — Yönetilen In-Memory Veri Deposu

### Özellikler

| Özellik | Detay |
|---------|-------|
| **Kullanım** | Önbellek, oturum yönetimi, gerçek zamanlı veri |
| **Yüksek performans** | In-memory veri erişimi |
| **TLS** | Şifreli bağlantı desteği |
| **SNI Routing** | Benzersiz hostname ile erişim |

### Valkey Oluşturma

RabbitMQ ve PostgreSQL ile aynı akışı takip eder:
1. **Addons** sayfasında **Valkey** sekmesini seçin
2. Plan seçin ve oluşturun
3. Bağlantı bilgilerinizi panelden alın

---

## Addon Yönetim Paneli

Her addon instance'ının detay panelinde şu bölümler bulunur:

| Bölüm | Açıklama |
|-------|----------|
| **Genel Bilgiler** | Durum, plan, topoloji, oluşturma tarihi |
| **Bağlantı Bilgileri** | Hostname, port, kullanıcı adı, şifre, connection string |
| **İşlemler** | Yeniden başlatma, plan değişikliği, silme |
| **Yedekleme** | Yedekleme geçmişi ve manuel yedek tetikleme |
| **Plan Değişikliği** | Kaynak artırma/azaltma |

---

## Güvenlik

Tüm yönetilen servisler için:

- **TLS/SSL:** Tüm bağlantılar şifrelidir
- **Ağ izolasyonu:** NetworkPolicy ile pod seviyesinde izolasyon
- **Güvenli saklama:** Erişim bilgileri Kubernetes Secret olarak saklanır
- **SNI Routing:** Her instance'a özel subdomain ile erişim

---

## İpuçları

- **Connection pooling:** PostgreSQL bağlantılarınızı PgBouncer üzerinden yönlendirin; doğrudan bağlantı kullanmaktan kaçının
- **HA planları:** Üretim ortamları için HA topolojisini tercih edin
- **Yedekleme kontrolü:** Otomatik yedeklemelerin düzenli oluşturulduğunu addon panelinden takip edin
- **Plan seçimi:** Başlangıçta küçük bir planla başlayıp ihtiyaç arttıkça ölçeklendirme yapabilirsiniz
