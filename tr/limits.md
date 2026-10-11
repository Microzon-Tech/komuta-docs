# Sınırlar ve Kotalar

Bu sayfa Komuta'da karşılaşabileceğiniz sınırları tek yerde toplar: hesap ve organizasyon, harcama limiti, ücretsiz plan, servisler, build'ler, Job'lar, Stack'ler, domain'ler, yönetilen servisler, API Gateway, loglar ve alarmlar. Her satırda sınırın değeri ve varsa nasıl aşılacağı yazar; özelliğin nasıl kullanıldığı bağlantı verilen sayfada anlatılır.

> **Not:** Sınırlar zamanla değişebilir. Organizasyonunuza uygulanan güncel değeri konsolda görürsünüz: harcama limiti ve seviyeniz **Hesap → Cüzdan** sayfasında, paketler ve planlar oluşturma sihirbazlarında, ücretsiz servisin kalan saatleri servis panosundaki **Ücretsiz saatler** alanında.

---

## Hesap ve Organizasyon

| Sınır | Değer | Not |
|---|---|---|
| Hesap başına organizasyon | En fazla 5 | Kendi açtığınız ve davetle katıldığınız organizasyonlar birlikte sayılır. Sınıra ulaşınca yeni organizasyon açılamaz, davet de kabul edilemez. |
| Organizasyon adı | En fazla 64 karakter | Yeniden adlandırırken en az 2 karakter. |
| Davetin geçerliliği | 72 saat | Daveti yeniden gönderdiğinizde yeni bir 72 saat başlar. Aynı adrese yeni davet gönderirseniz önceki geçersiz olur. |
| API anahtarı adı | En fazla 128 karakter | **Hesap → API Anahtarları** |
| API anahtarının ömrü | 30 gün, 90 gün ya da 1 yıl; en fazla 365 gün | Bitiş tarihi verilmeyen anahtar 90 gün sonra sona erer; konsoldaki **Sona erme yok** seçeneği de bu kurala tabidir. |

Ekip arkadaşlarını davet etmek ve rolleri için bkz. [Organizasyon ve Ekip](organizations-team.md).

---

## Harcama Limiti ve Faturalama

Her organizasyonun aylık bir kaynak tahmini limiti vardır. Limit yalnızca yeni bir şey oluştururken ya da planını büyütürken kontrol edilir: aylık tahmin limiti aşacaksa işlem yapılmaz, zaten çalışan kullanım ise durdurulmaz. Limiti kendiniz ayarlayamazsınız; seviyeniz yükseldikçe limit otomatik artar.

| Seviye | Aylık limit | Nasıl açılır |
|---|---|---|
| Starter | 20 USD | Yeni organizasyonun başlangıç seviyesi |
| Verified | 50 USD | Doğrulanmış bir kart |
| Basic | 100 USD | Toplam 10 USD bakiye yüklemesi ya da 10 USD ödenmiş ekstre |
| Pro | 300 USD | Toplam 50 USD bakiye yüklemesi ya da 50 USD ödenmiş ekstre |
| Scale | 1.000 USD | Toplam 300 USD bakiye yüklemesi ya da 300 USD ödenmiş ekstre |

Seviye, bu koşullardan karşıladığınız en yükseğine göre hesaplanır. Scale'in üzerinde bir limite ya da seviyenizin sağladığından fazlasına ihtiyacınız varsa **Hesap → Cüzdan** sayfasındaki **Limit artışı talep et** düğmesini kullanın ya da [destek talebi](support-tickets.md) açın.

| Sınır | Değer | Not |
|---|---|---|
| Tek seferde bakiye yükleme | 5–1.000 USD | Daha büyük tutarları birkaç yüklemeye bölün. |
| Hoş geldin kredisi | 10 USD, 30 gün geçerli | Yalnızca ilk organizasyona yüklenir; süresi dolunca kullanılmayan kısım düşer. |
| Fatura dönemi | Ayın 6'sından 6'sına (UTC) | Ekstre her ayın 6'sında kesilir. |
| En küçük ekstre | 0,50 USD | Bu tutarın altındaki dönem için ekstre kesilmez; kullanım bir sonraki döneme devreder. |
| Ödenmemiş fatura | 14 gün ödemesiz dönem, ardından askıya alma | Ödemesiz dönemde ölçeklendirme, plan yükseltme ve yönetilen servis planı değişikliği durur; konsoldaki uyarı askıya alma tarihini gösterir. Askıdaki hesapta değişiklik yapılamaz; görüntüleme ve ödeme açık kalır. |

Fiyatlar ve faturalama ayrıntıları için bkz. [Faturalama](billing.md) ve [Fiyatlandırma](https://www.komuta.io/pricing).

---

## Ücretsiz Plan

| Sınır | Değer | Not |
|---|---|---|
| Organizasyon başına ücretsiz servis | 1 | Bazı kuponlar ek ücretsiz servis hakkı verir. |
| Ücretsiz paket | `dz-nano`: 0,1 vCPU, 256 MB RAM | Daha fazla kaynak ücretli paketlerle alınır. |
| Uyanık kalma havuzu | Takvim ayı (UTC) başına toplam 450 saat | Organizasyonun ücretsiz servisleri aynı havuzu paylaşır. Havuz bitince ücretsiz servisler ayın 1'ine kadar uyur; bu sürede uyandırılamaz ve deploy edilemez. |
| Boşta kalma süresi | 30 dakika | Bu süre boyunca trafik almayan ücretsiz servis uyur, ilk istekle uyanır. Uyanma yaklaşık bir dakika sürebilir. |
| Replika | Tek replika, otomatik ölçekleme yok | — |
| Git hesabı | Organizasyona bağlı kişisel bir GitHub, GitLab ya da Azure DevOps hesabı | Ücretsiz plandan serbest bırakılan hesap en az 30 gün ve en az o ayın sonuna kadar başka bir organizasyonun ücretsiz planında kullanılamaz. |

Bir ücretsiz servisi havuzun dışında tutmak için **Her zaman açık** eklentisini açabilirsiniz: servis hiç uyumaz ve ücretsiz saatleri kullanmaz. Ücreti fatura dönemi başına 3 USD + vergidir ve açık kaldığı süreye göre hesaplanır; doğrulanmış bir kart, tek replika ve ücretsiz plana bağlı bir Git hesabı gerekir. Ayrıntılar: [Servis Uyku Modu](service-sleep.md).

---

## Servisler

| Sınır | Değer | Not |
|---|---|---|
| Servis adı | 1–53 karakter | Küçük harf, rakam ve tire; harfle başlar, harf ya da rakamla biter. Aynı repoda benzersiz olmalıdır. |
| Proje adı | 3–100 karakter | Açıklama da zorunludur: 3–500 karakter. |
| Otomatik ölçekleme: replika | En az ve en fazla replika 1–100 | İkisini eşit verirseniz servis hep o sayıda replikayla çalışır. |
| Otomatik ölçekleme: hedef | CPU ve bellek hedefi %1–100 | En az bir hedef açık olmalıdır. |
| Otomatik ölçekleme: pencere | 0–3600 saniye | Dakikada eklenecek ya da kaldırılacak pod sayısı 1–1000. |
| Port | 1–65535 | Port, servis portu ve hedef port için aynı aralık. |
| Ortam değişkeni anahtarı | Harf, rakam, `-`, `_` ve `.` | Boşluk ve başka özel karakter kullanılamaz. |
| Toplu içe aktarma | Tek seferde en fazla 500 değişken | Daha fazlasını birkaç seferde aktarın. |
| Ortam grubu | Grup başına en fazla 500 değişken | Grup adı en fazla 128, anahtar en fazla 256 karakter. |
| Kalıcı depolama planları | Free 1 GB, Small 10 GB, Medium 50 GB, Large 200 GB, XLarge 1024 GB | Free planla servis başına tek depolama eklenir. Aynı serviste ücretsiz ve ücretli depolama birlikte kullanılamaz. |
| Depolama adı ve yolu | Volume adı en fazla 63 karakter; bağlama yolu 2–256 karakter | Aynı serviste aynı ad ya da aynı yol ikinci kez kullanılamaz. |

### Kaynak paketleri

| Paket | vCPU | RAM | Geçici disk | Aylık |
|---|---|---|---|---|
| `dz-nano` | 0,1 | 256 MB | 512 MB | Ücretsiz plan |
| `dz-dev` | 0,5 | 1 GB | 1 GB | 9 USD |
| `dz-standard` | 1 | 2 GB | 2 GB | 18 USD |
| `dz-premium` | 2 | 4 GB | 3 GB | 36 USD |
| `dz-enterprise` | 4 | 8 GB | 4 GB | 72 USD |

Fiyatlar vergi hariçtir; kullanım saatlik ölçülür ve aylık tutar saatlik ücretin 720 katıdır. Güncel fiyatlar [Fiyatlandırma](https://www.komuta.io/pricing) sayfasındadır.

İlgili sayfalar: [Otomatik Ölçekleme](services-autoscaling.md), [Portlar](services-ports.md), [Ortam Değişkenleri](services-environment-variables.md), [Ortam Grupları](environment-groups.md), [Depolama](storage.md).

---

## Ağ Trafiği

Her servisin bir bant genişliği planı vardır. Plan, servisin her pod'unun giden (çıkış) trafik hızını sınırlar; gelen trafik sınırsızdır ve faturalanmaz.

| Plan | Aylık | Pod başına çıkış hızı |
|---|---|---|
| Starter (varsayılan) | Ücretsiz | 5 Mbit/s |
| Growth | 4,99 USD | 25 Mbit/s |
| Business | 14,99 USD | 100 Mbit/s |
| Enterprise | 49,99 USD | 500 Mbit/s |
| Scale | 149,99 USD | 2 Gbit/s |

Ayrıntılar: [Bant Genişliği Planları](bandwidth-plans.md).

---

## Build ve Deploy

Aynı anda kaç build çalışacağı ve kaç build'in kuyrukta bekleyebileceği, harcama limiti seviyenize bağlıdır:

| Seviye | Eşzamanlı build | Kuyrukta bekleyebilecek build |
|---|---|---|
| Starter | 1 | 5 |
| Verified | 1 | 10 |
| Basic | 2 | 20 |
| Pro | 4 | 30 |
| Scale | 8 | 50 |

| Sınır | Değer | Not |
|---|---|---|
| Build süresi | En fazla 1 saat | Bu süreyi aşan build zaman aşımıyla biter. |
| Kuyrukta bekleme | En fazla 6 saat | Bu sürede başlayamayan build zaman aşımına düşer. |
| Dahil build dakikası | Her fatura döneminde 2.000 dakika | Aşan dakika başına 0,008 USD. Kuyrukta beklenen süre sayılmaz. |
| Otomatik deploy bekleme aralığı | 0–3600 saniye, varsayılan 30 saniye | İki otomatik tetikleme arasındaki en kısa süre. |
| CI deploy token'ı | Servis başına en fazla 10 etkin token | Ömür varsayılan 365 gün, en fazla 730 gün. |
| CI deploy istekleri | Token başına dakikada 30 deploy ve 300 durum sorgusu; servis başına saatte 60 deploy | Aşılınca `429` döner; `Retry-After` kadar bekleyin. |
| CI deploy kayıtları | 180 gün | Daha eski kayıtların durum adresi `404` döner. |

İlgili sayfalar: [Build Kuyruğu](build-queue.md), [Pipeline'lar](service-pipeline-guide.md), [Otomatik Deploy](service-auto-deploy.md), [CI'dan Deploy](ci-deploy.md).

---

## Job ve CronJob

| Sınır | Değer | Not |
|---|---|---|
| Betik boyutu | En fazla 64 KB (65.536 bayt) | Daha büyük işleri repodan build edilen bir kaynakla çalıştırın. |
| Zamanlama | Beş alanlı cron ifadesi ya da `@hourly`, `@daily`, `@weekly`, `@monthly`, `@yearly` | En sık dakikada bir. Saniye alanı, `@every` ve ifadenin başına `CRON_TZ=` yazmak kabul edilmez; hiç gerçekleşmeyecek bir tarih (ör. `0 0 30 2 *`) reddedilir. |
| Saat dilimi | IANA adı (ör. `Europe/Istanbul`) ya da `UTC` | — |

Ayrıntılar: [Job ve CronJob'lar](jobs-cronjobs.md).

---

## Stack'ler

| Sınır | Değer | Not |
|---|---|---|
| Manifest sürümü | Yalnızca `version: 1` | — |
| Manifest boyutu | En fazla 1 MiB, tek bir YAML belgesi | — |
| İç içe derinlik | En fazla 32 seviye | — |
| Kaynak sayısı | Manifest başına en fazla 200 | Servisler, oluşturulan ve kullanılan mevcut yönetilen servisler, domain'ler ve secret'lar birlikte sayılır. |
| Ad uzunluğu | En fazla 160 karakter | — |
| `apply.maxParallel` | 1–10 | — |
| Planın geçerliliği | 30 dakika | Süresi dolan plan uygulanamaz; yeni plan oluşturun. |
| Güvenli secret bağlantısı | 15 dakika, tek kullanımlık | Yalnızca bağlantıyı oluşturan kişi açabilir. |

Ayrıntılar: [Stack'ler](stacks.md), [Stack Manifest Başvurusu](stack-manifest-reference.md).

---

## Domain'ler

| Sınır | Değer | Not |
|---|---|---|
| Yeni özel domain ekleme | Organizasyon başına günde en fazla 10 | Sayaç her gün 00:00 UTC'de sıfırlanır. |

Ayrıntılar: [Özel Domain'ler](custom-domains.md).

---

## Erişim Koruması

Erişim korumasının sınırları (IP listesi, yol kuralları, paylaşımlar, servis token'ları, oturum süreleri, kod gönderim hızları, erişim kaydının saklanması) ayrı bir tabloda toplanmıştır: [Erişim Koruması: Başvuru → Sınırlar](access-protection-reference.md#sınırlar).

---

## Yönetilen Servisler

PostgreSQL, Valkey ve RabbitMQ'da CPU, bellek, disk ve sunucu sayısını plan belirler; planlar ve güncel boyutları oluşturma sihirbazında listelenir. Plana bağlı bağlantı, bellek ve sorgu sınırları servisin kendi sayfasında anlatılır.

| Sınır | Değer | Not |
|---|---|---|
| Yedek saklama süresi | 2–365 gün, varsayılan 7 gün | Üç serviste aynı. Süreyi kısaltmak geri alınamaz: yeni pencerenin dışında kalan yedekler silinir. |
| Planlı silme | 7 gün | Süre dolmadan iptal edilebilir; bu sürede servis çalışmaya ve faturalanmaya devam eder. |

Ayrıntılar: [PostgreSQL](managed-postgresql.md), [Valkey](managed-valkey.md), [RabbitMQ](managed-rabbitmq.md).

---

## API Gateway

| Sınır | Değer | Not |
|---|---|---|
| API sayısı | Gateway başına en fazla 40 | — |
| Endpoint sayısı | Gateway başına en fazla 300 | Bütün API'ler aynı sınırı paylaşır. |
| API anahtarı | Gateway başına en fazla 1.000 | İptal edilmemiş anahtarlar sayılır; süresi dolanlar da dahil. |
| **En büyük istek gövdesi (MiB)** | 1–100, varsayılan 10 | — |
| Zaman aşımı | En fazla 300 saniye, varsayılan 10 saniye | — |
| Hız limiti | Anahtar başına saniyede istek; endpoint formunda 0–10.000 (0 = sınırsız) | Her replika ayrı sayar; sayaç kendi Valkey örneğinizde tutulursa bütün replikalar tek bütçeyi paylaşır. Bir çağırana kota ve hız limiti birlikte verilemez. |
| Önbellek süresi | En az 1 saniye, varsayılan 60 saniye | Endpoint formunda en fazla 3600 saniye. |
| **Önbelleğe alınacak en büyük yanıt (KiB)** | 64–32.768, varsayılan 512 | Gateway'in önbellek alanının dörtte birini geçemez. |
| Replika | 1–10 | Hız limiti sayacı kendi Valkey örneğinizde tutuluyorsa 20'ye kadar. |

Ayrıntılar: [API Gateway](api-gateway.md).

---

## Loglar

| Sınır | Değer | Not |
|---|---|---|
| Tek sorgunun zaman aralığı | En fazla 12 saat | Daha eskiye **Daha eski logları yükle** ile gidin. |
| Uygulamalar arası arama | Tek aramada en fazla 25 uygulama | Daha fazlası için uygulama seçiciyle kapsamı daraltın. |

Ayrıntılar: [Servis Logları](service-logs.md), [Gözlemlenebilirlik](observability.md).

---

## Alarmlar

| Sınır | Değer | Not |
|---|---|---|
| Kural adı | En fazla 128 karakter | Konsoldaki kural formunda. |
| Özet | En fazla 256 karakter | — |
| Sorgu | En fazla 2.000 karakter | — |
| Bildirim tekrar aralığı | En az 5 dakika | Boş bırakılırsa bildirimler kapanmaz. |

Ayrıntılar: [Alarm Kuralları](alerts-rules.md).

---

## Destek Talepleri

| Sınır | Değer | Not |
|---|---|---|
| Konu | 3–256 karakter | — |
| Mesaj ve yanıt | 3–4000 karakter | — |

---

## Sınıra Takıldığınızda

- **Harcama limiti:** Kart doğruladıkça, bakiye yükledikçe ve ekstre ödedikçe seviyeniz kendiliğinden yükselir. Daha fazlası için **Hesap → Cüzdan** sayfasında **Limit artışı talep et**'i kullanın.
- **Ücretsiz servis:** Ek ücretsiz servis hakkı veren bir kuponunuz varsa kullanın; diğer servisleri ücretli paketlerle çalıştırın.
- **Ücretsiz saatler:** Havuz yalnızca ücretsiz servisler için geçerlidir. Ücretli paketteki servisler ve **Her zaman açık** eklentisi açık olan servis havuzu kullanmaz.
- **Diğer sınırlar:** İhtiyacınızı bir [destek talebiyle](support-tickets.md) iletin.

---

## İlgili Dokümanlar

- [Hızlı Başlangıç](quick-start.md) — Hesap, organizasyon ve ilk adımlar.
- [Faturalama](billing.md) — Cüzdan, ekstreler ve harcama limiti.
- [Servis Uyku Modu](service-sleep.md) — Ücretsiz servislerin uyuma ve uyanma kuralları.
- [Build Kuyruğu](build-queue.md) — Eşzamanlı build'ler ve kuyruk sırası.
- [Erişim Koruması: Başvuru](access-protection-reference.md) — Erişim korumasının tüm sınırları.
- [Destek Talepleri](support-tickets.md) — Limit artışı ve diğer talepler.
