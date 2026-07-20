# Komuta MCP Sunucusunun Kurulumu

Bu doküman, bir AI kodlama ajanının Komuta MCP sunucusuna nasıl bağlanacağını açıklar.

| Özellik | Değer |
|---|---|
| Server adı | `komuta` |
| Transport | HTTP |
| URL | `https://mcp.komuta.io/mcp` |
| Kimlik doğrulama | Bearer token |

## Adım 1: Sunucu yapılandırmasının eklenmesi

Aşağıdaki promptu, değiştirmeden AI kodlama ajanının chat'ine yapıştır. Agent, sunucuyu kendi standart yöntemiyle kaydedecek — bir CLI komutu, bir yapılandırma dosyası veya ayarlar arayüzü, hangisi geçerliyse. `<API_KEYIN>` ifadesini literal bir placeholder olarak koru; bu Adım 2'de değiştirilecek.

```
Şu anda çalıştığın AI kodlama client'ına, kendi standart MCP sunucusu kaydetme yöntemini kullanarak Komuta MCP sunucusunu ekle.

- Server adı: komuta
- Transport: HTTP
- URL: https://mcp.komuta.io/mcp
- Auth: Bearer token, değer: <API_KEYIN>

<API_KEYIN> ifadesini literal bir placeholder olarak koru — ayrı bir adımda değiştirilecek. Sunucuyu ekledikten sonra, <API_KEYIN>'in tam olarak nerede değiştirilmesi gerektiğini raporla (hangi dosya, ya da bir ayarlar ekranındaki hangi alan).
```

Chat arayüzü üzerinden prompt alamayan client'lar için (örneğin Claude Desktop veya claude.ai), sunucuyu manuel olarak ekle: connector veya MCP ayarlarını aç, yukarıdaki URL ile özel bir sunucu ekle, kimlik doğrulama alanını şimdilik boş bırak.

## Adım 2: API key'in yapılandırılması

Komuta hesap panelinden API key'i al, ardından Adım 1'de agent'ın `<API_KEYIN>`'i eklediğini raporladığı yere git ve placeholder'ı gerçek değerle değiştir.

## Adım 3: Bağlantının doğrulanması

Agent'a Komuta MCP sunucusunun bağlı olduğunu doğrulamasını veya mevcut araçlarını listelemesini söyle. `komuta` görünmeli.

## Sorun giderme

| Belirti | Neden |
|---|---|
| `401 Unauthorized` | Key eksik, yanlış yazılmış veya süresi dolmuş. Komuta hesabından yenisini oluştur. |
| Sunucu tanınmıyor | Agent'ın yapılandırmayı gerçekten uyguladığını doğrula — ne eklediğini göstermesini iste. |
| Key ifşa riski | Key'i versiyon kontrolüne commit'leme veya düz metin olarak paylaşma; key doğrudan Komuta hesabına erişim sağlar. |
