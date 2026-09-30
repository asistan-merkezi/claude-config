---
name: pitch-deck
description: Yatırımcı ve müşteri sunum destesi (pitch deck) hazırlama standartları — anlatı kurgusu, slayt akışı, metrik seçimi, Türkçe/İngilizce sunum dili. Kullanıcı yatırımcı sunumu, pitch deck, tanıtım destesi, demo day sunumu, müşteri satış sunumu veya "projeyi sunacağım" tipi bir istek geldiğinde MUTLAKA bu skill'i kullan. "Pitch deck hazırla", "yatırımcı sunumu", "deck yap", "sunum akışı", "traction slaytı" gibi ifadeler geçtiğinde de kullan. Dosya üretimi Anthropic pptx skill'ine, görsel dil brand-ui'ye tabidir — bu skill anlatının kurgusunu ve içerik disiplinini taşır.
---

# Pitch Deck (Yatırımcı/Müşteri Sunumu)

Sunum destesinin anlatı ve içerik standartları. PPTX üretim tekniği Anthropic pptx skill'ine, tasarım dili brand-ui'ye aittir.

## Temel ilkeler

1. **Gerçek metrik kuralı**: Deck'e giren her sayı doğrulanabilir kaynaktan gelir (Supabase sorgusu, ödeme kaydı, analitik). Yuvarlama serbest, uydurma ve "hedefi gerçekleşmiş gibi gösterme" yasak. Projeksiyon her zaman "projeksiyon" olarak etiketlenir ve varsayımları görünür.
2. **Slayt başına tek mesaj**: Her slaytın başlığı, slaytın iddiasını tam cümle olarak söyler ("Pazar büyük" değil → "Türkiye'de X segmentinde Y bin işletme bu işi hâlâ Excel'le yapıyor").
3. **10-12 slayt disiplini**: Ana deste kısa; detay isteyen her şey ek (appendix) slaytlarına gider.
4. **Dinleyiciye göre versiyon**: Yatırımcı destesi (büyüme + pazar + ekip) ile müşteri satış destesi (problem + çözüm + ROI + güven) ayrı kurgulardır; tek deste ikisine birden hizmet etmez.

## Standart yatırımcı akışı

problem → çözüm (demo görüntüsüyle) → neden şimdi → pazar boyutu (aşağıdan yukarı hesap: adet × fiyat, TAM/SAM/SOM üçlüsü gösterişi değil hesabın kendisi) → iş modeli → traction (en güçlü gerçek metrik) → rekabet (dürüst konumlandırma tablosu) → ekip → finansal özet + talep (ne kadar, ne için, hangi kilometre taşına).

## Müşteri satış akışı

müşterinin bugünkü acısı (onun diliyle) → maliyeti (zaman/para) → çözüm demoları → benzer müşteri kanıtı → ROI hesabı (business-case skill formatıyla) → fiyat + başlangıç adımı.

## İçerik disiplini

- Metin az: slayt konuşma metni değildir; madde başına 8-10 kelime, konuşulacaklar sunucu notuna.
- Rekabet slaytında "bizde her şey var onlarda yok" tablosu yasak; gerçek güçlü/zayıf eksende dürüst konum.
- Ekip slaytı unvan değil kanıt anlatır ("neden bu problemi biz çözeriz").
- Talep slaytı net: tutar, kullanım planı, hangi metriğe ulaştıracağı. "Görüşmek isteriz" ile biten deste eksiktir.

## Dil (Türkçe / İngilizce)

- Dil, **dinleyiciye** göre seçilir, yazana göre değil: yerli yatırımcı/müşteri → Türkçe; yurt dışı yatırımcı veya uluslararası demo day → İngilizce. Karışık dinleyicide deste İngilizce, sunucu notları Türkçe olabilir.
- İki dil ayrı dosya olarak tutulur (TR/EN), aynı slayt sırası ve aynı rakamlarla; "çeviri" değil **yerelleştirme**: İngilizce versiyonda para TL yanında USD karşılığı (kur tarihiyle), tarih biçimi ve hitap uyarlanır.
- Türkçede çeviri kokan kalıplardan ve İngilizce jargon yığmaktan kaçın (copywriting); sektör terimi dinleyicinin bildiği dilde kalır.

## Metrik seçimi

- Traction slaytına **tek** ana metrik girer, en güçlü gerçek olan: aylık yinelenen gelir, aktif kullanıcı, pilot sayısı veya büyüme oranı. Yanına en fazla 2 destekleyici metrik; metrik duvarı yasak.
- Metrik seçim testi: rakam (1) kaynağı gösterilebilir, (2) zaman aralığı belli, (3) kıyas noktası var (önceki dönem/hedef). Üçünden biri eksikse slayta girmez, sunucu notuna gider.
- Erken aşamada (gelir yok) kanıt: pilot/bekleme listesi/görüşme sayısı ve bunlardan çıkan somut öğrenme; "potansiyel pazar" metrik yerine geçmez.

## Çıktı formatı

Slayt listesi tablosu: no | başlık (iddia cümlesi) | kanıt/metrik ve kaynağı | sunucu notu (2-3 cümle). Varsa ek (appendix) slaytları ayrı bölümde. Dosya üretimi pptx skill'ine devredilir.
