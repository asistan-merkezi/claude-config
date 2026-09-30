---
name: media-editing
description: Görsel ve video düzenleme/işleme standartları (boyutlandırma, kırpma, format dönüşümü, sıkıştırma, arka plan kaldırma, upscale, metin bindirme, video montaj). Kullanıcı görsel boyutlandırma, format çevirme (webp, mp4 vb.), logo/watermark ekleme, görsel optimizasyonu, video kesme/birleştirme, ffmpeg veya PIL ile işlem istediğinde MUTLAKA bu skill'i kullan. "Resmi küçült", "webp'ye çevir", "arka planı kaldır", "videoyu kes", "watermark ekle", "görseli optimize et", "yüklenen fotoğrafı sunucuda işle", "sharp ile thumbnail üret", "EXIF temizle" gibi ifadeler geçtiğinde de kullan. Uygulama içi yükleme hattının işleme adımı (sharp, yön düzeltme, varyant üretimi) da bu skill'e tabidir; modülün mimarisi medya-kutuphanesi'ndedir. Programatik-öncelikli, kayıpsız ve tekrarlanabilir medya işleme üretir.
---

# Media Editing

Görsel ve video düzenleme/işleme standartları. Projeye özel detaylar (marka renkleri, fontlar, logo dosyaları) CLAUDE.md'dedir.

**Sınırlar:** İçeriğin NE olacağı (senaryo, konsept) ve platform standart boyutları `media-content`'tedir — bu skill oradaki boyutları uygular, kendi başına platform ölçüsü tanımlamaz. Uygulama içi medya modülünün mimarisi (storage düzeni, DB kaydı, yetki, listeleme) `medya-kutuphanesi`, yükleme güvenliği (MIME/magic byte/boyut sınırı) `security-baseline` işidir. Bu skill **dosyanın kendisini işler**: nasıl boyutlandırılır, dönüştürülür, sıkıştırılır, bindirilir.

## Çalışma bağlamı: hangi araç nerede

İki ayrı bağlam vardır, kuralları aynı ama araçları farklıdır — karıştırılmaz:

| Bağlam | Araç | Tipik yer |
|---|---|---|
| Offline / toplu / tek seferlik iş | **Python: PIL/Pillow + ffmpeg** | script, CLI, Codespaces |
| Uygulama runtime'ı (istek anında işleme) | **Node: sharp** (video gerekiyorsa sunucuda ffmpeg) | Next.js route handler, worker |

Next.js route'unda PIL çağıran kod yazma; toplu klasör işinde sharp'a zorlama. Boyut/kalite hedefleri, EXIF kuralı ve varyant sözlüğü her iki bağlamda **aynıdır**.

## Araç seçim önceliği

1. **Programatik (PIL/Pillow, ffmpeg, sharp)**: Varsayılan. Deterministik, ücretsiz, tekrarlanabilir, toplu işlenebilir. Boyutlandırma, kırpma, format, sıkıştırma, metin/logo bindirme, video kesme/birleştirme işleri BURADA yapılır.
2. **AI araçları (Higgsfield vb.)**: Yalnızca programatik yapılamayan işlerde — arka plan kaldırma (karmaşık kenarlar), upscale, içerik üretme/değiştirme. AI'a giden işte bile son montaj (metin, logo, boyut) programatik yapılır.
3. Tek seferlik ufak iş bile olsa script yaz — komutu/scripti sakla, aynı iş tekrar geldiğinde parametre değiştirip koşulur.

## Görsel işleme (PIL/Pillow)

**Boyutlandırma/kırpma:**
- Oranı koruyarak küçült: `Image.thumbnail()`. Hedef orana kırpmak gerekiyorsa merkezden kırp (ör. 4:5 post için), önemli özne merkezde değilse kırpma kutusunu elle parametre yap.
- Asla küçük görseli büyütme (kalite kaybı) — büyütme gerekiyorsa AI upscale'e yönlendir.
- Yeniden örnekleme: küçültmede `Image.LANCZOS`.

**Format ve sıkıştırma:**
- Web hedefi: WebP (kalite 80-85) varsayılan; şeffaflık gerekiyorsa WebP/PNG; e-posta uyumluluğu gerekiyorsa JPEG.
- JPEG kaydında `quality=85, optimize=True`.
- Aşıyorsa önce boyut (piksel) düşür, sonra kalite.

**Varyant sözlüğü (tek kaynak — diğer skill'ler bu adları kullanır):**

| Varyant | En uzun kenar | Ağırlık hedefi | Kullanım |
|---|---|---|---|
| `thumb` | ~400px | < 50KB | liste, ızgara, kart önizleme |
| `md` | ~1200px | < 200KB | içerik görseli, detay ekranı |
| `lg` | ~2000px | < 400KB | hero, tam ekran önizleme |

"thumbnail / içerik görseli / hero" ifadeleri sırasıyla `thumb / md / lg` demektir; kodda ve storage path'inde kısa adlar kullanılır.

**EXIF ve yön (zorunlu sıra):**
1. Önce `.rotate()` / `exif_transpose()` ile **yönü uygula**.
2. Sonra tüm meta/EXIF'i **sil** (GPS dahil).

Bu sıra bozulursa telefon fotoğrafları yan döner: EXIF silinince oryantasyon bilgisi de gider. "Sadece EXIF temizle" yeterli değildir. Gizlilik gerekçesiyle EXIF temizliği web'e çıkan her görselde zorunludur, öneri değildir.

**Metin/logo bindirme:**
- Fontlar dosyadan yüklenir (`ImageFont.truetype`) — marka font dosya yolları CLAUDE.md'de. Sistem varsayılan fontuyla marka işi ÇIKMAZ.
- Türkçe karakter testi zorunlu: bindirmeden sonra ğ/ş/ı/İ kontrol et (font desteklemiyorsa değiştir).
- Okunabilirlik: metin arkasına yarı saydam bant veya hafif gölge; kontrastı düşük bindirme yapma.
- Logo/watermark: köşe yerleşimi + kenarlardan sabit pay (%3-5); opaklık watermark'ta %30-50, imza logoda %100.

**Toplu işleme:** klasör bazlı script deseni — girdi klasörü → işlem → çıktı klasörü (orijinalin ÜZERİNE yazma). Dosya adlarını koru veya anlamlı sonek ekle (`_1080x1350`).

> Bu adlandırma kuralı **yalnızca offline/toplu iş** içindir. Uygulama içi yüklemede dosya adı UUID'dir, kullanıcı girdisi dosya adına/path'e girmez — bkz. aşağıdaki yükleme hattı ve `medya-kutuphanesi`.

## Video işleme (ffmpeg)

**Temel kurallar:**
- Kayıpsız işler (kesme, birleştirme, ses ayırma) yeniden encode ETMEDEN yapılır: `-c copy`. Yalnızca filtre/boyut değişiminde encode edilir.
- Encode gerekiyorsa: H.264 (`libx264`), `crf 20-23`, `preset medium`, ses `aac 128k`. Sosyal medya dikeyi: 1080x1920, 30fps.
- Kesme: `-ss`/`-to` girdiden ÖNCE verilir (`-ss 10 -i girdi.mp4`): hızlıdır ve yeniden encode edilen kesimde (modern ffmpeg) kare hassasiyetindedir. `-c copy` ile kesimde ise en yakın keyframe'e yuvarlanır; tam kare hassasiyeti gerekiyorsa encode ederek kes.

**Sık desenler:**
- Birleştirme (aynı codec): concat demuxer + dosya listesi txt.
- Görsellerden video (slayt/şablon içerik): `-framerate` + `-loop` ile görsel başına süre; geçişler `xfade` filtresiyle.
- Ses ekleme/değiştirme: `-map` ile video ve ses akışları açıkça seçilir; süre uyumsuzsa `-shortest`.
- Boyut/kırpma: `scale` ve `crop` filtreleri; 9:16'ya dönüştürmede önemli alan kaybolmasın diye önce `crop` sonra `scale`.
- Metin bindirme: `drawtext` (fontfile ile marka fontu). Çok satır/stilli işlerde: metni PIL ile şeffaf PNG üret + `overlay` filtresiyle bindir — Türkçe ve tipografi kontrolü daha iyi.

**Kontrol:** işlem sonrası `ffprobe` ile süre/çözünürlük/codec doğrula; çıktıyı açıp ilk-orta-son saniyeleri gözle kontrol etme alışkanlığı öner.

## Uygulama içi otomatik yükleme hattı

Kullanıcı dosya yüklediğinde çalışan işleme hattı. Modülün mimarisi (storage path şeması, DB kaydı, yetki, listeleme, silme) `medya-kutuphanesi`; doğrulama (MIME/magic byte/boyut sınırı) `security-baseline`. Burada yalnızca **işleme** tanımlıdır.

- **İşleme sunucuda yapılır.** Client tarafı küçültme yalnızca yükleme süresini kısaltmak içindir (UX); işlemenin kendisi client'a bırakılmaz — route'a doğrudan işlenmemiş dosya atılabileceği varsayılır.
- Sıra: `sharp(input).rotate()` → meta temizliği → varyant üretimi → WebP çıktı. `.rotate()` argümansız çağrılır (EXIF yönünü uygular), meta temizliği varsayılandır — `withMetadata()` çağrılmaz.
- Varyant seti yukarıdaki sözlükten: `thumb` / `md` / `lg`. Kaynak `lg`'den küçükse büyütülmez, o varyant üretilmez (kalite kaybı).
- **Dosya adı UUID**, uzantı çıktının gerçek formatına göre yazılır (girdi uzantısı kopyalanmaz). Kullanıcının gördüğü orijinal ad DB'de tutulur.
- Tek modül: `lib/image-pipeline.ts` — projeler arası aynı arayüz (`processUpload(file) → { thumb, md, lg, width, height, mime, size }`). Her yükleme ekranı kendi işleme kodunu yazmaz.
- **Toplu yüklemede kuyruk:** dosyalar sırayla işlenir, durum `processing / ready / failed` olarak tutulur. Bir dosyanın hatası kalanları düşürmez; başarısız kayıt `failed` kalır, sessizce yok sayılmaz.
- Video yüklemesi varsa aynı hat: sunucuda ffmpeg ile normalize et (H.264, hedef çözünürlük), kapak karesi (`poster`) üret.

## AI destekli işlemler

- Arka plan kaldırma: önce programatik dene (`rembg` kütüphanesi — yerel, ücretsiz); sonuç yetersizse AI servisine geç.
- Upscale: küçük kaynak görseli büyütme ihtiyacında AI upscale kullan; çıktıda yüz/metin bozulması kontrol et.
- AI çıktısı her zaman son montajdan geçer: boyut standardı + marka öğeleri programatik eklenir.

## Kalite kontrol listesi

- [ ] Çıktı boyutu hedef platform standardında (`media-content`)
- [ ] Dosya ağırlığı varyant sözlüğündeki hedefin altında
- [ ] Yön doğru (rotate, meta temizliğinden ÖNCE uygulanmış)
- [ ] EXIF/GPS temizlenmiş
- [ ] Türkçe karakterler doğru görünüyor
- [ ] Orijinal dosya korunmuş (üzerine yazılmamış)
- [ ] Toplu işse rastgele 2-3 çıktı gözle doğrulanmış

## Çıktı formatı

- Tek iş: çalıştırmaya hazır komut/script + 1 cümle ne yaptığı.
- Toplu/tekrarlanacak iş: parametreli script (girdi/çıktı klasörü, boyut, kalite argümanlarıyla) + örnek çağrı.
- Kurulum gereken paket varsa komutuyla belirt (`pip install pillow rembg`; ffmpeg: Linux `apt install ffmpeg`, Windows `winget install ffmpeg`).
