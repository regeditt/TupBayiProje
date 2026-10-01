# ADR-005: Stok Projection

## Durum

Kabul Edildi

## Tarih

2026-10-01

## Jira Kaynağı

- `TBP-7`
- Normatif kaynaklar: `SOT-INV-003`, `INV-STK-001`

## Bağlam

Operasyonel sorguların her seferinde tüm movement geçmişini taraması verimsizdir; ancak hız için ikinci bir gerçek kaynak oluşturulamaz.

## Karar

Stok bakiyesi movement ledger'dan üretilen, yeniden oluşturulabilir bir projection'dır. Projection, movement ile aynı transaction içinde güncellenir veya ledger'dan deterministik olarak yeniden üretilir.

## Değiştirilemez Sınırlar

- Projection authoritative değildir ve movement olmadan değiştirilemez.
- FULL, EMPTY ve fiziksel lokasyon ayrımları korunur.
- Projection hatası ledger geçmişini değiştiremez.

## Değerlendirilen Alternatifler

- Projection'ı tek gerçek kaynak yapmak: ledger otoritesini ihlal eder.
- Her sorguda tam ledger taramak: operasyonel sorgu maliyetini gereksiz artırır.

## Sonuçlar

Rebuild ve reconciliation yeteneği gerekir. Projection drift'i gözlemlenir ve ledger esas alınarak düzeltilir.

## Doğrulama

Aynı ledger girdisinin aynı projection'ı ürettiği, rebuild ve transaction rollback senaryolarıyla kanıtlanır.
