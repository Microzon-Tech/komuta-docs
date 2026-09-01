# Hızlı Başlangıç

Komuta'da kodunuzu yayına almaktan veritabanınızı ayağa kaldırmaya kadar ihtiyacınız olan her şey dört hizmette toplanır:

- **[Servisler](service-guide.md)** — Git reponuzdaki uygulamaları build edip canlıya alır.
- **Job ve CronJob'lar** — Tek seferlik ya da zamanlanmış işleri çalıştırır.
- **Stack'ler** — Servis, kaynak ve domain'lerden oluşan bütün bir kurulumu tek dosyada tanımlar.
- **[Yönetilen servisler](managed-addons.md)** — PostgreSQL, Valkey, RabbitMQ ve API Gateway'i kurulum gerektirmeden sunar.

---

## Servis Yayınlama

Git reponuzdaki bir uygulamayı build edip canlıya almanın yolu budur. Akış [**Uygulamalar**](https://console.komuta.io/services) sayfasındaki **Yeni Proje** ile başlar ve dört adımdan geçer: servisin çalışacağı proje ve altyapı (**Komuta PaaS** ya da kendi cluster'ınız), Git hesabı ile imajın gönderileceği registry, repodan bulunan servislerin yapılandırması ve servisin yayınlanması.

> **Not:** Komuta **Yapılandır** adımında repoyu tarar ve Dockerfile'lardan servisleri bulur; repoda Dockerfile yoksa build için otomatik olarak bir tane üretir. **Deploy** ile kod build edilir ve servis canlıya alınır.

![Yeni proje sihirbazının Yapılandır adımı — repodan bulunan servisler listeli](https://cdn.komuta.io/docs/tr/images/services/quick-start-create-service.png)

---

## Job ve CronJob

Job bir kez çalışıp biten iştir; CronJob aynı işi takvime bağlar. İkisi de [**Job ve CronJob'lar**](https://console.komuta.io/jobs) sayfasından oluşturulur ve akışları yalnızca zamanlama kısmında ayrılır: Job'da **Elle çalıştıracağım**, CronJob'da **Takvime bağla** seçilir.

İşin ne çalıştıracağı üç kaynaktan seçilir — repodaki bir proje (build edilir), doğrudan yazılan kısa bir betik (Bash, Python veya Node) ya da hazır bir container imajı. Yayınlanan Job istenen anda elle tetiklenir; CronJob zamanlamasına göre kendi çalışır ve çalıştırma geçmişi aynı sayfada tutulur.

![CronJob oluşturma akışının zamanlama adımı](https://cdn.komuta.io/docs/tr/images/jobs/create-jobs-page.png)

---

## Stack ile Kurulum

Stack, bir uygulamayı oluşturan servisleri, yönetilen kaynakları ve domain'leri tek dosyada tanımlar; böylece bütün kurulum tek seferde ve tekrarlanabilir şekilde yapılır. [**Stack'ler**](https://console.komuta.io/stacks) sayfasından oluşturulur.

Tanım hazırlandıktan sonra **Doğrula** yapısal sorunları gösterir, **Plan oluştur** ise neyin oluşturulacağını ve tahmini aylık maliyeti kurulumdan önce listeler. Şifre ve anahtar gibi gizli değerler tek kullanımlık güvenli bir bağlantı üzerinden girilir, Stack dosyasına yazılmaz. Kaynaklar yalnızca plan onaylandıktan sonra oluşturulur.

![Oluşturulan Stack planı — kaynak listesi ve tahmini maliyet](https://cdn.komuta.io/docs/tr/images/stacks/create-stack-page.png)

---

## Yönetilen Servisler

Yönetilen servisler kendi repo veya build sürecine ihtiyaç duymaz; Komuta bunları kurar, yapılandırır, yedekler ve izler.

**PostgreSQL, Valkey ve RabbitMQ** aynı adımlarla oluşturulur: [**Yönetilen Servisler**](https://console.komuta.io/addons) sayfasında servis seçilip **Yeni örnek** ile bölge, plan, sürüm ve ilişkilendirilecek proje belirlenir. Servis birkaç dakika içinde kullanıma hazır olur ve plan sonradan yükseltilebilir.

**API Gateway**, **Yönetilen Servisler > API Gateway'ler** sayfasından oluşturulur. Kurulumdan sonra kendi API'leriniz gateway'in altına eklenir ve her biri için ayrı yol, kimlik doğrulama ve rate limit tanımlanır.

> **Uyarı:** Gateway'in API anahtarı yalnızca oluşturma ekranında bir kez gösterilir ve sonradan tekrar görüntülenemez.

![Yeni PostgreSQL örneği oluşturma — plan seçim ekranı](https://cdn.komuta.io/docs/tr/images/managed-services/managed-services-page.png)

