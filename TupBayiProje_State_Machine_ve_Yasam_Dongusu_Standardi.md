# TupBayiProje State Machine ve Yaşam Döngüsü Standardı

**Jira:** `TBP-4`  
**Durum:** Normatif  
**Bağlı belgeler:** `TupBayiProje_Global_Invariantlar.md`, `TupBayiProje_Source_of_Truth_ve_Domain_Ownership.md`

Bu belge, yaşam döngüsü bulunan domain kayıtlarının durumlarının nasıl tanımlanacağını ve değiştirileceğini belirler. Amaç; serbest metin durumları, denetimsiz atamaları, geçersiz geçişleri ve geçmişi gizleyen düzeltmeleri engellemektir. Bu standart business transition üretmez; her somut state machine'in geçişleri ilgili Jira gereksiniminden türetilir.

## 1. Temel kurallar

1. Yaşam döngüsü olan her aggregate'ın tek bir sahibi ve tek authoritative state alanı vardır.
2. State serbest `string` değildir; kapalı bir enum veya eşdeğer kapalı tip ile temsil edilir.
3. State yalnızca aggregate'ın sahibi olan domain içindeki tanımlı transition operasyonuyla değiştirilebilir.
4. Public veya genel amaçlı state setter'ı bulunamaz. Repository, ORM, migration, istemci veya başka bir domain transition kuralını atlayamaz.
5. Her transition açık bir kaynak state, hedef state, tetikleyici, önkoşul, yetki, yan etki ve hata davranışı tanımlar.
6. Tanımlanmamış transition güvenli biçimde reddedilir; fail-open veya sessiz no-op uygulanamaz.
7. Terminal state'ten çıkış yalnızca açıkça tanımlanmış bir correction/reversal transition'ı ile mümkündür. Kayıt silinerek veya state yerinde geri alınarak geçmiş değiştirilemez.
8. Cross-domain tetikleyici, sahibi olan domain'in command/application contract'ını çağırır. Başka domain state alanına doğrudan yazamaz.
9. State adı, anlamı veya transition grafiği değişikliği Jira Change Request, etki analizi, migration/rollback planı ve gerekli Human Gate olmadan yapılamaz.

## 2. Zorunlu state machine kayıt şeması

Her state machine, uygulamadan önce aşağıdaki alanlarla kayıt altına alınır:

| Alan | Zorunluluk | Açıklama |
|---|---|---|
| `MachineId` | Zorunlu | Değişmez ve benzersiz kimlik; örnek biçim `SM-SAL-001` |
| Sahip domain | Zorunlu | State'i değiştirebilen tek domain |
| Aggregate | Zorunlu | Yaşam döngüsü yönetilen aggregate |
| Authoritative kaynak | Zorunlu | Source of Truth kayıt kimliği ve fiziksel store |
| Initial state | Zorunlu | Aggregate oluşturulduğunda geçerli tek başlangıç durumu |
| States | Zorunlu | Kanonik state adı, anlamı ve terminal olup olmadığı |
| Transitions | Zorunlu | Kaynak, hedef, tetikleyici ve kurallar |
| Error namespace | Zorunlu | Bu machine için kararlı hata kodu öneki |
| Audit sınıfı | Zorunlu | Kaydedilecek olay ve veri sınıflandırması |
| Jira kaynağı | Zorunlu | Business rule'un geldiği Jira işi |
| Test Oracle | HIGH/CRITICAL için zorunlu | Geçiş grafiğini kanıtlayan test veya Golden Scenario |

State machine tanımında kaynak veya sahibi belirsizse implementasyon `SPEC_REQUIRED`; ownership kaydıyla çelişiyorsa `BLOCKED` olur.

## 3. Transition tanımı

Her transition aşağıdaki contract ile tanımlanır:

| Alan | Açıklama |
|---|---|
| `TransitionId` | Machine içinde benzersiz, değişmez kimlik |
| `From` | İzin verilen kaynak state veya açıkça tanımlanmış kaynak state kümesi |
| `To` | Tek hedef state |
| `Trigger` | Command, doğrulanmış event veya zaman tabanlı worker tetikleyicisi |
| `Actor` | İşlemi başlatabilecek rol veya sistem principal'ı |
| `Guard` | State dışında doğrulanması gereken business önkoşulları |
| `Authorization` | Server-side kimlik, tenant ve yetki kontrolü |
| `Idempotency` | Tekrar çağrının aynı domain etkisini üretmesini engelleyen kural |
| `Effects` | Aynı transaction içindeki state, ledger, outbox ve audit etkileri |
| `Failure` | Kararlı hata kodu ve hiçbir kısmi etki bırakmama davranışı |
| `Risk` | LOW, MEDIUM, HIGH veya CRITICAL |

Birden fazla kaynak state aynı hedefe gidebiliyorsa her kaynak açıkça listelenir. `ANY -> X`, joker transition ve örtük fallback yasaktır.

## 4. İşleme sırası ve atomiklik

Transition isteği aşağıdaki sırayla değerlendirilir:

1. Doğrulanmış identity ve server-side tenant context çözülür.
2. Aggregate doğru tenant store'undan yüklenir; bulunamama bilgisi yetkisiz veri sızdırmayacak biçimde ele alınır.
3. Authorization ve actor kuralı doğrulanır.
4. Idempotency anahtarı ve daha önceki sonuç kontrol edilir.
5. Beklenen/current state ve concurrency version doğrulanır.
6. Transition'ın `From -> To` olarak tanımlı olduğu doğrulanır.
7. Guard ve business önkoşulları değerlendirilir.
8. Sahip domain state değişikliğini ve aynı işlemde zorunlu etkileri uygular.
9. Audit kaydı ve gerekiyorsa outbox kaydı aynı transaction sınırında kalıcılaştırılır.
10. Commit sonrasında sonuç, yeni state ve correlation bilgisiyle döndürülür.

Başarısız olan herhangi bir adım state, ledger, outbox veya projection üzerinde kısmi etki bırakamaz. Cross-database süreçlerde sahip state'in atomik kaydı ile durable outbox aynı transaction'da tutulur; uzak etki retry edilebilir ve görünür bir süreç state'i taşır.

## 5. Geçersiz transition davranışı

Geçersiz transition:

- state'i değiştirmez,
- yan etki, ledger kaydı veya domain event üretmez,
- başarılı veya idempotent replay gibi raporlanmaz,
- correlation kimliğiyle güvenli audit/telemetry üretir,
- istemciye kararlı bir hata kodu verir,
- hassas state veya başka tenant verisi sızdırmaz.

İstek mevcut state'e geçiş talep ediyorsa yalnızca aynı idempotency anahtarına ait daha önce başarıyla tamamlanmış işlemin replay'i başarılı sonuç döndürebilir. Bunun dışındaki `X -> X` isteği, transition açıkça tanımlı değilse geçersizdir.

## 6. Hata kodu standardı

Domain hataları HTTP durumundan bağımsız, makinece okunabilir ve geriye uyumlu kod taşır.

| Kod biçimi | Anlam | Önerilen HTTP karşılığı |
|---|---|---|
| `<DOMAIN>_STATE_INVALID_TRANSITION` | Mevcut state'ten istenen geçiş tanımlı değil | `409 Conflict` |
| `<DOMAIN>_STATE_PRECONDITION_FAILED` | Guard veya business önkoşulu sağlanmadı | `422 Unprocessable Content` |
| `<DOMAIN>_STATE_CONCURRENCY_CONFLICT` | Beklenen version/state güncel değil | `409 Conflict` |
| `<DOMAIN>_STATE_NOT_FOUND` | Aggregate bulunamadı veya görünür değil | `404 Not Found` |
| `<DOMAIN>_STATE_FORBIDDEN` | Actor transition için yetkili değil | `403 Forbidden` |
| `<DOMAIN>_STATE_IDEMPOTENCY_CONFLICT` | Aynı anahtar farklı payload/amaçla kullanıldı | `409 Conflict` |

Hata response'u en az `code`, kullanıcıya güvenli `message`, `correlationId` ve uygunsa yeniden okunabilir `currentState` içerir. Stack trace, secret, token, kişisel veri veya başka tenant bilgisi dönülmez. Hata kodunun anlamı değiştirilemez; yeni anlam için yeni kod eklenir.

## 7. Audit ve gözlemlenebilirlik

Başarılı transition audit kaydı en az şu alanları içerir:

- `MachineId` ve `TransitionId`,
- aggregate türü ve güvenli kimliği,
- önceki ve yeni state,
- tenant, actor ve cihaz/session referansı,
- UTC timestamp,
- correlation ve causation kimlikleri,
- idempotency anahtarının güvenli referansı,
- kaynak command/event türü,
- sonuç ve risk sınıfı.

Reddedilen HIGH/CRITICAL transition denemeleri de neden sınıfı ve correlation bilgisiyle audit edilir. Audit içine secret, parola, token, tam ödeme verisi veya gereksiz kişisel veri yazılmaz. Audit kaydı authoritative business state değildir ve state'i yeniden yazmak için kullanılamaz.

## 8. Event, retry ve sıra kuralları

- State değişikliğini bildiren event, commit edilmemiş state için yayımlanamaz.
- Event en az aggregate kimliği, yeni state, state version, occurred-at ve event kimliği taşır.
- Consumer, event'i tekrar aldığında ikinci bir domain etkisi oluşturamaz.
- Daha eski version'a ait event güncel projection veya state'i geri alamaz.
- Event tüketicisi authoritative state'i sahip domain dışında değiştiremez.
- Retry, transition kuralını ve güncel authorization/tenant kontrollerini atlayamaz.

## 9. Persistence ve migration kuralları

1. State store'da kanonik ve kararlı bir değerle saklanır; kullanıcıya gösterilen çeviri metni state değeri değildir.
2. Bilinmeyen state değeri varsayılan bir state'e çevrilmez. Okuma fail-closed olur ve operasyonel alarm üretir.
3. Database constraint uygulanabiliyorsa izin verilen değerleri korur; tek başına transition grafiğinin yerine geçmez.
4. Yeni state ekleme veya state kaldırma; eski ve yeni uygulama sürümlerinin birlikte çalışmasını, veri backfill'ini ve rollback'i kapsayan migration planı ister.
5. State silinmez veya yeniden adlandırılmaz; gerekiyorsa yeni state eklenir, eski state deprecated edilir ve açık migration ile taşınır.
6. Projection state'i authoritative state'ten yeniden üretilebilir olmalıdır.

## 10. Uygulama sahipliği

- Transition fonksiyonu aggregate veya sahip domain'in application/domain service katmanında bulunur.
- API/controller yalnızca input mapping, authentication boundary ve sonuç mapping yapar; transition kuralını barındırmaz.
- Repository state'i doğrudan set eden genel amaçlı update sunmaz.
- Background worker ve webhook handler aynı transition contract'ını kullanır.
- UI izin verilen aksiyonları gösterebilir fakat transition yetkisini belirleyen kaynak değildir; server her isteği yeniden doğrular.
- Raporlama, bildirim ve projection consumer'ları state değişikliği başlatamaz; bunun için sahibin açık command contract'ını kullanır.

## 11. Zorunlu test matrisi

Her state machine için en az aşağıdaki kanıtlar sağlanır:

| Test | Beklenen kanıt |
|---|---|
| Her allowed transition | Doğru kaynakta hedef state ve zorunlu etkiler birlikte oluşur |
| Her invalid transition sınıfı | State ve yan etkiler değişmez, doğru kararlı hata kodu döner |
| Authorization reddi | State değişmez ve veri sızıntısı olmaz |
| Idempotent replay | Tek domain etkisi ve aynı mantıksal sonuç oluşur |
| Aynı anahtar/farklı payload | Idempotency conflict oluşur |
| Concurrency yarışı | En fazla bir geçiş commit olur; diğeri conflict alır |
| Event retry/out-of-order | Duplicate etki oluşmaz, eski version geri alma yapmaz |
| Transaction rollback | State, ledger, outbox ve audit kısmi kalmaz |
| Terminal state | Tanımsız çıkış reddedilir |
| Bilinmeyen persisted state | Fail-closed ve alarm davranışı görülür |

HIGH ve CRITICAL transition'lar somut Test Oracle veya Golden Scenario olmadan tamamlanmış sayılamaz.

## 12. Review kontrol listesi

- State machine'in Jira kaynağı ve sahibi açık mı?
- State kapalı tip mi; serbest string veya public setter var mı?
- Initial, terminal ve bütün allowed transition'lar açık mı?
- Joker veya örtük transition bulunuyor mu?
- Authorization, tenant, idempotency ve concurrency kontrolleri server-side mı?
- Geçersiz transition hiçbir kısmi etki bırakmadan kararlı hata kodu üretiyor mu?
- State, zorunlu ledger/outbox/audit etkileriyle doğru transaction sınırında mı?
- Audit alanları yeterli ve hassas veriden arındırılmış mı?
- Replay ve out-of-order olaylar güncel state'i geri alabiliyor mu?
- Migration, compatibility ve rollback planı var mı?
- Risk sınıfına uygun test kanıtı mevcut mu?

Bu kontrollerden biri karşılanmıyorsa iş `REQUEST_CHANGES`; ownership, tenant sınırı veya değiştirilemez invariant ihlal ediliyorsa `BLOCKED`; eksik business transition ya da kritik yönetişim kararı gerekiyorsa `SPEC_REQUIRED` veya `HUMAN_REQUIRED` olur.
