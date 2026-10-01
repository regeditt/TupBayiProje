# TupBayiProje Kodlama Dili ve Adlandırma Standardı

**Jira:** `TBP-6`  
**Durum:** Normatif  
**Kapsam:** Kaynak kod, testler, veritabanı nesneleri, dış contract'lar, yapılandırma, script'ler ve teknik dokümantasyon  
**Bağlı belgeler:** `TupBayiProje_Global_Invariantlar.md`, `TupBayiProje_Source_of_Truth_ve_Domain_Ownership.md`, `TupBayiProje_State_Machine_ve_Yasam_Dongusu_Standardi.md`, `TupBayiProje_Transaction_Concurrency_Idempotency_ve_Guvenlik_Standardi.md`

Bu belge, aynı kavramın backend, Flutter, Admin Web, veritabanı ve contract katmanlarında farklı adlarla temsil edilmesini önler. Amaç bütün dilleri tek bir sözdizimine zorlamak değil; ortak domain sözlüğünü korurken her ekosistemin yerleşik biçim kurallarını uygulamaktır.

## 1. Öncelik ve normatif dil

Çelişki halinde öncelik sırası şöyledir:

1. İnsan tarafından onaylanmış Jira gereksinimi ve global invariant,
2. Source of Truth ve domain ownership kaydı,
3. İlgili contract veya yaşam döngüsü standardı,
4. Bu belge,
5. Dilin ve kullanılan aracın resmi stil kılavuzu.

Bu belgede:

- **zorunlu** uyulması gereken kuralı,
- **yasak** kabul edilmeyen uygulamayı,
- **önerilir** gerekçeli istisna olmadıkça uygulanacak tercihi ifade eder.

Çelişki, eksik domain terimi veya birden fazla makul ad bulunduğunda agent yeni terim icat edemez. İş `SPEC_REQUIRED` olarak durdurulur veya gerekli risk seviyesinde Human Gate'e taşınır.

## 2. İnsan dili ve karakter kümesi

| Yüzey | Dil ve karakter kuralı |
|---|---|
| Jira başlığı, açıklaması, kabul kriteri ve review sonucu | Türkçe; Türkçe karakterler kullanılır |
| Normatif belge, ADR ve kullanıcıya yönelik açıklama | Türkçe; Türkçe karakterler kullanılır |
| Kaynak kod identifier'ı | Türkçe anlamlı; yalnız ASCII karakterleri |
| Elle yazılmış domain dosyası ve klasörü | Türkçe anlamlı; yalnız ASCII karakterleri; ilgili ekosistemin casing kuralı uygulanır |
| Kod yorumu ve elle yazılmış test açıklaması | Türkçe; yalnız ASCII karakterleri |
| Yerleşik teknik terim | Resmi ve yaygın yazımı korunur |
| Kullanıcı arayüzü metni | Ürün diline ve localization kaynağına göre; identifier içine gömülmez |

Kaynak identifier'larında `ç`, `Ç`, `ğ`, `Ğ`, `ı`, `İ`, `ö`, `Ö`, `ş`, `Ş`, `ü`, `Ü` kullanılamaz. Örnek dönüşümler:

| Türkçe kavram | Kaynak kod biçimi |
|---|---|
| müşteri | `Musteri` / `musteri` |
| satış | `Satis` / `satis` |
| tüp hareketi | `TupHareketi` / `tupHareketi` |
| cari hareket | `CariHareket` / `cariHareket` |
| gün sonu | `GunSonu` / `gunSonu` |

Harf dönüşümü adın anlamını veya kelime sınırlarını değiştiremez. Aynı kavram için farklı transliterasyon kullanılamaz.

## 3. Ortak sözlük ve teknik terimler

Domain adı, `TupBayiProje_Source_of_Truth_ve_Domain_Ownership.md` içindeki sahiplikle aynı anlamı taşımalıdır. Bir kavram bütün elle yazılmış katmanlarda tek kök ad kullanır. Örneğin `Musteri`, başka bir katmanda aynı anlam için `Customer`, `CariKart` veya `Client` olarak yeniden adlandırılamaz.

Aşağıdaki terimler teknik ekosistemin yerleşik adlarıdır ve çevrilmez:

- `API`, `HTTP`, `OpenAPI`, `JSON`, `URL`, `UTC`,
- `DbContext`, `CancellationToken`, `PostgreSQL`, `Npgsql`, `Redis`, `SignalR`,
- `Docker`, `Flutter`, `Dart`, `.NET`, `TypeScript`,
- `Tenant`, `Identity`, `Authorization`, `Authentication`,
- `Command`, `Query`, `Event`, `Handler`, `Middleware`, `Repository`,
- `Outbox`, `Inbox`, `Webhook`, `Idempotency`, `Concurrency`, `Migration`.

Bu liste domain terimlerini İngilizceleştirme izni vermez. Teknik terim ile domain terimi birleşebilir: `MusteriQuery`, `SatisCommandHandler`, `TenantContext`, `TupHareketiRepository`.

Yeni bir eş anlamlı veya kısaltma eklemek yerine mevcut sözlük kullanılır. Yaygın olmayan kısaltmalar yasaktır. `Id`, `Api`, `Http`, `Url`, `Utc` gibi kısaltmalar identifier içinde ilgili dilin resmi casing kuralına göre yazılır; protokol veya belge metninde resmi büyük harfli biçim korunur.

## 4. Diller arası ortak adlandırma kuralları

1. Tür, fonksiyon ve alan adları yaptığı işi ve domain anlamını açıklar; `Data`, `Info`, `Manager`, `Helper`, `Util`, `Common`, `Misc`, `Temp` gibi belirsiz adlar gerekçesiz kullanılamaz.
2. Tür ve tekil nesne adları tekil; koleksiyonlar çoğul yazılır.
3. Komutlar niyeti belirten emir kipiyle adlandırılır: `SatisTamamla`, `TahsilatKaydet`, `GunSonuKapat`.
4. Sorgular döndürdüğü kavramı ve gerekli ayrımı belirtir: `AktifMusterileriGetir`, `SatisDetayiBul`.
5. Boolean adları olumlu ve soru biçiminde olur: `AktifMi`, `SilindiMi`, `YetkiliMi`. Çifte olumsuzluk yasaktır.
6. Kimlik alanı `<Kavram>Id`, zaman alanı anlamına göre `<Olay>ZamaniUtc` veya `<Olay>Tarihi`, sürüm alanı `Version` kökünü kullanır.
7. Ölçü birimi belirsiz olabilecek sayısal alanın adında birim bulunur: `TimeoutSeconds`, `BoyutBytes`. Para alanı tutarın ne olduğunu belirtir; belirsiz `Value` veya `Amount1` adları kullanılamaz.
8. Aynı ad iki farklı business anlamında kullanılamaz. Aynı business anlamı da gerekçesiz birden fazla adla temsil edilemez.
9. Katman adı domain adının yerine geçemez. `Service`, `Entity` veya `Model` tek başına anlamlı tür adı değildir.
10. Geçici implementasyon ayrıntısı public contract adına sızdırılamaz.

## 5. C# ve .NET kuralları

Dil ve framework'ün resmi biçimiyle uyumlu olarak:

| Öğe | Biçim | Örnek |
|---|---|---|
| Namespace, type, record, enum | `PascalCase` | `TupBayiProje.Satis`, `SatisTamamlamaSonucu` |
| Public/internal üye ve metot | `PascalCase` | `SatisiTamamlaAsync` |
| Parametre ve local değişken | `camelCase` | `musteriId` |
| Private instance alanı | `_camelCase` | `_satisRepository` |
| Interface | `I` + `PascalCase` | `ISatisRepository` |
| Generic type parametresi | `T` veya `T` + anlam | `TEntity` |
| Sabit | `PascalCase` | `VarsayilanSayfaBoyutu` |

- Gerçekten asynchronous çalışan metot `Async` son ekini taşır ve mümkün olduğunda `CancellationToken` kabul eder.
- `CancellationToken` son parametre olur ve `cancellationToken` diye adlandırılır.
- Event handler veya framework'ün zorunlu kıldığı imza dışında `Async` son eki atlanamaz.
- Dosya, birincil type ile aynı adı taşır. Bir dosyada ilgisiz birden fazla public type bulunamaz.
- Namespace fiziksel modül sahipliğini yansıtır; başka domain'e aitmiş gibi adlandırılarak ownership atlanamaz.
- Extension metodu genişlettiği type ve davranışı açıkça belirtir; genel amaçlı `Extensions` yığını oluşturulamaz.

Örnek:

```csharp
public sealed class SatisTamamlamaService
{
    public Task<SatisTamamlamaSonucu> SatisiTamamlaAsync(
        SatisId satisId,
        CancellationToken cancellationToken);
}
```

## 6. Dart ve Flutter kuralları

| Öğe | Biçim | Örnek |
|---|---|---|
| Class, enum, extension, typedef | `UpperCamelCase` | `SatisDetayi` |
| Değişken, parametre, fonksiyon, üye | `lowerCamelCase` | `satisiTamamla` |
| Private üye | `_lowerCamelCase` | `_satisRepository` |
| Dosya ve library adı | `lower_snake_case` | `satis_detayi.dart` |

- Dart'ın yerleşik stiline aykırı `Async` son eki eklenmez; dönüş type'ı ve API davranışı async niteliğini gösterir.
- Widget adı görsel görünümü değil, ürün içindeki sorumluluğunu açıklar. `RedButton`, `NewScreen`, `CommonWidget` gibi geçici adlar kullanılmaz.
- UI metni doğrudan widget içine dağıtılmaz; localization anahtarı anlamı belirtir ve dil metninden bağımsız kalır.
- Generated API client dosyaları elle değiştirilmez ve generator'ın adlandırma çıktısı istisna olarak kabul edilir.

## 7. TypeScript ve Admin Web kuralları

Admin Web teknoloji seçimi ilgili ADR ve Jira işiyle yapılır; bu belge framework seçmez. TypeScript kullanıldığında:

| Öğe | Biçim | Örnek |
|---|---|---|
| Type, interface, enum, component | `PascalCase` | `SatisDetayi` |
| Değişken, parametre, fonksiyon, üye | `camelCase` | `satisiTamamla` |
| Kaynak dosyası | `kebab-case` | `satis-detayi.ts` |
| Environment değişkeni | `UPPER_SNAKE_CASE` | `API_BASE_URL` |

- TypeScript interface adında yalnız type olduğunu göstermek için `I` öneki kullanılmaz.
- Component adı varsa ürün sorumluluğunu belirtir; görünüşe veya geçici konuma göre adlandırılmaz.
- Boolean adları `aktifMi`, `yetkiliMi` gibi ortak sözlüğün `camelCase` karşılığını kullanır.
- Generated client ve schema çıktıları elle değiştirilmez.

## 8. PostgreSQL adlandırması

- Schema, tablo, kolon, index ve constraint adları ASCII `lower_snake_case` olur.
- Identifier'lar PostgreSQL'de quote gerektirmeyecek küçük harfli biçimde oluşturulur.
- Tablo adı temsil ettiği kaydı tekil adlandırır: `musteri`, `satis`, `stok_hareketi`.
- Primary key kolonu `id`; foreign key kolonu `<hedef>_id` biçimindedir.
- Standart constraint ve index önekleri şunlardır:

| Nesne | Biçim | Örnek |
|---|---|---|
| Primary key | `pk_<tablo>` | `pk_satis` |
| Foreign key | `fk_<tablo>_<hedef>` | `fk_satis_musteri` |
| Unique constraint | `uq_<tablo>_<alanlar>` | `uq_musteri_telefon` |
| Check constraint | `ck_<tablo>_<kural>` | `ck_satis_toplam_negatif_degil` |
| Index | `ix_<tablo>_<alanlar>` | `ix_satis_musteri_id` |

- Veritabanı nesnesinin adı tenant sınırı, domain ownership veya ledger niteliğini gizleyemez.
- ORM adı ile fiziksel ad arasında mapping varsa iki ad aynı business kavramını temsil eder.
- Migration'ın araç tarafından üretilen teknik bölümü generator kuralını izleyebilir; elle verilen migration adı niyeti açıklar.

## 9. HTTP, OpenAPI, JSON ve event contract'ları

Bu bölüm yalnız ortak adlandırma tabanını belirler. Endpoint sürümleme, envelope, event type biçimi ve compatibility politikası `TBP-21` ile `TBP-25` kapsamındaki contract standartlarında kesinleştirilir.

- Elle tanımlanan JSON property adları `camelCase` olur: `musteriId`, `correlationId`, `currentState`.
- HTTP route segmentleri ASCII `kebab-case` olur ve domain sözlüğünü kullanır.
- Enum/state wire değerleri kararlı `UPPER_SNAKE_CASE` olur: `FULL`, `EMPTY`, `PAYMENT_PENDING`.
- Hata kodları ilgili standarttaki kararlı `UPPER_SNAKE_CASE` biçimini korur: `<DOMAIN>_STATE_INVALID_TRANSITION`.
- Correlation, causation, idempotency ve tenant alanları bütün contract'larda onaylanan tek yazımı kullanır.
- Yayınlanmış property, route, event type, enum veya hata kodu yalnız yeniden adlandırma amacıyla değiştirilemez. Değişiklik compatibility planı, ADR ve gerekli Human Gate olmadan yapılamaz.
- Generated client'taki ad sapması generator mapping/configuration katmanında çözülür; generated dosyada elle patch uygulanmaz.

## 10. Yapılandırma, secret ve telemetry adları

- Environment değişkenleri `UPPER_SNAKE_CASE`; hiyerarşik uygulama ayarları ilgili platformun standart ayırıcısıyla adlandırılır.
- Secret adı amacını ve kapsamını belirtir fakat secret değerini, müşteri verisini veya credential parçasını içermez.
- Log, metric ve trace alanları aynı kavram için aynı anahtarı kullanır; serbest metin içine gömülmüş kimlik yerine structured alan tercih edilir.
- Hassas veri güvenli görünen bir alan adı altında loglanamaz. Adlandırma, veri sınıflandırma ve maskeleme zorunluluğunu ortadan kaldırmaz.
- Zaman alanları UTC ise adında veya contract tanımında UTC niteliği açıkça belirtilir. Yerel zaman sessizce UTC gibi adlandırılamaz.

## 11. Yorum ve dokümantasyon kuralları

- Yorum kodun ne yaptığını tekrar etmez; kararın nedenini, invariantı veya beklenmeyen kısıtı açıklar.
- Elle yazılan kod yorumları Türkçe ve ASCII'dir.
- Public contract dokümantasyonu; önkoşul, sonuç, hata ve güvenlik davranışını açıklar.
- `TODO`, `FIXME` veya `HACK` tek başına bırakılamaz. Gerçek ertelenmiş iş Jira anahtarı ve güvenli mevcut davranışla ilişkilendirilir.
- Yorum içine alınmış ölü kod tutulamaz; geçmiş sürüm kontrolündedir.
- Domain kuralını yalnız yorumda tanımlamak yasaktır; kural Jira/specification ve sorumlu implementasyon katmanında bulunur.

## 12. Test adlandırması

- Test adı senaryo ve gözlenebilir sonucu anlatır; implementasyon ayrıntısını tekrar etmez.
- C# test adı `<Metot>_<Senaryo>_<BeklenenSonuc>` biçimini kullanır: `SatisiTamamla_YetersizStokta_StokHareketiOlusturmaz`.
- Dart ve TypeScript test açıklamaları Türkçe anlamlı ve ASCII olur: `yetersiz stokta satisi tamamlamaz`.
- `Test1`, `Works`, `HappyPath`, `ShouldWork`, `Temp` gibi kanıt üretmeyen adlar yasaktır.
- Concurrency, idempotency, authorization ve tenant izolasyonu testleri hangi yarışın, replay'in veya reddin kanıtlandığını adında belirtir.
- Test fixture ve builder adları production domain sözlüğünden farklı bir business kavramı üretemez.

## 13. Generated code ve dış sistem sınırı

1. Generated dosya elle değiştirilmez.
2. Dış sistemin değiştirilemeyen alan adı boundary'de aynen alınabilir; içeride canonical modele açık mapping yapılır.
3. Dış sistem adı domain modeline sızdırılmaz; adapter veya anti-corruption boundary içinde kalır.
4. Generator çıktısındaki ad sorunu template, schema veya generator configuration'da çözülür.
5. Generated ve elle yazılmış kod klasör, header veya build metadata ile ayırt edilebilir olmalıdır.

## 14. Otomatik uygulama ve istisna yönetimi

`TBP-15`, `TBP-16` ve `TBP-17` kapsamındaki iskeletler kendi dillerinin formatter, analyzer ve linter'ını etkinleştirmek zorundadır. `TBP-19` ve sonraki CI işleri en az format, lint/analyze, build ve test kontrollerini merge öncesi çalıştırır.

Araçla doğrulanabilen bir kural yalnız review dikkatine bırakılamaz. Araçların kapsamadığı domain sözlüğü ve ownership kuralları architecture review ile doğrulanır.

Bu standardın uygunluğu Architecture Guardian ve Codex tarafından bağımsız olarak denetlenir. İki denetleyici de Jira'daki `TBP-6` açıklamasını authoritative kabul eder; yalnız bu belgenin özetine dayanarak ticket kuralı daraltılamaz.

İstisna ancak aşağıdakilerin tümüyle kabul edilir:

1. Teknik veya compatibility gerekçesi,
2. Dar kapsam ve sahibi,
3. Jira kaydı,
4. Gerekliyse ADR ve kaldırma/migration planı,
5. İlgili test veya contract kanıtı.

Kişisel tercih istisna gerekçesi değildir.

## 15. Review kontrol listesi

- Adlar Jira ve canonical domain sözlüğündeki kavramlarla aynı mı?
- Domain terimleri Türkçe anlamlı ve ASCII mi?
- Teknik terimler resmi yazımıyla mı kullanılmış?
- Aynı kavram farklı katmanlarda farklı adlara bölünmüş mü?
- C#, Dart, TypeScript ve PostgreSQL casing kuralları uygulanmış mı?
- Boolean, koleksiyon, zaman, kimlik ve ölçü alanları anlamını açıkça taşıyor mu?
- Belirsiz kısaltma veya `Manager`, `Helper`, `Data`, `Common` gibi sorumluluğu gizleyen ad var mı?
- Public contract adı kararlı mı; rename compatibility planı gerektiriyor mu?
- Generated dosya elle değiştirilmiş mi?
- Yorum kararın nedenini mi açıklıyor ve Türkçe ASCII mi?
- Test adı senaryo ile sonucu kanıtlanabilir biçimde anlatıyor mu?
- Formatter, analyzer ve linter sonucu temiz mi?
- İstisna varsa Jira/ADR ve kaldırma planıyla kayıtlı mı?

Biçim veya sözlük ihlali `REQUEST_CHANGES`; domain ownership'i gizleyen ya da invariantı etkisizleştiren adlandırma `BLOCKED`; eksik business terimi `SPEC_REQUIRED`; yayınlanmış contract'ı kıracak ad değişikliği gerekli onay yoksa `HUMAN_REQUIRED` sonucu üretir.
