---
name: enforcement-hooks
description: Claude Code hook yapılandırma standartları (PreToolUse/PostToolUse ile deterministik kural zorlaması). Kullanıcı hook ekleme, otomatik kural zorlaması, secret/API key sızıntı koruması, tehlikeli komut engelleme, config dosyası koruması veya "Claude bunu asla yapmasın" tipi kesin kısıt istediğinde MUTLAKA bu skill'i kullan. "Hook ekle", "şunu engelle", "otomatik kontrol", "secret taraması", "bu dosyaya dokunmasın", "tehlikeli komutları blokla" gibi ifadeler geçtiğinde de kullan. Skill'in "genellikle uyulan" rehberliğinden farklı olarak her zaman, istisnasız zorlanan deterministik kurallar üretir.
---

# Enforcement Hooks

Claude Code hook'larıyla deterministik kural zorlaması.

## 1. Hook mu skill mi? — temel ayrım

| | Skill | Hook |
|---|---|---|
| Doğası | Rehberlik — model okur ve **genellikle** uyar | Kod — **her zaman, istisnasız** çalışır |
| Uygun olduğu yer | "Nasıl yapılır" standartları, tercihler | "Asla/mutlaka" kuralları, güvenlik sınırları |
| Atlanabilir mi? | Model bağlam baskısında atlayabilir | Hayır — deterministik script |

Karar kuralı: kuralın ihlali **geri alınamaz zarar** doğuruyorsa (secret sızması, prod config bozulması, yıkıcı git işlemi) hook; sadece kalite/tutarlılık meselesiyse skill.

## 2. Kurulum yapısı

- Hook'lar `~/.claude/settings.json` (kişisel, tüm projeler) veya proje `.claude/settings.json` içinde tanımlanır.
- Script'ler `~/.claude/hooks/` altında tutulur. Kalıcı olanlar claude-config repo'sunda `hooks/` klasöründe versiyonlanmalıdır — **bu klasör henüz repoda yok**; ilk hook yazıldığında oluşturulur ve `setup-mcp.sh` gibi idempotent bir kurulum betiğiyle `~/.claude/hooks/`a kopyalanır (Windows'ta komutlar Git Bash üzerinden `bash ...` ile çalışır).
- Hook script'i stdin'den JSON alır (tool adı + input); **exit 2** aracın çalışmasını engeller ve stderr mesajı Claude'a iletilir, exit 0 izin verir. **Diğer tüm sıfır olmayan çıkış kodları (ör. script hatası, exit 1) engellemez** — araç yine çalışır. Yani çöken bir guard script'i sessizce koruma dışı kalır: kritik guard'larda `set -u`, beklenmeyen girdide bilinçli karar (engelle mi izin ver mi) ve ilk kurulumda gerçek bir engelleme testi şarttır.

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash|Write|Edit",
        "hooks": [{ "type": "command", "command": "bash ~/.claude/hooks/guard.sh" }]
      }
    ]
  }
}
```

## 3. Standart koruma seti

### a) Secret taraması (Write/Edit öncesi)
Yazılacak içerikte şu desenler varsa engelle:
- `sk-`, `sk_live_`, `AKIA`, `ghp_`, `xoxb-` gibi bilinen key önekleri
- `SUPABASE_SERVICE_ROLE` değeri düz metin olarak
- 32+ karakterlik hex/base64 dizeleri `=` atamasıyla (`API_KEY=...`)
- Engel mesajı çözümü de söyler: "env değişkeni kullan, değeri koda yazma"

### b) Config dosyası yazma koruması
- `.env*`, `devcontainer.json`, `settings.json`, `vercel.json`, CI dosyalarına Write/Edit → engelle veya onaya düşür.
- Amaç: Claude'un "yardım ederken" ortam yapılandırmasını sessizce değiştirmesini önlemek — bu dosyalar bilinçli insan kararıyla değişir.

### c) Tehlikeli git flag'leri (Bash öncesi)
Engellenen desenler:
- `git push --force` / `-f` (main/master'a)
- `git reset --hard` (uncommitted iş kaybı)
- `git clean -fd`, `git checkout -- .` (toplu geri alma)
- `rm -rf` (proje kökü veya üstüne işaret ediyorsa)
- Mesaj alternatif önerir: `--force-with-lease`, önce `git stash` vb.

## 4. Yazım kuralları

- Hook script'i **hızlı** olmalı (<1 sn) — her araç çağrısında koşar, yavaş hook tüm oturumu yavaşlatır.
- **Sessiz engelleme yok**: exit 2 ile mutlaka tek satırlık, ne yapılacağını söyleyen bir stderr mesajı ver.
- Hook'lar dar kapsamlı yazılır: her koruma ayrı fonksiyon/dosya; dev bir "her şeyi kontrol eden" script bakım kabusudur.
- Yeni hook eklerken önce **log-only modda** dene (engellemeden sadece kaydeder), yanlış pozitifleri gör, sonra engellemeye çevir.
- Hook'ların kendisi de koddur: değişiklikleri claude-config'e commit'lenir, denetlenebilir kalır.

## 5. Ne hook'a taşınmaz

- Üslup/format kuralları (code-standards'ın işi)
- "Genellikle şöyle yap" rehberliği (skill'lerin işi)
- LLM yargısı gerektiren kontroller (kod kalitesi, mimari uygunluk) — hook deterministik string/desen kontrolü yapar, muhakeme yapmaz
