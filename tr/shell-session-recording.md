# Kabuk Oturumu Kaydı

Oturum kaydı, organizasyonunuzda açılan terminal oturumlarında ekranda görünenleri saklar. Yöneticiler kayıtları sonradan izleyebilir. Kayıt varsayılan olarak kapalıdır ve tüm planlarda kullanılabilir.

---

## Kaydı açma

1. **Hesap → Organizasyonlar** sayfasında **Kabuk erişimi** bölümünü bulun.
2. **Kabuk oturumlarını kaydet** anahtarını açın.
3. Açılan pencereyi okuyun, saklama süresini seçin (7, 30 ya da 90 gün; varsayılan 30) ve onaylayın.

Bu işlemi organizasyonu yönetme yetkisi olan kişiler yapabilir. Açma, kapatma ve süre değişikliği denetim kaydına yazılır.

Kayıt, ayar açıldıktan sonra başlatılan oturumlar için geçerlidir. O anda açık olan kabuklar kaydedilmez. Ayarı kapatmak da yalnızca yeni oturumları etkiler: kayıtla başlamış bir oturum kapanana kadar kaydedilir.

## Kullanıcılar ne görür

Kayıt açıkken **Terminal** sayfasında, bağlanmadan önce ve oturum boyunca kırmızı bir bant görünür. Bu bant oturumun kaydedildiğini söyler ve kapatılamaz.

## Ne kaydedilir

- Ekranda görünen her şey: yankılanan komutlar, komut çıktıları, dosya içerikleri, ortam değişkenleri.
- Terminal penceresinin boyut değişiklikleri ve olayların zamanı.

**Ekrana basılan parola, anahtar, belirteç ve kişisel veriler de kayda girer.** Bu nedenle kaydı açmadan önce ekiplerinizin kabukta hangi verileri görebileceğini gözden geçirin.

## Ne kaydedilmez

Ekrana yansımayan girdiler kaydedilmez. Parola istemine yazılan parolalar buna örnektir. Kayıt bir komut listesi değildir: yankılanmayan bir komut kayıtta görünmez, kabuk içinden başlatılan bir programın ne yaptığını da kanıtlamaz.

## Kayıtları izleme ve silme

Kayıtlar aynı bölümdeki **Kayıtlar** listesinde görünür: servis, kişi, başlangıç zamanı ve silineceği tarih. **İzle** kaydı oynatır; duraklatabilir, hızlandırabilir ve ileri geri sarabilirsiniz. Uzun sessiz aralıklar oynatırken kısaltılır.

- **Sil**, kaydı kalıcı olarak kaldırır. Onay için servis adını yazmanız istenir.
- İzleme ve silme denetim kaydına yazılır. Aynı kişinin aynı kaydı izlemesi bir saat içinde bir kez yazılır.
- Süren bir oturumun kaydı izlenebilir, ancak oturum bitene kadar silinemez.
- **Eksik** etiketi, oturumun sonunun kaydedilemediğini gösterir. Bağlantı hizmetinin beklenmedik şekilde kapanması son birkaç saniyeyi kaybettirebilir.

## Saklama

Kayıtlar seçtiğiniz sürenin sonunda otomatik olarak silinir. Süreyi değiştirirseniz yeni süre mevcut kayıtlar için de geçerli olur: süreyi kısaltmak, yeni süreden eski kayıtların bir sonraki günlük temizlikte silinmesine yol açar.

## Kayıt alınamazsa

Kayıt açıkken bir oturum kaydedilemiyorsa **kabuk açılmaz; açık olan kabuk kapanır.** Sıkıştırılmış hali 64 MiB'ı aşan oturumlar da kapanır. Kayıt altyapısı kullanılamıyorsa kabuk açmak için ayarı kapatmanız gerekir.

---

## Veri işleme koşulları

*Sürüm: 2026-11-04. Kayıt özelliği açıldığında geçerli olur ve hizmet sözleşmenizin ekidir.*

**Roller.** Kayıt alma kararını organizasyonunuz verir. Kayıtlar bakımından organizasyonunuz veri sorumlusu, Komuta veri işleyendir.

**Sizin sorumluluğunuz.** Kabuk açan kişileri ve kayıtlarda verisi görünebilecek kişileri bilgilendirmek, gerekli hukuki dayanağı sağlamak ve kayıtların erişimini organizasyon içinde yönetmek sizin sorumluluğunuzdadır. Terminaldeki bant bu bilgilendirmenin yerine geçmez.

**İşlenen veri.** Oturumda ekrana yansıyan içerik, oturumun hangi serviste ve hangi kullanıcı tarafından açıldığı ve zaman bilgisi. Bu içerik ekrana basılan kişisel verileri ve gizli bilgileri kapsayabilir.

**Amaç.** Kayıtlar yalnızca size bu özelliği sunmak için saklanır. Komuta kayıtların içeriğini başka bir amaçla kullanmaz.

**Saklama yeri ve alt işleyen.** Kayıtlar, ayrı bir depolama alanında AB bölgesinde Backblaze B2 üzerinde saklanır.

**Güvenlik.** Her kayıt kendine özgü bir anahtarla AES-256-GCM kullanılarak şifrelenir. Kayıt anahtarı Komuta'nın anahtarıyla şifrelenmiş olarak veritabanında tutulur. Bu nedenle Komuta kayıtları teknik olarak çözebilir; şifreleme, depolama sağlayıcısının içeriği okumasını engeller, Komuta'nın erişimini engellemez.

**Silme.** Silme ve süre dolması, kayıt anahtarını yok eder ve depodaki parçaları siler. Silinen parçalar depodan birkaç gün içinde tamamen kalkar. Denetim kayıtları kaydın içeriğini içermez ve kendi saklama süresine tabidir.

**Değişiklikler.** Bu koşulların yeni sürümü yayınlandığında sürüm tarihi güncellenir.
