# ADR-014: Event Contract Sürümleme

## Durum

Önerildi

## Tarih

2026-10-01

## Jira Kaynağı

- `TBP-7`
- Ayrıntı işi: `TBP-26`

## Bağlam

Üretici ve tüketicilerin farklı hızlarda değişmesi, event schema değişikliklerini uyumluluk riski haline getirir.

## Karar

Yayımlanan event contract'ları açık kimlik ve sürüm taşır; schema değişiklikleri kayıtlı compatibility kuralına tabidir. Tüketici, bilinmeyen veya uyumsuz sürümü sessizce işleyemez. Kesin sürüm biçimi ve compatibility matrisi `TBP-26` ile belirlenecektir.

## Değiştirilemez Sınırlar

- Duplicate ve out-of-order event ikinci veya geri alıcı etki oluşturamaz.
- Event başka domainin authoritative state'ini sahiplenemez.
- Correlation, causation ve message kimliği korunur.

## Değerlendirilen Alternatifler

- Sürümsüz event: değişiklik etkisini görünmez yapar.
- Her değişiklikte yeni topic: yaşam döngüsü ve operasyon maliyetini kontrolsüz büyütür.

## Sonuçlar

Registry, compatibility kontrolü ve tüketici hata politikası gerekir; ayrıntılar henüz sabitlenmemiştir.

## Doğrulama

`TBP-26` kabulü ve eski/yeni üretici-tüketici compatibility testleri gerekir.
