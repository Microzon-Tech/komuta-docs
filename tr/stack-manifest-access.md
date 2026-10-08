# Stack Manifestinde Erişim Koruması

Bir [Stack](quick-start.md#stack-ile-kurulum)'teki servis, erişim korumasını Stack dosyasında; kaynak kod, işlem gücü ve ağ ayarlarının yanında tanımlayabilir. Plan uygulandığında Komuta korumayı bu ayarlarla açar. Bu sayfa `access` bloğunu anlatır: neyi ifade edebildiğini, nasıl denetlendiğini ve bir servis oluşturulurken, güncellenirken ya da sahiplenilirken ne olduğunu.

Erişim korumasının ne yaptığı ve konsoldan ayarlanabilen her şey [Erişim Koruması](service-access-protection.md) sayfasında anlatılır.

---

## access bloğu

```yaml copy
version: 1
name: main
services:
  - name: api
    source: { repository: 'https://github.com/acme/api.git', branch: main }
    compute: { plan: dz-dev }
    network: { public: true }
    access:
      signIn: true
      allowIps: ['203.0.113.0/24']
```

| Alan | Tür | Varsayılan | Anlamı |
|---|---|---|---|
| `signIn` | `true` / `false` | `false` | Sitenin tamamı için **Komuta girişi iste**. |
| `organization` | `true` / `false` | `signIn` `true` ise `true` | Servisi **Organizasyonunuz** ile (tüm aktif üyelerle) paylaşır. Girişi açıp organizasyonla paylaşmamak için `false` yapın; kişileri sonra konsoldan ekleyin. |
| `allowIps` | adres listesi | boş | **IP izin listesi**: her girdide bir IPv4 ya da IPv6 adresi veya CIDR aralığı. |

Yeni bir serviste `signIn` ve `allowIps` birlikte verildiğinde **İkisi birden gereksin** olarak birleşir: ziyaretçi listedeki bir adresten gelmeli ve giriş yapmalıdır. Zaten korunan bir serviste konsolda seçilen birleşim korunur.

Alan adları büyük/küçük harfe duyarlıdır. Bu üçü dışındaki bir alan reddedilir ("Unknown field '…'. Expected fields: signIn, organization, allowIps.").

**Açık blok** — `access: {}` ya da `allowIps` olmadan `signIn: false` — "koruma yok" demektir (oluşturmada ve güncellemede ne yaptığı aşağıda).

---

## Gereksinimler

- Servis bir `service` iş yükü olmalıdır; iş (job) ve zamanlanmış işlerin (cronjob) genel adresi yoktur.
- Servis genel olmalıdır: `network: { public: true }`. Yalnızca bir port yetmez.
- Servis Komuta barındırmasında çalışmalıdır.
- Erişim koruması organizasyonunuz için kullanılabilir olmalıdır.
- Yeni bir serviste `access` bloğu varsa (açık blok olsa bile) ya da plan mevcut bir servisin `access` bloğunu değiştiriyorsa, Stack'i planlayan kişinin **Servis erişim korumasını yönet** izni olmalıdır. Planlama ile uygulama arasında yetkiler değişirse uygulama reddedilir ("Planlamadan sonra yetkiler değişti; yeni bir plan oluşturun.").

---

## Doğrulama

**Doğrula** ve **Plan oluştur** bloğu denetler. `allowIps` girdileri konsoldaki [IP izin listesiyle](access-protection-rules.md#ip-izin-listesi) aynı kurallara uyar: en fazla 100 girdi, yalnızca genel internet adresleri, `/0` yok, host bitleri sıfır, baştaki sıfırlar yok.

| Kod | Mesaj |
|---|---|
| `ACCESS_WORKLOAD_UNSUPPORTED` | Access protection is supported only for service workloads. |
| `ACCESS_REQUIRES_PUBLIC_ADDRESS` | Access protection needs a public address; set network.public to true. |
| `ACCESS_ORGANIZATION_NEEDS_SIGN_IN` | Sharing with the organization needs signIn: true. |
| `ACCESS_ALLOW_IPS_TOO_LONG` | allowIps accepts at most 100 entries. |
| `ACCESS_ALLOW_IP_NOT_PUBLIC` | allowIps entries must be public internet addresses; private, loopback and documentation ranges never reach the gateway. |
| `ACCESS_ALLOW_IP_EVERYTHING` | A /0 range admits everyone; leave allowIps empty instead. |
| `ACCESS_ALLOW_IP_INVALID` | Every allowIps entry must be an IPv4/IPv6 address or CIDR range with no host bits set. |

Plan, uygulayamayacağı bir bloğu da reddeder (kod `CAPABILITY_NOT_AVAILABLE`); örneğin "Access protection is not available to this organization.", "Access protection is supported only on Komuta hosting." ya da "Access protection cannot be combined with private network exposure on this platform.". Bu mesajlar İngilizce gösterilir.

---

## Servis oluşturma

Plan yeni bir servis oluşturduğunda:

- **Koruma isteyen bir blokla** (`signIn: true` ya da boş olmayan `allowIps`) koruma servisle birlikte kurulur: tanımlanan giriş, IP izin listesi ve organizasyon paylaşımı; yol kuralı ve bitiş tarihi olmadan. Konsoldan açılan koruma gibi, servisin ilk dağıtımından birkaç dakika sonra devreye girer. Koruma kurulamazsa servis oluşturulmaz ve çalıştırma adımı [Çalıştırma hataları](#çalıştırma-hataları) altındaki kodlardan biriyle başarısız olur.
- **Açık blokla** (`access: {}`) servis, organizasyonun **Yeni servisleri koru** ayarı açık olsa bile korumasız başlar.
- **Blok yoksa**, konsoldan oluşturulan servislerde olduğu gibi organizasyonun **Yeni servisleri koru** ayarı belirler.

---

## Servis güncelleme

Stack'in zaten yönettiği bir serviste `access` değiştiğinde:

- **`access`'i tek başına değiştirin.** Aynı servisin başka alanlarıyla birlikte `access`'i değiştiren bir plan yerinde uygulanamaz; `access` değişikliğini ayrı bir revizyonda uygulayın.
- **Konsoldaki değişiklikler sessizce ezilmez.** Komuta uygulamadan önce servisin mevcut girişini, IP izin listesini ve organizasyon paylaşımını önceki revizyonun tanımladığıyla karşılaştırır. Konsoldan değiştirilmişlerse uygulama `ACCESS_DRIFT` ile durur ("Access protection changed outside the approved baseline; inspect it in the console before replanning."). Önceki revizyonda `access` bloğu yoksa ve koruma konsoldan kurulduysa, önce mevcut ayarları tanımlayın, sonra bir sonraki revizyonda değiştirin.
- **Manifestte olmayanlar korunur.** Yol kuralları, birleşim, bitiş tarihi, giriş yapanı uygulamaya bildirme ve organizasyon paylaşımı dışındaki tüm paylaşımlar konsolda ayarlandığı gibi kalır.
- **Açık blok korumayı kapatır.** Uygulama, koruma kapanana kadar bekler.
- **Bloğu silmek korumayı olduğu gibi bırakır.** Korumayı Stack'ten kapatmak için açık bir blok tanımlayın.
- Uygulama, değişiklik geçerli olana (**Korunuyor**) kadar bekler. Bu sırada biri korumayı konsoldan değiştirir ya da kapatırsa `ACCESS_DRIFT` ile durur.

Stack sapma (drift) denetimleri erişim korumasına bakmaz; konsoldaki bir değişiklik ancak bir sonraki `access` güncellemesi uygulanırken ortaya çıkar.

---

## Mevcut bir servisi sahiplenme

Stack bir servisi sahiplenirken erişim korumasını değiştiremez. Bir servisi sahiplenen ve aynı servis için `access` bloğu da içeren plan reddedilir: "Access protection is not applied while a service is adopted; adopt it first, then declare access in a later revision." Bloksuz sahiplenmek servisin mevcut korumasına dokunmaz.

---

## Manifestin ifade edemedikleri

Bunlar yalnızca konsoldan (**Servis Detay → Yapılandırma → Erişim ve portlar**) ayarlanır ve Stack güncellemesi bunları korur:

- yol kuralları ve webhook (açık) yolları, yöntem kuralları ve CORS ayarı, ülkeler ve hız sınırı;
- **Biri yeterli** birleşimi;
- **Organizasyonunuz** dışındaki paylaşımlar (üyeler, bağlı organizasyonlar, e-posta adresleri), paylaşım bağlantıları ve servis token'ları;
- oturum süresi, koruma bitiş tarihi ve giriş yapanı uygulamaya bildirme;
- özel ağdan gelebilecek servisler.

Stack'in **Dışa aktar** çıktısı `access`'i yalnızca uygulanan manifest tanımladıysa yazar; konsoldan kurulan koruma dışa aktarıma yazılmaz.

---

## İlgili Dokümanlar

- [Erişim Koruması](service-access-protection.md) — korumanın ne yaptığı, açma ve kapatma.
- [Kurallar](access-protection-rules.md) — Komuta girişi ve IP izin listesinin ayrıntıları.
- [Giriş ve Paylaşım](access-protection-sign-in-sharing.md) — organizasyon paylaşımı ve diğer paylaşımlar.
- [Başvuru](access-protection-reference.md) — sınırlar ve hata mesajları.

---

## Çalıştırma hataları

Bir plan uygulanırken tanımlanan korumayı kuramayan ya da değiştiremeyen adım bu kodlardan biriyle durur. Mesaj, Komuta'nın döndürdüğü haliyle (İngilizce) gösterilir.

| Kod | Mesaj |
|---|---|
| `ACCESS_PROTECTION_NOT_AVAILABLE` | Access protection is not available to this organization. (ya da: Access protection is not enabled on this platform.) — erişim koruması bu organizasyonda ya da platformda kullanılamıyor. |
| `ACCESS_REQUIRES_PUBLIC_ADDRESS` | Access protection needs a public service on Komuta hosting. — koruma, Komuta barındırmasında herkese açık adresi olan bir servis ister. |
| `ACCESS_MESH_EXPOSED` | Access protection cannot be combined with private network exposure on this platform. — bu platformda koruma, özel ağa açma ile birlikte kullanılamaz. |
| `ACCESS_UNSUPPORTED_CLUSTER` | Access protection is supported only on Komuta hosting clusters. — koruma yalnızca Komuta barındırma kümelerinde desteklenir. |
| `ACCESS_RULES_NOT_AVAILABLE` | This service uses access rules that cannot be changed on this platform right now. — servis, bu platformda şu an değiştirilemeyen kurallar kullanıyor. |
| `ACCESS_DECLARATION_INVALID` | The declared access protection needs sign-in or an allow-list, and sharing with the organization needs sign-in; provisioning was not started. — tanım giriş ya da izin listesi içermeli, organizasyonla paylaşım da giriş ister; servis oluşturulmadı. |
| `ACCESS_PROTECTION_NOT_PREPARED` | The declared access protection could not be prepared; provisioning was not started. — tanımlanan koruma hazırlanamadı; servis oluşturulmadı. |
