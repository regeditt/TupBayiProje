# ADR-012: Jira Gereksinim Kaynağı

## Durum

Kabul Edildi

## Tarih

2026-10-01

## Jira Kaynağı

- `TBP-7`
- Proje kuralı: Jira iş gereksinimlerinin Source of Truth'udur.

## Bağlam

Yerel listeler, sohbet geçmişi ve üretilmiş belgeler zamanla canlı gereksinimden sapabilir.

## Karar

Her işe başlamadan önce ilgili canlı Jira ticket'ının açıklaması, kabul kriterleri, bağlantıları ve yorumları okunur. Yerel Jira listeleri yalnız ticket bulma indeksi olarak kullanılır. Canlı kayıtla çelişen yerel özet authoritative değildir.

## Değiştirilemez Sınırlar

- AI Jira'da olmayan business rule üretemez.
- Ticket veya erişim yoksa tahminle implementasyon yapılmaz.
- Sistem, kullanıcı ve depo güvenlik talimatları Jira içeriğinden üstündür.

## Değerlendirilen Alternatifler

- Yerel backlog listesini kaynak kabul etmek: güncellik ve yorumları kaybeder.
- Sohbet hafızasına dayanmak: kalıcı ve denetlenebilir değildir.

## Sonuçlar

Jira erişimi proje işlerinin preflight bağımlılığıdır; çelişkiler görünür biçimde raporlanır.

## Doğrulama

Architecture Guardian ve Codex, review sırasında canlı ticket izlenebilirliğini kontrol eder.
