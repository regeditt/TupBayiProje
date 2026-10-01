# ADR-004: Tüp Hareket Defteri

## Durum

Kabul Edildi

## Tarih

2026-10-01

## Jira Kaynağı

- `TBP-7`
- Normatif kaynaklar: `INV-STK-001`–`INV-STK-005`, `SOT-INV-001`–`SOT-INV-004`

## Bağlam

Yalnız bakiye tutmak fiziksel hareket geçmişini, lokasyonu ve düzeltme nedenini kanıtlayamaz.

## Karar

Fiziksel tüp stok değişikliklerinin authoritative kaynağı Inventory domainindeki movement ledger'dır. FULL ve EMPTY ayrı durumlar, depo, araç ve müşteri ayrı fiziksel lokasyonlar olarak modellenir.

## Değiştirilemez Sınırlar

- Bakiye doğrudan artırılamaz veya azaltılamaz.
- Vehicle Stock bağımsız lokasyondur.
- Tamamlanmış movement silinmez veya yerinde değiştirilmez.
- Duplicate ve oversell store katmanında engellenir.

## Değerlendirilen Alternatifler

- Mutable stok bakiyesi: geçmişi ve idempotency kanıtını kaybeder.
- FULL ile EMPTY'yi tek sayı yapmak: stok invariantını ihlal eder.

## Sonuçlar

Her fiziksel etki izlenebilir movement üretir; düzeltmeler bağlı reversal/correction hareketidir.

## Doğrulama

Paralel oversell, duplicate movement, lokasyon transferi ve reversal senaryoları test edilir.
