# One-Paste Setup Prompt for Claude Code (portable)

**How to use:** open Claude Code **in the root of the repo you want to optimise**, paste the whole code block below, and send. It **detects your environment** (OS, shell, git, package managers, languages, installed language servers, and whatever MCP servers you actually have) and applies the proven token/cost wins — safely, with a plan and confirmation first. Nothing is hard-coded to a specific machine, repo, or MCP set.

> It changes global config and repo files, so it backs up what it overwrites and shows a plan before writing. It **verifies** each step and **reports** anything it can't do cleanly (missing binary, no git, OS quirk) instead of failing silently or pretending it worked.

---

```text
You are setting up my current Claude Code environment and THIS repo for maximum token/cost efficiency WITHOUT losing quality. Discover everything — do not assume my OS, paths, languages, or MCP servers. Work in phases. Before writing anything, print a short PLAN and ask me to confirm. Back up any file you overwrite to <file>.bak. Verify each change; if something can't be done cleanly, SKIP it and REPORT why with manual steps — never claim success for something you didn't verify.

PHASE 0 — DETECT (report a short findings list, don't change anything yet):
- OS + default shell; whether `git` is available (and if not, where it is / how to add it to PATH).
- Package managers / runtimes present: npm/pnpm, dotnet, python/py/uv, go, cargo, etc.
- This repo's stack: scan for project files (package.json, *.csproj/*.sln, pyproject.toml/requirements.txt, go.mod, Cargo.toml, etc.) and the main languages by file extension.
- Build/test/lint commands for this repo (from package.json scripts, *.csproj, Makefile, etc.).
- Natural module boundaries (e.g. separate sub-projects/workspaces) — note them for Phase 2.
- Existing config: ~/.claude/settings.json, any CLAUDE.md files, ~/.claude/skills, .claude/commands.

PHASE 1 — settings.json (~/.claude/settings.json, MERGE, don't clobber):
- "model": "sonnet" (default to Sonnet, not Opus).
- "outputStyle": "Concise" (if the key is unsupported on this version, run /output-style Concise instead and tell me).
- permissions.allow: ONLY the safe, non-destructive build/test/lint/read-only-git commands you actually detected for this repo (e.g. build, test, lint, git status/diff/log/add). NEVER auto-allow destructive commands (push, rm, reset, drop, deploy).
- permissions.deny: Read on generated/vendor paths for THIS repo's stack (e.g. build output dirs, dependency dirs, minified files, lockfiles) so context isn't wasted on junk.

PHASE 2 — CLAUDE.md hierarchy (the big recurring saving — re-sent every session):
- Shrink ~/.claude/CLAUDE.md to a few UNIVERSAL lines only (e.g. "Prefer the simplest working solution; no unrequested abstractions. Be concise. Keep code blocks and exact errors verbatim."). Add any genuinely machine-universal quirks you detected (e.g. tool not on PATH).
- Root CLAUDE.md for this repo: keep < 200 lines, DECISIONS NOT DESCRIPTIONS — delete anything a competent engineer or capable AI already knows; keep only non-obvious, project-specific facts/gotchas/commands. Add the "Prefer the simplest working solution; no unrequested abstractions." line.
- For each natural module boundary you found in Phase 0, create a subdirectory CLAUDE.md (e.g. <module>/CLAUDE.md) with that area's rules ONLY — these load on demand only when I work in that folder.
- If a /pare-claude-md skill exists, use it on each CLAUDE.md; otherwise apply the same "obviousness" pruning yourself. NEVER remove validation, error-handling, security, or accessibility guidance — only the obvious.

PHASE 3 — project-overview skill (stop re-reading this repo every session; Claude has no codebase index):
- Read the repo to learn its architecture, entry points, how modules connect, run commands, and the real conventions/gotchas.
- Create ~/.claude/skills/<repo-name>-overview/SKILL.md with a tight description ("Architecture & conventions for the <repo-name> repo; use when navigating or modifying it") and a short body (stack, where things live, run commands, gotchas).
- Put bulky reference (full route/endpoint list, schema, component/module map) in references/*.md next to it (loads only when needed), NOT in the SKILL.md body.

PHASE 4 — slash commands for repetitive tasks (.claude/commands/ in this repo):
- Create 3–5 reusable command templates that bake in this repo's actual conventions/commands so I never re-explain them. Derive them from what the repo actually does (e.g. add a module/component/endpoint following the existing pattern; run-checks = build+test, report failures only; review-diff = review current git diff against our conventions, issues only). Use $ARGUMENTS/$1 for slots.

PHASE 5 — code intelligence (LSP) — CHECK before installing (semantic nav instead of grep = far fewer file reads):
- For EACH main language you detected, check whether a language server is already installed and whether a matching Claude Code LSP plugin is available (run /plugin, Discover tab, search "lsp").
- Install only the MISSING language-server binaries, using the right package manager for THIS machine, e.g.:
  - TypeScript/JS → `npm install -g typescript-language-server typescript`
  - Python → prefer the node-based `pyright` (`npm install -g pyright`) if `python3` isn't reliable; otherwise `pip install pyright`. Point it at the real interpreter if needed.
  - C#/.NET → `dotnet tool install -g csharp-ls` (or Roslyn/OmniSharp if that fails).
  - Other languages → the standard server (gopls, rust-analyzer, etc.).
- Installing LSP PLUGINS via /plugin needs git on PATH. If git isn't on PATH, add it (or tell me exactly how) BEFORE installing plugins. If a plugin still can't install, REPORT it with manual steps rather than failing.
- VERIFY each LSP actually responds (a go-to-definition / type-check works) and report which languages are live vs which need manual follow-up. Do not assume any language or server is present.

PHASE 6 — MCP audit (discover what I actually have; convert what's convertible, trim the rest):
- Run /mcp and /context to list MY configured MCP servers and what each costs in context. Do NOT assume any specific servers.
- For EACH server, recommend one of: KEEP (live external system needed now), DISABLE-WHEN-UNUSED, PREFER-CLI (a CLI exists that's more context-efficient, e.g. gh, supabase, aws), or WRAP-IN-SKILL — with a one-line reason. Apply this rule of thumb: live API/DB/browser access generally must STAY as MCP (skills can't call live APIs); but a recurring multi-step workflow over an MCP can be captured as a SKILL that sequences it ("skill wrapping MCP"), and any server with a good CLI is cheaper as a CLI.
- Implement only the SAFE changes: create the wrapper skills for recurring workflows, note CLI swaps and how to install the CLI. Do NOT delete MCP servers — tell me which to disable via /mcp and why.

PHASE 7 — offload verbose output to a hook (OS/shell-aware):
- Add a PreToolUse hook (matching the test/build runner you detected) that filters output to failures/errors only, so big runs return hundreds of tokens, not thousands. Use a shell that exists on THIS machine; make it non-blocking and degrade gracefully if the shell isn't found.

PHASE 8 — verify & report:
- Where possible, capture /context before-and-after and report the token delta.
- Print a concise summary table: what changed, what was skipped (and why), which LSPs are live, the MCP recommendations, and the manual follow-ups I still need to do (e.g. make a PATH change permanent, confirm /model Sonnet, disable idle MCP servers via /mcp).
- Keep quality high throughout; prefer reversible changes; show me the plan before writing.
```

---

## Why these steps (research-backed, biggest-cost-first)

| Step | Why it saves (evidence) |
|---|---|
| Sonnet default + Concise + effort | Opus is priciest; Sonnet fine for most coding; routing studies ~40–80%. (Anthropic *Manage costs*.) |
| permissions.deny (junk reads) | Stops Claude pouring lockfiles/deps into context — real token waste. |
| permissions.allow (safe cmds) | Fewer permission interrupts/retried turns — mostly speed, some token. (`/fewer-permission-prompts` extends it.) |
| Tiered + pared CLAUDE.md | Re-sent every session; pare measured −72% / −580 tok/session; subdir files load only on demand. (Anthropic <200 lines.) |
| project-overview skill | Claude has **no codebase index** — it re-greps/re-reads every session; a skill loads ~100 tokens until needed. Progressive disclosure up to **140×** vs upfront. |
| Slash commands | Repeated workflows measured **3,200 → 850 tokens/run**. |
| **LSP / code intelligence** | Semantic nav ~**50ms vs 30–60s grep** → drastically **fewer file reads** (the dominant input cost). Anthropic ships TS/Python/C# (and more) LSP plugins. |
| **MCP audit** | A 5-server MCP setup can cost ~**55k tokens**; skills ~**30–50 tokens** on-demand. Native deferral helps; disable-unused + CLI-over-MCP + skills-wrapping-MCP cut more. |
| Output-filter hook | Anthropic's pattern: 10k-line log → hundreds of tokens. |

**Built-in honesty:** the prompt detects rather than assumes — it checks whether git, language servers, and CLIs exist before acting, audits *your* actual MCP servers, and reports anything it can't verify. Live-action MCP servers generally **can't** become skills; the gain there is disable-when-unused + CLI + skill-wrapped workflows.

### Sources
- Anthropic *Manage costs* (model, hooks-filter, CLAUDE.md<200, CLI-over-MCP, subagents) — https://code.claude.com/docs/en/costs
- LSP/code-intelligence plugins (TS/Python/C#+; 50ms-vs-grep) — https://claude.com/plugins/typescript-lsp · https://claude.com/plugins/pyright-lsp · https://github.com/Piebald-AI/claude-code-lsps · https://zircote.com/blog/2025/12/lsp-tools-plugin-for-claude-code/
- Skills vs MCP (MCP ~55k vs skills 30–50 tok; "skills wrapping MCP"; MCP for connectivity, skills for methodology) — https://dev.to/jimquote/claude-skills-vs-mcp-complete-guide-to-token-efficient-ai-agent-architecture-4mkf · https://www.verdent.ai/guides/claude-skills-vs-mcp
- Skills progressive disclosure / slash commands savings — https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview · https://code.claude.com/docs/en/slash-commands
- Our measured results — `token-tools-report.md`, `local-setup-context-engineering.md` (this repo)
