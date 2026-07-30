# Komuta MCP Sunucusunun Kurulumu

Bu doküman, Claude ve Codex'in Komuta MCP sunucusuna nasıl bağlanacağını açıklar.

## MCP ile neler yapabilirsin?

Kuruluma geçmeden önce, MCP sunucusunun sunduğu araçlara göz atabilirsin: bir git deposundan servis provision etme, Service Doctor ile servis sağlığını değerlendirme, yönetilen Postgres / RabbitMQ / Valkey addon'ları oluşturma, log ve trace inceleme, paket ve fiyatlandırma bilgisi alma.

Araçların tam listesi ve ne işe yaradıkları: [Komuta MCP Araçları](https://www.komuta.io/docs/mcp/mcp-tools)

## Claude için kurulum

### Claude Code (CLI)

```bash
claude mcp add --transport http komuta https://mcp.komuta.io/mcp -s user
```

Giriş yap:

```bash
claude mcp login komuta
```

### Claude Desktop

1. `Settings` → `Connectors` → `Add` → `Custom Connector` yoluna git.
2. Şunları doldur:
   - **Name:** Komuta
   - **Remote MCP Server URL:** `https://mcp.komuta.io/mcp`
3. `Add`'e tıkla.
4. `Connect`'e tıkla.

## Codex için kurulum

1. Codex CLI'yi indir:

   ```bash
   npm install -g @openai/codex
   ```

2. `~/.codex/config.toml` dosyasına ekle (Windows'ta `C:\Users\<Kullanıcı_Adı>\.codex\config.toml`):

   ```toml
   [mcp_servers.komuta]
   url = "https://mcp.komuta.io/mcp"
   scopes = ["openid", "email", "profile", "offline_access", "komuta:mcp"]
   ```

3. Giriş yap:

   ```bash
   codex mcp login komuta
   ```

## Bağlantının doğrulanması

Agent'a Komuta MCP sunucusunun bağlı olduğunu doğrulamasını veya mevcut araçlarını listelemesini söyle. `komuta` görünmeli. Dilersen `/mcp` yazarak bağlantıyı kendin de doğrulayabilirsin. Eğer `komuta` listede görünmüyorsa, uygulamayı tamamen kapatıp tekrar açmayı denemelisiniz.

## Sorun giderme

| Belirti | Neden |
|---|---|
| Tarayıcı üzerinden giriş açılmıyor veya tamamlanmıyor | `codex mcp login komuta` komutunu tekrar çalıştır; Claude Desktop'ta connector'ı kaldırıp yeniden ekle. |
| Sunucu tanınmıyor | Yapılandırmanın doğru dosyaya (`~/.codex/config.toml`) veya doğru CLI komutuyla eklendiğini doğrula. |
| `401 Unauthorized` / yetkilendirme hatası | Oturumun süresi dolmuş olabilir; giriş adımını tekrarla. |
