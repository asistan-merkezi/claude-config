---
name: lbo-model
description: Kaldıraçlı satın alma (LBO) ve şirket satın alma matematiği standartları — borç yapısı, kaynak-kullanım tablosu, getiri hesabı (IRR/MOIC), çıkış senaryoları. Kullanıcı LBO modeli, kaldıraçlı satın alma, şirket satın alma analizi, "bu şirketi almaya değer mi", borçla satın alma matematiği veya çıkış getirisi hesabı istediğinde MUTLAKA bu skill'i kullan. "LBO kur", "satın alma modeli", "IRR hesapla", "MOIC", "kaç yılda çıkarım", "borç kapasitesi" gibi ifadeler geçtiğinde de kullan. Excel üretimi Anthropic xlsx skill'ine, veri çekimi financial-research'e tabidir — bu skill modelin matematiğini ve varsayım disiplinini taşır. Yatırım tavsiyesi değildir çizgisi her çıktıda korunur.
---

# LBO Model (Kaldıraçlı Satın Alma Matematiği)

Şirket satın alma değerlendirmelerinde LBO model kurma standartları. Canlı finansal veri financial-research'e, Excel dosya tekniği xlsx skill'ine aittir. Her çıktı analiz/eğitim amaçlıdır; yatırım tavsiyesi dili kurulmaz.

## Temel ilkeler

1. **Kaynak-kullanım eşitliği**: Model, Sources & Uses tablosuyla başlar: kullanım (satın alma bedeli + işlem maliyetleri + refinanse borç) = kaynak (özkaynak + borç dilimleri). Eşitlenmeyen tablo model hatasıdır.
2. **Varsayımlar tek sayfada**: giriş çarpanı, borç/FAVÖK oranı, faiz, büyüme, marj, çıkış çarpanı — hepsi tek varsayım bloğunda, her biri gerekçeli. Çıkış çarpanı varsayılan olarak giriş çarpanına eşit alınır; çarpan genişlemesine dayanan getiri ayrıca ve dürüstçe etiketlenir ("getirinin şu kadarı çarpan varsayımından geliyor").
3. **Getiri ayrıştırması zorunlu**: IRR/MOIC tek sayı olarak değil, kaynaklarına ayrılarak sunulur: FAVÖK büyümesi + borç ödemesi (deleveraging) + çarpan değişimi. Getirinin nereden geldiği görünmeden model tamamlanmış sayılmaz.
4. **Senaryo üçlüsü**: business-case ile aynı disiplin — kötümser/baz/iyimser + kritik varsayım duyarlılık tablosu (çıkış çarpanı × büyüme matrisi).

## Model yapısı

- **İşletme projeksiyonu**: gelir → FAVÖK → vergi → yatırım harcaması → işletme sermayesi değişimi → borç servisi öncesi serbest nakit akışı. 5 yıl varsayılan ufuk.
- **Borç şelalesi**: dilim bazında (banka kredisi, mezzanine vb.) faiz + zorunlu anapara + nakit süpürme (cash sweep) sırası; yıl sonu borç bakiyeleri izlenir.
- **Çıkış**: yıl bazında çıkış senaryosu (3./5./7. yıl) → çıkış değeri − kalan borç = özkaynak değeri → IRR ve MOIC.
- Küçük ölçekli (KOBİ) satın almalarda gerçekçilik: satıcı finansmanı, kazanç bağlı ödeme (earn-out) ve kişisel kefalet gibi Türkiye pratiğinde yaygın yapılar modele eklenebilir olmalı.

## Dürüstlük kuralları

- Borç kapasitesi hedef getiriye göre değil, nakit akışının taşıyabildiğine göre belirlenir (faiz karşılama ve borç servis karşılama oranları eşikleri görünür).
- Stres testi zorunlu: gelir %20 düşerse borç servisi döner mü — dönmüyorsa model bu kırılganlığı ilk sayfada söyler.
- Veri kaynağı belirsizse hesap yapılmaz; "yaklaşık FAVÖK" üzerine kurulu modelde bu belirsizlik sonuç aralığına yansıtılır.

## TL ve enflasyon

- TL bazlı modelde yüksek faiz ve enflasyon açıkça modellenir: nominal mi reel mi çalışıldığı varsayım bloğunda yazılır; FAVÖK büyümesi ile faiz aynı (nominal) bazda olmalı — reel büyümeyi nominal faizle kıyaslamak yasak.
- Döviz borcu varsa kur riski ayrı senaryodur. Faiz giderlerinin vergi matrahından indirilebilirliği (finansman gider kısıtlaması gibi düzenlemeler) mali müşavirle doğrulanır; model bu varsayımı görünür tutar.

## Çıktı formatı

Sources & Uses tablosu + varsayım bloğu + yıllık projeksiyon ve borç şelalesi + senaryo/duyarlılık tablosu + getiri ayrıştırması (FAVÖK büyümesi / borç ödemesi / çarpan) + stres testi sonucu. Excel üretimi xlsx skill'ine devredilir.
