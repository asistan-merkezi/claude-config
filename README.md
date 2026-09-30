# claude-config

Claude Code için kişisel skill, agent ve kurulum dosyaları.

## İçerik

- `skills/` — her klasörde bir `SKILL.md`. Evrensel "nasıl" standartları burada; projeye özel değerler (marka, tablo adları, roller, hex kodları) her projenin `CLAUDE.md` dosyasındadır.
- `agents/` — `skills/agent-orchestration` içindeki rol tablosuna bağlı uzmanlaşmış subagent tanımları.
- `setup-mcp.sh` — Playwright ve Context7 MCP sunucularını `--scope user` ile idempotent kurar.

## Kurallar

- Skill `description` alanı tek satırdır ve "MUTLAKA ... kullan" tetik ifadelerini içerir.
- Bir bilgi birden çok projede aynen geçerliyse skill'e, yalnızca bir projeye aitse o projenin `CLAUDE.md`'sine yazılır.
- Yazma yetkisi olmayan agent'lar çıktısını yanıt olarak döner; `_agent/` dosyalarını ana oturum yazar.
- `hooks/` klasörü henüz yok (bkz. `skills/enforcement-hooks`).
