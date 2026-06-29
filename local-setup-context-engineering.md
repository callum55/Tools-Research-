# Optimising Your Local Claude Setup for Token Efficiency

**Your question:** how do I set up my CLAUDE.md / files / project so Claude gets the *right* context for repeat work on the *same* codebase — without paying for tokens I don't need?

**The one principle that answers all of it — Progressive Disclosure:** structure context so it loads **on demand**, not upfront. Anthropic's own measurement: loading all domain docs at startup = **70,000 tokens before any work**; the skills/progressive model loads a **~500-token index** and pulls full content only when relevant — a **140× difference**. Everything below is an application of this.

> **Key nuance most guides get wrong:** `@import` in CLAUDE.md **loads at launch** — it helps *organisation* but does **not** reduce context. The things that actually defer loading are **subdirectory CLAUDE.md**, **Skills**, and **slash commands**. Build around those.

---

## 1. The CLAUDE.md hierarchy — put each fact at the cheapest level

Claude merges three levels; the lower two load **on demand**:

| Level | File | Loads | Put here |
|---|---|---|---|
| **Global** | `~/.claude/CLAUDE.md` | **Every session, every project** | Only universal, tiny prefs. Keep it ~10–20 lines. |
| **Project** | `{repo}/CLAUDE.md` | Every session *in that repo* | Non-obvious architecture + gotchas + commands. **< 200 lines** (Anthropic). |
| **Subdirectory** | `{repo}/backend/CLAUDE.md`, `{repo}/frontend/CLAUDE.md` | **Only when Claude touches that folder** | Area-specific rules. **Free until used.** |

**For your dashboard** (.NET 8 backend + React/Vite/PixiJS/Zustand/SignalR frontend) this is a big win — split context so backend rules don't load when you're in frontend code and vice-versa:

```
claude-dashboard/
  CLAUDE.md                 # 1 screen: stack, how to run (start.ps1), the 3-4 real gotchas
  backend/CLAUDE.md         # .NET 8 specifics, FileSystemWatcher quirk, port 5000 — loads only in backend/
  frontend/CLAUDE.md        # PixiJS/WebGL + Zustand + SignalR patterns — loads only in frontend/
```

**Rules for the content (this is where the tokens are):**
- **Decisions, not descriptions.** "We pin React 18.2.0 — 18.3 breaks acme-charts in prod" ✅. "src/ has the source code" ❌ (obvious → delete; that's what `pare-claude-md` does).
- Run **`/pare-claude-md`** on each one. (We measured −72% size / −580 tok/session on a typical bloated file.)
- Keep it **stable** — editing CLAUDE.md mid-session busts the prompt cache (cache = 10% of input price; don't break it).
- Global `~/.claude/CLAUDE.md` is the most expensive real-estate (every session) — your whole global file should be ~5 lines (e.g. "Prefer the simplest working solution; no unrequested abstractions. Be concise.").

## 2. Skills — the right home for *repeat knowledge* (3-tier, mostly free)

A Skill (`SKILL.md`) loads in three tiers, so you can store a *lot* at near-zero cost:
- **Tier 1 — name + description (~100 tokens):** always in context, just enough for Claude to know it exists.
- **Tier 2 — SKILL.md body:** loads **only when the skill is triggered.**
- **Tier 3 — linked files** (`references/`, `scripts/`, `assets/`): load **only when that file is opened.** Bundle huge API docs/schemas here with **zero penalty** until used.
- **Scripts in `scripts/`:** Claude runs them and sees **only the output — the script code never enters context.**

**Two high-value skills for same-codebase work:**

**(a) A `project-overview` skill** — so Claude stops re-exploring your repo every session (its #1 hidden cost; Claude has *no* index — it greps + reads files fresh each time).
```
~/.claude/skills/  (or {repo}/.claude/skills/)
  dashboard-overview/
    SKILL.md            # architecture, entry points, key dirs, conventions, "where things live"
    references/
      api-routes.md     # full route list — loaded only if asked about routes
      db-schema.md      # schema — loaded only when touching data
```
SKILL.md description: *"Architecture & conventions for the claude-dashboard repo. Use when navigating or modifying this codebase."* Now a structural question costs ~the skill body instead of 10 file-reads.

**(b) Domain skills** for the recurring areas: `signalr-realtime`, `pixijs-canvas`, `dotnet-filewatcher`. Each loads only when that topic comes up.

**Migrate detail OUT of CLAUDE.md into skills:** anything that isn't needed *every* session (PR-review checklist, migration steps, deploy runbook) belongs in a skill — it's in context only when invoked, keeping your base context small.

## 3. Slash commands — encode *repetitive tasks* once, run forever

Reusable prompt templates in `.claude/commands/*.md` (project) or `~/.claude/commands/` (global). Measured impact: a repeated workflow dropped from **~3,200 → ~850 tokens/run** by replacing re-typed instructions with a command.

For your dashboard, create `{repo}/.claude/commands/`:
```
add-endpoint.md     # "Add a {$1} endpoint to the backend following our controller + DTO + test pattern…"
add-component.md    # "Create a React component {$1} using Zustand store + our file layout…"
run-checks.md       # "Run dotnet build and npm run build; report only failures."
review-diff.md      # "Review the current git diff for our conventions; list issues only."
```
Use `$ARGUMENTS` / `$1 $2` for slots — `/add-endpoint Orders` expands into the full, specific instruction. Each command bakes in your conventions so you never re-explain them (and Claude doesn't re-derive them by reading files).

## 4. Stop re-reading the same files (the biggest repeat-session cost)

Claude Code has **no vector index** — every session it re-greps and re-reads. Counter it:
- **`project-overview` skill** (above) — answers "where/how" without file reads.
- **Reference files by exact path / `@mention`** instead of vague asks, so Claude opens one file instead of searching many.
- **Delegate exploration to a subagent** — "use a subagent to find where X is handled and report back." The subagent burns its *own* context reading 10 files; your main session gets only the summary. (⚠️ don't overuse — subagent-heavy flows can hit ~7× tokens; worth it only when the read volume is large.)
- **Symbol navigation** — install a code-intelligence plugin for TS/.NET so "go to definition" replaces grep-then-read-several-candidates.
- **(Big repos only) a codebase-memory / vector-index MCP** — persists a map across sessions; one implementation claims **99.2%** fewer tokens on structural queries. Weigh against the MCP's own overhead; for a single dashboard repo the `project-overview` skill is usually enough.

## 5. Keep junk out of context (settings)

Generated/vendor files waste context when Claude reads or searches them. In `{repo}/.claude/settings.json`, deny reads of build output:
```jsonc
{ "permissions": { "deny": [
  "Read(./backend/bin/**)", "Read(./backend/obj/**)",
  "Read(./frontend/node_modules/**)", "Read(./frontend/dist/**)",
  "Read(**/*.min.js)", "Read(**/package-lock.json)"
] } }
```
(`.claudeignore` is still only a feature request; `permissions.deny` is the working mechanism today.)

## 6. Settings that quietly save tokens

- **Default model Sonnet** (`/config`) — biggest lever; you're on Opus by default.
- **Output style: Concise** (`/output-style`) — trims response verbosity globally.
- **Permissions allowlist** for your safe, frequent commands (`dotnet build`, `npm run build`, `git status`) — fewer permission round-trips = fewer wasted half-turns. *(You can run `/fewer-permission-prompts` to auto-generate this from your history.)*
- **A `SessionStart` hook** can inject a tiny "current focus" line if you want, but prefer the skill/CLAUDE.md hierarchy — it's cache-friendlier.

---

## Anti-patterns that quietly cost you tokens
- ❌ A 400-line project CLAUDE.md full of obvious content → ✅ pare it, < 200 lines, decisions-only.
- ❌ Putting *everything* in CLAUDE.md → it loads every session. ✅ Move non-universal detail to **skills** (load on demand).
- ❌ Relying on `@import` to "save context" → it **loads at launch anyway**. ✅ Use **subdirectory CLAUDE.md** + **skills** for true deferral.
- ❌ Re-explaining the same workflow each chat → ✅ a **slash command**.
- ❌ Letting Claude re-explore the repo every session → ✅ a **`project-overview` skill** + subagent exploration.
- ❌ Editing CLAUDE.md mid-task → busts prompt caching. ✅ keep stable content stable.

## Your concrete setup checklist (for claude-dashboard)
1. **Split CLAUDE.md** into root + `backend/CLAUDE.md` + `frontend/CLAUDE.md`; pare each; decisions-only; < 200 lines.
2. **Shrink global `~/.claude/CLAUDE.md`** to ~5 universal lines.
3. **Create a `dashboard-overview` skill** (architecture + Tier-3 reference files for routes/schema) so Claude stops re-exploring.
4. **Add 3–5 slash commands** in `.claude/commands/` for your recurring tasks (add-endpoint, add-component, run-checks, review-diff).
5. **Move runbooks** (deploy, migrations, PR-review) out of CLAUDE.md into invoked skills.
6. **Add `permissions.deny`** for bin/obj/node_modules/dist.
7. **Set Sonnet default + Concise output style + permission allowlist.**

> **Why this works in one line:** you pay full price only for the *universal* base (tiny global + lean project CLAUDE.md), and everything else — area rules, deep knowledge, repeat workflows, the codebase map — sits one tier down and is billed **only when actually used**. Same context quality, a fraction of the tokens.

---

### Sources
- Anthropic, **Memory / CLAUDE.md hierarchy** (global→project→subdir, on-demand subdir loading, <200 lines) — https://code.claude.com/docs/en/memory
- Anthropic, **Agent Skills + authoring best practices** (3-tier progressive disclosure; ~100-token index; scripts' code never enters context) — https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview · https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills
- 70k→500 token / 140× progressive-disclosure measurement — https://dev.to/jimquote/claude-skills-vs-mcp-complete-guide-to-token-efficient-ai-agent-architecture-4mkf
- Subdirectory CLAUDE.md on-demand loading — https://dev.to/subprime2010/claude-code-project-memory-how-claudemd-files-work-across-nested-directories-1mk8
- `@import` loads at launch (organisation, not reduction) — https://agentfactory.panaversity.org/docs/General-Agents-Foundations/claude-code-teams-cicd/claude-md-configuration-hierarchy
- Anthropic, **Slash commands** (`.claude/commands/`, `$ARGUMENTS`) + 3,200→850/run — https://code.claude.com/docs/en/slash-commands · https://oneaway.io/blog/claude-code-skills-and-slash-commands-the-complete-guide
- Claude Code has **no vector index** (greps/reads on demand); subagents + codebase-memory-mcp (99.2% on structural queries) — https://vadim.blog/claude-code-no-indexing/ · https://www.russ.cloud/2026/05/10/codebase-memory-mcp-giving-claude-code-and-codex-a-map/
- `permissions.deny` as the working exclude mechanism — https://github.com/anthropics/claude-code/issues/29455
