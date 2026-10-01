# TupBayiProje Agent Kurallari

## Zorunlu Canli Jira Calisma Dongusu

Her proje isine baslamadan, bir sonraki ise gecmeden ve herhangi bir dosyayi degistirmeden once:

1. Canli Atlassian Jira'ya baglan ve `TBP` projesine erisimi dogrula.
2. Kullanici belirli bir ticket vermediyse siradaki uygun ticket'i canli Jira'dan al. Jira sirasi/rank'i, durum, `Blocks` baglantilari, tamamlanmis on kosullar ve faz GATE'leri birlikte degerlendirilir; blokeli is secilmez.
3. Secilen ticket'in canli Jira kaydindaki aciklamayi, kabul kriterlerini, baglantilari ve tum yorumlari oku.
4. Gereksinimleri ilgili yerel belge, kod ve testlerle karsilastir; isi canli kayda gore sinirla.
5. Is tamamlandiginda siradaki ticket'i yerel listeden tahmin etme; donguye yeniden canli Jira'ya baglanarak basla.

Canli Jira hem is gereksinimlerinin hem de siradaki is seciminin Source of Truth'udur. `TupBayiProje_Jira_Projesi.md` ve diger yerel ticket listeleri yalnizca arama yardimcisi olabilir; is sirasi, uygunluk, durum veya gereksinim kaynagi olarak kullanilamaz ve canli kaydin yerine gecmez. Yerel icerik ile canli Jira celisirse sistem, kullanici ve depo talimatlari sakli kalmak uzere canli Jira esas alinir ve celiski kullaniciya bildirilir.

Canli Jira'ya baglanilamiyor, siradaki uygun ticket belirlenemiyor veya secilen ticket'in gerekli alani okunamiyorsa tahminle implementasyona ya da belge degisikligine baslama; engeli bildir ve gerekli erisim veya insan kararini iste.
