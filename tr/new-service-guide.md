# Yeni Servis Oluşturma

Servis, Komuta'nın dağıtım hedefidir: bir git deposundaki — kısaca repodaki — kod build edilir ve çalışan bir uygulama olarak yayına alınır. Her servis kendi adını, portunu, ortam değişkenlerini, kaynak paketini ve dağıtım geçmişini taşır. Servis oluşturma sihirbazı bu üç soruyu sırayla çözer — hangi projeye ait, kodu nereden gelecek, nasıl çalışacak.

Yeni servislerinizi Komuta MCP üzerinden de yayınlayabilirsiniz; kurulum için [MCP kurulumu](https://www.komuta.io/docs/mcp/mcp-setup) sayfasına bakın.

---

## Servis Nedir?

Bir servis tek bir uygulamaya karşılık gelir. Bir repoda birden fazla uygulama varsa (örneğin bir API ve bir arayüz), bunların her biri ayrı bir servistir; aynı repodan tek seferde birlikte oluşturulabilirler.

Servisler bir **proje** altında gruplanır. Proje, birlikte çalışan servisleri bir arada tutan yapıdır; yeni bir servis her zaman bir projeye bağlanır. Servisin adı, oluşturulduktan sonra platformda o servise ait her kaydın kimliği olur — bu yüzden servis adının anlamlı ve benzersiz olması istenir.

Servisin kaynağı bir repo ve o reponun belirli bir branch'idir. Komuta bu branch'i tarar, build edilebilir uygulamaları bulur ve bulduklarını yapılandırılabilir birer servis olarak sunar.

---

## 1. Proje ve Konum

İlk adım servisin nereye ait olduğunu ve nerede çalışacağını belirler.

### Proje Seçimi

Servis, seçilen projenin altına eklenir. Uygun bir proje yoksa **Yeni proje oluştur** seçeneği kullanılır; proje, servisten hemen önce oluşturulur.

### Bölge Seçimi

Servisin çalışacağı bölge buradan seçilir.

> **Not:** Servis bir kez oluşturulduktan sonra bölge değiştirilemez.

![Proje adımı — proje seçici ve bölge listesi yan yana](https://cdn.komuta.io/docs/tr/images/create-service/create-service-first-step.png)

---

## 2. Repo ve Branch

İkinci adım kodun nereden geleceğini belirler.

### Repo Seçimi

Listedeki repolar, bağlı git hesaplarından gelir. Aranan repo listede yoksa o repoyu kapsayan hesap bağlı olmayabilir; bağlantı [Integrations](https://console.komuta.io/integrations) sayfasından eklenir.

### Branch Seçimi

Repo seçildiğinde branch listesi açılır. Seçilen branch, hem taramanın hem de ilk dağıtımın kaynağıdır.

![Repo adımı — seçili repo kartı açılmış, altında branch listesi](https://cdn.komuta.io/docs/tr/images/create-service/create-service-second-step.png)

---

## 3. Servisleri Yapılandırma

Komuta, seçilen branch'i tarar ve build edilebilir uygulamaları listeler. Bulunanların her biri, kendi ayarlarıyla birlikte ayrı bir servis olarak oluşturulabilir.

### Bulunan Servisler

Komuta, taramada repoda build edilebilir bulduğu her uygulamayı ayrı bir servis olarak sunar.

Beklediğiniz servis listede yoksa **Yeniden tara** ile tekrar kontrol edebilirsiniz.

### Port

Port, uygulamanın gelen istekleri dinlediği kapıdır; Komuta trafiği bu porta yönlendirir. Komuta port değerini projeden çıkarır; bulamazsa genel bir varsayılan kullanır.

### Yapılandırma Dosyası

Servisin hangi Dockerfile ile build edileceği **Yapılandırma dosyası** alanında yazar; Komuta bu yolu taramada bulur. Dockerfile repoda başka bir yerdeyse yol buradan düzeltilir.

### Dockerfile Bulunamazsa

Komuta yalnızca bir Dockerfile üzerinden build alır. Seçilen branch'te hiç Dockerfile yoksa yapay zekâ ile bir tane üretmesi önerilir; bu mümkün değilse repoyu dağıtabilmek için bir Dockerfile eklenmesi gerekir. Dockerfile'ı Komuta MCP ile de oluşturabilirsiniz — kurulum için [MCP kurulumu](https://www.komuta.io/docs/mcp/mcp-setup) sayfasına bakın.

> **Uyarı:** Dockerfile yapay zekâ ile üretildiğinde repoda bir pull request açılır ve bu pull request birleştirilmelidir. Birleştirilmediği sürece sonraki dağıtımlar Dockerfile'ı bulamaz ve başarısız olur.

### Kaynak Paketi

Kaynak paketi, servisin ne kadar CPU ve bellek kullanacağını ve aylık ücretini belirler. Seçilen her servis için ayrı ayrı belirlenir.

Tarama, uygulamanın yapısına bakarak ayrıca bir paket önerir.

> **İpucu:** Kaynak paketini servisi oluşturduktan sonra yükseltmek mümkündür.

### Ortam Değişkenleri

Uygulamanın çalışmak için ihtiyaç duyduğu ortam değişkenleri (`DATABASE_URL`, `API_KEY` vb.) servis oluşturulurken girilebilir. Gizli tutulması gereken değerler **Secret** olarak işaretlenir.

### Otomatik Dağıtım

[Otomatik dağıtım](https://www.komuta.io/docs/services/service-auto-deploy) açıkken, seçilen branch'e yapılan her push yeni bir dağıtım başlatarak servisin yeni sürümünü yayına alır.

Bu ayar **Gelişmiş seçenekler** altında, servis bazında bulunur — çok uygulamalı bir repoda bir servis her push'ta güncellenirken diğeri elle yayınlanabilir.

**Yol filtreleri**, dağıtımın yalnızca belirli klasörlerdeki değişikliklerde tetiklenmesini sağlar — örneğin `api/**` yazıldığında yalnızca `api` klasörüne gelen push'lar servisi güncellerken diğer değişiklikler görmezden gelinir. Boş bırakıldığında servis kendi klasörünü izler.

---

## Otomatik Kurulan Alarmlar

Her servis için üç alarm, seçim yapılmadan kurulur: **pod yeniden başlatma**, **pod hazır olmama** ve **bellek aşımı nedeniyle sonlandırma**.

> **Not:** Kendi alarmlarınızı servis yayına alındıktan sonra [Alarmlar](https://console.komuta.io/alerts/overview) sayfasından oluşturabilir ve düzenleyebilirsiniz.

---

## Dağıtım

**Dağıt** düğmesi, seçilen servis sayısını gösterir ve tüm servisleri tek işlemde oluşturur. Dağıtım tamamlandığında [servis listesine](https://console.komuta.io/services) dönülür; build ve yayına alma süreci oradan izlenir.

![Dağıtım öncesi özet paneli](https://cdn.komuta.io/docs/tr/images/create-service/create-service-third-step.png)

---

## Sonraki Adımlar

- **Servis detay sayfası** — dağıtım sonrası build durumu, loglar ve pod bilgileri buradan izlenir.
- **Otomatik ölçekleme (HPA)** — servis tek bir kopya ile başlar; ölçekleme kuralları servis oluşturulduktan sonra servisin kendi sayfasından tanımlanır.
- [Integrations](https://console.komuta.io/integrations) — git hesapları ve registry bağlantıları buradan yönetilir.
