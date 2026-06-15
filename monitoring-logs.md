# İzleme ve Log Yönetimi

DevOpsZon, servislerinizin performansını ve sağlığını gerçek zamanlı olarak izlemenizi sağlar. **Komuta Metrics** ile metrik izleme ve **Komuta Logs** ile merkezi log yönetimi sayesinde uygulamalarınızı her açıdan gözlemleyebilirsiniz.

---

## Servis Dashboard'u

Her servisin **Dashboard** sekmesi, o servisin anlık durumunu gösteren merkezi izleme noktasıdır.

### Canlı Trafik Görselleştirmesi

Dashboard'un ana bileşeni, servisinize gelen trafiğin akışını görsel olarak gösteren interaktif haritadır:

- **Ingress noktası:** Trafik giriş noktası ve hostname
- **Pod'lar:** Her pod'un durumu renk koduyla gösterilir
  - **Yeşil:** Sağlıklı ve trafik alıyor
  - **Sarı:** Başlatılıyor veya hazır değil
  - **Kırmızı:** Sağlıksız veya başarısız
- **Trafik okları:** Pod'lar arasındaki trafik akışı
- **Rollout durumu:** Canary/Blue-Green stratejilerinde her grubun durumu

> Bu görselleştirme gerçek zamanlı olarak güncellenir — sayfa yenilemesine gerek yoktur.

### Durum Kartları

Dashboard üzerindeki özet kartlar:

| Kart | Gösterim |
|------|----------|
| **Pod Durumu** | Running / Pending / Failed pod sayıları |
| **Replica Sayısı** | Mevcut / İstenen / Hazır replica |
| **HPA** | Mevcut / Min / Max replica (HPA aktifse) |
| **Kaynak Tüketimi** | CPU ve bellek kullanım yüzdeleri |
| **Son Pipeline** | Son build'in durumu ve süresi |
| **Aktif Uyarılar** | Tetiklenen uyarı sayısı |

---

## Metrik İzleme

DevOpsZon, **Komuta Metrics** altyapısı üzerinden servislerinizin metriklerini toplar ve izler.

### Temel Metrikler

| Metrik | Açıklama | Birim |
|--------|----------|-------|
| **CPU Kullanımı** | Servisin tükettiği CPU kaynağı | mCPU / core |
| **Bellek Kullanımı** | Servisin tükettiği RAM miktarı | MiB / GiB |
| **Ağ Trafiği** | Gelen ve giden ağ trafiği | bytes/s |
| **HTTP İstek Sayısı** | Saniyedeki istek sayısı | req/s |
| **HTTP Hata Oranı** | 4xx ve 5xx yanıtların oranı | % |
| **İstek Gecikme Süresi** | Yanıt süresi (latency) | ms |

### Kaynak İzleme (Resources Sekmesi)

**Service Management** → **Resources** sekmesinde:

- CPU request ve limit değerleri ile mevcut tüketim
- Bellek request ve limit değerleri ile mevcut tüketim
- Kaynak kullanım trendi

---

## Log Yönetimi

DevOpsZon, **Komuta Logs** altyapısı üzerinden tüm servis loglarını merkezi olarak toplar.

### Log Görüntüleme

**Service Management** → **Logs** sekmesinden servisinizin loglarına erişin:

1. **Tarih aralığı:** Görmek istediğiniz zaman dilimini seçin
2. **Arama:** Loglar içinde anahtar kelime arayın
3. **Pod filtresi:** Belirli bir pod'un loglarını görüntüleyin
4. **Gerçek zamanlı:** Yeni logları canlı olarak takip edin

### Log Arama

Log arama alanında şu yöntemleri kullanabilirsiniz:

| Yöntem | Örnek | Açıklama |
|--------|-------|----------|
| **Düz metin** | `error` | "error" içeren tüm loglar |
| **Büyük/küçük harf** | `ERROR` | Tam eşleşme ile arama |
| **Birden fazla terim** | `database connection failed` | Tüm terimleri içeren loglar |

### Pipeline Logları

Pipeline çalıştırmalarının loglarına **Pipeline Summary** sekmesinden erişin:

1. İlgili pipeline çalıştırmasına tıklayın
2. Görev listesinde log görmek istediğiniz adıma tıklayın
3. Detaylı build ve deploy logları görüntülenir

---

## Gerçek Zamanlı Güncellemeler

DevOpsZon, gerçek zamanlı iletişim altyapısı ile anlık bildirimler sunar:

| Olay | Bildirim |
|------|----------|
| **Pipeline ilerlemesi** | Her görev tamamlandığında durum güncellenir |
| **Pod durumu değişikliği** | Pod başlatma, durdurma, hata durumları |
| **Rollout ilerlemesi** | Canary/Blue-Green adımları |
| **Addon kurulumu** | Provisioning adımları ve tamamlanma |
| **Uyarı tetiklenmesi** | Yeni uyarılar anlık olarak gösterilir |

Tüm bu güncellemeler sayfa yenilemesi gerektirmeden otomatik olarak panele yansır.

---

## Cluster Olayları (Events)

Dashboard'da servisinizle ilgili cluster olayları gösterilir:

| Olay Tipi | Örnekler |
|-----------|----------|
| **Normal** | Pod başarıyla oluşturuldu, sağlık kontrolü geçti |
| **Warning** | Image pull hatası, kaynak yetersizliği, probe başarısız |

Bu olaylar, sorun giderme sırasında ilk bakılacak kaynaklardır.

---

## Hata Giderme Rehberi

### Pod Başlatılamıyor

| Olası Neden | Kontrol Yeri | Çözüm |
|-------------|-------------|-------|
| İmaj bulunamıyor | Dashboard → Events | Registry bağlantısını ve imaj adını kontrol edin |
| Kaynak yetersiz | Resources sekmesi | Request/limit değerlerini artırın veya node ekleyin |
| Crash döngüsü | Logs sekmesi | Uygulama loglarından hata nedenini bulun |

### Servis Yanıt Vermiyor

| Olası Neden | Kontrol Yeri | Çözüm |
|-------------|-------------|-------|
| Readiness probe başarısız | Health Probes sekmesi | Probe endpoint'ini ve parametrelerini kontrol edin |
| Port uyuşmazlığı | Ports sekmesi | Uygulamanın dinlediği port ile yapılandırılan port eşleşmeli |
| Bellek taşması (OOM) | Dashboard → Events | Memory limit'i artırın |

### Pipeline Başarısız

| Olası Neden | Kontrol Yeri | Çözüm |
|-------------|-------------|-------|
| Build hatası | Pipeline Logs | Dockerfile'ı kontrol edin |
| Git erişim hatası | Pipeline Logs → fetch-credentials | Git bağlantısını kontrol edin |
| Registry push hatası | Pipeline Logs → build-and-push | Registry bağlantısını kontrol edin |

---

## İpuçları

- **Dashboard'u açık tutun:** Gerçek zamanlı güncellemeler sayesinde anlık değişiklikleri kaçırmayın
- **Uyarılarla birlikte kullanın:** Metrik eşikleri için uyarı kuralları tanımlayarak proaktif izleme yapın
- **Log seviyesi:** Uygulamanızda yapılandırılabilir log seviyesi (DEBUG, INFO, WARN, ERROR) kullanarak gereksiz log hacmini azaltın
- **Zaman aralığı:** Log aramasında dar bir zaman aralığı seçerek sorgu performansını artırın
