---
name: ad-campaign-management
description: Meta reklam kampanyalarında strateji ve harcama güvenliği standartları — kampanya hedefi seçimi, bütçe/teklif stratejisi, hedefleme, performans metrik yorumu, harcama güvenlik sınırları. Kullanıcı "reklam kampanyası", "bütçe belirle", "hedefleme yap", "ROAS/CPA hesapla", "reklam performansı", "kampanya optimize et", "harcama limiti" gibi ifadeler kullandığında MUTLAKA bu skill'i kullan. API bağlantısı/teknik entegrasyon için meta-integration skill'i geçerlidir — bu skill "ne zaman ne kadar bütçeyle ne yapılır" kararını taşır, API çağrısının kendisini değil.
---

# Ad Campaign Management

Meta reklamlarında kampanya stratejisi ve — en kritik olarak — **harcama güvenliği** standartları. Bu skill gerçek parayla ilgilidir; approval-workflow'un content için uyguladığı "AI önce draft üretir" kuralı burada daha da katıdır, çünkü yayından geri alınabilen içeriğin aksine harcanan bütçe geri alınamaz.

## 1. Kampanya hiyerarşisi ve hedef seçimi

- Campaign seviyesinde tek bir hedef (objective) seçilir: farkındalık, trafik, etkileşim, lead, dönüşüm — hedef karışık kampanyalar (hem farkındalık hem dönüşüm) optimizasyonu bozar, ayrı campaign aç.
- AdSet seviyesinde bütçe + hedefleme + teklif; Ad seviyesinde yaratıcı (görsel/video/caption, media-content skill'inin çıktısı).
- Yeni sektör/ürün için önce küçük test bütçesiyle (öğrenme aşaması) 2-3 varyant dene, sonuca göre büyüt — büyük bütçeyle tek varyanta başlama.

## 2. Bütçe ve teklif stratejisi

- **Campaign Budget Optimization (CBO)** vs **AdSet bazlı bütçe**: çok adset'li kampanyada CBO otomatik dağıtır ama kontrolü azaltır; az sayıda, birbirinden çok farklı adset varsa manuel adset bütçesi tercih edilir.
- Teklif stratejisi: `en düşük maliyet` (varsayılan, öğrenme kolay) vs `maliyet tavanı` (CPA hedefi netse). Yeni kampanyada varsayılanla başla, veri biriktikçe sıkılaştır.
- **Öğrenme aşaması** (learning phase) her önemli değişiklikte sıfırlanır (bütçe/hedefleme/yaratıcı büyük değişikliği) — sık müdahale performansı düşürür; değişiklikleri toparlayıp haftalık uygula, günlük ince ayar yapma.

## 3. Hedefleme temelleri

- Sıralama: geniş hedefleme (algoritmaya bırak) → ilgi alanı bazlı → retargeting (siteyi ziyaret eden/etkileşim kuran) → lookalike (mevcut müşteri bazlı). Küçük bütçede aşırı daraltılmış hedefleme (birden fazla ilgi alanı kesişimi) yeterli gösterim almaz.
- Retargeting/lookalike için kaynak veri (piksel, müşteri listesi) gerekir — bu veri toplama security-baseline'daki kişisel veri kurallarına tabi, KVKK/GDPR açık rıza şartı unutulmaz.
- Reklam politika sınırları (sağlık, finans, hassas kategori hedefleme kısıtları) proje sektörüne göre CLAUDE.md'de not düşülür — özellikle finansal veya sağlık ilişkili içerikte önceden kontrol edilir.

## 4. Yaratıcı rotasyonu

- Aynı yaratıcı 1-2 haftadan uzun aynı audience'a gösterilirse "ad fatigue" (frekans artar, CTR düşer) — media-content skill'i ile üretilen varyantlar rotasyona sokulur.
- Her adset'te en az 2-3 yaratıcı varyantı aynı anda test edilir (dinamik kreatif veya manuel A/B); tek yaratıcıyla uzun süre kampanya koşturma.

## 5. Performans metrik yorumu

Hedefe göre öncelikli metrik değişir; hepsine aynı anda odaklanma:

| Kampanya Hedefi | Öncelikli Metrik | Yardımcı Metrik |
|---|---|---|
| Farkındalık | CPM, Reach | Frekans (3'ü geçmesin) |
| Trafik | CPC, CTR | Bağlantı tıklama oranı |
| Dönüşüm | CPA, ROAS | Dönüşüm oranı |

- ROAS/CPA hesaplarken **atfetme penceresini** (attribution window) sabit tut ve raporlarda belirt — pencere değişimi (7 gün tık vs 1 gün görüntüleme) sonucu yapay şekilde değiştirir.
- Tek günlük veriyle karar verme; en az 3-4 günlük (öğrenme aşaması bitmiş) veriye bak.

## 6. Harcama güvenlik sınırları (kritik)

Bu bölüm approval-workflow ile doğrudan bağlanır — reklam bütçesinde AI'ın yetkisi content'ten daha dar olmalı:

- **Sunucu tarafında zorlanan sabit tavan**: hesap/kampanya bazlı günlük ve aylık harcama tavanı Meta'nın kendi bütçe alanına VE uygulama tarafında ayrı bir kontrol katmanına yazılır — tek katmana güvenme (Meta tarafı arayüz hatası/gecikmesiyle aşılabilir).
- **Otomatik durdurma eşiği**: harcama, beklenen günlük tavanın belirlenen bir oranını (ör. %120) geçerse kampanya API üzerinden otomatik `PAUSED` durumuna alınır ve Telegram'a acil bildirim gider (telegram-bot deseni, `❌` seviyeli).
- **Her artış insan onayından geçer**: bütçe artışı, yeni kampanya açma, hedefleme genişletme — approval-workflow'daki durum makinesiyle aynı mantık: öneri `draft` olarak doğar, insan onaylamadan Marketing API'ye yazma isteği gitmez. Bu, AI ajanının (ads-analyst) sadece **öneri** üretip asla **uygulama** yapmamasıyla mimari olarak sağlanır (bkz. agent-orchestration'daki en az yetki ilkesi).
- **Anomali tespiti**: CPA/CPM aniden anlamsız değere sıçrarsa (ör. 10 kat artış) — bu genelde hesap askıya alma, ödeme sorunu veya izleme (pixel) kopması işaretidir; harcamayı durdurmaktan önce insan bilgilendirilir, otomatik "daha fazla harca" kararı asla verilmez.

## 7. Raporlama

- Günlük özet (approval-workflow'un digest deseniyle aynı prensip): harcama, temel metrikler, anomali varsa üstte. Her kampanya değişikliği için ayrı mesaj spam'ine düşme.
- Haftalık strateji notu: hangi yaratıcı/adset performans gösterdi, hangi durduruldu, önerilen bir sonraki adım — karar insanda kalır, bu sadece özet.

## Çıktı formatı

- Kampanya kurulum önerisi: hedef + bütçe önerisi (aralık, kesin sayı değil — kullanıcı sektörüne göre CLAUDE.md'den referans alınmadıkça) + hedefleme taslağı + yaratıcı sayısı önerisi.
- Performans analizi: tablo (metrik, değer, önceki döneme göre değişim) + 2-3 cümlelik yorum + varsa önerilen aksiyon (uygulama değil, öneri).
- Harcama güvenliği kurulum işi: tavan değerleri + hangi katmanda zorlandığı + otomatik durdurma eşiği + bildirim kanalı net şekilde listelenir.
