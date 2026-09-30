---
name: telegram-bot
description: Telegram bot ve bildirim entegrasyonu üretim standartları. Kullanıcı Telegram bildirimi, bot kurulumu, webhook ile mesaj alma, onay/komut akışı, zamanlanmış rapor gönderimi veya sistemden Telegram'a mesaj gönderme istediğinde MUTLAKA bu skill'i kullan. "Telegram'a gönder", "bot kur", "bildirim gelsin", "telegram webhook", "onay mesajı", "günlük rapor at" gibi ifadeler geçtiğinde de kullan. Güvenli, dayanıklı Telegram bot entegrasyonları üretir.
---

# Telegram Bot

Telegram Bot API ile bildirim ve etkileşim entegrasyonu standartları. Projeye özel detaylar (bot adı, chat/kullanıcı id'leri, mesaj içerikleri, komut listesi) CLAUDE.md'dedir — bu skill evrensel deseni taşır.

## Kullanım tipi seçimi

İhtiyaca göre iki desen; karıştırma:

1. **Salt bildirim (push)**: Sistem → Telegram tek yön. Webhook GEREKMEZ; `sendMessage` çağrısı yeter. Zamanlanmış rapor, hata uyarısı, iş özeti için varsayılan budur. Basit ihtiyaca webhook altyapısı kurma.
2. **Etkileşimli bot (komut/onay)**: Kullanıcı → bot mesaj/butonuna yanıt verir. Webhook (veya polling) gerekir. Onay akışları, komutla sorgu için.

## Kurulum temelleri

- Bot @BotFather'dan oluşturulur; token `TELEGRAM_BOT_TOKEN` env değişkeninde. Token ASLA koda/loga yazılmaz, commit edilmez.
- Hedef chat id'si `TELEGRAM_CHAT_ID` env'de. Chat id bulma: kullanıcı bota `/start` atar → `getUpdates` ile id okunur. Grup için bot gruba eklenir, grup id'si negatif sayıdır.
- Mesaj gönderim yardımcısı TEK yerde tanımlanır (`lib/telegram.ts` veya `telegram.py`); dağınık fetch çağrısı yapılmaz.

## Bildirim gönderim deseni

```python
# telegram.py
import os, requests, logging

def telegram_gonder(mesaj: str, sessiz: bool = False) -> bool:
    """Telegram'a mesaj gonderir. Hata bildirimi is akisini DURDURMAZ."""
    if sessiz:
        return True
    try:
        r = requests.post(
            f"https://api.telegram.org/bot{os.environ['TELEGRAM_BOT_TOKEN']}/sendMessage",
            json={
                "chat_id": os.environ["TELEGRAM_CHAT_ID"],
                "text": mesaj,
                "parse_mode": "HTML",
                "disable_web_page_preview": True,
            },
            timeout=10,
        )
        return r.ok
    except Exception as e:
        # {e} loglanmaz: requests hataları URL'yi (token dahil) içerir
        logging.error("Telegram gonderilemedi: %s", type(e).__name__)
        return False
```

Kurallar:

- **Bildirim hatası ana işi düşürmez**: gönderim başarısızsa logla, devam et. Bildirim "nice to have"dir.
- `parse_mode` olarak `HTML` tercih et (MarkdownV2'nin kaçış karakteri sorunları bela). Kullanıcı girdisi mesaja giriyorsa `<`, `>`, `&` karakterlerini escape et.
- Mesaj limiti 4096 karakter: uzun raporları böl veya özetle; tabloyu `<pre>` bloğunda hizala.
- Hız limiti: aynı sohbete saniyede yaklaşık 1, gruba dakikada yaklaşık 20 mesaj; toplu gönderimde araya gecikme koy, 429 yanıtındaki `retry_after` kadar bekle (güncel limitler için Bot API dokümanı).
- Test için `sessiz` bayrağı her gönderim fonksiyonunda bulunmalı.
- Mesaj biçimi: başlıkta emoji ile durum işareti (✅ ⚠️ ❌), altında kısa özet satırları. Örnek: `✅ Gunluk veri cekimi\nislenen: 120 | hatali: 2`.

## Webhook deseni (etkileşimli bot)

- Webhook endpoint'i: Next.js route handler veya Supabase Edge Function. `setWebhook` ile kaydedilir; URL HTTPS zorunlu.
- **Güvenlik (zorunlu)**:
  - `setWebhook`'ta `secret_token` parametresi ver; gelen her istekte `X-Telegram-Bot-Api-Secret-Token` header'ını doğrula. Doğrulamayan istek 401 ile reddedilir.
  - Gelen mesajın `chat.id`'sini izinli listeyle karşılaştır — bot herkese açıktır, yabancı chat'ten gelen komut İŞLENMEZ.
- Telegram, 200 dönmezse update'i tekrar gönderir: handler'da işi hızlı kabul et (hemen 200 dön), uzun işlemi arkada yürüt. Aynı `update_id` iki kez gelebilir — kritik işlemlerde update_id bazlı tekrar kontrolü yap.
- **Onay akışı deseni** (insan onayı gereken işlemler): mesajda inline keyboard (Onayla/Reddet butonları, `callback_data` içinde işlem id'si) → callback geldiğinde işlem id'si veritabanından okunur → **yaş kontrolü**: onay isteği belirli süreden eskiyse (ör. 5 dk) otomatik REDDET — geç gelen onay bayat veriyle işlem yapmasın. Sonuç mesajla teyit edilir ve butonlar kaldırılır (`editMessageReplyMarkup`). Callback geldiğinde `answerCallbackQuery` çağrılır; yoksa butonda yükleniyor göstergesi takılı kalır.

## Komut işleme

- Komutlar `/komut` formatında; bilinmeyen komuta kısa Türkçe yardım mesajı dön.
- Komut → fonksiyon eşlemesi tek bir dict/map'te tutulur; if-else zinciri kurma.
- Uzun süren komutlarda önce "İşleniyor..." mesajı gönder, sonuç hazır olunca ikinci mesajla ilet.

## Test ve doğrulama

- Kurulum testi sırası: token doğrula (`getMe`) → chat id doğrula (test mesajı) → varsa webhook doğrula (`getWebhookInfo`).
- Webhook geliştirme aşamasında `getUpdates` ile polling kullanılabilir; production'a geçerken webhook'a alınır (ikisi aynı anda çalışmaz).

## Çıktı formatı

- Kod dosya yollarıyla; webhook varsa endpoint + `setWebhook` kurulum komutu (curl) birlikte verilir.
- Sonda "Kurulum" notu: env değişkenleri, BotFather adımları (gerekiyorsa), test komutu.
