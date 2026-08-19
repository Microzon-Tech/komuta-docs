# Çalışma Zamanı Koruması (Runtime Protection)

Komuta, uygulamalarınızı **çalışma zamanı saldırılarına** karşı korumak için kernel seviyesinde çalışan bir koruma katmanı sunar: **Çalışma-zamanı Koruması (Runtime Protection)**. Bu katman, container'larınızın *içindeki* davranışları — hangi process'in çalıştığını, hangi dosyaya erişildiğini, hangi ağ bağlantısının açıldığını — gözlemler ve politika dışı davranışları engeller.

Bu doküman koruma katmanının kavramlarını anlatır. Ekran kullanımı için bkz. [Servis Güvenliği Çalışma Alanı](service-security-guide.md) ve [Güvenlik Merkezi](security-center-guide.md).

---

## Koruma Katmanları

Komuta'da güvenlik birden fazla katmanda uygulanır; her katman farklı bir saldırı yüzeyini kapatır:

| Katman | Kapsam |
|--------|--------|
| **Ağ Politikası** | Servisler arası trafik, DNS — *pod'lar arasındaki* iletişimi kısıtlar |
| **L7 Hız Limiti** | HTTP istek hızı sınırlama |
| **Çalışma-zamanı Koruması** | *Container içindeki* process / dosya / yetenek davranışları |
| **Host Koruması** | Node'un kendisindeki kritik dosya ve process'ler |
| **Çalışma-zamanı İzleme** | Gelişmiş davranış telemetrisi (process soy ağacı, sistem çağrıları) |

Çalışma-zamanı Koruması şu davranışları engelleyebilir:

- **Process çalıştırma**: `apt-get`, `curl`, `nmap` gibi binary'lerin container içinde çalıştırılması
- **Dosya erişimi**: hassas dosyaların okunması/yazılması, servis hesabı kimlik dosyasının salt-okunur kılınması
- **Ağ erişimi**: hangi process'lerin ağ bağlantısı açabileceği
- **Linux yetenekleri**: tehlikeli kernel yetkilerinin (raw soket, sistem yönetimi vb.) kullanımı

---

## Otomatik Kurulum ve Platform Baseline'ları

Koruma katmanı, Komuta tarafından yönetilen kümelere **otomatik olarak** kurulur — sizin bir şey yapmanız gerekmez:

1. Küme kurulumunda koruma bileşenleri **ilk eklenti** olarak yüklenir (güvenlik önceliklidir).
2. **Host baseline politikası** node'ların kritik dizinlerini (küme sertifikaları, kimlik dosyaları, kullanıcı yönetim araçları) koruma altına alır.
3. **Küme baseline politikası** kullanıcı workload'larında bilinen saldırı araçlarını (paket yöneticileri, ağ tarama araçları) engeller.
4. CNI, depolama, gözlemlenebilirlik ve GitOps gibi **platform sistem bileşenlerinin çalıştığı alanlar otomatik hariç tutulur** — kritik altyapının koruma katmanıyla çakışması saha deneyimiyle önlenmiştir.

### Servis Baseline Politikası

Dağıtım hattı, her servis için otomatik bir **çalışma-zamanı baseline politikası** üretir. Bu politika:

- Paket yöneticileri ve saldırı araçlarını (`apt`, `yum`, `wget`, `nc`, `nmap` vb.) engeller
- `curl` / `wget`'i **akıllıca** ele alır: sağlık probe'larınızda kullanılıyorsa izin verir, kullanılmıyorsa engeller
- Tehlikeli Linux yeteneklerini (raw soket, sistem yönetimi, kernel modülü, process izleme) engeller
- Servis hesabı kimlik dosyasını **salt-okunur** yapar

Baseline politikaları **platform tarafından yönetilir** ve elle düzenlemeye karşı korumalıdır. Servis bazlı ince ayar (yetenek ekleme, yazılabilir dizin, ek yasaklar) için servisin [Güvenlik → Politikalar](service-security-guide.md#politikalar) sekmesini kullanın.

---

## Politika Aksiyonları

### Block (Önerilen — Üretim Hedefi)

Politika dışı işlem **kernel seviyesinde reddedilir**: sistem çağrısı başarısız olur, process binary'yi çalıştıramaz. Saldırgan container içine girse bile işlem asla gerçekleşmez.

```
$ apt-get update
bash: /usr/bin/apt-get: Permission denied
```

### Audit (Gözlem Modu)

İhlal **kaydedilir ama engellenmez** — uygulama normal çalışmaya devam eder. Yeni bir servisi üretime almadan önce gerçek davranışı öğrenmek için kullanılır. Audit modunda biriken gözlemler, [Gözlemler sekmesinde](service-security-guide.md#g%C3%B6zlemler) işlenir ve baseline'a dönüştürülür.

### Allow (İleri Düzey — Beyaz Liste)

Yalnızca açıkça izin verilen işlemler çalışır; geri kalan her şey reddedilir. Maksimum güvenlik sağlar ancak kapsamlı test gerektirir.

> Önerilen yol: **Audit ile başla → gözlemleri işle → Block'a yükselt.** Ayrıntılı akış için bkz. [Servis Güvenliği → Pratik Akışlar](service-security-guide.md#pratik-ak%C4%B1%C5%9Flar).

---

## İhlal İzleme

Koruma katmanının yakaladığı her olay şu yüzeylere akar:

- **Güvenlik Merkezi → İhlaller**: tenant genelindeki canlı ihlal akışı
- **Güvenlik Merkezi → Bulgular**: tekilleştirilmiş, karar alınabilir kayıtlar
- **Servis detayı → Security**: servise özel bulgular ve canlı istatistikler (üst şeritte son 1 saatin Engellenen / Critical sayıları)
- **Üst çubuk rozeti**: Critical + High ihlal sayacı her ekranda görünür; kritik artışta anlık bildirim düşer
- **Uyarı kuralları**: küme kurulumuyla birlikte yüksek ihlal oranı, kritik engelleme, host ihlali ve sensör sağlığı için hazır uyarı kuralları tanımlanır; bunlar [bildirim kanallarınıza](notification-guide.md) yönlendirilir

---

## Politika Önerileri ve İstisnalar

- **Önerilen Politikalar**: Platform, gözlenen davranışlardan otomatik politika önerileri üretir ("bu serviste şu erişim 47 kez engellendi — kalıcı kural önerilir"). Önerileri inceleyip tek tıkla uygularsınız.
- **Politika İstisnaları**: Meşru ama politikaya takılan bir iş akışı için **en fazla 90 günlük**, gerekçeli ve onay akışlı istisna tanımlayabilirsiniz.

Her ikisi de [Güvenlik Merkezi → Politikalar](security-center-guide.md#politikalar) altında yönetilir ve tüm geçişler değiştirilemez denetim zincirine kaydedilir.

---

## Sorun Giderme

### Pod "CrashLoopBackOff" oldu ve "Permission denied" hatası var

Muhtemelen uygulamanızın meşru bir davranışı Block modunda engellendi:

1. Servisin **Güvenlik → Bulgular** sekmesinde son Engellendi kayıtlarını inceleyin — hangi process, hangi yol?
2. Yazma engeli ise **ekstra yazılabilir dizin** ekleyin; yetki engeli ise **Linux yeteneği** ekleyin (Politikalar sekmesi).
3. Karmaşık durumda servisi geçici **Audit** moduna alın, gözlem toplayın, sonra tekrar **Block'a yükseltin**.

### "Yüksek ihlal oranı" uyarısı geldi

1. **Güvenlik Merkezi → İhlaller**'de ilgili servisi filtreleyin.
2. Hangi process/politikanın tetiklediğini bulun.
3. Meşru kullanım ise: bulguda **Allow** kararı verin veya politika istisnası tanımlayın.
4. Şüpheli ise: eşleşen [olay müdahale playbook'unu](security-center-guide.md#olay-m%C3%BCdahale-playbooklar%C4%B1-ir-playbooks) izleyin.

### Yeni politika kümeye uygulanmadı

- Politikanın **etkin** olduğundan emin olun (devre dışı politikalar dağıtılmaz).
- Politika listesindeki dağıtım durumu alanını kontrol edin; hata varsa ayrıntı orada görünür.
- Küme bağlantısı sorunu görünüyorsa küme durumunu **Güvenlik Merkezi → Genel Bakış → Küme Güvenlik Matrisi**'nden doğrulayın.

---

## İlgili Dokümanlar

- [Güvenlik Merkezi](security-center-guide.md) — bulgular, politikalar, uyumluluk, SIEM, tatbikatlar
- [Servis Güvenliği Çalışma Alanı](service-security-guide.md) — modlar, gözlemler, yetenekler, yazılabilir dizinler
- [Bildirim Yönetimi](notification-guide.md) — uyarıların kanallara yönlendirilmesi
- [Erişim Kontrolü](access-control-guide.md) — güvenlik izinleri
