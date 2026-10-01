# TupBayiProje Test Oracle ve Golden Scenario Standardı

**Jira:** `TBP-10`  
**Durum:** Normatif  
**Kapsam:** Jira kabul kriterleri, unit/integration/contract/security/concurrency/idempotency/E2E testleri ve scenario validation  
**Bağlı belgeler:** `TupBayiProje_Global_Invariantlar.md`, `TupBayiProje_Jira_Story_Sablonu_ve_Definition_of_Ready_Standardi.md`, `TupBayiProje_Definition_of_Done_ve_Risk_Bazli_Kalite_Kapilari_Standardi.md`, `TupBayiProje_Source_of_Truth_ve_Domain_Ownership.md`, `TupBayiProje_State_Machine_ve_Yasam_Dongusu_Standardi.md`, `TupBayiProje_Transaction_Concurrency_Idempotency_ve_Guvenlik_Standardi.md`

Bu standart, beklenen sonucun implementasyondan bağımsız olarak nasıl belirleneceğini ve HIGH/CRITICAL işlerde Jira kabul kriterine yazılacak Golden Scenario kanıtını tanımlar. Canlı Jira gereksinimin Source of Truth'udur; test kodu veya mevcut uygulama davranışı gereksinimin yerine geçmez.

## 1. Değiştirilemez ilkeler

1. HIGH ve CRITICAL iş, somut Test Oracle veya Golden Scenario Jira'da bulunmadan `READY` veya `DONE` olamaz.
2. Beklenen sonuç test edilen implementasyonun çıktısından, aynı algoritmanın test içindeki kopyasından veya üretim verisinin tesadüfi anlık durumundan türetilemez.
3. Given başlangıç state'i, When eylemi ve Then sonucu ölçülebilir değerlerle yazılır; `doğru olmalı`, `çalışmalı` veya `uygun kayıt oluşmalı` tek başına kabul edilmez.
4. İlgili bütün authoritative state ve zorunlu yan etkiler birlikte doğrulanır. Yalnız UI mesajı veya HTTP status code yeterli değildir.
5. Negatif veya failure senaryosunda beklenen ret kadar sıfır kısmi etki de oracle'ın parçasıdır.
6. Para, miktar, zaman, timezone, yuvarlama, sıra, tolerans ve karşılaştırma kuralları belirsiz bırakılamaz.
7. Concurrency ve idempotency davranışı ilgiliyse Golden Scenario gerçek paralel yarış ile duplicate/replay sonuçlarını kapsar.
8. Oracle ile Jira gereksinimi, global invariant veya Source of Truth kaydı çelişirse iş uygulanmaz; `SPEC_REQUIRED` ya da gerekli yetkide `HUMAN_REQUIRED` sonucu üretilir.
9. Golden Scenario fixture'ı production business rule icat edemez. Senaryoda kullanılan özel fiyat, bakiye veya zaman değeri yalnız açık test girdisidir.
10. Secret, token, parola, tam ödeme verisi ve gereksiz kişisel veri scenario girdisi veya kanıtına yazılamaz.

## 2. Tanımlar

| Terim | Tanım |
|---|---|
| Test Oracle | Bir testin gerçek sonucunu karşılaştıracağı, implementasyondan bağımsız beklenen değer veya karar kuralı |
| Golden Scenario | Başlangıç değerleri, eylem, beklenen authoritative sonuçlar, zorunlu yan etkiler ve negatif/failure davranışları önceden sabitlenmiş uçtan uca örnek |
| Authoritative sonuç | İlgili Source of Truth store veya sahip domain tarafından kalıcılaştırılan state |
| Derived sonuç | Authoritative kayıttan üretilen projection, rapor, cache veya UI görünümü |
| Sıfır kısmi etki | Başarısız işlem sonrasında state, ledger, movement, outbox, audit ve idempotency kayıtlarında izin verilmeyen parça bırakılmaması |
| Tolerans | Yalnız teknik ölçümlerde kullanılabilen açık karşılaştırma aralığı; para ve adet doğruluğunu gevşetemez |
| Scenario kimliği | Jira acceptance criteria, test ve CI kanıtını aynı örneğe bağlayan kararlı kimlik |

Golden Scenario bir test framework'ü veya fixture dosyası değildir. Aynı normatif senaryo unit, integration, contract ve E2E testlerinde farklı teknik seviyelerde uygulanabilir.

## 3. Oracle kaynak hiyerarşisi

Beklenen sonuç aşağıdaki sırayla belirlenir:

1. Sistem, kullanıcı ve repository talimatları.
2. Canlı Jira açıklaması, acceptance criteria, bağlantıları ve yorumları.
3. Global invariantlar ve Source of Truth/domain ownership kayıtları.
4. Kabul edilmiş ADR, state machine, transaction, security ve contract standartları.
5. Yetkili dış provider'ın version'ı sabit resmi contract'ı.
6. İnsan tarafından açıkça onaylanmış test fixture'ı veya referans hesap.

Aşağıdakiler oracle olamaz:

- test edilen production metodu veya aynı algoritmanın kopyası,
- UI, cache, projection, log ya da audit kaydının business state iddiası,
- mevcut bug'lı davranışın snapshot'ı,
- kaynağı ve sürümü belirsiz örnek veri,
- mock'un kendi ürettiği değer,
- yalnız status code veya `success=true` sonucu.

Hiyerarşide iki kaynak farklı sonuç veriyorsa düşük öncelikli kaynak kullanılmaz; çelişki Jira'da kaydedilir.

## 4. Risk seviyesine göre kapsam

| Risk | Zorunlu oracle kapsamı |
|---|---|
| `LOW` | Test edilebilir acceptance sonucu; sayısal veya state etkisi varsa deterministic beklenen değer |
| `MEDIUM` | En az bir deterministic Given/When/Then; değişen state ve ilgili dış görünüm |
| `HIGH` | Golden Scenario; normal, negatif ve ilgili idempotency/concurrency/failure yolları; bütün authoritative ve zorunlu yan etkiler |
| `CRITICAL` | HIGH kapsamına ek olarak Human Gate gerektiren karar kanıtı, rollback/recovery senaryosu, veri sızıntısı ve cross-tenant negatifleri |

HIGH ve CRITICAL işlerde yalnız happy path Golden Scenario yeterli değildir. İlgili olmayan bir boyut `N/A - <somut gerekçe>` ile yazılır.

## 5. Zorunlu Golden Scenario alanları

Her Golden Scenario aşağıdaki alanları içerir:

| Alan | Zorunlu içerik |
|---|---|
| Kimlik | `GS-<JIRA_KEY>-<SIRA>` biçiminde kararlı scenario kimliği |
| Amaç | Kanıtlanan acceptance criteria ve korunan invariant |
| Risk | `HIGH` veya `CRITICAL` ve en yüksek risk gerekçesi |
| Oracle kaynağı | Jira, invariant, SOT, ADR veya version'lı dış contract referansı |
| Aktör ve scope | Identity, tenant, branch/warehouse/resource scope |
| Given | Başlangıç state'i ve bütün sayısal/temporal değerler |
| When | Tek anlamlı command, event veya actor eylemi |
| Then | Authoritative state ve observable sonuçların tam beklenen değerleri |
| Yan etkiler | Ledger, movement, cash, cari, payment, outbox, audit ve telemetry beklentileri |
| Değişmemesi gerekenler | Ret/failure halinde veya kapsam dışındaki state için sabit kalacak değerler |
| Hassasiyet | Para birimi, decimal scale, rounding, timezone, sıralama ve tolerans |
| Tekrar/yarış | Duplicate, replay, stale veya concurrency sonucu; ilgisizse gerekçe |
| Failure/recovery | Hata noktası, rollback, retry ve recovery sonrası beklenen state |
| Kanıt eşlemesi | Test adı/kimliği, CI run ve acceptance criteria bağlantısı |

Bir alanın boş bırakılması yerine `N/A - <neden>` yazılır.

## 6. Given/When/Then yazım kuralları

### 6.1 Given

Given bölümü, testten önceki authoritative state'i yeniden üretilebilir biçimde sabitler:

- tenant ve actor scope,
- aggregate kimliği, state'i ve concurrency version'ı,
- stok için ürün, tüp durumu, lokasyon ve adet,
- finans için currency, decimal değer, rounding ve bakiye,
- zaman için ISO 8601 değer, timezone ve gerekiyorsa iş günü,
- kullanılan idempotency key, event/provider kimliği ve sıra numarası,
- dış bağımlılık cevabı veya version'lı contract fixture'ı.

`Geçerli müşteri vardır` gibi gizli fixture'a dayalı ifade yeterli değildir; test sonucunu etkileyen alanlar görünür olmalıdır.

### 6.2 When

When bölümü tek bir business niyetini belirtir. Birden çok command gerekiyorsa hazırlık adımları Given'a taşınır veya ayrı scenario oluşturulur. Command payload, actor ve idempotency bilgisi açıkça yazılır.

### 6.3 Then

Then bölümü sonuçları exact değerlerle belirtir:

- yeni state ve version,
- her stok/lokasyon/durum için önce ve sonra adet,
- movement/ledger satırlarının türü, yönü, miktarı ve business referansı,
- cash/cari/payment bakiyeleri ve kayıtları,
- outbox/inbox mesaj sayısı, event type ve correlation,
- audit event türü, actor, scope, neden ve sonuç,
- API/event sonucu ve kararlı hata kodu,
- değişmemesi gereken state ve duplicate etki sayısı.

## 7. Hassasiyet ve karşılaştırma kuralları

| Veri türü | Kural |
|---|---|
| Adet | Exact integer karşılaştırması; tolerans yok |
| Para | Currency ve decimal scale açık; onaylı rounding kuralından sonra exact karşılaştırma |
| Zaman | ISO 8601 ve timezone açık; business zamanında tolerans yok, teknik latency yalnız açık aralıkla |
| Sıra | Order anlamlıysa exact sıra; anlamsızsa canonical sort kuralı |
| Kimlik | Sabit fixture kimliği veya format/presence kuralı; rastgele değer exact beklenmez |
| Metin | Case, normalization ve locale davranışı açık |
| Set | Eleman eşitliği ve cardinality; sıralama yalnız contract gerektiriyorsa |
| Telemetry | Business state yerine geçmez; event/metric adı, label sınırı ve correlation doğrulanır |

Floating-point yaklaşık karşılaştırma finansal değerlerde kullanılamaz. Teknik ölçüm toleransı business doğruluğunu gevşetemez.

## 8. Zorunlu etki matrisi

Her scenario aşağıdaki matrisi doldurur:

| Boyut | Önce | Beklenen sonra | Oracle kaynağı | Kanıt |
|---|---|---|---|---|
| Domain state | <değer> | <değer> | <referans> | <test/assertion> |
| Stok/movement | <değer> | <değer> | <referans veya N/A> | <kanıt> |
| Ledger | <değer> | <değer> | <referans veya N/A> | <kanıt> |
| Cash | <değer> | <değer> | <referans veya N/A> | <kanıt> |
| Cari | <değer> | <değer> | <referans veya N/A> | <kanıt> |
| Payment | <değer> | <değer> | <referans veya N/A> | <kanıt> |
| Outbox/event | <değer> | <değer> | <referans veya N/A> | <kanıt> |
| Audit | <değer> | <değer> | <referans veya N/A> | <kanıt> |
| API/UI sonucu | <değer> | <değer> | <referans veya N/A> | <kanıt> |
| Observability | <değer> | <değer> | <referans veya N/A> | <kanıt> |

Projection veya UI sonucu authoritative state ile birlikte doğrulanır; onun yerine geçmez.

## 9. Negatif, failure, concurrency ve idempotency oracle'ları

### 9.1 Negatif ve security

- Yetkisiz veya yanlış tenant/resource scope isteği kararlı hata koduyla reddedilir.
- Varlık bilgisi veya başka tenant verisi response, log ya da telemetry ile sızmaz.
- Authoritative state ve business audit/movement kayıtlarında izin verilmeyen etki oluşmaz.
- Güvenlik politikasının gerektirdiği ret audit'i ayrı ve maskelenmiş olabilir.

### 9.2 Failure ve rollback

- Hata enjekte edilen adım açıkça belirtilir.
- Transaction içindeki state, ledger, movement, outbox, audit ve idempotency etkilerinin hangilerinin sıfır kalacağı yazılır.
- Commit sonucu belirsizse kör retry yerine authoritative sonuç çözümleme beklenir.
- Recovery sonrasında beklenen son state exact değerlerle belirtilir.

### 9.3 Concurrency

- Başlangıç state'i ve aynı anda çalışan actor/command sayısı sabitlenir.
- Kaç işlemin commit olacağı, diğerlerinin hangi conflict sonucunu alacağı belirtilir.
- Final state, movement/ledger cardinality ve invariant exact doğrulanır.
- Gerçek paralel yürütme kullanılmadan concurrency oracle'ı geçmiş sayılmaz.

### 9.4 Idempotency

- Aynı key ve aynı payload replay'inde ilk sonuç döner; ikinci domain etkisi oluşmaz.
- Aynı key ve farklı payload kararlı `IDEMPOTENCY_CONFLICT` üretir.
- Movement, ledger, outbox ve audit cardinality'si exact yazılır.
- Retention sonrası davranış ilgili standarda göre ayrıca belirtilir.

## 10. Jira Golden Scenario şablonu

HIGH ve CRITICAL Story açıklamasına veya acceptance criteria bölümüne aşağıdaki şablon eklenir:

```markdown
## Golden Scenario: GS-<JIRA_KEY>-01

### Amaç ve Risk

- Acceptance Criteria: <madde>
- Korunan Invariant: <referans>
- Risk: <HIGH|CRITICAL> - <gerekçe>
- Oracle Kaynağı: <Jira/SOT/ADR/contract referansı>

### Given

- Tenant/Scope: <değer>
- Actor: <değer>
- Başlangıç State/Version: <değer>
- Stok/Finans/Zaman: <exact değerler>
- Idempotency/Event: <değer veya N/A - gerekçe>

### When

<Tek command, event veya actor eylemi>

### Then

- Authoritative State: <exact sonuç>
- Movement/Ledger: <exact kayıt ve cardinality veya N/A - gerekçe>
- Cash/Cari/Payment: <exact sonuç veya N/A - gerekçe>
- Outbox/Audit: <exact event ve cardinality veya N/A - gerekçe>
- API/UI: <gözlenebilir sonuç>
- Değişmemesi Gerekenler: <exact değerler>

### Negatif ve Failure

- <ret/failure başlangıcı, eylem, hata kodu ve sıfır kısmi etki>

### Concurrency ve Idempotency

- <paralel/replay sonucu veya N/A - gerekçe>

### Hassasiyet

- Currency/Scale/Rounding: <kural veya N/A>
- Timezone/Clock: <kural veya N/A>
- Order/Tolerance: <kural veya N/A>

### Kanıt

- Test Kimliği: <test veya suite>
- CI/Run: <bağlantı>
- Sonuç: <PASS|FAIL>
```

## 11. Golden Scenario örneği

Aşağıdaki değerler yalnız test fixture'ıdır; yeni üretim fiyatı veya finans kuralı tanımlamaz.

```gherkin
Scenario: GS-TBP-ORNEK-01 - Bir EMPTY verilip bir FULL alınması
  Given bayi stokunda FULL=10 ve EMPTY=5
  And müşteri zimmetinde FULL=0 ve EMPTY=1
  And cash=200.00 TRY ve cari=300.00 TRY
  And senaryo girdisinde finansal etki "Yok" olarak tanımlı
  And ledger kayıt sayısı=4 ve tamamlanmış işlem audit kayıt sayısı=7
  And idempotency key "GS-TBP-ORNEK-01-A"
  When müşteri 1 EMPTY verir ve bayiden 1 FULL alır
  Then bayi stoku FULL=9 ve EMPTY=6 olur
  And müşteri zimmeti FULL=1 ve EMPTY=0 olur
  And exact 1 FULL çıkış movement ve 1 EMPTY giriş movement oluşur
  And cash=200.00 TRY ve cari=300.00 TRY olarak değişmeden kalır
  And ledger kayıt sayısı=4 olarak değişmeden kalır
  And tamamlanmış işlem audit kayıt sayısı=8 olur
  And aynı idempotency key ile replay movement, ledger veya audit sayılarını değiştirmez
```

Bu örneğin oracle'ı başlangıç değerleri ve açık senaryo girdileridir. Gerçek Story; stok sahipliği, zimmet, finansal etki ve audit event türünü ilgili Jira/SOT/invariant referansına bağlamak zorundadır.

## 12. Scenario validation akışı

1. Canlı Jira ve bağlı normatif kaynaklardan scenario kimliği ve oracle okunur.
2. Fixture yalnız Given değerleriyle kurulur; üretim çıktısından seed edilmez.
3. Before snapshot authoritative store'lardan alınır.
4. When eylemi tek kez uygulanır.
5. After snapshot ve gözlenebilir sonuç etki matrisiyle karşılaştırılır.
6. Negatif, failure, concurrency ve idempotency varyantları ilgiliyse ayrı çalıştırılır.
7. Farklar expected/actual biçiminde, hassas veri içermeden raporlanır.
8. Test kimliği, commit/artefakt ve CI run Jira acceptance criteria'ya bağlanır.

Scenario yalnız bütün zorunlu boyutlar eşleştiğinde `PASS` olur. Bir authoritative sonuç yanlışken UI doğru görünse de sonuç `FAIL` olur.

## 13. Review ve karar sonuçları

Review en az şu soruları yanıtlar:

- Oracle implementasyondan bağımsız mı?
- Kaynak hiyerarşisi ve version referansları açık mı?
- Given bütün sonucu etkileyen değerleri sabitliyor mu?
- Then exact authoritative state ve yan etkileri kapsıyor mu?
- Para, zaman, rounding, sıra ve tolerans belirsiz mi?
- Negatif/failure yolunda sıfır kısmi etki doğrulanıyor mu?
- Concurrency ve idempotency ilgiliyse cardinality exact mı?
- Test ve Jira acceptance criteria aynı scenario kimliğiyle izlenebilir mi?

| Sonuç | Koşul |
|---|---|
| `PASS` | Oracle bağımsız, bütün zorunlu alanlar ve etkiler exact, test kanıtı eşleşiyor. |
| `REQUEST_CHANGES` | Scenario veya test bilinen standarda uymuyor. |
| `SPEC_REQUIRED` | Beklenen business sonucu ya da oracle kaynağı eksik/çelişkili. |
| `BLOCKED` | Bağımlılık, test ortamı veya GATE kanıtı eksik. |
| `HUMAN_REQUIRED` | Kritik business, güvenlik, veri veya geri dönüşü zor karar onay bekliyor. |

## 14. Değişiklik yönetimi ve kapsam sınırı

Oracle veya Golden Scenario değişikliği, acceptance criteria değişikliği sayılır. Değişiklik Jira geçmişinde gerekçe, etkilenen testler ve gerekiyorsa Human Gate ile kaydedilir. Başarısız testi geçirmek için expected değer sessizce güncellenemez.

Bu belge beklenen sonuç ve Golden Scenario yazım standardını tanımlar. Risk bazlı kalite kapıları `TupBayiProje_Definition_of_Done_ve_Risk_Bazli_Kalite_Kapilari_Standardi.md`; dependency DAG ve faz GATE politikası `TupBayiProje_Dependency_DAG_ve_Faz_GATE_Politikasi.md` (`TBP-11`); testlerin CI enforcement'ı `TBP-19` kapsamındadır. Bu belge yeni domain business rule üretmez.
