# ADR-013: OpenAPI Contract Kaynağı

## Durum

Önerildi

## Tarih

2026-10-01

## Jira Kaynağı

- `TBP-7`
- Ayrıntı işleri: `TBP-21`, `TBP-22`

## Bağlam

API implementasyonu, client ve mock birbirinden bağımsız tanımlanırsa contract drift oluşur.

## Karar

HTTP API yüzeyi için tek, version-controlled OpenAPI belgesi contract kaynağı olacaktır. Server, client ve mock aynı contract'a karşı doğrulanacaktır. OpenAPI business gereksiniminin değil, Jira ile onaylanmış API temsilinin kaynağıdır.

## Değiştirilemez Sınırlar

- Tenant, authorization ve domain invariantları yalnız schema doğrulamasına bırakılamaz.
- Breaking change, compatibility ve migration kararı olmadan yayımlanamaz.
- Contract üretim yönü ve araç seçimi `TBP-21` ile belirlenir.

## Değerlendirilen Alternatifler

- Ayrı el yazımı client/server modelleri: drift üretir.
- Çalışan server'ı tek başına contract saymak: mock ve review kanıtını zayıflatır.

## Sonuçlar

Contract lint, diff ve mock doğrulama kapıları gerekecektir; araç ayrıntısı henüz sabitlenmemiştir.

## Doğrulama

`TBP-21` ve `TBP-22` kabul kriterleri tamamlanmadan kabul edilmez.
