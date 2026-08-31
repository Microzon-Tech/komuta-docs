# Bağlantılar

Bağlantılar, bir uygulamanın başka bir uygulamanıza özel ağ üzerinden erişmesini sağlar. Trafik internete çıkmaz, iki uygulama doğrudan birbirine ulaşır — bunun için public URL tanımlamaya gerek kalmaz.

![alt text](https://cdn.komuta.io/docs/tr/images/connections/services-connections.png)

---

## Uygulama Bağlama

**Uygulama bağla** ile erişilecek uygulama seçilir. Bağlantı kurulduğunda Komuta o uygulamaya özel bir adres üretir ve listede gösterir; bağlantının kurulması birkaç dakika sürebilir.

Üretilen adresin kullanılabilmesi için uygulamanın bu adresi okuması gerekir. Listedeki **Ortam değişkeni olarak ekle** işlemi adresi servisin ortam değişkenlerine yazar.

![alt text](https://cdn.komuta.io/docs/tr/images/connections/services-connections-list.png)

---

## Bağlanan ve Bağlanılan Uygulamalar

Liste iki yönü ayrı gösterir: bu servisin **bağlandığı** uygulamalar ve bu servise **bağlanan** uygulamalar. Bağlantı tek yönlüdür — bir uygulamayı buradan bağlamak, o uygulamanın da size erişebileceği anlamına gelmez.

> **Not:** Bir servise hangi servislerin bağlandığı henüz listelenemiyor. O yönü görmek için ilgili servisin kendi bağlantı sayfasına bakılır.

---

## İlgili Dokümanlar

- [Ortam Değişkenleri](environment-variables.md) — Bağlantı adresinin değişken olarak tanımlanması.
- [Portlar](ports.md) — Servisin dinlediği portlar ve özel ağ ayarı.
