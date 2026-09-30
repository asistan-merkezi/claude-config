---
name: sop-builder
description: Standart operasyon prosedürü (SOP) yazım standartları — tekrarlanan iş süreçlerinin adım adım, tek başına uygulanabilir dokümana dönüştürülmesi. Kullanıcı SOP, süreç dokümanı, operasyon prosedürü, işletme talimatı, "bunu her seferinde aynı yapalım", devir-teslim dokümanı veya runbook istediğinde MUTLAKA bu skill'i kullan. "SOP yaz", "süreci belgele", "prosedür çıkar", "adım adım talimat", "checklist yap", "yeni kişi de yapabilsin" gibi ifadeler geçtiğinde de kullan. Bilgiyi kişiden bağımsızlaştıran, güncel tutulabilir süreç dokümanları üretir.
---

# SOP Builder (Standart Operasyon Prosedürleri)

Tekrarlanan süreçleri, işi hiç yapmamış birinin (veya bir agent'ın) hatasız uygulayabileceği dokümana çevirme standartları.

## Temel ilkeler

1. **Uygulayıcı testi**: SOP'un ölçüsü, süreci bilmeyen birinin yalnızca dokümanla işi tamamlayabilmesidir. "Malum olduğu üzere" varsayımı yasak; her ön koşul (erişim, araç, yetki) baştan listelenir.
2. **Adım = tek eylem + doğrulama**: Her adım tek gözlemlenebilir eylemdir ve "doğru yaptığını nereden anlarsın" satırı içerir ("komut çalıştır → çıktıda ✅ N kayıt görmelisin").
3. **Otomasyon önce sorgulanır**: SOP yazmadan önce sor: bu süreç script/cron/hook olabilir mi? Otomatikleşebilen iş için SOP yazmak, ponytail ilkesine aykırıdır. SOP, insan kararı gerektiren veya nadir işler içindir.
4. **Tek sahip, tek güncel sürüm**: Her SOP'un sahibi ve son güncelleme tarihi vardır; repo'da tek yerde durur (`docs/sops/`). Süreç değişince SOP aynı commit'te güncellenir — bayat SOP, olmayan SOP'tan tehlikelidir.

## SOP şablonu

- **Başlık + amaç**: hangi durumda bu SOP uygulanır (tetikleyici net: "yeni tenant açılışında", "ayın 1'inde").
- **Ön koşullar**: gereken erişimler, araçlar, önceden tamamlanmış olması gerekenler.
- **Adımlar**: numaralı, tek eylem + doğrulama; komutlar kopyalanabilir blok halinde; ekran gerektiren adımda hangi ekran/menü olduğu açık.
- **Hata dalları**: en olası 2-3 hata için "şu olursa şunu yap"; öngörülemeyen hata için eskalasyon (kime/nereye haber ver, neyi ASLA yapma).
- **Süre + sıklık**: tahmini süre ve ne sıklıkta uygulandığı.

## İçerik kuralları

- Karar noktaları akış olarak yazılır ("X ise 5. adıma, değilse 7. adıma") — belirsiz "gerekirse" ifadesi yasak.
- Geri alınamaz adımlar (silme, ödeme, müşteri e-postası) ⚠️ ile işaretlenir ve öncesine kontrol adımı konur.
- Secret/şifre SOP'a yazılmaz; nerede saklandığı yazılır (security-baseline log hijyeni kuralıyla tutarlı).
- Agent'ların uygulayacağı SOP'lar, agent-orchestration brief formatına dönüştürülebilir yapıda tutulur — SOP'lar gelecekteki agent tanımlarının hammaddesidir.

## Çıktı formatı

`docs/sops/<ad>.md` dosyası: başlık-amaç, ön koşullar, numaralı adımlar (eylem + doğrulama), hata dalları, süre/sıklık, sahip ve son güncelleme tarihi. Komutlar kopyalanabilir blok halinde.
