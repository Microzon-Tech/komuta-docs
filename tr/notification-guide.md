# Bildirim Ayarları

DevOpsZon'un bildirim sistemi, uyarılar, pipeline durumları ve önemli olaylar hakkında sizi bilgilendirmek için çoklu kanal desteği sunar. Bu rehber, bildirim kanallarınızı nasıl yapılandıracağınızı açıklar.

---

## Bildirim Sistemi Genel Bakışı

DevOpsZon'daki bildirimler üç kategori altında toplanır:

| Kategori | Örnekler |
|----------|----------|
| **Uyarı Bildirimleri** | Alert kuralı tetiklendiğinde gönderilen bildirimler |
| **Pipeline Bildirimleri** | Build başarılı/başarısız durumları |
| **Sistem Bildirimleri** | Fatura, bakiye, bakım ve platform duyuruları |

---

## Bildirim Kanalları

Sol menüden **Notifications** sayfasına giderek bildirim kanallarınızı yapılandırın.

### E-posta

| Parametre | Açıklama |
|-----------|----------|
| **Alıcı adresleri** | Bildirimin gönderileceği e-posta adresleri |

E-posta bildirimleri varsayılan olarak hesap e-posta adresinize gönderilir. Ek alıcılar ekleyebilirsiniz.

### Slack

Slack entegrasyonu için bir Incoming Webhook URL'si gereklidir:

1. Slack workspace'inizde bir Incoming Webhook oluşturun
2. Webhook URL'sini **Notifications** sayfasına girin
3. Bildirimin gönderileceği kanalı belirleyin
4. Test bildirimi göndererek doğrulayın

### Telegram

| Parametre | Açıklama |
|-----------|----------|
| **Bot Token** | Telegram BotFather'dan alınan bot token'ı |
| **Chat ID** | Bildirimin gönderileceği chat veya grup ID'si |

### Microsoft Teams

| Parametre | Açıklama |
|-----------|----------|
| **Webhook URL** | Teams kanalındaki Incoming Webhook URL'si |

### PagerDuty

| Parametre | Açıklama |
|-----------|----------|
| **Integration Key** | PagerDuty servisine ait integration key |

PagerDuty entegrasyonu, uyarı şiddetine (severity) göre olay oluşturur ve olay yönetim sürecinize dahil eder.

### SMS

| Parametre | Açıklama |
|-----------|----------|
| **Telefon numarası** | Bildirimin gönderileceği telefon numarası |

### WhatsApp

| Parametre | Açıklama |
|-----------|----------|
| **Business API** | WhatsApp Business API yapılandırması |

### Webhook (Özel)

Kendi sistemlerinize bildirim göndermek için özel webhook tanımlayın:

| Parametre | Açıklama |
|-----------|----------|
| **URL** | Bildirimin POST edileceği HTTP endpoint |
| **Headers** | İsteğe bağlı HTTP başlıkları (ör: Authorization) |

---

## Kanal Yapılandırma

### Yeni Kanal Ekleme

1. **Notifications** sayfasına gidin
2. **Yeni Kanal Ekle** butonuna tıklayın
3. Kanal tipini seçin
4. Gerekli bilgileri girin
5. **Test** butonuyla deneme bildirimi gönderin
6. **Kaydet** butonuyla kanalı kaydedin

### Kanalı Uyarı Kuralına Bağlama

Bildirim kanalları, uyarı kuralları ile ilişkilendirilir:

1. **Alert Management** sayfasına gidin
2. Bir uyarı kuralı oluşturun veya düzenleyin
3. **Bildirim kanalları** bölümünde hangi kanallara bildirim gönderileceğini seçin
4. Birden fazla kanal seçebilirsiniz (ör: hem Slack hem e-posta)

---

## Panel İçi Bildirimler

Kanal bildirimlerine ek olarak, DevOpsZon Console'da gerçek zamanlı panel içi bildirimler alırsınız:

- Sağ üst köşedeki bildirim simgesi ile yeni bildirimleri görüntüleyin
- Uyarı, pipeline ve sistem bildirimleri kronolojik sırayla listelenir
- Bildirime tıklayarak ilgili sayfaya yönlendirilebilirsiniz

---

## Bildirim Akışı

Bir uyarı tetiklendiğinde bildirim süreci:

```
Alert Tetiklenir → Alertmanager → DevOpsZon API → Bildirim Kuyruğu → Kanal'a Gönderim
                                       ↓
                               Panel'de Gerçek Zamanlı
                               Bildirim (SignalR)
```

1. Prometheus veya Loki'de kural eşiği aşılır
2. Alertmanager uyarıyı gruplar ve DevOpsZon webhook'una gönderir
3. DevOpsZon API, uyarıyı işler ve ilişkili bildirim kanallarını bulur
4. Her kanal için bildirim kuyruğa eklenir (RabbitMQ)
5. Kuyruk tüketicisi bildirimi ilgili kanala gönderir
6. Aynı anda panel üzerinde SignalR ile gerçek zamanlı güncelleme yapılır

### Retry Mekanizması

Bildirim gönderimi başarısız olursa:
- Otomatik olarak yeniden denenir
- Birden fazla başarısız denemeden sonra Dead Letter Queue'ya (DLQ) taşınır
- DLQ'daki mesajlar manuel olarak yeniden işlenebilir

---

## İpuçları

- **Çoklu kanal:** Kritik uyarılar için birden fazla bildirim kanalı tanımlayın (ör: Slack + E-posta + PagerDuty)
- **Severity bazlı:** Warning uyarıları Slack'e, Critical uyarıları PagerDuty'ye yönlendirin
- **Test:** Her yeni kanalı ekledikten sonra test bildirimi göndererek doğrulayın
- **Bildirim yorgunluğu:** Çok fazla bildirim, önemli bildirimlerin gözden kaçmasına neden olur; yalnızca aksiyon gerektiren olaylar için bildirim ayarlayın
