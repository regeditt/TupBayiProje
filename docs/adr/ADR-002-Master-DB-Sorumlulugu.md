# ADR-002: Master DB Sorumluluğu

## Durum

Kabul Edildi

## Tarih

2026-10-01

## Jira Kaynağı

- `TBP-7`
- Normatif kaynaklar: `INV-TEN-002`, `SOT-IDN-001`–`SOT-PRV-001`

## Bağlam

Control-plane ile tenant operasyon verisinin aynı kaynakta tutulması ownership ve izolasyon sınırını belirsizleştirir.

## Karar

Master DB yalnız Identity, Tenancy, Licensing, Billing ve provisioning/migration job gibi control-plane kayıtlarını tutar. Müşteri, ürün, satış, sipariş, stok, cari ve teslimat verisi tenant DB'de kalır.

## Değiştirilemez Sınırlar

- Her veri kümesinin tek sahibi ve tek authoritative kaynağı vardır.
- Master DB tenant iş verisinin rapor veya kolaylık kopyasını tutamaz.
- Domainler birbirinin tablolarına doğrudan yazamaz.

## Değerlendirilen Alternatifler

- Tüm veriyi Master DB'de toplamak: tenant izolasyonunu ve ownership'i ihlal eder.
- Control-plane verisini tenant DB'lere çoğaltmak: ikinci authoritative kaynak oluşturur.

## Sonuçlar

Cross-database işlerde distributed transaction varsayılmaz; outbox/inbox ve idempotency gerekir.

## Doğrulama

Şema ownership incelemesi ve tenant iş verisinin Master DB'de bulunmadığını kanıtlayan architecture testleri gerekir.
