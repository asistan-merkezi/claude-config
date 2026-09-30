---
name: gorsel-uretim
description: Statik görsel tasarım üretimi standartları — sosyal medya görseli, kapak, banner, sertifika, davetiye, infografik gibi tek görsellerin programatik üretimi (PNG/PDF/SVG çıktı). Kullanıcı sosyal medya görseli, kapak tasarımı, banner, OG image, sertifika, davetiye, duyuru görseli veya infografik istediğinde MUTLAKA bu skill'i kullan. "Görsel tasarla", "kapak yap", "banner üret", "OG image", "sertifika şablonu", "infografik çıkar" gibi ifadeler geçtiğinde de kullan. AI görsel üretimi (Higgsfield) media-content'e, fotoğraf işleme media-editing'e tabidir — bu skill tipografi/yerleşim ağırlıklı tasarımların kod ile deterministik üretimini taşır.
---

# Görsel Üretim (Programatik Statik Görsel Üretimi)

Tipografi ve yerleşim ağırlıklı görsellerin kodla üretimi. Marka renk/font kuralları brand-ui'ye, AI görsel üretimi media-content'e, mevcut görselin işlenmesi media-editing'e aittir.

## Temel ilkeler

1. **Deterministik üretim**: Tekrar üretilebilirlik esastır — aynı girdi aynı görseli verir. Araç önceliği: HTML/CSS → screenshot (karmaşık yerleşim, Playwright ile), SVG → PNG dönüşümü (vektörel/şablon işler), PIL/Pillow (basit birleştirme). Şablonlaşan her görsel parametrik fonksiyon olur (başlık, tarih, isim değişkenleri).
2. **Marka dilinden çıkma yok**: Renk, font ve ton brand-ui'deki sistemden gelir; görsel başına yeni renk icat edilmez. Marka tanımsızsa üretimden önce tek soruyla netleştirilir.
3. **Türkçe tipografi**: Seçilen font Türkçe karakterleri (ğ, ş, ı, İ, ç, ö, ü) tam desteklemeli — üretim sonrası ilk kontrol budur. Büyük harf dönüşümünde İ/ı sorunu (locale-aware upper) gözetilir.
4. **Slop denetimi görsele de uygulanır**: brand-ui'nin anti-jenerik kuralları geçerli — varsayılan mor gradyan, ortalanmış-her-şey, anlamsız stok ikon yığını yasak.

## Format standartları

- Hedef platform ölçüsü baştan sorulur/bilinir; ölçülerin tek kaynağı `media-content`'tir (OG image, post, story, YouTube kapak — burada tekrarlanmaz). Tek görseli her boyuta bozarak sığdırmak yerine yerleşim boyuta uyarlanır.
- Metin güvenli alanı: kenarlardan içeride tutulur; platformun kırptığı bölgelere (story alt/üst UI şeridi) kritik içerik konmaz.
- Kontrast erişilebilirlik eşiğini geçer; görsel üstü metin gerekiyorsa karartma katmanı/plaka ile okunabilirlik garanti edilir.
- Çıktı formatı kullanım yerine göre: web'de WebP/PNG, baskı-benzeri işlerde (sertifika, davetiye) PDF (pdf-generation standartlarıyla), şablonlar SVG olarak da saklanır.

## İş akışı

taslak üret → gerçek boyutta görsel kontrol (yazım hatası, taşma, Türkçe karakter) → parametrik hale getir → çıktıyı hedef formatta teslim. Seri üretimde (ör. 50 sertifika) önce 1 örnek onaya sunulur, onay sonrası toplu üretim yapılır (approval-workflow ilkesiyle tutarlı).
