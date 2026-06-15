# Platform Genel Bakış

DevOpsZon, modern uygulamalarınızı Kubernetes üzerinde uçtan uca yönetmenizi sağlayan kapsamlı bir DevOps platformudur. Bu sayfa, platformun sunduğu tüm yetenekleri ve çalışma modellerini özetler.

---

## Çalışma Modelleri

DevOpsZon üç farklı çalışma modeli sunar:

### SaaS (Platform as a Service)

DevOpsZon'un yönettiği altyapı üzerinde uygulamalarınızı yayınlayın. Kubernetes cluster kurulumu, bakım ve izleme tamamen DevOpsZon tarafından yapılır.

- Kaynak ihtiyacınıza göre plan seçin
- Git reponuzu bağlayın, servisleri keşfedin ve yayınlayın
- Her servis için otomatik hostname oluşturulur (ör: `my-api-a1b2c3d4.devopszon.com`)
- Aylık faturalandırma ile kullandığınız kadar ödeyin

### Hibrit

Kendi Kubernetes cluster'ınızı DevOpsZon'a bağlayarak yönetim panelinin tüm yeteneklerinden yararlanın.

- **Mevcut cluster:** Kubeconfig ile bağlayın
- **Yeni cluster:** DevOpsZon arayüzünden bulut sağlayıcınız üzerinde otomatik kurulum yapın
- Birden fazla cluster bağlayabilirsiniz
- Tüm kaynaklar size aittir; sabit fiyat politikası uygulanır

### On-Premise

Tamamen izole ortamınızda çalışan Kubernetes platformuna DevOpsZon kurulumu. Verileriniz ve altyapınız %100 sizin kontrolünüzde kalır.

---

## Temel Yetenekler

### Proje ve Servis Yönetimi

| Yetenek | Açıklama |
|---------|----------|
| **Proje organizasyonu** | Uygulamalarınızı projeler altında gruplandırın |
| **Otomatik servis keşfi** | Git repo'larınızdaki Dockerfile'lar otomatik tespit edilir |
| **Çoklu repo desteği** | Bir projeye birden fazla repository bağlayabilirsiniz |
| **Toplu işlemler** | Birden fazla servisi aynı anda deploy edebilirsiniz |

### CI/CD Pipeline

| Yetenek | Açıklama |
|---------|----------|
| **Komuta Pipeline** | Kubernetes-native CI/CD altyapısı |
| **Otomatik build & push** | Dockerfile'dan imaj oluşturma ve registry'ye gönderme |
| **Canlı pipeline izleme** | Build sürecini adım adım gerçek zamanlı takip edin |
| **Pipeline geçmişi** | Tüm build geçmişinizi görüntüleyin ve karşılaştırın |
| **Kuyruk yönetimi** | Eşzamanlı build'lerde otomatik kuyruk |

### Deploy Stratejileri

| Strateji | Açıklama |
|----------|----------|
| **Rolling Update** | Varsayılan strateji; pod'lar kademeli olarak güncellenir |
| **Canary** | Yeni sürümü trafiğin küçük bir yüzdesine yönlendirerek test edin |
| **Blue/Green** | Yeni sürümü paralel ortamda hazırlayıp tek seferde geçiş yapın |
| **Auto-Promote** | Belirlenen koşullar sağlandığında otomatik tam geçiş |

### Yönetilen Servisler (Addon'lar)

| Servis | Özellikler |
|--------|-----------|
| **PostgreSQL** | Tam yönetimli veritabanı; Single ve HA (Primary + Replica) topolojileri; otomatik yedekleme; connection pooling (PgBouncer); TLS ile güvenli bağlantı |
| **RabbitMQ** | Tam yönetimli mesaj kuyruğu; Single Broker ve HA Quorum Cluster; Management UI; otomatik yedekleme; TLS |
| **Valkey** | Tam yönetimli in-memory veri deposu; yüksek performanslı önbellek çözümü |

### İzleme ve Gözlemlenebilirlik

| Yetenek | Açıklama |
|---------|----------|
| **Gerçek zamanlı dashboard** | Pod durumu, trafik akışı ve kaynak tüketimi |
| **Canlı trafik görselleştirmesi** | Ingress → Pod arası trafik akışını canlı izleyin |
| **Komuta Metrics** | CPU, bellek, ağ kullanımı ve özel metrikler |
| **Komuta Logs** | Merkezi log toplama ve arama |
| **Komuta Alerts** | Metrik ve log tabanlı uyarı kuralları |
| **Çoklu bildirim kanalı** | E-posta, Slack, Telegram, Teams, PagerDuty, SMS, WhatsApp |

### Ağ ve Erişim

| Yetenek | Açıklama |
|---------|----------|
| **Ingress yönetimi** | Host tabanlı yönlendirme kuralları |
| **Özel domain** | Kendi alan adınızı servislere bağlayın |
| **TLS/SSL** | Otomatik sertifika yönetimi (Let's Encrypt) |
| **Cloudflare entegrasyonu** | DNS ve CDN entegrasyonu |
| **Otomatik hostname** | Her servis için benzersiz `*.devopszon.com` adresi |

### Güvenlik ve Yetkilendirme

| Yetenek | Açıklama |
|---------|----------|
| **Erişim kontrolü** | Proje, servis, cluster ve entegrasyon bazında Read/Edit yetkileri |
| **Çoklu kullanıcı** | Takım üyelerinizi davet edin ve yetkilerini yönetin |
| **Tenant izolasyonu** | Her organizasyon tamamen izole ortamda çalışır |
| **Ağ politikaları** | Pod seviyesinde ağ izolasyonu ve kural tabanlı trafik yönetimi |

---

## Mimari Genel Bakış

```
┌──────────────────────────────────────────────────┐
│                DevOpsZon Console                  │
│                 (Web Uygulaması)                  │
└────────────────────┬─────────────────────────────┘
                     │ REST API + Canlı Güncelleme
┌────────────────────▼─────────────────────────────┐
│                  DevOpsZon API                    │
│    Proje / Servis / Cluster / Pipeline / Alert    │
└───┬──────────┬──────────┬──────────┬─────────────┘
    │          │          │          │
┌───▼────┐ ┌──▼────┐ ┌───▼────┐ ┌───▼────┐
│ Komuta │ │Komuta │ │ Komuta │ │ Komuta │
│Pipeline│ │Rollout│ │Metrics │ │Gateway │
│(CI/CD) │ │(Deploy│ │ + Logs │ │  + DNS │
└────────┘ └───────┘ └────────┘ └────────┘
```

- **Komuta Pipeline:** Kubernetes-native CI/CD pipeline'ları
- **Komuta Rollout:** Gelişmiş deploy stratejileri (Canary, Blue/Green, Auto-Promote)
- **Komuta Metrics + Logs:** Metrik toplama ve merkezi log yönetimi
- **Komuta Gateway:** Trafik yönlendirme, DNS yönetimi ve otomatik hostname kayıtları
- **Gerçek zamanlı güncelleme:** Bildirimler ve durum değişiklikleri anında panele yansır

---

## Desteklenen Entegrasyonlar

### Git Sağlayıcıları
- GitHub
- GitLab
- Bitbucket
- Azure DevOps

### Container Registry'ler
- Docker Hub
- GitHub Container Registry (GHCR)
- GitLab Container Registry
- Azure Container Registry (ACR)
- Amazon Elastic Container Registry (ECR)
- Google Container Registry (GCR)
- Özel (Custom) Registry

### Bildirim Kanalları
- E-posta (SMTP)
- Slack
- Telegram
- Microsoft Teams
- PagerDuty
- SMS
- WhatsApp
- Webhook

### Bulut Sağlayıcıları
- Hetzner Cloud (birincil)
- Genişletilebilir bulut sağlayıcı altyapısı
