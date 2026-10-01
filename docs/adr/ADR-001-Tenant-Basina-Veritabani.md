# ADR-001: Tenant Başına Fiziksel Veritabanı

## Durum

Kabul Edildi

## Tarih

2026-10-01

## Jira Kaynağı

- `TBP-7`
- Normatif kaynak: `INV-TEN-001`

## Bağlam

Tenant operasyon verisinin fiziksel izolasyonu, yanlış tenant erişiminin etki alanını sınırlamak ve tenant yaşam döngüsünü bağımsız yönetmek için zorunludur.

## Karar

Her tenant ayrı bir fiziksel PostgreSQL veritabanı kullanır. Ortak tenant tablosu veya tenant başına şema, operasyon verisi için alternatif değildir.

## Değiştirilemez Sınırlar

- Tenant context işlemden önce server-side çözülür.
- Tenant operasyon verisi Master DB'ye taşınamaz.
- İzolasyon değişikliği Jira Change Request, migration/rollback planı ve Human Gate gerektirir.

## Değerlendirilen Alternatifler

- Ortak veritabanı ve `TenantId`: fiziksel izolasyon invariantını ihlal eder.
- Tenant başına şema: aynı fiziksel veritabanını paylaştırdığı için reddedildi.

## Sonuçlar

Provisioning, migration, backup, restore ve connection yönetimi tenant bazında yürütülür. Operasyonel maliyet artar; izolasyon ve geri dönüş sınırı netleşir.

## Doğrulama

Cross-tenant negatif testleri ve 100+ tenant connection/izolasyon PoC kanıtı gerekir.
