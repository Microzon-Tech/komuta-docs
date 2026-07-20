# Komuta MCP

Komuta MCP sayesinde, Komuta üzerindeki işlerinizi artık panele girmeden, doğrudan yapay zeka asistanınıza söyleyerek halledebilirsiniz. Proje açmak, bir uygulamayı deploy etmek, veritabanı kurmak, servis loglarına bakmak veya faturaları görüntülemek gibi işler tek bir mesajla yapılabilir hale gelir. Aşağıda asistanın sizin için yapabileceği her şey, kategori kategori listelenmiştir.

Kurulum için: [https://www.komuta.io/docs/mcp/mcp-setup](https://www.komuta.io/docs/mcp/mcp-setup)

## Project Tools (4)

| Tool | Açıklama |
|---|---|
| `komuta_list_regions` | PaaS bölgelerini listeler. |
| `komuta_list_projects` | Kullanıcının projelerini listeler. |
| `komuta_create_project` | Yeni proje oluşturur. |
| `komuta_check_repo_connected` | Bir repo'nun zaten bir projeye bağlı olup olmadığını kontrol eder. |

## Service Tools (8)

| Tool | Açıklama |
|---|---|
| `komuta_discover_repo_services` | Bir repo içindeki deploy edilebilir servisleri keşfeder. |
| `komuta_provision_service` | Bir projeye yeni servis deploy eder. |
| `komuta_list_services` | Servisleri listeler. |
| `komuta_get_service_deploy_options` | Servis için varsayılan deploy ayarlarını getirir. |
| `komuta_get_service_overview` | Servisin durum kartını (status overview) getirir. |
| `komuta_get_service_pipeline_status` | Servisin son/belirli deploy'unun pipeline durumunu getirir. |
| `komuta_trigger_pipeline` | Servisi yeniden deploy eder (redeploy). |
| `komuta_change_branch` | Servisin bağlı olduğu branch'i değiştirir. |

## Dockerfile Tools (3)

| Tool | Açıklama |
|---|---|
| `komuta_generate_dockerfile` | Dockerfile üretimi için prompt + platform build kurallarını getirir (Dockerfile yazmadan/düzenlemeden ÖNCE zorunlu). |
| `komuta_validate_dockerfile` | Dockerfile'ı platform kurallarına karşı doğrular (yazdıktan/düzenledikten SONRA zorunlu). |
| `komuta_validate_dockerfile_location` | Dockerfile'ın, seçilen servisin kendi dizininde olduğunu doğrular (monorepo senaryoları için zorunlu). |

## Addon Tools (7)

| Tool | Açıklama |
|---|---|
| `komuta_create_database` | Yönetilen Postgres instance'ı oluşturur. |
| `komuta_create_rabbitmq` | Yönetilen RabbitMQ instance'ı oluşturur. |
| `komuta_create_valkey` | Yönetilen Valkey instance'ı oluşturur. |
| `komuta_list_addons` | Yönetilen addon'ları listeler. |
| `komuta_get_addon` | Addon durumunu getirir (status poll). |
| `komuta_get_addon_connection_info` | Addon bağlantı bilgilerini getirir. |
| `komuta_list_addon_plans` | Yönetilen addon planlarını listeler. |

## Connection Tools (6)

| Tool | Açıklama |
|---|---|
| `komuta_list_git_connections` | Git servis bağlantılarını listeler. |
| `komuta_list_registry_connections` | Container registry bağlantılarını listeler. |
| `komuta_get_github_install_url` | GitHub App kurulum URL'ini getirir. |
| `komuta_list_connection_repos` | Bir Git bağlantısı üzerinden görünen repoları listeler. |
| `komuta_list_connection_branches` | Bir repo'nun branch'lerini bağlantı üzerinden listeler. |
| `komuta_create_registry_connection` | Yeni container registry bağlantısı oluşturur. |

## Autodeploy Tools (2)

| Tool | Açıklama |
|---|---|
| `komuta_get_service_auto_deploy` | Servisin auto-deploy konfigürasyonunu getirir. |
| `komuta_set_service_auto_deploy` | Servisin auto-deploy konfigürasyonunu uygular. |

## Scaling Tools (2)

| Tool | Açıklama |
|---|---|
| `komuta_get_service_hpa` | Servisin otomatik ölçeklendirme (HPA) ayarlarını getirir. |
| `komuta_set_service_hpa` | Servisin otomatik ölçeklendirme (HPA) ayarlarını uygular. |

## Logs Tools (1)

| Tool | Açıklama |
|---|---|
| `komuta_query_service_logs` | Servisin pod loglarını sorgular. |

## Billing Tools (1)

| Tool | Açıklama |
|---|---|
| `komuta_get_service_invoices` | Servisin güncel dönem ücretlerini ve fatura geçmişini getirir. |

## Pricing Tools (1)

| Tool | Açıklama |
|---|---|
| `komuta_list_resource_packages` | PaaS kaynak paketlerini listeler (paket seçimi her zaman kullanıcıya aittir, ajan seçmez). |
