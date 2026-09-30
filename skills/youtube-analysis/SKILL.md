---
name: youtube-analysis
description: YouTube videolarını altyazı (transcript) üzerinden analiz etme standartları. Kullanıcı bir YouTube videosunun özetini, analizini, içindeki bilgilerin çıkarımını, rakip video incelemesini veya video hakkında soru-cevap istediğinde MUTLAKA bu skill'i kullan. "Bu videoyu özetle", "videoda ne anlatıyor", "şu YouTube linkine bak", "rakip videoyu analiz et", "videodaki adımları çıkar" gibi ifadeler geçtiğinde de kullan. API key gerektirmeden, zaman damgalı, transkripte sadık analizler üretir. Not: açık ağ erişimi gerektirir (Claude Code/Codespaces ortamı).
---

# YouTube Analysis

YouTube videolarının altyazı bazlı analizi (API key'siz).

## 1. Transcript alma

- Araç: `youtube-transcript-api` (Python, API key gerektirmez):

```bash
pip install youtube-transcript-api
```

```python
from youtube_transcript_api import YouTubeTranscriptApi
# Öncelik: manuel Türkçe > otomatik Türkçe > manuel İngilizce > otomatik İngilizce
transcript = YouTubeTranscriptApi().fetch(video_id, languages=['tr', 'en'])
```

- Video ID, URL'nin `v=` parametresinden veya `youtu.be/` yolundan çıkarılır.
- Altyazı tamamen kapalıysa: kullanıcıya bildir; **Whisper fallback yalnızca kullanıcı açıkça isterse** denenir (ses indirme + yerel transkripsiyon maliyetli, otomatik yapılmaz).

## 2. Transkript sadakati kuralları

- Analiz **yalnızca transkriptte olanla** yapılır; videoda söylenmemiş bilgi eklenmez, boşluklar genel bilgiyle doldurulmaz.
- Emin olunamayan yer: "transkriptte net değil" diye işaretlenir, tahmin edilmez.
- Otomatik altyazıysa başta belirtilir: "otomatik altyazıdan çalışıyorum, teknik terimlerde hata olabilir".
- Konuşmacının iddiası ile doğrulanmış bilgi ayrılır: "X olduğunu iddia ediyor" ≠ "X'tir".

## 3. Zaman damgalı çıktı

- Tüm önemli noktalar `[mm:ss]` damgasıyla verilir; kullanıcı videoda o ana atlayabilmelidir.
- Uzun videolarda bölüm yapısı: transkriptteki doğal konu geçişlerine göre bölümlenir, her bölüm damgayla başlar.

## 4. Beş analiz modu

Kullanıcının niyetine göre mod seçilir (belirsizse sor):

**a) Özet** — Bölüm bölüm, zaman damgalı yapılandırılmış özet. Ana tez + destekleyen noktalar + sonuç.

**b) Hype ayıklama** — Pazarlama/abartı dili ile somut bilgiyi ayırır. Çıktı iki liste: "Somut iddialar (kanıt/demo gösterilen)" ve "Desteksiz iddialar/abartı". Ürün tanıtımı ve "X her şeyi değiştirecek" tarzı videolar için.

**c) Rakip video analizi** — İçerik üreticisi gözüyle: hook (ilk 30 sn ne yapıyor), yapı/tempo, kullanılan formatlar, CTA'lar, hedef kitle sinyalleri, alınabilecek dersler. kendi içerik stratejisini beslemek için.

**d) Soru-cevap** — Kullanıcının videoya dair sorularını transkriptten damga göstererek yanıtlar. Cevap transkriptte yoksa "videoda buna değinilmiyor" der.

**e) Uygulanabilir notlar** — Eğitim/tutorial videosundan adım adım uygulama notu çıkarır: komutlar, ayarlar, sıra, dikkat noktaları — takip edilebilir checklist formatında.

## 5. Ortam notu

- Bu skill **açık ağ erişimi** gerektirir → Claude Code / Codespaces ortamı için tasarlanmıştır.
- claude.ai ortamında ağ kısıtlıysa transcript çekilemez; bu durumda kullanıcıdan transkripti yapıştırmasını iste — analiz modları aynen çalışır.
- YouTube, bulut/veri merkezi IP'lerinden (Codespaces, VPS) gelen transcript isteklerini sık sık engeller (`RequestBlocked`/`IpBlocked` hataları); bu durumda komut kullanıcının kendi bilgisayarından çalıştırılır, proxy kullanılır veya transkript yapıştırılır.

## 6. Çoklu video

- Birden fazla video karşılaştırması istenirse: her video ayrı analiz edilir, sonra karşılaştırma tablosu (konu kapsamı, derinlik, güncellik, damgalı referanslar).

## Çıktı formatı

Seçilen mod etiketiyle başlar; ardından zaman damgalı bölümler (`[mm:ss]`), altyazı türü (manuel/otomatik) notu ve "transkriptte net değil" işaretleri. Çoklu videoda karşılaştırma tablosu.
