# TupBayiProje Dependency DAG ve Faz GATE Politikası

**Jira:** `TBP-11`  
**Durum:** Normatif  
**Kapsam:** Canlı Jira `TBP` projesindeki iş seçimi, `Blocks` ilişkileri, faz geçişleri, claim ve dependency graph değişiklikleri  
**Bağlı belgeler:** `AGENTS.md`, `TupBayiProje_Jira_Story_Sablonu_ve_Definition_of_Ready_Standardi.md`, `TupBayiProje_Definition_of_Done_ve_Risk_Bazli_Kalite_Kapilari_Standardi.md`, `TupBayiProje_Test_Oracle_ve_Golden_Scenario_Standardi.md`, `docs/adr/ADR-012-Jira-Gereksinim-Kaynagi.md`

Bu politika, hangi işin ne zaman uygulanabilir olduğunu belirleyen kanonik dependency DAG'ı ve faz GATE kurallarını tanımlar. Graph'ın authoritative kaynağı canlı Jira'daki issue durumları, `Blocks` bağlantıları, açıklamalar ve yorumlardır. Bu belge graph okuma ve doğrulama kuralıdır; yerel ticket listesi veya bu belgedeki snapshot canlı Jira'nın yerine geçmez.

## 1. Değiştirilemez ilkeler

1. `Blocks` ilişkisi yalnızca **önkoşul → bağımlı iş** yönünde kurulur. Jira'da A `blocks` B ise A önkoşul, B bağımlı iştir.
2. Self-loop, circular dependency ve tamamlanması kendi sonucuna bağlı olan örtük çevrim yasaktır.
3. Yüksek numaralı bir fazdan düşük numaralı bir faza dependency kurulamaz. Sonraki faz, önceki fazın önkoşulu yapılamaz.
4. Faz geçişi yalnız tamamlanmış GATE üzerinden yapılır. Faz içindeki işlerin tek tek tamamlanması GATE'i kendiliğinden geçmiş saydırmaz.
5. Her fazın tam olarak bir çıkış GATE issue'su bulunur. GATE, fazın bütün zorunlu işlerinden gelen `Blocks` bağlantılarını ve doğrulama kanıtını taşır.
6. Rank yalnız uygun işler arasındaki sırayı belirler; dependency, GATE, durum veya claim engelini geçersiz kılamaz.
7. Duplicate, obsolete, superseded, iptal edilmiş veya başka bir kanonik issue'ya yönlendirilmiş iş claim edilemez.
8. Canlı Jira okunamıyorsa, gerekli issue alanı eksikse veya graph uygunluğu kesin belirlenemiyorsa iş seçilmez ve dosya değişikliği yapılmaz.
9. Graph ihlali sessizce düzeltilmiş varsayılmaz. İhlal Jira'da görünür hale getirilir ve düzeltilene kadar etkilenen iş `BLOCKED` kabul edilir.

## 2. Kanonik graph modeli

Dependency graph `G = (V, E)` yönlü graph'ıdır:

- `V`, canlı `TBP` projesindeki aktif plan kapsamına giren issue'lardır.
- `E`, Jira `Blocks` bağlantılarıdır.
- `A → B`, A tamamlanmadan B'nin başlayamayacağını ifade eder.
- `phase(v)`, issue'nun `p00`–`p17` faz etiketidir.
- GATE düğümü, `gate` etiketi ve `[Pxx-GATE]` özetiyle fazın çıkış doğrulamasını temsil eder.

Parent/Epic ilişkisi kapsam gruplamasıdır; tek başına dependency değildir. Rank sıralamadır; tek başına dependency değildir. Açıklamadaki serbest metin, eksik bir `Blocks` bağlantısının yerine geçmez. Gerçek bir önkoşul varsa canlı Jira'da doğru yönlü `Blocks` bağlantısı bulunmalıdır.

### 2.1 Durum semantiği

- Bir önkoşul yalnız Jira status category'si `Done` olduğunda tamamlanmış sayılır.
- `Yapılacaklar`, `DEVAM EDİYOR` veya başka bir tamamlanmamış kategori dependency'yi açık tutar.
- Resolution, yorum veya bağlantı issue'nun duplicate/obsolete olduğunu gösteriyorsa status açık görünse bile issue uygun değildir.
- Reopen edilen önkoşul, kendisine bağlı tamamlanmamış işlerin uygunluğunu anında geri çeker. Etkilenmiş devam eden işler yeniden değerlendirilir ve gerekirse `BLOCKED` tutulur.

## 3. İş uygunluğu ve deterministik seçim

Kullanıcı belirli bir ticket vermediyse seçim canlı Jira'da aşağıdaki sırayla yapılır:

1. `TBP` erişimi ve proje kimliği doğrulanır.
2. Çözülmemiş işler Jira Rank sırasıyla alınır; pagination varsa bütün gerekli sayfalar okunur.
3. Her aday için tam issue, status, description, acceptance criteria, issue links ve tüm yorumlar okunur.
4. Adayın kendisi ve bütün inward `is blocked by` önkoşulları incelenir.
5. Adayın faz girişini açan bütün GATE önkoşullarının `Done` olduğu doğrulanır.
6. Duplicate/obsolete işaretleri ve aktif claim/lock/heartbeat çakışması olmadığı doğrulanır.
7. Graph üzerinde cycle ve geri faz dependency ihlali olmadığı doğrulanır.
8. Bütün kontrolleri geçen ilk Rank'li issue seçilir. Aynı Rank varsa Jira'nın döndürdüğü kararlı issue key sırası tie-breaker olarak kullanılır.
9. Claim veya status değişikliğinden hemen önce issue yeniden okunur. Arada dependency, durum, yorum veya claim değişmişse uygunluk baştan hesaplanır.

Bir issue ancak şu predicate bütünü doğruysa `READY_FOR_CLAIM` olur:

```text
READY_FOR_CLAIM(issue) =
  unresolved(issue)
  AND actionable(issue)
  AND all_inward_blocks_done(issue)
  AND all_required_phase_gates_done(issue)
  AND NOT duplicate_or_obsolete(issue)
  AND NOT claimed_by_other_worker(issue)
  AND dag_is_valid_for(issue)
```

Epic/faz issue'su iş gruplama düğümüdür ve doğrudan uygulama işi olarak claim edilmez. GATE yalnız kendi kapanış koşulları sağlandığında doğrulama işi olarak seçilebilir.

## 4. Dependency doğrulama kuralları

### 4.1 Yön kontrolü

Bir iş B, A'nın çıktısına ihtiyaç duyuyorsa ilişki A `blocks` B biçiminde kurulur. B `blocks` A biçimindeki ters bağlantı uygunluk anlamını bozar ve `REQUEST_CHANGES` üretir.

### 4.2 Cycle kontrolü

Graph değişikliğinde ve claim öncesinde en az şu kontroller yapılır:

- `A → A` self-loop reddedilir.
- Yeni `A → B` kenarı eklenmeden önce B'den A'ya erişilebilir yol aranır; varsa kenar cycle oluşturduğu için reddedilir.
- Aktif graph topological sort ile doğrulanır. Bütün düğümler sıralanamıyorsa kalan strongly connected component'ler cycle kanıtıdır.
- Cycle içindeki ve cycle'a bağımlı işler, bağlantılar düzeltilene kadar claim edilemez.

### 4.3 Geri faz dependency kontrolü

`A → B` için `phase(A) > phase(B)` olamaz. Aynı faz içi dependency ve daha erken fazdan daha geç faza dependency geçerlidir. Faz etiketi eksik veya birbiriyle çelişkiliyse graph geçerli kabul edilmez; Jira kaydı düzeltilene kadar issue `SPEC_REQUIRED` veya `BLOCKED` olur.

### 4.4 Duplicate ve obsolete kontrolü

- Duplicate issue, Jira'daki duplicate bağlantısı, resolution'ı veya yetkili yorumuyla kanonik issue'ya yönlendirilir; iki issue birden claim edilmez.
- Obsolete/superseded/iptal edilmiş iş yeniden açılıp gereksinimi güncellenmeden claim edilmez.
- Aynı hedefi tarif eden fakat resmi duplicate bağlantısı bulunmayan iki açık issue otomatik olarak birleştirilmez. Çakışma `HUMAN_REQUIRED` olarak kaydedilir.
- Geçmiş issue silinmez; kanonik issue bağlantısı ve karar gerekçesi korunur.

## 5. Faz GATE politikası

Her Pxx fazı şu yapıyı taşır:

```text
önceki zorunlu GATE'ler → Pxx faz girişi → faz işleri → Pxx-GATE → bağımlı faz girişleri
```

### 5.1 Faz girişi

- P00 başlangıç fazıdır; önceki faz GATE'i yoktur.
- Diğer fazlar yalnız kendi faz issue'suna inward `Blocks` ile bağlı bütün GATE'ler `Done` olduğunda açılır.
- Bir önceki sayısal fazın GATE'i her zaman tek önkoşul olmak zorunda değildir. Canlı Jira'daki kanonik graph paralel dalları ve birleşme noktalarını belirler.
- Doğrudan bir alt işe bağlantı eklemek, faz giriş GATE'ini atlama yetkisi vermez.

### 5.2 Faz çıkışı

Faz GATE'i ancak aşağıdakilerin tamamı sağlandığında `Done` olabilir:

1. Faz kapsamındaki bütün zorunlu işler GATE'i doğru yönde bloklar ve `Done` durumundadır.
2. GATE açıklamasındaki çıkış kanıtları eksiksizdir.
3. Açık `REQUEST_CHANGES`, `SPEC_REQUIRED`, `BLOCKED` veya `HUMAN_REQUIRED` kararı yoktur.
4. Risk bazlı kalite kapıları ve gerekli Human Gate kanıtları geçmiştir.
5. Tamamlanma kanıtı Jira yorumunda veya Jira'dan erişilen kalıcı artefaktta kayıtlıdır.
6. Graph doğrulamasında cycle, geri faz dependency, duplicate claim veya bypass bulunmamıştır.

Faz GATE'i geçtikten sonra faza yeni zorunlu iş eklenirse downstream iş seçilmeden önce GATE yeniden açılır veya yeni işin faz dışı/change-request niteliği yetkili insan kararıyla belgelenir. Tamamlanmış GATE kanıtı sessizce geçerli sayılmaya devam etmez.

## 6. Canlı Jira ile doğrulanmış faz DAG snapshot'ı

Aşağıdaki tablo 2026-10-01 tarihinde canlı Jira `Blocks` bağlantılarından doğrulanmıştır. Giriş önkoşulu sütunu faz issue'sundaki inward `is blocked by` GATE'lerini gösterir. Bu tablo değişiklik tespiti ve review kolaylığı sağlar; güncel karar için her zaman canlı Jira yeniden okunur.

| Faz | Faz issue | Giriş için tamamlanması gereken GATE'ler | Çıkış GATE |
|---|---|---|---|
| P00 | `TBP-1` | Başlangıç | `TBP-12` |
| P01 | `TBP-13` | `TBP-12` | `TBP-27` |
| P02 | `TBP-28` | `TBP-27` | `TBP-46` |
| P03 | `TBP-47` | `TBP-27` | `TBP-56` |
| P04 | `TBP-57` | `TBP-27` | `TBP-72` |
| P05 | `TBP-73` | `TBP-72` | `TBP-83` |
| P06 | `TBP-84` | `TBP-83` | `TBP-93` |
| P07 | `TBP-94` | `TBP-93` | `TBP-104` |
| P08 | `TBP-105` | `TBP-93` | `TBP-116` |
| P09 | `TBP-117` | `TBP-56`, `TBP-104`, `TBP-116` | `TBP-128` |
| P10 | `TBP-129` | `TBP-128` | `TBP-140` |
| P11 | `TBP-141` | `TBP-128` | `TBP-151` |
| P12 | `TBP-152` | `TBP-140`, `TBP-151` | `TBP-164` |
| P13 | `TBP-165` | `TBP-72`, `TBP-83` | `TBP-177` |
| P14 | `TBP-178` | `TBP-128`, `TBP-140`, `TBP-151`, `TBP-164` | `TBP-189` |
| P15 | `TBP-190` | `TBP-177`, `TBP-189` | `TBP-200` |
| P16 | `TBP-201` | `TBP-72`, `TBP-83`, `TBP-164`, `TBP-177`, `TBP-189` | `TBP-213` |
| P17 | `TBP-214` | `TBP-200`, `TBP-213` | `TBP-226` |

P02-GATE `TBP-46` ve nihai P17-GATE `TBP-226` için bu snapshot'ta outward faz bağlantısı yoktur. Bu durum yeni bir kenar varsayma yetkisi vermez; canlı Jira graph'ı değiştirilmeden bağımlılık üretilemez.

## 7. Claim, lock ve yarış güvenliği

Uygunluk kontrolü claim garantisi değildir; iki worker aynı issue'yu eşzamanlı görebilir. Claim mekanizması şu minimum sözleşmeyi korur:

- Claim tek bir kanonik worker kimliğiyle atomik veya optimistic-concurrency korumalı yazılır.
- Aynı issue için aynı anda en fazla bir aktif claim bulunur.
- Claim kaydı worker, zaman, lease/heartbeat ve correlation bilgisi taşır.
- Stale claim yalnız tanımlı timeout ve recovery prosedürüyle serbest bırakılır; aktif heartbeat varken steal edilemez.
- Claim başarısızsa worker implementasyona başlamaz ve sonraki uygun adayı canlı Jira'dan yeniden hesaplar.
- Duplicate veya obsolete kararı claim sonrasında ortaya çıkarsa çalışma durdurulur, üretilen kanıt korunur ve kanonik issue'ya otomatik aktarım yapılmaz.

Claim/lock/heartbeat'in teknik implementasyonu `TBP-31` kapsamındadır. Bu belge davranış sözleşmesini tanımlar, altyapı implementasyonu üretmez.

## 8. Graph değişiklik yönetişimi

Yeni issue, dependency veya faz geçişi eklenirken:

1. Değişiklik canlı Jira'da yapılır; yerel liste önce değiştirilmez.
2. İki uç issue ve faz etiketleri doğrulanır.
3. Yön, cycle ve geri faz kontrolleri çalıştırılır.
4. GATE membership değişiyorsa ilgili faz GATE'inin inbound bağlantıları ve kanıt kapsamı güncellenir.
5. Downstream uygunluk yeniden hesaplanır.
6. Değişiklik gerekçesi Jira yorumunda kaydedilir.

Silinmek istenen dependency için de aynı etki analizi gerekir. Bir kenarı kaldırmak GATE bypass veya eksik önkoşul oluşturuyorsa değişiklik reddedilir. Graph'ın geçmişi Jira changelog ve yorumlarında korunur.

## 9. Doğrulama sonuçları

| Sonuç | Koşul |
|---|---|
| `READY_FOR_CLAIM` | Bütün dependency, GATE, durum, duplicate/obsolete ve claim kontrolleri geçti. |
| `BLOCKED` | Tamamlanmamış önkoşul, GATE, cycle, geri faz dependency veya aktif claim engeli var. |
| `SPEC_REQUIRED` | Faz, önkoşul, yön veya kanonik issue belirsiz. |
| `REQUEST_CHANGES` | Graph bilinen politikaya aykırı yapılandırılmış. |
| `HUMAN_REQUIRED` | Kapsam çakışması, kanonik issue seçimi veya geri dönüşü zor graph değişikliği için yetkili karar gerekiyor. |

## 10. Review kontrol listesi

- Canlı Jira bu çalışma turunda yeniden okundu mu?
- Seçilen issue Rank sırasındaki ilk uygun iş mi?
- Bütün inward `Blocks` önkoşulları `Done` mı?
- `Blocks` yönü önkoşuldan bağımlı işe mi?
- Self-loop, cycle veya geri faz dependency var mı?
- Gerekli faz giriş GATE'lerinin tamamı `Done` mı?
- Fazın tek çıkış GATE'i ve bütün zorunlu faz işi bağlantıları mevcut mu?
- Duplicate, obsolete, superseded veya iptal iş claim edilmeye çalışılıyor mu?
- Başka bir aktif claim/lock/heartbeat var mı?
- Rank, dependency veya GATE'i bypass etmek için kullanılmış mı?
- Graph değişikliği Jira yorumu ve changelog ile izlenebilir mi?

## 11. Kapsam sınırı

Bu belge dependency ve faz GATE politikasını tanımlar. Story hazır olma koşulları `TupBayiProje_Jira_Story_Sablonu_ve_Definition_of_Ready_Standardi.md`; risk bazlı tamamlanma kanıtı `TupBayiProje_Definition_of_Done_ve_Risk_Bazli_Kalite_Kapilari_Standardi.md`; test sonucu hesaplama standardı `TupBayiProje_Test_Oracle_ve_Golden_Scenario_Standardi.md`; factory üzerinde otomatik DAG/claim/lock/heartbeat implementasyonu `TBP-31` kapsamındadır.
