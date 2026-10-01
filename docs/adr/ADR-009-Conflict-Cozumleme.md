# ADR-009: Conflict Çözümleme

## Durum

Kabul Edildi

## Tarih

2026-10-01

## Jira Kaynağı

- `TBP-7`
- Normatif kaynaklar: `INV-OFF-001`, `INV-OFF-004`, `SOT-SYN-001`

## Bağlam

Offline veya eşzamanlı değişiklikler özellikle finans ve fiziksel stokta sessiz veri kaybına yol açabilir.

## Karar

Conflict, güncel server state ve domain politikasıyla açıkça değerlendirilir. Finansal ve fiziksel stok etkilerinde silent last-write-wins yasaktır. Otomatik ve güvenli biçimde çözülemeyen kayıt `conflicted` durumuna alınır ve Conflict Review'a gider.

## Değiştirilemez Sınırlar

- Eski event güncel state'i geri alamaz.
- Mevcut state okunmadan otomatik merge yapılmaz.
- Tamamlanmış ledger kaydı conflict çözümü için değiştirilmez; gerekirse reversal/correction üretilir.

## Değerlendirilen Alternatifler

- Last-write-wins: geçmiş ve kullanıcı kararını sessizce kaybeder.
- Sınırsız otomatik retry: aynı geçersiz kararı tekrarlar.

## Sonuçlar

Domain bazlı conflict politikası, kullanıcıya görünür durum ve audit izi gerekir.

## Doğrulama

Stale version, out-of-order event ve eşzamanlı finans/stok conflict senaryoları test edilir.
