# ADR-015: Tenant Bağlantı Yaşam Döngüsü

## Durum

Kabul Edildi

## Tarih

2026-10-01

## Jira Kaynağı

- `TBP-7`
- Normatif kaynaklar: `INV-TEN-003`, `INV-TEN-004`, `SOT-TEN-001`
- Ayrıntı işleri: `TBP-62`, `TBP-63`

## Bağlam

Database-per-Tenant modelinde sınırsız veya client seçimine bağlı connection yönetimi bellek, secret ve izolasyon riski doğurur.

## Karar

Tenant database metadata Tenancy'nin sahip olduğu Master DB kaydından server-side çözülür. Connection/data source yalnız doğrulanmış tenant context sonrasında alınır ve bounded bir yaşam döngüsüyle yönetilir. Request kodu connection string seçemez.

## Değiştirilemez Sınırlar

- Secret kaynak koda, loga veya client'a çıkamaz.
- Yanlış tenant connection'ı fail-closed reddedilir.
- Cache boyutu, eviction ve disposal davranışı ölçülebilir olmalıdır.

## Değerlendirilen Alternatifler

- Her istekte sınırsız yeni pool: kaynak tüketimini kontrolsüz büyütür.
- Sınırsız kalıcı tenant cache'i: tenant sayısıyla birlikte bellek ve connection riskini büyütür.

## Sonuçlar

Kesin cache parametreleri `TBP-63` ve 100+ tenant PoC ölçümüyle belirlenir.

## Doğrulama

İzolasyon, eviction/disposal, secret sızıntısı ve yüksek tenant sayısı test edilir.
