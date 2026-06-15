# Ingress ve Domain Yönetimi

Bu rehber, servislerinizin dış dünyadan nasıl erişileceğini, hostname yapılandırmasını ve özel domain bağlama sürecini açıklar.

---

## Otomatik Hostname

DevOpsZon'da her servis oluşturulduğunda otomatik olarak benzersiz bir hostname atanır:

```
{servis-adı}-{benzersiz-id}.devopszon.com
```

**Örnek:**
```
my-api-a1b2c3d4.devopszon.com
```

Bu hostname, servisinize HTTPS üzerinden doğrudan erişim sağlar. TLS sertifikası otomatik olarak Let's Encrypt ile oluşturulur ve yenilenir.

### Blue/Green Preview Hostname

Blue/Green deployment stratejisi kullanan servislerde ek olarak bir preview hostname oluşturulur:

```
{servis-adı}-{benzersiz-id}-preview.devopszon.com
```

Bu adres, yeni sürümü canlıya almadan önce test etmeniz için kullanılır.

---

## Ingress Yapılandırması

Servislerinizin trafik yönlendirme kurallarını **Service Management** → **Ingress Management** sekmesinden yapılandırabilirsiniz.

### Temel Ayarlar

| Ayar | Açıklama |
|------|----------|
| **Host** | Servisinize yönlendirilecek hostname |
| **Path** | URL yolu bazlı yönlendirme (ör: `/api`, `/web`) |
| **Backend Port** | Servisinizin dinlediği port numarası |
| **TLS** | SSL/TLS sertifika durumu |

### Host Kuralları

Bir servise birden fazla hostname bağlayabilirsiniz:

- Otomatik oluşturulan `*.devopszon.com` hostname'i
- Özel domain (ör: `api.mycompany.com`)
- Preview hostname (Blue/Green stratejisinde)

### Path Kuralları

Aynı hostname altında farklı path'leri farklı servislere yönlendirebilirsiniz:

```
myapp.devopszon.com
├── /api    → backend-service
├── /admin  → admin-service
└── /       → frontend-service
```

---

## Özel Domain Bağlama

Kendi alan adınızı DevOpsZon'daki bir servise bağlayarak profesyonel bir erişim adresi oluşturabilirsiniz.

### Adımlar

**1. Domain Ekleme**

- Sol menüden **Domains** sayfasına gidin
- **Yeni Domain Ekle** butonuna tıklayın
- Alan adınızı girin (ör: `api.mycompany.com`)
- Bağlanacak servisi seçin

**2. DNS Doğrulama**

DevOpsZon, domain sahipliğinizi doğrulamak için bir DNS kaydı oluşturmanızı isteyecektir:

| Kayıt Tipi | Host | Değer |
|------------|------|-------|
| **CNAME** | `api.mycompany.com` | DevOpsZon tarafından verilen hedef adres |

veya

| Kayıt Tipi | Host | Değer |
|------------|------|-------|
| **TXT** | `_devopszon.api.mycompany.com` | Doğrulama kodu |

> DNS değişikliklerinin yayılması birkaç dakika ile 48 saat arasında sürebilir.

**3. Doğrulama Kontrolü**

DNS kaydını oluşturduktan sonra **Doğrula** butonuna tıklayın. DevOpsZon, DNS kaydınızı kontrol edecektir.

- **Başarılı:** Domain aktif edilir ve servisinize bağlanır
- **Başarısız:** DNS kaydınızı kontrol edin ve tekrar deneyin

**4. TLS Sertifikası**

Domain doğrulandıktan sonra TLS sertifikası otomatik olarak oluşturulur. Servisinize `https://api.mycompany.com` üzerinden güvenli erişim sağlanır.

---

## Cloudflare Entegrasyonu

DevOpsZon, SaaS modunda Cloudflare ile entegre çalışır:

| Özellik | Açıklama |
|---------|----------|
| **DNS yönetimi** | Otomatik DNS kayıt oluşturma ve güncelleme |
| **CDN** | Statik içerik önbellekleme ile hızlı erişim |
| **DDoS koruması** | Otomatik saldırı engelleme |
| **SSL/TLS** | Full (strict) TLS modu |

Ingress Management sekmesinde her hostname'in Cloudflare durumunu görebilirsiniz.

---

## Trafik Yönetimi

### Gateway API

Servislerinizin trafik yönetimi için Kubernetes Gateway API desteği mevcuttur. Bu, daha gelişmiş yönlendirme senaryolarını destekler:

- Ağırlıklı trafik dağıtımı
- Header tabanlı yönlendirme
- gRPC desteği

### Rate Limiting

SaaS modunda her servis için sabit bir rate limit uygulanır. Bu, platformun kararlılığını ve adil kullanımını sağlar.

---

## Yönetilen Servislerin Erişim Adresleri

Addon servislerinin (PostgreSQL, RabbitMQ, Valkey) erişim adresleri farklı bir alt alan adı kullanır:

| Servis | Format | Port |
|--------|--------|:----:|
| **PostgreSQL** | `pg-{id}.devopszon.app` | 30930 |
| **RabbitMQ (AMQPS)** | `rmq-{id}.devopszon.app` | 5671 |
| **RabbitMQ (Management)** | `rmq-{id}.devopszon.app` | 15672 |
| **Valkey** | `valkey-{id}.devopszon.app` | Yapılandırılır |

Bu adresler SNI (Server Name Indication) routing ile çalışır ve her instance'a özel TLS sertifikası atanır.

---

## İpuçları

- **Wildcard domain:** `*.mycompany.com` şeklinde wildcard CNAME kaydı oluşturarak tüm alt domainleri yönlendirebilirsiniz
- **DNS yayılım süresi:** Doğrulama başarısız olursa birkaç dakika bekleyip tekrar deneyin
- **HTTPS zorunluluğu:** Tüm servisler varsayılan olarak HTTPS üzerinden sunulur; HTTP istekleri otomatik yönlendirilir
