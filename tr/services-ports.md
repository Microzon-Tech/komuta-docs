# Portlar

Portlar, servisinize trafiğin hangi kapıdan gireceğini ve uygulamanın içindeki hangi porta yönleneceğini belirler. Bir servis dışarıdan istek alacaksa en az bir port tanımlı olmalıdır.

![alt text](https://cdn.komuta.io/docs/tr/images/ports/services-ports-page.png)
---

## Port Tanımı

Her port kaydında girilen değerler:

- **Ad** — portu ayırt etmek için kullanılan etiket (`http`, `grpc` gibi).
- **Port** — servise dışarıdan gelen isteğin ulaştığı numara.
- **Hedef port** — uygulamanın container içinde dinlediği numara. İkisi farklı olabilir; trafik porttan hedef porta yönlendirilir.
- **Protokol** — TCP, UDP veya SCTP.
- **Birincil** — servisin varsayılan giriş noktası. Domain bağlandığında trafik bu porta gider.
- **Aktif** — kapalıyken port tanımlı kalır ama uygulamaya bağlanmaz.

Port ve hedef port 1 ile 65535 arasında bir değer alır.

![alt text](https://cdn.komuta.io/docs/tr/images/ports/services-ports-add-port.png)

---

## Özel Ağ (Mesh)

Servis, özel ağ üzerinden diğer cluster'larınıza açılabilir. Bu durumda uygulamalar birbirine public ingress olmadan, doğrudan erişir — internete çıkmayan servisler arası iletişim için kullanılır.

Açıldıktan sonra hangi cluster'ların erişebildiği listelenir; bağlantı kurulumu birkaç dakika sürebilir.

![alt text](https://cdn.komuta.io/docs/tr/images/ports/services-ports-private-mesh.png)

---

## İlgili Dokümanlar

- [Ingress ve Domainler](ingress-domains.md) — Dış dünyaya açılan adreslerin yönetimi.
- [Servis Dashboard](service-dashboard.md) — Trafiğin ingress'ten pod'lara akışı.
