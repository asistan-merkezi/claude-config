---
name: agent-orchestration
description: Claude Code subagent orkestrasyon standartları (görev bölme, ajan rolleri, paralel çalışma, çıktı birleştirme). Kullanıcı subagent kullanımı, paralel görev, ajan ekibi, görev delegasyonu, büyük refactor bölme veya çoklu ajan akışı istediğinde MUTLAKA bu skill'i kullan. "Subagent kullan", "paralel yap", "ajanlarla böl", "görevi dağıt", "researcher/producer/auditor" gibi ifadeler geçtiğinde de kullan. En az yetki ilkesiyle, dosya bazlı devir-teslimli, denetlenebilir ajan akışları üretir.
---

# Agent Orchestration

Claude Code'da subagent kullanım standartları.

## 1. Ne zaman bölünür, ne zaman bölünmez

**Subagent'a uygun:**
- Ana bağlamı şişirecek geniş keşif (çok dosyalı codebase taraması, dokümantasyon araştırması)
- Birbirinden bağımsız, paralelleştirilebilir işler (3 ayrı modülün ayrı ayrı refactor'u)
- İzole, net tanımlı, tek soruluk görevler ("bu modülde güvenlik riski var mı?")

**Ana oturumda kalır:**
- Adım adım debugging, canlı geri bildirimle evrilen işler
- Tek dosyalık küçük değişiklikler (delegasyon maliyeti işin kendisinden büyük)
- Mimari kararlar — karar her zaman ana oturumda verilir, subagent öneri getirir

## 2. Roller ve en az yetki

Dört temel rol; her biri yalnızca ihtiyacı olan araçlarla tanımlanır. Özel görevler için uzmanlaşmış ajanlar (aşağıda) bu rollerin varyantlarıdır:

| Rol | Görev | Araçlar |
|---|---|---|
| **researcher** | Keşif, okuma, araştırma; özet rapor döner | Read, Grep, Glob, WebSearch — **yazma yok** |
| **producer** | Brief'e göre kod/içerik üretir | Read, Write, Edit — kapsamındaki dizinle sınırlı |
| **auditor** | Üretileni plana/standartlara karşı denetler | Read, Grep — **yazma yok**; bulguları severity ile raporlar |
| **test-runner** | Testleri koşar, kırmızıları analiz eder | Read, Grep, Glob, Bash; Write/Edit yalnızca test dosyaları (`*.test.*`, `*.spec.*`, `tests/`) için — kaynak koda yazma yok |

- Auditor'a yazma yetkisi vermek denetimin bağımsızlığını bozar; researcher'a yazma yetkisi keşfi üretime kaydırır. Rol sınırları bilinçli olarak dardır.
- **Yazma yetkisi olmayan roller (researcher, auditor, attacker, defender, auditor-security) çıktısını yanıt olarak döner; `_agent/` dosyasını ana oturum yazar.** Brief'teki "ÇIKTI" satırı bu roller için "yanıt olarak dön, ana oturum şuraya kaydeder: `_agent/...`" biçiminde yazılır.

### Uzmanlaşmış ajanlar (repodaki `agents/`)

| Varyant | Ajanlar | Ne zaman |
|---|---|---|
| Düşmanca inceleme (adversarial-review) | `attacker` → `defender` → `auditor-security` | Yüksek riskli yüzeyler; her biri ayrı bağlamda, yazma yok |
| Üretim | `producer`, `migrator`, `content-writer`, `legal-drafter`, `support-agent`, `finance-modeler` | Alana özel içerik/kod taslağı; kapsam dışına yazmaz |
| Denetim / analiz | `auditor`, `seo-auditor`, `cro-analyst`, `ads-analyst` | Salt okunur değerlendirme ve öneri; uygulama yapmaz |
| Yürütme | `data-runner`, `reconciliation-runner`, `test-runner`, `publisher`, `tryon-runner`, `postmortem-scribe` | Tanımlı bir hattı koşturur; her biri kendi skill'inin sınırlarına tabidir |
| Keşif | `researcher` | Geniş okuma/araştırma |
| Görsel | `designer` | Üret → ekran görüntüsü → denetle döngüsü |

## 3. Brief yapısı

Her subagent görevi şu şablonla verilir; belirsiz brief belirsiz çıktı üretir:

```
GÖREV: (tek cümle, tek soru/iş)
KAPSAM: (hangi dizin/dosyalar; neye DOKUNMAYACAĞI açıkça)
BAĞLAM: (görev için gereken minimum bilgi — tüm projeyi anlatma)
ÇIKTI: (format + nereye yazacağı: _agent/ altındaki dosya yolu)
BİTTİ KRİTERİ: (neyi tamamlayınca duracağı)
```

- Bir brief'te birden fazla soru olmaz. "İncele, riskleri bul, çözüm öner" → üç ayrı görev.

## 4. Dosya bazlı devir-teslim (_agent/ klasörü)

- Ajanlar arası veri aktarımı proje kökünde geçici `_agent/` klasörü üzerinden yapılır:

```
_agent/
  01-research-auth.md      ← researcher çıktısı
  02-plan.md               ← ana oturumun onaylı planı
  03-produce-auth/notes.md ← producer notları
  04-audit-report.md       ← auditor bulguları
```

- Her ajan girdisini bu dosyalardan okur, çıktısını buraya yazar; ana oturum sadece özetleri okur.
- `_agent/` projenin `.gitignore`'una ilk kullanımda eklenir (yoksa ana oturum ekler); iş bitince silinir. Kalıcı bilgi CLAUDE.md'ye veya koda taşınır.

## 5. Birleştirme + denetim akışı

Standart döngü:

1. **researcher** keşfi yapar → rapor
2. Ana oturum raporu okur, planı yazar, **kullanıcı onaylar**
3. **producer**(lar) plandaki görevleri üretir (bağımsızsa paralel)
4. **auditor** üretimi plana + code-standards/security-baseline'a karşı denetler
5. Critical bulgu → producer'a düzeltme turu; temizse **test-runner** koşar
6. Ana oturum birleştirir, kullanıcıya sunar

- Paralel producer'lar **aynı dosyaya yazamaz** — kapsam çakışması brief aşamasında engellenir.

## 6. Sık hatalar

- **Subagent'ı ana Claude yerine koymak**: orkestrasyon ana oturumda kalır; subagent karar vermez.
- **Bağlam kopyalamak**: tüm CLAUDE.md'yi her brief'e yapıştırma; görevin ihtiyacı kadarını ver.
- **Denetimsiz birleştirme**: producer çıktısı audit'ten geçmeden merge edilmez.
- **Sonsuz düzeltme turu**: audit-düzeltme döngüsü 2 turu geçerse durup ana oturumda kök nedene bakılır (muhtemelen plan hatalı).
- **Ephemeral çıktıyı kaybetmek**: önemli karar/bulgu `_agent/`'ta bırakılmaz, kalıcı yere taşınır.
