---
name: streaming-film-finder
description: >
  Searches the internet for the best movies currently available in TURKEY on Netflix TR, Prime Video TR,
  HBO Max TR, and Disney+ TR that were added in the last 2 months from today's date. CRITICAL: every film
  must be verified as actually watchable in Turkey using JustWatch Turkey (justwatch.com/tr) before inclusion.
  Filters by IMDB score (must be above 6) and validates review consistency. Delivers exactly 5 qualified
  Turkey-available films and saves them to a formatted Excel file with Turkish platform watch links.
  Optionally filters by genre. Rejected films (low score, bad reviews, or NOT available in Turkey) are
  automatically replaced.

  Use this skill whenever the user asks for:
  - Streaming movie recommendations (Netflix, Prime, HBO, Disney+)
  - "What should I watch?" or "What's new on streaming?"
  - Top films from the last 2 months / recent streaming releases
  - Movies by genre on streaming platforms
  - A watchlist or film list saved to Excel/spreadsheet
  - "En iyi filmler", "izleyecek film öner", "streaming film önerileri"
  - "Son 2 ayda çıkan filmler", "Netflix'te ne var", "Prime'da iyi film var mı"
  - "Türkiye'de izlenebilir filmler", "hangi filmler açılıyor"
---

# Streaming Film Finder — Türkiye Kataloğu

Bu skill **yalnızca Türkiye'de gerçekten izlenebilen** filmleri bulur, doğrular ve Excel'e kaydeder.
Türkiye'de açılmayan filmler kesinlikle kabul edilmez.

---

## Adım 1: Tarihi ve Kategoriyi Belirle

- Bugünün tarihini belirle (sistem tarihini kullan).
- Son 2 aylık pencereyi hesapla (örn: bugün 11 Mayıs 2026 ise → 11 Mart 2026'dan itibaren).
- Kullanıcının mesajında film türü var mı? (Aksiyon, Korku, Komedi, Dram, Bilim Kurgu, Romantik vb.)
  - Varsa: sadece o türe odaklan.
  - Yoksa: tüm türleri dahil et.

---

## Adım 2: JustWatch Türkiye'den Aday Listesi Oluştur

**JustWatch Türkiye birincil kaynağımızdır** — çünkü ülke bazlı katalog gösterir ve hangi filmin
Türkiye'de hangi platformda açık olduğunu kesin olarak bilir.

### 2a. JustWatch TR'de Yeni Eklenen Filmleri Ara

Şu arama sorgularını kullan:

```
site:justwatch.com/tr yeni filmler Netflix [AY YIL]
site:justwatch.com/tr yeni filmler Prime Video [AY YIL]
justwatch.com/tr yeni eklenen filmler [AY YIL]
"justwatch" "türkiye" yeni film [AY YIL]
```

Ayrıca doğrudan JustWatch Türkiye filtreli URL'leri kontrol et:
- `https://www.justwatch.com/tr/filmler?providers=nfx&release_year_from=YYYY` (Netflix TR)
- `https://www.justwatch.com/tr/filmler?providers=amp&release_year_from=YYYY` (Prime TR)
- `https://www.justwatch.com/tr/filmler?providers=hbm&release_year_from=YYYY` (HBO Max TR)
- `https://www.justwatch.com/tr/filmler?providers=dnp&release_year_from=YYYY` (Disney+ TR)

### 2b. Platform TR Kataloglarını Destekle

Ek kaynaklar olarak şunları da ara:
```
"Netflix Türkiye" yeni film [AY YIL]
"Prime Video Türkiye" yeni film [AY YIL]
"HBO Max Türkiye" yeni film [AY YIL]
"Disney Plus Türkiye" yeni film [AY YIL]
```

Eğer genre belirtilmişse ek arama:
```
"[TÜR] film" "Netflix Türkiye" VEYA "Prime Video Türkiye" [YIL]
justwatch.com/tr [tür] filmler
```

### 2c. Aday Listesi

En az **15 aday** toplayana kadar aramaya devam et. Her aday için kaydet:
```
Film Adı | Platform (TR) | Türkiye'ye Eklenme Tarihi | Tür | JustWatch TR Linki
```

**Tarih filtresi:** Türkiye platformuna eklenme tarihi son 2 aylık pencere içinde olmalı.

---

## Adım 3: Türkiye Erişilebilirliğini Doğrula (KRİTİK)

Her aday için JustWatch Türkiye'de ayrıca doğrulama yap:

1. Şu sorgu ile filmin Türkiye'deki durumunu kontrol et:
   ```
   "[Film Adı]" site:justwatch.com/tr
   ```
   veya
   ```
   "[Film Adı]" justwatch türkiye izle
   ```

2. **Doğrulama kriterleri:**
   - JustWatch TR sayfasında film var VE platform ikonu (Netflix, Prime vb.) görünüyor → **TÜRKİYE'DE MEVCUT**
   - Film sadece belirli ülkelerde var veya JustWatch TR'de platform ikonu yok → **RET** (Türkiye'de yok)
   - Film "kısa süre önce kaldırıldı" veya "yakında geliyor" durumunda → **RET**

3. Kabul edilen filmler için JustWatch TR film sayfasındaki **platform izleme linkini** kaydet.
   Bu link genellikle şu formattadır:
   - Netflix: `https://www.netflix.com/tr/title/XXXXX`
   - Prime: `https://www.primevideo.com/...` veya `https://www.amazon.com.tr/...`
   - HBO Max: `https://www.hbomax.com/tr/...` veya `https://www.max.com/tr/...`
   - Disney+: `https://www.disneyplus.com/tr-tr/movies/...`

> **Neden bu kadar önemli?** Netflix, Prime vb. platformların küresel kataloğuyla Türkiye kataloğu
> farklıdır. Birçok film ABD'de açık olsa da Türkiye'de telif hakkı kısıtlamaları nedeniyle kapalı
> olabilir. JustWatch TR bu farkı kesin olarak gösterir.

---

## Adım 4: IMDB Puanı Doğrula

Türkiye'de mevcut olduğu doğrulanan filmler için:

1. IMDB'de ara: `"[Film Adı] [yıl] IMDB rating"`
2. IMDB puanını bul.
3. **Kriter:** Puan 6.0 ALTI → RET (yetersiz puan). Puan 6.0 ve üzeri → Adım 5'e geç.

---

## Adım 5: Yorum Tutarlılık Kontrolü

IMDB puanı 6+ olan filmler için:

1. Şu kaynaklarda ara:
   - `"[Film Adı]" Rotten Tomatoes` → taze/çürük yüzdesi
   - `"[Film Adı]" Metacritic` → metaskor

2. **Tutarlılık kriteri:**
   - IMDB 6+ **VE** RT/Metacritic yorumları ağırlıklı pozitif → **KABUL**
   - IMDB 6+ **AMA** RT %40 altında veya yorumlar çoğunlukla negatif → **RET** (tutarsızlık)

3. **Tür doğrulaması:** Kullanıcı belirli tür istediyse, film o türün gerçek temsilcisi olmalı.
   Sınırda filmler (örn. korku listesi için thriller) reddedilir.

4. Kabul edilen her film için şu bilgileri kaydet:
   - Film adı (orijinal) + Türkçe adı (varsa)
   - Türler
   - Platform (Türkiye'de erişilebilir)
   - Türkiye'ye eklenme tarihi
   - IMDB puanı
   - RT % (bulunabiliyorsa)
   - Kısa yorum özeti (Türkçe, 1-2 cümle)
   - IMDB linki
   - Türkiye platform izleme linki (JustWatch TR'den alınan)

5. Tam olarak **5 kabul edilen film** bulana kadar devam et. Gerekirse daha fazla aday ara.

---

## Adım 6: Excel Dosyası Oluştur

`scripts/save_to_excel.py` scriptini kullan:

```bash
python "SKILL_DIR/scripts/save_to_excel.py" '[
  {
    "rank": 1,
    "title": "Film Adi",
    "title_tr": "Turkce Adi (varsa)",
    "genres": "Drama, Gerilim",
    "platforms": "Netflix TR",
    "added_date": "2026-04-01",
    "imdb_score": 7.5,
    "rt_score": 82,
    "review_summary": "Kisa Turkce ozet.",
    "imdb_url": "https://www.imdb.com/title/ttXXXXX/",
    "watch_url": "https://www.netflix.com/tr/title/XXXXX"
  }
]' "CIKTI_KLASORU/streaming_top5_YYYY-MM-DD.xlsx"
```

**Dosya adı:** `streaming_top5_[YYYY-MM-DD].xlsx`  
**Kayıt yeri:** Kullanıcının çalışma dizini veya masaüstü.  
**Not:** Script openpyxl'i otomatik yükler. JSON argümanı sorun çıkarırsa önce bir .py dosyasına yaz.

---

## Adım 7: Sonuçları Kullanıcıya Göster

```
🎬 Son 2 Ayın En İyi 5 Streaming Filmi — Türkiye Kataloğu ([TARİH])
[Kategori: XXX] veya [Tüm Kategoriler]

1. [Film Adı] ([Türkçe Ad]) — [Platform TR] | IMDB: X.X | RT: XX%
   [Kısa Türkçe özet]
   İzle: [platform TR linki] | IMDB: [imdb linki]

2. ...

📊 Excel kaydedildi: streaming_top5_[tarih].xlsx

⚠️ Elenen filmler: [N] film — [Nedenler: Türkiye'de yok / IMDB düşük / yorum uyumsuz]
```

---

## Önem Sırası ile Eleme Kuralları

Bir film şu durumlarda KESİNLİKLE reddedilir (öncelik sırasıyla):

1. **Türkiye'de mevcut değil** — JustWatch TR'de platform ikonu yok veya bölge kısıtlaması var
2. **Tarih penceresi dışı** — son 2 ay içinde Türkiye platformuna eklenmemiş
3. **IMDB 6.0 altı** — puan yetersiz
4. **Yorum tutarsızlığı** — IMDB ile RT/Metacritic çelişiyor, yorumlar ağırlıklı negatif
5. **Yanlış tür** — kullanıcı belirli bir tür istedi, film o türün gerçek temsilcisi değil

Elenen filmlerin yerine sıradaki adaydan devam et. Tüm elemeler kullanıcıya bildirilir.
