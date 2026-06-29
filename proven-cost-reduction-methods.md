# Proven Ways to Cut Claude Code Cost & Tokens — Research-Backed, For Your Setup

**Your setup (what this is tuned for):** Windows 11, Claude Code 2.1.x, default model **Opus 4.8** (expensive), **coding-heavy** work, several **MCP servers** connected (n8n, playwright, tends2, supabase, vapi), `git` not on PATH, no working `python3`.

**How to read the evidence grade:**
- 🟢 **A — First-party / measured:** Anthropic docs or a controlled benchmark with numbers.
- 🟡 **B — Independent benchmark:** a reproducible third-party (or our own) measured test.
- 🔵 **C — Widely-reported practice:** consistent community reports, mechanism sound, not formally benchmarked.

> **Two facts that decide what works (both independently confirmed):**
> 1. **Cost is dominated by *input* tokens — mostly re-sent/cached context**, not Claude's output. (Anthropic pricing + token-tracing studies.) So shrinking *context* beats shrinking *replies*.
> 2. **A cache-read costs ~10% of a fresh input token.** Keeping the context *stable* (so it stays cached) is one of the biggest levers there is.

---

## The ranked method set (20, evidence-graded)

### Tier 1 — Biggest, proven levers (do these)

| # | Method | Evidence | Backing / measured saving |
|---|---|---|---|
| 1 | **Right-size the model: don't run Opus for everything** — set **Sonnet** as default in `/config`, switch with `/model`, reserve Opus for genuinely hard reasoning | 🟢 A | Anthropic *Manage costs* doc explicitly recommends this. Haiku is ~**15× cheaper** than Opus per token; one measured user had **93.8% of tokens on Opus with zero need**. Model routing studies show **~51% saving** (3-tier) up to 80% claims. |
| 2 | **Protect prompt caching — keep context stable, avoid edits to early context mid-task** | 🟢 A | Anthropic: cache hit = **10% of input price**, up to **90% cost / 85% latency** reduction on repeated context. Claude Code caches system prompt + tools + file reads automatically; you preserve it by not churning early context. |
| 3 | **`/clear` between unrelated tasks; `/compact` (with focus) when mid-flow** | 🟢 A + 🔵 C | Anthropic-recommended. Measured: a 70k-token conversation `/compact`s to **~4k** (and `/clear` frees nearly all of it). Rule: if you'd open a new doc for it, `/clear`. |
| 4 | **Slim CLAUDE.md (< 200 lines); move workflow detail into Skills** — incl. running **pare-claude-md** | 🟢 A + 🟡 B | Anthropic says keep CLAUDE.md < 200 lines and move specialized instructions to on-demand Skills. **We measured** pare-claude-md cut a bloated CLAUDE.md 72% → **−580 tokens *every session*** (more for bigger files). |
| 5 | **Lower thinking/effort for routine work** — `/effort` low/medium, or `MAX_THINKING_TOKENS=8000`, or disable thinking in `/config` for simple tasks | 🟢 A | Anthropic: thinking tokens billed as **output** (the pricier kind), default budget can be *tens of thousands* per request. Use high/max only for hard problems. |

### Tier 2 — High value, especially for *your* MCP-heavy, coding setup

| # | Method | Evidence | Backing / measured saving |
|---|---|---|---|
| 6 | **Disable MCP servers you aren't using right now** (`/mcp`) | 🟢 A | You run 5 MCP servers (n8n, playwright, tends2≈40 tools, supabase, vapi). Even with deferral, names + occasional schema loads add up. Anthropic: disable unused servers; `/context` shows the cost. |
| 7 | **Native MCP tool-search deferral (already ON) — don't pay to "fix" it** | 🟢 A | Anthropic: tool defs **deferred by default** in CC 2.1.7+, **~77k→8.7k (85%)**. *We verified* this makes paid "MCP compressors" (mcp-compressor) redundant on normal tasks. **Action: nothing — just don't buy a tool to redo it.** |
| 8 | **Prefer CLI tools over MCP servers where one exists** (`gh`, `supabase` CLI, `aws`, etc.) | 🟢 A | Anthropic: CLIs add **zero per-tool listing** overhead vs MCP. You have a Supabase MCP *and* the Supabase CLI exists — favor the CLI for routine ops. |
| 9 | **Filter verbose tool output with a PreToolUse hook** (return only test failures / `ERROR` lines) | 🟢 A | Anthropic ships this exact pattern (grep test output → hundreds of tokens instead of tens of thousands). ⚠️ *Windows note:* their example is a **bash** script — works for you via your Git-bash, but mind the shell path; this is the **safe, native version of what squeez tried to do** (and squeez failed to install on your machine). |
| 10 | **ponytail (or a one-line YAGNI prompt) for coding** | 🟡 B | **We measured −21% cost / −8% output / fewer turns**, task still correct. ponytail's official benchmark: **−20% cost, −22% tokens**. Caveat (Scott Logic): a free *"follow YAGNI, prefer one-liners"* system note may capture most of it — try that first, it's zero-install. |
| 11 | **Write specific prompts; use Plan mode for big tasks; course-correct early** | 🟢 A | Anthropic: vague "improve this codebase" triggers broad scans; specific prompts minimize file reads. Plan mode (Shift+Tab) prevents expensive wrong-path rework; Esc/`/rewind` stops waste fast. |
| 12 | **Delegate verbose operations to subagents (model: haiku)** — tests, log scans, doc fetches | 🟢 A | Anthropic: verbose output stays in the subagent's context; only a summary returns. ⚠️ *Not free* — independent testing shows subagent-heavy flows can hit **~7× tokens**; use only when the isolated volume is large. Set `model: haiku` for the subagent. |

### Tier 3 — Worth it situationally / measurement & hygiene

| # | Method | Evidence | Backing / note |
|---|---|---|---|
| 13 | **Measure first: `ccusage` + `/usage` breakdown** (by skill/subagent/plugin/MCP server) | 🟢 A + 🔵 C | Anthropic's `/usage` now attributes usage per MCP server/skill/plugin (v2.1.174+). `ccusage` (npx, reads local logs) gives daily/session trends + cache-hit rate. *You can't improve what you don't measure* — start here to find **your** waste. |
| 14 | **Code-intelligence plugins (symbol navigation)** for your TS/.NET code | 🟢 A | Anthropic: "go to definition" replaces grep-then-read-several-files; fewer unnecessary reads. Good fit for your React/TS + .NET dashboard. |
| 15 | **Context editing + memory tool** (auto-trim stale tool results) | 🟢 A | Anthropic measured **−84% tokens** on a 100-turn web-search agent, **+39%** task performance with memory. Mostly an API/agent-SDK feature today; relevant if you build agents (you do — tends2/vapi). |
| 16 | **Test incrementally & give verification targets** (paste expected output, test cases) | 🟢 A | Anthropic: when Claude can verify its own work it avoids fix-it round-trips (each round = full context re-sent). |
| 17 | **Avoid Agent Teams unless the task truly needs parallelism** | 🟢 A | Anthropic: agent teams use **~7× tokens** (each teammate = own context). Powerful but expensive; keep teammates few, on Sonnet, short-lived. |
| 18 | **caveman / terse-output — only if you do prose/chat, not coding** | 🟡 B | 3 independent tests: ~**61–68%** on prose but only **~4–10%** whole-session on coding; **we measured it slightly *negative* on coding.** Skip for your coding work; keep in mind for doc/PR-summary chat. |
| 19 | **Tool-output compressors (squeez/headroom/rtk) — Mac/Linux/WSL only** | 🟡 B | Help on **huge** command logs (up to 60–95%). **We found squeez won't install on your Windows** (needs python3) and **headroom won't build**; RTK's own docs say "use WSL." **Skip on native Windows.** |
| 20 | **Set spend guardrails** — `/usage-credits` (Pro/Max) or workspace spend/rate limits (API/teams) | 🟢 A | Anthropic: caps runaway cost. Useful once you've baselined with #13. |

---

## Your tailored action plan (in order)

**Today (zero/low effort, highest return):**
1. **`/config` → set default model to Sonnet**, and consciously `/model opus` only for hard reasoning. *(Biggest single lever given you're defaulted to Opus.)*
2. **Run `pare-claude-md` on your CLAUDE.md files** (dashboard repo, any project ones) — permanent per-session saving, no downside. *(Proven in our test.)*
3. **`/mcp` → disable the MCP servers you're not currently using.** With 5 connected (tends2 alone ≈40 tools), keep only what the current task needs.
4. **Add a one-line YAGNI note** to your CLAUDE.md (`Prefer the simplest working solution; no unrequested abstractions`) — captures most of ponytail's win for free. Add the **ponytail hooks** too if you want the measured version (install via its hook files, not the plugin — your `git` isn't on PATH).
5. **Install `ccusage`** (`npx ccusage`) and check the `/usage` per-server breakdown — find *your* actual waste before optimizing further.

**This week (habits + light setup):**
6. **`/clear` between unrelated tasks; `/compact` mid-flow** when context climbs (~40–50% full).
7. **Lower effort for routine edits** (`/effort medium` or low); reserve high/max for genuinely hard problems.
8. **Prefer CLIs over MCP** where you have both (e.g. `supabase` CLI, `gh`).
9. **Add the PreToolUse test-output filter hook** (native, Anthropic's pattern) — point it at your Git-bash; this is the safe Windows-friendly version of what squeez couldn't do.
10. **Use Plan mode** (Shift+Tab) before big multi-file changes; **delegate test runs / log scans to a `model: haiku` subagent.**

**Avoid on your machine:**
- **squeez, headroom, rtk** (don't run / won't install on native Windows — proven in our test; use WSL if you ever want them).
- **caveman** for coding (slightly net-negative for you).
- **mcp-compressor** (Claude Code already defers MCP tools — redundant).
- **Agent Teams** for routine work (~7× tokens).

---

## What the evidence says you'll actually save
- **Model right-sizing (Sonnet default + Haiku subagents)** is your single biggest lever — independent routing studies show **~40–80%** on the portion of work that doesn't need Opus, and you're currently defaulted to the most expensive model.
- **Caching discipline + `/clear`/`/compact` + slim CLAUDE.md** attack the **input/context** cost that dominates the bill — Anthropic-measured up to **90%** on repeated context.
- **ponytail/YAGNI** is the one coding-output lever that's actually proven (**~−20%**); everything else output-side (caveman) is marginal for coding.
- **The MCP "compression" tools are mostly solved for you already** (native deferral) or **don't run on Windows** — don't spend money or setup time there.

> **If you do only three things:** (1) stop defaulting to Opus, (2) run pare-claude-md + keep CLAUDE.md lean, (3) `/clear` between tasks and disable unused MCP servers. Those three hit the dominant cost (input/context) with first-party evidence behind every one.

---

### Sources
- Anthropic, **Manage costs effectively** (Claude Code docs) — model selection, `/clear`+`/compact`, MCP deferral, CLI-over-MCP, hooks-filter, CLAUDE.md<200/skills, effort/thinking, subagents, plan mode, agent-teams 7×, specific prompts — https://code.claude.com/docs/en/costs
- Anthropic, **Prompt caching** (90% / 85% latency; 10% cache-read price) — https://www.anthropic.com/news/prompt-caching · https://platform.claude.com/docs/en/build-with-claude/prompt-caching
- Anthropic, **Advanced tool use / MCP Tool Search** (deferred by default, 77k→8.7k, 85%) — https://www.anthropic.com/engineering/advanced-tool-use
- Anthropic, **Context management (context editing + memory tool)** (−84% on 100-turn agent, +39% perf) — https://claude.com/blog/context-management
- Anthropic, **Effort / adaptive thinking** (thinking billed as output; effort levels) — https://platform.claude.com/docs/en/build-with-claude/effort
- Model routing: musistudio/**claude-code-router**, 0xrdan/**claude-router**; routing studies (~51% 3-tier; Haiku ~15× cheaper) — https://github.com/musistudio/claude-code-router · https://duet.so/guides/claude-opus-vs-sonnet-model-routing
- **ccusage** (local token/cost analysis, cache-hit rate) — https://ccusage.com/ · https://github.com/ryoppippi/ccusage
- `/clear` vs `/compact` (70k→4k) — https://www.mindstudio.ai/blog/how-to-stop-burning-through-claude-code-tokens-context-management-guide-beginners
- Subagent cost caution (~7×) + isolation — https://www.mindstudio.ai/blog/claude-code-sub-agents-explained
- "Where do your Claude Code tokens go" (input dominates; system prompt ≈14.3k; cache 10× cheaper) — https://dev.to/slima4/where-do-your-claude-code-tokens-actually-go-we-traced-every-single-one-423e
- ponytail benchmark (−20% cost) + Scott Logic critique (YAGNI one-liner) — https://github.com/DietrichGebert/ponytail/blob/main/benchmarks/results/2026-06-18-agentic.md · https://blog.scottlogic.com/2026/06/16/ponytail-yagni-and-the-problem-with-prompt-benchmarks.html
- caveman independent tests (prose 61–68%, coding 4–21%) — https://dev.to/jakguzik/i-benchmarked-the-viral-caveman-prompt-to-save-llm-tokens-then-my-6-line-version-beat-it-2o81
- RTK Windows note ("use WSL") — https://github.com/rtk-ai/rtk
- *Our own measured results:* `token-tools-report.md`, `validation-against-independent-research.md` (this repo)
