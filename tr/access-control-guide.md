# Erişim Kontrolü (Access Control)

DevOpsZon'un erişim kontrol sistemi, organizasyonunuzdaki kullanıcıların hangi kaynaklara erişebileceğini ve hangi işlemleri yapabileceğini yönetmenizi sağlar. Entity (kayıt) bazında okuma ve düzenleme yetkileri atayarak güvenli bir çoklu kullanıcı ortamı oluşturun.

---

## Erişim Kontrol Modeli

DevOpsZon, **ACL (Access Control List)** tabanlı yetkilendirme kullanır. Her kullanıcıya, her kaynak (entity) için ayrı ayrı yetki atanabilir.

### Yetki Türleri

| Yetki | Açıklama |
|-------|----------|
| **Read** | Kaynağı görüntüleme; detaylarını inceleme; izleme verilerine erişme |
| **Edit** | Kaynağı güncelleme, yapılandırma değiştirme, deploy tetikleme, silme |

> **Not:** Edit yetkisi, Read yetkisini de kapsar. Bir kullanıcıya Edit verdiğinizde, otomatik olarak okuma yetkisi de sağlanır.

### Kaynak Türleri

Erişim kontrolü dört temel kaynak türü üzerinde uygulanır:

| Kaynak Türü | Kapsamı |
|-------------|---------|
| **Project** | Proje görüntüleme, servis listeleme, deploy tetikleme |
| **Service** | Servis detayı, pipeline, ortam değişkenleri, ingress, loglar |
| **Cluster** | Cluster bilgileri, node yönetimi, addon durumu |
| **Integration** | Git ve registry bağlantıları, repo listeleme |

---

## Erişim Kontrolü Sayfası

Sol menüden **Access Control** sayfasına gidin. Sayfa iki bölümden oluşur:

### Sol Panel — Kullanıcı Listesi

- Organizasyonunuzdaki tüm kullanıcılar listelenir
- Arama alanı ile kullanıcı bulabilirsiniz
- Yetkilendirmek istediğiniz kullanıcıyı tıklayarak seçin

### Sağ Panel — Yetkilendirme

Seçilen kullanıcı için dört sekme halinde yetki yönetimi:

| Sekme | Gösterim |
|-------|----------|
| **Project** | Tüm projeler ve kullanıcının her projedeki Read/Edit durumu |
| **Service** | Tüm servisler ve kullanıcının her servisteki Read/Edit durumu |
| **Cluster** | Tüm cluster'lar ve kullanıcının her cluster'daki Read/Edit durumu |
| **Integration** | Tüm entegrasyonlar ve kullanıcının her entegrasyondaki Read/Edit durumu |

---

## Yetki Atama

### Tek Tek Atama

1. Sol panelden kullanıcıyı seçin
2. Sağ panelde ilgili sekmeye gidin (ör: **Project**)
3. Her kayıt satırında **Read** ve **Edit** checkbox'larını işaretleyin/kaldırın
4. **Kaydet** butonuna tıklayın

### Toplu Atama

Her sekmenin üstünde toplu atama seçenekleri bulunur:

- **Tümüne Read Ver:** Seçili türdeki tüm kaynaklara okuma yetkisi verir
- **Tümüne Edit Ver:** Seçili türdeki tüm kaynaklara düzenleme yetkisi verir
- **Tümünü Kaldır:** Seçili türdeki tüm yetkileri kaldırır

---

## Kullanım Senaryoları

### Senaryo 1: Geliştirici Erişimi

Bir geliştirici, yalnızca çalıştığı projeleri ve servisleri görebilmeli ve deploy yapabilmeli:

| Kaynak | Yetki |
|--------|-------|
| Çalıştığı proje | **Edit** |
| Projedeki servisler | **Edit** |
| Diğer projeler | Yetki yok |
| Cluster | **Read** |
| Entegrasyonlar | **Read** |

### Senaryo 2: Takım Lideri

Takım lideri, tüm projeleri görebilmeli ancak yalnızca kendi takımının projelerini düzenleyebilmeli:

| Kaynak | Yetki |
|--------|-------|
| Kendi takımının projeleri | **Edit** |
| Diğer projeler | **Read** |
| Tüm cluster'lar | **Read** |
| Entegrasyonlar | **Edit** |

### Senaryo 3: İzleme / Operasyon

Operasyon ekibi, tüm kaynakları izleyebilmeli ancak değişiklik yapamamalı:

| Kaynak | Yetki |
|--------|-------|
| Tüm projeler | **Read** |
| Tüm servisler | **Read** |
| Tüm cluster'lar | **Read** |
| Entegrasyonlar | **Read** |

---

## Yetki Etkisi

Erişim kontrol ayarları, platformun her noktasında geçerlidir:

| Alan | Yetkisiz Kullanıcı İçin |
|------|-------------------------|
| **Proje listesi** | Yalnızca yetkili projeler görünür |
| **Servis listesi** | Yalnızca yetkili servisler görünür |
| **Servis yönetimi** | Edit yetkisi olmadan deploy, ayar değişikliği yapılamaz |
| **Cluster listesi** | Yalnızca yetkili cluster'lar görünür |
| **Entegrasyonlar** | Yetkisiz bağlantılar gizlenir |

---

## Tenant İzolasyonu

DevOpsZon'da her organizasyon (tenant) tamamen izole bir ortamda çalışır:

- Bir organizasyonun verileri diğer organizasyonlar tarafından erişilemez
- Her organizasyonun kendi kullanıcı listesi ve yetki yapısı vardır
- Kubernetes namespace'leri tenant bazında ayrıştırılır

Erişim kontrolü, aynı organizasyon içindeki kullanıcılar arasındaki yetki yönetimini sağlar.

---

## İpuçları

- **En az yetki prensibi:** Kullanıcılara yalnızca ihtiyaç duydukları minimum yetkileri verin
- **Düzenli gözden geçirme:** Yetki atamalarını periyodik olarak gözden geçirin; ayrılan çalışanların yetkilerini kaldırın
- **Proje bazlı atama:** Her yeni proje oluşturduğunuzda, ilgili kullanıcılara yetki atamayı unutmayın
- **Edit dikkatli kullanın:** Edit yetkisi silme dahil tüm değişikliklere izin verir
