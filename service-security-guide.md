# Servis Güvenliği Çalışma Alanı (Service Security)

Her servisin detay sayfasındaki **Security** sekmesi, o servise özel güvenlik çalışma alanıdır: koruma modunu yönetir, çalışma zamanı gözlemlerini inceler, sapmaları (drift) yakalar ve servis bazlı bulguları tek yerden takip edersiniz.

Tenant genelindeki görünüm için bkz. [Güvenlik Merkezi](security-center-guide.md).

---

## Koruma Modları

Her servis bir **çalışma-zamanı koruma modunda** çalışır:

| Mod | Davranış | Ne zaman kullanılır? |
|-----|----------|----------------------|
| **Kapalı (Off)** | Koruma katmanı pasif | Önerilmez; yalnızca özel durumlar |
| **Gölge (Shadow)** | Politika hazırlanır ama uygulanmaz | Geçiş hazırlığı |
| **Denetim (Audit)** | Davranışlar **izlenir ve kaydedilir**, hiçbir şey engellenmez | Yeni servis devreye alma, baseline öğrenme |
| **Engelleme (Block)** | Politika dışı davranış **kernel seviyesinde reddedilir** | Üretim ortamı hedef modu |

Önerilen yaşam döngüsü:

```
Yeni servis → Audit modunda gözlem → Gözlemleri işle → Block'a yükselt
```

Audit modunda servisinizin gerçek davranışı öğrenilir; meşru davranışları onayladıktan sonra **Block'a Yükselt** dediğinizde koruma, uygulamanızı bozmadan devreye girer.

---

## Üst Bilgi Şeridi (Güvenlik Özeti)

Security sekmesinin her görünümünde üstte sabit duran özet şerit şunları gösterir:

- **Mod rozeti** — ör. *"Runtime Protection: Audit"*. Üzerine gelince modun tam açıklamasını görürsünüz (Audit: "izler, kaydeder, engellemez"; Block: "politika dışı işlem kernel'de reddedilir").
- **Sapma (Drift) rozeti** — çalışan pod'un güvenlik duruşu, tanımlı baseline'dan ayrıştıysa kırmızı rozet belirir. Tıklayınca ayrıntıya gidersiniz.
- **Üç canlı istatistik** — Bekleyen gözlem sayısı · Engellenen işlem sayısı · Son 1 saatteki Critical bulgu sayısı.
- **Sonraki adım önerisi** — bulunduğunuz duruma göre tek tıklık yönlendirme: *"Gözlemleri incele"*, *"Duruşu gözden geçir"* veya *"Block'a yükselt"*.

### "Canlı posture baseline'dan saptı" uyarısı ne demek?

Platform, servisinizin **çalışan halini** (canlı pod'lar) tanımlı güvenlik baseline'ı ile sürekli karşılaştırır. Tespit edilen sapma türleri:

| Sapma | Anlamı | Yapılacak |
|-------|--------|-----------|
| Baseline imza uyuşmazlığı | Çalışan yapılandırma, beklenen baseline sürümünden farklı | Servisi yeniden dağıtın |
| Root ile çalışıyor | Servis root kullanıcıyla çalışıyor ama bunun için tanımlı bir muafiyet yok | İmajı non-root yapın veya muafiyet tanımlayın |
| Ağ politikası eksik | Beklenen ağ politikası kümede bulunamadı | Yeniden dağıtım / destek |
| Koruma politikası eksik | Çalışma-zamanı koruma politikası kümede bulunamadı | Yeniden dağıtım / destek |
| Hâlâ Audit modunda | Baseline Block istiyor ama politika Audit'te kalmış | "Block'a Yükselt" akışını tamamlayın |
| Küme sorgusu başarısız | Kümeye anlık erişilemedi (geçici olabilir) | Küme bağlantısını kontrol edin |

---

## Sekmeler

### Genel Bakış

Servisin güvenlik durumunun özeti:

- **Kritik bulgular** beslemesi (Critical + High)
- **En çok düşürülen trafik** — son saatte ağ politikalarının engellediği trafiğin özeti (tıklayınca Ağ sekmesine gider)
- **Aktif ağ olayları** — devam eden ağ güvenlik olayları
- Her şey yolundaysa moda duyarlı "temiz" durum satırı görünür

> Servis bir kümeye bağlı değilse bu sekme sizi önce dağıtım yapmaya yönlendirir.

### Ağ

Servisin ağ güvenliği görünümü, alt sekmelerle:

- **Akışlar (Flows)** — servisin gerçek ağ trafiği akışı
- **Düşürülenler (Drops)** — ağ politikasının engellediği bağlantılar (kim, nereye, hangi port)
- **Olaylar (Incidents)** — şüpheli ağ etkinliklerinden üretilen olay kayıtları
- **Politikalar** — servise uygulanan ağ politikası kuralları (gelen/giden kural sayıları)

> Uygulama katmanı (L7 — HTTP yolu/metodu seviyesinde) ayrıntılı trafik görünürlüğü **Service Insights** aboneliğinin parçasıdır; bkz. [Faturalandırma ve Planlar](billing-plans.md).

### Politikalar

Servisin güvenlik duruşunu yönettiğiniz sekme. Üç ana kart içerir:

**1. Baseline Duruşu** — canlı duruş + sapma özeti. Sapma varsa sekme başlığında kırmızı nokta görünür.

**2. Linux Yetenekleri (Capabilities)** — Servisleriniz varsayılan olarak en sıkı yetki setiyle (tüm yetenekler düşürülmüş) çalışır. Platform, yaygın uygulamaların sorunsuz başlaması için küçük ve güvenli bir varsayılan set tanımlar:

| Yetenek | Tipik ihtiyaç |
|---------|---------------|
| `CHOWN` | Başlangıçta dosya sahipliği düzenleme (ör. web sunucuları, cache dizini hazırlığı) |
| `NET_BIND_SERVICE` | 80/443 gibi ayrıcalıklı portlara bağlanma |
| `FSETID` / `FOWNER` | Dosya izin yönetimi |
| `KILL` | Process sinyalleme |

- Bu seti **daraltabilirsiniz** (sıkılaştırma her zaman serbesttir) veya listeden kontrollü ekleme yapabilirsiniz.
- Ekleme yaparken onay istenir, **gerekçe zorunludur** ve değişiklik değiştirilemez denetim zincirine kaydedilir.
- Listede olmayan (platformun güvenli bulmadığı) bir yetenek eklenemez.

**3. Baseline Geçersiz Kılmaları (Overrides)** — read-only kök dosya sistemi altında ince ayar:

- **Ekstra yazılabilir dizinler**: Kök dosya sistemi salt-okunur olduğunda uygulamanızın yazması gereken yolları tanımlayın (ör. `/app/uploads`). Hassas sistem yolları (`/etc`, `/proc`, `/sys`, servis hesabı kimlik dizini) platform tarafından **reddedilir**.
- **Framework otomatik yolları**: Platform, tespit edilen framework'e göre gerekli yazma yollarını otomatik ekler (ör. .NET için `/app/App_Data`) — bunlar salt-okunur rozetle görünür, sizin eklemenize gerek yoktur.
- **Framework anahtar kalıcılığı**: Varsayılan olarak framework yazma alanı geçicidir (pod yeniden başlayınca silinir). Kalıcılık anahtarını açarsanız bu alan kalıcı ve replikalar arası paylaşımlı depolamaya taşınır — ör. .NET oturum/anti-forgery anahtarlarının yeniden başlatmalarda korunması için.
- **Ek engelli yollar / engelli çalıştırma yolları**: Baseline'ın üzerine kendi yasaklarınızı ekleyin.
- **YAML önizleme**: Uygulanacak politikanın tam içeriğini değişiklik öncesi görüntüleyin.

### Gözlemler

Servis **Audit modundayken** sensörün yakaladığı tüm davranışların kuyruğu: hangi process, hangi dosya/ağ işlemi, kaç kez.

Çalışma akışı:

1. Servisi Audit modunda normal yükü altında çalıştırın (önerilen: birkaç gün, tüm iş senaryoları çalışsın).
2. Gözlemleri tek tek işleyin: **İzin Ver** (meşru davranış) / **Engelle** (istenmeyen) / **Yoksay**.
3. Bekleyen gözlem kalmayınca üst şeritteki **Block'a Yükselt** adımıyla korumayı etkinleştirin.

Sekme başlığındaki rozet, bekleyen gözlem sayısını gösterir.

### Bulgular

Bu servise ait güvenlik bulguları (tenant genel listesinin servise filtrelenmiş hali) + **kanıt zaman çizelgesi**: son 7 güne ait, bulgu ve kararlarla ilişkili olay geçmişi. Bulgu kararlarının ayrıntısı için bkz. [Güvenlik Merkezi → Bulgular](security-center-guide.md#bulgular-findings).

---

## Pratik Akışlar

### Yeni bir servisi güvenli devreye alma

1. Servisi dağıtın — koruma **Audit** modunda başlar, hiçbir şey engellenmez.
2. Birkaç gün normal trafikle çalıştırın; tüm iş akışlarının (cron, rapor, yedekleme vb.) en az bir kez koştuğundan emin olun.
3. **Gözlemler** sekmesinde birikenleri işleyin: meşru olanlara İzin Ver.
4. Gerekliyse **Politikalar** sekmesinden yazılabilir dizin/yetenek ekleyin.
5. Üst şeritten **Block'a Yükselt** — artık politika dışı her davranış engellenir ve bulgu üretir.

### Pod "Permission denied" hatası veriyor / CrashLoopBackOff oldu

Muhtemelen meşru bir davranış Block modunda engellendi:

1. **Bulgular** sekmesinde son Engellendi kayıtlarına bakın — hangi process, hangi yol?
2. Yazma hatası ise: **Politikalar → Ekstra yazılabilir dizinler**'e ilgili yolu ekleyin.
3. Yetki hatası ise: **Politikalar → Linux Yetenekleri**'nden gerekli yeteneği ekleyin.
4. Davranış politikaya takılıyorsa: bulguda **Allow** kararı verin veya süreli bir [politika istisnası](security-center-guide.md#politika-istisnalar%C4%B1-policy-exceptions) tanımlayın.
5. Sorunu izole edemiyorsanız servisi geçici olarak **Audit** moduna alın, gözlem toplayıp tekrar Block'a yükseltin.

### Şüpheli bir bulgu gördüm

1. Bulgunun detayını açın — process, yol, tekrar sayısı, ilk/son görülme.
2. Eşleşen [olay müdahale playbook'unu](security-center-guide.md#olay-m%C3%BCdahale-playbooklar%C4%B1-ir-playbooks) izleyin.
3. Kanıt zaman çizelgesinden olayın bağlamını (öncesi/sonrası) inceleyin.
4. Kararınızı verin (Block/Acknowledge) — karar gerekçenizle birlikte denetim zincirine işlenir.

---

## İzinler

| İşlem | Gerekli yetki |
|-------|---------------|
| Güvenlik sekmesini görüntüleme | Güvenlik duruşu görüntüleme |
| Gözlem işleme, Block'a yükseltme | Güvenlik baseline yönetimi |
| Linux yeteneği ekleme/çıkarma | Yetenek yönetimi |
| Yazılabilir dizin yönetimi | Yazılabilir dizin yönetimi |
| Servis bulgularında karar | Bulgu yönetimi |

Yetkiler **Erişim Kontrolü → Roller** ekranından atanır; bkz. [Erişim Kontrolü](access-control-guide.md).

---

## İlgili Dokümanlar

- [Güvenlik Merkezi](security-center-guide.md) — tenant geneli güvenlik konsolu
- [Çalışma Zamanı Koruması](runtime-security-guide.md) — koruma modları ve politika kavramları
- [Servis Yönetimi](service-guide.md) — servis yaşam döngüsü
