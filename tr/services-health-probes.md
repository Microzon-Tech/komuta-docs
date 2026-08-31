# Sağlık Kontrolleri

Sağlık kontrolleri, Komuta'nın servisinizin çalışır durumda olup olmadığını düzenli aralıklarla sorgulamasını sağlar. Kontrol başarısız olduğunda servis ya trafikten çıkarılır ya da yeniden başlatılır — hangisinin olacağı kontrolün türüne bağlıdır.

Kaydedilen ayarlar yaklaşık 30 saniye içinde servise uygulanır.

![asd](https://cdn.komuta.io/docs/tr/images/health-probes/services-health-controls.png)
---

## Kontrol Türleri

Üç kontrol birbirinden bağımsız açılıp kapatılır ve farklı sonuçlar doğurur.

| Kontrol | Başarısız olduğunda ne olur |
|---|---|
| **Readiness** | Servis load balancer'dan çıkarılır; trafik almaz ama çalışmaya devam eder. |
| **Liveness** | Container yeniden başlatılır. |
| **Startup** | Container'ın açılışını tamamlaması beklenir; bu kontrol geçene kadar diğer ikisi devreye girmez. |

Yavaş açılan servislerde **Startup** kontrolü olmadan Liveness devreye girip container'ı henüz hazır olmadan yeniden başlatabilir. Açılışı uzun süren bir uygulama varsa önce bu kontrol tanımlanır.

---

## Kontrol Yöntemi

Her kontrol üç yöntemden biriyle yapılır:

- **HTTP GET** — belirtilen yola istek atılır, başarılı yanıt beklenir. Varsayılan yol `/health`, port 80.
- **TCP** — belirtilen porta bağlantı kurulabiliyorsa kontrol başarılı sayılır. HTTP uç noktası olmayan servisler için kullanılır.
- **Exec** — container içinde bir komut çalıştırılır; komut sıfır çıkış koduyla biterse kontrol geçer.

---

## Zamanlama Ayarları

| Ayar | Ne belirler | Varsayılan |
|---|---|---|
| **İlk gecikme** | Container başladıktan sonra ilk kontrole kadar beklenen süre | 10 sn |
| **Periyot** | Kontroller arasındaki aralık | 10 sn |
| **Zaman aşımı** | Bir kontrolün yanıtsız kalması durumunda başarısız sayılma süresi | 5 sn |
| **Başarı eşiği** | Başarısız durumdan sağlıklıya dönmek için gereken ardışık başarı sayısı | 1 |
| **Hata eşiği** | Başarısız sayılmak için gereken ardışık hata sayısı | 3 |

> **Uyarı:** Liveness kontrolünde hata eşiğini ve periyodu çok düşük tutmak, geçici bir yavaşlamada container'ın gereksiz yere yeniden başlatılmasına yol açar. Yeniden başlatma sırasında servis kısa süreli kesintiye girer.

---

## İlgili Dokümanlar

- [Servis Dashboard](service-dashboard.md) — Servisin anlık sağlık durumu ve pod'lar.
- [Otomatik Ölçekleme](autoscaling.md) — Replika sayısının yüke göre ayarlanması.
