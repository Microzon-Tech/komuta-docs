# Komuta Nedir? Temel Kavramlar

Bu sayfa Komuta'nın ne olduğunu ve konsoldaki parçaların birbirine nasıl bağlandığını anlatır. Sonunda konsolda karşılaşacağınız terimlerin bir sözlüğü vardır. Konsolu ilk kez açıyorsanız ya da bir terimin neyi kastettiğinden emin değilseniz buradan başlayın.

İlk servisinizi yayına almanın adımları [Hızlı Başlangıç](quick-start.md) sayfasındadır; bu sayfa o adımları tekrar etmez.

---

## Komuta Kısaca

Komuta bir platform hizmetidir (PaaS). Kodunuzu Git'e gönderirsiniz; Komuta onu build eder, çalıştırır, ölçekler, korur ve izler. Sunucu, işletim sistemi ya da altyapı yönetmezsiniz.

- **Build eder.** Repodaki Dockerfile'lardan servisleri bulur. Dockerfile yoksa yapay zekâ bir tane üretir ve reponuza pull request olarak açar. Her build güvenlik ve zafiyet taramasından geçer.
- **Çalıştırır.** Uygulamanız seçtiğiniz kaynak paketiyle, bir ya da daha fazla replika olarak çalışır. Yeni sürüm, seçtiğiniz rollout stratejisiyle yayına alınır.
- **Ölçekler.** Otomatik ölçekleme replika sayısını CPU ve bellek kullanımına göre artırır ve azaltır.
- **Korur.** Erişim koruması, servisinizin adresini kimin açabileceğini belirler. Güvenlik Merkezi bulguları ve çalışan servislerin davranışını tek yerde toplar.
- **İzler.** Loglar, metrikler ve trace'ler **Gözlemlenebilirlik** ekranındadır. Uyarı kuralları e-posta, Slack ya da Microsoft Teams'e bildirim gönderir.

Kod ve build gerektirmeyen **yönetilen servisler** de sunar: PostgreSQL, Valkey, RabbitMQ ve API Gateway. Kurulumu, yedeği ve izlemeyi Komuta yapar.

Komuta'yı yapay zekâ asistanınızdan da yönetebilirsiniz. Komuta MCP sunucusuna bağlanan bir asistan (Claude, Codex) proje açmak, deploy etmek, veritabanı kurmak ve log incelemek gibi işleri sizin yerinize yapar.

Servisler şu anda tek bölgede, **Germany-East** (`de-east`) bölgesinde çalışır.

---

## Parçalar Nasıl Bir Araya Gelir

Komuta'da her şey bir **organizasyona** aittir. Organizasyonun içinde **projeler**, projelerin içinde de çalışan iş yükleri bulunur:

```text
Kullanıcı hesabınız
└── Organizasyon ("Acme")
    │   Üyeler ve roller · Cüzdan, ekstreler, harcama limiti
    │   Entegrasyonlar (Git hesapları, konteyner registry'leri)
    │   Alan adları · Ortam değişkeni grupları · Uyarı kanalları
    │
    ├── Proje: magaza
    │   ├── Uygulama: api
    │   ├── Uygulama: web ─── özel bağlantı ──► api
    │   ├── CronJob: gece-raporu
    │   ├── API Gateway: genel-api
    │   ├── PostgreSQL örneği: magaza-db
    │   └── Stack: magaza   (yukarıdakileri tek dosyada tanımlar)
    │
    ├── Proje: ic-araclar
    │   └── Uygulama: yonetim-paneli
    │
    └── Paylaşılan: Valkey örneği   (hiçbir projeye bağlı değil)
```

### Organizasyon

Organizasyon, ekibinizin çalışma alanıdır. Kayıt olduğunuzda size ait bir organizasyon açılır ve yöneticisi siz olursunuz. Servisler, yönetilen servisler, üyeler, Git bağlantıları, cüzdan ve harcama limiti organizasyona aittir; her organizasyonun servisleri ve faturası ayrıdır.

Birden fazla organizasyona üye olabilirsiniz. Sağ üstteki kullanıcı menüsünün **Organizasyonlar** bölümünden aralarında geçiş yapar ya da **Organizasyon oluştur** ile yenisini açarsınız. Bir hesap en fazla 5 organizasyona sahip olabilir.

### Proje

Proje, birbiriyle ilgili repoları ve uygulamaları bir arada tutan klasördür. **Uygulamalar → Yeni Proje** ile açılır: **Proje Adı** (3–100 karakter) ve **Açıklama** girilir. Yeni uygulama ya da Job oluştururken proje listesindeki **Yeni proje oluştur…** seçeneğiyle de açabilirsiniz.

Bir projede şunlar bulunur:

- Bağlanan Git repoları ve bu repolardan çıkan uygulamalar,
- Job ve CronJob'lar,
- API Gateway'ler,
- Stack'ler,
- İsterseniz yönetilen servis örnekleri. Örnek oluştururken **Proje** alanında bir proje seçilmezse örnek **Paylaşılan** olarak kalır.

Konsolun üst çubuğundaki proje seçici, listeleri seçtiğiniz projeye daraltır; **Tüm projeler** hepsini gösterir.

Proje aynı zamanda bir yetki sınırıdır. **Erişim Yönetimi** sayfasında bir üyeye bir projedeki her şey için **Okuma** ya da **Düzenleme** yetkisi verebilirsiniz.

> **Not:** Proje bölge belirlemez. Bölge, her uygulama, yönetilen servis örneği ya da API Gateway oluşturulurken ayrıca seçilir.

### Uygulama (servis)

Uygulama, Git reponuzdaki bir uygulamanın Komuta'da sürekli çalışan halidir: bir web sitesi, bir API, bir arka plan işçisi. Konsolun menüsünde **Uygulamalar**, sayfaların çoğunda ve bu dokümanlarda **servis** denir; ikisi aynı şeydir.

Bir repodan birden fazla servis çıkabilir. Her servisin kendi kaynak paketi, replikaları, portları ve adresi, ortam değişkenleri, pipeline geçmişi ve logları vardır.

### Job ve CronJob

Job bir kez çalışıp biten iştir; CronJob aynı işi bir takvime bağlar. Veritabanı taşıma, gece raporu ya da toplu e-posta gibi sürekli çalışması gerekmeyen işler içindir. **Job ve CronJob'lar** sayfasından oluşturulur.

### Yönetilen servisler

**Yönetilen Servisler** menüsünde iki tür ürün vardır:

- **PostgreSQL, Valkey ve RabbitMQ** — İlişkisel veritabanı, bellek içi anahtar-değer deposu ve mesaj kuyruğu. Her biri bir **örnek** olarak oluşturulur; CPU, bellek ve disk seçtiğiniz **plandan** gelir.
- **API Gateway** — API'lerinizin önünde tek bir adres. Anahtarları ve hız limitlerini konsoldan yönetirsiniz. Bir projenin altında oluşturulur.

### Stack

Stack, bir uygulamayı oluşturan servisleri, yönetilen servisleri, alan adlarını ve gizli değerleri tek bir dosyada tanımlar. **Doğrula** dosyadaki hataları gösterir, **Plan oluştur** neyin oluşturulacağını ve **Tahmini aylık maliyet**'i listeler; kaynaklar yalnızca plan onaylandıktan sonra oluşturulur. Repo kökünde `komuta.yaml` bulunan bir repo, yeni uygulama akışında Stack oluşturmaya yönlendirilir.

---

## Koddan Canlıya: Bir Servisin Yolculuğu

Bir servisin yaşamı, konsolda gördüğünüz kavramları şu sırayla bağlar:

1. **Kaynak.** Git hesabınız **Entegrasyonlar** sayfasından bağlanır. Servis bir repoyu ve bir branch'i takip eder.
2. **Pipeline.** **Dağıt** ile ya da otomatik dağıtım açıkken her push'ta bir pipeline başlar. Kaynak kod hazırlanır, imaj build edilir, güvenlik ve zafiyet taramasından geçer, imzalanır ve canlıya dağıtılır. Build'ler sıraya girer; aynı anda kaç build çalışacağı hesap seviyenize bağlıdır.
3. **İmaj.** Build edilen imaj bir konteyner registry'sine gönderilir. Komuta'nın kendi registry'si hazır gelir.
4. **Dağıtım.** Yeni sürüm, servisin rollout stratejisine göre canlıya alınır: **Blue/Green**, **Otomatik Yükseltme** ya da **Canary**. Her dağıtım **Geçmiş** sayfasında kalır; **Geri al** ile önceki bir sürüme dönülür.
5. **Çalışma.** Servis, kaynak paketinin CPU ve belleğiyle, belirlenen sayıda replika olarak çalışır. Sağlık kontrolleri başarısız olan bir replikayı trafikten çıkarır ya da yeniden başlatır. Otomatik ölçekleme replika sayısını yüke göre değiştirir.
6. **Trafik.** İnternetten gelen istek servisin **Genel URL**'ine ulaşır (Komuta'nın verdiği alan adı ya da sizin özel alan adınız), erişim korumasından geçer ve servisin portuna yönlenir.
7. **Gözlem.** Loglar, metrikler ve trace'ler toplanır. Bir koşul gerçekleştiğinde uyarı kuralı bildirim gönderir.

Servis başka bir servisinize özel ağ üzerinden **Bağlantılar** sayfasından erişir. Bir yönetilen veritabanına ise örneğin bağlantı bilgisini servisin ortam değişkenlerine ekleyerek bağlanır; Stack dosyasında bu değer doğrudan örnekten okunabilir.

---

## Konsolda Nerede

Sol menü dört başlık altında toplanır:

| Başlık | İçindekiler |
|---|---|
| **İş Yükleri** | **Uygulamalar**, **Job ve CronJob'lar**, **Stack'ler**, **Yönetilen Servisler** (PostgreSQL, Valkey, RabbitMQ, **API Gateway'ler**) |
| **İzleme** | **Gözlemlenebilirlik**, **Uyarılar**, **Güvenlik** |
| **Yapılandırma** | **Entegrasyonlar**, **Alan Adları**, **Ortam Değişkeni Grupları** |
| **Yönetim** | **Erişim Yönetimi**, **Hesap** |

Bir servisi açtığınızda servisin kendi menüsü gelir: **Genel Bakış**, **Loglar**, **Dağıtım** (**Pipeline'lar**, **Geçmiş**, **Otomatik dağıtım**), **Gözlemlenebilirlik**, **Güvenlik**, **Yapılandırma** (**Ortam değişkenleri**, **Bağlantılar**, **Erişim ve portlar**, **Sağlık kontrolleri**, **Otomatik ölçekleme**), **Bant genişliği**, **Depolama** ve **Fatura**.

**Hesap** sayfalarının kendi menüsü üç gruptur: **Kişisel** (**Profil**, **Parola**, **İki Adımlı Doğrulama**, **API Anahtarları** ve diğerleri), **Organizasyon** (**Organizasyonlar**, **Genel**, **Bildirimler ve Uyarılar**) ve **Faturalandırma** (**Güncel Harcama**, **Cüzdan**, **Ekstreler**, **Ödeme Yöntemleri**). **Faturalandırma** grubunu yalnızca organizasyon yöneticileri görür.

---

## Sözlük

Konsol Türkçe ve İngilizce kullanılabilir; aşağıda her terimin iki dildeki adı verilmiştir.

| Konsolda (TR / EN) | Ne anlama gelir | Daha fazlası |
|---|---|---|
| **Organizasyon** / Organization | Servislerin, üyelerin, Git bağlantılarının ve faturalandırmanın sahibi olan çalışma alanı. | [Organizasyonlar ve Ekip](organizations-team.md) |
| **Proje** / Project | Organizasyon içinde ilgili repoları, uygulamaları, Job'ları, gateway'leri ve Stack'leri gruplayan klasör. Aynı zamanda yetki sınırı. | [Yeni Servis Oluşturma](new-service-guide.md) |
| **Uygulamalar** / Apps (servis) | Git reposundan build edilip sürekli çalışan uygulama. Dokümanlarda "servis" olarak geçer. | [Yeni Servis Oluşturma](new-service-guide.md), [Servis Dashboard](service-dashboard.md) |
| **Job ve CronJob'lar** / Jobs & CronJobs | Bir kez çalışıp biten (Job) ya da takvime bağlı (CronJob) iş. | [Job ve CronJob'lar](jobs-cronjobs.md) |
| **Yönetilen Servisler** / Managed Services | Komuta'nın kurup işlettiği PostgreSQL, Valkey, RabbitMQ ve API Gateway. | [PostgreSQL](managed-postgresql.md), [Valkey](managed-valkey.md), [RabbitMQ](managed-rabbitmq.md) |
| Örnek / Instance (**Yeni örnek** / New instance) | Bir yönetilen servisin kendi diski, kullanıcısı ve yedekleri olan tek kopyası. | [PostgreSQL](managed-postgresql.md) |
| **Plan** / Plan | Yönetilen servis örneğinin CPU, bellek ve disk boyutu. Sonradan yükseltilebilir. | [PostgreSQL](managed-postgresql.md) |
| **Paylaşılan** / Shared | Hiçbir projeye bağlanmamış yönetilen servis örneği. | [PostgreSQL](managed-postgresql.md) |
| **API Gateway'ler** / API Gateways | API'lerinizin önünde anahtar ve hız limitiyle çalışan tek adres. | [API Gateway](api-gateway.md) |
| **Stack'ler** / Stacks | Servisleri, yönetilen servisleri ve alan adlarını tek dosyada tanımlayıp planla birlikte kuran yapı. | [Stack'ler](stacks.md), [Stack Manifest Başvurusu](stack-manifest-reference.md) |
| **Bölge** / Region | Servisin ya da örneğin çalıştığı konum. Şu anda tek bölge vardır: Germany-East. | [Yeni Servis Oluşturma](new-service-guide.md) |
| **Kaynak Paketi** / Resource Package | Servisin CPU ve bellek miktarı; fiyatı da bunu belirler. | [Faturalandırma](billing.md) |
| **Replika** / Replicas | Servisin aynı anda çalışan kopyaları. | [Otomatik Ölçekleme](services-autoscaling.md) |
| **Otomatik ölçekleme** / Autoscaling | Replika sayısını CPU ve bellek kullanımına göre min ve max arasında değiştiren ayar. | [Otomatik Ölçekleme](services-autoscaling.md) |
| **Pipeline'lar** / Pipelines | Kodun build edilip taranarak servise dağıtıldığı çalışmalar. | [Pipeline'lar](service-pipeline-guide.md), [Build Kuyruğu](build-queue.md) |
| **Dağıtım** / Deployment | Yeni bir sürümün canlıya alınması; geçmiş sürümler **Geçmiş** sayfasındadır. | [Dağıtım Geçmişi](service-deployment-history.md) |
| Rollout stratejisi (**Blue/Green**, **Otomatik Yükseltme**, **Canary**) | Yeni sürümün trafiği nasıl devraldığını belirleyen yöntem. | [Dağıtım Stratejileri](deployment-strategies.md) |
| **Otomatik dağıtım** / Auto-deploy | Takip edilen branch'e gelen push'ta pipeline'ın kendiliğinden başlaması. CI'dan da tetiklenebilir. | [Otomatik Dağıtım](service-auto-deploy.md), [CI'dan Deploy Tetikleme](ci-deploy.md) |
| **Ortam değişkenleri** / Environment variables | Uygulamanın çalışırken okuduğu yapılandırma değerleri. | [Ortam Değişkenleri](services-environment-variables.md) |
| **Ortam Değişkeni Grupları** / Environment Groups | Birden fazla servisin paylaştığı değişkenler. Değerler servise kopyalanır; grubu değiştirmek, güncellemeyi gönderene kadar servisi değiştirmez. | [Ortam Değişkeni Grupları](environment-groups.md) |
| **Bağlantılar** / Connections | Bir servisin başka bir servisinize genel adres kullanmadan, özel ağ üzerinden erişmesi. Tek yönlüdür. | [Bağlantılar](services-connections.md) |
| **Erişim ve portlar** / Access & ports | Trafiğin servise hangi porttan girdiği ve adresi kimin açabileceği. | [Erişim ve Portlar](services-ports.md) |
| **Genel URL** / Public URL | Servise internetten ulaşılan adres: **Komuta alan adı** ve varsa **Özel alan adı**. | [Erişim ve Portlar](services-ports.md) |
| **Alan Adları** / Domains | Kendi alan adınızı bir uygulamaya bağladığınız sayfa. Sertifika otomatik verilir ve yenilenir. | [Özel Alan Adları](custom-domains.md) |
| **Erişim koruması** / Access protection | Servis adresini yalnızca seçtiğiniz kişilere, ağlara ya da makinelere açan koruma. Her plana dahildir. | [Erişim Koruması](service-access-protection.md) |
| **Sağlık kontrolleri** / Health probes | Servisin hazır ve ayakta olup olmadığını düzenli sorgulayan kontroller. | [Sağlık Kontrolleri](services-health-probes.md) |
| **Depolama** / Storage | Servise bağlanan, pod yeniden başlasa da korunan kalıcı hacimler. | [Depolama](storage.md) |
| **Bant genişliği** / Bandwidth | Servisin trafiğini ve bant genişliği planını gösteren sayfa. Plan, servisin her pod'u için giden trafik hız sınırını belirler; varsayılan plan ücretsizdir, üst planlar aylık ücretlidir. | [Bant Genişliği Planları](bandwidth-plans.md) |
| **Uyuyor** / Sleeping | Trafik almayan ücretsiz bir servisin replikalarının sıfıra indirildiği durum. İlk istekle uyanır. | [Servis Uykusu](service-sleep.md) |
| **Her zaman açık** / Always on | Servisin uyumasını engelleyen ücretli eklenti. Açıkken servis ücretsiz saat havuzundan tüketmez. | [Servis Uykusu](service-sleep.md) |
| **Loglar** / Logs | Servisin container çıktısının arandığı ve canlı izlendiği sayfa. | [Loglar](service-logs.md) |
| **Gözlemlenebilirlik** / Observability | Log, metrik ve trace'lerin birlikte incelendiği alan. | [Gözlemlenebilirlik](observability.md) |
| **Uyarılar** / Alerts | Metrik ya da log koşulu gerçekleştiğinde bildirim gönderen kurallar. | [Uyarılar: Genel Bakış](alert-guide.md) |
| **Kanallar** / Channels | Uyarıların gittiği e-posta, Slack ve Microsoft Teams hedefleri. | [Kanallar ve Bildirim Teslimi](notification-guide.md) |
| **Güvenlik** / Security (Güvenlik Merkezi) | Güvenlik bulgularını, servis korumasını, erişim etkinliğini ve denetim kayıtlarını bir araya getiren alan. | [Güvenlik Merkezi](security-center-guide.md) |
| **Entegrasyonlar** / Integrations | Git sağlayıcılarına (GitHub, GitLab, Azure DevOps) ve konteyner registry'lerine bağlantılar. | [Entegrasyonlar](integrations.md) |
| **Erişim Yönetimi** / Access Control | Üyeleri davet ettiğiniz, rollerini ve proje bazında yetkilerini belirlediğiniz sayfa. | [Organizasyonlar ve Ekip](organizations-team.md) |
| **Yönetici** / Admin, **Üye** / Member | Organizasyondaki iki rol. Faturalandırma sayfalarını yalnızca yöneticiler görür. | [Organizasyonlar ve Ekip](organizations-team.md) |
| **Cüzdan** / Wallet | Bakiye, harcama seviyesi ve limit artışı talebi. | [Faturalandırma](billing.md) |
| **Ekstreler** / Statements | Her fatura döneminin sonunda kesilen ekstreler. | [Faturalandırma](billing.md) |
| **Güncel Harcama** / Current Spend | İçinde bulunulan dönemde o ana kadarki harcama. | [Faturalandırma](billing.md) |
| **Aylık kaynak maliyeti tahmin limiti** / Monthly resource estimate limit | Yeni bir şey oluşturmanıza ya da planı büyütmenize izin verilen aylık tahmin üst sınırı. Çalışan kullanımı durdurmaz. | [Faturalandırma](billing.md), [Limitler](limits.md) |
| **API Anahtarları** / API Keys | Kendi agent'ınızın ya da CI'ınızın Komuta'ya programatik erişimi için kişisel anahtarlar. Gizli değer bir kez gösterilir. | [Hesap Güvenliği](account-security.md) |
| Komuta MCP | Yapay zekâ asistanınızın Komuta'da sizin adınıza işlem yapmasını sağlayan sunucu. | [Komuta MCP Sunucusunun Kurulumu](mcp-setup.md), [Komuta MCP](mcp-tools.md) |

---

## İlgili Dokümanlar

- [Hızlı Başlangıç](quick-start.md) — Kayıttan ilk servise kadar adımlar.
- [Yeni Servis Oluşturma](new-service-guide.md) — Yeni uygulama sihirbazının ayrıntıları.
- [Organizasyonlar ve Ekip](organizations-team.md) — Üyeler, roller ve organizasyonlar arası geçiş.
- [Faturalandırma](billing.md) — Cüzdan, ekstreler ve harcama limiti.
- [Limitler](limits.md) — Hesap seviyeleri ve platform sınırları.
- [Fiyatlandırma](https://www.komuta.io/pricing) — Paketler, ücretsiz plan ve fiyatlar.
