---
name: seo-content
description: Türkçe SEO ve organik trafik odaklı içerik/teknik optimizasyon standartları. Kullanıcı SEO iyileştirmesi, meta etiketleri, sitemap, blog yazısı planı, anahtar kelime hedefli sayfa, Google sıralaması, structured data (schema), Open Graph veya arama trafiği artırma istediğinde MUTLAKA bu skill'i kullan. "SEO yap", "Google'da çıksın", "meta ekle", "sitemap oluştur", "blog planı", "anahtar kelime" gibi ifadeler geçtiğinde de kullan. Next.js App Router için teknik SEO + Türkçe içerik SEO'su birlikte üretir.
---

# SEO Content

Türkçe pazar için SEO standartları: teknik altyapı (Next.js App Router) + içerik optimizasyonu birlikte. Projeye özel detaylar (domain, hedef kelimeler, marka adı) CLAUDE.md'dedir. Metin kalitesi kuralları için `copywriting` skill'i de geçerlidir — SEO metni önce İYİ metin olmalıdır.

## Temel yaklaşım

1. **Arama niyeti > anahtar kelime**: Sayfa, kelimeyi tekrarlamakla değil, o kelimeyi arayanın sorusunu tam cevaplamakla sıralanır. İçerik planlarken önce niyeti sınıflandır: bilgi arıyor / karşılaştırıyor / satın almaya hazır — sayfa tipi buna göre seçilir (blog / karşılaştırma / ürün-landing).
2. **Türkçe pazar gerçeği**: rekabet İngilizce'ye göre düşüktür; uzun kuyruk Türkçe sorgular ("x nasıl hesaplanır", "x programı ücretsiz") hızlı kazanımdır. Kelime araştırmasında Google otomatik tamamlama + "İlgili aramalar" + rakip başlıkları temel kaynak.
3. **Bir sayfa = bir ana niyet**: aynı niyeti hedefleyen iki sayfa üretme (kendi kendinle rekabet); yakın konuları tek güçlü sayfada birleştir.

## Teknik SEO (Next.js App Router)

**Metadata:**
- Her sayfada `generateMetadata` (dinamik) veya `metadata` export'u (statik): `title`, `description`, `alternates.canonical`, Open Graph (`og:title`, `og:description`, `og:image` 1200x630) ve `twitter` kartı.
- Title şablonu layout'ta: `{ template: '%s | Marka', default: 'Marka — ana vaat' }`.
- Title 50-60 karakter, ana kelime başta; description 140-160 karakter, fayda + eylem (`copywriting` formatları geçerli).

**Site altyapısı:**
- `app/sitemap.ts` ile dinamik sitemap (statik sayfalar + veritabanından içerik URL'leri, `lastModified` ile); `app/robots.ts` ile robots (panel/auth route'ları `disallow`).
- Canonical her sayfada mutlak URL; www/apex ve sondaki slash tutarlılığı tek biçimde.
- URL yapısı: kısa, Türkçe, tireli, TÜRKÇE KARAKTERSİZ slug (`/metraj-hesaplama`, `/ogrenci-kayit` — ğ/ş/ı dönüştürülür). Slug üretimini tek yardımcı fonksiyonda topla.
- 404 yerine kalıcı taşınan içerikte 301 redirect (`next.config` `redirects`; istek düzeyi mantık gerekiyorsa Next 16'da `middleware.ts` yerine `proxy.ts`).

**Structured data (JSON-LD):**
- Uygun tiplerle script bloğu ekle: `Organization` (ana sayfa), `Article` (blog), `FAQPage` (SSS bölümü olan sayfalar), `SoftwareApplication`/`Product` (ürün), `BreadcrumbList`.
- FAQPage: Google 2023'ten beri SSS zengin sonucunu yalnızca bilinen devlet ve sağlık sitelerine gösteriyor; diğer sitelerde zengin sonuç BEKLENMEZ. SSS bölümü okuyucuya ve "nasıl/nedir" sorgularına değer kattığı için yazılır; şema eklenebilir ama sonuç vaat edilmez.

**Performans sinyalleri:**
- Görseller `next/image` ile (otomatik boyut + lazy load), LCP görseline `priority`.
- Font `next/font` ile (layout kayması önlenir). Core Web Vitals'ı bozan üçüncü parti script'leri `afterInteractive`/`lazyOnload` ile yükle.

## İçerik SEO'su

**Sayfa yapısı:**
- H1 tek ve ana niyeti karşılar; H2'ler alt soruları cevaplar (H2'leri "insanların sorduğu sorular" gibi kur — otomatik tamamlamadan beslen). Hiyerarşi atlanmaz (H2'den H4'e zıplama).
- İlk paragraf sorunun kısa net cevabını verir (featured snippet hedefi); detay sonra açılır.
- Anahtar kelime H1'de, ilk paragrafta ve birkaç H2'de DOĞAL geçer; yoğunluk takıntısı yok, eş anlamlılar serbest ve faydalı.
- İç bağlantı: her içerik, ilgili 2-4 sayfaya anlamlı çapa metniyle bağlanır ("buraya tıklayın" YASAK); yeni içerik yayınlanınca eski ilgili sayfalardan ona link verilir.

**Blog/içerik planı istendiğinde:**
- Çıktı: konu kümesi (pillar) + etrafında 5-10 destek içerik; her satırda hedef sorgu, niyet tipi, sayfa tipi, öncelik.
- Araç/hesaplayıcı tipi sayfalar (ör. ücretsiz mini hesaplama aracı) Türkçe pazarda güçlü backlink/trafik mıknatısıdır; SaaS projelerinde ürünün küçük bir dilimini ücretsiz araç olarak açmayı değerlendir.

**Güncellik:** sıralanan içerik yılda 1-2 kez gözden geçirilir (tarih, veri, ekran görüntüsü tazele); güncelleme `lastModified`'a yansır.

## Ölçüm

- Kurulum: Google Search Console (domain doğrulama + sitemap gönderimi) ilk gün yapılır; analytics aracı proje tercihi.
- Takip edilen: tıklama/gösterim (GSC), sorgu bazlı pozisyon, sayfa bazlı dönüşüm. Sıralama 3-6 ay sabır işidir; ilk sinyal "gösterim artışı"dır.

## Çıktı formatı

- Teknik iş: dosya yollarıyla kod (`app/sitemap.ts`, `generateMetadata` örneği, JSON-LD bloğu).
- İçerik işi: plan tablosu veya sayfa taslağı (H yapısı + her bölümün 1 cümle özeti); tam metin istenirse `copywriting` kurallarıyla yazılır.
- Sonda kısa kontrol notu: GSC doğrulama, sitemap gönderimi, canonical tutarlılığı.

## On-page SEO denetimi

Mevcut sayfa/site denetiminde standart kontrol listesi — denetim çıktısı "sorun → etki → düzeltme" formatında verilir, genel geçer tavsiye listesi değil:

1. **Title/description**: her sayfada benzersiz, hedef sorguyu taşıyan title (50-60 karakter) ve tıklama gerekçesi veren description (140-160 karakter); şablon tekrarı işaretlenir.
2. **Başlık hiyerarşisi**: tek H1, mantıklı H2/H3 ağacı; başlıkta anahtar kelime doğal kullanım (yığma yasak).
3. **İndekslenebilirlik**: robots/noindex kazaları, canonical doğruluğu, yanlışlıkla client-only render edilen kritik içerik (GEO kuralıyla birlikte).
4. **İç bağlantı**: yetim sayfalar, kırık linkler, anchor metinlerin açıklayıcılığı ("tıklayın" yasak).
5. **Görsel**: alt metinler, dosya adları, boyut/format (WebP), lazy loading — LCP etkisiyle birlikte raporlanır.
6. **Core Web Vitals**: LCP/CLS/INP ölçümü gerçek veriden (CrUX/Search Console); tahminle "hızlı sayılır" denmez.
7. **Yapısal veri**: mevcut schema'ların doğrulaması + sayfa tipine göre eksik fırsatlar.
Denetim önceliklendirilir: en yüksek trafik/potansiyel sayfadan başlanır, her şeyi aynı anda düzeltme planı yazılmaz.

## Pillar / hub-cluster otorite yapısı

Konu otoritesi için içerik mimarisi — tekil blog yazıları yerine küme stratejisi:

- **Pillar (hub) sayfa**: geniş konuyu kapsayan, sorgu hacmi yüksek ana rehber ("Okul CRM nedir ve nasıl seçilir"); her cluster yazısına bağlanır.
- **Cluster yazıları**: pillar'ın alt sorularını derinlemesine cevaplar (uzun kuyruk sorgular); her biri pillar'a geri bağlanır ve ilgili kardeşlere yatay bağlanır.
- **Kural**: pillar'sız cluster yazılmaz, cluster'sız pillar şişirilmez — küme planı içerik üretiminden ÖNCE tablo olarak çıkarılır (pillar | cluster başlıkları | hedef sorgu | durum).
- Programatik sayfa ağları (programmatic-seo skill'i) bu yapıya hub üzerinden bağlanır; iki sistem ayrı adacık olmaz.
- Küme tamamlandıkça performans izlenir: kümenin toplam görünürlüğü tekil yazı metriğinden önce gelir.
