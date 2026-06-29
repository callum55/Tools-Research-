# Biggest Wins — Easy Token/Cost Cuts (≈10 min total)

The shortlist: highest impact, lowest effort, all evidence-backed. Each shows **why it wins** and **exactly what to do** on your Windows + claude-dashboard setup.

---

## ⚡ The 4 quick wins

### 1. Stop defaulting to Opus → make Sonnet the default  *(biggest single lever, 1 min)*
Opus is ~the priciest model; Haiku is **~15× cheaper**, Sonnet far cheaper than Opus and fine for most coding. You're currently defaulted to Opus.
- **In a session:** run `/model` → pick **Sonnet**. Bump to Opus (`/model opus`) only for genuinely hard reasoning.
- **Make it stick:** add `"model": "sonnet"` to `~/.claude/settings.json` (full file below).
- *Evidence:* Anthropic recommends Sonnet for most coding; routing studies show ~40–80% saving on non-Opus-worthy work.

### 2. Pare your CLAUDE.md + shrink the global one  *(one-time, permanent saving)*
CLAUDE.md is re-sent every session. Obvious boilerplate is pure tax.
- Run `/pare-claude-md` on your dashboard's CLAUDE.md (keep < 200 lines, decisions-only).
- Shrink `~/.claude/CLAUDE.md` to a few universal lines (file below).
- *Evidence:* we measured −72% size / −580 tokens **every session**.

### 3. Add a one-line "keep it simple" rule  *(free ~20% on coding output)*
- Paste into your project CLAUDE.md: **`Prefer the simplest working solution; no unrequested abstractions. Be concise.`**
- *Evidence:* the proven part of ponytail (−20% cost); a plain YAGNI line captures most of it, zero install.

### 4. Lower effort for routine work  *(toggle per task)*
Thinking tokens bill as the pricey **output** kind, and the default budget can be huge.
- Use `/effort` → **medium** (or low) for routine edits; reserve high/max for hard problems.

---

## 📋 Copy-paste configs — and what they actually do

These two files live in your home folder and Claude Code reads them **automatically on every session** — set once, applies forever. You never re-type these preferences again.

### `~/.claude/settings.json` (merge into yours)
```jsonc
{
  "model": "sonnet",
  "outputStyle": "Concise",
  "env": { "MAX_THINKING_TOKENS": "8000" },
  "permissions": {
    "allow": [
      "Bash(dotnet build:*)",
      "Bash(dotnet test:*)",
      "Bash(npm run build:*)",
      "Bash(npm run test:*)",
      "Bash(npm test:*)",
      "Bash(npm run lint:*)",
      "Bash(git status:*)",
      "Bash(git diff:*)",
      "Bash(git log:*)",
      "Bash(git add:*)"
    ],
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

**What each line does:**
- `"model": "sonnet"` → defaults every session to Sonnet instead of Opus (win #1, made permanent).
- `"outputStyle": "Concise"` → tells Claude to keep replies short by default (less output = less cost). *If your version ignores this key, just run `/output-style` → Concise once.*
- `"env": { "MAX_THINKING_TOKENS": "8000" }` → caps how much "thinking" Claude does per request (thinking is billed at the pricey output rate). 8000 is plenty for normal coding.
- `"permissions": { "allow": [...] }` → **this is the "stop asking me permission" list.** Each entry auto-approves a safe command so Claude just runs it instead of pausing to ask. Result: fewer interruptions and fewer half-finished turns. I've listed only **safe, non-destructive** commands (build, test, lint, read-only git, `git add`). I deliberately left out anything risky (`git push`, `rm`, `git reset`) so those still ask first.
- `"permissions": { "deny": [...] }` → blocks Claude from *reading* generated/vendor junk (bin, obj, node_modules, dist, minified, lockfiles). This is the real token saver here — Claude can't accidentally pour a 10,000-line lockfile into context.

> **Easiest way to build your allow-list:** run **`/fewer-permission-prompts`** in Claude Code — it scans what you've already approved and auto-writes a tailored allow-list for you. Or use `/permissions` to manage it interactively. (My list above is a safe starting point you can paste now.)

### `~/.claude/CLAUDE.md` (keep it tiny — it loads EVERY session)
```markdown
- Prefer the simplest working solution; no unrequested abstractions.
- Be concise. Code blocks and exact errors stay verbatim.
- Windows: git is at D:\Apps_Installers\Git (not on PATH); no working python3.
```
**What it does:** this is your *global* memory — Claude reads it at the start of every project. Because it's loaded every single time, it must stay tiny (a few lines). It bakes in your "keep it simple / be concise" rule and the two Windows quirks so you never re-explain them.

---

## 🔭 The dashboard-overview skill — what it is, basically

**The problem it solves:** Claude Code has **no memory of your codebase between sessions and no index of it.** Every new session, when it needs to understand your project, it starts from scratch — running `grep`, listing folders, and **reading the same files again** to re-learn where things live. That re-exploration is one of your biggest hidden token costs on a repo you work in daily.

**What the skill is:** just a small markdown file (`SKILL.md`) where you write down, once, the stuff Claude keeps rediscovering — "the backend is .NET 8 in `/backend`, port 5000; the frontend is React/Vite/PixiJS in `/frontend`, port 5173; state is Zustand; realtime is SignalR; here are the key folders and how they connect." You can also attach reference files (e.g. the full API route list) that load *only if* Claude actually needs them.

**Why it saves tokens:** the skill sits dormant as a ~100-token line ("a description of the dashboard repo") until Claude needs it. When a task touches the codebase, Claude reads this one short file and instantly knows the layout — **instead of reading 8–10 files to figure it out again.** Same understanding, a fraction of the tokens, every session.

**Think of it as:** a one-page "onboarding doc for the new dev" — except the new dev is Claude, every single session.

👉 *Point me at the dashboard repo and I'll write this for you* — I'll read the actual structure and fill it in accurately.

---

## 🛑 Don't bother (proven waste of time on your setup)
- **squeez / headroom / rtk** — don't install/run on native Windows (need python3 / Unix shell / native build).
- **mcp-compressor** — Claude Code already defers MCP tools (85%); redundant.
- **caveman** — slightly *negative* on coding (it's a chat/prose tool).

---

### If you only do TWO things
1. `/model` → **Sonnet** (+ paste the `settings.json` so it sticks).
2. `/pare-claude-md` + paste the one-line "keep it simple" rule into your CLAUDE.md.

Both hit the dominant cost (model price + re-sent context) with first-party evidence behind them. Deeper detail is in this repo's other files (`cost-reduction-catalog-38-methods.md`, `local-setup-context-engineering.md`).
