# CI'dan Deploy Tetikleme

Kendi CI hattınız (GitHub Actions, GitLab CI, Jenkins, Azure Pipelines, Bitbucket…) testlerini ve onaylarını bitirdikten sonra son adımda Komuta'yı çağırabilir. Komuta gönderilen branch'i derleyip yayına alır; CI de deploy'un sonucunu bekleyip kendi işini başarılı ya da başarısız sayar.

Bu yol **dağıtım token'ı** ile çalışır. Dağıtım token'ı tek bir servise bağlıdır, yalnızca izin verdiğiniz işlemleri yapabilir ve hesabınızın genel API anahtarlarıyla karıştırılamaz.

---

## Dağıtım Token'ı Oluşturma

1. Servisin **Otomatik dağıtım** sayfasını açın ve **CI/CD entegrasyonu** sekmesine geçin.
2. **Token oluştur**'a tıklayın ve token'ı tanımlayın:
   - **Ad:** Token'ın nerede kullanıldığını anlatan bir ad (ör. `github-actions-main`).
   - **İzin verilen işlemler:** Derleyip dağıtma, hazır imaj dağıtma ya da ikisi.
   - **İzin verilen dallar ve etiketler:** Token'ın hangi branch ve tag'leri deploy edebileceği (ör. `main`, `release/*`). Boş bırakılırsa yalnızca servisin takip ettiği branch.
   - **İzin verilen imaj depoları:** Hazır imaj dağıtımında kabul edilen depolar. Boş bırakılırsa yalnızca servisin kendi imaj deposu.
   - **IP izin listesi:** İsteklerin gelebileceği adresler (CIDR). Boşsa her adresten kabul edilir.
   - **Geçerlilik süresi:** Varsayılan 365 gün, en fazla 730 gün.
3. Token **yalnızca bir kez** gösterilir. Kopyalayıp CI'nızın gizli değişkenlerine (ör. GitHub'da `KOMUTA_DEPLOY_TOKEN` secret'ı) kaydedin.

Token'lar `kmtd_` ile başlar. Bir serviste aynı anda en fazla 10 aktif token olabilir. Token'ları servis üzerinde düzenleme yetkisi olan kullanıcılar oluşturup iptal edebilir.

> **Push ile otomatik dağıtım açıksa:** Servis branch'ine yapılan her push zaten bir build başlatır. CI'nız da Komuta'yı çağırırsa her push iki kez derlenir. CI üzerinden dağıtım çalışmaya başlayınca **Push ile otomatik dağıtım** sekmesinden otomatik dağıtımı kapatın.

---

## Sırsız Dağıtım (OIDC)

GitHub Actions ve GitLab CI kullanıyorsanız CI'nızda hiç sır saklamadan dağıtım yapabilirsiniz. İş, CI sağlayıcısının imzaladığı kısa ömürlü bir kimlik token'ı (OIDC) sunar; Komuta imzayı sağlayıcının açık anahtarlarıyla doğrular ve isteği yalnızca güvendiğiniz depodan geliyorsa kabul eder.

1. **Token oluştur**'da **CI'nız nasıl giriş yapacak** alanından **GitHub Actions OIDC** ya da **GitLab CI OIDC**'yi seçin.
2. Depoyu girin: GitHub için `sahip/depo`, GitLab için `grup/proje`. İsterseniz bir **ortam** (ör. `production`) ekleyin; o zaman yalnızca bu ortamı kullanan işler dağıtım yapabilir.
3. İzin verilen işlemler, dallar, imaj depoları ve IP listesi gizli token'larda olduğu gibi uygulanır. İşin çalıştığı dal da bu kurala tabidir: dal deseni girmediyseniz yalnızca servisin dalında çalışan işler, girdiyseniz desenlere uyan dallarda çalışan işler dağıtım yapabilir.
4. Kod parçalarında **OIDC (sırsız)** seçeneğini açın.

GitHub Actions işi `permissions: id-token: write` ister; GitLab CI işinde `id_tokens` altında `aud: https://api.komuta.io` olan bir `KOMUTA_ID_TOKEN` tanımlanır. Bu modda saklanacak, sızabilecek ya da yenilenmesi gereken bir sır yoktur.

---

## Hazır Kod Parçaları

**CI/CD entegrasyonu** sekmesi GitHub Actions, GitLab CI, Jenkins, Azure Pipelines, Bitbucket Pipelines ve `curl` için servisinize göre doldurulmuş, kopyalanmaya hazır örnekler verir. Her örnek deploy'u başlatır, sonucu bekler ve deploy başarısız olursa CI işini başarısız sayar. Sekmenin üstündeki **Derleyip dağıt / Hazır imaj dağıt** seçimiyle örnekler hazır imaj moduna geçer; bu modda betik, push ettiğiniz imajı digest ile `IMAGE` değişkeninden okur.

GitHub Actions için en kısa örnek. Bu örnek deploy'u yalnızca **başlatır**, sonucunu beklemez; Komuta isteği kabul ettiği anda CI işi başarılı biter:

```yaml copy
name: Deploy to Komuta

on:
  push:
    branches: ["main"]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy with Komuta
        env:
          KOMUTA_DEPLOY_TOKEN: ${{ secrets.KOMUTA_DEPLOY_TOKEN }}
        run: |
          curl -sSf -X POST \
            "https://api.komuta.io/api/v1/deploy-hooks/services/<SERVIS_ID>/deployments" \
            -H "Authorization: Bearer $KOMUTA_DEPLOY_TOKEN" \
            -H "Content-Type: application/json" \
            -d '{"mode":"build","ref":"${{ github.ref_name }}","clientRequestId":"${{ github.run_id }}-${{ github.run_attempt }}"}'
```

Sonucu bekleyen tam sürümü **CI/CD entegrasyonu** sekmesinden kopyalayın.

---

## Hazır Araçlar

Kod parçalarını kopyalamak yerine resmi aracı da kullanabilirsiniz ([Microzon-Tech/komuta-deploy-action](https://github.com/Microzon-Tech/komuta-deploy-action)):

- **GitHub Actions:** `uses: Microzon-Tech/komuta-deploy-action@v1` (`token` ve `service-id` girdileriyle; `token` boş bırakılırsa OIDC kullanılır).
- **GitLab CI:** depodaki `gitlab/komuta-deploy.gitlab-ci.yml` şablonunu `include` edip `.komuta-deploy` işini genişletin.
- **Herhangi bir CI:** `komuta-deploy.sh --service <SERVIS_ID> --ref main --wait`. `bash`, `curl` ve `jq` ister; deploy başarısız olursa sıfırdan farklı bir kodla çıkar.

---

## Deploy İsteği

```http
POST https://api.komuta.io/api/v1/deploy-hooks/services/{serviceId}/deployments
Authorization: Bearer kmtd_...
Content-Type: application/json
```

```json
{
  "mode": "build",
  "ref": "main",
  "commitSha": "9f1c2ab...",
  "clientRequestId": "gh-123456-1",
  "metadata": {
    "ciProvider": "github-actions",
    "runUrl": "https://github.com/org/repo/actions/runs/123456",
    "actor": "alice",
    "commitMessage": "Fix login redirect"
  }
}
```

| Alan | Açıklama |
|------|----------|
| `mode` | `build`: Komuta branch'i derleyip yayınlar. `image`: sizin derleyip push ettiğiniz imajı yayınlar. |
| `ref` | Derlenecek branch ya da tag. Token'ın branch desenlerine uymalıdır. |
| `commitSha` | İsteğe bağlı. Verilirse tam o commit derlenir. Commit sabitleme platformda etkin değilse istek `400` ile döner; bu durumda alanı göndermeyin, branch'in en güncel commit'i derlenir. |
| `image` | `image` modunda zorunlu. Imaj digest ile verilmelidir (`registry.example.com/app@sha256:...`). |
| `clientRequestId` | Tekrar güvenliği için anahtar. Aynı token ve aynı gövdeyle tekrar gönderilirse yeni deploy açılmaz, aynı deploy döner. Aynı anahtar farklı bir gövdeyle gelirse `409`. |
| `metadata` | İsteğe bağlı. Konsolda deploy'un hangi CI çalışmasından geldiğini göstermek için kullanılır. |

Komuta isteği kabul edince `202 Accepted` döner:

```json
{
  "deploymentId": "3a2f...",
  "status": "queued",
  "queue": { "position": 1, "reason": "PreviousBuildRunning", "estimatedStartSeconds": 240 },
  "statusUrl": "https://api.komuta.io/api/v1/deploy-hooks/deployments/3a2f...",
  "consoleUrl": "https://console.komuta.io/services/.../pipelines"
}
```

`queue`, build kuyrukta bekliyorsa sıranızı, bekleme nedenini ve tahmini başlama süresini taşır. Ayrıntı için [Build Kuyruğu](build-queue.md).

---

## Sonucu Bekleme

`statusUrl`'i aynı token ile sorgulayın:

```http
GET https://api.komuta.io/api/v1/deploy-hooks/deployments/{deploymentId}
Authorization: Bearer kmtd_...
```

| `status` | Anlamı | CI ne yapmalı |
|----------|--------|---------------|
| `queued` | Build kuyrukta. | Beklemeye devam. |
| `building` | Build çalışıyor. | Beklemeye devam. |
| `deploying` | Yeni sürüm yayına alınıyor. | Beklemeye devam. |
| `succeeded` | Yeni sürüm yayında. | İşi başarılı bitir. |
| `failed` | Build ya da yayın başarısız. `failureReason` nedeni taşır. | İşi başarısız bitir. |
| `cancelled` | Deploy iptal edildi. | İşi başarısız bitir. |
| `superseded` | Aynı servis için daha yeni bir deploy bunun yerini aldı. `supersededBy` yeni deploy'u gösterir. | Genellikle başarılı sayılır. |

Deploy kayıtları 180 gün saklanır; bundan eski bir deploy'un `statusUrl`'i `404` döner.

Kuyruktaki ya da çalışan bir deploy'u iptal etmek için:

```http
POST https://api.komuta.io/api/v1/deploy-hooks/deployments/{deploymentId}/cancel
Authorization: Bearer kmtd_...
```

---

## İmzalı İmaj Zorunluluğu

Hazır imaj (`image` modu) dağıtıyorsanız, servise yalnızca **sizin anahtarınızla imzalanmış** imajların dağıtılmasını zorunlu kılabilirsiniz. Ayar varsayılan olarak kapalıdır.

1. Bir anahtar çifti oluşturun: `cosign generate-key-pair`. Özel anahtarı (`cosign.key`) CI'nızın gizli değişkenlerinde saklayın; Komuta'ya **asla** vermeyin.
2. CI'da imajı push ettikten sonra digest'ini imzalayın: `cosign sign --key cosign.key registry.example.com/app@sha256:...`
3. Servisin **Otomatik dağıtım → CI/CD entegrasyonu** sekmesindeki **İmzalı imajlar** kartına açık anahtarı (`cosign.pub`, `-----BEGIN PUBLIC KEY-----` ile başlar) yapıştırın ve ayarı açın.

Ayar açıkken Komuta her `image` modu isteğinde imzayı doğrular. İmzasız ya da başka bir anahtarla imzalanmış imaj `403` ile reddedilir ve denetim kaydına geçer. İmza o an doğrulanamazsa (ör. imaj deposuna erişilemiyor) istek `503` döner; kısa süre sonra tekrar deneyin. ECDSA (cosign varsayılanı) ve en az 2048 bit RSA anahtarları kabul edilir.

---

## Hata Kodları

| HTTP | Ne zaman |
|------|----------|
| `400` | İstek geçersiz (eksik alan, tag'li imaj, sabitleme kapalıyken `commitSha`). |
| `401` | Token geçersiz, iptal edilmiş ya da süresi dolmuş. |
| `402` | Hesabın ödemesi askıda. |
| `403` | İşlem, branch, imaj deposu ya da IP adresi token'ın izinleri dışında; ya da imzalı imaj zorunluyken imaj sizin anahtarınızla imzalanmamış. |
| `404` | Servis bu token'a ait değil ya da deploy bulunamadı. |
| `409` | Aynı `clientRequestId` farklı bir gövdeyle gönderildi ya da deploy artık iptal edilemiyor. |
| `429` | İstek sınırı aşıldı. `Retry-After` başlığındaki süre kadar bekleyip tekrar deneyin. |
| `503` | Deploy şu an başlatılamadı; kısa süre sonra tekrar deneyin. |

İstek sınırları: token başına dakikada 30 deploy ve 300 durum sorgusu; servis başına saatte 60 deploy.

---

## Güvenlik

- Token yalnızca bağlı olduğu serviste ve izin verdiğiniz branch'lerde çalışır. Sızan bir token başka servislere ya da hesabınızın diğer işlemlerine dokunamaz.
- Token sırrı Komuta'da saklanmaz; yalnızca oluşturulduğu anda gösterilir. Kaybederseniz token'ı iptal edip yenisini oluşturun.
- Token'ın son kullanıldığı zaman, IP adresi ve CI sağlayıcısı token listesinde görünür. Token oluşturma, iptal, her deploy ve reddedilen istekler denetim kaydına geçer.
- Token'ın süresi dolmadan 7 gün ve 1 gün önce **Deploy token süresi dolmak üzere** bildirimi gönderilir.

---

## İlgili Dokümanlar

- [Otomatik Dağıtım](service-auto-deploy.md) — Push ile otomatik dağıtım kuralları.
- [Build Kuyruğu](build-queue.md) — Deploy'un ne zaman başlayacağı.
- [Pipeline'lar](service-pipeline-guide.md) — Build aşamaları ve loglar.
