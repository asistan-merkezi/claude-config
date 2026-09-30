---
name: programmatic-seo
description: Programatik SEO standartları — veriden ölçekli, indekslenebilir sayfa üretimi (şablon sayfa + veri seti → yüzlerce/binlerce landing). Kullanıcı veriden otomatik sayfa üretme, şehir/kategori/ürün bazlı sayfa çoğaltma, "her X için ayrı sayfa", karşılaştırma sayfaları ağı veya ölçekli SEO sayfa yapısı istediğinde MUTLAKA bu skill'i kullan. "Programatik SEO", "her il için sayfa", "fon başına sayfa", "otomatik landing", "binlerce sayfa" gibi ifadeler geçtiğinde de kullan. Klasik on-page/GEO kuralları seo-content'e tabidir — bu skill ölçekli üretimin mimarisini ve kalite eşiğini taşır.
---

# Programmatic SEO (Veriden Ölçekli Sayfa Üretimi)

Şablon + veri setiyle çok sayıda indekslenebilir sayfa üretme standartları. Sayfa içi SEO/GEO kuralları seo-content'e, veri toplama data-pipeline'a aittir.

## Temel ilkeler

1. **Thin content eşiği**: Her üretilen sayfa, o sorgu için tek başına yararlı olmalı. Kural: sayfadaki benzersiz (şablon dışı) veri/içerik payı belirgin olmalı — sadece başlıktaki kelime değişiyorsa o sayfa üretilmez. Google'ın ölçekli düşük kaliteli içerik cezası gerçek risktir.
2. **Veri yoksa sayfa yok**: Veri seti eksik olan kombinasyonlar için boş şablonlu sayfa yayınlanmaz; o URL hiç oluşturulmaz veya noindex alır.
3. **Sorgu talebi doğrulaması**: Sayfa ağı, insanların gerçekten aradığı kalıba oturur ("[fon adı] getirisi", "[il] [hizmet] fiyatları"). Kimsenin aramadığı kombinasyon çoğaltılmaz.
4. **Deterministik üretim**: Sayfalar veriden şablonla üretilir; LLM yalnızca yorum katmanında ve ai-report kurallarıyla kullanılır (uydurma istatistik yasak).

## Mimari (Next.js App Router)

- Dinamik segment + `generateStaticParams` ile SSG/ISR; sayfa sayısı büyükse ISR + on-demand revalidation. Client-side render edilen içerik indekslenme açısından yok hükmündedir (seo-content GEO kuralıyla tutarlı).
- URL şeması insan-okur ve hiyerarşik: `/fonlar/[fon-kodu]`, `/[il]/[hizmet]`. Query string ile sayfa çoğaltılmaz.
- `generateMetadata` veriden benzersiz title/description üretir; kalıp cümleye sadece değişken gömme ("X hakkında her şey") yerine sayfanın gerçek verisinden özet.
- Sitemap otomatik ve bölünmüş (50k URL sınırı); yeni/silinen sayfalar sitemap'e cron ile yansır.
- Yapısal veri (schema.org) sayfa tipine göre şablonda: Product, FAQPage, Dataset, LocalBusiness vb.

## İç bağlantı ağı

- Her programatik sayfa yukarı (kategori/hub) ve yana (ilgili kardeş sayfalar) bağlanır; yetim sayfa yasak.
- Hub sayfaları (seo-content'teki pillar yapısıyla) programatik sayfaların keşif kapısıdır; ana menüden erişilebilir.
- İlgili sayfa önerileri veriden hesaplanır (aynı kategori, benzer değer aralığı), rastgele seçilmez.

## Kalite kontrol döngüsü

- Yayın öncesi örneklem denetimi: rastgele 10 sayfa elle okunur — benzersiz değer var mı, veri doğru mu, şablon sırıtıyor mu.
- Yayın sonrası izleme: Search Console'da indekslenme oranı ve "crawled - not indexed" birikimi izlenir; indekslenmeyen sayfa kümesi büyüyorsa şablon zenginleştirilir veya küme budanır.
- Sayfa ağı büyütme kademeli: önce küçük küme (50-100 sayfa) → indekslenme/trafik kanıtı → genişletme. Günde binlerce sayfa basıp beklemek yasak.

## Çıktı formatı

Sayfa ağı planı tablosu (şablon | veri kaynağı | URL kalıbı | tahmini sayfa sayısı | hedef sorgu kalıbı | benzersiz içerik payı) + şablon / `generateMetadata` / sitemap kodu dosya yollarıyla + ilk kademe (50-100 sayfa) yayın ve izleme planı.
