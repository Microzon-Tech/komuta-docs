# Dağıtım Geçmişi

Bu sayfa, servisin şu anda hangi sürümü çalıştırdığını ve daha önce hangi sürümlerin yayına alındığını gösterir. Bir dağıtım sorun çıkardığında sürüm buradan değiştirilir.

![alt text](https://cdn.komuta.io/docs/tr/images/deployments/deployment-history-page.jpeg)
---

## Mevcut Sürüm

Sayfanın üstünde servisin canlıda çalışan sürümü yer alır: hangi imaj, kaçıncı revizyon, kaç replika ve hangi rollout stratejisiyle çalıştığı. Rollout stratejisi de buradan değiştirilir; değişiklik bir sonraki deploy'da geçerli olur.

![alt text](https://cdn.komuta.io/docs/tr/images/deployments/deployment-history-current-version.jpeg)
---

## Dağıtım Stratejileri

Strateji, yeni sürümün canlıya nasıl alınacağını belirler. Servisin hangi stratejiyle çalıştığı mevcut sürüm bölümünde görünür ve oradan değiştirilir.

| Strateji | Kime uygun | Nasıl çalışır |
|---|---|---|
| **Blue-Green** | Çoğu servis | İki özdeş ortam. Sıfır kesinti, kolay geri alma; canlıya geçiş için manuel onay gerekir. |
| **Auto Promote** | Müdahalesiz akış isteyenler | Blue-Green ile aynı, ancak yeni sürüm hazır olduğunda geçiş otomatik yapılır. |
| **Canary** | Gelişmiş kullanım | Trafiği yeni sürüme kademeli aktarır. Yeni sürüme geçiş manuel yönetim gerektirir. |

Ayrıntılar için [Deploy Stratejileri](deployment-strategies.md) sayfasına bakılabilir.

---

## Rollout İşlemleri

Devam eden bir rollout'a müdahale etmek için kullanılan işlemler burada toplanır.

| İşlem | Ne yapar |
|---|---|
| **Canlıya al** | Yeni sürümü tam trafiğe geçirir |
| **Devam** | Duraklatılmış rollout'u bir sonraki adımdan sürdürür |
| **Duraklat** | Rollout'u olduğu yerde bekletir |
| **Durdur** | Rollout'u iptal eder; stabil sürüm yayında kalır |
| **Yeniden dene** | Başarısız olan rollout'u baştan dener |
| **Yeniden başlat** | Çalışan sürümün pod'larını yeniden oluşturur |

---

## Önceki Bir Sürüme Dönme

Her dağıtım kaydında **Geri al** işlemi bulunur. Onaylandığında servis o revizyona döner.

> **Uyarı:** Geri alma, devam eden bir rollout'u iptal eder. Canary dağıtımı sürerken geri alınırsa kademeli geçiş yarıda kalır ve trafik eski sürüme döner.

![alt text](https://cdn.komuta.io/docs/tr/images/deployments/deployment-history-rollback.jpeg)
---

## İlgili Dokümanlar

- [Deploy Stratejileri](deployment-strategies.md) — Blue-Green, Canary ve Auto Promote'un nasıl işlediği.
- [Pipeline'lar](pipeline-guide.md) — Dağıtımı üreten derleme ve yayınlama süreci.
- [Servis Dashboard](service-dashboard.md) — Yeni sürüm dağıtma ve anlık rollout durumu.
