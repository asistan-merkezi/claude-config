---
name: meta-integration
description: Meta (Facebook/Instagram) Graph API ve Marketing API teknik entegrasyon standartları — kimlik doğrulama, izinler/App Review, organik içerik yayınlama, reklam hesabı erişimi, webhook, rate limit. Kullanıcı "Instagram'a bağlan", "Facebook'a paylaş", "Meta API", "Graph API", "reklam hesabına eriş", "Meta ile entegrasyon", "instagram_content_publish", "ads_management izni" gibi ifadeler kullandığında veya projede Meta platformlarına organik paylaşım/reklam okuma-yazma bağlantısı kurulurken MUTLAKA bu skill'i kullan. Kampanya stratejisi, bütçe/hedefleme kararları için ad-campaign-management skill'i geçerlidir — bu skill sadece API bağlantı katmanını taşır.
---

# Meta Integration

Meta Graph API (organik paylaşım) ve Marketing API (reklam) için teknik entegrasyon standartları — api-integration skill'inin genel wrapper deseninin Meta'ya özel uygulanışı. Bu skill "nasıl bağlanılır"ı taşır; "ne zaman hangi bütçe/hedef" kararı ad-campaign-management'a aittir.

**Önemli**: Meta'nın API sürümleri, izin politikaları ve rate limit değerleri zamanla değişir. Bu skill mimariyi ve deseni doğru taşır; kesin sayısal limitleri/izin adlarını uygulamadan önce Meta'nın güncel geliştirici dokümantasyonundan doğrula.

## 1. İki ayrı API yüzeyi

- **Graph API**: Organik paylaşım, sayfa/hesap yönetimi, yorum/DM okuma. Instagram için `Instagram Graph API` (Business/Creator hesap zorunlu — kişisel hesaplarla çalışmaz).
- **Marketing API**: Reklam hesabı, kampanya/adset/ad oluşturma-okuma, Insights (performans verisi). Ayrı izin seti, ayrı erişim seviyesi gerektirir.
- İkisini aynı wrapper modülüne karıştırma — api-integration'ın "her sağlayıcı için tek wrapper modülü" kuralı burada iki alt-modül olarak uygulanır: `meta-graph.ts` (organik) ve `meta-marketing.ts` (reklam), ortak auth katmanını paylaşır ama endpoint mantığı ayrı.

## 2. Kimlik doğrulama

- Meta App, [developers.facebook.com](https://developers.facebook.com) üzerinden oluşturulur; Business Manager hesabına bağlanır.
- **Sunucu tarafı işler için System User token** tercih edilir (kişisel kullanıcı token'ı gibi düzenli yenileme gerektirmez, Business Manager'da tanımlanır) — mümkün değilse long-lived user token (60 gün) + refresh akışı.
- Token'lar api-integration'daki "encrypted OAuth token storage with single-point refresh" kuralına göre saklanır: şifreli, tek yerden yenilenen, asla client'a sızmayan.
- İzin (scope) örnekleri — proje ihtiyacına göre daralt, gereksiz izin isteme:
  - Organik paylaşım: `instagram_content_publish`, `pages_manage_posts`, `pages_read_engagement`
  - Reklam: `ads_management` (yazma), `ads_read` (salt okuma — sadece analiz gerekiyorsa bunu tercih et)
- **App Review**: Development mode'da sadece App'e eklenmiş test kullanıcıları/hesapları çalışır. Production'da canlı hesaplara erişim için Meta'nın App Review sürecinden geçmek gerekir (kullanım senaryosu videosu + izin gerekçesi) — bu süreç günler sürebilir, proje takvimine erken eklenmeli. Gizlilik politikası URL'si ve veri silme talebi yolu (data deletion URL/callback) App Review ve canlıya geçiş için hazır olmalı.

## 3. Organik içerik yayınlama (Instagram Graph API)

- Akış iki adımlı: (1) medya container oluştur (`POST /{ig-user-id}/media` — image_url/video_url + caption), (2) container'ı yayınla (`POST /{ig-user-id}/media_publish` — container_id ile). Tek adımda "yayınla" yoktur.
- Video/Reels'te container işlenmesi zaman alır — publish'ten önce container durumunu (`status_code`) poll et, `FINISHED` olmadan publish çağırma.
- Hesap başına günlük yayın sayısı sınırlıdır (Content Publishing API limiti) — güncel değeri doğrula, publisher ajanı bu limiti aşarsa kuyruğa alıp ertesi gün devam etmeli, hata fırlatıp durmamalı.
- Fotoğraf/video/reels/story her birinin farklı gereksinimleri var (format, süre, en-boy oranı) — media-content skill'indeki boyut standartlarıyla (1080x1350, 1080x1920) tutarlı üret, Meta tarafında reddedilme riskini üretim aşamasında azalt.

## 4. Reklam hesabı erişimi (Marketing API)

- Her istek `act_{ad-account-id}` altında yapılır; ad account ID CLAUDE.md'de veya env'de tutulur.
- Hiyerarşi: Campaign (hedef/objective) → AdSet (bütçe, hedefleme, teklif) → Ad (yaratıcı). Oluşturma sırası da bu yönde.
- **Yazma işlemleri (kampanya/bütçe oluşturma-değiştirme) yalnızca approval-workflow'dan geçmiş, insan onaylı isteklerle yapılır** — bu skill API çağrısının nasıl yapılacağını taşır, ne zaman/ne kadar bütçeyle karar ad-campaign-management'a ve onay sürecine ait.
- Insights endpoint'i (`GET /{campaign-id}/insights`) salt okunur performans verisi çeker — ads-analyst ajanı bunu kullanır, yazma yetkisi gerekmez.

## 5. Webhook (yorum/DM)

- Abonelik `App Dashboard > Webhooks` üzerinden kurulur; endpoint HTTPS zorunlu.
- Gelen her istekte `X-Hub-Signature-256` header'ı app secret ile HMAC doğrulanır — telegram-bot skill'indeki `secret_token` doğrulama mantığıyla aynı prensip, security-baseline'daki webhook imza kuralına bağlı.
- Doğrulama isteği (subscribe sırasında): `hub.verify_token` kendi belirlediğin gizli değerle eşleşiyorsa `hub.challenge` aynen geri döndürülür, eşleşmiyorsa 403; imza (`X-Hub-Signature-256`) ham body üzerinden hesaplanır (api-integration).
- Meta, 200 dönmezse tekrar dener — handler hızlı 200 dönüp işi arkada yürütmeli (api-integration'daki "fast-ack-async-process" deseni).

## 6. Rate limit ve hata yönetimi

- Business Use Case (BUC) limiti hesap/app bazlı saatlik pencerede çalışır; response header'larında kullanım oranı döner (`X-Business-Use-Case-Usage`) — bu değeri logla, eşiğe yaklaşınca istekleri yavaşlat.
- api-integration'daki genel kural burada da geçerli: 5xx/timeout'ta exponential backoff retry, 4xx'te (izin/parametre hatası) retry etme, hatayı sınıflandırıp logla.
- Token süresi dolmuş/geçersiz hatası (kod değişebilir, güncel dokümantasyona bak) ayrı yakalanır — otomatik yeniden auth denenmez, insan müdahalesi gerektiren bir uyarı olarak Telegram'a düşer (telegram-bot deseni).

## Çıktı formatı

- Kod dosya yollarıyla; `meta-graph.ts` / `meta-marketing.ts` ayrımı ve ortak auth modülü belirtilir.
- Yeni izin gerektiren her entegrasyonda: hangi izin, App Review gerektirip gerektirmediği, geliştirme sırasında test hesabıyla nasıl doğrulanacağı ayrıca not edilir.
- Sonda "Kurulum" notu: env değişkenleri, Meta App/Business Manager tarafında yapılması gereken manuel adımlar, test komutu.
