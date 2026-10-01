# ADR-006: Finansal Reversal

## Durum

Kabul Edildi

## Tarih

2026-10-01

## Jira Kaynağı

- `TBP-7`
- Normatif kaynaklar: `INV-FIN-001`–`INV-FIN-004`, `SOT-ACC-001`, `SOT-ACC-002`

## Bağlam

Tamamlanmış finansal kaydı silmek veya geçmiş etkisini yerinde değiştirmek denetim izini ve mutabakatı bozar.

## Karar

Tamamlanmış satış, tahsilat, ödeme, gider ve cari ledger kayıtları değiştirilemez. Hata, özgün kayda bağlı yeni reversal veya correction kaydıyla giderilir.

## Değiştirilemez Sınırlar

- Hard-delete ve geçmişi gizleyen update yasaktır.
- Finans, stok ve cari etkileri tanımlı transaction sınırında atomiktir.
- Her logical operation idempotent ve audit edilebilirdir.

## Değerlendirilen Alternatifler

- Kaydı silip yeniden oluşturmak: kanıt zincirini kırar.
- Bakiyeyi doğrudan düzeltmek: ledger'ı authoritative olmaktan çıkarır.

## Sonuçlar

Raporlar net etkiyi özgün ve düzeltici kayıtların toplamından üretir; kullanıcı düzeltme ilişkisini görebilir.

## Doğrulama

Double reversal, duplicate request, kısmi hata ve eşzamanlı correction senaryoları test edilir.
