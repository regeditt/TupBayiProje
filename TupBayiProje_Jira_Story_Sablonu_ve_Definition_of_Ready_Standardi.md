# TupBayiProje Jira Story Şablonu ve Definition of Ready Standardı

**Jira:** `TBP-8`  
**Durum:** Normatif  
**Kapsam:** Story ve Story niteliğindeki uygulama, entegrasyon, tasarım, test ve operasyon işleri  
**Bağlı belgeler:** `TupBayiProje_Global_Invariantlar.md`, `TupBayiProje_Source_of_Truth_ve_Domain_Ownership.md`, `TupBayiProje_State_Machine_ve_Yasam_Dongusu_Standardi.md`, `TupBayiProje_Transaction_Concurrency_Idempotency_ve_Guvenlik_Standardi.md`, `TupBayiProje_Kodlama_Dili_ve_Adlandirma_Standardi.md`, `TupBayiProje_Definition_of_Done_ve_Risk_Bazli_Kalite_Kapilari_Standardi.md`, `TupBayiProje_Test_Oracle_ve_Golden_Scenario_Standardi.md`, `TupBayiProje_Dependency_DAG_ve_Faz_GATE_Politikasi.md`

Bu standart, bir Story uygulamaya alınmadan önce gereksinimin eksiksiz, izlenebilir ve test edilebilir olmasını sağlar. Canlı Jira kaydı iş gereksiniminin Source of Truth'udur. Bu belge Jira açıklamasının yerine geçmez; açıklamanın nasıl tamamlanacağını ve hazırlık kararının nasıl verileceğini tanımlar.

## 1. Temel kurallar

1. Story açıklamasında bu belgede tanımlanan bütün zorunlu alanlar bulunur.
2. Alan ilgisizse boş bırakılmaz; `N/A - <neden ilgisiz>` biçiminde somut gerekçe yazılır.
3. `N/A` tek başına, `yok`, `gerekmez` veya benzeri gerekçesiz cevaplar kabul edilmez.
4. Bilinmeyen bilgi varsayımla doldurulmaz. Eksik bilgi `SPEC_REQUIRED`, gerekli insan kararı `HUMAN_REQUIRED`, ön koşul eksiği `BLOCKED` sonucunu üretir.
5. Story, canlı Jira bağlantıları ve yorumlarıyla birlikte değerlendirilir. Yerel ticket listesi gereksinim veya sıra kaynağı değildir.
6. Kabul kriterleri gözlenebilir sonuç tarif eder; implementasyon adımlarını veya yalnız happy path'i tekrar etmez.
7. HIGH ve CRITICAL riskli Story, somut Test Oracle veya Golden Scenario olmadan hazır sayılmaz.
8. Design Required veya Human Gate gerektiren iş, ilgili kanıt ya da onay bağlanmadan implementasyona alınmaz.

## 2. Zorunlu alanlar

| Alan | Zorunlu içerik | Hazırlık reddi örneği |
|---|---|---|
| İş Hedefi | Çözülecek problem, beklenen business sonucu ve başarı ölçütü | Yalnız teknik faaliyet yazılması |
| Aktör | İşlemi başlatan insan, cihaz veya trusted system actor | `Kullanıcı` gibi belirsiz rol |
| Tetikleyici | Akışı başlatan olay, komut veya zamanlama | Başlangıç koşulunun olmaması |
| Gerçek Kaynak | Authoritative veri veya karar kaynağı ve ilgili SOT/invariant/Jira referansı | UI, cache veya local kopyanın kaynak sayılması |
| Veri Sahibi | Veriyi yazmaya yetkili domain ve store | Birden fazla yazma sahibi tanımlanması |
| Normal Akış | Ön koşuldan gözlenebilir sonuca kadar numaralı temel akış | Yalnız ekran veya endpoint adı |
| State Model | Başlangıç state'i, izinli transition'lar, guard'lar ve terminal durumlar | State değişiminin belirsiz olması |
| Negatif Senaryolar | Yetkisiz, geçersiz, duplicate, stale ve sınır durumları | Yalnız happy path |
| Offline/Cache | Offline davranış, freshness, invalidation ve authoritative fallback | Cache'in gerçek kaynak gibi kullanılması |
| Security | Authentication, authorization, tenant ve resource scope kontrolleri | Kontrolün yalnız UI'a bırakılması |
| Transaction | Atomik etkiler, transaction sınırı ve rollback davranışı | Kısmi başarının tanımsız olması |
| Concurrency | Yarış senaryosu, korunan invariant ve conflict davranışı | Silent last-write-wins |
| Idempotency | Anahtar kapsamı, duplicate/replay sonucu ve payload conflict davranışı | Retry'nin ikinci etki üretebilmesi |
| Failure | Beklenen hata sınıfları, kararlı hata sonucu, retry ve recovery | Belirsiz `bir hata oluştu` davranışı |
| Audit | Kim, neyi, hangi sonuç ve gerekçeyle yaptı kanıtı | Kritik kararın izsiz kalması |
| Observability | Log, metric, trace ve alarm kanıtı; hassas veri sınırı | Secret veya gereksiz kişisel veri loglama |
| Acceptance Criteria | Given/When/Then veya eşit kesinlikte gözlenebilir koşullar | `Çalışmalı` gibi ölçülemez ifade |
| Test Matrisi | Unit, integration, contract, security, concurrency, idempotency ve E2E gereksiniminin risk bazlı kapsamı | Riskli yolun test seviyesinin belirtilmemesi |
| Test Oracle | Beklenen sonucu hesaplayan authoritative kural, somut örnek veya Golden Scenario | Testin aynı hatalı algoritmayı tekrar etmesi |
| Yasak Uygulamalar | Bu Story'de kabul edilmeyecek shortcut, bypass ve veri sahipliği ihlalleri | Kritik yasakların belirtilmemesi |
| Design Required | `Evet`/`Hayır`; evetse onaylı Figma kaynağı ve kapsanan state'ler, hayırsa gerekçe | UI etkisi varken tasarım kararının olmaması |
| Human Gate | `Gerekli`/`Gerekli Değil`; gerekliyse karar konusu, onay mercii ve kanıt, değilse gerekçe | Kritik değişikliğin otomatik kabul edilmesi |
| Risk Seviyesi | `LOW`, `MEDIUM`, `HIGH` veya `CRITICAL` ve kısa gerekçe | Seviyenin gerekçesiz seçilmesi |

## 3. Jira Story şablonu

Aşağıdaki şablon Story açıklamasına kopyalanır. Başlıklar silinemez veya boş bırakılamaz.

```markdown
## İş Hedefi

<Problem, beklenen business sonucu ve başarı ölçütü>

## Aktör

<Rol, cihaz veya trusted system actor>

## Tetikleyici

<Akışı başlatan olay, komut veya zamanlama>

## Gerçek Kaynak

<Authoritative kaynak ve SOT/invariant/Jira referansı>

## Veri Sahibi

<Domain ve store>

## Normal Akış

1. <Ön koşul ve ilk adım>
2. <Doğrulama ve domain davranışı>
3. <Gözlenebilir sonuç>

## State Model

<Başlangıç state'i, izinli transition, guard ve terminal durum>

## Negatif Senaryolar

- <Yetkisiz veya yanlış scope>
- <Geçersiz state veya input>
- <Duplicate, replay, stale veya concurrency>
- <Bağımlılık ya da altyapı hatası>

## Offline/Cache

<Davranış veya N/A - somut gerekçe>

## Security

<Authentication, authorization, tenant ve resource scope>

## Transaction

<Atomik etkiler, sınır ve rollback>

## Concurrency

<Yarış, korunan invariant ve conflict sonucu>

## Idempotency

<Anahtar kapsamı, duplicate/replay ve payload conflict>

## Failure

<Hata sınıfları, kararlı sonuç, retry ve recovery>

## Audit

<Kaydedilecek karar, actor, neden ve sonuç>

## Observability

<Log, metric, trace, alarm ve hassas veri sınırı>

## Acceptance Criteria

1. Given <başlangıç>, when <eylem>, then <gözlenebilir sonuç>.
2. Given <negatif başlangıç>, when <eylem>, then <kararlı ret ve sıfır kısmi etki>.

## Test Matrisi

| Test seviyesi | Senaryo ve kanıt |
|---|---|
| Unit | <Kural ve guard> |
| Integration | <Store, transaction ve dış sınır> |
| Contract | <API/event uyumluluğu veya N/A - gerekçe> |
| Security | <Yetki, tenant ve veri sızıntısı> |
| Concurrency | <Gerçek paralel yarış veya N/A - gerekçe> |
| Idempotency | <Duplicate/replay veya N/A - gerekçe> |
| E2E | <Kritik kullanıcı akışı veya N/A - gerekçe> |

## Test Oracle

<Authoritative kural ve somut beklenen sonuç; HIGH/CRITICAL için `TupBayiProje_Test_Oracle_ve_Golden_Scenario_Standardi.md` ile uyumlu Golden Scenario>

## Yasak Uygulamalar

- <Bu Story'de yasak shortcut veya bypass>

## Design Required

<Evet - Figma kaynağı ve state'ler | Hayır - somut gerekçe>

## Human Gate

<Gerekli - konu, onay mercii ve kanıt | Gerekli Değil - somut gerekçe>

## Risk Seviyesi

<LOW | MEDIUM | HIGH | CRITICAL> - <gerekçe>
```

## 4. Definition of Ready

Story ancak aşağıdaki koşulların tamamı sağlandığında `READY` kabul edilir:

- Canlı Jira açıklaması, kabul kriterleri, bağlantıları ve yorumları okunabilir.
- Zorunlu 23 alanın tamamı dolu veya açıklamalı `N/A` değerine sahip.
- İş hedefi, kapsam ve gözlenebilir sonuç birbiriyle tutarlı.
- Gerçek kaynak ve veri sahibi Source of Truth kaydıyla uyumlu.
- State, transaction, concurrency ve idempotency etkileri ilgili standartlarla uyumlu.
- Negatif senaryolar ile failure/recovery davranışı tanımlı.
- Acceptance Criteria test edilebilir ve Test Matrisi bunları kapsıyor.
- HIGH/CRITICAL riskte somut Test Oracle veya Golden Scenario mevcut.
- Blocks ilişkileri doğru yönlü ve tamamlanmamış ön koşul yok.
- Önceki faz GATE tamamlanmış; mevcut faz GATE bypass edilmiyor.
- Design Required ise onaylı tasarım kaynağı ve zorunlu state'ler bağlı.
- Human Gate gerekli ise karar ve kanıt Jira'da kayıtlı.
- Yasak uygulamalar ve kapsam dışı davranışlar açık.
- Secret, hassas veri veya güven sınırını zayıflatan talimat bulunmuyor.

## 5. Hazırlık sonucu

| Sonuç | Anlam |
|---|---|
| `READY` | Bütün DoR koşulları karşılandı; iş uygulanabilir. |
| `SPEC_REQUIRED` | Business davranışı, kabul kriteri veya zorunlu alan eksik ya da belirsiz. |
| `BLOCKED` | Tamamlanmamış ön koşul, Blocks bağlantısı veya faz GATE engeli var. |
| `HUMAN_REQUIRED` | Geri dönüşü zor, kritik veya yetki gerektiren karar insan onayı bekliyor. |
| `REQUEST_CHANGES` | İçerik bilinen standarda aykırı; düzeltilmeden uygulanamaz. |

Ready değerlendirmesi Jira yorumu veya ilgili workflow kanıtıyla kaydedilir. `READY` kararı eksik alanı ortadan kaldırmaz; daha sonra ortaya çıkan çelişki işi yeniden `SPEC_REQUIRED`, `BLOCKED` veya `HUMAN_REQUIRED` durumuna alır.

## 6. Kapsam sınırı

Bu belge Story'nin implementasyona hazır olma koşulunu tanımlar. Risk seviyesine göre Definition of Done ve kalite kapıları `TupBayiProje_Definition_of_Done_ve_Risk_Bazli_Kalite_Kapilari_Standardi.md` (`TBP-9`), Test Oracle ve Golden Scenario ayrıntıları `TupBayiProje_Test_Oracle_ve_Golden_Scenario_Standardi.md` (`TBP-10`), dependency DAG ve faz GATE politikası `TupBayiProje_Dependency_DAG_ve_Faz_GATE_Politikasi.md` (`TBP-11`) kapsamındadır. Bu belge bu standartların yerine yeni politika üretmez.
