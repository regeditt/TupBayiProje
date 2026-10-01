# ADR-010: Cihaz Offline Lisansı

## Durum

Önerildi

## Tarih

2026-10-01

## Jira Kaynağı

- `TBP-7`
- Ayrıntı işi: `TBP-172`
- Ownership kaynakları: `SOT-LIC-001`, `SOT-DEV-001`

## Bağlam

Cihazın bağlantısız çalışırken lisans kararını doğrulayabilmesi gerekir; client tarafından üretilebilen bir bayrak lisans kanıtı olamaz.

## Karar

Offline lisans kanıtı Licensing tarafından üretilen, cihaza bağlı ve doğrulanabilir imzalı bir artefact olacaktır. Doğrulama fail-closed çalışacak; revoke, süre ve clock-tamper politikalarının ayrıntısı `TBP-172` ile belirlenecektir.

## Değiştirilemez Sınırlar

- Billing ödeme gerçeğinin, Licensing lisans state'inin sahibidir.
- Client lisans veya payment sonucu authoritative değildir.
- Signing key client'a gömülemez veya loglanamaz.

## Değerlendirilen Alternatifler

- Yerel boolean lisans bayrağı: taklit edilebilir.
- Süresiz offline token: revoke ve süre kontrolünü etkisizleştirir.

## Sonuçlar

İmza algoritması, anahtar rotasyonu, geçerlilik süresi ve clock-tamper toleransı Human Gate gerektiren açık kararlardır.

## Doğrulama

`TBP-172` kabul kriterleri ve negatif imza/revoke/zaman testleri tamamlanmadan kabul edilmez.
