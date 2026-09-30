---
name: adversarial-review
description: Yüksek riskli yüzeyler için üç geçişli düşmanca inceleme standardı (Attacker/Defender/Auditor). Kullanıcı bir RLS policy, auth akışı, onay durum makinesi, ödeme akışı veya agent/skill tanımını "kırılabilir mi", "güvenli mi", "saldırgan gözüyle bak", "istismar edilebilir mi" diye derinlemesine denetlemek istediğinde MUTLAKA bu skill'i kullan. "Adversarial review", "saldırı senaryosu çıkar", "bunu kırmaya çalış", "güvenlik incelemesi yap" gibi ifadeler geçtiğinde de kullan. Sadece gerçekten yüksek riskli yüzeylerde uygulanır; sıradan CRUD/UI kodunda kullanılmaz.
---

# Adversarial Review

Yüksek riskli yüzeyler için üç geçişli düşmanca inceleme.

## 1. Ne zaman uygulanır (kapsam kontrolü)

**Sadece** şu yüzeylerde:
- RLS policy'ler ve tenant izolasyonu (saas-patterns)
- Auth akışları — giriş, oturum, rol/yetki kontrolü (auth-flow)
- Onay durum makineleri — geçiş kısıtları, kolon koruması (approval-workflow)
- Ödeme/harcama akışları — tutar, idempotency, yetki (payment-integration, ad-campaign-management harcama sınırları)
- Agent/skill tanımları — yetki sınırları, araç erişimi (agent-orchestration)

**Uygulanmaz**: sıradan CRUD, UI bileşenleri, içerik sayfaları, raporlama. Her şeyi adversarial review'dan geçirmek maliyeti işe değmez hale getirir; bu ağır bir denetimdir, seçici kullanılır.

## 2. Üç geçiş

İncelemenin gücü, üç rolün **birbirinden habersiz/bağımsız** çalışmasından gelir. Tek bir oturumun "şimdi saldırgan gibi düşüneyim, şimdi savunucu" demesi zayıftır — geçişler ayrı subagent bağlamlarında koşulur (agent-orchestration'daki attacker / defender / auditor-security ajanları tam bu iş için).

### Geçiş 1 — Attacker (saldırgan)
- Görev: yüzeyi **kırmanın** yollarını bul. Çözüm önermez, sadece istismar senaryosu üretir.
- Araçlar: Read/Grep/Glob — **yazma yok** (saldırgan sistemi değiştirmez, keşfeder).
- Çıktı: raporu yanıt olarak döner, ana oturum `_agent/attack-findings.md`'ye kaydeder — her bulgu bir istismar senaryosu: "X koşulunda Y yaparak Z'ye erişilir."
- Sorulacak sorular: Bu RLS policy'yi hangi tenant_id ile atlatabilirim? Bu auth kontrolü hangi sırayla çağrılırsa baypas olur? Bu durum geçişini hangi ara adımı atlayarak bozabilirim? Bu ödeme akışını retry ile iki kez tetikleyebilir miyim?

### Geçiş 2 — Defender (savunucu)
- Girdi: `_agent/attack-findings.md`. Görev: her saldırıya karşı **düzeltme öner**.
- Araçlar: Read/Grep/Glob — yazma yok; düzeltmeleri yanıt olarak döner (ana oturum `_agent/defense.md`'ye kaydeder), **gerçek dosyalara dokunmaz** (öneri aşaması).
- Çıktı: her saldırı için ya somut düzeltme ya "bu saldırı geçerli değil, çünkü..." gerekçesi.

### Geçiş 3 — Auditor (denetçi)
- Girdi: `_agent/attack-findings.md` + `_agent/defense.md`. Görev: ikisini **bağımsızca yargıla**.
- Araçlar: Read/Grep/Glob — yazma yok.
- Çıktı: yanıt olarak döner, ana oturum `_agent/verdict.md`'ye kaydeder — her bulguya severity (kritik/yüksek/orta/düşük) + karar: **kabul** (savunma yeterli) / **yetersiz** (düzeltme eksik/yanlış) / **reddet** (saldırı geçersizdi).
- Kritik ve çözülmemiş bir bulgu varsa: **deploy-blocker** olarak işaretlenir; deploy-checklist bunu geçmeden yayına izin vermez.

## 3. Ana oturumun rolü

- Ana oturum üç geçişi orkestre eder ve `_agent/` dosyalarını kendisi yazar (ajanların yazma yetkisi yoktur); **karar vermez** — kararı auditor raporuna bakıp insan verir.
- Düzeltmeler ana oturumda (veya producer ile) uygulanır; adversarial review düzeltmeyi **uygulamaz**, sadece bulur ve yargılar.
- Düzeltme uygulandıktan sonra kritik bulgular için **tek tur yeniden attack** yapılır (düzeltme yeni açık yarattı mı?). İki turdan fazla dönülüyorsa tasarım hatalıdır, mimariye geri dönülür (agent-orchestration sonsuz döngü kuralı).

## 4. Neden üç ayrı geçiş

- Aynı bağlam hem saldırıp hem savunduğunda, saldırgan bulgularını farkında olmadan yumuşatır (kendi savunmasını haklı çıkarma eğilimi).
- Ayrı bağlamlar bu yanlılığı kırar: attacker savunmayı hiç görmez, auditor ikisine de bağlı değildir.
- Bu izolasyon subagent'ların ayrı context'iyle sağlanır; tek session içinde "rol değiştirme" taklidi bu güvenceyi vermez.

## 5. Çıktı formatı (özet rapor)

İnceleme sonunda ana oturum tek sayfalık özet üretir:
- Yüzey: (ne incelendi)
- Kritik bulgular: (varsa, deploy-blocker işaretli)
- Kabul edilen savunmalar: (kaç saldırı karşılandı)
- Reddedilen saldırılar: (geçersiz çıkanlar)
- Karar: yayına uygun / düzeltme sonrası tekrar / tasarım revizyonu gerekli
