# Servis Dashboard

Dashboard, bir servisi açtığınızda ilk gördüğünüz ekrandır. Servisin o anki durumunu, çalıştığı planı, bağlı olduğu repo ve branch'i ve trafiğin pod'lara nasıl aktığını tek sayfada toplar. Aynı zamanda bir kontrol paneli gibi çalışır: strateji, plan ve branch değişikliklerini hızlıca yaparsınız.

![alt text](https://cdn.komuta.io/docs/tr/images/services-dashboard/service-dashboard.png)

---

## Rollout Durumu

Kart, servisin anlık durumunu ve aktif rollout stratejisinin adını gösterir. Durum canlıdır — deploy sürerken sayfayı yenilemeden değişir.

Rollout stratejisini, kartın üzerindeki ayarlar ikonundan değiştirebilirsiniz.

| Strateji | Kime uygun | Nasıl çalışır |
|---|---|---|
| **Blue-Green** | Çoğu servis | İki özdeş ortam. Sıfır kesinti, kolay geri alma; canlıya geçiş için manuel onay gerekir. |
| **Auto Promote** | Müdahalesiz akış isteyenler | Blue-Green ile aynı, ancak preview pod'lar hazır olduğunda geçiş otomatik yapılır. |
| **Canary** | Gelişmiş kullanım | Trafiği yeni sürüme kademeli aktarır. Yeni sürüme geçiş manuel yönetim gerektirir. |

> **Not:** Strateji değişikliği bir sonraki deploy'da geçerli olur. Devam eden bir rollout varsa etkilenmez, mevcut stratejisiyle tamamlanır.

![alt text](https://cdn.komuta.io/docs/tr/images/deployments/rollout-strategies.png)
---

## Plan / Paket

Kart, servisin çalıştığı kaynak planını, cluster tipini ve pakete tanımlı CPU ile bellek değerini gösterir.

Servisiniz daha fazla kaynağa ihtiyaç duyduğunda kart üzerindeki **Yükselt** butonundan paketi değiştirebilirsiniz.

> **İpucu:** Servisiniz sık sık yeniden başlıyor ya da bellek yetersizliğinden düşüyorsa ilk bakılacak yer bu karttır — buradaki CPU ve bellek değeri servisin kullanabileceği üst sınırdır.

![Plan / paket kartı ve Kaynak Planı penceresindeki paket listesi](https://cdn.komuta.io/docs/tr/images/resources/resource-plan-prices.png)

---

## Git Deposu

Bir sonraki deploy'un farklı bir branch'ten gelmesini istiyorsanız, kart üzerindeki düzenleme ikonundan branch'i değiştirebilirsiniz. Değişiklikten sonra build'ler ve otomatik dağıtımlar yeni branch'i takip eder. Aynı pencerede yer alan **Yeni branch'ten hemen dağıt** anahtarı açıksa deploy anında başlar; kapalıysa değişiklik kaydedilir ve yeni branch'e yapılan ilk push dağıtımı tetikler.

![Git deposu kartı ve branch değiştirme penceresi](https://cdn.komuta.io/docs/tr/images/branches/change-branch.png)

---

## MCP ID

MCP ID, servisinizin Komuta üzerindeki kimlik değeridir. Bu değeri kod ajanınıza (Claude, Codex vb.) verdiğinizde ajan doğrudan bu servis üzerinde çalışır — durumunu sorar, loglarını okur, deploy tetikler. Böylece basit operasyonlar için arayüze dönmeniz gerekmez.

Değeri karttaki kopyalama ikonuyla alırsınız. Ajanınızı Komuta'ya ilk kez bağlıyorsanız MCP ID tek başına yeterli değildir; önce bağlantı kurulumunu tamamlayın — [MCP Kurulumu](mcp-setup.md).

![MCP ID kartı — kimlik değeri ve kopyalama ikonu](https://cdn.komuta.io/docs/tr/images/mcp/mcp-id-dashboard.png)

---
 
## Servis Doktoru

Servis Doktoru, sayfanın üst kısmındaki butondan açılır ve servisin operasyonel durumunu tek seferde değerlendirir. Dağıtım, çalışma zamanı, ağ, bağımlılıklar, güvenilirlik ve güvenlik gibi alanları inceleyip tek bir sonuca bağlar.
 
Panelde sonucun yanında **dikkat gerektiren bulgular** (ne tespit edildi, neden önemli ve hangi kanıta dayanıyor), **ileriye dönük riskler** (henüz sorun değil ama yaklaşmakta olan durumlar) ve sonucun sağlıklıya dönmesi için nelerin değişmesi gerektiği yer alır. **Yeniden değerlendir** ile analizi istediğiniz an tekrar çalıştırabilirsiniz.
 
![Servis Doktoru paneli — sonuç başlığı, kapsam ve güven göstergeleri, bulgular listesi](https://cdn.komuta.io/docs/tr/images/service-doctor/service-doctor.png)
 
---

## Yeni Sürüm Dağıtma

Sayfanın üst kısmındaki **Dağıt** butonu, servisin yeni bir sürümünü yayına almanın en hızlı yoludur.

Açılan pencere mevcut sürüm etiketini gösterir ve yeni etiketi girmenizi ister. İsterseniz aynı pencereden bu dağıtımı servisin varsayılan branch'i yerine başka bir branch'ten çalıştırabilirsiniz — bu seçim yalnızca o dağıtım için geçerlidir, servisin kalıcı branch'ini değiştirmez.

Dağıtım tetiklendikten sonra ilerleyişini Rollout durumu kartından ve topolojiden canlı izlersiniz.

[RESIM EKLE: "Yeni Sürüm Dağıt" penceresi — mevcut sürüm, yeni etiket alanı ve branch seçimi]

---

## Durum Kartı

Bilgi kartlarının altındaki durum kartı; uyku durumu, güvenlik taraması, deploy sağlığı ve servis sağlığı gibi servis durumlarını tek yerde görüntüler. Servisinizde ilgilenmeniz gereken bir şey olup olmadığını anlamak için bakacağınız ilk yer burasıdır — aktif bir uyarı ya da devam eden bir işlem varsa kart bunu öne çıkarır.

![Durum kartı — "Her şey yolunda" durumu ve gezinme noktaları](https://cdn.komuta.io/docs/tr/images/deployments/smart-status-cards.png)

---

## Topoloji

Sayfanın sağ tarafındaki topoloji, servisinize gelen trafiğin canlı haritasıdır. Giriş noktasından başlayıp pod'lara ve bağlı disklere uzanan zinciri gösterir; pod eklendiğinde, kaldırıldığında ya da trafik dağılımı değiştiğinde harita güncellenir.

Asıl faydası bir sorunun *nerede* olduğunu bir bakışta göstermesidir: servis sağlıksız görünüyorsa hangi pod'un sorunlu olduğunu, hangi düğümün trafik almadığını doğrudan görürsünüz.

![Topoloji haritası — HTTPRoute'tan pod'lara uzanan bağlantılar](https://cdn.komuta.io/docs/tr/images/services-dashboard/topology.png)

### Haritadaki Düğümler

| Düğüm | Ne temsil eder |
|---|---|
| **HTTPRoute** | Trafiğin servise girdiği nokta |
| **Service** | Trafiği pod'lara dağıtan katman |
| **Preview / Canary** | Yeni sürümün henüz tüm trafiği almayan kopyası (kesik çizgi) |
| **Pod** | Uygulamanızın çalışan bir kopyası |
| **Storage** | Pod'a bağlı kalıcı disk; bağlama yolu ve kapasitesiyle gösterilir |

Düğümleri birleştiren çizgiler trafiği taşır. Canary veya Blue-Green rollout sırasında trafiğin sürümler arasında nasıl bölündüğünü bu çizgilerden canlı izlersiniz.

### Pod Loglarına Erişim

Haritadaki bir pod'a tıkladığınızda o pod'un logları açılır — log sekmesine gidip pod aramanıza gerek kalmaz.

![Topolojide bir pod'a tıklandığında açılan log penceresi](https://cdn.komuta.io/docs/tr/images/logs/pod-logs.png)

### Servisi Yeniden Başlatma

Panelin **Yeniden Başlat** butonu servisin aktif pod'larının tamamını yeniden oluşturur.

> **Uyarı:** Yeniden başlatma sırasında yeni pod'lar hazır olana kadar canlı trafik kısa süreli hata alabilir. Yoğun saatlerde kullanmaktan kaçının.

---

## İlgili Dokümanlar

- [Deploy Stratejileri](deployment-strategies.md) — Blue-Green, Canary ve Auto Promote'un nasıl işlediği.
- [Servis Yönetimi](service-guide.md) — Dashboard dışındaki servis sekmeleri.
- [MCP Kurulumu](mcp-setup.md) — Kod ajanınızı Komuta'ya bağlama.
- [İzleme ve Loglar](monitoring-logs.md) — Log arama, metrik geçmişi ve uyarılar.
