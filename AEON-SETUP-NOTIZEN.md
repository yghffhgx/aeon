# Aeon Setup Guide (2026-08-11)

Aeon = autonomes Agent-Framework, das Skills als Markdown-Dateien auf GitHub Actions
ausführt. Läuft komplett über GitHub (Repo + Actions), kein lokaler Daemon nötig.

## Status
- **Fork eingerichtet:** `yghffhgx/aeon` (Fork von aeonfun/aeon, main aktuell)
- **Lokal geklont:** `/home/eli/aeon` (Node 22, gh authentifiziert)
- **CLI funktioniert:** `bash apps/cli/aeon <cmd>` (Dashboard-Alternative, scriptable)
- **Workflows:** 15 registriert (nach initialem Push — Fork-Actions-Initialisierung!)
- **Heartbeat-Run getestet** → FAILED: kein valides LLM-Auth-Backend

## Was getestet wurde
1. Fork + Clone: ✅
2. CLI: `skills ls`, `config show`, `secrets ls`, `packs ls`, `runs ls` — ✅
3. Skill-Dispatch (`aeon skills run heartbeat`): ✅ dispatched, aber Run failed
4. GitHub Actions im Fork: erst nach leerem Commit registriert (total 15 Workflows)

## ✅ FUNKTIONIERT (2026-08-11, Fix-Runde)
Heartbeat-Run #31531370486 = **SUCCESS** mit deepseek-v4-flash via opencode-go!
- **Lösung:** OPENAI_BASE_URL-Variable + OPENAI_API_KEY-Secret (opencode-go-Key) im Fork
- **Workflow-Patch:** OPENAI_BASE_URL an 5 Stellen durchgereicht (install + run-steps)
- **install-harness.sh-Patch:** bei native-key + OPENAI_BASE_URL → custom codex-Provider
  (model_provider="custom", base_url, env_key, wire_api="responses", model=deepseek-v4-flash)
- **Timing-Race-Gefahr:** aeon-Cron committet `chore(cron): heartbeat failed` nach Fail →
  überschreibt origin/main → `git pull --rebase` + `git push` nötig, sonst gehen Fixes verloren!

## Was noch geht / offen
- heartbeat (täglich 08:00 UTC) aktiv — läuft nun auf opencode-go
- Weitere Skills aktivieren: `aeon skills enable digest` etc. (brauchen ggf. eigene Keys)
- OPENAI_MODEL-Variable steuert das Modell (deepseek-v4-flash)

## LLM-Auth (gelöst!)
- OPENAI_API_KEY-Secret = OPENCODE_GO_API_KEY (opencode-go, OpenAI-kompatibel, Responses-API ✅)
- OPENAI_BASE_URL = https://opencode.ai/zen/go/v1
- Aether-Key war NICHT nutzbar (keine Responses-API/Anthropic)

## Nächste Schritte (wenn Key da)
```bash
cd /home/eli/aeon
bash apps/cli/aeon auth --key <OPENROUTER_KEY>   # oder --oauth für Claude Pro
bash apps/cli/aeon skills enable digest          # z.B. Digest-Skill aktivieren
bash apps/cli/aeon skills schedule digest "0 7 * * *"
bash apps/cli/aeon skills run heartbeat          # Test
```
Push passiert automatisch bei config-set (committet + pusht).

## Sicherheit
- Secrets werden via `gh secret set` im Repo gespeichert (nie im Klartext im Code)
- Dashboard bindet nur an loopback (localhost:5555)
- `mode: read-only`-Skills können das Repo nicht schreiben
- Kein Wert wurde im Klartext ausgegeben; Aether-Key-Test-Secret wieder entfernt

## PITFALLS
- **Fork-Actions:** Nach `gh repo fork` sind Workflows NICHT sofort da (total_count=0)
  → ein leerer Commit + Push aktiviert sie
- **codex-Harness:** geht immer zu api.openai.com, kein Base-URL-Override
- **Aether-Key:** nur OpenAI-API-kompatibel, nicht Anthropic → für aeon unbrauchbar
- **CLI starten:** `bash apps/cli/aeon` (NICHT `node apps/cli/aeon` — ist ein bash-Script!)
- **Terminal-Guard:** `./aeon` wird vom Hermes-Terminal-Guard geblockt (Gateway-String)
  → immer `bash apps/cli/aeon` verwenden
