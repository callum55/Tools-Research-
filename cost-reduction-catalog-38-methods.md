# The Big Catalog: 38 Evidence-Backed Ways to Cut Claude Code Cost

> **Read this first.** The earlier plugin test did **not** say "cost can't be reduced." It said *seven specific plugins* mostly underperformed on Windows/coding. That's the opposite of the truth about cost overall: **there are dozens of proven levers**, several with first-party Anthropic measurements showing **50–98% reductions**. This catalog collects 38 of them, each with an evidence grade and the research behind it.

**Evidence grades:** 🟢 **A** = Anthropic first-party or a controlled benchmark with numbers · 🟡 **B** = independent reproducible benchmark (incl. our own) · 🔵 **C** = consistent community practice, sound mechanism, not formally benchmarked.

**The one principle behind almost all of it:** Claude Code's bill is dominated by **input/context tokens that get re-sent every turn**, and a cached token costs ~**10%** of a fresh one. So the highest-leverage moves either (a) make context smaller, (b) keep it cached/stable, or (c) run it on a cheaper model. Output-shrinking is the *weakest* category — which is exactly why the output-compressor plugins disappointed.

---

## A. Model & inference economics — *usually the biggest single lever*

| # | Method | Grade | Evidence / measured |
|---|---|---|---|
| 1 | **Stop defaulting to Opus; make Sonnet the default** (`/config`, `/model`) | 🟢 A | Anthropic recommends Sonnet for most coding. Haiku ~**15× cheaper** than Opus; one user had **93.8% of tokens on Opus** needlessly. |
| 2 | **Automatic model routing** (claude-code-router / claude-router) — route by complexity | 🟡 B | Measured **~51%** saving on a 3-tier setup; vendor claims 40–80%. |
| 3 | **Haiku subagents for grunt work** (`model: haiku` in subagent config) | 🟢 A | Anthropic-native; offload search/test/lint to Haiku while Opus/Sonnet drives. |
| 4 | **Lower effort / thinking budget for routine work** (`/effort`, `MAX_THINKING_TOKENS`, disable thinking in `/config`) | 🟢 A | Thinking tokens billed as **output** (the pricey kind), default budget can be *tens of thousands*/request. |
| 5 | **Per-skill effort** (`effort: low` frontmatter on routine skills) | 🟢 A | Anthropic added skill-level effort; `effort: low` cuts usage on formatting/lint/simple reviews "without noticeable quality loss." |
| 6 | **Batch API for async/bulk work** (evals, migrations, bulk gen) | 🟢 A | **50% off input *and* output**; **stacks with caching → ~95%** total. |
| 7 | **Cap response length** (`max_tokens`, or "be concise" system rule) | 🟢 A | Anthropic output-control guidance: hard cap first, then verbosity rules. |
| 8 | **Use 1M context as single-shot, not many turns** (Opus only) | 🟢 A | Long-context **premium pricing was removed** (Mar 2026) for Opus 4.6+/Sonnet 4.6 — but single big request still beats re-sending context per turn; Opus actually uses long context (76% MRCR vs Sonnet 18.5%). |
| 9 | **(Caution) Free/cheap third-party models via OpenRouter** | 🔵 C | Claims up to "99% cost cut," but **real quality risk** on coding — treat as experiment, not default. |

## B. Caching & context stability — *the cheap-token lever*

| # | Method | Grade | Evidence / measured |
|---|---|---|---|
| 10 | **Maximize prompt caching** — keep system prompt, tools, early context **stable**; don't churn early messages mid-task | 🟢 A | Cache hit = **10% of input price**; up to **90% cost / 85% latency** reduction. Claude Code caches automatically — your job is not to break it. |
| 11 | **`/clear` between unrelated tasks** | 🟢 A | Anthropic-recommended; frees nearly all conversation context. "If you'd open a new doc, `/clear`." |
| 12 | **`/compact` with focus instructions mid-flow** (`/compact focus on code + tests`) | 🟢 A | Measured **70k → ~4k** tokens; add compact rules to CLAUDE.md. |
| 13 | **Context editing + memory tool** (auto-trim stale tool results; external memory files) | 🟢 A | Anthropic measured **−84% tokens** (100-turn agent) and **+39%** task performance with memory. |
| 14 | **Let auto-compaction work; tune the context buffer** | 🔵 C | Auto-compacts ~83% capacity; reserve ~33k buffer. Awareness avoids surprise re-sends. |

## C. Shrinking what enters the context window — *biggest input-side category*

| # | Method | Grade | Evidence / measured |
|---|---|---|---|
| 15 | **Code execution with MCP** (agent writes code against MCP "APIs" instead of raw tool calls) | 🟢 A | Anthropic engineering: **150k → 2k tokens (98.7%)** on an example workflow. (This is the *validated* version of what context-mode attempts.) |
| 16 | **Native MCP tool-search deferral — already on; don't pay to redo it** | 🟢 A | Default in CC 2.1.7+: **77k → 8.7k (85%)**. *We verified* paid MCP-compressors are redundant for this. |
| 17 | **Disable unused MCP servers** (`/mcp`) | 🟢 A | You run 5 (n8n, playwright, **tends2 ≈40 tools**, supabase, vapi). Keep only what the task needs; `/context` shows the cost. |
| 18 | **Prefer CLIs over MCP servers** (`gh`, `supabase` CLI, `aws`) | 🟢 A | CLIs add **zero per-tool listing** vs MCP. You have Supabase MCP *and* CLI — favor CLI for routine ops. |
| 19 | **Slim CLAUDE.md < 200 lines; run `pare-claude-md`** | 🟢 A + 🟡 B | Anthropic: keep < 200 lines. **We measured** pare cut a file 72% → **−580 tokens/session** (more for bigger files). |
| 20 | **Move workflow detail from CLAUDE.md into on-demand Skills** | 🟢 A | CLAUDE.md loads every session; skills load only when invoked → smaller base context. |
| 21 | **Exclude junk from context** — `permissions.deny` reads, ignore `node_modules`/build/dist/vendor | 🔵 C | Anthropic settings support deny rules; community uses them to stop Claude wasting context on generated/vendor files. (`.claudeignore` still a feature request.) |
| 22 | **Code-intelligence plugins (symbol navigation)** for TS/.NET | 🟢 A | "Go to definition" replaces grep-then-read-several-files → fewer unnecessary reads. |
| 23 | **Targeted reads, not whole files** — grep/glob + offset/limit; `@`-mention exact files | 🟢 A | Anthropic: specific prompts/targets minimize broad scanning and full-file reads. |
| 24 | **PreToolUse hook to filter verbose output** (return only test failures / `ERROR` lines) | 🟢 A | Anthropic ships this pattern: 10k-line log → hundreds of tokens. *(Windows: runs via your Git-bash; the safe native version of what squeez couldn't do.)* |
| 25 | **Pre-process data with scripts, feed Claude the result** | 🔵 C | Compute/aggregate outside the model; send the summary, not the raw dump. |
| 26 | **(Large-output workflows only) tool-output compressors** (squeez/headroom/rtk) | 🟡 B | Up to 60–95% on **huge** logs — but **won't run on your native Windows** (proven); WSL/Mac/Linux only. |

## D. Output / generation reduction — *real but smallest category for coding*

| # | Method | Grade | Evidence / measured |
|---|---|---|---|
| 27 | **ponytail or a one-line YAGNI rule** ("simplest working solution; no unrequested abstractions") | 🟡 B | **We measured −21% cost** / −8% output / fewer turns, correct. Official benchmark **−20% cost**. A free YAGNI line captures much of it. |
| 28 | **Output style: Concise** (`/output-style`) | 🟢 A | Anthropic built-in; trims response verbosity (avoid Explanatory/Learning styles, which *increase* output). |
| 29 | **caveman / terse output — prose/chat only, not coding** | 🟡 B | 3 independent tests: **61–68%** on prose, **4–21%** on coding; we saw it slightly **negative** on coding. |

## E. Workflow & process — *cuts the expensive "wrong path → redo" loops*

| # | Method | Grade | Evidence / measured |
|---|---|---|---|
| 30 | **Plan mode before big changes** (Shift+Tab) | 🟢 A | Prevents expensive wrong-path rework; explore + approve before editing. |
| 31 | **Spec-driven / plan-first development** | 🟡 B | "Agent loops without specs are **quadratic** (resends growing history)"; specs front-load thinking → **~20%** token cut, **40–70%** combined; one team **−75%** cycle time. |
| 32 | **Skills / slash commands for repeated workflows** (encode once, call by name) | 🟡 B | Measured **3,200 → 850 tokens/run** for a repeated workflow; skills cut context bloat **60–80%**. |
| 33 | **Specific prompts, not "improve this codebase"** | 🟢 A | Vague prompts trigger broad scans; specific ones minimize reads. |
| 34 | **Give verification targets + test incrementally** | 🟢 A | Self-verifiable work avoids fix-it round-trips (each round = full context re-sent). |
| 35 | **Course-correct early** (Esc, `/rewind`, double-Esc checkpoint) | 🟢 A | Stop wrong direction immediately instead of paying for more turns. |
| 36 | **Use subagents to isolate high-volume ops** (tests, log scans) — but sparingly | 🟢 A | Verbose output stays in subagent; only summary returns. ⚠️ subagent-heavy flows can hit **~7×** tokens — use when isolated volume is large. |

## F. Measurement, governance & billing strategy — *find waste, cap spend, pick the right plan*

| # | Method | Grade | Evidence / measured |
|---|---|---|---|
| 37 | **Measure to find waste** — `ccusage` (npx) + `/usage` per-component breakdown (skill/subagent/plugin/**MCP server**) + Claude-Code-Usage-Monitor | 🟢 A + 🔵 C | `/usage` attributes usage per MCP server (v2.1.174+); ccusage shows cache-hit rate + cost/session. *Can't optimize what you don't measure.* |
| 38 | **Right billing model: subscription vs API** (+ spend caps) | 🟢 A | **Max 20x ($200) ≈ $600–1,500 of API value**; subscriptions run **15–30× cheaper** than per-token for daily heavy use. Add `/usage-credits` or workspace spend/rate limits as guardrails. |

---

## Your top 10 for *this* setup (Windows · Opus default · 5 MCP servers · coding)

1. **Make Sonnet your default** (`/config`); `/model opus` only for hard reasoning. *(Likely your single biggest saving — you're defaulted to the priciest model.)*
2. **Confirm you're on a Max subscription, not API pay-per-token** — for daily coding this alone is a 15–30× cost structure difference. Add a spend cap.
3. **`pare-claude-md` your CLAUDE.md files + keep them < 200 lines**; move workflow detail to skills. *(Proven, permanent.)*
4. **`/mcp` → disable MCP servers you're not using right now** (tends2 ≈40 tools is a prime candidate when not onboarding).
5. **Prefer the Supabase CLI / `gh` over their MCP servers** for routine ops.
6. **`/clear` between tasks; `/compact focus on code+tests` mid-flow.**
7. **Lower `/effort` for routine edits;** reserve high/max for genuinely hard problems.
8. **Add a one-line YAGNI rule** to CLAUDE.md (free ~20% on coding output); add ponytail's hooks if you want the measured version.
9. **Add the native PreToolUse test-output filter hook** (Windows-safe, via Git-bash) — the legitimate version of tool-output compression.
10. **Install `ccusage` and read `/usage`** weekly to see *your* real waste (which MCP server, which projects) and target the next round.

**Avoid (proven/again):** squeez/headroom/rtk on native Windows · mcp-compressor (deferral already does it) · caveman for coding · Agent Teams for routine work (~7×) · defaulting to 1M context per-turn.

---

## So… can cost be reduced? Yes — substantially.
- **Model right-sizing + subscription billing**: the two structural levers, easily **40–80%+** for someone defaulted to Opus on per-token-equivalent work.
- **Caching + `/clear`/`/compact` + lean CLAUDE.md + disabling MCP**: attack the dominant input/context cost — Anthropic-measured up to **85–90%** on the relevant pieces.
- **Code-execution-with-MCP / context-editing**: **84–98%** in Anthropic's own agent measurements.
- **ponytail/YAGNI**: the one coding-output lever that's actually proven (**~−20%**).

The earlier report's real message wasn't "nothing works" — it was **"don't buy niche output-compressor plugins; spend your effort on the model, the context, and the billing model instead."** That's where the proven savings are.

---

### Sources (by method group)
- **Models/inference:** Anthropic *Manage costs*, *Pricing*, *Effort/Adaptive thinking*, *Batch processing* — code.claude.com/docs/en/costs · platform.claude.com/docs/en/about-claude/pricing · platform.claude.com/docs/en/build-with-claude/effort · platform.claude.com/docs/en/build-with-claude/batch-processing ; routing: github.com/musistudio/claude-code-router ; 1M: claudecodecamp.com/p/claude-code-1m-context-window
- **Caching/context:** Anthropic *Prompt caching* (anthropic.com/news/prompt-caching), *Context management* (claude.com/blog/context-management), *Advanced tool use* (anthropic.com/engineering/advanced-tool-use)
- **Context shrink:** Anthropic *Code execution with MCP* (anthropic.com/engineering/code-execution-with-mcp, 98.7%), *Manage costs* (CLI-over-MCP, hooks-filter, code-intelligence, CLAUDE.md<200); permissions.deny (github.com/anthropics/claude-code issues #29455/#30810); our pare measurement (this repo)
- **Output:** Anthropic *Output styles* (code.claude.com/docs/en/output-styles); ponytail benchmark + our test; caveman independent tests (dev.to/jakguzik/...)
- **Workflow:** Anthropic *Manage costs* (plan mode, specific prompts, verification, subagents); spec-driven (github.blog, arxiv 2602.00180); skills savings (oneaway.io, mindstudio)
- **Measure/billing:** ccusage.com · github.com/ryoppippi/ccusage · github.com/Maciek-roboblog/Claude-Code-Usage-Monitor ; subscription-vs-API (buildthisnow.com Max-vs-API break-even, productcompass.pm, finout.io/blog/claude-code-pricing-2026)
