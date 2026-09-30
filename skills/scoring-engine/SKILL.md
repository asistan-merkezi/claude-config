---
name: scoring-engine
description: Özellik (feature) bazlı skorlama/tahmin motoru tasarım standartları. Kullanıcı bir skorlama sistemi, puanlama motoru, tahmin modeli, feature mühendisliği, ağırlıklandırma, backtest veya "en iyi adayı sırala" tipi bir sistem istediğinde MUTLAKA bu skill'i kullan. "Skor hesapla", "puanlama yap", "feature ekle", "ağırlıkları ayarla", "backtest çalıştır", "tahmin motoru" gibi ifadeler geçtiğinde de kullan. Ölçülebilir, backtest'le doğrulanan, açıklanabilir skorlama motorları üretir.
---

# Scoring Engine

Özellik bazlı skorlama/sıralama motorları (adayları puanla → sırala → öner) için tasarım standartları. Projeye özel detaylar (domain, feature listesi, veri tabloları) CLAUDE.md'dedir — bu skill evrensel yöntemi taşır.

## Temel felsefe

1. **Deterministik çekirdek**: Skor hesabı saf koddur — aynı girdi her zaman aynı skoru verir. LLM skor HESAPLAMAZ; LLM'in rolü varsa hazır skorları yorumlamaktır (bkz. `ai-report` skill).
2. **Açıklanabilirlik**: Her toplam skor, bileşen feature skorlarına ayrıştırılabilir olmalı. "Neden bu aday önde?" sorusunun cevabı her zaman verilebilmeli.
3. **Kanıt olmadan feature yok**: Bir feature'ın motorda kalma hakkı backtest'te gösterdiği katkıdır, sezgi değil. Katkısı ölçülemeyen feature eklenmez; ölçülüp katkısız çıkan çıkarılır.

## Mimari

Standart dosya ayrımı (Python varsayılan):

```
engine/
  ozellikler.py   # feature hesaplama — her feature ayrı fonksiyon
  skorla.py       # ağırlıklandırma + toplam skor + sıralama
  config.py       # ağırlıklar ve eşikler TEK yerde (veya yaml/json)
backtest/
  backtest.py     # geçmiş veriyle değerlendirme
```

- **Feature fonksiyonu sözleşmesi**: girdi = aday + bağlam verisi, çıktı = normalize skor (0-1 veya 0-100, projede TEK ölçek seç ve hiç sapma). Veri eksikse `None` döner — sahte nötr değer (0.5 gibi) üretme; eksiklik ağırlıklandırmada açıkça ele alınır (feature atlanır, kalan ağırlıklar yeniden normalize edilir) — ama bu sessiz yapılmaz: aşağıdaki `veri_kapsama` ile mutlaka raporlanır.
- **Ağırlıklar config'te**: kod içine gömülü sayı YOK. Config sürümlenir (`v1`, `v1_5` gibi) — hangi sürümle hangi backtest sonucunun alındığı izlenebilir olmalı.
- Toplam skor varsayılanı ağırlıklı ortalama; feature'lar arası etkileşim gerekiyorsa (X yalnızca Y varken anlamlı) bunu ayrı bir bileşik feature olarak yaz, formülü karmaşıklaştırma.

## Eksik veri: cezalandırma değil, raporlama

Eksik veriyi renormalizasyonla sessizce yutmak riski gizler; skora keyfi ceza puanı eklemek ise ölçülmemiş bir feature'ı arka kapıdan sokar. Doğru yol eksikliği ÖLÇÜP dışarı vermektir.

- Motor her aday için skorun yanında **`veri_kapsama`** döndürür: hesaplanabilen feature'ların ağırlık toplamı / tüm feature'ların ağırlık toplamı (0-1).
- **Skora ceza puanı (−2, −3 gibi) EKLENMEZ.** Backtest'le doğrulanmamış ceza da bir feature'dır ve "kanıt olmadan feature yok" ilkesini ihlal eder; üstelik skoru bozduğu için sıralamanın açıklanabilirliğini de düşürür.
- Kapsama eşiğinin (varsayılan 0.7) altındaki adaylar bir **bayrak** taşır. Bayrağın karar anlamını domain skill'i / CLAUDE.md tanımlar (ör. "banko adayı olamaz", "yoruma zorunlu belirsizlik notu eklenir").
- Kapsama aynı zamanda bir veri kalitesi ölçüsüdür: bir feature adayların büyük kısmında `None` kalıyorsa önce veri sorunu çözülür.

## Feature tasarımı

- Her feature TEK hipotezi kodlar ve adı hipotezi söyler (`form_skor`, `zemin_uyum` gibi). "Karışık sinyal" feature'ı yazma.
- Normalizasyon yöntemini feature içinde belgele (min-max mı, persentil mi, kategori eşlemesi mi) — yorum satırıyla.
- Sızıntı (leakage) kontrolü: feature, tahmin ANINDA bilinemeyecek veri kullanamaz (sonuç sonrası oluşan alanlar, gelecek tarihli kayıtlar). Her yeni feature'da açıkça sor: "Bu bilgi karar anında elimde olur muydu?" Yalnızca belirli zamanda oluşan veriler (ör. gün içinde açıklanan oranlar) varsa, backtest bunu zaman-uyumlu kullanmalı.

## Backtest standartları

- **Baseline zorunlu**: Motor her zaman naif bir referansla karşılaştırılır (rastgele seçim, popülerlik/favori sırası gibi). "Skor %X isabetli" tek başına anlamsız; "baseline %Y'ye karşı %X" anlamlı.
- **Zaman bazlı ayırma**: eğitim/ayar dönemi ile test dönemi zamanda ayrılır (walk-forward). Ağırlıklar test dönemine bakılarak AYARLANMAZ — test verisiyle ayar yapıldıysa o test yanmıştır, yeni dönem gerekir.
- **Metrik seçimi domain'e göre**: sıralama problemi ise isabet@k (ilk k tahminde doğru var mı), ikili sonuç ise precision/recall, getiri problemi ise birim başına net getiri. Metrik CLAUDE.md'de tanımlanır ve sürümler arası SABİT tutulur — metrik değişirse eski sonuçlarla karşılaştırma yapılamaz.
- **Sıralama motorları için asgari metrik seti** — tek başına isabet@1 yanıltıcıdır (şanslı bir dönem başarı gibi görünür):
  1. **İsabet@k** (@1, @3 ve domain'in tabela/plase karşılığı) — sıralama kalitesi.
  2. **Simüle ROI** — getiri hedefi olan domain'lerde zorunlu. Yalnızca GERÇEK ödeme/fiyat verisi veritabanında varsa hesaplanır; tahmini ödemeyle ROI üretmek uydurmadır. Birim maliyet ve strateji kuralları (hangi strateji kuralı simüle edildi; örn. kupon/bahis mantığı) açıkça yazılır; küçük örneklemde sonuç "gösterge" etiketlenir.
  3. **Kalibrasyon (Brier / log-loss)** — YALNIZCA motor skor→olasılık dönüşümü tanımlıysa. Normalize edilmiş 0-100 skor olasılık İDDİASI taşımaz; olasılık olmayan bir sayının kalibrasyonunu ölçmek yanıltıcı bir kesinlik hissi üretir.
  - Metrik seti genişletilirse eski sonuçlar ya yeni metrikle geriye dönük hesaplanır ya da "yalnızca eski metrik setiyle kıyaslanabilir" diye etiketlenir. Metrik sabitliği kuralı bu şekilde korunur.
- **Dürüst raporlama**: örneklem büyüklüğü her sonuçta belirtilir; küçük örneklemde (ör. <100 olay) sonuç "gösterge" diye etiketlenir, zafer ilan edilmez. Başarısız hipotez de raporlanır ve kayda geçer — başarısızlık bilgidir, silinmez.
- Sürüm karşılaştırması: yeni feature/ağırlık seti (`v2`) eski setle (`v1`) AYNI dönem ve AYNI metrikle yan yana koşulur; tek tablo halinde raporlanır.

## Sürüm ve deney disiplini

- Her deney kaydı: config sürümü + veri aralığı + metrik sonuçları + tarih. Basit bir `DENEYLER.md` dosyası yeterli; hafızaya güvenme.
- Aynı anda TEK değişken değiştir: hem yeni feature hem yeni ağırlık aynı deneyde denenmez — hangisinin etki ettiği bilinemez.
- Prod'a alma eşiği: yeni sürüm, baseline'ı VE mevcut sürümü anlamlı farkla geçmeli; berabere ise basit olan kazanır.

## Yaşayan model yönetimi (şampiyon/rakip döngüsü)

Motor canlıya çıktıktan sonra ağırlıkların bakımı — tek seferlik kalibrasyonla bırakılmaz, kontrollü bir döngüyle yaşatılır.

- **Config kaynağı canlıda tablodur**: Aktif ağırlıklar dosyadan değil, versiyonlu bir config tablosundan okunur. Her kayıt durum taşır: `aday / aktif / emekli / geri_alindi`; tip başına aynı anda TEK aktif config olur. Repo'daki config dosyaları yalnızca ilk yükleme ve deney içindir ve dosyada bu açıkça etiketlenir ("canlı kaynak değildir").
- **Periyodik yeniden kalibrasyon**: Kayan pencereyle (son N dönem) yeniden ayar; değerlendirme her zaman ayara hiç girmemiş dokunulmamış test setinde yapılır. Şampiyon (aktif config) ile rakip (yeni aday) AYNI test setinde yan yana koşulur — farklı dönemlerde ölçülüp kıyaslanmaz.
- **Terfi kuralları (otonom sistemde hepsi birden sağlanmalı)**: (1) ana metriklerde şampiyona karşı galibiyet, (2) çoklu alt-dönem doğrulaması (tek şanslı dönem değil, dilimlerin çoğunda üstünlük), (3) asgari örneklem eşiği (altındaysa hüküm yok, terfi yok), (4) **ağırlık savrulma freni**: tek terfide hiçbir feature ağırlığı kendi mevcut değerinin %25'inden fazla değişemez ve feature'ların göreli önem sıralaması korunur. Daha büyük değişim isteyen aday tek hamlede geçemez; ara config'lerle kademeli terfi eder ve her adımda aynı kapılardan yeniden geçer.
- **Bekçi + otomatik geri alma**: Canlı performans iki referansla sürekli kıyaslanır: terfi anındaki test beklentisi ve piyasa/doğal baseline. Belirgin düşüş ARDIŞIK kontrollerde sürerse önceki aktif config'e otomatik dönüş yapılır (tek kötü kontrol yeterli değildir); küçük örneklemde bekçi hüküm vermez, bekler. Geri alınan config `geri_alindi` durumuna geçer ve bir daha OTOMATİK terfi edemez — ancak insan kararıyla yeniden aday olabilir.
- **İnsanlı / otonom varyant**: Onay kapılı kurulumlarda terfi insan komutuyla olur (approval-workflow deseni); otonom kurulumda kapının yerini yukarıdaki kural seti alır. İkisi de meşrudur — seçim projeye aittir ve CLAUDE.md'de yazar.
- **Denetlenebilirlik**: Her terfi ve geri alma, karar gerekçesiyle loglanır (hangi metrikler, hangi test seti, hangi eşikler sağlandı) — "neden bu config aktif?" sorusunun cevabı her an sorgulanabilir olmalı.

## Sık hatalar

- Kalibre edilmemiş ağırlıklarla özellik katkısı ölçmeye çalışmak: bir feature'ın ağırlığı ~0 iken katkısı da 0 görünür — bundan "özellik işe yaramıyor" sonucu çıkmaz. Katkı hükmü ancak ağırlık kalibrasyonundan SONRA verilir.
- Eksik veriyi keyfi ceza puanıyla cezalandırmak: ölçülmemiş ceza, gizli ve doğrulanmamış bir feature'dır. Eksiklik `veri_kapsama` ile raporlanır.
- Olasılık iddiası taşımayan bir skorda kalibrasyon metriği (Brier/log-loss) raporlamak.
- Gerçek ödeme/fiyat verisi yokken tahmini rakamlarla ROI üretmek.
- Metrik setini sürüm ortasında değiştirip yeni sonuçları eskilerle aynı tabloda yan yana koymak.

## Çıktı formatı

- Yeni feature: hipotez (1 cümle) → hesaplama kodu → normalizasyon notu → backtest planı (hangi dönem, hangi metrik).
- Backtest sonucu: karşılaştırma tablosu (sürümler × metrikler, örneklem büyüklüğü ile) + 2-3 cümle yorum + net karar önerisi (al / alma / veri yetersiz).
- Kod dosya yollarıyla verilir; config değişiklikleri ayrı ve açık gösterilir.
