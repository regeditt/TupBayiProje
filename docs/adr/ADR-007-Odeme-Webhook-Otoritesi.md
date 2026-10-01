# ADR-007: Ödeme Webhook Otoritesi

## Durum

Kabul Edildi

## Tarih

2026-10-01

## Jira Kaynağı

- `TBP-7`
- Normatif kaynaklar: `INV-PAY-001`–`INV-PAY-003`, `SOT-BIL-001`

## Bağlam

Client callback veya redirect kolayca taklit edilebilir ve ödeme sağlayıcısındaki nihai sonucu kanıtlamaz.

## Karar

Ödeme sonucu için authoritative kanıt, provider standardına göre imzası ve replay penceresi doğrulanmış webhook ile Master DB payment state'in birlikte değerlendirilmesidir. Billing ödeme gerçeğinin sahibidir; Licensing yalnız doğrulanmış payment state event'iyle ilerler.

## Değiştirilemez Sınırlar

- Client bildirimi ödeme kanıtı değildir.
- Duplicate, replay ve out-of-order event ikinci veya geri alıcı business etkisi oluşturamaz.
- İmza doğrulaması başarısızsa fail-closed davranılır.

## Değerlendirilen Alternatifler

- Client callback: güvenilir otorite değildir.
- Licensing'in webhook'u doğrudan yorumlaması: domain ownership'i ihlal eder.

## Sonuçlar

Webhook inbox, provider event kimliği, idempotency ve gözlemlenebilir sıra yönetimi gerekir.

## Doğrulama

Geçersiz imza, replay, duplicate ve out-of-order webhook testleri zorunludur.
