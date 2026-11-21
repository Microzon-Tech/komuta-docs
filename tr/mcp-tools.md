# Komuta MCP

Komuta MCP sayesinde, Komuta üzerindeki işlerinizi artık panele girmeden, doğrudan yapay zeka asistanınıza söyleyerek halledebilirsiniz. Proje açmak, bir uygulamayı yayınlamak, veritabanı kurmak, servis loglarına bakmak veya faturaları görüntülemek gibi işler tek bir mesajla yapılabilir hale gelir. Aşağıda asistanın sizin için yapabileceği her şey, kategori kategori listelenmiştir.

Kurulum için: [https://www.komuta.io/docs/mcp/mcp-setup](https://www.komuta.io/docs/mcp/mcp-setup)

---

## Kaç Araç Var?

Komuta MCP **210 araç** sunar; hepsi aşağıda kategori kategori listelenmiştir.

Bu kadar aracın hepsi aynı anda kullanılmaz. Asistan önce isteğinizin hangi işe ait olduğunu belirler (`komuta_select_workflow`) ve yalnızca o işin araçlarıyla ilerler.

---

## İş Akışı ve Başlangıç Araçları (6)

| Araç | Açıklama |
|---|---|
| `komuta_select_workflow` | İsteğin hangi işe ait olduğunu belirler; diğer tüm araçlardan önce çağrılır. |
| `komuta_end_workflow` | İş bittiğinde veya bırakıldığında aktif iş akışını kapatır. |
| `komuta_get_started` | Komuta MCP bağlantısı ve ilk adımlar için rehber getirir. |
| `komuta_list_organizations` | Hesabın üye olduğu organizasyonları ve bağlı olunanı listeler. |
| `komuta_get_local_service_context` | Çalışma klasörünüz için kaydedilmiş varsayılan proje/servisi okur. |
| `komuta_set_local_service_context` | Çalışma klasörünüz için varsayılan proje/servis çiftini kaydeder. |

## Proje Araçları (4)

| Araç | Açıklama |
|---|---|
| `komuta_list_regions` | PaaS bölgelerini listeler. |
| `komuta_list_projects` | Kullanıcının projelerini listeler. |
| `komuta_create_project` | Yeni proje oluşturur. |
| `komuta_check_repo_connected` | Bir repo'nun zaten bir projeye bağlı olup olmadığını kontrol eder. |

## Stack Araçları (14)

Yeni servisler Stack ile yayınlanır: repo kökündeki `komuta.yaml` doğrulanır, bir plan çıkarılır ve onayınızla uygulanır. Ayrıntılar: [Stack](stacks.md).

| Araç | Açıklama |
|---|---|
| `komuta_verify_stack_manifest_file` | Yerel `komuta.yaml` dosyasının doğrulamaya hazır olduğunu kontrol eder. |
| `komuta_get_stack_publish_context` | Yayın için gereken hesap, hedef, bağlantı ve fiyat seçeneklerini getirir. |
| `komuta_resolve_repo_source` | Bir repo URL'ini hangi Git bağlantısının gördüğünü ve kullanılacak branch'i bulur. |
| `komuta_set_stack_profile` | Tercih edilen Stack yazım stilini kaydeder. |
| `komuta_list_stacks` | Mevcut Stack'leri listeler. |
| `komuta_validate_stack` | `komuta.yaml` dosyasını doğrular. |
| `komuta_plan_stack` | 30 dakika geçerli, değişmez bir Stack planı oluşturur. |
| `komuta_get_stack_plan` | Oluşturulmuş bir planı yeniden okur. |
| `komuta_create_stack_secret_handoff` | Eksik gizli değerlerin girilmesi için tek kullanımlık, 15 dakikalık konsol linki üretir. |
| `komuta_apply_stack` | Geçerli bir planı uygular. |
| `komuta_get_stack_run` | Bir Stack çalışmasının durumunu getirir. |
| `komuta_resume_stack_run` | Kısmen başarısız bir çalışmanın yalnızca başarısız adımlarını yeniden dener. |
| `komuta_get_stack_drift` | Canlı kaynakların Stack'in son uyguladığı hâlle eşleşip eşleşmediğini raporlar. |
| `komuta_export_stack` | Aktif Stack'i `komuta.yaml` olarak dışa aktarır. |

## Servis Araçları (16)

| Araç | Açıklama |
|---|---|
| `komuta_discover_repo_services` | Bir repo branch'indeki deploy edilebilir servisleri keşfeder. |
| `komuta_list_services` | Servisleri listeler. |
| `komuta_get_service_overview` | Servisin durum kartını (status overview) getirir. |
| `komuta_get_service_details` | Servis yapılandırmasını getirir. |
| `komuta_get_service_health_probes` | Servisin sağlık kontrollerini getirir. |
| `komuta_get_service_routing` | Servisin yönlendirme ayarlarını getirir. |
| `komuta_list_service_dependencies` | Servisin bağımlılıklarını listeler. |
| `komuta_list_service_ports` | Servisin portlarını listeler. |
| `komuta_get_service_deploy_options` | Servis için varsayılan deploy ayarlarını getirir. |
| `komuta_get_service_pipeline_status` | Servisin son/belirli deploy'unun pipeline durumunu getirir. |
| `komuta_get_pipeline_task_logs` | Bir pipeline adımının ham loglarını getirir. |
| `komuta_trigger_pipeline` | Servisi yeniden deploy eder (redeploy). |
| `komuta_change_branch` | Servisin bağlı olduğu branch'i değiştirir. |
| `komuta_set_service_replicas` | HPA kapalıyken replica sayısını değiştirir. |
| `komuta_start_service_maintenance` | Servisi bakım moduna alır; ziyaretçiler bakım sayfası görür. |
| `komuta_end_service_maintenance` | Bakım modunu kapatır ve servisi yeniden başlatır. |

## Rollout Araçları (10)

| Araç | Açıklama |
|---|---|
| `komuta_list_service_rollouts` | Servisin rollout'larını listeler. |
| `komuta_get_service_rollout_status` | Rollout durumunu getirir. |
| `komuta_get_service_rollout_history` | Rollout geçmişini getirir. |
| `komuta_pause_service_rollout` | Rollout'u duraklatır. |
| `komuta_resume_service_rollout` | Duraklatılmış rollout'u devam ettirir. |
| `komuta_promote_service_rollout` | Rollout'u canlıya alır. |
| `komuta_abort_service_rollout` | Rollout'u iptal eder. |
| `komuta_retry_service_rollout` | Rollout'u yeniden dener. |
| `komuta_rollback_service_rollout` | Önceki sürüme geri döner. |
| `komuta_restart_service_rollout` | Rollout'u yeniden başlatır. |

## Uyku ve Her Zaman Açık Araçları (9)

| Araç | Açıklama |
|---|---|
| `komuta_get_service_sleep_state` | Servisin çalışıyor mu uyuyor mu olduğunu ve nedenini getirir. |
| `komuta_get_service_sleep_history` | Uyku, uyanma ve Always on olaylarını listeler. |
| `komuta_hibernate_service` | Servisi uyutur. |
| `komuta_wake_service` | Servisi uyandırır. |
| `komuta_set_service_auto_sleep` | Otomatik uykuyu açar veya kapatır. |
| `komuta_get_free_hours_usage` | Organizasyonun bu ayki ücretsiz saat kullanımını getirir. |
| `komuta_get_service_keep_awake` | Servisin Always on eklentisinin durumunu getirir. |
| `komuta_purchase_service_keep_awake` | Servis için Always on eklentisini satın alır. |
| `komuta_cancel_service_keep_awake` | Always on eklentisini kapatır. |

## Service Doctor Araçları (8)

| Araç | Açıklama |
|---|---|
| `komuta_assess_service` | Service Doctor ile servisi değerlendirir ve öneriler üretir. |
| `komuta_accept_critical_recommendation` | Bir öneriyi kabul eder. |
| `komuta_defer_critical_recommendation` | Bir öneriyi seçilen tarihe kadar erteler. |
| `komuta_dismiss_critical_recommendation` | Bir öneriyi reddeder. |
| `komuta_prepare_recommendation_action_plan` | Kabul edilen öneri için uygulama planı hazırlar. |
| `komuta_approve_recommendation_action_plan` | Hazırlanan planı onaylar. |
| `komuta_record_recommendation_applied` | Elle uygulanan planı tamamlandı olarak kaydeder. |
| `komuta_verify_recommendation_action` | Uygulanan önerinin sonucunu yeniden değerlendirerek doğrular. |

## Dockerfile Araçları (3)

| Araç | Açıklama |
|---|---|
| `komuta_generate_dockerfile` | Dockerfile üretimi için prompt + platform build kurallarını getirir (Dockerfile yazmadan/düzenlemeden ÖNCE zorunlu). |
| `komuta_validate_dockerfile` | Dockerfile'ı platform kurallarına karşı doğrular (yazdıktan/düzenledikten SONRA zorunlu). |
| `komuta_validate_dockerfile_location` | Dockerfile'ın, seçilen servisin kendi dizininde olduğunu doğrular (monorepo senaryoları için zorunlu). |

## Ortam Değişkeni Araçları (4)

| Araç | Açıklama |
|---|---|
| `komuta_list_service_env_vars` | Servisin ortam değişkenlerini listeler. |
| `komuta_set_service_env_vars` | Bir veya birden çok ortam değişkenini tek seferde ekler/günceller. |
| `komuta_copy_service_secrets` | Gizli değerleri, değerleri göstermeden başka bir servisten kopyalar. |
| `komuta_apply_service_env_vars` | Kaydedilen ortam değişkenlerini çalışan servise uygular. |

## Ortam Değişkeni Grubu Araçları (7)

| Araç | Açıklama |
|---|---|
| `komuta_list_env_groups` | Ortam gruplarını listeler. |
| `komuta_get_env_group` | Bir grubun değişkenlerini ve hangi servislerde kullanıldığını getirir. |
| `komuta_create_env_group` | Yeni, boş bir ortam grubu oluşturur. |
| `komuta_set_env_group_variables` | Gruptaki değişkenleri ekler/günceller. |
| `komuta_preview_env_group_import` | Bir grubu servise aktarmadan önce farkları gösterir. |
| `komuta_import_env_group` | Grubun seçilen değişkenlerini servise aktarır. |
| `komuta_push_env_group` | Grubun güncel değerlerini seçilen servislere gönderir. |

## Otomatik Dağıtım Araçları (2)

| Araç | Açıklama |
|---|---|
| `komuta_get_service_auto_deploy` | Servisin auto-deploy konfigürasyonunu getirir. |
| `komuta_set_service_auto_deploy` | Servisin auto-deploy konfigürasyonunu uygular. |

## Ölçekleme Araçları (2)

| Araç | Açıklama |
|---|---|
| `komuta_get_service_hpa` | Servisin otomatik ölçeklendirme (HPA) ayarlarını getirir. |
| `komuta_set_service_hpa` | Servisin otomatik ölçeklendirme (HPA) ayarlarını uygular. |

## Alan Adı Araçları (5)

| Araç | Açıklama |
|---|---|
| `komuta_list_domains` | Servislere bağlı özel domain'leri ve DNS/TLS durumlarını listeler. |
| `komuta_list_attachable_services` | Domain bağlanabilecek servisleri listeler. |
| `komuta_create_domain` | Bir servise özel domain bağlar. |
| `komuta_refresh_domain_validation` | DNS ve sertifika doğrulamasını yeniden çalıştırır. |
| `komuta_delete_domain` | Domain bağlantısını kalıcı olarak kaldırır. |

## Depolama Araçları (6)

| Araç | Açıklama |
|---|---|
| `komuta_list_storage_attachments` | Servise bağlı kalıcı depolama alanlarını listeler. |
| `komuta_list_storage_packages` | Depolama planlarını listeler. |
| `komuta_list_service_directories` | Bağlama yolu seçmek için servisin dizinlerini listeler. |
| `komuta_create_storage_attachment` | Servise yeni depolama alanı ekler. |
| `komuta_update_storage_attachment` | Depolama alanının planını veya ayarlarını değiştirir. |
| `komuta_delete_storage_attachment` | Depolama alanını ve içindeki veriyi kalıcı olarak siler. |

## Erişim Koruması Araçları (18)

| Araç | Açıklama |
|---|---|
| `komuta_get_service_access_policy` | Servise kimlerin erişebildiğini gösterir. |
| `komuta_preview_service_access_change` | Bir erişim değişikliğinin etkisini uygulamadan gösterir. |
| `komuta_apply_service_access_change` | Onaylanan erişim değişikliğini uygular. |
| `komuta_explain_service_access` | Bir ziyaretçinin bir sayfayı açıp açamayacağını ve hangi kuralın karar verdiğini açıklar. |
| `komuta_list_service_access_shares` | Korumalı servise giriş yapabilenleri listeler. |
| `komuta_list_service_access_share_candidates` | Servisin paylaşılabileceği üyeleri ve organizasyonları listeler. |
| `komuta_save_service_access_share` | Bir paylaşım ekler veya günceller. |
| `komuta_delete_service_access_share` | Bir paylaşımı kaldırır. |
| `komuta_list_service_access_tokens` | Makineler için servis token'larını listeler. |
| `komuta_create_service_access_token` | Yeni servis token'ı oluşturur. |
| `komuta_delete_service_access_token` | Bir servis token'ını siler. |
| `komuta_get_service_access_mesh_peers` | Özel ağ üzerinden doğrudan erişebilen servisleri gösterir. |
| `komuta_set_service_access_mesh_peers` | Özel ağ üzerinden doğrudan erişebilen servisleri belirler. |
| `komuta_list_service_access_events` | Giriş, erişim ve ret olaylarını listeler. |
| `komuta_get_organization_access_sharing` | Organizasyon dışına paylaşım ayarını gösterir. |
| `komuta_set_organization_access_sharing` | Organizasyon dışına paylaşımı açar veya kapatır. |
| `komuta_list_organization_access_identity_providers` | Organizasyonun kimlik sağlayıcılarını listeler. |
| `komuta_get_access_identity_keys` | Uygulamanın ziyaretçi kimliğini doğrulamak için kullanacağı anahtarları getirir. |

## Job Araçları (8)

| Araç | Açıklama |
|---|---|
| `komuta_get_job_configuration` | Job yapılandırmasını getirir. |
| `komuta_get_cronjob_state` | CronJob durumunu getirir. |
| `komuta_get_active_job_runs` | Devam eden çalışmaları getirir. |
| `komuta_list_job_runs` | Çalışma geçmişini listeler. |
| `komuta_run_job_now` | Job'ı hemen bir kez çalıştırır. |
| `komuta_cancel_job_run` | Devam eden bir çalışmayı iptal eder. |
| `komuta_suspend_cronjob` | Zamanlanmış çalışmaları durdurur. |
| `komuta_resume_cronjob` | Zamanlanmış çalışmaları yeniden başlatır. |

## Addon Araçları (7)

| Araç | Açıklama |
|---|---|
| `komuta_create_database` | Yönetilen Postgres instance'ı oluşturur. |
| `komuta_create_rabbitmq` | Yönetilen RabbitMQ instance'ı oluşturur. |
| `komuta_create_valkey` | Yönetilen Valkey instance'ı oluşturur. |
| `komuta_list_addons` | Yönetilen addon'ları listeler. |
| `komuta_get_addon` | Addon durumunu getirir (status poll). |
| `komuta_get_addon_connection_info` | Addon bağlantı bilgilerini getirir. |
| `komuta_list_addon_plans` | Yönetilen addon planlarını listeler. |

## Addon Yaşam Döngüsü Araçları (28)

Postgres, Valkey ve RabbitMQ için aynı işlemler ayrı araçlarla yapılır.

| Araç | Açıklama |
|---|---|
| `komuta_restart_postgres` / `_valkey` / `_rabbitmq` | Instance'ı yeniden başlatır. |
| `komuta_backup_postgres` / `_valkey` / `_rabbitmq` | Yedek almayı başlatır. |
| `komuta_schedule_deletion_postgres` / `_valkey` / `_rabbitmq` | Instance'ın silinmesini zamanlar. |
| `komuta_cancel_deletion_postgres` / `_valkey` / `_rabbitmq` | Zamanlanmış silmeyi iptal eder. |
| `komuta_preview_postgres_plan_change` / `valkey` / `rabbitmq` | Plan değişikliğini uygulamadan gösterir. |
| `komuta_change_postgres_plan` / `valkey` / `rabbitmq` | Onaylanan plana geçer. |
| `komuta_soft_suspend_postgres` | Postgres'i geçici olarak askıya alır. |
| `komuta_hard_suspend_postgres` | Postgres'i tamamen askıya alır. |
| `komuta_resume_postgres` | Askıya alınan Postgres'i devam ettirir. |
| `komuta_get_postgres_restore_points` | Kullanılabilir geri yükleme noktalarını listeler. |
| `komuta_restore_latest_postgres` | Son yedekten geri yükler. |
| `komuta_cancel_postgres_restore` | Devam eden geri yüklemeyi iptal eder. |
| `komuta_rotate_postgres_credentials` | Postgres kimlik bilgilerini yeniler. |
| `komuta_get_postgres_backup_health` | Yedekleme sağlığını getirir. |
| `komuta_refresh_addon_backup_status` | Addon yedek durumunu yeniler. |
| `komuta_list_addon_alerts` | Addon alarm kurallarını listeler. |

## Bağlantı Araçları (6)

| Araç | Açıklama |
|---|---|
| `komuta_list_git_connections` | Git servis bağlantılarını listeler. |
| `komuta_list_registry_connections` | Container registry bağlantılarını listeler. |
| `komuta_get_github_install_url` | GitHub App kurulum URL'ini getirir. |
| `komuta_list_connection_repos` | Bir Git bağlantısı üzerinden görünen repoları listeler. |
| `komuta_list_connection_branches` | Bir repo'nun branch'lerini bağlantı üzerinden listeler. |
| `komuta_create_registry_connection` | Yeni container registry bağlantısı oluşturur. |

## API Gateway Araçları (19)

| Araç | Açıklama |
|---|---|
| `komuta_list_api_gateways` | API Gateway'leri listeler. |
| `komuta_get_api_gateway` | Gateway detaylarını getirir. |
| `komuta_get_api_gateway_pricing` | Gateway fiyatlandırmasını getirir. |
| `komuta_list_gateway_apis` | Gateway'deki API'leri listeler. |
| `komuta_list_gateway_api_candidates` | Gateway'e eklenebilecek servisleri listeler. |
| `komuta_list_gateway_operations` | Gateway operasyonlarını listeler. |
| `komuta_list_gateway_keys` | Gateway anahtarlarını listeler. |
| `komuta_revoke_gateway_key` | Seçilen anahtarı iptal eder. |
| `komuta_preview_gateway_config` | Kaydedilmiş yapılandırmayı önizler. |
| `komuta_get_gateway_pending_changes` | Bekleyen değişiklikleri getirir. |
| `komuta_apply_gateway_config` | Önizlenen yapılandırmayı uygular. |
| `komuta_get_gateway_live_config` | Canlı yapılandırmayı getirir. |
| `komuta_get_gateway_version` | Gateway sürümünü getirir. |
| `komuta_get_gateway_metrics` | İstek metriklerini getirir. |
| `komuta_get_gateway_scale` | Replica ayarlarını getirir. |
| `komuta_set_gateway_scale` | Replica ayarlarını değiştirir. |
| `komuta_get_gateway_bandwidth` | Bant genişliği planlarını ve kullanımı getirir. |
| `komuta_set_gateway_bandwidth_plan` | Bant genişliği planını değiştirir. |
| `komuta_delete_api_gateway` | Gateway'i siler. |

## Uyarı Araçları (12)

| Araç | Açıklama |
|---|---|
| `komuta_list_alert_rules` | Alarm kurallarını listeler. |
| `komuta_get_alert_rule` | Bir alarm kuralını getirir. |
| `komuta_list_alert_templates` | Alarm şablonlarını listeler. |
| `komuta_create_alert_rule` | Yeni alarm kuralı oluşturur. |
| `komuta_update_alert_rule` | Alarm kuralını günceller. |
| `komuta_delete_alert_rule` | Alarm kuralını siler. |
| `komuta_deploy_alert_rule` | Alarm kuralını devreye alır. |
| `komuta_undeploy_alert_rule` | Alarm kuralını devreden çıkarır. |
| `komuta_set_alert_enabled` | Alarm kuralını açar veya kapatır. |
| `komuta_get_alert_history` | Alarm geçmişini getirir. |
| `komuta_list_notification_channels` | Bildirim kanallarını listeler. |
| `komuta_set_notification_channel_active` | Bildirim kanalını açar veya kapatır. |

## Gözlemlenebilirlik ve Denetim Araçları (4)

| Araç | Açıklama |
|---|---|
| `komuta_query_service_logs` | Servisin pod loglarını sorgular. |
| `komuta_query_service_traces` | Servisin trace'lerini arar. |
| `komuta_get_trace_detail` | Bir trace'in detayını getirir. |
| `komuta_query_security_audit` | Güvenlik denetim olaylarını getirir. |

## Faturalandırma ve Fiyatlandırma Araçları (12)

| Araç | Açıklama |
|---|---|
| `komuta_get_service_invoices` | Servisin güncel dönem ücretlerini ve fatura geçmişini getirir. |
| `komuta_get_account_cost` | Hesabın güncel dönem maliyetini getirir. |
| `komuta_get_account_cost_history` | Hesabın maliyet geçmişini getirir. |
| `komuta_get_build_minutes_usage` | Build dakikası hakkını ve kullanımını getirir. |
| `komuta_get_account_billing_status` | Hesabın faturalandırma durumunu getirir. |
| `komuta_get_account_wallet` | Cüzdan bakiyesini getirir. |
| `komuta_list_wallet_transactions` | Cüzdan hareketlerini listeler. |
| `komuta_list_account_statements` | Hesap ekstrelerini listeler. |
| `komuta_get_statement_items` | Ekstre kalemlerini getirir. |
| `komuta_get_statement_payment_link` | Seçilen ekstre için ödeme linki üretir. |
| `komuta_pay_statement_from_balance` | Ekstreyi cüzdan bakiyesinden öder. |
| `komuta_list_resource_packages` | PaaS kaynak paketlerini listeler (paket seçimi her zaman kullanıcıya aittir, ajan seçmez). |
