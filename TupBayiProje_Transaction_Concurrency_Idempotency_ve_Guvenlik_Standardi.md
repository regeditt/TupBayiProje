# TupBayiProje Transaction, Concurrency, Idempotency ve Güvenlik Standardı

**Jira:** `TBP-5`  
**Durum:** Normatif  
**Bağlı belgeler:** `TupBayiProje_Global_Invariantlar.md`, `TupBayiProje_Source_of_Truth_ve_Domain_Ownership.md`, `TupBayiProje_State_Machine_ve_Yasam_Dongusu_Standardi.md`

Bu belge, finansal ve fiziksel stok etkileri başta olmak üzere bütün kritik backend işlemlerinin transaction, concurrency, idempotency, duplicate protection ve authorization kurallarını tanımlar. Mekanizma standardıdır; domain'e özgü yeni business rule üretmez.

## 1. Değiştirilemez ilkeler

1. Kritik bir iş işlemi ya bütün zorunlu etkileriyle commit olur ya da hiçbir etki bırakmadan geri alınır.
2. Her yazma işlemi doğrulanmış identity ve server-side çözümlenmiş tenant context ile yürütülür. Client `TenantId` değeri authoritative değildir.
3. Her veri yalnızca Source of Truth kayıt defterindeki sahibi tarafından değiştirilir.
4. Finansal ve stok etkisi üreten tekrar çağrı ikinci bir domain etkisi oluşturamaz.
5. Concurrency conflict silent last-write-wins ile çözülemez.
6. Tamamlanmış finansal veya stok kaydı yerinde değiştirilmez ya da silinmez; reversal/correction kullanılır.
7. Authorization yalnızca UI görünürlüğüne veya client beyanına bırakılamaz; backend her komutta yeniden doğrular.
8. Güvenlik, tenant, imza veya yetki doğrulaması başarısız olduğunda sistem fail-closed davranır.

## 2. Transaction sınırı

### 2.1 Aynı veritabanındaki atomik işlem

Aynı Tenant DB içinde tek iş sonucuna ait aşağıdaki etkiler gerektiğinde tek database transaction içinde tutulur:

- aggregate state ve version değişikliği,
- stok movement ledger kayıtları,
- finansal/cari ledger kayıtları,
- idempotency sonucu,
- durable outbox kayıtları,
- iş sonucunun kanıtı olan audit kaydı.

Orchestration birden fazla domain kuralını çağırabilir; buna rağmen her domain yalnızca kendi tablolarına yazar. Transaction atomikliği domain ownership'i geçersiz kılmaz.

### 2.2 Birden fazla veritabanı veya dış sistem

Master DB, Tenant DB, ödeme sağlayıcısı veya başka bir dış sistem arasında distributed transaction varsayılmaz. Bu sınırda:

1. Yerel authoritative değişiklik ve outbox kaydı aynı transaction'da commit edilir.
2. Uzak etki outbox üzerinden, benzersiz mesaj kimliğiyle gönderilir.
3. Consumer inbox/deduplication kaydıyla tekrar işlemeyi engeller.
4. Retry güvenli ve gözlemlenebilir olur; başarısızlık pending/failed state olarak saklanır.
5. Gerekli telafi yalnızca tanımlı reversal/correction veya saga adımıyla yapılır.

Dual-write, yani önce veritabanına sonra doğrudan mesaj sistemine yazıp iki başarıyı ayrı ayrı varsaymak yasaktır.

### 2.3 Transaction yaşam döngüsü

Standart sıra:

1. Identity, tenant ve authorization doğrulanır.
2. Request ve idempotency kimliği doğrulanır.
3. Transaction açılır.
4. Idempotency kaydı kilitlenir veya atomik olarak oluşturulur.
5. Aggregate'lar gerekli concurrency kontrolüyle okunur.
6. Business guard ve invariantlar değerlendirilir.
7. Sahip domainler değişiklikleri uygular.
8. Ledger, outbox ve audit kayıtları oluşturulur.
9. Idempotency sonucu tamamlanır.
10. Transaction commit edilir.
11. Commit sonrası response döndürülür; dış yayın outbox dispatcher tarafından yapılır.

Exception, timeout, cancellation veya conflict halinde transaction rollback edilir. Commit sonucu belirsizse aynı idempotency anahtarıyla server-side sonuç sorgulanmadan işlem yeniden uygulanmaz.

## 3. Transaction kullanım kuralları

- Transaction mümkün olan en kısa süre açık tutulur.
- Transaction içinde kullanıcı girdisi, UI bekleme, uzun CPU işi veya dış HTTP çağrısı yapılmaz.
- Database connection ve transaction tenant context çözülmeden açılamaz.
- Cancellation commit başlamadan önce dikkate alınır; commit sonucu belirsiz hale geldikten sonra kör tekrar yapılmaz.
- Retry yalnızca sınıflandırılmış transient hatalara uygulanır ve bütün transaction yeniden yürütülür.
- Retry sayısı sınırlı, exponential backoff ve jitter içeren bir policy ile yönetilir.
- Validation, authorization, business rejection, unique constraint veya idempotency conflict transient kabul edilmez.
- Deadlock veya serialization failure retry edilebilir; her retry güncel state'i yeniden okur ve kuralları yeniden çalıştırır.
- Uzun raporlama sorguları kritik write transaction'larından ayrılır.

## 4. Concurrency stratejisi

### 4.1 Seçim sırası

Concurrency kontrolü aşağıdaki sırayla seçilir:

| Strateji | Kullanım | Kural |
|---|---|---|
| Atomic database constraint/statement | Benzersizlik, sıfır altına düşmeme, compare-and-set | İlk tercih; invariant veri katmanında da korunur |
| Optimistic concurrency | Çoğu aggregate update'i | Varsayılan; version token ile stale write reddedilir |
| Pessimistic row lock | Kısa, yüksek contention'lı kritik bölüm | Ölçülmüş ihtiyaç ve açık lock sırası gerektirir |
| PostgreSQL advisory lock | Birden fazla satır/kaynak için doğal row lock yoksa | Yalnızca Jira/ADR ile tanımlı key ve kapsamda kullanılır |
| Queue/serialization | Aynı anahtar için sürekli contention | Ölçümle kanıtlanmışsa ve gecikme kabul ediliyorsa |

Uygun stratejiyle korunamayan HIGH/CRITICAL invariant için iş `BLOCKED`; rastgele retry veya son yazan kazanır yaklaşımı çözüm değildir.

### 4.2 Optimistic concurrency

- Mutable aggregate bir version token taşır.
- Update, beklenen version'ı `WHERE` koşulunda doğrular ve başarıda version'ı atomik artırır.
- Etkilenen satır sayısı beklenenden farklıysa `CONCURRENCY_CONFLICT` üretilir.
- Conflict halinde mevcut state yeniden okunmadan otomatik merge yapılmaz.
- Finansal ve stok işlemlerinde conflict, kullanıcının veya tanımlı orchestration'ın açık retry/review davranışına gider.
- Client version değeri yalnızca beklenti belirtebilir; server authoritative current version'ı store'dan okur.

### 4.3 Pessimistic lock ve lock sırası

Pessimistic lock yalnızca optimistic conflict oranı veya invariant gereksinimiyle kanıtlandığında kullanılır.

- Lock kapsamı en dar aggregate/satır kümesiyle sınırlıdır.
- Bütün akışlar aynı deterministik lock sırasını kullanır.
- Lock tutulurken dış servis çağrısı yapılmaz.
- Lock timeout sınırlıdır ve gözlemlenir.
- `SKIP LOCKED` yalnızca iş kuyruğu claim senaryosunda kullanılabilir; business kaydını sessizce atlamak için kullanılamaz.
- Deadlock gözlemlenir, rollback edilir ve yalnızca sınırlı policy ile baştan denenir.

### 4.4 Advisory lock

Advisory lock kullanımı aşağıdakilerin tamamını ister:

- doğal row lock ile korunamayan açık kaynak kimliği,
- collision üretmeyen, tenant dahil deterministik lock key şeması,
- transaction-scoped lock,
- timeout ve telemetry,
- lock sırası,
- çökme/retry testi,
- ilgili Jira işi ve ADR.

Session-scoped ve manuel release'e bağımlı advisory lock varsayılan olarak yasaktır.

## 5. Stok ve finans concurrency koruması

- Movement veya ledger kaydı benzersiz iş/idempotency referansı taşır.
- Aynı logical movement ikinci kez insert edilemez; database unique constraint ile korunur.
- FULL ve EMPTY ayrı değerlerdir ve ayrı invariantlarla doğrulanır.
- Stok projection güncellemesi movement oluşturma ile aynı transaction sınırındadır veya ledger'dan yeniden üretilebilir.
- Oversell koruması yalnızca uygulama ön kontrolüne dayanamaz; atomic statement, constraint, lock veya serializable kontrol ile store katmanında korunur.
- Finansal posting ve ilgili cari/kasa etkisi aynı logical operation kimliğine bağlanır.
- Tamamlanmış kaydı değiştiren conflict çözümü yasaktır; reversal/correction yeni ve izlenebilir kayıt üretir.

## 6. Idempotency contract'ı

### 6.1 Zorunlu alanlar

Finansal, stok, ödeme, provisioning, migration, offline sync ve retry edilebilir diğer kritik command'lar aşağıdaki idempotency bilgilerini taşır:

| Alan | Açıklama |
|---|---|
| `IdempotencyKey` | Client veya trusted producer tarafından üretilen yüksek entropili benzersiz anahtar |
| `OperationType` | Anahtarın hangi command/operation için geçerli olduğu |
| `TenantId` | Server-side çözümlenen tenant kapsamı |
| `ActorId` / producer | Kimlik veya trusted system producer |
| `RequestHash` | Kanonik, hassas veri içermeyen request fingerprint'i |
| `Status` | `Processing`, `Succeeded` veya `FailedFinal` |
| `ResultReference` | Oluşan aggregate/iş sonucu için güvenli referans |
| `ResponseSnapshot` | Gerekliyse güvenli ve boyut sınırına tabi tekrar response'u |
| `CreatedAt` / `CompletedAt` | UTC zamanları |
| `ExpiresAt` | Domain gereksinimine göre tanımlanan saklama sınırı; süre dolmadan silinemez |

Idempotency uniqueness en az `(TenantId, OperationType, IdempotencyKey)` üzerinde database constraint ile korunur. Global control-plane işleminde tenant yerine ilgili authoritative scope kullanılır.

### 6.2 İşleme algoritması

1. Anahtar biçimi ve boyutu doğrulanır.
2. Tenant, operation ve canonical request hash hesaplanır.
3. Idempotency kaydı transaction içinde atomik olarak oluşturulur veya mevcut kayıt kilitlenir.
4. Aynı key ve aynı request daha önce `Succeeded` ise saklanan mantıksal sonuç dönülür; domain etkisi tekrarlanmaz.
5. Aynı key farklı request hash veya operation ile geldiyse `IDEMPOTENCY_CONFLICT` dönülür.
6. `Processing` kayıt için ikinci yürütme başlatılmaz; tanımlı in-progress/retry response'u verilir.
7. İlk yürütme domain etkileriyle birlikte sonucu atomik olarak tamamlar.
8. Final business rejection ikinci denemeyle değişmeyecekse `FailedFinal` saklanabilir; transient teknik hata yeni bir domain etkisi kaydetmez.

Request hash tek başına kimlik değildir ve secret içeremez. JSON property sırası, whitespace veya temsil farkı yanlış conflict üretmeyecek kanonikleştirmeyle hesaplanır.

### 6.3 Saklama ve temizleme

- Saklama süresi, producer'ın en uzun retry/replay penceresinden kısa olamaz.
- Payment webhook event kimlikleri provider replay penceresi ve finansal kanıt gereksinimi boyunca korunur.
- Temizleme yalnızca tamamlanmış ve retention süresi dolmuş kayıtları etkiler.
- Processing kayıtlarının timeout/recovery davranışı açıkça tanımlanır; kör silme yapılmaz.
- Temizleme audit ve metric üretir; authoritative ledger kaydını silemez.

## 7. Duplicate, replay ve sıra koruması

- API retry, message redelivery, offline sync, webhook replay ve kullanıcı çift tıklaması aynı duplicate modeline tabidir.
- HTTP request kimliği idempotency anahtarının yerine geçmez.
- Outbox mesajı ve inbox tüketimi benzersiz `MessageId` ile korunur.
- Provider webhook'u imza doğrulamasından sonra provider event kimliğiyle inbox'a kaydedilir.
- Duplicate event ikinci business etkisi oluşturmaz ancak gözlemlenebilir duplicate sonucu üretir.
- Out-of-order event, event version/sequence/provider zaman bilgisiyle değerlendirilir; eski event güncel state'i geri alamaz.
- Sıra boşluğu varsa iş pending/review state'ine alınır; sessizce başarılı sayılmaz.

## 8. Backend authentication ve authorization

### 8.1 Güven sınırı

Backend aşağıdaki client alanlarına güvenmez:

- `TenantId`, branch, warehouse veya role beyanı,
- kullanıcı/actor kimliği,
- fiyat, indirim veya approval yetkisi,
- record ownership bilgisi,
- license/payment sonucu,
- UI'ın aksiyonu görünür veya etkin göstermesi.

Kimlik doğrulanmış principal/session'dan; tenant ise Identity ve Tenancy kayıtlarından server-side çözülür.

### 8.2 Zorunlu authorization sırası

Her command ve hassas query için:

1. Credential/session doğrulanır ve revoke state kontrol edilir.
2. Server-side tenant context çözülür.
3. Tenant lifecycle ve license enforcement sonucu kontrol edilir.
4. Role/permission doğrulanır.
5. Branch, warehouse, device veya record scope doğrulanır.
6. Aggregate'ın çözülen tenant/store içinde olduğu doğrulanır.
7. Domain guard ve gerekiyorsa approval doğrulanır.
8. İşlem yürütülür; kritik karar audit edilir.

Controller attribute veya route guard tek başına yeterli değildir. Application boundary command/query seviyesinde authorization uygular; sahip domain kritik invariantı ayrıca doğrular.

### 8.3 Yetki hatası ve veri sızıntısı

- Authentication başarısızlığı `401`, doğrulanmış fakat yetkisiz erişim `403` üretir.
- Bir kaydın varlığını açıklamak tenant/record enumeration yaratıyorsa `404` kullanılabilir; policy tutarlı olmalıdır.
- Hata mesajı başka tenant, kullanıcı, bakiye, stok veya iç policy ayrıntısı sızdırmaz.
- Batch işlemlerde her item aynı tenant/scope kontrolünden geçer; bir item yetkisi diğerine taşınmaz.
- Background job, webhook ve message consumer açık service identity ve en az yetkiyle çalışır.
- Support/admin erişimi normal tenant kullanıcısı gibi taklit edilemez; süreli, açık yetkili ve audit edilen ayrı akış ister.

## 9. Güvenlik kontrolleri

- Bütün external input'lar boyut, biçim, enum, aralık ve semantic kurallarla allowlist yaklaşımıyla doğrulanır.
- SQL yalnızca parameterized query/ORM parameter binding ile çalışır.
- Secret, token, parola, connection string ve signing key kaynak koda veya log'a yazılmaz.
- Hassas değerler güvenli secret store'dan alınır ve en az yetkili identity ile erişilir.
- Ödeme webhook'u body işlenmeden önce provider standardına göre imza, timestamp ve replay penceresiyle doğrulanır.
- Authorization kararı cache'leniyorsa revoke ve permission değişikliği için güvenli invalidation/TTL gerekir.
- Rate limit identity, tenant, IP veya operation riskine göre uygulanır; authorization'ın yerine geçmez.
- Request/body limitleri kaynak tüketimi saldırılarını engeller.
- Log ve audit içeriği maskelenir; idempotency key'in tamamı gerekiyorsa hash/güvenli referansla tutulur.
- Database rolleri Master DB ile Tenant DB ayrımına ve least privilege ilkesine uyar.
- Production'da debug hata ayrıntısı ve stack trace client'a dönülmez.

## 10. Hata sınıfları ve retry kararı

| Hata sınıfı | Örnek | Retry |
|---|---|---|
| Validation | Eksik/bozuk input | Hayır |
| Authentication/authorization | Geçersiz session veya yetki | Hayır; kimlik/yetki yenilenmeden denenmez |
| Business rejection | Guard veya izinli transition başarısız | Hayır; veri/karar değişmeden denenmez |
| Idempotent replay | Aynı key ve aynı request tamamlanmış | Etki tekrarlanmaz, önceki sonuç döner |
| Idempotency conflict | Aynı key, farklı request | Hayır |
| Concurrency conflict | Stale version veya lock conflict | Yalnızca güncel state yeniden okunarak sınırlı retry/review |
| Transient infrastructure | Deadlock, serialization, geçici bağlantı | Bounded retry + backoff + jitter |
| Unknown commit result | Timeout sırasında commit belirsiz | İdempotency sonucu sorgulanır; kör tekrar yok |
| Permanent integration | İmza hatası, geçersiz contract | Hayır; DLQ/review ve alarm |

## 11. Audit, telemetry ve alarm

Kritik işlem kaydı en az şunları ilişkilendirir:

- tenant ve actor/service identity,
- operation type ve güvenli aggregate referansı,
- correlation/causation kimlikleri,
- idempotency anahtarının güvenli referansı,
- transaction sonucu ve süre,
- concurrency stratejisi, conflict/retry sayısı,
- authorization sonucu ve policy kimliği,
- outbox/inbox/message kimliği,
- hata sınıfı ve kararlı hata kodu.

Metric'ler en az transaction rollback, deadlock, concurrency conflict, idempotent replay, idempotency conflict, duplicate event, authorization denial, outbox gecikmesi ve retry exhaustion sayılarını kapsar. Secret veya gereksiz kişisel veri telemetry'ye yazılmaz.

Alarm adayları:

- aynı tenant/operation için olağandışı conflict artışı,
- Processing durumunda takılı idempotency kayıtları,
- outbox backlog veya inbox failure artışı,
- webhook signature/replay reddi artışı,
- authorization denial veya cross-tenant deneme artışı,
- deadlock/retry exhaustion,
- stok/finans invariant ihlal girişimi.

## 12. Zorunlu test matrisi

| Alan | Kanıt |
|---|---|
| Transaction başarı | Bütün state, ledger, outbox, audit ve idempotency etkileri birlikte commit olur |
| Transaction rollback | Her failure noktasında hiçbir kısmi etki kalmaz |
| Duplicate command | Aynı key ile tek domain etkisi oluşur |
| Key/payload conflict | Farklı request aynı key ile reddedilir |
| Concurrent update | En fazla izin verilen işlem commit olur; stale işlem conflict alır |
| Oversell yarışı | Paralel talepler store invariantını aşamaz |
| Deadlock/serialization | Bounded retry, güncel state okuma ve tek etki kanıtlanır |
| Unknown commit result | Kör tekrar yerine kayıtlı sonuç çözülür |
| Outbox/inbox replay | Duplicate mesaj ikinci etki oluşturmaz |
| Out-of-order event | Eski event güncel state'i geri alamaz |
| Cross-tenant erişim | Yanlış tenant/store erişimi reddedilir ve veri sızmaz |
| Revoked session | Cache olsa bile revoked identity işlem yapamaz |
| Scope authorization | Branch/warehouse/record sınırı backend'de uygulanır |
| Webhook güvenliği | Geçersiz imza, eski timestamp ve replay reddedilir |
| Hassas veri | Response, log, audit ve telemetry'de secret bulunmaz |

HIGH ve CRITICAL işlerde concurrency yarışları gerçek paralel yürütmeyle; atomiklik failure injection ile; idempotency duplicate/replay testleriyle kanıtlanır. Yalnızca happy-path unit test yeterli değildir.

## 13. Review kontrol listesi

- Transaction sınırı business işleminin bütün zorunlu etkilerini kapsıyor mu?
- Cross-database/dış sistem sınırında outbox/inbox kullanılıyor mu?
- Domain ownership korunuyor mu?
- Concurrency stratejisi seçim matrisine ve ölçüme dayanıyor mu?
- Database constraint/atomic operation ile korunabilecek invariant yalnızca uygulamaya bırakılmış mı?
- Idempotency uniqueness tenant ve operation scope'unu içeriyor mu?
- Aynı key/farklı payload conflict üretiyor mu?
- Duplicate/replay/out-of-order ikinci veya geri alıcı etki oluşturabiliyor mu?
- Identity, tenant, role ve resource scope backend'de yeniden doğrulanıyor mu?
- Client `TenantId`, role, fiyat veya payment sonucuna güveniliyor mu?
- Retry yalnızca transient sınıfa, bounded policy ile uygulanıyor mu?
- Unknown commit result kör tekrar üretebiliyor mu?
- Secret veya hassas veri log/audit/response'a sızıyor mu?
- HIGH/CRITICAL risk için paralel test, failure injection ve Golden Scenario kanıtı var mı?

Eksik mekanizma `REQUEST_CHANGES`; tenant izolasyonu, ownership, finans/stok bütünlüğü veya fail-closed güvenlik ihlali `BLOCKED`; yeni business rule, kritik lock stratejisi veya güvenlik politikası kararı gerekiyorsa `SPEC_REQUIRED` ya da `HUMAN_REQUIRED` sonucunu üretir.
