# Ortam Değişkenleri

Ortam değişkenleri, uygulamanızın çalışırken okuduğu yapılandırma değerleridir — veritabanı adresi, API anahtarı, log seviyesi gibi. Kodun içine yazılmadıkları için ortamdan ortama farklı olabilirler.

Değişken eklendikten, düzenlendikten veya silindikten sonra **Değişiklikleri Uygula** ile çalışma zamanına aktarılır.

![alt text](https://cdn.komuta.io/docs/tr/images/enviroment-variables/services-enviroment-variables-page.png)
---

## Değişken Türleri

Her değişkenin bir türü vardır ve bu tür, değerin nerede görünür olacağını belirler.

| Tür | Ne anlama gelir |
|---|---|
| **Düz** | Değer açık metin olarak saklanır ve arayüzde görünür. |
| **Gizli** | Değer şifreli saklanır, arayüzde gizlenir ve sunucudan dışarı çıkmaz. |
| **Build** | Değer yalnızca çalışan container'a değil, derleme aşamasına da verilir. |

**Build** türü, değeri derleme sırasında okunan değişkenler için gerekir. Kapalıyken değer yalnızca çalışma zamanında var olur; derleme sırasında okunmak istenirse boş görünür. Bilinen bazı önekleri (`VITE_`, `NEXT_PUBLIC_`, `REACT_APP_` gibi) taşıyan bir anahtar yazıldığında bu seçenek işaretli gelir.

> **Uyarı:** Build türündeki bir değer imaja, public önekli ise tarayıcı paketine de gömülür. Gizli kalması gereken bilgileri bu türde tanımlamayın.

![alt text](https://cdn.komuta.io/docs/tr/images/enviroment-variables/services-enviroment-variables-types.png)

---

## Toplu İçe Aktarma

Çok sayıda değişkeni tek seferde eklemek için mevcut yapılandırma dosyanızın içeriği yapıştırılır. Üç format desteklenir: `.env`, düz JSON ve iç içe appsettings JSON. Format yapıştırılan içerikten otomatik tespit edilir.

---

## Dışa Aktarma

Değişkenler `.env`, JSON, YAML, CSV, shell ve Kubernetes manifest formatlarında indirilebilir ya da panoya kopyalanabilir.

> **Not:** Gizli değerler şifreli saklandığı ve sunucudan çıkmadığı için dışa aktarımda boş gelir.

---

## İlgili Dokümanlar

- [Servis Dashboard](service-dashboard.md) — Servisin genel durumu ve deploy işlemleri.
- [Pipeline'lar](pipeline-guide.md) — Build aşamasının nasıl işlediği.
