# komuta.yaml Manifest Başvurusu

Bu sayfa, [Stack](stacks.md) dosyası `komuta.yaml`'ın 1. sürüm biçimini anlatır: her alanın anlamı, zorunlu olup olmadığı, varsayılan değeri, kabul edilen değerler ve **Plan oluştur** sırasında görebileceğiniz hata mesajları. Dosyanın nasıl yükleneceği, planlanacağı ve uygulanacağı [Stack'ler](stacks.md) sayfasındadır.

---

## Dosya Kuralları

- Dosya tek bir YAML belgesidir ve kökü bir nesnedir. En fazla 1 MiB olabilir, iç içe en fazla 32 seviye içerebilir.
- Bir dosyada en fazla 200 kaynak olabilir: `services`, `addons`, `uses.addons`, `domains` ve `secrets` öğelerinin toplamı.
- Alan adları camelCase yazılır ve büyük/küçük harfe duyarlıdır. Tanınmayan bir alan reddedilir: `Unknown field 'repo'. Expected fields: repository, branch, root.`
- Aynı nesnede bir anahtar iki kez yazılamaz. YAML anchor/alias ve özel etiketler (`!tag`) kullanılamaz.
- Komuta'nın iç kimlikleri (GUID biçimindeki değerler) dosyaya yazılamaz; her şeye adıyla başvurun.
- `gatewayClass` alanı Komuta tarafından yönetilir ve hiçbir yerde yazılamaz.
- Her ad (`name`, `hostname`) boş olamaz ve en fazla 160 karakterdir. Aynı bölümde aynı ad iki kez kullanılamaz; bu denetim büyük/küçük harfe duyarsızdır.
- `<proje-adı>` gibi açılı ayraç içindeki değerler yer tutucu sayılır ve uyarı verir.
- Şifre, anahtar ve token gibi değerler dosyaya yazılamaz; bunlar için [`secrets`](#secrets) kullanın.

---

## Üst Düzey Alanlar

| Alan | Zorunlu | Anlamı |
|---|---|---|
| `version` | Evet | Biçim sürümü. Yalnızca `1`. |
| `name` | Evet | Stack'in adı ve kimliği. Aynı adla planlanan dosya aynı Stack'i günceller. |
| `target` | Evet | Kaynakların kurulacağı proje, bölge ve bağlantılar. |
| `apply` | Hayır | Uygulama seçenekleri. |
| `secrets` | Hayır | Dosyaya yazılmayan gizli değerlerin adları. |
| `addons` | Hayır | Stack'in oluşturup yönettiği yönetilen servisler. |
| `uses` | Hayır | Stack'in yönetmeden kullandığı mevcut yönetilen servisler. |
| `services` | Hayır | Servisler, Job'lar ve CronJob'lar. |
| `domains` | Hayır | Servislere bağlanan alan adları. |

---

## target

| Alan | Varsayılan | Anlamı |
|---|---|---|
| `project` | — | Hedef projenin adı. Zorunludur; 3–100 karakter. Proje yoksa `apply.createMissingProject` açıkken oluşturulur. |
| `region` | — | Bölgenin adı ya da kodu, ör. `de-east`. Servisler ve yönetilen servisler için gereklidir; yazılmazsa plan geçer ama bu adımlar uygulama sırasında `REGION_NOT_FOUND` ile durur. |
| `envGroup` | — | Kabul edilir; bugün oluşturulan kaynakları etkilemez. |
| `deploymentTarget` | `paas` | Komuta barındırması. Bu alanı yazmanıza gerek yoktur. |
| `connections.git` | — | **Entegrasyonlar** sayfasındaki Git bağlantınızın adı. Her servis için gereklidir. |
| `connections.registry` | — | Konteyner kayıt defteri bağlantısının adı (Komuta'nın kendi kayıt defteri dahil). Her servis için gereklidir; bulunamazsa servis adımı `REGISTRY_CONNECTION_NOT_FOUND` ile durur. |

---

## apply

| Alan | Varsayılan | Kabul edilen | Anlamı |
|---|---|---|---|
| `prune` | `false` | yalnızca `false` | Dosyadan çıkarılan kaynaklar silinmez, **Sahipsiz** olarak kalır. |
| `createMissingProject` | `true` | `true` / `false` | `false` ise proje yoksa plan engellenir. |
| `adoptExisting` | `explicit` | yalnızca `explicit` | Mevcut kaynaklar ancak plan ekranında tek tek onaylanınca devralınır. |
| `failurePolicy` | `stop-dependents` | — | Başarısız bir adıma bağlı adımlar her zaman engellenir, diğerleri sürer. Değer bugün denetlenmez ve davranışı değiştirmez. |
| `maxParallel` | `3` | 1–10 | Aynı anda çalışan en fazla adım sayısı. |

---

## secrets

Gizli değerin kendisi dosyaya yazılmaz; burada yalnızca adı ve nereden geleceği tanımlanır. Ortam değişkeni değere `valueFrom.secret` ile başvurur.

| Alan | Varsayılan | Anlamı |
|---|---|---|
| `name` | — | Zorunlu. Başvurular büyük/küçük harfe duyarlıdır. |
| `source` | `prompt` | `prompt`: tek kullanımlık güvenli bağlantıdan girilir. `generate`: Komuta uygulama sırasında üretir. `existing`: bu Stack'te daha önce saklanmış bir değer kullanılır. |
| `required` | `true` | `false` olan bir `prompt` değeri güvenli formda istenmez; bir değişken ona başvuruyorsa adım `SECRET_NOT_RESOLVED` ile durur. |
| `existingName` | — | `existing` için zorunlu: değerin bu Stack'te saklandığı ad. O adla değer yoksa secret'ın kendi adına bakılır. |
| `length` | `48` | `generate` için üretilecek değerin uzunluğu; 24–256. |

```yaml
secrets:
  - name: payment-api-key           # güvenli bağlantıdan girilir
  - name: session-secret
    source: generate
    length: 64
```

Değerler Stack'e ait şifreli depoda saklanır ve aynı Stack'in sonraki planlarında yeniden sorulmaz. Ayrıntılar [Stack'ler](stacks.md#gizli-değerler) sayfasındadır.

---

## services

Her öğe bir servis, Job ya da CronJob'dur.

| Alan | Varsayılan | Anlamı |
|---|---|---|
| `name` | — | Zorunlu. Komuta'daki servis adı olur. |
| `workload` | `service` | `service`, `job` ya da `cronjob`. |
| `source` | — | Kodun alınacağı repo ve branch. |
| `build` | — | Build ayarları. |
| `compute` | — | Paket ve replika sayısı. |
| `rollout` | — | Yayına alma stratejisi. |
| `autoDeploy` | — | Push'ta otomatik dağıtım. |
| `ports` | — | Portlar (yalnızca `service`). |
| `network` | — | Genel adres ve yollar (yalnızca `service`). |
| `env` | — | Ortam değişkenleri. |
| `storage` | — | Kalıcı depolama (yalnızca `service`). |
| `health` | — | Sağlık kontrolleri (yalnızca `service`). |
| `security` | — | Çalışma zamanı güvenlik ayarları. |
| `bandwidth` | — | Bant genişliği planı. |
| `schedule`, `execution` | — | Job ve CronJob ayarları. |
| `access` | — | Erişim koruması; bkz. [Stack Manifestinde Erişim Koruması](stack-manifest-access.md). |

### source

| Alan | Varsayılan | Anlamı |
|---|---|---|
| `repository` | — | Zorunlu. Reponun adresi, ör. `https://github.com/acme/shop.git`. |
| `branch` | — | Zorunlu. Plan bu branch'in en son commit'ini sabitler. |
| `root` | — | Servisin repodaki klasörü (monorepo). Varsayılan Dockerfile yolu ve build bağlamı buna göre belirlenir. |

### build

| Alan | Varsayılan | Anlamı |
|---|---|---|
| `strategy` | `dockerfile` | `dockerfile`: repodaki Dockerfile ile build edilir. `image`: hazır bir imaj build edilmeden dağıtılır. |
| `dockerfile` | `<root>/Dockerfile` | Dockerfile'ın repodaki yolu. `dockerfile` stratejisinde dosya push edilmiş branch'te bulunmalıdır, yoksa plan engellenir. |
| `context` | `root`, yoksa `.` | Build bağlamı. |
| `image.name` | servis adı | `dockerfile` stratejisinde build edilen imajın adı. `image` stratejisinde dağıtılacak imajın tam referansı, ör. `nginx:1.27`. |
| `image.tag` | `git-sha` | Build edilen imajın etiketi. |
| `dockerfileAutoFix` | `false` | `true` ise build başarısız olduğunda Komuta, AI ile düzeltilmiş bir Dockerfile'ı ayrı bir dalda pull request olarak açar ve build'i o dalda yeniden dener. |
| `detectedFramework` | — | Uygulamanın çatısı. Komuta, çatının çalışırken yazması gereken klasörleri açmak için kullanır. |
| `runtimeLibc` | — | Çalışma imajının C kütüphanesi: `musl` ya da `glibc`. .NET uygulamalarında otomatik izlemenin doğru bileşeni seçmesi için kullanılır. |

> **Not:** `image` stratejisinde de `source.repository`, `source.branch` ve `target.connections` alanları zorunludur.

### compute

| Alan | Varsayılan | Anlamı |
|---|---|---|
| `plan` | — | Zorunlu. Paket adı: `dz-nano` (ücretsiz paket), `dz-dev`, `dz-standard`, `dz-premium`, `dz-enterprise`. Adı küçük harfle, konsoldaki gibi yazın. |
| `replicas.mode` | `fixed` | `fixed` ya da otomatik ölçekleme için `autoscaling` (`hpa` da kabul edilir). |
| `replicas.count` | `1` | `fixed` modunda replika sayısı. |
| `replicas.min`, `replicas.max` | — | Otomatik ölçeklemenin alt ve üst sınırı. |
| `replicas.cpuTargetPercent`, `replicas.memoryTargetPercent` | `80` | Otomatik ölçekleme hedefleri. |

> **Uyarı:** Komuta barındırmasında yeni servis sabit bir replika sayısıyla oluşturulur: `fixed` modunda `count`, `autoscaling` modunda 1. `min` ve `max` oluşturma sırasında uygulanmaz; gerçek otomatik ölçekleme için servis oluştuktan sonra [Otomatik Ölçekleme](services-autoscaling.md) sayfasında üst sınırı yükseltin. Maliyet tahmini `autoscaling` modunda `min` değerini kullanır.

Ücretsiz paket (`dz-nano`) tek replikayla çalışır. `fixed` modunda `count` 1'den büyükse plan engellenir.

### rollout ve autoDeploy

| Alan | Varsayılan | Anlamı |
|---|---|---|
| `rollout.strategy` | `auto-promote` | `auto-promote`, `blue-green` ya da `canary`. Ayrıntılar [Dağıtım Stratejileri](deployment-strategies.md) sayfasındadır. Ortam değişkenlerini Stack üzerinden güncelleyebilmek için `auto-promote` gerekir. |
| `autoDeploy.enabled` | `false` | Branch'e push'ta otomatik dağıtım. `autoDeploy` bloğu hiç yazılmazsa otomatik dağıtım, Git bağlantınız push bildirimi gönderebiliyorsa açık başlar. |
| `autoDeploy.includePaths`, `autoDeploy.excludePaths` | — | Hangi yollardaki değişikliklerin dağıtımı tetikleyeceği. Bkz. [Otomatik Dağıtım](service-auto-deploy.md). |

`autoDeploy.cooldownSeconds` ve `autoDeploy.skipCommitPatterns` kabul edilir ama plan bunları reddeder.

### ports

Yalnızca `service` için. `ports` yazılmazsa servis 8080 numaralı tek bir birincil portla oluşturulur.

| Alan | Varsayılan | Anlamı |
|---|---|---|
| `name` | `http` | Portun adı. |
| `protocol` | `TCP` | Protokol. |
| `containerPort` | — | Uygulamanın dinlediği port. |
| `servicePort` | `containerPort` | Servisin dışa açtığı port. |
| `targetPort` | `containerPort` | Trafiğin yönlendirildiği port. |
| `primary` | `false` | Birincil port. Hiçbiri işaretlenmezse ilk port birincil olur. |
| `active` | `true` | Kabul edilir; oluşturmada port her zaman etkin kurulur. |

Port kavramları için [Erişim ve Portlar](services-ports.md) sayfasına bakın.

### network

| Alan | Varsayılan | Anlamı |
|---|---|---|
| `public` | `false` | `true` ise servise genel bir adres verilir. Erişim koruması için gerekir. |
| `routes[].host` | — | Rotanın alan adı. `domains` altında **aynı servise** bağlanan bir alan adı olmalıdır; değilse adım `ROUTE_HOST_NOT_OWNED` ile durur. |
| `routes[].matches[].path` | `/` | `/` ile başlayan yol. |
| `routes[].matches[].pathType` | `Prefix` | `Prefix` ya da `Exact`. |
| `meshExpose`, `dependencies` | — | Kabul edilir ama plan reddeder: `Private mesh dependencies are not enabled on this target.` |

### env

Her öğe bir ortam değişkenidir. Değer ya `value` ile doğrudan yazılır ya da `valueFrom` ile bir kaynaktan alınır; ikisi birlikte kullanılamaz.

| Alan | Varsayılan | Anlamı |
|---|---|---|
| `name` | — | Zorunlu; aynı serviste benzersiz. |
| `value` | — | Düz değer. |
| `sensitive` | `false` | Değer gizli saklanır. |
| `buildTime` | `false` | Değer build sırasında da verilir. |
| `valueFrom.secret` | — | `secrets` altındaki bir adın değeri. Her zaman gizli saklanır. |
| `valueFrom.addon` | — | Bir yönetilen servisin bağlantı bilgisi: `name`, `property`, `endpoint`. |
| `valueFrom.service` | — | Stack'teki bir servisin adresi: `name`, `property`. |

Kurallar:

- `sensitive: true` olan ya da adı kimlik bilgisi gibi okunan bir değişkene `value` yazılamaz. Adı `API_KEY`, `SECRET`, `PASSWORD`, `PRIVATE_KEY`, `ACCESS_TOKEN`, `CREDENTIALS`, `CONNECTION_STRING`, `DATABASE_URL`, `REDIS_URL` gibi bitenler her zaman gizli sayılır (alt çizgi ve büyük/küçük harf fark etmez).
- Gizli bir değer `buildTime: true` olamaz.
- `NEXT_PUBLIC_`, `NUXT_PUBLIC_`, `EXPO_PUBLIC_`, `REACT_APP_`, `GATSBY_`, `VUE_APP_`, `VITE_` ya da `PUBLIC_` ile başlayan bir değişken `buildTime: true` değilse uyarı alırsınız; build edilen ön yüz bu değeri göremez.
- Değişken ayrıntıları için [Ortam Değişkenleri](services-environment-variables.md) sayfasına bakın.

**`valueFrom.addon`** — değer, yönetilen servis hazır olduktan sonra uygulama sırasında yazılır.

| Alan | Varsayılan | Kabul edilen |
|---|---|---|
| `name` | — | `addons` ya da `uses.addons` altındaki bir ad. |
| `property` | — | `host`, `port`, `database`, `user`, `password`, `connectionString`. `password` ve `connectionString` için değişkende `sensitive: true` zorunludur. |
| `endpoint` | `private` | `private` ya da `public`. `public` için yönetilen servisin genel erişimi açık olmalıdır. RabbitMQ'da yalnızca `public` kullanılabilir. |

`connectionString` biçimi: PostgreSQL için `postgresql://kullanıcı:şifre@host:port/veritabanı?sslmode=require`, Valkey için `rediss://kullanıcı:şifre@host:port`, RabbitMQ için `amqps://kullanıcı:şifre@host:port/%2F`. Valkey'de `database` değeri `0`, RabbitMQ'da sanal sunucudur.

**`valueFrom.service`** — `property: host` servisin adresini, `property: url` ise `https://<adres>/` biçimini verir. Adres, `domains` altında o servise bağlanan ilk alan adıdır; yoksa Komuta'nın servise verdiği adrestir. Başvurulan servis önce oluşturulur; servisler birbirine döngüsel başvuramaz.

### storage

Yalnızca `service` için. Kavramlar [Kalıcı Depolama](storage.md) sayfasındadır.

| Alan | Varsayılan | Anlamı |
|---|---|---|
| `name` | — | Küçük harf, rakam ve tire; harf ya da rakamla başlayıp biter; en fazla 63 karakter. |
| `plan` | — | Depolama planının kodu ya da adı, ör. `storage-small`. |
| `mountPath` | — | Mutlak yol; `/` olamaz, `..` içeremez. |
| `subPath` | — | Göreli yol; `..` içeremez. |
| `readOnly` | `false` | Salt okunur bağlama. |
| `active` | `true` | `false` kabul edilir ama plan reddeder. |

### health

Yalnızca `service` için. `readiness`, `liveness` ve `startup` kontrollerinin her biri aynı alanları alır. Kavramlar [Sağlık Kontrolleri](services-health-probes.md) sayfasındadır.

| Alan | Varsayılan | Anlamı |
|---|---|---|
| `enabled` | `true` | Kapalı bir kontrol denetlenmez. |
| `type` | `httpGet` | `httpGet`, `tcpSocket` ya da `exec`. |
| `path` | — | `httpGet` için zorunlu. |
| `port` | — | `httpGet` ve `tcpSocket` için zorunlu; 1–65535. |
| `scheme` | `HTTP` | `HTTP` ya da `HTTPS`. |
| `command` | — | `exec` için en az bir öğe. |
| `initialDelaySeconds` | `0` | Negatif olamaz. |
| `periodSeconds` | `10` | En az 1. |
| `timeoutSeconds` | `3` | En az 1. |
| `successThreshold` | `1` | En az 1; `liveness` ve `startup` için yalnızca 1. |
| `failureThreshold` | `3` | En az 1. |

### security

Bu alanlar servis oluşturulurken uygulanır; çoğu ayrı bir izin ister ve izniniz yoksa plan engellenir. Kavramlar [Servis Güvenliği](service-security-guide.md) sayfasındadır.

| Alan | Varsayılan | Anlamı | Gereken izin |
|---|---|---|---|
| `allowRunAsRoot` | `false` | Container'ın root olarak çalışmasına izin verir. | Servis container'ının root olarak çalışmasına izin ver |
| `addCapabilities` | — | Ek Linux yetenekleri; yalnızca `CHOWN`, `FOWNER`, `FSETID`, `NET_BIND_SERVICE`, `KILL` (`CAP_` öneki kabul edilir). | Servisi Linux capability izin listesine dahil et |
| `extraWritableMounts` | — | Ek yazılabilir yollar; korunan yollar reddedilir. | Servise ek yazılabilir bağlama noktası ekle |
| `persistFrameworkKeys` | `false` | Çatının veri koruma anahtarlarını yeniden başlatmalar arasında saklar. Yalnızca `service`. | Servise ek yazılabilir bağlama noktası ekle |
| `autoInstrumentation.disabled` | `false` | `true` ise Komuta servise otomatik izleme eklemez; servis iz (trace) üretmeden çalışır. İzleme bileşeni uygulamanızı bozuyorsa kullanın. | Servise ek yazılabilir bağlama noktası ekle |
| `autoInstrumentation.pythonOptIn` | `false` | Kabul edilir ve kaydedilir; bugün etkisi yoktur. Python için ayrıca açmanız gerekmez; otomatik izlemeyi kapatmak için `disabled` kullanılır. | Servise ek yazılabilir bağlama noktası ekle |
| `networkAlerts` | `false` | Servis için ağ uyarı kurallarını açar. | — |
| `policyGuardMode` | `audit` | Yalnızca `audit` uygulanabilir; `off`, `shadow` ve `enforce` kabul edilir ama plan reddeder. | — |

Plan, bu ayarlardan izin gevşetenleri **Güvenlik gevşetmeleri** kartında gösterir.

### bandwidth

| Alan | Anlamı |
|---|---|
| `plan` | Bant genişliği planının adı: `Starter`, `Growth`, `Business`, `Enterprise`, `Scale`. Yazılmazsa servis varsayılan planla başlar. Bkz. [Bant Genişliği Planları](bandwidth-plans.md). |

### Job ve CronJob alanları

`workload: job` ya da `workload: cronjob` için. Bu iş yüklerinde `ports`, `network`, `storage`, `health` ve `compute.replicas` kullanılmaz.

| Alan | Varsayılan | Anlamı |
|---|---|---|
| `schedule.cron` | — | CronJob için zorunlu; beş alanlı cron ifadesi, ör. `0 3 * * *`. |
| `schedule.timeZone` | — | IANA saat dilimi, ör. `Europe/Istanbul`. |
| `schedule.suspended` | `false` | Zamanlamayı duraklatır. |
| `schedule.concurrencyPolicy` | — | Kabul edilir ama plan reddeder. |
| `execution.restartPolicy` | `OnFailure` | Yalnızca `OnFailure` uygulanabilir. |
| `execution.backoffLimit` | — | Yeniden deneme sayısı; 0 ya da daha büyük. |
| `execution.activeDeadlineSeconds` | — | En uzun çalışma süresi; 0'dan büyük. |
| `execution.ttlSecondsAfterFinished` | — | Bitmiş çalıştırmanın saklanma süresi; 0 ya da daha büyük. |
| `execution.command`, `execution.args` | — | İmajın giriş komutunu ve argümanlarını değiştirir. |

Cron, saat dilimi ve süre değerleri plan sırasında değil, Job oluşturulurken denetlenir; uymayan bir değer o adımı durdurur.

> **Uyarı:** Şu an Stack ile oluşturulan Job ve CronJob'ların adımı uygulama sırasında başarısız olmaktadır. Job ve CronJob'larınızı [Job ve CronJob'lar](jobs-cronjobs.md) sayfasından oluşturun.

### Henüz uygulanamayan alanlar

Şu alanlar dosyada kabul edilir ama plan `CAPABILITY_NOT_AVAILABLE` ile engellenir:

| Alan | Plan mesajı |
|---|---|
| `observability` (`slos`, `alerts`, `dashboards`) | Declarative SLO, alert and dashboard adapters are not enabled on this target. |
| `network.meshExpose`, `network.dependencies` | Private mesh dependencies are not enabled on this target. |
| `autoDeploy.cooldownSeconds`, `autoDeploy.skipCommitPatterns` | Auto-deploy cooldown and skip patterns are not enabled on this target. |
| `storage[].active: false` | Creating inactive storage attachments is not enabled on this target. |
| `security.policyGuardMode` (`audit` dışında) | Non-default Policy Guard modes require the correlated transition adapter on this target. |
| `schedule.concurrencyPolicy`, `execution.restartPolicy` (`OnFailure` dışında) | The requested job execution policy is not enabled on this target. |
| `addons[].backups` | Declarative addon backup configuration is not enabled on this target. |
| `addons[].type: opensearch` | OpenSearch is not provisionable on Komuta yet. |

---

## addons

Stack'in oluşturduğu ve yönettiği yönetilen servisler.

| Alan | Varsayılan | Anlamı |
|---|---|---|
| `name` | — | Zorunlu. Küçük harfle başlar; küçük harf, rakam ve tire; en fazla 40 karakter. Bu kural plan sırasında değil, oluşturma sırasında denetlenir. |
| `type` | — | `postgres`, `valkey` ya da `rabbitmq`. |
| `plan` | — | Zorunlu. Plan kodu ya da adı, ör. `postgres-2g`, `valkey-1g`, `rabbitmq-1broker`. |
| `region` | `target.region` | Bölgenin adı ya da kodu. |
| `topology` | planın varsayılanı | Topoloji kodu ya da adı, ör. `standalone`. |
| `version` | planın varsayılanı | Sürüm. |
| `database` | `app` | Yalnızca PostgreSQL: ilk veritabanının adı. Küçük harfle başlar; küçük harf, rakam ve alt çizgi; en fazla 63 karakter. |
| `network.access` | `private-only` | `private-only`: yalnızca Komuta içinden erişilir. `public`: genel adres açılır. RabbitMQ için `public` yazılmalıdır. `public-allowlist` kabul edilmez; IP izin listesini oluşturduktan sonra konsoldan ekleyin. |
| `network.allowedCidrs` | — | Yalnızca `public-allowlist` ile kullanılabilir. |
| `config.evictionPolicy` | `noeviction` | Yalnızca Valkey: `noeviction`, `allkeys-lru`, `volatile-lru`, `allkeys-lfu`, `volatile-lfu`, `allkeys-random`, `volatile-random`, `volatile-ttl`. |
| `config.persistenceMode` | `rdb` | Yalnızca Valkey: `rdb`, `aof` ya da `none`. `none` için `topology: standalone` açıkça yazılmalıdır. |
| `scope` | `project` | Kabul edilir; bugün etkisi yoktur. |
| `backups` | — | Kabul edilir ama plan reddeder. |

Yönetilen servisler hedef projeye bağlanır. Planlar ve kavramlar için [PostgreSQL](managed-postgresql.md), [Valkey](managed-valkey.md) ve [RabbitMQ](managed-rabbitmq.md) sayfalarına bakın.

> **Not:** Bir Stack'in yönettiği ya da bağlantı bilgisini bir Stack üzerinden servislere vermiş bir PostgreSQL'in kimlik bilgileri konsoldan yenilenemez; örneğin "Bu eklenti bir Stack tarafından yönetiliyor. Bağımlı servisler güvenli biçimde yenilenene kadar kimlik bilgisi rotasyonu kullanılamaz; hiçbir kimlik bilgisi değiştirilmedi." mesajı görünür.

---

## uses

Stack'in yönetmeden kullandığı mevcut yönetilen servisler. Plan bunları **Kullan** işlemiyle listeler; maliyet tahminine girmezler.

| Alan | Anlamı |
|---|---|
| `addons[].name` | Hedef projedeki ya da hiçbir projeye bağlı olmayan ortak bir yönetilen servisin adı. Büyük/küçük harf tam eşleşmelidir. |
| `addons[].type` | `postgres`, `valkey` ya da `rabbitmq`; mevcut servisin türüyle aynı olmalıdır. |

Kullanılan servis **Active** durumda olmalı ve silinmek üzere işaretlenmemiş olmalıdır. Aynı ad hem `addons` hem `uses.addons` altında olamaz.

---

## domains

| Alan | Varsayılan | Anlamı |
|---|---|---|
| `hostname` | — | Zorunlu. Geçerli bir alan adı. |
| `service` | — | Zorunlu. `services` altındaki bir servisin adı. Servis bu Stack'e ait olmalıdır. |
| `tls` | `managed` | Yalnızca `managed`: sertifikayı Komuta yönetir. |

Alan adı, servisi hazır olduktan sonra eklenir. DNS doğrulamasını [Özel Alan Adları](custom-domains.md) sayfasındaki gibi tamamlayın.

---

## Doğrulama Mesajları

**Plan oluştur** önce dosyayı denetler. Her sorun kodu, dosyadaki yolu ve mesajıyla listelenir. Mesajlar Komuta'nın döndürdüğü haliyle İngilizce gösterilir.

### Dosya yapısı

| Kod | Mesaj |
|---|---|
| `MANIFEST_REQUIRED` | Manifest is required. |
| `MANIFEST_TOO_LARGE` | Manifest exceeds 1048576 bytes. |
| `INVALID_YAML` | Invalid YAML or field value at line …, column …. Check the field type and manifest schema. |
| `MULTI_DOCUMENT_NOT_ALLOWED` | Exactly one YAML document is allowed. |
| `ROOT_OBJECT_REQUIRED` | The manifest root must be an object. |
| `MAX_DEPTH_EXCEEDED` | Manifest nesting cannot exceed 32. |
| `YAML_ANCHOR_NOT_ALLOWED` | YAML anchors and aliases are not allowed. |
| `YAML_CUSTOM_TAG_NOT_ALLOWED` | Custom YAML tags are not allowed. |
| `DUPLICATE_KEY` | Duplicate key '…' is not allowed. |
| `UNKNOWN_OR_INVALID_FIELD` | Unknown field '…'. Expected fields: …. |
| `OBJECT_REQUIRED` | A manifest object is required here. |
| `FIELD_PLATFORM_MANAGED` | gatewayClass is managed by Komuta and must not be declared. |
| `PLATFORM_ID_NOT_ALLOWED` | Platform identifiers must not be stored in the manifest; use resource names. |
| `SCHEMA_VERSION_UNSUPPORTED` | Only manifest version 1 is supported. |
| `RESOURCE_LIMIT_EXCEEDED` | A manifest can contain at most 200 resources. |
| `NAME_REQUIRED` | A non-empty name is required. |
| `NAME_TOO_LONG` | A name cannot exceed 160 characters. |
| `DUPLICATE_RESOURCE_NAME` | Resource name '…' is duplicated. |

### Alanlar

| Kod | Mesaj |
|---|---|
| `PRUNE_NOT_SUPPORTED` | prune must be false in version 1. |
| `ADOPTION_MUST_BE_EXPLICIT` | adoptExisting must be explicit. |
| `MAX_PARALLEL_INVALID` | maxParallel must be between 1 and 10. |
| `SECRET_SOURCE_INVALID` | Secret source must be prompt, existing or generate. |
| `EXISTING_SECRET_NAME_REQUIRED` | existingName is required for existing secrets. |
| `GENERATED_SECRET_LENGTH_INVALID` | Generated secret length must be between 24 and 256 when specified. |
| `ADDON_TYPE_INVALID` | Addon type must be postgres, valkey, rabbitmq or opensearch. |
| `ADDON_PLAN_REQUIRED` | Addon plan is required. |
| `ADDON_NETWORK_ACCESS_INVALID` | Addon network access must be private-only, public or public-allowlist. |
| `ADDON_NETWORK_PRIVATE_UNSUPPORTED` | RabbitMQ does not support private-only access yet; use public. |
| `ADDON_NETWORK_ALLOWLIST_UNSUPPORTED` | Initial public-allowlist enforcement is not supported by Stack creates; create with public access and restrict it through the addon API. |
| `ADDON_NETWORK_ALLOWLIST_UNEXPECTED` | allowedCidrs can only be used with public-allowlist access. |
| `ADDON_EVICTION_POLICY_INVALID` | Valkey evictionPolicy must be one of: noeviction, allkeys-lru, … |
| `ADDON_PERSISTENCE_MODE_INVALID` | Valkey persistenceMode must be one of: rdb, aof, none. |
| `ADDON_PERSISTENCE_REQUIRES_STANDALONE` | Valkey persistenceMode 'none' requires an explicit `topology: standalone`; replication requires persistence. |
| `USE_TYPE_INVALID` | A used addon's type must be postgres, valkey or rabbitmq. |
| `USE_ADOPT_CONFLICT` | Addon '…' is declared under both addons and uses.addons; a Stack either manages an addon or uses it. |
| `WORKLOAD_INVALID` | Workload must be service, job or cronjob. |
| `REPOSITORY_REQUIRED` | Repository is required. |
| `BRANCH_REQUIRED` | Branch is required. |
| `COMPUTE_PLAN_REQUIRED` | Compute plan is required. |
| `ROLLOUT_STRATEGY_INVALID` | Rollout strategy must be canary, blue-green or auto-promote. |
| `BANDWIDTH_PLAN_REQUIRED` | Bandwidth plan is required. |
| `STORAGE_WORKLOAD_UNSUPPORTED` | Persistent storage is supported only for service workloads. |
| `HEALTH_WORKLOAD_UNSUPPORTED` | Health probes are supported only for service workloads. |
| `ROUTING_WORKLOAD_UNSUPPORTED` | Routing is supported only for service workloads. |
| `FRAMEWORK_KEYS_WORKLOAD_UNSUPPORTED` | Framework key persistence is supported only for service workloads. |
| `STORAGE_NAME_INVALID` | Storage name must be a DNS-1123 label of at most 63 characters. |
| `STORAGE_PLAN_REQUIRED` | Storage plan is required. |
| `STORAGE_MOUNT_PATH_INVALID` | Storage mountPath must be an absolute, non-root path without parent traversal. |
| `STORAGE_SUB_PATH_INVALID` | Storage subPath must be relative and cannot contain parent traversal. |
| `PROBE_TYPE_INVALID` | Probe type must be httpGet, tcpSocket or exec. |
| `PROBE_TIMING_INVALID` | Probe initialDelaySeconds cannot be negative and all thresholds/periods must be at least 1. |
| `PROBE_SUCCESS_THRESHOLD_INVALID` | Liveness and startup probes require successThreshold: 1. |
| `HTTP_PROBE_TARGET_REQUIRED` | An enabled httpGet probe requires a path and a port between 1 and 65535. |
| `HTTP_PROBE_SCHEME_INVALID` | HTTP probe scheme must be HTTP or HTTPS. |
| `TCP_PROBE_PORT_REQUIRED` | An enabled tcpSocket probe requires a port between 1 and 65535. |
| `EXEC_PROBE_COMMAND_REQUIRED` | An enabled exec probe requires at least one command item. |
| `POLICY_GUARD_MODE_INVALID` | policyGuardMode must be off, shadow, audit or enforce. |
| `SECURITY_CAPABILITY_NOT_ALLOWED` | Capability '…' is not in the platform allowlist. |
| `SECURITY_WRITABLE_MOUNT_NOT_ALLOWED` | Writable mount '…' is invalid or targets a protected path. |
| `ROUTE_HOST_REQUIRED` | Route host is required. |
| `ROUTE_PATH_INVALID` | Route path must start with '/'. |
| `ROUTE_PATH_TYPE_INVALID` | Route pathType must be Prefix or Exact. |
| `ENV_NAME_REQUIRED` | Environment variable name is required. |
| `SECRET_VALUE_NOT_ALLOWED` | Sensitive values cannot be stored in the manifest. |
| `ENV_VALUE_CONFLICT` | Use either value or valueFrom, not both. |
| `BUILD_TIME_SECRET_NOT_ALLOWED` | A sensitive value cannot be made available at build time; keep it runtime-only. … |
| `VALUE_FROM_INVALID` | valueFrom must reference exactly one secret, addon or service. |
| `SECRET_REFERENCE_NOT_FOUND` | Secret '…' does not exist. |
| `SECRET_REFERENCE_CASE_MISMATCH` | Secret '…' does not exist; secret names are case-sensitive and the declared name is '…'. |
| `ADDON_REFERENCE_NOT_FOUND` | Addon '…' is not declared under addons or uses.addons. |
| `ADDON_REFERENCE_PROPERTY_INVALID` | Addon property must be host, port, database, user, password or connectionString. |
| `ADDON_REFERENCE_ENDPOINT_INVALID` | Addon endpoint must be private or public. |
| `ADDON_PUBLIC_ENDPOINT_NOT_DECLARED` | Addon '…' does not declare public network access. |
| `ADDON_PASSWORD_MUST_BE_SENSITIVE` | An addon password or connectionString binding must declare sensitive: true. |
| `SERVICE_REFERENCE_NOT_FOUND` | Service '…' does not exist. |
| `SERVICE_REFERENCE_PROPERTY_INVALID` | Service property must be host or url. |
| `SERVICE_DEPENDENCY_CYCLE` | Service dependencies must not contain a cycle. |
| `DOMAIN_SERVICE_NOT_FOUND` | Service '…' does not exist. |
| `DOMAIN_TLS_INVALID` | Only managed TLS is supported. |

`access` bloğunun mesajları [Stack Manifestinde Erişim Koruması](stack-manifest-access.md#doğrulama) sayfasındadır.

### Plan sırasında

Dosya geçerliyse plan, hesabınızdaki kaynaklarla karşılaştırma yaparken şu sorunları da gösterebilir:

| Kod | Mesaj |
|---|---|
| `PROJECT_NOT_FOUND` | Project '…' does not exist and apply.createMissingProject is false. |
| `GIT_CONNECTION_REQUIRED` | A Git connection is required to verify source-built services. |
| `GIT_CONNECTION_NOT_FOUND` | Git connection '…' was not found. |
| `SOURCE_CHANGE_REQUIRED` | Dockerfile '…' does not exist in pushed branch '…'. Create, test, commit and push it before planning. |
| `SOURCE_REVISION_VERIFICATION_FAILED` | The remote source revision could not be verified. |
| `STORAGE_PLAN_NOT_FOUND` | Storage plan '…' is unavailable. |
| `BANDWIDTH_PLAN_NOT_FOUND` | Bandwidth plan '…' is unavailable. |
| `PRICE_UNAVAILABLE` | A required resource price is unavailable. Refresh pricing and create a new plan; unknown cost is not free. |
| `PERMISSION_REQUIRED` | Permission '…' is required for this action. |
| `Komuta:Sleep:FreePlanSingleReplica` | The free compute plan runs a single replica without autoscaling; choose a paid plan to scale out. |
| `ADOPTION_CONFIRMATION_REQUIRED` | Existing resource '…' must be explicitly adopted. |
| `ADOPTION_NAME_CASE_MISMATCH` | The existing … is named '…', which differs from '…' only by letter case. … |
| `ADOPTION_TYPE_MISMATCH` / `ADOPTION_PLAN_MISMATCH` / `ADOPTION_NETWORK_ACCESS_MISMATCH` | Devralınacak yönetilen servisin türü, planı ya da ağ erişimi dosyadakiyle aynı değil; mesaj mevcut değeri söyler. |
| `ADOPTION_SHARED_ADDON_IN_USE` | Shared addon '…' is used by … service(s) outside the target project; … Use it with uses.addons instead. |
| `OWNERSHIP_CONFLICT` | The … '…' is already managed by Stack '…'. A resource has exactly one owning Stack, so this Stack cannot adopt it. |
| `USE_ADDON_NOT_FOUND` | No addon named '…' exists in the target project or among the shared addons. |
| `USE_OUTSIDE_TARGET` | Addon '…' belongs to project '…', not the target project; … |
| `USE_ADDON_NOT_ACTIVE` | Addon '…' is …; it must be Active before a service can use it. |
| `DOMAIN_HOSTNAME_INVALID` | Hostname '…' is not a valid domain name. |
| `CAPABILITY_NOT_AVAILABLE` | Bkz. [Henüz uygulanamayan alanlar](#henüz-uygulanamayan-alanlar). Var olan bir kaynağın desteklenmeyen bir alanı değiştiğinde: In-place updates for Stack-managed … resources are not enabled on this target. |

Hedef proje yazılmamışsa "Hedef proje zorunludur.", yazılan bölge bulunamazsa "Hedef bölge bulunamadı." mesajıyla plan hiç oluşturulmaz.

### Uyarılar

Uyarılar planı engellemez.

| Kod | Mesaj |
|---|---|
| `PLACEHOLDER_VALUE` | This value still looks like a template placeholder and must be reviewed before production use. |
| `BUILD_TIME_PREFIX_NOT_STAGED` | This name is usually read by the frontend build; a new service needs buildTime: true or the built app will not see it. … |
| `VALUE_MATERIALIZED_AT_APPLY` | Addon property '…' will be resolved securely during apply after '…' is Active. (Servis başvurularında: Service property '…' will be resolved during apply from the custom domain or assigned platform address.) |

---

## Örnekler

### Tam örnek

Bir API, bir web ön yüzü, PostgreSQL ve Valkey ile bir alan adı. Bağlantı adlarını **Entegrasyonlar** sayfasındaki kendi bağlantılarınızla değiştirin.

```yaml copy
version: 1
name: shop

target:
  project: shop
  region: de-east
  connections:
    git: my-github
    registry: my-registry

secrets:
  - name: payment-api-key
  - name: session-secret
    source: generate
    length: 64

addons:
  - name: shop-db
    type: postgres
    plan: postgres-2g
    database: shop
  - name: shop-cache
    type: valkey
    plan: valkey-1g
    config:
      evictionPolicy: allkeys-lru

services:
  - name: api
    source:
      repository: https://github.com/acme/shop.git
      branch: main
      root: api
    compute:
      plan: dz-dev
      replicas:
        mode: fixed
        count: 2
    ports:
      - name: http
        containerPort: 8080
        primary: true
    network:
      public: true
    health:
      readiness:
        type: httpGet
        path: /healthz
        port: 8080
      liveness:
        type: httpGet
        path: /healthz
        port: 8080
        initialDelaySeconds: 10
    env:
      - name: LOG_LEVEL
        value: info
      - name: DATABASE_URL
        sensitive: true
        valueFrom:
          addon: { name: shop-db, property: connectionString }
      - name: REDIS_HOST
        valueFrom:
          addon: { name: shop-cache, property: host }
      - name: REDIS_PASSWORD
        sensitive: true
        valueFrom:
          addon: { name: shop-cache, property: password }
      - name: PAYMENT_API_KEY
        valueFrom: { secret: payment-api-key }
      - name: SESSION_SECRET
        valueFrom: { secret: session-secret }

  - name: web
    source:
      repository: https://github.com/acme/shop.git
      branch: main
      root: web
    compute:
      plan: dz-dev
    ports:
      - containerPort: 3000
    network:
      public: true
    env:
      - name: NEXT_PUBLIC_API_URL
        buildTime: true
        valueFrom:
          service: { name: api, property: url }

domains:
  - hostname: shop.example.com
    service: web
```

Bu dosyanın planı iki yönetilen servis, iki servis ve bir alan adı oluşturur; `shop` projesi yoksa önce onu oluşturur. `api` ve `web` için `api/Dockerfile` ve `web/Dockerfile` push edilmiş `main` branch'inde bulunmalıdır. `payment-api-key` değeri güvenli bağlantıdan istenir; `session-secret` uygulama sırasında üretilir. `web`, `api`'nin adresini build sırasında alır; bu yüzden `api`'den sonra oluşturulur.

### En küçük Stack

Ücretsiz paketle tek bir servis:

```yaml copy
version: 1
name: hello

target:
  project: hello
  region: de-east
  connections:
    git: my-github
    registry: my-registry

services:
  - name: hello
    source:
      repository: https://github.com/acme/hello.git
      branch: main
    compute:
      plan: dz-nano
    ports:
      - containerPort: 8080
    network:
      public: true
```

### Mevcut bir veritabanını kullanmak

Projede zaten çalışan `main-db` adlı PostgreSQL'i yönetmeden kullanan bir servis. Plan `main-db` için **Kullan** işlemi gösterir. Servis yönetilen servise özel adresinden bağlanır (`endpoint: private`); yönetilen servisin özel adresi yoksa adım `ADDON_ENDPOINT_NOT_AVAILABLE` ile durur, bu durumda `endpoint: public` yazın.

```yaml copy
version: 1
name: reports

target:
  project: shop
  region: de-east
  connections:
    git: my-github
    registry: my-registry

uses:
  addons:
    - name: main-db
      type: postgres

services:
  - name: reports
    source:
      repository: https://github.com/acme/reports.git
      branch: main
    compute:
      plan: dz-dev
    env:
      - name: DATABASE_URL
        sensitive: true
        valueFrom:
          addon: { name: main-db, property: connectionString }
```

---

## İlgili Dokümanlar

- [Stack'ler](stacks.md) — dosyayı yükleme, planlama, uygulama ve izleme.
- [Stack Manifestinde Erişim Koruması](stack-manifest-access.md) — `access` bloğu.
- [Ortam Değişkenleri](services-environment-variables.md) — gizli ve build türündeki değişkenler.
- [PostgreSQL](managed-postgresql.md), [Valkey](managed-valkey.md), [RabbitMQ](managed-rabbitmq.md) — yönetilen servis planları.
- [Fiyatlandırma](https://www.komuta.io/pricing) — paket ve plan fiyatları.
