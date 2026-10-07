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

## Özel Alan Adı Bağlama

Servisinizi, Komuta'nın verdiği adresin yanında kendi alan adınızdan da (ör. `api.sirketim.com`) yayınlayabilirsiniz.

### Başlamadan Önce

- Alan adı, HTTP trafiği sunan bir servise bağlanmalıdır. Worker, Job ve CronJob'lara özel alan adı bağlanamaz.
- Alt alan adıyla başlayın (`www.sirketim.com`, `api.sirketim.com`). Kök alan adı (`sirketim.com`) için DNS sağlayıcınızın kök kayıtta CNAME flattening, ALIAS veya ANAME desteklemesi gerekir.
- Wildcard alan adları (`*.sirketim.com`) desteklenmez. Her alt alan adını ayrı ekleyin.

### 1. Alan Adını Ekleyin

1. Sol menüden **Alan Adları** sayfasını açın.
2. **Alan adı ekle**'ye tıklayın.
3. Alan adını girin ve **Hedef Uygulama**'yı seçin.

### 2. DNS Kayıtlarını Ekleyin

Alan adının detayındaki **Eklenecek DNS Kayıtları** bölümü gerekli kayıtları listeler. Hepsini DNS sağlayıcınızda gösterildiği gibi oluşturun:

| Amaç | Tip | Ad | Değer |
|------|-----|----|-------|
| Trafik yönlendirme | CNAME | `api.sirketim.com` | `origin.komuta.app` |
| Sahiplik doğrulama | TXT | `_cf-custom-hostname.api.sirketim.com` | Konsolda gösterilen kod |
| Sertifika doğrulama | CNAME | `_acme-challenge.api.sirketim.com` | `api.sirketim.com.<id>.dcv.cloudflare.com` (konsolda gösterilir) |

- Değerleri yanlarındaki kopyalama düğmesiyle alın. Sahiplik kodu ve sertifika doğrulama hedefi alan adınıza özeldir.
- Sertifika doğrulama kaydını bir kez eklersiniz. Sertifikalar bundan sonra otomatik yenilenir; DNS'e tekrar dokunmanız gerekmez.
- DNS'iniz Cloudflare'deyse bu kayıtları **DNS only** (gri bulut) yapın. Kendi Cloudflare hesabınızda proxy'lenen (turuncu bulut) bir kayıt doğrulanamaz ve erişim koruması onu kapsayamaz.
- Daha önce eklenen alan adları `origin.edge-1.komuta.app` gibi bir adrese yönleniyor olabilir. Bunlar çalışmaya devam eder; değiştirmeniz gerekmez.

> DNS değişikliklerinin yayılması birkaç dakika ile 48 saat arasında sürebilir.

### 3. Etkinleşmesini Bekleyin

Her kaydın yanında Komuta'nın kaydı görüp görmediği yazar. Sahiplik doğrulaması ve sertifika tamamlandığında alan adının durumu **Aktif** olur ve servisinize `https://api.sirketim.com` üzerinden erişilir.

- Durumu hemen kontrol etmek için **Doğrulamayı Yenile**'ye tıklayın.
- 14 gün içinde etkinleşmeyen bir alan adı otomatik olarak kaldırılır. İstediğiniz zaman yeniden ekleyebilirsiniz.
- Aktif bir alan adı dikkat gerektiriyor diye işaretlenirse (ör. DNS kaydı artık Komuta'yı göstermiyorsa) yukarıdaki kayıtları kontrol edin.

### Alan Adını Kaldırma

Alan adını seçip **Kaldır**'a tıklayın. Bir servisi silmek, ona bağlı özel alan adlarını da kaldırır.

---

## Cloudflare Entegrasyonu

Özel alan adları Cloudflare üzerinden sunulur:

| Özellik | Açıklama |
|---------|----------|
| **CDN** | Statik içerik önbellekleme ile hızlı erişim |
| **DDoS koruması** | Otomatik saldırı engelleme |
| **SSL/TLS** | Sertifikalar otomatik oluşturulur ve yenilenir |

Her alan adının durumunu **Alan Adları** sayfasında görebilirsiniz.

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

- **DNS yayılım süresi:** Bir kayıt hâlâ görünmüyorsa birkaç dakika bekleyip **Doğrulamayı Yenile**'ye tıklayın
- **HTTPS zorunluluğu:** Tüm servisler varsayılan olarak HTTPS üzerinden sunulur; HTTP istekleri otomatik yönlendirilir
