# TupBayiProje Definition of Done ve Risk Bazlı Kalite Kapıları Standardı

**Jira:** `TBP-9`  
**Durum:** Normatif  
**Kapsam:** Kod, test, contract, migration, tasarım, dokümantasyon ve operasyon değişiklikleri  
**Bağlı belgeler:** `TupBayiProje_Global_Invariantlar.md`, `TupBayiProje_Jira_Story_Sablonu_ve_Definition_of_Ready_Standardi.md`, `TupBayiProje_Test_Oracle_ve_Golden_Scenario_Standardi.md`, `TupBayiProje_Dependency_DAG_ve_Faz_GATE_Politikasi.md`, `TupBayiProje_Source_of_Truth_ve_Domain_Ownership.md`, `TupBayiProje_State_Machine_ve_Yasam_Dongusu_Standardi.md`, `TupBayiProje_Transaction_Concurrency_Idempotency_ve_Guvenlik_Standardi.md`, `TupBayiProje_Kodlama_Dili_ve_Adlandirma_Standardi.md`

Bu standart, bir işin hangi somut kanıtlarla tamamlanmış sayılacağını ve risk seviyesine göre hangi kalite kapılarından geçeceğini tanımlar. Canlı Jira kaydı gereksinim ve iş sırası için Source of Truth'tur. Bu belge, Jira kabul kriterlerini değiştirmez; tamamlanma kararının nasıl kanıtlanacağını belirler.

## 1. Değiştirilemez ilkeler

1. Bir iş yalnız kod yazıldığı veya happy path çalıştığı için tamamlanmış sayılmaz.
2. Canlı Jira açıklaması, kabul kriterleri, bağlantıları ve yorumları uygulanmış ve kanıtlanmış olmalıdır.
3. Risk seviyesi, değişikliğin en yüksek etkili boyutuna göre belirlenir; ortalama veya çoğunluk alınmaz.
4. Zorunlu bir kapı atlanamaz. İlgisiz kapı yalnız `N/A - <somut gerekçe>` ile kapatılabilir.
5. `N/A`, testin zor veya pahalı olması, zaman baskısı ya da altyapının henüz bulunmaması gerekçesiyle kullanılamaz.
6. Test, review veya otomasyon hatası varken iş `DONE` olamaz. Flaky test başarı sayılmaz; kök neden giderilir veya iş blokeli tutulur.
7. HIGH ve CRITICAL işlerde Test Oracle veya Golden Scenario kanıtı zorunludur. Ayrıntılı format `TBP-10` kapsamında tanımlanır.
8. Kritik güvenlik, tenant izolasyonu, finans, stok, ödeme, migration, restore ve offline conflict kararları yalnız otomatik review ile onaylanamaz.
9. Kanıt Jira'ya veya Jira'dan erişilebilen kalıcı bir build, test, review ya da tasarım kaynağına bağlanır.
10. Secret, token, parola, tam ödeme verisi veya gereksiz kişisel veri test çıktısı, log, audit, ekran görüntüsü ya da Jira yorumuna yazılamaz.

## 2. Risk sınıflandırması

Risk değerlendirmesi blast radius, geri döndürülebilirlik, veri bütünlüğü, güven sınırı, eşzamanlılık, dış contract ve operasyon etkisini birlikte inceler. Birden fazla seviye eşleşirse en yüksek seviye seçilir.

| Seviye | Seçim koşulu | Örnek |
|---|---|---|
| `LOW` | Runtime davranışı, authoritative veri, güven sınırı veya dış contract değiştirmeyen; yerel ve kolay geri alınabilir değişiklik | Yazım düzeltmesi, davranış değiştirmeyen dokümantasyon, izole stil düzeltmesi |
| `MEDIUM` | Sınırlı runtime davranışı veya tek bileşen etkisi; veri kaybı ve kritik güven sınırı etkisi yok; standart rollback mümkün | Basit doğrulama, yönetim ekranı davranışı, kritik olmayan sorgu veya yapılandırma |
| `HIGH` | Authoritative state, stok/finans akışı, dış API/event contract'ı, migration, background job, cache tutarlılığı, concurrency/idempotency, authorization veya birden çok bileşen etkileniyor | Sipariş state geçişi, webhook işleme, schema değişikliği, tenant-scoped yetki kontrolü |
| `CRITICAL` | Tenant izolasyonu, identity/authentication çekirdeği, ödeme veya finansal doğruluk, destructive migration, tenant DB silme/restore, production deploy, geri dönüşü zor veri dönüşümü ya da offline finans conflict kararı etkileniyor | Cross-tenant erişim riski, ledger kuralı, ödeme webhook otoritesi, restore, production release |

Belirsiz risk düşük seçilemez. Gereksinim eksikse `SPEC_REQUIRED`, insanın kabul etmesi gereken geri dönüşü zor karar varsa `HUMAN_REQUIRED` sonucu üretilir.

## 3. Evrensel Definition of Done

Risk seviyesinden bağımsız olarak aşağıdaki koşulların tamamı sağlanır:

- İşin canlı Jira kaydı, kabul kriterleri, bağlantıları ve yorumları son kez okunmuştur.
- Uygulama kapsamı Jira gereksinimiyle sınırlıdır; kapsam dışı davranış veya spekülatif altyapı eklenmemiştir.
- İlgili global invariant, Source of Truth, domain ownership, state ve transaction kuralları korunmuştur.
- Kod, test ve teknik dokümantasyon proje dil ve adlandırma standardına uyar.
- Değişen davranışın başarı, ret ve hata yolları test edilmiş veya gerekçeli biçimde ilgisizdir.
- Zorunlu build, lint, static analysis ve test komutları temiz biçimde tamamlanmıştır.
- Yeni veya değişen dış contract backward/forward compatibility açısından doğrulanmıştır.
- Secret ve hassas veri kaynak kodda, fixture'da, logda veya kanıtta bulunmaz.
- Rollback veya recovery gerektiren değişiklik için uygulanabilir plan ve kanıt vardır.
- Review bulguları kapatılmış; açık `REQUEST_CHANGES`, `SPEC_REQUIRED`, `BLOCKED` veya `HUMAN_REQUIRED` sonucu kalmamıştır.
- Kabul kriteri ile test/kanıt arasında izlenebilir eşleme Jira'da veya bağlı artefaktta görülebilir.
- Jira'ya tamamlanma özeti, çalıştırılan kontroller ve kanıt bağlantıları yazılmıştır.

## 4. Risk bazlı zorunlu kalite kapıları

`Zorunlu`, kapının geçmesi gerektiğini; `Koşullu`, ilgili davranış değişiyorsa zorunlu olduğunu; `N/A`, normalde ilgisiz olduğunu belirtir. Koşullu bir kapının ilgisizliği somut gerekçeyle kanıtlanır.

| Kalite kapısı | LOW | MEDIUM | HIGH | CRITICAL |
|---|---|---|---|---|
| Build, lint, static analysis | Koşullu | Zorunlu | Zorunlu | Zorunlu |
| Unit test | Koşullu | Zorunlu | Zorunlu | Zorunlu |
| Integration test | N/A | Koşullu | Zorunlu | Zorunlu |
| Contract test | Koşullu | Koşullu | Değişen contract için zorunlu | Değişen contract için zorunlu |
| Security test | Koşullu | Koşullu | Güven sınırı etkisinde zorunlu | Zorunlu |
| Concurrency test | N/A | Koşullu | Paylaşılan state etkisinde zorunlu | Paylaşılan state etkisinde zorunlu |
| Idempotency/duplicate/replay testi | N/A | Koşullu | Retry edilebilir command/event için zorunlu | Retry edilebilir command/event için zorunlu |
| E2E test | N/A | Koşullu | Kritik kullanıcı/operasyon akışında zorunlu | Zorunlu |
| Rollback/recovery doğrulaması | N/A | Koşullu | Zorunlu | Zorunlu ve Human Gate kanıtlı |
| Audit doğrulaması | N/A | Koşullu | Kritik karar/state etkisinde zorunlu | Zorunlu |
| Observability doğrulaması | N/A | Runtime etkisinde zorunlu | Zorunlu | Zorunlu |
| Codex review | Zorunlu | Zorunlu | Bağımsız review zorunlu | Bağımsız review ve Human Gate zorunlu |
| Scenario validation | Kabul kriteri doğrulaması | Deterministic senaryo | Test Oracle/Golden Scenario | Golden Scenario ve negatif/failure senaryoları |

Yalnız dokümantasyon veya tasarım değişikliğinde build ve runtime testleri `N/A` olabilir; bağlantı bütünlüğü, içerik tutarlılığı, dil/adlandırma, Jira izlenebilirliği ve review yine zorunludur.

## 5. Kapıların geçme ölçütleri

### 5.1 Unit

- Değişen business kuralı, guard, hesaplama ve hata sınıfı doğrudan test edilir.
- Testler dış sisteme, saate veya rastgele sıraya kontrolsüz bağımlı değildir.
- Yalnız satır kapsaması yeterli kanıt değildir; acceptance sonucu assertion ile doğrulanır.

### 5.2 Integration

- Gerçek database, queue, cache, dosya sistemi veya dış sınır adapter'ı kullanılan davranış doğrulanır.
- Transaction commit ve rollback etkileri birlikte kontrol edilir.
- Test izolasyonu sağlanır; test sırası sonucu değiştirmez.

### 5.3 Contract

- OpenAPI, event, webhook, generated client ve provider/consumer beklentileri doğrulanır.
- Zorunlu alan, enum, hata envelope'u, status code ve version compatibility kapsanır.
- Breaking değişiklik açık migration/deprecation planı ve gerekli Human Gate olmadan geçemez.

### 5.4 Security

- Authentication, authorization, tenant scope ve resource scope server-side doğrulanır.
- Yetkisiz, yanlış tenant ve veri keşfi denemeleri kararlı biçimde reddedilir.
- Ret sonucu hassas veri sızdırmaz; log/audit/telemetry masking doğrulanır.

### 5.5 Concurrency

- Gerçek paralel yürütme ile yarış oluşturulur; yalnız ardışık çağrı concurrency testi sayılmaz.
- Korunan invariant ve beklenen winner/conflict sonucu doğrulanır.
- Deadlock, retry sınırı ve partial effect yokluğu ilgiliyse test edilir.

### 5.6 Idempotency

- Aynı key ve aynı payload replay'i ikinci domain etkisi üretmez.
- Aynı key ve farklı payload kararlı `IDEMPOTENCY_CONFLICT` sonucu üretir.
- Timeout veya belirsiz commit sonrasında retry, authoritative sonucu bulur; duplicate ledger, movement, outbox veya audit üretmez.

### 5.7 E2E

- Kullanıcının veya trusted actor'ün başlangıç eyleminden authoritative sonuca kadar kritik akış doğrulanır.
- UI tek başına doğruluk kanıtı değildir; server state ve zorunlu yan etkiler de kontrol edilir.
- En az bir negatif yol ve recovery yolu risk seviyesine uygun biçimde kapsanır.

### 5.8 Rollback ve recovery

- Kod rollback'i, feature disable veya veri recovery adımı açık ve uygulanabilirdir.
- Migration için forward fix ve rollback sınırı; veri kaybı ihtimali; eski/yeni sürüm birlikte çalışma davranışı tanımlıdır.
- CRITICAL işte yalnız yazılı plan yeterli değildir; dry-run, staging veya eşdeğer güvenli prova kanıtı gerekir.

### 5.9 Audit

- Actor, tenant/scope, operation, correlation, karar nedeni, önce/sonra referansı ve sonuç ilgili standarda uygun kaydedilir.
- Başarılı ve zorunlu ret olayları kapsanır.
- Audit kaydı authoritative business state yerine kullanılmaz ve sessizce değiştirilemez.

### 5.10 Observability

- Log, metric ve trace aynı correlation ile izlenebilir.
- Failure, retry exhaustion, conflict ve kritik latency için ölçüm; operasyon gerektiriyorsa alarm vardır.
- Telemetry başarı kanıtı olarak business state'in yerine geçmez ve hassas veri içermez.

## 6. Codex review kapısı

Codex review, Jira kapsamı ile diff'i karşılaştırır ve en az şu başlıkları değerlendirir:

- acceptance criteria ve scope uyumu,
- invariant, Source of Truth ve domain ownership korunumu,
- güvenlik ve tenant izolasyonu,
- transaction, concurrency ve idempotency,
- contract ve compatibility,
- testlerin risk seviyesine yeterliliği,
- rollback/recovery, audit ve observability,
- kodlama dili ve adlandırma standardı,
- gereksiz karmaşıklık, duplicate ownership ve bypass.

HIGH ve CRITICAL işte review, implementasyonu yapan aynı bağlamın öz-onayı olamaz; bağımsız ve taze bağlamlı review kanıtı gerekir. CRITICAL işte Codex sonucu insan kararının yerine geçmez.

| Sonuç | Anlam |
|---|---|
| `APPROVE` | Zorunlu kapılar geçti; açık kritik bulgu yok. |
| `REQUEST_CHANGES` | Bilinen standarda veya acceptance criteria'ya aykırılık var. |
| `SPEC_REQUIRED` | Gereksinim veya beklenen sonuç belirsiz/eksik. |
| `BLOCKED` | Ön koşul, bağımlılık, ortam veya GATE engeli var. |
| `HUMAN_REQUIRED` | İnsan yetkisi ya da geri dönüşü zor karar gerekiyor. |

Yalnız `APPROVE` sonucu Definition of Done'a katkı sağlar.

## 7. Scenario validation

Scenario validation test dosyası varlığını değil, kabul edilen business sonucunu doğrular:

1. Given başlangıç state'i ve authoritative değerler kaydedilir.
2. When actor eylemi veya system event'i uygulanır.
3. Then yeni authoritative state, ledger/movement, cash/cari, outbox, audit ve gözlenebilir sonuç ilgili olanların tamamı için doğrulanır.
4. Ret veya hata senaryosunda sıfır kısmi etki ve kararlı hata sınıfı doğrulanır.
5. Retry, replay, stale state ve paralel yarış ilgiliyse ayrıca çalıştırılır.

LOW işte acceptance criteria'nın gözle görülür doğrulaması yeterli olabilir. MEDIUM işte deterministic örnek gerekir. HIGH ve CRITICAL işte `TupBayiProje_Test_Oracle_ve_Golden_Scenario_Standardi.md` ile uyumlu, beklenen değerleri önceden belirlenmiş Test Oracle veya Golden Scenario zorunludur; test edilen implementasyon kendi beklenen sonucunu üretemez.

## 8. Kanıt kaydı

Her zorunlu kapı için aşağıdaki bilgiler Jira yorumunda veya bağlı CI/review artefaktında bulunur:

| Alan | İçerik |
|---|---|
| Risk | Seviye ve en yüksek risk gerekçesi |
| Kapı | Unit, Integration, Contract, Security, Concurrency, Idempotency, E2E, Rollback, Audit, Observability, Review veya Scenario |
| Sonuç | `PASS`, `FAIL`, `N/A`, `REQUEST_CHANGES`, `BLOCKED`, `SPEC_REQUIRED` veya `HUMAN_REQUIRED` |
| Komut/kaynak | Tekrarlanabilir komut, CI run, test raporu, PR review, Figma veya operasyon kanıtı |
| Kapsam | Doğrulanan acceptance criteria ve negatif/failure senaryosu |
| Zaman/sürüm | Commit, artefakt veya sürüm kimliği ve çalıştırma zamanı |
| Gerekçe | Özellikle `N/A` veya başarısız sonuç için somut açıklama |

Önerilen tamamlanma özeti:

```markdown
## Tamamlanma Kanıtı

- Risk: <LOW|MEDIUM|HIGH|CRITICAL> - <gerekçe>
- Acceptance Criteria: <madde -> kanıt eşlemesi>
- Testler: <kapı, komut/CI bağlantısı, sonuç>
- Rollback/Recovery: <kanıt veya N/A - gerekçe>
- Audit/Observability: <kanıt veya N/A - gerekçe>
- Codex Review: <sonuç ve bağlantı>
- Scenario Validation: <senaryo ve sonuç>
- Açık Risk: <yok | açık risk ve Human Gate kararı>
```

## 9. Tamamlanma kararı

İş ancak evrensel Definition of Done koşulları, risk matrisindeki zorunlu kapılar ve Jira acceptance criteria birlikte sağlandığında `DONE` kabul edilir.

| Karar | Koşul |
|---|---|
| `DONE` | Bütün zorunlu kapılar `PASS`, review `APPROVE`, gerekli Human Gate onaylı ve kanıtlar bağlıdır. |
| `REQUEST_CHANGES` | Uygulama veya kanıt standarda uymuyor. |
| `SPEC_REQUIRED` | Doğru sonucu belirleyecek gereksinim eksik. |
| `BLOCKED` | Tamamlanmamış ön koşul, dependency veya faz GATE engeli var. |
| `HUMAN_REQUIRED` | Yetkili insan kararı bekleniyor. |

`DONE` sonrasında yeni bir çelişki, regression, güvenlik açığı veya eksik kanıt bulunursa iş yeniden açılır; geçmiş kanıt silinmez.

## 10. Kapsam sınırı

Bu belge tamamlanma ve kalite kapısı politikasını tanımlar. Test Oracle ve Golden Scenario'nun ayrıntılı yazım standardı `TupBayiProje_Test_Oracle_ve_Golden_Scenario_Standardi.md` (`TBP-10`); dependency DAG ve faz GATE politikası `TupBayiProje_Dependency_DAG_ve_Faz_GATE_Politikasi.md` (`TBP-11`); CI üzerinde teknik enforcement `TBP-19` kapsamındadır. Bu belge bu ticket'ların yerine implementasyon veya yeni business rule üretmez.
