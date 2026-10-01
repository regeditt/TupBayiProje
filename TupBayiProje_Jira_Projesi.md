# TupBayiProje — Jira Projesi

**Jira Projesi:** TupBayiProje  
**Jira Anahtarı:** `TBP`  
**Toplam Jira İşi:** 226  
**Ana Faz:** 18 (`P00`–`P17`)  
**GATE:** 18  
**Durum:** Planlama / geliştirme öncesi

## Jira ve Kodlama Dil Standardı

**Normatif belge:** `TupBayiProje_Kodlama_Dili_ve_Adlandirma_Standardi.md`

Jira tarafında başlıklar, açıklamalar, kabul kriterleri, agent emirleri, review açıklamaları ve Human Gate metinleri Türkçe yazılır.

Kaynak kodda:
- Domain isimleri Türkçe anlamlı adlandırılır.
- Türkçe özel karakter kullanılmaz.
- Örnek: `Musteri`, `Satis`, `TupHareketi`, `CariHareketi`, `GunSonuKapatAsync`.
- Kod yorumları Türkçe fakat ASCII karakterlidir.
- Test isimleri Türkçe anlamlı ve ASCII'dir.
- `DbContext`, `CancellationToken`, `HTTP`, `OpenAPI`, `PostgreSQL`, `Npgsql`, `Redis`, `SignalR` gibi standart teknik adlar çevrilmez.
- Bu standarda aykırı implementasyon `REQUEST_CHANGES` alır.

## Jira Çalışma Kuralları

- `Blocks` ilişkisi önkoşul → bağımlı iş yönündedir.
- Circular dependency yasaktır.
- GATE atlama yasaktır.
- Eksik specification `SPEC_REQUIRED` üretir.
- Kritik mimari ihlal `BLOCKED` üretir.
- Riskli değişiklikler `HUMAN_REQUIRED` ile insan onayına gider.
- AI yeni business rule icat edemez.
- Jira, iş gereksinimlerinin Source of Truth'udur.

## Faz Bağımlılık Haritası

```text
P00
↓
P01
├→ P02 Factory MVP
├→ P03 Figma Core
└→ P04 Multi-Tenant Core + Connection PoC

P04-GATE → P05 Identity/Security → P06 Tenant Core
P06 → P07 Customer/Cari
P06 → P08 Cylinder/Inventory
P03-GATE + P07-GATE + P08-GATE → P09 POS
P09-GATE → P10 Orders/Delivery/Fleet
P09-GATE → P11 Cash/Day Close
P10-GATE + P11-GATE → P12 Offline/Sync/Conflict Review
P04-GATE + P05-GATE → P13 Licensing/Billing/Device
P09+P10+P11+P12 GATE'leri → P14 Reports/Admin
P13-GATE + P14-GATE → P15 Product Readiness
P04+P05+P12+P13+P14 GATE'leri → P16 Hardening
P15-GATE + P16-GATE → P17 Pilot/QA/Yayına Alma
```

## Tüm Jira Ticket'ları

### P00
- `TBP-1` — **Epic** — [P00] Ürün Gereksinimleri ve Mimari Yönetişim
- `TBP-2` — Değiştirilemez ürün ilkelerini ve global invariant'ları tanımla
- `TBP-3` — Source of Truth ve domain ownership kayıt defterini oluştur
- `TBP-4` — State machine ve yaşam döngüsü yönetişim standardını tanımla
- `TBP-5` — Transaction, concurrency, idempotency ve güvenlik standartlarını tanımla
- `TBP-6` — Kodlama dili ve adlandırma standardını tanımla
- `TBP-7` — ADR çerçevesini ve ilk mimari karar kayıtlarını oluştur
- `TBP-8` — Jira Story şablonunu ve Definition of Ready standardını oluştur
- `TBP-9` — Definition of Done ve risk bazlı kalite kapılarını tanımla
- `TBP-10` — Test Oracle ve Golden Scenario standardını oluştur
- `TBP-11` — Kanonik dependency DAG ve faz GATE politikasını tanımla
- `TBP-12` — **GATE** — [P00-GATE] Ürün ve mimari yönetişim temelini doğrula

### P01
- `TBP-13` — **Epic** — [P01] Platform Temeli ve Contract Altyapısı
- `TBP-14` — Kanonik repository ve monorepo yapısını oluştur
- `TBP-15` — .NET 10 modular monolith solution iskeletini oluştur
- `TBP-16` — Flutter çoklu platform workspace iskeletini oluştur
- `TBP-17` — Merkezi SaaS Admin Web iskeletini oluştur
- `TBP-18` — Docker yerel geliştirme temelini oluştur
- `TBP-19` — CI kalite kapılarını oluştur
- `TBP-20` — Architecture test altyapısını oluştur
- `TBP-21` — OpenAPI contract standardını oluştur
- `TBP-22` — OpenAPI mock server altyapısını oluştur
- `TBP-23` — Flutter generated API client altyapısını oluştur
- `TBP-24` — Admin Web generated API client altyapısını oluştur
- `TBP-25` — API breaking-change kontrolünü CI'a ekle
- `TBP-26` — Event Contract Registry temelini oluştur
- `TBP-27` — **GATE** — [P01-GATE] Platform ve contract temelini doğrula

### P02
- `TBP-28` — **Epic** — [P02] Otonom Yazılım Fabrikası MVP
- `TBP-29` — Factory orchestrator çekirdeğini ve state machine'i geliştir
- `TBP-30` — Jira V2 iş kuyruğu okuyucusu yerine TBP uygunluk motorunu geliştir
- `TBP-31` — Dependency DAG doğrulayıcı, claim, lock ve heartbeat geliştir
- `TBP-32` — Specification Guardian geliştir
- `TBP-33` — Architecture Guardian geliştir
- `TBP-34` — Risk değerlendirme motorunu geliştir
- `TBP-35` — Claude scoped execution runner geliştir
- `TBP-36` — Bağımsız test ve kalite runner geliştir
- `TBP-37` — Codex pre/post review runner geliştir
- `TBP-38` — Review parser ve düzeltme döngüsünü geliştir
- `TBP-39` — Spec drift detector geliştir
- `TBP-40` — Figma router ve frame-lock mekanizmasını geliştir
- `TBP-41` — Gerçek senaryo doğrulayıcısını geliştir
- `TBP-42` — Human Gate politika motorunu geliştir
- `TBP-43` — GitHub branch, PR ve merge politikasını geliştir
- `TBP-44` — Factory Jira güncelleyici ve audit writer geliştir
- `TBP-45` — Factory health, recovery ve WSL daemon temelini geliştir
- `TBP-46` — **GATE** — [P02-GATE] Otonom factory MVP akışını doğrula

### P03
- `TBP-47` — **Epic** — [P03] Figma Temelleri ve Tasarım Sistemi
- `TBP-48` — Figma proje yapısını ve sayfa yönetişimini oluştur
- `TBP-49` — Görsel temelleri ve design token'ları oluştur
- `TBP-50` — Çekirdek component kütüphanesini oluştur
- `TBP-51` — Desktop operasyon shell ve touch-first pattern'larını oluştur
- `TBP-52` — Mobile kurye ve yönetici shell pattern'larını oluştur
- `TBP-53` — Empty, loading, error, offline, stale ve conflict UI durumlarını oluştur
- `TBP-54` — POS ve telefon siparişi prototiplerini oluştur
- `TBP-55` — Kurye, araç mutabakatı, gün sonu ve Conflict Review prototiplerini oluştur
- `TBP-56` — **GATE** — [P03-GATE] Figma temelini ve kritik operasyon tasarımlarını doğrula

### P04
- `TBP-57` — **Epic** — [P04] Multi-Tenant Kontrol Düzlemi ve Connection PoC
- `TBP-58` — Master DB sahipliğini ve control-plane şemasını tanımla
- `TBP-59` — MasterDbContext temelini geliştir
- `TBP-60` — Tenant DB metadata ve secret yönetimini geliştir
- `TBP-61` — Tenant provisioning state machine geliştir
- `TBP-62` — Server-side TenantConnectionResolver geliştir
- `TBP-63` — Bounded NpgsqlDataSource cache ve connection lifecycle geliştir
- `TBP-64` — PgBouncer uyumluluk PoC'sini geliştir
- `TBP-65` — Tenant DB bootstrap ve schema oluşturma akışını geliştir
- `TBP-66` — Tenant migration version registry geliştir
- `TBP-67` — Tenant migration queue ve background worker geliştir
- `TBP-68` — Migration batch, throttle, retry ve recovery geliştir
- `TBP-69` — Provisioning application service ve API contract'larını geliştir
- `TBP-70` — Tenant yaşam döngüsü durum yayılım contract'ını geliştir
- `TBP-71` — 100+ tenant connection, memory ve izolasyon PoC testlerini çalıştır
- `TBP-72` — **GATE** — [P04-GATE] Multi-Tenant Core ve Connection PoC'yi doğrula

### P05
- `TBP-73` — **Epic** — [P05] Kimlik, Yetkilendirme ve Güvenlik
- `TBP-74` — Identity source of truth ve tenant login çözümleme contract'ını tanımla
- `TBP-75` — Authentication primitives ve session modelini geliştir
- `TBP-76` — Authenticated identity'den tenant context türetmeyi geliştir
- `TBP-77` — Rol ve permission authorization modelini geliştir
- `TBP-78` — Branch ve warehouse scope authorization geliştir
- `TBP-79` — Session, kullanıcı ve cihaz revoke yayılımını geliştir
- `TBP-80` — Login abuse protection ve security control'larını geliştir
- `TBP-81` — Desktop ve Mobile auth shell ile secure storage entegrasyonunu geliştir
- `TBP-82` — Authorization, revocation ve tenant-boundary negatif testlerini çalıştır
- `TBP-83` — **GATE** — [P05-GATE] Kimlik, yetkilendirme ve güvenlik temelini doğrula

### P06
- `TBP-84` — **Epic** — [P06] Tenant Operasyon Çekirdeği
- `TBP-85` — Tenant operasyon çekirdeği ve sahiplik sınırlarını tanımla
- `TBP-86` — İşletme profili ve şube domainini geliştir
- `TBP-87` — Depo domainini ve depo türlerini geliştir
- `TBP-88` — Tenant kullanıcı referansı ve şube atama modelini geliştir
- `TBP-89` — Ürün ve tüp ürün kataloğu temelini geliştir
- `TBP-90` — Tenant operasyon ayarlarını geliştir
- `TBP-91` — İnsan okunabilir belge numaralandırmasını geliştir
- `TBP-92` — Şube, depo, ürün ve kullanıcı sınırı negatif testlerini çalıştır
- `TBP-93` — **GATE** — [P06-GATE] Tenant operasyon çekirdeğini doğrula

### P07
- `TBP-94` — **Epic** — [P07] Müşteri, Adres, Fiyatlandırma ve Cari Çekirdeği
- `TBP-95` — Müşteri, adres, fiyatlandırma ve cari sahiplik contract'larını tanımla
- `TBP-96` — Bireysel ve ticari müşteri domainini geliştir
- `TBP-97` — Çoklu adres ve teslimat notu modelini geliştir
- `TBP-98` — Telefonla hızlı müşteri arama ve duplicate policy geliştir
- `TBP-99` — Fiyat geçmişi ve müşteri özel fiyatlandırmasını geliştir
- `TBP-100` — Cari ledger çekirdeğini geliştir
- `TBP-101` — Kredi limiti, vade ve aging kurallarını geliştir
- `TBP-102` — Müşteri/cari API contract ve authorization geliştir
- `TBP-103` — Müşteri, fiyat ve cari bütünlüğü negatif testlerini çalıştır
- `TBP-104` — **GATE** — [P07-GATE] Müşteri, fiyatlandırma ve cari çekirdeğini doğrula

### P08
- `TBP-105` — **Epic** — [P08] Tüp Hareket Defteri ve Envanter
- `TBP-106` — Cylinder ledger, inventory projection ve lokasyon sahipliğini tanımla
- `TBP-107` — Tüp türleri ve canonical movement sözlüğünü geliştir
- `TBP-108` — Stok lokasyon modelini geliştir
- `TBP-109` — Movement ledger'dan envanter projection geliştir
- `TBP-110` — Stok sayımı, adjustment ve fark iş akışını geliştir
- `TBP-111` — Müşterideki tüp ve depozito takibini geliştir
- `TBP-112` — Cross-brand CylinderExchangePolicy veri modelini geliştir
- `TBP-113` — Tedarikçi dolu kabul ve boş iade hareketlerini geliştir
- `TBP-114` — Concurrency ve oversell korumasını geliştir
- `TBP-115` — Ledger rebuild, duplicate movement ve izolasyon testlerini çalıştır
- `TBP-116` — **GATE** — [P08-GATE] Tüp ledger ve envanter çekirdeğini doğrula

### P09
- `TBP-117` — **Epic** — [P09] Satış ve POS Dikey Dilimi
- `TBP-118` — POS satış transaction contract'ını tanımla
- `TBP-119` — Sale aggregate ve atomik transaction orchestration geliştir
- `TBP-120` — Nakit, kart, banka, cari ve parçalı ödeme akışlarını geliştir
- `TBP-121` — Standart dolu satış ve boş tüp dönüş akışını geliştir
- `TBP-122` — Depozito ve customer-held cylinder entegrasyonunu geliştir
- `TBP-123` — Cross-brand exchange policy'yi POS'a entegre et
- `TBP-124` — Fiyat override, indirim ve yetkilendirmeyi geliştir
- `TBP-125` — Sale reversal, cancellation ve correction akışını geliştir
- `TBP-126` — Onaylı Figma'dan touch-first Desktop POS geliştir
- `TBP-127` — POS Golden Scenario, atomicity, concurrency ve idempotency testlerini çalıştır
- `TBP-128` — **GATE** — [P09-GATE] Satış ve POS dikey dilimini doğrula

### P10
- `TBP-129` — **Epic** — [P10] Sipariş, Teslimat ve Filo
- `TBP-130` — Sipariş, teslimat ve filo sahiplik/state contract'larını tanımla
- `TBP-131` — Telefonla hızlı sipariş oluşturmayı geliştir
- `TBP-132` — Dispatch kuyruğu, kurye ve araç atamasını geliştir
- `TBP-133` — Araç yükleme ve boşaltma transferlerini geliştir
- `TBP-134` — Kurye Mobile teslimat akışını geliştir
- `TBP-135` — Teslimat tamamlama ve Vehicle Stock hareketlerini geliştir
- `TBP-136` — Teslimat tahsilat entegrasyonunu geliştir
- `TBP-137` — Başarısız teslimat nedenleri ve retry/cancellation geliştir
- `TBP-138` — Vehicle return ve reconciliation iş akışını geliştir
- `TBP-139` — Delivery duplicate, assignment race ve vehicle-stock testlerini çalıştır
- `TBP-140` — **GATE** — [P10-GATE] Sipariş, teslimat ve filo dikey dilimini doğrula
### P11
- `TBP-141` — **Epic** — [P11] Kasa, Tahsilat ve Gün Sonu
- `TBP-142` — Kasa, tahsilat, gider ve gün sonu ledger contract'larını tanımla
- `TBP-143` — Kasa ve vardiya açılış modelini geliştir
- `TBP-144` — Müşteri tahsilat posting ve reversal geliştir
- `TBP-145` — Operasyonel gider kaydı ve authorization geliştir
- `TBP-146` — Beklenen kasa ve ödeme yöntemi mutabakatını geliştir
- `TBP-147` — Gün sonu gerçek sayım, fark ve approval akışını geliştir
- `TBP-148` — Kapanmış dönem kilidi ve correction policy geliştir
- `TBP-149` — Cash/cari/day-close audit ve rapor projection geliştir
- `TBP-150` — Duplicate collection, concurrent close ve reconciliation testlerini çalıştır
- `TBP-151` — **GATE** — [P11-GATE] Kasa, tahsilat ve gün sonu çekirdeğini doğrula

### P12
- `TBP-152` — **Epic** — [P12] Offline, Senkronizasyon ve Conflict Review
- `TBP-153` — Domain bazlı offline yetenek ve conflict matrisi tanımla
- `TBP-154` — Drift/SQLite local projection ve metadata temelini geliştir
- `TBP-155` — Durable outbox ve pending command altyapısını geliştir
- `TBP-156` — Offline Command Envelope modelini geliştir
- `TBP-157` — Sync coordinator, retry, backoff ve connectivity state geliştir
- `TBP-158` — Sequence ve idempotency doğrulamasını server tarafında geliştir
- `TBP-159` — Stale data ve refresh/invalidation kurallarını geliştir
- `TBP-160` — Domain conflict resolver framework geliştir
- `TBP-161` — Conflict Review ekranını geliştir
- `TBP-162` — Kontrollü offline POS, Delivery ve Collection akışlarını geliştir
- `TBP-163` — Reconnect reconciliation ve negatif offline senaryolarını test et
- `TBP-164` — **GATE** — [P12-GATE] Offline, sync ve Conflict Review çekirdeğini doğrula

### P13
- `TBP-165` — **Epic** — [P13] Lisanslama, Abonelik, Ödeme ve Cihaz Yaşam Döngüsü
- `TBP-166` — Abonelik, lisans, trial, grace ve suspension state machine'ini tanımla
- `TBP-167` — Master DB subscription, license, payment ve device modellerini geliştir
- `TBP-168` — IPaymentGateway abstraction ve provider adapter'larını geliştir
- `TBP-169` — Payment Webhook Inbox ve Idempotency Store geliştir
- `TBP-170` — Webhook signature, replay ve out-of-order korumasını geliştir
- `TBP-171` — Payment state machine'den subscription ve license aktivasyonunu geliştir
- `TBP-172` — Signed offline license ve clock-tamper korumasını geliştir
- `TBP-173` — Device activation, limit ve revocation yaşam döngüsünü geliştir
- `TBP-174` — License enforcement ve read-only politikasını geliştir
- `TBP-175` — Payment concurrency ve gerekli advisory-lock stratejisini doğrula
- `TBP-176` — Duplicate/out-of-order webhook, trial expiry, grace ve revoke testlerini çalıştır
- `TBP-177` — **GATE** — [P13-GATE] Lisanslama, ödeme ve cihaz yaşam döngüsünü doğrula

### P14
- `TBP-178` — **Epic** — [P14] Raporlar, Bildirimler ve Merkezi SaaS Yönetimi
- `TBP-179` — Rapor source'ları, aggregation ownership ve notification kurallarını tanımla
- `TBP-180` — Yönetici dashboard'unu geliştir
- `TBP-181` — Satış raporlarını geliştir
- `TBP-182` — Tüp ve araç stok raporlarını geliştir
- `TBP-183` — Cari ve finans raporlarını geliştir
- `TBP-184` — Teslimat ve kurye raporlarını geliştir
- `TBP-185` — Aksiyon odaklı notification engine geliştir
- `TBP-186` — Merkezi SaaS admin tenant ve lisans ekranlarını geliştir
- `TBP-187` — Merkezi payment, device, provisioning ve DB health ekranlarını geliştir
- `TBP-188` — Rapor tutarlılığı, permission ve tenant-isolation testlerini çalıştır
- `TBP-189` — **GATE** — [P14-GATE] Raporlar, bildirimler ve merkezi yönetimi doğrula

### P15
- `TBP-190` — **Epic** — [P15] Ürün Hazırlığı ve Müşteri Operasyonları
- `TBP-191` — İlk kullanım onboarding, import/export ve support policy'lerini tanımla
- `TBP-192` — Adım adım tenant onboarding sihirbazını geliştir
- `TBP-193` — Müşteri, cari, ürün ve başlangıç stok import'unu geliştir
- `TBP-194` — Müşteri verisi ve operasyon kayıtları export'unu geliştir
- `TBP-195` — Audit edilen süreli support session geliştir
- `TBP-196` — Windows installer ve uygulama update channel geliştir
- `TBP-197` — Update compatibility, post-update health ve rollback geliştir
- `TBP-198` — Kullanıcı yardım, recovery ve aksiyon alınabilir hata rehberlerini oluştur
- `TBP-199` — Temiz makine kurulumu ve ilk tenant yolculuğunu doğrula
- `TBP-200` — **GATE** — [P15-GATE] Ürün hazırlığını ve müşteri operasyonlarını doğrula

### P16
- `TBP-201` — **Epic** — [P16] Gözlemlenebilirlik, Yedekleme, Migration ve Hardening
- `TBP-202` — Observability, audit, backup, migration ve recovery contract'larını tanımla
- `TBP-203` — OpenTelemetry trace, metric ve correlation altyapısını geliştir
- `TBP-204` — Structured logging, veri masking ve audit pipeline geliştir
- `TBP-205` — Tenant, database ve uygulama health check'lerini geliştir
- `TBP-206` — Otomatik tenant backup policy ve backup doğrulamasını geliştir
- `TBP-207` — Human Gate kontrollü restore workflow geliştir
- `TBP-208` — Tenant migration orchestration'ı production için sertleştir
- `TBP-209` — API hardening, rate limit, request limit ve security header'ları geliştir
- `TBP-210` — Secret management ve operational security kontrollerini tamamla
- `TBP-211` — Disaster Recovery runbook ve kontrollü recovery drill'lerini oluştur
- `TBP-212` — Traceability, backup restore, migration failure ve security hardening testlerini çalıştır
- `TBP-213` — **GATE** — [P16-GATE] Observability, backup, migration ve hardening fazını doğrula

### P17
- `TBP-214` — **Epic** — [P17] Pilot, QA ve Yayına Alma
- `TBP-215` — MVP kabul matrisi ve release candidate giriş kriterlerini tanımla
- `TBP-216` — Gerçek operasyon senaryoları doğrulama paketini çalıştır
- `TBP-217` — Regression ve cross-platform compatibility matrisini çalıştır
- `TBP-218` — Performance, concurrency ve soak testlerini çalıştır
- `TBP-219` — Security ve tenant-isolation doğrulamasını çalıştır
- `TBP-220` — e-Fatura/e-Arşiv entegrasyon hazırlığını doğrula
- `TBP-221` — 1-3 gerçek bayi için kontrollü pilot hazırlığını yap
- `TBP-222` — Pilot support, incident capture ve ürün metriklerini çalıştır
- `TBP-223` — Pilot geri bildirimlerini Jira Change Request'lerine dönüştür
- `TBP-224` — Yayına alma checklist, rollback ve operasyon hazırlığını tamamla
- `TBP-225` — İlk production yayını için Human Gate kanıt paketini hazırla
- `TBP-226` — **GATE** — [P17-GATE] MVP ve production readiness kararını ver

## Kritik Mimari Hatırlatmalar

- Her tenant ayrı fiziksel PostgreSQL veritabanı kullanır.
- Master DB yalnızca control-plane verisi tutar.
- `Client TenantId` authoritative değildir.
- FULL ve EMPTY stok ayrı tutulur.
- Fiziksel stok değişiklikleri movement ledger üzerinden yapılır.
- Completed finansal ve stok kayıtları hard-delete edilmez; reversal/correction kullanılır.
- Vehicle Stock bağımsız fiziksel lokasyondur; kurye teslimatında ana depo doğrudan değişmez.
- Payment authority doğrulanmış webhook + Master DB state'tir.
- Offline finansal ve stok conflict'lerinde silent last-write-wins yasaktır.
- HIGH/CRITICAL işler için somut Golden Scenario / Test Oracle gerekir.
- Production deploy, destructive migration, restore, tenant isolation, auth/payment core gibi kritik değişiklikler Human Gate ister.

## Otonom Factory Akışı

```text
Jira
↓
Dependency DAG Check
↓
Specification Guardian
↓
Architecture Guardian
↓
Risk Evaluation
↓
Gerekirse Codex Pre-Review
↓
Gerekirse Figma
↓
Claude
↓
Bağımsız Testler
↓
Codex Final Review
↓
Scenario Validation
↓
Gerekirse Human Gate
↓
Merge
↓
DONE
↓
Sonraki Uygun İş
```

## Son Kural

> AI'nın ürettiği kod değişebilir; Jira'da tanımlanan sistem davranışı değişmemelidir.
