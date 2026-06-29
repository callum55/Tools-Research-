# One-Paste Setup Prompt for Claude Code

**How to use:** open Claude Code **in your `claude-dashboard` repo root**, paste everything in the code block below, and send. It sets up all the proven cost wins, checks/install code-intelligence (LSP) for C#/Python/TypeScript, audits your MCPs for skill/CLI conversion, and wires output-filtering hooks — safely, with a plan and confirmation first.

> Tip: it changes global config and repo files. It will back things up and show a plan before writing. If anything needs `git`/admin it can't do, it will tell you instead of failing silently.

---

```text
You are setting up my Claude Code environment and this repo (a .NET 8 backend + React/Vite/PixiJS/Zustand/SignalR frontend dashboard on Windows) for maximum token/cost efficiency WITHOUT losing quality. Work in phases. Before writing anything, print a short PLAN of what you'll change and ask me to confirm. Back up any file you overwrite (copy to <file>.bak). On Windows, my git is at D:\Apps_Installers\Git (not on PATH) and `python3` is the broken Store stub — real Python 3.12 is at %LOCALAPPDATA%\Programs\Python\Python312 and the `py` launcher works. Skip and REPORT anything that can't be done cleanly rather than forcing it.

PHASE 1 — settings.json (~/.claude/settings.json, merge, don't clobber):
- Set "model": "sonnet" (default to Sonnet, not Opus).
- Set "outputStyle": "Concise" (if unsupported on this version, run /output-style Concise instead and tell me).
- Add permissions.allow for SAFE, non-destructive commands only: dotnet build/test, npm run build/test/lint, npm test, git status/diff/log/add. Do NOT auto-allow git push, rm, git reset, or anything destructive.
- Add permissions.deny for Read on **/bin/**, **/obj/**, **/node_modules/**, **/dist/**, **/*.min.js, **/package-lock.json (stop wasting context on generated/vendor files).

PHASE 2 — CLAUDE.md hierarchy (the big recurring saving):
- Shrink ~/.claude/CLAUDE.md to ~5 universal lines: "Prefer the simplest working solution; no unrequested abstractions. Be concise. Code/errors verbatim." plus the two Windows quirks above.
- In this repo: split context so it loads on demand. Keep root CLAUDE.md < 200 lines, DECISIONS NOT DESCRIPTIONS (delete anything a competent dev already knows). Create backend/CLAUDE.md and frontend/CLAUDE.md with area-specific rules ONLY (these load only when I work in that folder).
- Add the line "Prefer the simplest working solution; no unrequested abstractions." to the root CLAUDE.md.
- If a /pare-claude-md skill is available, use it on each CLAUDE.md; otherwise apply the same "obviousness" pruning yourself.

PHASE 3 — dashboard-overview skill (stop re-reading my repo every session):
- Read the repo structure (top-level dirs, entry points, how backend/frontend connect, key conventions, ports 5000/5173, start.ps1).
- Create ~/.claude/skills/dashboard-overview/SKILL.md with a tight description ("Architecture & conventions for the claude-dashboard repo; use when navigating or modifying it") and a short body: stack, where things live, run commands, the real gotchas.
- Put bulky reference (full API route list, DB schema, component map) in references/*.md files next to it (Tier 3 — load only when needed), not in the SKILL.md body.

PHASE 4 — slash commands for my repetitive tasks (.claude/commands/ in this repo):
- Create 3–5 reusable command templates that bake in our conventions so I never re-explain them. Suggested: /add-endpoint (backend controller+DTO+test pattern), /add-component (React component + Zustand store + our layout), /run-checks (dotnet build + npm run build, report failures only), /review-diff (review current git diff against our conventions, issues only). Use $ARGUMENTS/$1 for slots.

PHASE 5 — code intelligence (LSP) for C#, Python, TypeScript (semantic nav instead of grep → far fewer file reads):
- Detect which of C#/Python/TypeScript this repo uses (it's at least C# + TS).
- Install the language-server BINARIES (Windows-aware):
  - TypeScript: `npm install -g typescript-language-server typescript`
  - Python: `npm install -g pyright` (node-based; point it at the Python312 interpreter via a pyrightconfig if needed — do NOT rely on `python3`).
  - C#: `dotnet tool install -g csharp-ls` (I have .NET 8). If that fails, tell me and suggest the Roslyn/OmniSharp alternative.
- Install the matching Claude Code LSP plugins via /plugin (Discover tab → search "lsp"): typescript, pyright/python, and C#. IMPORTANT: plugin install needs git on PATH and mine isn't — first add D:\Apps_Installers\Git\cmd to PATH for this to work (do it for the session and tell me how to make it permanent). If a plugin still can't install, REPORT it with the manual steps instead of failing.
- Verify each LSP responds (e.g., a go-to-definition / type-check works) and report which languages are live.

PHASE 6 — MCP audit (convert what's convertible, trim the rest):
- Run /mcp and /context to list my MCP servers (n8n, playwright, tends2, supabase, vapi) and what each costs in context.
- For EACH, recommend one of: KEEP (live API, needed now), DISABLE-WHEN-UNUSED, PREFER-CLI, or WRAP-IN-SKILL — with a one-line reason. Apply this guidance:
  - Live-action API/browser tools (tends2, vapi, n8n workflow ops, playwright) generally CANNOT become skills — keep them, but disable when not in use.
  - supabase: prefer the Supabase CLI for routine ops (zero per-tool overhead). If the CLI isn't installed, tell me the install command.
  - For any recurring multi-step MCP workflow I do often (e.g. tends2 client onboarding), create a SKILL that documents and sequences the steps ("skill wrapping MCP") so the methodology is a cheap on-demand skill and only the connectivity stays in MCP.
- Implement only the safe changes (create the wrapper skills, note CLI swaps). Don't delete MCP servers — just recommend which to disable via /mcp and tell me.

PHASE 7 — offload verbose output to a hook (Windows-safe):
- Add a PreToolUse(Bash) hook to ~/.claude/settings.json that filters test output to failures only (so a big test run returns hundreds of tokens, not thousands). Implement it so it runs through my Git bash (D:\Apps_Installers\Git\usr\bin\bash.exe) and degrades gracefully if the shell isn't found. Keep it minimal and non-blocking.

PHASE 8 — verify & report:
- Run /context before-and-after where possible and report the token delta from the changes.
- Print a concise summary table: what changed, what was skipped (and why), and the 3 things I should do manually (e.g., make the git PATH change permanent, run /model Sonnet to confirm, disable idle MCP servers via /mcp).
- Do NOT claim success for anything you didn't verify.

Constraints: keep quality high — never remove validation, error handling, security, or accessibility guidance from CLAUDE.md; only remove the OBVIOUS. Prefer reversible changes. Show me the plan first.
```

---

## Why these steps (research-backed, biggest-cost-first)

| Step | Why it saves (evidence) |
|---|---|
| Sonnet default + Concise + effort | Opus is priciest; Sonnet fine for most coding; routing studies ~40–80%. (Anthropic *Manage costs*.) |
| permissions.deny (junk reads) | Stops Claude pouring lockfiles/node_modules into context — real token waste. (Anthropic.) |
| permissions.allow (safe cmds) | Fewer permission interrupts/retried turns — mostly speed, some token. Use `/fewer-permission-prompts` to extend it. |
| Tiered + pared CLAUDE.md | Re-sent every session; we measured pare = −72% / −580 tok/session; subdir files load only on demand. (Anthropic <200 lines.) |
| dashboard-overview skill | Claude has **no codebase index** — it re-greps/re-reads every session; a skill loads ~100 tokens until needed vs reading 8–10 files. Progressive disclosure = up to **140×** vs upfront. |
| Slash commands | Repeated workflows measured **3,200 → 850 tokens/run**. |
| **LSP / code intelligence** | Semantic nav ~**50ms vs 30–60s grep**, drastically **fewer file reads** (the dominant input cost). Anthropic ships TS/Python/C# LSP plugins. |
| **MCP audit** | A 5-server MCP setup can cost ~**55k tokens**; skills are ~**30–50 tokens** on-demand. Native deferral helps, but disabling unused + CLI-over-MCP + skills-wrapping-MCP cuts more. |
| Output-filter hook | Anthropic's pattern: 10k-line log → hundreds of tokens. (Native, Windows-safe version of what squeez couldn't do.) |

**Honest caveats baked into the prompt:** your `git`-not-on-PATH blocks plugin installs until fixed (the prompt adds it); `python3` is broken so Python LSP uses the node `pyright` binary; and your MCPs are live-action tools, so they mostly **can't** become skills — the gain is disable-when-unused + CLI + skill-wrapped workflows, not deletion.

### Sources
- Anthropic LSP/code-intelligence plugins (TS/Python/C#) — https://claude.com/plugins/typescript-lsp · https://claude.com/plugins/pyright-lsp · https://github.com/Piebald-AI/claude-code-lsps ; LSP 50ms-vs-grep — https://zircote.com/blog/2025/12/lsp-tools-plugin-for-claude-code/ · https://www.amazingcto.com/lsp-in-claude/
- Skills vs MCP (MCP ~55k vs skills 30–50 tok; "skills wrapping MCP"; MCP for connectivity, skills for methodology) — https://dev.to/jimquote/claude-skills-vs-mcp-complete-guide-to-token-efficient-ai-agent-architecture-4mkf · https://www.verdent.ai/guides/claude-skills-vs-mcp · https://claude.com/blog/extending-claude-capabilities-with-skills-mcp-servers
- Anthropic *Manage costs* (model, hooks-filter, CLAUDE.md<200, CLI-over-MCP, subagents) — https://code.claude.com/docs/en/costs
- Skills progressive disclosure / slash commands savings — https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview · https://code.claude.com/docs/en/slash-commands
- Our measured results — `token-tools-report.md`, `local-setup-context-engineering.md` (this repo)
