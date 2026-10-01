# Katkı Kuralları

Bu belge TupBayiProje repository'sinin temel branch, PR, secret ve hijyen sözleşmesini tanımlar. GitHub branch protection ve merge otomasyonunun teknik uygulanması `TBP-43` kapsamındadır.

## 1. İş Kaynağı ve Kapsam

- Her işe başlamadan ve başka bir işe geçmeden önce [`AGENTS.md`](AGENTS.md) içindeki canlı Jira döngüsü uygulanır.
- Branch ve PR tek bir Jira işiyle ilişkilendirilir. İlgisiz düzenlemeler aynı değişikliğe eklenmez.
- Canlı Jira açıklaması, kabul kriterleri, bağlantıları ve yorumları yerel listelerden üstündür.
- Blokeli, eski, yinelenen veya faz GATE'ini atlayan iş başlatılmaz.

## 2. Branch Kuralları

`main` her zaman gözden geçirilmiş ve doğrulanmış değişiklikleri temsil eder. Normal geliştirme doğrudan `main` üzerinde yapılmaz; Jira işi için kısa ömürlü branch açılır ve PR ile birleştirilir.

Branch adı küçük harfli ASCII karakterlerden oluşur:

```text
feature/tbp-123-kisa-aciklama
fix/tbp-123-kisa-aciklama
chore/tbp-123-kisa-aciklama
docs/tbp-123-kisa-aciklama
```

- `feature/`: Kullanıcı veya sistem davranışı ekler.
- `fix/`: Hata düzeltir.
- `chore/`: Repository, araç veya bağımlılık bakımı yapar.
- `docs/`: Yalnız dokümantasyon değiştirir.
- Branch bir Jira işinden fazlasını taşımaz ve gereksiz yere uzun ömürlü tutulmaz.

## 3. Commit Kuralları

Commit'ler tek bir mantıksal değişiklik içerir ve Conventional Commits biçimini kullanır:

```text
<type>(<scope>): TBP-123 kısa amaç
```

Geçerli temel tipler `feat`, `fix`, `test`, `docs`, `refactor` ve `chore` değerleridir. Commit mesajı yapılan dosya hareketini değil değişikliğin amacını anlatır. Formatlama, refactor ve davranış değişikliği mümkün olduğunda ayrı commit'lerde tutulur.

## 4. Pull Request Kuralları

- PR başlığı `[TBP-123] Kısa amaç` biçimindedir.
- PR açıklaması `.github/PULL_REQUEST_TEMPLATE.md` şablonunu eksiksiz doldurur.
- Değişiklik kapsamı, risk seviyesi, doğrulama komutları ve sonuçları görünürdür.
- Secret veya hassas veri diff, log, ekran görüntüsü, test çıktısı ya da Jira kanıtına eklenmez.
- İlgili testler ve risk seviyesinin zorunlu kalite kapıları geçmeden merge yapılmaz.
- Açık kritik bulgu, çözülmemiş blocker veya gerekli Human Gate onayı eksikken merge yapılmaz.
- GitHub üzerindeki zorunlu review sayısı, branch protection ve merge yöntemi `TBP-43` ile teknik olarak uygulanacaktır.

## 5. Secret Yönetimi

- Parola, token, API key, private key, sertifika, connection string veya üretim credential'ı Git'e yazılmaz.
- Secret değerleri environment variable veya onaylı secret store üzerinden alınır.
- Örnek yapılandırma gerekiyorsa yalnız güvenli placeholder içeren `.env.example` kullanılır; gerçek veya gerçeğe benzeyen değer eklenmez.
- Local secret dosyaları ve private key biçimleri kök `.gitignore` tarafından dışlanır. Ignore kuralı güvenlik kontrolünün yerine geçmez; commit öncesi staged diff ayrıca incelenir.
- Secret yanlışlıkla açığa çıkarsa önce credential iptal edilir veya döndürülür, sonra güvenlik sorumlularına bildirilir ve geçmiş temizleme planı uygulanır. Yalnız dosyayı sonraki commit'te silmek yeterli değildir.

## 6. Repository Hijyeni

- Build çıktıları, dependency cache'leri, IDE ayarları, loglar ve yerel geçici dosyalar commit edilmez.
- Üretilen dosya ancak ilgili Jira işi açıkça sürüm kontrolünde tutulmasını gerektiriyorsa commit edilir.
- Metin dosyaları `.gitattributes` ve `.editorconfig` kurallarına uyar; Windows script'leri dışında kanonik satır sonu LF'dir.
- Yeni kök dizin yalnız açık sahiplik ve Jira kapsamıyla eklenir. Boş dizin yerine sınırı açıklayan README kullanılır.
- Dependency manifest ve lock dosyaları ilgili workspace kurulduğunda birlikte güncellenir.
- Mevcut normatif belgeler veya ADR'ler sessizce taşınmaz, yeniden adlandırılmaz ya da silinmez.

## 7. Tamamlanma

Değişiklik, canlı Jira kabul kriterleri ile [`TupBayiProje_Definition_of_Done_ve_Risk_Bazli_Kalite_Kapilari_Standardi.md`](TupBayiProje_Definition_of_Done_ve_Risk_Bazli_Kalite_Kapilari_Standardi.md) birlikte sağlandığında tamamlanmış sayılır. Kanıtlar tekrarlanabilir komutları ve sonuçlarını içermelidir.
