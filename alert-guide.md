# Uyarı Yönetimi (Alert Management)

DevOpsZon'un uyarı sistemi, servislerinizin ve altyapınızın sağlığını proaktif olarak izlemenizi sağlar. Metrik ve log tabanlı kurallar tanımlayarak, sorunları kullanıcılarınızdan önce tespit edin.

---

## Uyarı Sistemi Nasıl Çalışır?

DevOpsZon uyarı pipeline'ı şu aşamalardan oluşur:

```
Metrik/Log Toplama → Kural Değerlendirme → Uyarı Tetikleme → Bildirim Gönderme
 (Komuta Metrics    →    (Uyarı kuralı)  → (Komuta Alerts) → (E-posta/Slack/...)
  / Komuta Logs)
```

1. **Komuta Metrics** cluster'daki metrikleri toplar
2. **Komuta Logs** uygulama loglarını toplar
3. Tanımlanan kurallar periyodik olarak değerlendirilir
4. Koşul sağlandığında **Komuta Alerts** üzerinden uyarı tetiklenir
5. Bildirim kanallarınıza anlık bildirim gönderilir
6. Panel üzerinde **gerçek zamanlı** güncelleme yapılır

---

## Uyarı Kapsamları

DevOpsZon'da uyarılar iki kapsamda yönetilir:

### Global Uyarılar

Sol menüdeki **Alerts** sayfasından uygulama genelindeki tüm uyarı kurallarını yönetin. Bu sayfada:
- Tüm cluster ve servislere ait uyarılar listelenir
- Cluster veya servis bazında filtreleme yapabilirsiniz
- Yeni kural oluşturabilir, mevcut kuralları düzenleyebilirsiniz

### Servis Bazlı Uyarılar

**Service Management** → **Alert Management** sekmesinden yalnızca seçili servise ait uyarıları yönetin. Bu, tek bir servisin sağlığına odaklanmanızı sağlar.

---

## Uyarı Kuralı Oluşturma

### Adım 1: Kural Tipini Seçin

| Tip | Açıklama | Sorgu Dili |
|-----|----------|-----------|
| **Metrik tabanlı** | CPU, bellek, istek sayısı gibi metriklere dayalı | Metrik sorgu dili |
| **Log tabanlı** | Uygulama loglarındaki kalıplara dayalı | Log sorgu dili |

### Adım 2: Sorguyu Tanımlayın

**Metrik tabanlı örnekler:**

| Senaryo | Metrik Sorgusu |
|---------|----------------|
| CPU %80'in üzerinde | `rate(container_cpu_usage_seconds_total[5m]) > 0.8` |
| Bellek %90'ın üzerinde | `container_memory_working_set_bytes / container_spec_memory_limit_bytes > 0.9` |
| 5xx hata oranı yüksek | `rate(http_requests_total{status=~"5.."}[5m]) > 0.05` |
| Pod restart sayısı arttı | `increase(pod_container_status_restarts_total[1h]) > 3` |

**Log tabanlı örnekler:**

| Senaryo | Log Sorgusu |
|---------|----------------|
| Error logu tespit | `{app="my-service"} |= "ERROR"` |
| Exception sayısı yüksek | `count_over_time({app="my-service"} |= "Exception" [5m]) > 10` |

### Adım 3: Şiddeti (Severity) Belirleyin

| Seviye | Kullanım Alanı |
|--------|----------------|
| **Info** | Bilgilendirme amaçlı; acil müdahale gerektirmez |
| **Warning** | Dikkat gerektiren durum; yakında sorun olabilir |
| **Critical** | Acil müdahale gerekli; servis etkilenmiş olabilir |
| **Emergency** | Sistem tamamen etkilenmiş; anında müdahale şart |

### Adım 4: Süreyi Belirleyin

Uyarının tetiklenmesi için koşulun kaç dakika boyunca sürekli sağlanması gerektiğini belirleyin. Bu, geçici dalgalanmaların yanlış alarm oluşturmasını önler.

| Süre | Kullanım |
|------|----------|
| **1 dakika** | Anlık sorunlar için hızlı tespit |
| **5 dakika** | Genel amaçlı; çoğu senaryo için uygun |
| **15 dakika** | Trend bazlı sorunlar; kısa süreli dalgalanmaları filtreler |

### Adım 5: Bildirim Kanalını Seçin

Uyarı tetiklendiğinde hangi kanallara bildirim gönderileceğini seçin. Birden fazla kanal seçebilirsiniz.

### Adım 6: Test Edin

Kuralınızı kaydetmeden önce **Test** butonuyla doğrulayın. Test, sorgunuzu cluster üzerinde çalıştırarak mevcut durumda uyarının tetiklenip tetiklenmeyeceğini gösterir.

### Adım 7: Kubernetes'e Deploy Edin

Kaydettiğiniz kural otomatik olarak Kubernetes cluster'ına PrometheusRule CRD olarak deploy edilir ve izleme başlar.

---

## Uyarı Şablonları

DevOpsZon, sık kullanılan senaryolar için hazır uyarı şablonları sunar:

| Şablon | Açıklama |
|--------|----------|
| **Yüksek CPU Kullanımı** | CPU limiti aşılmak üzere |
| **Yüksek Bellek Kullanımı** | Bellek limiti aşılmak üzere |
| **Pod Restart Döngüsü** | Pod sürekli yeniden başlatılıyor |
| **5xx Hata Oranı** | HTTP 5xx hataları artıyor |
| **Disk Doluluk** | Disk kapasitesi azalıyor |
| **Veritabanı Bağlantı Havuzu** | Bağlantı havuzu dolmak üzere |

> Şablonları doğrudan kullanabilir veya özelleştirerek yeni kurallar oluşturabilirsiniz.

---

## Bildirim Kanalları

Uyarı bildirimlerini şu kanallar üzerinden alabilirsiniz:

| Kanal | Yapılandırma |
|-------|-------------|
| **E-posta** | SMTP ayarları ve alıcı adresleri |
| **Slack** | Webhook URL ile kanal entegrasyonu |
| **Telegram** | Bot token ve chat ID |
| **Microsoft Teams** | Incoming webhook URL |
| **PagerDuty** | Integration key ile olay yönetimi |
| **SMS** | Telefon numarası ve SMS sağlayıcı ayarları |
| **WhatsApp** | Business API entegrasyonu |
| **Webhook** | Özel HTTP endpoint'e POST isteği |

Bildirim kanallarını **Notifications** sayfasından yapılandırabilirsiniz.

---

## Uyarı Susturma (Silence)

Planlı bakım veya bilinen sorunlar sırasında belirli uyarıları geçici olarak susturabilirsiniz:

1. Uyarı listesinde susturmak istediğiniz kuralın yanındaki **Sustur** butonuna tıklayın
2. Süre belirleyin (ör: 2 saat, 1 gün)
3. İsteğe bağlı bir açıklama ekleyin
4. Belirlenen süre sonunda uyarı otomatik olarak tekrar aktif olur

---

## Addon Uyarıları

Yönetilen servisler (PostgreSQL, RabbitMQ, Valkey) oluşturulduğunda otomatik olarak temel uyarı kuralları tanımlanır:

| Servis | Otomatik Uyarılar |
|--------|-------------------|
| **PostgreSQL** | Yüksek CPU, bellek, disk, bağlantı havuzu, replication lag |
| **RabbitMQ** | Kuyruk doluluk, bellek, bağlantı sayısı |
| **Valkey** | Bellek kullanımı, bağlantı sayısı |

Bu uyarılar varsayılan olarak aktiftir ve ihtiyacınıza göre özelleştirebilirsiniz.

---

## İpuçları

- **Kademeli uyarı:** Aynı metrik için farklı eşiklerle Warning ve Critical uyarıları tanımlayın
- **Süre ayarı:** Çok kısa süreler yanlış alarm üretir, çok uzun süreler geç tespit'e neden olur
- **Test:** Her kuralı kaydetmeden önce mutlaka test edin
- **Bildirim yorgunluğu:** Çok fazla uyarı, önemli uyarıların gözden kaçmasına neden olur; sadece aksiyon gerektiren uyarılar tanımlayın
