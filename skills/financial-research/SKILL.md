---
name: financial-research
description: MCP finansal veri connector'larıyla (Alpha Vantage, FMP) hisse/portföy araştırması, finansal model ve kazanç çağrısı analizi üretim standartları. Kullanıcı bir hisse/portföy özeti, analist notu derlemesi, DCF/finansal model, Excel finansal tablo, kazanç çağrısı (earnings call) analizi, beklenti-gerçekleşen karşılaştırması veya piyasa haber-duygu özeti istediğinde MUTLAKA bu skill'i kullan. "Hisseyi analiz et", "finansal model kur", "kazanç çağrısını incele", "DCF çıkar", "analist hedefleri", "portföy özeti" gibi ifadeler geçtiğinde de kullan. Deterministik veri çekimi + kontrollü LLM yorumu ile üç iş akışı üretir: Market Researcher, Model Builder, Earnings Reviewer.
---

# Financial Research

MCP connector'larıyla (Alpha Vantage, FMP) finansal araştırma iş akışları. Üç mod: Market Researcher, Model Builder, Earnings Reviewer.

## Temel ilke: veri deterministik, yorum ayrı katman

- Sayısal veri (fiyat, oran, tahmin, tablo) **her zaman connector'dan** gelir — LLM'e "X hissesinin fiyatı nedir" diye sorulmaz, tool'dan çekilir.
- LLM yalnızca **yorum katmanıdır**: çekilen veriyi özetler, bağlamlandırır, çelişkileri işaret eder. Rakam üretmez, rakamı yorumlar. (ai-report skill'inin finansal özelleşmesi.)
- Her çıktı **"analiz, yatırım tavsiyesi değildir"** notuyla biter. Analytics-only duruşu (tavsiye değil, analiz) burada da geçerli — kişiselleştirilmiş al/sat önerisi üretilmez.

## Connector uçları (hangi veri nereden)

| İhtiyaç | Birincil | Alternatif |
|---|---|---|
| Anlık fiyat/oran | FMP `quote`, `company` | Alpha Vantage `GLOBAL_QUOTE`, `COMPANY_OVERVIEW` |
| Haber + duygu | Alpha Vantage `NEWS_SENTIMENT` | FMP `news` |
| Analist derece/hedef | FMP `analyst` | — |
| Finansal tablolar | FMP `statements` | Alpha Vantage `INCOME_STATEMENT` vb. |
| DCF değerleme | FMP `discountedCashFlow` | (elle model) |
| Kazanç transkripti | Alpha Vantage `EARNINGS_CALL_TRANSCRIPT` | FMP `earningsTranscript` |
| Tahmin (EPS/gelir) | FMP `analyst` estimates | Alpha Vantage `EARNINGS_ESTIMATES` |

**Plan uyarısı:** FMP'nin bazı uçları (transcript, DCF, estimates, form13F) ücretli pakete bağlıdır. Uç 402/403 dönerse: kullanıcıya net söyle ("bu veri mevcut planında yok"), uydurma; mümkünse ücretsiz alternatife düş (ör. transcript için Alpha Vantage).

## Mod 1 — Market Researcher (hisse/portföy özeti)

Amaç: bir sembol veya portföy için tek sayfalık durum özeti.

Akış:
1. `quote` + `company` → fiyat, piyasa değeri, temel oranlar (P/E, P/B, marjlar).
2. `NEWS_SENTIMENT` → son haberler + toplu duygu skoru; başlıkları tarih damgasıyla derle.
3. `analyst` → derecelendirme dağılımı (alım/tut/sat) + ortalama fiyat hedefi + mevcut fiyata göre yukarı/aşağı potansiyel.
4. LLM sentezi: üç kaynağı birleştir — "veriler ne diyor" (nötr, sayısal) + "dikkat çeken noktalar" (çelişki, aykırılık). Örnek çelişki işareti: analistler pozitif ama haber duygusu negatifse bunu vurgula.
5. Portföyse: her sembol için özet + portföy düzeyinde toplulaştırma (ağırlıklı getiri, sektör dağılımı).

Çıktı: başlık + rakam bloğu + haber özeti + analist bloğu + "dikkat çekenler" + analytics-only notu.

## Mod 2 — Model Builder (Excel finansal model)

Amaç: gerçek verilerle beslenmiş, senaryo satırlı finansal model. **xlsx skill'i ile birlikte çalışır** (formül, formatlama, çoklu sayfa standartları oradan gelir).

Akış:
1. `statements` → son 3-5 yıl gelir tablosu, bilanço, nakit akışı (ham veri ayrı sayfaya, kaynak olarak).
2. Model sayfası: geçmiş veriden büyüme oranları türet, tahmin yıllarını **formülle** kur (elle sayı yazma — varsayım hücreleri ayrı, formüller onlara referans versin).
3. `discountedCashFlow` varsa değerleme sayfası; yoksa FCF projeksiyonundan elle DCF (WACC ve terminal büyüme **varsayım hücresi** olarak, kullanıcı oynayabilsin).
4. Senaryo satırları: baz / iyimser / kötümser — varsayım hücrelerini değiştiren tek panel.
5. Her sayfaya veri kaynağı + çekim tarihi dipnotu.

Kural: **varsayımlar hardcode edilmez**, hepsi işaretli girdi hücrelerinde; formüller onlara bağlanır. Kullanıcı bir varsayımı değiştirince tüm model güncellenmeli.

## Mod 3 — Earnings Reviewer (kazanç çağrısı analizi)

Amaç: bir çeyrek kazanç çağrısının yapılandırılmış analizi.

Akış:
1. `EARNINGS_CALL_TRANSCRIPT` → ilgili çeyrek transkripti. Yoksa plan uyarısı + alternatif dene.
2. `analyst` estimates + `statements` → beklenti-gerçekleşen: EPS ve gelir için tahmin vs açıklanan, sapma yüzdesi.
3. Transkript analizi (youtube-analysis'in "hype ayıklama" mantığı): 
   - Somut açıklamalar (rakamlı, rehberlik içeren) vs genel/pazarlama dili ayrımı.
   - Rehberlik (guidance): yönetim gelecek için ne dedi, önceki çeyreğe göre yukarı mı aşağı mı.
   - Ton değişimi: dikkat/temkin sinyalleri, tekrarlanan temalar.
4. Transkript sadakati: yalnızca söylenenle çalış, damga/alıntı göster (kısa), yorumla iddiayı ayır ("yönetim X olduğunu söyledi" ≠ "X'tir").

Çıktı: beklenti-gerçekleşen tablosu + rehberlik özeti + somut/genel ayrımı + ton notu + analytics-only notu.

## Güvenlik ve hijyen

- Connector çağrıları security-baseline'ın LLM/dış servis kurallarına tabidir: yanıt Zod/şema ile doğrulanır, ham API alanları uygulamaya sızmaz (api-integration sınır dönüşümü).
- Rate limit ve maliyet: connector'ların kendi limitleri var; toplu portföy taramasında sembol başına ayrı çağrı yapılır ama makul batch'lerde, gereksiz tekrar çağrı yok.
- Veri çekim tarihi her çıktıda görünür — finansal veri hızlı eskir, "ne zamanki veri" belirsiz kalmaz.

## Projede uygulama (ürün modülü)

- Bu skill connector tabanlı hızlı araştırma içindir. Kalıcı ürün özelliğine dönüşecekse (ör. bir ürüne hisse modülü): veri çekimi Python scraper + Supabase katmanına taşınır (data-pipeline skill'i), LLM yorumu ai-report deseniyle cron'a bağlanır. Connector doğrudan production akışına konmaz — geliştirme/araştırma aracıdır.
