# ADR-011: Figma Tasarım Kaynağı

## Durum

Önerildi

## Tarih

2026-10-01

## Jira Kaynağı

- `TBP-7`
- Yönetişim işi: `TBP-40`
- Ayrıntı işleri: `TBP-48`–`TBP-56`

## Bağlam

UI implementasyonu ile onaylı tasarım arasında birincil referans belirsiz olursa ekranlar ve kritik durumlar sessizce ayrışır.

## Karar

Onaylı UI görünümü, akışı ve component davranışı için Figma kaynağı esas alınacaktır. Jira business davranışının kaynağı olmaya devam eder; Figma Jira gereksinimini değiştiremez veya yeni business rule üretemez.

## Değiştirilemez Sınırlar

- Yalnız onaylı ve kilitli tasarım düğümleri implementasyon kaynağı olabilir.
- Empty, loading, error, offline, stale ve conflict durumları ilgili Jira kapsamıyla izlenebilir olmalıdır.
- Figma ile Jira çelişkisi çözülmeden implementasyon yapılmaz.

## Değerlendirilen Alternatifler

- Çalışan UI'ı tasarım kaynağı saymak: design drift'i kalıcılaştırır.
- Figma'yı business rule kaynağı yapmak: Jira Source of Truth kuralını ihlal eder.

## Sonuçlar

Figma sayfa yönetişimi, frame-lock ve approval ayrıntıları ilgili Jira işleriyle tanımlanacaktır.

## Doğrulama

`TBP-48`–`TBP-56` kararları ve P03 GATE olmadan kabul edilmez.
