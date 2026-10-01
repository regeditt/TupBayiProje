# ADR-016: Tenant Migration Orkestrasyonu

## Durum

Kabul Edildi

## Tarih

2026-10-01

## Jira Kaynağı

- `TBP-7`
- Normatif kaynaklar: `SOT-TEN-001`, `SOT-PRV-001`
- Ayrıntı işleri: `TBP-66`–`TBP-68`, `TBP-208`

## Bağlam

Ayrı fiziksel veritabanları migration'ın tek deploy transaction'ı gibi yürütülmesini engeller. Kontrolsüz paralellik ve belirsiz retry geniş çaplı tenant etkisi yaratır.

## Karar

Tenant migration version ve job state Tenancy sahipliğinde Master DB'de tutulur. Migration'lar kuyruklanmış, idempotent, gözlemlenebilir ve tenant bazında sonuçlanan orchestration ile yürütülür. Batch, throttle, retry ve recovery parametreleri ilgili Jira işleriyle belirlenir.

## Değiştirilemez Sınırlar

- Destructive migration Human Gate ve rollback/recovery planı gerektirir.
- Bir tenant başarısızlığı diğer tenantların sonucu olarak gizlenemez.
- Belirsiz sonuçta kör tekrar yapılmaz; kayıtlı job/version state çözülür.

## Değerlendirilen Alternatifler

- Deploy sırasında tüm tenantlara eşzamanlı migration: blast radius ve recovery riskini büyütür.
- Elle ve kayıtsız migration: izlenebilirlik ve idempotency sağlamaz.

## Sonuçlar

Per-tenant durum, bounded paralellik, retry sınıflandırması, audit ve recovery kanıtı gerekir.

## Doğrulama

Kısmi batch hatası, retry exhaustion, restart recovery ve destructive rollback senaryoları test edilir.
