# Hızlı Başlangıç

Komuta'da kodunuzu yayına almaktan veritabanınızı ayağa kaldırmaya kadar ihtiyacınız olan her şey dört hizmette toplanır:

- **[Servisler](https://www.komuta.io/docs/services/new-service-guide)** — Git reponuzdaki uygulamaları build edip canlıya alır.
- **[Job ve CronJob'lar](jobs-cronjobs.md)** — Tek seferlik ya da zamanlanmış işleri çalıştırır.
- **[Stack'ler](stacks.md)** — Servis, kaynak ve domain'lerden oluşan bütün bir kurulumu tek dosyada tanımlar.
- **[Yönetilen servisler](https://www.komuta.io/docs/managed-services/postgresql)** — PostgreSQL, Valkey, RabbitMQ ve API Gateway'i kurulum gerektirmeden sunar.

---

## Başlamadan Önce

**Hesap.** [console.komuta.io](https://console.komuta.io) adresinden e-posta ve şifreyle ya da Google veya GitHub hesabınızla kayıt olun. E-postayla kayıt olduysanız gelen bağlantıyla adresinizi doğrulayın. Kayıt için kredi kartı gerekmez.

**Organizasyon.** Kayıt olunca size ait bir organizasyon açılır ve yöneticisi siz olursunuz. Servisler, yönetilen servisler, cüzdan ve harcama limiti organizasyona aittir. Ekip arkadaşlarınızı **Erişim Yönetimi** sayfasından **Üye** ya da **Yönetici** olarak davet edebilirsiniz; davet 72 saat geçerlidir. Birden fazla organizasyona üyeyseniz sağ üstteki kullanıcı menüsünden aralarında geçiş yaparsınız. Ayrıntılar: [Organizasyon ve Ekip](organizations-team.md).

**Git hesabı.** Servisler Git reposundan build edilir. İlk servisten önce **Entegrasyonlar** sayfasından GitHub, GitLab ya da Azure DevOps hesabınızı bağlayın; herkese açık bir repo da bağlı bir hesap üzerinden yayınlanır. Ayrıntılar: [Entegrasyonlar](integrations.md).

**Ücretsiz başlangıç.** Kayıtla açılan organizasyona 30 gün geçerli 10 USD hoş geldin kredisi yüklenir. Her organizasyon bir servisi ücretsiz pakette çalıştırabilir; ücretsiz servis bir süre trafik almazsa uyur ve ilk istekle uyanır ([Servis Uyku Modu](service-sleep.md)). Her fatura döneminde 2.000 build dakikası ücretsizdir. PostgreSQL, Valkey, RabbitMQ ve API Gateway ücretlidir. Ücretsiz planın kuralları ve fiyatlar [Fiyatlandırma](https://www.komuta.io/pricing) sayfasındadır.

**Maliyet.** Fiyatlar USD cinsindendir ve vergi hariçtir; kullanım saatlik ölçülür. Ücretli bir şeyi oluşturmadan önce fiyatı görürsünüz: servis sihirbazında paketlerin fiyatı, yönetilen servis sihirbazında ve Stack planında **Tahmini aylık maliyet** gösterilir. Organizasyon yöneticileri o ana kadarki harcamayı **Hesap → Güncel Harcama**, kesilen ekstreleri **Hesap → Ekstreler** sayfasında görür; ekstre her ayın 6'sında kesilir.

**Harcama limiti.** Yeni bir organizasyon aylık 20 USD'lik kaynak limitiyle başlar. Yeni bir şey oluşturduğunuzda ya da planını büyüttüğünüzde aylık tahmin bu limiti aşacaksa işlem yapılmaz; zaten çalışan kullanım ise durdurulmaz. Kart doğruladıkça, bakiye yükledikçe ve ekstre ödedikçe limit yükselir; güncel seviye **Hesap → Cüzdan** sayfasındadır. Ayrıntılar: [Faturalandırma ve Cüzdan](billing.md).

---

## Servis Yayınlama

Git reponuzdaki bir uygulamayı build edip canlıya almanın yolu budur. Akış [**Uygulamalar**](https://console.komuta.io/services) sayfasındaki **Yeni Uygulama** ile başlar ve üç adımdan geçer:

1. **Proje** — Servisin ekleneceği proje ve çalışacağı bölge.
2. **Bağlan** — Git hesabı, repo, branch ve build edilen imajın gönderileceği konteyner kayıt defteri. Komuta'nın kendi kayıt defteri hazır gelir; isterseniz **Entegrasyonlar** sayfasından kendinizinkini ekleyebilirsiniz.
3. **Yapılandır** — Repoda bulunan servislerin ayarları. Son düğmeyle kod build edilir ve servis canlıya alınır.

> **Not:** Komuta repoyu tarar ve servisleri Dockerfile'lardan bulur. Repoda Dockerfile yoksa yapay zekâ bir tane üretir ve reponuza pull request olarak açar; sonraki build'lerin çalışması için bu pull request'i branch'inize merge etmeniz gerekir. Repo kökünde `komuta.yaml` varsa akış Stack oluşturmaya geçer.

![Yeni uygulama sihirbazının Yapılandır adımı — repoda bulunan servisler listeli](https://cdn.komuta.io/docs/tr/images/services/quick-start-create-service.png)

---

## Job ve CronJob

Job bir kez çalışıp biten iştir; CronJob aynı işi takvime bağlar. İkisi de [**Job ve CronJob'lar**](https://console.komuta.io/jobs) sayfasından oluşturulur ve akışları yalnızca zamanlama kısmında ayrılır: Job'da **Elle çalıştıracağım**, CronJob'da **Takvime bağla** seçilir.

İşin ne çalıştıracağı üç kaynaktan seçilir — repodaki bir proje (build edilir), doğrudan yazılan kısa bir betik (Bash, Python veya Node) ya da hazır bir container imajı. Yayınlanan Job istenen anda elle tetiklenir; CronJob zamanlamasına göre kendi çalışır ve çalıştırma geçmişi aynı sayfada tutulur.

![CronJob oluşturma akışının zamanlama adımı](https://cdn.komuta.io/docs/tr/images/jobs/create-jobs-page.png)

---

## Stack ile Kurulum

Stack, bir uygulamayı oluşturan servisleri, yönetilen kaynakları ve domain'leri tek dosyada tanımlar; böylece bütün kurulum tek seferde ve tekrarlanabilir şekilde yapılır. [**Stack'ler**](https://console.komuta.io/stacks) sayfasından oluşturulur.

Dosyanın biçimi [komuta.yaml Referansı](stack-manifest-reference.md) sayfasındadır. Tanım hazırlandıktan sonra **Plan oluştur** dosyayı doğrular; dosyada sorun yoksa neyin oluşturulacağını ve tahmini aylık maliyeti kurulumdan önce listeler. Şifre ve anahtar gibi gizli değerler tek kullanımlık güvenli bir bağlantı üzerinden girilir, Stack dosyasına yazılmaz. Kaynaklar yalnızca plan onaylandıktan sonra oluşturulur.

![Oluşturulan Stack planı — kaynak listesi ve tahmini maliyet](https://cdn.komuta.io/docs/tr/images/stacks/create-stack-page.png)

---

## Yönetilen Servisler

Yönetilen servisler kendi repo veya build sürecine ihtiyaç duymaz; Komuta bunları kurar, yapılandırır, yedekler ve izler.

**PostgreSQL, Valkey ve RabbitMQ** aynı adımlarla oluşturulur: [**Yönetilen Servisler**](https://console.komuta.io/addons) sayfasında servis seçilip **Yeni örnek** ile bölge, plan, sürüm ve ilişkilendirilecek proje belirlenir. Servis birkaç dakika içinde kullanıma hazır olur ve plan sonradan yükseltilebilir.

**API Gateway**, **Yönetilen Servisler > API Gateway'ler** sayfasından oluşturulur. Kurulumdan sonra kendi API'leriniz gateway'in altına eklenir ve her biri için ayrı yol, kimlik doğrulama ve rate limit tanımlanır. Ayrıntılar: [API Gateway](api-gateway.md).

> **Uyarı:** Gateway'in API anahtarı yalnızca oluşturma ekranında bir kez gösterilir ve sonradan tekrar görüntülenemez.

![Yeni PostgreSQL örneği oluşturma — plan seçim ekranı](https://cdn.komuta.io/docs/tr/images/managed-services/managed-services-page.png)

