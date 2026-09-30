---
name: email-sequences
description: Yaşam döngüsü (lifecycle) e-posta dizileri tasarım standartları — onboarding, aktivasyon, churn önleme, geri kazanma, deneme süresi bitişi dizileri. Kullanıcı hoş geldin dizisi, onboarding e-postaları, damla (drip) kampanya, deneme bitiyor hatırlatması, terk edilmiş işlem maili veya e-posta otomasyonu akışı istediğinde MUTLAKA bu skill'i kullan. "E-posta dizisi", "onboarding mailleri", "drip kampanya", "kullanıcıyı geri kazan", "trial bitince mail" gibi ifadeler geçtiğinde de kullan. Teknik gönderim email-service skill'ine tabidir — bu skill dizinin kurgusunu, zamanlamasını ve içerik stratejisini taşır.
---

# Email Sequences (Yaşam Döngüsü E-posta Dizileri)

Kullanıcı yaşam döngüsüne bağlı otomatik e-posta dizilerinin kurgusu. Gönderim altyapısı, domain/deliverability ve şablon tekniği email-service'e; metin dili copywriting'e aittir.

## Temel ilkeler

1. **Tetikleyici bazlı, takvim bazlı değil**: Diziler kullanıcı davranışına bağlanır ("kayıt oldu ama 3 gün giriş yapmadı"), körlemesine "3. gün maili" atılmaz. Davranış gerçekleştiyse ilgili mail iptal edilir (ilk projeyi oluşturan kişiye "ilk projeni oluştur" maili gitmez).
2. **Her mail tek iş**: Tek amaç, tek CTA. İki şey isteyen mail ikiye bölünür.
3. **Değer/istek dengesi**: Dizide her "bizden bir şey iste" mailine karşılık en az bir "değer ver" maili (ipucu, kullanım örneği) bulunur.
4. **Çıkış her zaman açık**: Her pazarlama mailinde tek tıkla abonelik iptali; iptal edilen kullanıcıya yalnızca zorunlu transactional mailler gider. Bu ayrım sunucu tarafında zorlanır (email-service).
5. **İYS (Türkiye)**: Pazarlama niteliğindeki dizi mailleri yalnızca İYS'de ticari ileti onayı kayıtlı alıcılara gider; onay kaydı olmayan kullanıcı yalnızca transactional mailler alır. Dizi, alıcı listesini İYS onay durumuyla süzmeden çalışmaz (kvkk-legal).

## Standart diziler

- **Onboarding (kayıt sonrası)**: hoş geldin (anında, tek sonraki adım) → aktivasyon dürtmesi (ilk temel eylem yapılmadıysa) → değer maili (örnek/ipucu) → sosyal kanıt + davet. 4-5 mail, 7-14 güne yayılır.
- **Deneme süresi**: kalan gün hatırlatması abartısız ve dürüst; bitişten önce "ne kaybedersin değil ne elde ettin" özeti (kullanım verisiyle: "bu ay X kayıt oluşturdun"); bitiş günü net karar maili.
- **Churn önleme**: kullanım düşüşü tetikler (son X gün giriş yok); suçlayıcı değil yardımcı ton ("bir engel mi var?" + tek soru).
- **Geri kazanma**: iptal sonrası 1 mail + uzun sessizlik; ısrarcı geri kazanma dizisi kurulmaz.

## Kurgu standartları

- Dizi tanımı tablo formatında teslim edilir: tetikleyici | bekleme | iptal koşulu | konu satırı | tek CTA | ölçülecek metrik.
- Konu satırı 50 karakter altı, clickbait yasak; ön izleme metni (preheader) konuyu tekrarlamaz, tamamlar.
- Zamanlama alıcının saat dilimine göre; gece gönderimi yapılmaz.
- Dizi durum makinesi (hangi kullanıcı hangi adımda) veritabanında izlenebilir olmalı — "gönderildi mi bilmiyoruz" durumu kabul edilmez.

## Ölçüm

- Dizi bazında izlenen metrikler: açılma değil sonuç metriği esas alınır (aktivasyon oranı, dönüşüm, iptal oranı). Açılma oranı yalnızca deliverability sinyali olarak okunur.
- Spam şikâyet oranı eşiği aşılırsa dizi otomatik durdurulur ve insana raporlanır.
