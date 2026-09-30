---
name: data-pipeline
description: Veri toplama ve işleme hattı (scraper → Supabase → zamanlanmış görev) üretim standartları. Kullanıcı web scraper, veri çekme scripti, API'den veri toplama, backfill, zamanlanmış veri güncelleme (cron), idempotent upsert veya veri doğrulama/temizleme istediğinde MUTLAKA bu skill'i kullan. "Scraper yaz", "veri çek", "backfill yap", "cron kur", "otomatik güncelleme", "API'den veri al" gibi ifadeler geçtiğinde de kullan. Python + Supabase için dayanıklı, tekrar çalıştırılabilir veri hatları üretir.
---

# Data Pipeline

Python tabanlı veri toplama hatları (scraper/API → Supabase → zamanlanmış çalıştırma) için üretim standartları. Projeye özel detaylar (kaynak siteler/API'ler, tablo adları, çalışma sıklığı) CLAUDE.md'dedir — bu skill evrensel deseni taşır.

## Mimari varsayılanlar

- Dil: Python 3.11+. Bağımlılıklar `requirements.txt`'de sabitlenmiş sürümlerle.
- Hedef: Supabase (PostgreSQL). Yazma işlemleri `service_role` key ile yapılır (RLS bypass) — key yalnızca env'den okunur, ASLA koda gömülmez, `.env` commit edilmez.
- Zamanlama önceliği: GitHub Actions cron (ücretsiz, log'lu, secrets yönetimi hazır) → Vercel Cron (Next.js route tetiklemeli işler) → yerel cron. Seçimi iş süresine göre yap: 10+ dakika sürebilecek işler GitHub Actions'ta.
- Klasör deseni:

```
scraper/ (veya pipeline/)
  fetch_*.py        # kaynak başına çekme modülü
  parse_*.py        # ayrıştırma/temizleme (fetch'ten ayrı test edilebilir)
  db.py             # Supabase client + upsert yardımcıları (tek yerde)
  main.py           # orkestrasyon + CLI argümanları
requirements.txt
```

## Idempotentlik (temel kural)

Pipeline'ın HER çalışması güvenle tekrarlanabilir olmalı; aynı veri iki kez çekilirse çift kayıt OLUŞMAMALI.

- Her hedef tabloda doğal anahtar belirle ve `UNIQUE` index kur (tarih + kaynak_id gibi bileşik olabilir) — index migration'ı `sql-migration` skill formatında üret.
- Yazma her zaman upsert: `on_conflict` ile doğal anahtar hedeflenir, değişen kolonlar güncellenir.
- "Önce sil sonra yaz" (delete-insert) desenini KULLANMA; kesinti anında veri kaybı yaratır. İstisna: tam yenilenen küçük referans tabloları — o zaman da transaction içinde.
- Script'e tarih aralığı parametresi koy (`--baslangic`, `--bitis` veya `--gun`): hem günlük çalışma hem backfill aynı kodla yapılır, ikinci bir backfill scripti YAZMA.

## Dayanıklılık

- **Retry**: ağ istekleri exponential backoff ile 3 deneme (ör. 2s → 4s → 8s). 4xx'te retry etme (istek hatalı), 5xx ve timeout'ta et.
- **Rate limit / nezaket**: istekler arası bekleme koy (kaynağa göre 0.5-2s), `User-Agent` belirt, robots.txt'e ve site kullanım şartlarına uy. Paralel istek sayısını sınırla.
- **Kısmi başarı**: bir kaydın hatası tüm işi düşürmemeli — hatalı kaydı logla, sayacı artır, devam et. İş sonunda özet: `islenen=120 basarili=118 hatali=2`.
- **Şema değişikliği tespiti**: parser beklenen alanı bulamazsa sessizce None yazma; uyarı logla (kaynak site değişmiş olabilir) ve hatalı sayısını artır.
- **Zaman dilimi**: tarihleri UTC'de sakla (`timestamptz`), gösterimde yerelleştir. Kaynak saat dilimini parse ederken açıkça belirt; örtük yerel saat varsayımı yapma.

## Veri doğrulama

- Yazmadan önce satır bazlı doğrulama: zorunlu alanlar dolu mu, sayısal alanlar makul aralıkta mı, tarih geleceğe/1900 öncesine düşmüyor mu.
- Doğrulamadan geçemeyen satır ya düzeltilir ya loglanıp atlanır — bozuk veri ASLA sessizce yazılmaz.
- Çekilen toplam kayıt sayısı beklenenin çok altındaysa (ör. %50 altı) işi "şüpheli" olarak işaretle ve bildir; kaynak bozulmuş olabilir.

## Loglama ve bildirim

- `logging` modülü, seviyeli (INFO akış, WARNING şüpheli, ERROR hata); `print` kullanma.
- Zamanlanmış işlerde sonuç bildirimi opsiyonel Telegram mesajı ile (bkz. `telegram-bot` skill): başarıda kısa özet, hatada uyarı. Bildirim gönderimi `--sessiz` gibi bir bayrakla kapatılabilir olmalı (test için).

## Zamanlanmış çalıştırma (GitHub Actions deseni)

- Workflow: `schedule` cron (UTC!) + `workflow_dispatch` (elle tetikleme her zaman açık olsun). GitHub, public repolarda 60 gün hareketsizlikten sonra zamanlanmış workflow'ları kendiliğinden devre dışı bırakır; `schedule` tetikleri yoğun saatlerde dakikalarca gecikebilir — kesin saat gerektiren iş için uygun değildir, idempotent yazıldığı için gecikme zararsız olmalıdır. Saat dilimi hesabı: tarih-saat-donem.
- Secrets: `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY` ve kaynak API anahtarları repo secrets'ta; workflow env üzerinden geçirilir.
- Adımlar: checkout → Python kurulum → `pip install -r requirements.txt` → script. Timeout belirle (`timeout-minutes`), takılı iş dakikaları yemesin.

## Çıktı formatı

- Kodu dosya yollarıyla ver; birden çok dosya varsa önce 2-3 maddelik akış özeti.
- Gerekli migration'ları (UNIQUE index dahil) `sql-migration` formatında ayrı blok olarak ekle.
- Sonda "Kurulum" notu: env değişkenleri, secrets, elle test komutu (`python main.py --gun 2026-07-10 --sessiz`).
