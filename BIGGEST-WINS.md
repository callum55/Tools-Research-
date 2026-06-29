# Biggest Wins — Easy Token/Cost Cuts (≈15 min total)

The shortlist: highest impact, lowest effort, all evidence-backed. Do them top to bottom. Each shows **why it wins** and **exactly what to do** on your Windows + claude-dashboard setup.

---

## ⚡ The 7 quick wins

### 1. Stop defaulting to Opus → make Sonnet the default  *(biggest single lever, 1 min)*
Opus is ~the priciest model; Haiku is **~15× cheaper**, Sonnet far cheaper than Opus and fine for most coding. You're currently defaulted to Opus.
- **In a session:** run `/model` → pick **Sonnet**. Bump to Opus (`/model opus`) only for genuinely hard reasoning.
- **Make it stick:** add `"model": "sonnet"` to `~/.claude/settings.json` (full file below).
- *Evidence:* Anthropic recommends Sonnet for most coding; routing studies show ~40–80% saving on non-Opus-worthy work.

### 2. Confirm you're on a Max subscription, not API pay-per-token  *(verify, 1 min)*
For daily Claude Code use a subscription is **15–30× cheaper** than per-token (Max 20x = $200 flat ≈ $600–1,500 of API value). If you're already on Max/Pro — done. If you're billing per-token for daily work, switch.
- Check with `/usage`.

### 3. `/clear` between unrelated tasks  *(free habit, biggest ongoing lever)*
Your bill is dominated by context that gets **re-sent every turn**. Stale context you forgot to clear is pure waste.
- **Rule:** if you'd open a new doc for it, `/clear` first. Use `/compact focus on code and tests` when you want to keep going mid-task.
- *Evidence:* Anthropic-recommended; `/compact` measured 70k → ~4k tokens.

### 4. Disable MCP servers you're not using right now  *(30 sec, recurring win)*
You have 5 connected (n8n, playwright, **tends2 ≈40 tools**, supabase, vapi). Keep only what the task needs.
- Run `/mcp` → disable the idle ones. Re-enable when needed.
- Bonus: prefer the **Supabase CLI / `gh`** over their MCP servers for routine ops (zero per-tool overhead).

### 5. Pare your CLAUDE.md + shrink the global one  *(one-time, permanent saving)*
CLAUDE.md is re-sent every session. Obvious boilerplate is pure tax.
- Run `/pare-claude-md` on your dashboard's CLAUDE.md (keep < 200 lines, decisions-only).
- Shrink `~/.claude/CLAUDE.md` to a few universal lines (file below).
- *Evidence:* we measured −72% size / −580 tokens **every session**.

### 6. Add a one-line "keep it simple" rule  *(free ~20% on coding output)*
- Paste into your project CLAUDE.md: **`Prefer the simplest working solution; no unrequested abstractions. Be concise.`**
- *Evidence:* the proven part of ponytail (−20% cost); a plain YAGNI line captures most of it, zero install.

### 7. Lower effort for routine work  *(toggle per task)*
Thinking tokens bill as the pricey **output** kind, default budget can be huge.
- Use `/effort` → **medium** (or low) for routine edits; reserve high/max for hard problems.

---

## 📋 Copy-paste configs

### `~/.claude/settings.json` (merge into yours)
```jsonc
{
  "model": "sonnet",
  "outputStyle": "Concise",
  "env": { "MAX_THINKING_TOKENS": "8000" },
  "permissions": {
    "deny": [
      "Read(**/bin/**)",
      "Read(**/obj/**)",
      "Read(**/node_modules/**)",
      "Read(**/dist/**)",
      "Read(**/*.min.js)",
      "Read(**/package-lock.json)"
    ]
  }
}
```
*Why:* Sonnet default + concise replies + capped thinking + Claude never wastes context reading build/vendor junk. (If `outputStyle` isn't picked up on your version, just run `/output-style` → Concise once.)

### `~/.claude/CLAUDE.md` (keep it tiny — it loads EVERY session)
```markdown
- Prefer the simplest working solution; no unrequested abstractions.
- Be concise. Code blocks and exact errors stay verbatim.
- Windows: git is at D:\Apps_Installers\Git (not on PATH); no working python3.
```

---

## 🔭 One slightly-bigger win (worth 20 min later)
**A `dashboard-overview` skill** so Claude stops re-reading your repo structure every session (it has no index — it re-greps and re-reads files fresh each time). Put architecture + key dirs + conventions in `~/.claude/skills/dashboard-overview/SKILL.md`; it loads only when relevant. Biggest recurring saving for same-codebase work. *(Ask me to scaffold it.)*

---

## 🛑 Don't bother (proven waste of time on your setup)
- **squeez / headroom / rtk** — don't install/run on native Windows (need python3 / Unix shell / native build).
- **mcp-compressor** — Claude Code already defers MCP tools (85%); redundant.
- **caveman** — slightly *negative* on coding (it's a chat/prose tool).

---

### If you only do THREE things
1. `/model` → **Sonnet** (and confirm Max subscription).
2. `/clear` between tasks; `/mcp` → disable idle servers.
3. `/pare-claude-md` + paste the one-line "keep it simple" rule.

All three hit the dominant cost (model price + re-sent context) with first-party evidence behind them. Sources and the deeper detail are in this repo's other files (`cost-reduction-catalog-38-methods.md`, `local-setup-context-engineering.md`).
