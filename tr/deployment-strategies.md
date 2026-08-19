# Deploy Stratejileri

DevOpsZon, uygulamalarınızı güvenle yayınlamanız için **Komuta Rollout** altyapısı üzerinde gelişmiş deployment stratejileri sunar. Bu rehber, her stratejiyi adım adım anlatır ve dashboard üzerindeki görselleştirmeleri ekran görüntüleriyle destekler.

---

## Genel Bakış

DevOpsZon üç farklı deployment stratejisi sunar:

| Özellik | Canary | Blue/Green | Auto-Promote |
|---------|:------:|:----------:|:------------:|
| **Trafik kontrolü** | Kademeli (yüzde ile) | Anlık geçiş (%100) | Otomatik geçiş |
| **Preview ortamı** | Yok (tek hostname) | Var (ayrı hostname) | Var (ayrı hostname) |
| **Manuel onay** | Evet (her adımda) | Evet (promote) | Hayır |
| **Trafik yüzdesi ayarı** | Evet (adım adım artış) | Hayır | Hayır |
| **Anında rollback** | Evet (abort) | Evet (abort) | Evet (abort) |
| **Hostname sayısı** | 1 | 2 (active + preview) | 2 (active + preview) |
| **Sıfır kesinti** | Evet | Evet | Evet |
| **Önerilen kullanım** | Yüksek trafikli servisler | Kritik servisler | Güvenli otomatik dağıtım |

Her servisin aktif stratejisi, servis kartında badge olarak görünür:

![Servis listesinde strateji badge'leri](https://raw.githubusercontent.com/Microzon-Tech/docs/main/user-guide/img/deployment-strategies/services-overview.png)

---

## Yeni Deploy Başlatma

Herhangi bir strateji ile deploy başlatmak için:

1. **Servis Dashboard**'una gidin
2. Topoloji alanının üstündeki **Yeni Versiyon Yayınla** butonuna tıklayın
3. Açılan dialog'da:
   - **Mevcut Versiyon:** Şu an çalışan imaj etiketi görünür
   - **Yeni Versiyon Etiketi:** Deploy etmek istediğiniz etiketi girin (ör: `v1.2.3`, `latest`)
4. **Şimdi Dağıt** butonuna tıklayın

![Deploy dialog — yeni versiyon etiketi seçimi](https://raw.githubusercontent.com/Microzon-Tech/docs/main/user-guide/img/deployment-strategies/deploy-dialog.png)

### Deploy Kilidi

Bir rollout **Paused** durumdayken aynı servise yeni bir deploy tetiklenemez. Önce mevcut rollout'u **Canlıya Al** (Promote) veya **İptal Et** (Abort) ile sonlandırmanız gerekir.

---

## Canary Deployment

Canary stratejisi, yeni sürümü canlı trafiğin küçük bir yüzdesine yönlendirerek gerçek kullanıcılarla test etmenizi sağlar. Her adımda trafik yüzdesini kontrollü olarak artırırsınız.

### Nasıl Çalışır?

```
Deploy → %5 trafik → Durakla → İzle → %25 → Durakla → İzle → %50 → ... → %100 → Tamamlandı
```

1. Yeni sürüm pod'ları oluşturulur
2. **Komuta Gateway** üzerinden trafiğin küçük bir yüzdesi yeni pod'lara yönlendirilir
3. Rollout **duraklar** ve her adımda sizden onay bekler
4. **Resume** ile sonraki adıma geçersiniz, **Canlıya Al** ile doğrudan %100'e alırsınız
5. Herhangi bir adımda **İptal Et** ile tüm trafiği eski sürüme döndürebilirsiniz

### Trafik Adımları

Adımlar rollout manifest'inde tanımlanır. Platform hedef ağırlığa ulaşmak için gerekli promote komutlarını otomatik çalıştırır:

| Adım | Canary | Stable | Durum |
|:----:|:------:|:------:|-------|
| 1 | %5 | %95 | Duraklatılır |
| 2 | %25 | %75 | Duraklatılır |
| 3 | %50 | %50 | Duraklatılır |
| 4 | %100 | %0 | Tamamlandı |

> **Önemli:** Trafik yüzdesi yalnızca artırılabilir. Yüzdeyi düşürmek için **İptal Et** kullanın.

### Dashboard'da Canary

Canary deploy aktifken dashboard'da şunları görürsünüz:

- **Canary İlerlemesi:** Üst kısımda segmented progress bar — her adımın yüzdesini gösterir, aktif adımda dolgu animasyonu
- **Topoloji diyagramı:** HTTPRoute → Stable Service (%80) + Canary Service (%20) ayrımı, pod'lar ve trafik yüzdeleri
- **Aksiyon butonları:** ▶ Resume (sonraki adım yüzdesi) ve ▲ Canlıya Al butonları
- **Badge'ler:** Canary stratejisi, Duraklatıldı durumu

![Canary Paused — trafik bölüşümü ve progress bar](https://raw.githubusercontent.com/Microzon-Tech/docs/main/user-guide/img/deployment-strategies/canary-paused.png)

### Aksiyonlar

| Aksiyon | Açıklama |
|---------|----------|
| **Resume** | Sonraki trafik adımına ilerle |
| **Canlıya Al** | Tüm trafiği anında yeni sürüme yönlendir (%100) |
| **İptal Et** | Canary'yi iptal et, tüm trafiği eski sürüme döndür |
| **Duraklat** | İlerlemeyi geçici olarak durdur |

### Ne Zaman Kullanmalı?

- Yüksek trafikli servislerde riski minimize etmek istediğinizde
- Yeni sürümün performansını gerçek trafik ile ölçmek istediğinizde
- Kademeli ve kontrollü bir geçiş istediğinizde

---

## Blue/Green Deployment

Blue/Green stratejisi, yeni sürümü tamamen ayrı bir ortamda (**Preview**) hazırlayıp test ettikten sonra tek bir adımda canlıya almanızı sağlar.

### Nasıl Çalışır?

```
Deploy → Preview Oluştur → Test Et → Canlıya Al → Eski Temizle → Tamamlandı
```

1. **Active (Blue):** Mevcut canlı sürüm trafik almaya devam eder
2. **Preview (Green):** Yeni sürüm ayrı pod'larda hazırlanır
3. Preview hostname üzerinden yeni sürümü test edersiniz
4. **Canlıya Al** ile tüm trafik anlık olarak Preview'a yönlendirilir
5. Eski Active pod'ları temizlenir

### Hostname'ler

Blue/Green stratejisinde iki ayrı hostname oluşturulur:

| Tür | Format | Kullanım |
|-----|--------|----------|
| **Active** | `{servis}-{id}.devopszon.com` | Canlı trafik |
| **Preview** | `{servis}-{id}-preview.devopszon.com` | Test trafiği |

### Dashboard'da Blue/Green

Blue/Green deploy aktifken dashboard'da:

- **Topoloji diyagramı:** HTTPRoute → Active Service (canlı, %100 trafik) + Preview Service (test, kesikli çizgi) → Pod'lar
- **Progress bar:** 4 aşamalı stepper — Creating → Ready/Paused → Promoting → Active
- **Durum badge:** "Blue-Green" stratejisi ve "Duraklatıldı" / "Tamamlandı" durumu

![Blue/Green dashboard — topoloji ve trafik akışı](https://raw.githubusercontent.com/Microzon-Tech/docs/main/user-guide/img/deployment-strategies/bluegreen-dashboard-healthy.png)

Yeni bir deploy tetiklendiğinde rollout **Paused** duruma geçer. Bu durumda:

- **Blue-Green İlerlemesi:** 4 aşamalı stepper — Creating ✓ → Paused ⏸ → Promoting → Active
- Preview hostname ayrı bir URL olarak gösterilir
- **Canlıya Al** butonu aktif hale gelir

![Blue/Green Paused — Preview hazır, promote bekliyor](https://raw.githubusercontent.com/Microzon-Tech/docs/main/user-guide/img/deployment-strategies/bluegreen-paused.png)

### Aksiyonlar

| Aksiyon | Açıklama |
|---------|----------|
| **Canlıya Al** | Tüm trafiği yeni sürüme yönlendir |
| **İptal Et** | Preview'ı iptal et, eski sürüm devam etsin |
| **Geri Al** | Önceki sürüme geri dön |

### Sağlıksız Preview

Preview pod'ları sağlıksız duruma düşerse (crash, readiness başarısız):

- **Canlıya Al** önerilmez ve uyarı gösterilir
- **İptal Et** aksiyonu ön plana çıkar
- Sorun giderildikten sonra tekrar değerlendirilebilir

### Ne Zaman Kullanmalı?

- Kritik servislerde sıfır riskli geçiş istediğinizde
- Canlıya almadan önce kapsamlı test yapmanız gerektiğinde
- Veritabanı migration'ları test etmeniz gerektiğinde

---

## Auto-Promote

Auto-Promote, Blue/Green stratejisinin otomatikleştirilmiş versiyonudur. Preview ortamı hazırlanıp sağlık kontrolleri geçildikten sonra trafik geçişi **otomatik olarak** gerçekleşir — manuel onay gerekmez.

### Nasıl Çalışır?

```
Deploy → Preview Oluştur → Sağlık Kontrolü → Bekleme → Otomatik Promote → Tamamlandı
```

1. Yeni sürüm Preview ortamında oluşturulur
2. Preview pod'ları sağlıklı hale gelir
3. Sistem belirlenen süre boyunca izler
4. Sorun tespit edilmezse **otomatik promote** gerçekleşir
5. Manuel müdahale **gerekmez**

### Dashboard'da Auto-Promote

Auto-Promote aktifken dashboard'da:

- **Topoloji diyagramı:** Blue/Green ile aynı yapı — Active Service + Preview Service → Pod'lar
- **Progress bar:** 3 aşamalı stepper — Creating → Ready → Active
- **Otomatik badge:** "Otomatik Canlıya Al" ve "Tamamlandı" durumu

![Auto-Promote dashboard — otomatik geçiş tamamlanmış](https://raw.githubusercontent.com/Microzon-Tech/docs/main/user-guide/img/deployment-strategies/autopromote-dashboard-healthy.png)

### Aksiyonlar

| Aksiyon | Açıklama |
|---------|----------|
| **İptal Et** | Yalnızca hata durumunda — otomatik geçişi iptal eder |

> Auto-Promote'ta kullanıcı müdahalesi minimumdur. Promote otomatik gerçekleşir; yalnızca preview sağlıksızsa İptal Et ile müdahale edebilirsiniz.

### Ne Zaman Kullanmalı?

- CI/CD sürecinize tam güvendiğinizde
- Manuel onay darboğazını ortadan kaldırmak istediğinizde
- Geliştirme ve staging ortamlarında hızlı iterasyon için

---

## Strateji Değiştirme

Bir servisin rollout stratejisini istediğiniz zaman değiştirebilirsiniz:

1. **Dağıtım Geçmişi ve Geri Alma** sayfasına gidin
2. Sağ üstteki **Strateji** butonuna tıklayın
3. Açılan dialog'da yeni stratejiyi seçin
4. **Change Strategy** ile onaylayın

![Strateji değiştirme dialog'u](https://raw.githubusercontent.com/Microzon-Tech/docs/main/user-guide/img/deployment-strategies/strategy-change-dialog.png)

> **Not:** Strateji değişikliği bir sonraki deploy ile etkili olur. Mevcut çalışan pod'lar etkilenmez; yeni deployment YAML'ları seçilen strateji ile oluşturulur.

---

## Rollout Yönetimi ve Geçmiş

**Dağıtım Geçmişi ve Geri Alma** sayfasında rollout'larınızı yönetebilirsiniz:

- **Durum kartı:** Mevcut rollout durumu, strateji, replica bilgisi
- **Operasyonlar:** Canlıya Al, Geri Al, İptal Et, Yeniden Dene, Duraklat, Resume butonları
- **Geçmiş tablosu:** Tüm revision'lar, imaj bilgisi, trafik yüzdesi, durum ve tarih

![Rollout geçmişi ve operasyonlar](https://raw.githubusercontent.com/Microzon-Tech/docs/main/user-guide/img/deployment-strategies/rollout-history.png)

---

## Rollout Durumları

Deploy sürecinde servisiniz aşağıdaki durumlardan geçer:

| Durum | Görsel | Anlamı |
|-------|--------|--------|
| **Progressing** | Animasyonlu ilerleme | Deploy devam ediyor, yeni pod'lar oluşturuluyor |
| **Paused** | Sarı rozet | Kullanıcı müdahalesi bekleniyor (Canary adımı veya Blue/Green promote) |
| **Healthy / Completed** | Yeşil rozet | Deploy başarıyla tamamlandı, tüm pod'lar sağlıklı |
| **Degraded** | Kırmızı rozet | Bazı pod'lar sağlıksız; müdahale gerekebilir |

Tüm durumlar **gerçek zamanlı** olarak güncellenir — sayfayı yenilemenize gerek yoktur.

---

## Stale Rollout Politikası

Canary veya Blue/Green stratejisinde bir rollout uzun süre **Paused** durumda kalabilir. DevOpsZon, bayatlamış rollout'ları otomatik yönetmek için platform düzeyinde bir politika sunar:

- Platform yöneticisi **maksimum bekleme süresi** tanımlar (ör: 60 dakika)
- Süre aşıldığında:
  - Preview sağlıklıysa → **otomatik Canlıya Al**
  - Preview sağlıksızsa → **otomatik İptal Et**
- Belirli servisler politikadan hariç tutulabilir

> **Not:** Auto-Promote stratejisi zaten kendi otomatik geçişini yönettiğinden stale rollout politikası kapsamı dışındadır.

---

## Strateji Seçim Rehberi

| Soru | Önerilen Strateji |
|------|-------------------|
| Canlı trafikle yeni sürümü test etmek istiyorum | **Canary** |
| Sürümü izole ortamda test edip tek hamlede geçmek istiyorum | **Blue/Green** |
| Manuel onaya gerek duymadan otomatik geçiş istiyorum | **Auto-Promote** |
| Rollback anında gerçekleşmeli | **Blue/Green** veya **Auto-Promote** |
| Trafik yüzdesini kademeli kontrol etmeliyim | **Canary** |
| Preview URL ile QA testi yapmalıyım | **Blue/Green** veya **Auto-Promote** |

---

## Depolama ile Uyumluluk

Canary, Blue/Green ve Auto-Promote stratejilerinin üçü de rollout süresince **hem mevcut (stable) hem de yeni (preview/canary) pod'ları aynı anda ayakta tutar**. Eğer servisinize bir `ProjectRepoServiceStorageAttachment` (PVC) bağlıysa, stable ve preview pod'ları bu volume'u **eşzamanlı olarak mount etmek zorundadır** — aksi halde Kubernetes "Multi-Attach error" ile yeni pod'u Pending'de bırakır ve rollout sonsuza kadar "Yayına Hazırlanıyor" durumunda takılır.

Bu yüzden DevOpsZon pipeline'ı **her PVC'yi `ReadWriteMany` (RWX) olarak oluşturur** — tier, strateji veya replika sayısı ne olursa olsun. Sonuç:

- **Tüm planlar (ücretsiz dahil) her stratejiyi destekler.** Canary çalıştırmak için artık Small plan veya üstüne geçmek zorunda değilsiniz; Free plan ile de Canary / Blue-Green / Auto-Promote yapabilirsiniz.
- **HPA ve çoklu replika** herhangi bir plan üzerinde aktive edilebilir. Platform "her zaman RWX" kuralıyla tek tip davranır.
- **Plan değişikliği veya strategy değişikliği** sırasında Multi-Attach hatası riski ortadan kalkar — tüm volume'lar zaten RWX olduğu için stable ↔ preview geçişi sorunsuz ilerler.

### Mimari not

"Her şey RWX" kararı ve altında yatan Longhorn StorageClass'ları hakkında detay için [Storage RWX Mimarisi](../storage-rwx-architecture.md) dokümanına bakın. RWX volumes Longhorn share-manager üzerinden NFS ile servis edilir; random-I/O yoğun workload'lar (veritabanları gibi) için performans notlarını da aynı dokümanda bulabilirsiniz.

### Mevcut RWO servisleri

"Always RWX" refactor'undan önce oluşturulmuş servisler hâlâ `ReadWriteOnce` (RWO) volume kullanıyor. Kubernetes, bir PVC'nin access mode'unu immutable kabul ettiği için bu servislerin bir sonraki deploy'u platform tarafından durdurulacak ve "StorageAccessMode uyumsuzluğu" hatası alacaksınız.

Çözüm: **Servisi silip yeniden oluşturun.** Yeni servis otomatik olarak RWX PVC alır. Veri migrasyonu gerekiyorsa snapshot alıp yeni servise manuel geri yükleyin.
