# ADR-003: Tenant Çözümleme

## Durum

Kabul Edildi

## Tarih

2026-10-01

## Jira Kaynağı

- `TBP-7`
- Normatif kaynaklar: `INV-TEN-003`, `INV-TEN-004`, `SOT-TEN-001`

## Bağlam

Client tarafından taşınan tenant beyanı kimlik taklidi ve cross-tenant erişim riski doğurur.

## Karar

Tenant context doğrulanmış identity/session ile Tenancy kayıtlarından server-side çözülür. Header, query string veya body içindeki `TenantId` authoritative değildir. Çözümleme başarısızsa işlem fail-closed reddedilir.

## Değiştirilemez Sınırlar

- Veritabanı connection ve transaction tenant context çözülmeden açılamaz.
- Tenant lifecycle, license ve resource scope backend'de doğrulanır.
- Hatalar başka tenant verisinin varlığını sızdıramaz.

## Değerlendirilen Alternatifler

- Client `TenantId` değerine güvenmek: güven sınırını ihlal eder.
- Sonradan tenant filtresi uygulamak: yanlış connection açılmasını önlemez.

## Sonuçlar

Kimlik ve tenancy entegrasyonu bütün command ve hassas query'lerin ön koşuludur.

## Doğrulama

Sahte tenant beyanı, revoked session ve cross-tenant record erişimi negatif test edilir.
