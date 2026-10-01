# ADR-008: Offline Outbox

## Durum

Kabul Edildi

## Tarih

2026-10-01

## Jira Kaynağı

- `TBP-7`
- Normatif kaynaklar: `INV-OFF-002`–`INV-OFF-004`, `SOT-SYN-001`

## Bağlam

Offline komutların ağ bağlantısına bağlı anlık gönderimi veri kaybı, belirsiz sonuç ve duplicate etki üretir.

## Karar

Offline komutlar istemci local store'da durable pending/outbox kaydı olarak tutulur. Her komut benzersiz kimlik, cihaz sırası ve idempotency bilgisi taşır; server güncel identity, tenant, authorization ve domain kurallarıyla yeniden doğrular.

## Değiştirilemez Sınırlar

- Client tenant ve yetki beyanı authoritative değildir.
- Pending, stale, rejected ve conflicted durumları kullanıcıdan gizlenmez.
- Başarısız sync başarılı gösterilemez.

## Değerlendirilen Alternatifler

- Fire-and-forget gönderim: dayanıklılık ve kanıt sağlamaz.
- Server doğrulamasını atlamak: offline veriyi güvenilir kabul eder.

## Sonuçlar

Reconnect reconciliation, bounded retry ve açık hata durumları gerekir.

## Doğrulama

Uygulama kapanması, ağ kesintisi, duplicate gönderim, sıra boşluğu ve revoke edilmiş yetki senaryoları test edilir.
