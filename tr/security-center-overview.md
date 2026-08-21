# Güvenlik Merkezi (Security Center): Genel Bakış

Güvenlik Merkezi, kiracınızın tüm güvenlik telemetrisini (çalışma zamanı sensörleri, ağ görünürlüğü, politika koruması, kimlik olayları, tedarik zinciri taramaları) tek bir yerde toplayan, izlemenizi, müdahale etmenizi ve uyumluluğunuzu kanıtlamanızı sağlayan bölümdür. Sol menüde **Güvenlik** başlığı altında yer alır ve 16 sayfaya dağılır.

---

## Beş grup, tek mantık

Güvenlik Merkezi'nin 16 sayfası, arayüzde tek düzey (flat) bir sekme şeridi olarak başlamış; sayfa sayısı büyüdükçe gezinme çubuğu bunları genel bir "Daha fazla" menüsüne taşmaya başlamış ve kullanıcılarda bir yer belleği (muscle memory) oluşmamış. Bunun üzerine ekip, sayfaları güvenlik operasyonlarının doğal akışını izleyen 5 anlamsal gruba ayırmıştır. Bu doküman seti de aynı 5 gruba göre yapılandırılmıştır:

| Grup | Odak | Kapsadığı sayfalar |
|---|---|---|
| **İzle** | Canlı izleme yüzeyleri — "şu an durum ne?" | Genel Bakış, Bulgular |
| **Kayıt** | Denetim + kimlik aktivite kaydı — "kim ne yaptı, ne zaman?" | Denetim Günlüğü, Giriş Etkinliği |
| **Koru** | Aktif koruma + müdahale — "buna karşı ne yapabilirim?" | Politikalar, Çalışma Zamanı Uygulama, Bloklar, Honey Paths, Playbook'lar |
| **Doğrula** | Doğrulama — "korumam gerçekten çalışıyor mu, uyumlu muyum?" | Sentetik Ataklar, Tedarik Zinciri, Uyumluluk, Küme Sağlığı |
| **Yönet** | Bildirim + saklama + WORM depolama yönetimi — "sistemin kendisini nasıl yapılandırırım?" | Bildirimler, Denetim Depolama, Saklama Politikası |

Her grubun kendi doküman sayfası vardır:

- [İzle: Genel Bakış ve Bulgular](./security-center-monitor.md)
- [Kayıt: Denetim ve Giriş Aktivitesi](./security-center-record.md)
- [Koru: Politika ve Müdahale](./security-center-protect.md)
- [Doğrula: Uyumluluk ve Sağlık](./security-center-verify.md)
- [Yönet: Bildirim ve Saklama](./security-center-manage.md)

> **İpucu:** Hangi dokümana bakacağınızdan emin değilseniz, kendinize şu soruyu sorun: "Şu an bir şeye mi bakıyorum (İzle/Kayıt), bir şeye karşılık mı veriyorum (Koru), bir şeyi mi doğruluyorum (Doğrula), yoksa sistemin kendisini mi yapılandırıyorum (Yönet)?"

---

## Kimler neyi kullanır?

Her sayfa, sadece kendi işiyle ilgilenen role açıktır — bir uyumluluk denetçisinin bulgulara veya politikalara erişebilmesi gerekmez, bir geliştiricinin saklama politikasını değiştirebilmesi gerekmez. Genel eğilim şöyledir:

- **Tüm geliştirici ekip** çoğu sayfayı görüntüleyebilir ve hafif kararlar (Onayla/Reddet gibi) alabilir.
- **Güvenlik operatörü / takım lideri** kalıcı kararları (İzin Ver, Tehdit Olarak İşaretle), politika ve müdahale aksiyonlarını, bildirim yapılandırmasını yönetir.
- **Denetçi (auditor) rolü** Uyumluluk ve Denetim Depolama'yı, diğer Güvenlik Merkezi yetkilerine ihtiyaç duymadan görüntüleyebilir.
- **Platform yöneticisi** en hassas yüzeyleri yönetir: çalışma zamanı uygulama modu, küme sağlığı, WORM arşiv yapılandırması ve saklama politikası.

Her grup dokümanının sonunda, o gruba özel "Kimler Kullanır?" tablosu bulunur.

Ayrıca bazı sayfalar **kiracının çalışma zamanı (host-runtime) kapsamına** göre görünürlük kazanır/kaybeder: iş yükleri paylaşımlı bir platform kümesinde çalışıyorsa ve platformun host-runtime sensörü bu kümeyi gözlemleyemiyorsa (**yalıtılmış çalışma zamanı**), Çalışma Zamanı Uygulama / Bloklar / Honey Paths sayfaları menüden ve ilgili kartlardan tamamen kaldırılır — sıfır değerli, yanıltıcı kutucuklar göstermek yerine. Küme Sağlığı ise tersine, sadece **platform yöneticilerine** görünür ve tenant operatörlerinden gizlenir.

---

## Güvenlik Merkezi'nin göründüğü diğer yerler

Güvenlik Merkezi verisi sadece kendi sayfalarıyla sınırlı değildir; aynı veri ve bileşenler platformun başka yerlerinde de yeniden kullanılır:

- **Üst menü çubuğu (topbar) güvenlik göstergesi:** Kalkan simgesi, kiracı genelindeki Kritik + Yüksek ihlal sayısını gösterir ve kritik sayı yükseldiğinde bir bildirim (toast) tetikler. Tıklandığında bulgular akışına götürür.
- **Ana kontrol paneli (dashboard) kartları:** Skor kartı, en riskli servisler, kritik bulgu akışı, güncel etkinlik, çalışma zamanı engelleme ve tespit performansı kartları — Genel Bakış sayfasıyla aynı paylaşılan bileşen kütüphanesinden gelir.
- **Servis bazlı Güvenlik sekmesi:** Her servisin kendi detay sayfasında da bir Güvenlik sekmesi bulunur; bu sekme, o tek servise özel bulgu listesi, ihlal geçmişi ve zaman çizelgesini gösterir — Güvenlik Merkezi'nin kiracı genelindeki görünümünün servis ölçeğindeki karşılığıdır.

Bu doküman seti, Güvenlik Merkezi'nin kiracı genelindeki deneyimini kapsar; servis bazlı Güvenlik sekmesi ve dashboard kartları bu belgelerde referans olarak anılır ama ayrıntılı olarak ele alınmaz.

---

## İlgili

- [İzle: Genel Bakış ve Bulgular](./security-center-monitor.md)
- [Kayıt: Denetim ve Giriş Aktivitesi](./security-center-record.md)
- [Koru: Politika ve Müdahale](./security-center-protect.md)
- [Doğrula: Uyumluluk ve Sağlık](./security-center-verify.md)
- [Yönet: Bildirim ve Saklama](./security-center-manage.md)
