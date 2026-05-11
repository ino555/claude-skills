---
name: streaming-film-finder
description: >
  Searches the internet for the best movies currently available on Netflix, Prime Video, HBO Max, and Disney+
  that were added in the last 2 months from today's date. Filters candidates by IMDB score (must be above 6)
  and validates them against online reviews to ensure rating-review consistency. Delivers exactly 5 qualified
  films and saves them to a formatted Excel file with watch links. Optionally filters by genre if the user
  specifies one. If a film fails validation due to review inconsistency, it is automatically replaced with
  another candidate.

  Use this skill whenever the user asks for:
  - Streaming movie recommendations (Netflix, Prime, HBO, Disney+)
  - "What should I watch?" or "What's new on streaming?"
  - Top films from the last 2 months / recent streaming releases
  - Movies by genre on streaming platforms
  - A watchlist or film list saved to Excel/spreadsheet
  - "En iyi filmler", "izleyecek film öner", "streaming film önerileri"
  - "Son 2 ayda çıkan filmler", "Netflix'te ne var", "Prime'da iyi film var mı"
---

# Streaming Film Finder

Bu skill bugünün tarihini kullanarak son 2 ayda Netflix, Prime Video, HBO Max ve Disney+'ta yayına giren
en kaliteli 5 filmi bulur, doğrular ve Excel'e kaydeder.

## Adım 1: Tarihi ve Kategoriyi Belirle

- Bugünün tarihini belirle (sistem tarihini kullan).
- Son 2 aylık pencereyi hesapla (örn: bugün 11 Mayıs 2026 ise → 11 Mart 2026'dan itibaren).
- Kullanıcının mesajında bir film kategorisi/türü (genre) geçiyor mu? (Aksiyon, Korku, Komedi, Dram, Bilim Kurgu, Romantik vb.)
  - Geçiyorsa: sadece o türe odaklan.
  - Geçmiyorsa: tüm türleri dahil et.

## Adım 2: Aday Film Listesi Oluştur

En az 15 aday film bulana kadar arama yap. Bu buffer, doğrulama adımında bazıları elendikten sonra bile
5 kaliteli filme ulaşmamızı sağlar.

Aşağıdaki arama sorgularını kullan (ay ve yılı dinamik olarak güncel tarihe göre ayarla):
- `"new movies on Netflix [AY YIL]"` ve bir önceki ay için de
- `"new movies on Amazon Prime Video [AY YIL]"`
- `"new movies on HBO Max [AY YIL]"`
- `"new movies on Disney+ [AY YIL]"`
- Eğer genre belirtilmişse: `"best [TÜR] movies streaming [YIL]"` gibi ek sorgular

**Önemli:** Her aday için platforma eklenme/yayınlanma tarihini kaydet ve tarihin gerçekten son 2 aylık
pencere içinde olduğunu doğrula. Pencere dışındaki filmler kabul edilmez.

Aday filmleri bir liste olarak tut:
```
Film Adı | Platform | Eklenme Tarihi | Tür
```

## Adım 3: Her Aday İçin IMDB Doğrulaması

Her aday film için:
1. IMDB'de filmi ara: `"[Film Adı] [yıl] IMDB score"`
2. IMDB puanını bul.
3. **Kriter:** IMDB puanı 6.0'ın ALTINDA ise → filmi listeden çıkar, bir sonrakine geç.
4. IMDB puanı 6.0 ve üzerinde ise → Adım 4'e geç.

## Adım 4: İnceleme/Yorum Tutarlılık Kontrolü

IMDB puanı geçen her film için çevrimiçi yorumları kontrol et:

1. Şu kaynaklarda arama yap:
   - `"[Film Adı] Rotten Tomatoes score"` → % puanı bul
   - `"[Film Adı] Metacritic score"` → metacritic puanını bul
   - `"[Film Adı] review 2025 2026"` → genel izleyici tepkisine bak

2. **Tutarlılık kontrolü:**
   - IMDB 6+ VE Rotten Tomatoes/Metacritic yorumları büyük ölçüde pozitif → **KABUL**
   - IMDB 6+ AMA yorumlar ağırlıklı negatif (örn. RT %40 altında) → **RET** (uyumsuzluk)

3. **Tür doğrulaması:** Kullanıcı belirli bir tür istiyorsa, filmin o türün asıl temsilcisi olduğunu
   doğrula. Borderline filmler (örn. "aksiyon/thriller" yerine "korku" için istenen liste için "thriller")
   hariç tutulmalıdır.

4. Her kabul edilen film için şu bilgileri kaydet:
   - Film adı (orijinal dil + varsa Türkçe başlık)
   - Türler (genres)
   - Platformlar (Netflix, Prime Video, HBO Max, Disney+)
   - Platforma eklenme tarihi
   - IMDB puanı
   - Rotten Tomatoes % (bulunabiliyorsa)
   - Kısa yorum özeti (1-2 cümle, Türkçe)
   - IMDB linki
   - Platform izleme linki

5. Tam olarak 5 kabul edilen film bulana kadar devam et. Gerekirse daha fazla aday ara.

## Adım 5: Excel Dosyası Oluştur

`scripts/save_to_excel.py` scriptini kullan. Script'e film verilerini JSON string olarak geçir:

```bash
python "SKILL_DIR/scripts/save_to_excel.py" '[{"rank":1,"title":"Film","title_tr":"","genres":"Drama","platforms":"Netflix","added_date":"2026-04-01","imdb_score":7.5,"rt_score":82,"review_summary":"Harika bir film.","imdb_url":"https://www.imdb.com/title/ttXXXXX/","watch_url":"https://www.netflix.com/"}]' "output_path/streaming_top5_YYYY-MM-DD.xlsx"
```

**Dosya adı formatı:** `streaming_top5_[YYYY-MM-DD].xlsx` — bugünün tarihi ile.
**Kayıt yeri:** Kullanıcının çalışma dizini veya masaüstü.

**Not:** Script otomatik olarak openpyxl yükler. Eğer JSON arg'ını geçirirken sorun yaşanırsa,
önce bir Python dosyasına yaz ve oradan çalıştır.

## Adım 6: Sonuçları Kullanıcıya Göster

Excel kaydedildikten sonra kullanıcıya şu formatta göster:

```
🎬 Son 2 Ayın En İyi 5 Streaming Filmi ([TARİH])
[Kategori: XXX] veya [Tüm Kategoriler]

1. [Film Adı] — [Platform] | IMDB: X.X | RT: XX%
   [Kısa özet — Türkçe]
   IMDB: [link]

...

📊 Sonuçlar kaydedildi: streaming_top5_[tarih].xlsx

[Varsa] ⚠️ [N] film elendi: [Film Adı] (neden — örn: "IMDB 5.2, RT 41%")
```

## Önemli Kurallar

- Her film için **hem** IMDB 6+ **hem de** pozitif yorumlar şartı birlikte aranır.
- 2 aylık tarih penceresi katı uygulanır — pencere dışındaki filmler asla kabul edilmez.
- Yorum uyumsuzluğu nedeniyle elenen filmler kullanıcıya bildirilir.
- Film adları orijinal dilinde kalır; bilinen Türkçe başlık varsa parantez içinde eklenir.
- Eğer kullanıcı Türkçe konuşuyorsa, tüm açıklamalar Türkçe yapılır.
