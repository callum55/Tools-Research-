# Token-cost benchmark — raw findings (accumulating)

## Method
- Isolated `CLAUDE_CONFIG_DIR` per config, seeded only with credentials (clean baseline).
- Bucket A (startup overhead): trivial prompt `"Reply with exactly the word: pong"`, measure first-turn context = input + cache_creation + cache_read.
- Bucket B/C (end-to-end): fixed task (fix slugify bug + add truncate + tests), 3 runs each, `claude -p --output-format json --dangerously-skip-permissions`. Report avg + range of output tokens, total input, cost.
- Sample project reset from pristine before every run.

## BASELINE (clean, no tools) — CONFIRMED
- Bucket A startup: **20,657 tokens**
- E2E (3 runs): turns 9/9/9
  - output: 1572 / 1564 / 1755 -> avg **1,630** (range 1,564-1,755)
  - total input: 180,929 / 179,970 / 181,665 -> avg **180,855**
  - cost: $0.1732 / $0.1713 / $0.1392 -> avg **$0.1612** (range $0.139-$0.173)
  - time: ~38s avg
- Task correctness: PASS (4/4 tests)

## 1. CAVEMAN — v1.9.0, commit 25d22f8 (2026-06-12), 77,959 stars
- Type: skill/plugin; mechanism = SessionStart hook injects terse-mode ruleset (~3,039 chars / ~800 tok). Bucket C (output prose).
- Install: plugin method FAILED on Windows (needs `git`, not on PATH). Hooks method (git-free) worked.
- Bucket A startup: **21,797** vs 20,657 = **+1,140 (+5.5%)**
- E2E (3 runs): turns 9/9/9
  - output: 1520 / 1943 / 1885 -> avg **1,783** (range 1,520-1,943) = **+9% vs baseline**
  - total input: 213,113 / 189,317 / 162,225 -> avg **188,218** = +4%
  - cost: $0.1922 / $0.1504 / $0.1418 -> avg **$0.1615** = ~0% vs baseline
  - time: ~35s avg
- Task correctness: PASS (4/4). Final summary visibly terser but accurate.
- KEY FINDING: On agentic *coding* (Claude Code's main use), output is dominated by code+tool-calls which caveman leaves unchanged; prose savings are swamped by variance + per-turn ruleset re-injection. Net cost ~neutral here. Its advertised ~65% savings apply to prose/Q&A chat, not coding.
- Disadvantage axes:
  - Setup friction: SIGNIFICANT (documented plugin install needs git; failed on this Windows box; hooks fallback works)
  - Broke/hid a needed tool: No
  - Answer-quality degradation: No (task correct; terser but accurate). Mild risk in chat: suppresses tables/explanatory detail.
  - Added latency: Negligible (SessionStart node hook <0.2s; times comparable to baseline)

## 2. HEADROOM — latest v0.28.0 (aea3c35) NOT INSTALLABLE; tested v0.20.15; 53,782 stars
- Type: proxy/library/MCP. Mechanism = local proxy via ANTHROPIC_BASE_URL compresses tool outputs + prose. Buckets B+C.
- BLOCKED ON WINDOWS:
  - v0.27/0.28 sdist build fails: native Rust ext won't compile (MSVC linker "link: extra operand").
  - v0.20.15 wheel installs but missing headroom._core (Rust) -> proxy won't start (ModuleNotFoundError).
  - No Rust toolchain installed; building core needs Rust + MSVC env. Not reasonable for end user.
- Result: proxy never started; ZERO claude sessions run; no measured deltas. $0 tokens spent.
- Vendor ESTIMATE only: 60-95% on tool-heavy workloads (code search 92%). Caveat: default 'token' mode can harm prompt caching; --intercept-tool-results OFF by default in 0.20.15; proxy disables 1M-context beta; documented daemon child-session bypass.
- Disadvantage axes:
  - Setup friction: BLOCKING on Windows (no usable wheel; native build fails)
  - Broke/hid a needed tool: N/A (never ran)
  - Answer-quality degradation: Not measurable
  - Added latency: Not measurable (adds a proxy hop)
- VERDICT: Fails the "must work on Windows" requirement (June 2026, this machine).

## 3. MCP-COMPRESSOR (atlassian-labs) — commit d664676 (2026-06-28), 87 stars
- Type: MCP proxy (uvx mcp-compressor). Wraps a backend MCP, replaces N tool schemas with compact interface. Bucket A.
- Install: MINOR friction. `uvx mcp-compressor` = prebuilt Rust binary, downloaded + ran first try. No git/build needed. (Contrast headroom.)
- Wrap target measured: Playwright MCP (~25 browser tools) via --mcp-config.
- KEY MEASURED FINDINGS (Claude Code 2.1):
  - Baseline (no MCP), trivial prompt: 20,657
  - Raw Playwright MCP, trivial prompt (DEFERRED): 25,813  (+5,156 = just tool NAMES)
  - mcp-compressor med/max, trivial prompt: 25,813 / 25,810  => saving ~0 vs raw
  - Raw MCP when tools forced into context (full schemas): up to 62k-125k
  - Tools-heavy "enumerate all tools" prompt: RAW 125,797 vs COMP_max 90,094 => ~28% / ~35.7k saved (contrived scenario, noisy)
- INTERPRETATION: Claude Code 2.1 ALREADY defers MCP tool schemas (names only until a tool is used). So mcp-compressor's headline "70-97% tool-def reduction" is largely REDUNDANT on Claude Code for normal tasks. It only helps when a task pulls many MCP schemas at once. Atlassian's benchmark was on a client without deferral.
- Disadvantage axes:
  - Setup friction: MINOR (uvx one-liner, prebuilt binary; needs uv/Python which user has)
  - Broke/hid a needed tool: Risk — replaces real tools with generic get_tool_schema/invoke_tool indirection; adds round-trips; model had to "search" for tools. Not broken in test.
  - Answer-quality degradation: Not observed (answers correct)
  - Added latency: Yes, modest — extra uvx proxy hop + on-demand schema fetch per tool call
- VERDICT: Only worth it if you run MANY always-on MCP tools AND disable native deferral; otherwise Claude Code's built-in deferral already does the job for free.

## 4. PONYTAIL — v4.8.4, commit bc9ee94 (2026-06-29), ~67k stars
- Type: plugin + hooks. SessionStart hook injects "lazy senior dev" decision-ladder ruleset (5,852 chars / ~1.5k tok). Bucket C (less over-engineered code).
- Install: plugin method needs git (would fail like caveman); used git-free hooks method (faithful, same SessionStart mechanism).
- Bucket A startup: 22,369 vs 20,657 = +1,712 (+8.3%)
- E2E (3 clean runs; 1 earlier run discarded as contaminated by leftover mcp test files):
  - turns: 8 / 8 / 8  (baseline 9/9/9 -> FEWER turns, less exploration)
  - output: 1500 / 1516 / 1483 -> avg 1,500 (range 1,483-1,516) = -8% vs baseline
  - total input: 191,465 / 191,416 / 191,403 -> avg 191,428 = +5.8% (ruleset re-injected per turn)
  - cost: $0.1448 / $0.0936 / $0.1444 -> avg $0.1276 = -21% vs baseline (NOISY, range $0.094-0.145)
  - time: ~32s avg (faster than baseline ~38s, fewer turns)
- Task correctness: PASS (4/4). Did NOT under-deliver/YAGNI-skip; built exactly what was asked, minimally.
- KEY FINDING: Strongest real result. Output -8% (tight), turns 8 vs 9, cost generally lower. Fewer turns is the main saver (more direct, less over-engineering). +input from per-turn ruleset is outweighed by fewer turns.
- Disadvantage axes:
  - Setup friction: SIGNIFICANT via documented plugin install (needs git); MINOR via hooks (git-free, worked)
  - Broke/hid a needed tool: No
  - Answer-quality degradation: No (task correct, all tests pass, minimal correct code). Theoretical risk: aggressive YAGNI pushback in other contexts.
  - Added latency: Negligible (SessionStart hook <0.2s; runs faster overall due to fewer turns)

## 5. SQUEEZ — v1.32.1 (binary), pinned commit 752179e (2026-06-22), repo npm 0.4.0
- Type: hook-based compressor (PreToolUse/PostToolUse). Compresses bash/command output before context. Bucket B. Rust binary (prebuilt win exe - downloads fine, unlike headroom).
- WINDOWS OUT-OF-BOX: BLOCKED.
  - `squeez setup` FAILS: shells to python3 to patch settings -> exit 9009 (python3 = broken Win Store stub).
  - Runtime hooks (pretooluse.sh/posttooluse.sh) parse JSON via python3 -> no-op without real python3.
  - Got it working only via MANUAL fixes: created python3.exe shim from installed Python312, wired hooks with full bash.exe path. A normal user would not do this.
- Binary compression on THIS task's output: 0% (node --test: 298->298 tokens). squeez targets large/noisy logs (builds, npm install), not small clean test output.
- E2E (3 runs, hooks confirmed firing - created sessions/ tracking):
  - turns: 8/9/9
  - output: 1498/1525/1737 -> avg 1,587 = -3% (noise)
  - total input: 184439/185261/186669 -> avg 185,456 = +2.5%
  - cost: $0.178/$0.180/$0.186 -> avg $0.181 = +12% vs baseline (slightly WORSE)
  - time: 39/39/55s -> per-tool-call hook (bash+python+squeez spawn) adds latency
- Task correctness: PASS (4/4).
- Disadvantage axes:
  - Setup friction: BLOCKING on Windows out-of-box (python3 dependency; setup exit 9009; needs manual shim+bash wiring)
  - Broke/hid a needed tool: No
  - Answer-quality degradation: No
  - Added latency: YES, noticeable (each Bash/Read spawns bash+python3+squeez; up to +15s/run)
- VERDICT: Doesn't work on Windows out of the box. Even when forced to run, ~neutral-to-slightly-negative on focused coding tasks (small outputs). Would help on log-heavy workflows only.

## 6. CONTEXT-MODE — v1.0.169, commit 4dedadc (2026-06-29)
- Type: MCP server + plugin + hooks. Sandboxed code execution; agent meant to run code & return only stdout instead of reading files. Bucket B. Node-bundled (no native build).
- WINDOWS: WORKS out of the box. `context-mode doctor` PASS (storage ok, detects node/python-via-py/bash). `context-mode upgrade` wired PreToolUse+SessionStart hooks via node. git absence = non-blocking warning only.
- Setup: registered MCP (node cli.bundle.mjs) + upgrade hooks + user CLAUDE.md routing.
- E2E (3 clean runs; run2 discarded - reset lock):
  - turns: 8/9/9
  - output: 1535/1734/1862 -> avg 1,710 = +5%
  - total input: 205395/175492/204901 -> avg 195,263 = +8% (MCP tools + CLAUDE.md overhead; high variance)
  - cost: $0.158/$0.157/$0.167 -> avg $0.161 = ~0% vs baseline (NEUTRAL, cache pricing absorbs the extra input)
- KEY FINDING: Agent did NOT engage the sandbox (context-mode sessions storage stayed EMPTY) on a small-file task. So no compression benefit; just added ~+8% input overhead that washed out on cost. Its 98% claim needs LARGE tool outputs + the agent actually routing through ctx_execute - neither applies to focused small tasks.
- Task correctness: PASS (4/4).
- Disadvantage axes:
  - Setup friction: MINOR (runs on Windows; self-wires via `upgrade`; node-based). git needed only for marketplace/auto-update (non-blocking).
  - Broke/hid a needed tool: No (PreToolUse hook intercepts Read/Bash/Grep but task completed; one "no stdin in 3s" hook warning)
  - Answer-quality degradation: No
  - Added latency: Minor (SessionStart + PreToolUse node hooks per tool call)
- VERDICT: Harmless but useless on small coding tasks (agent ignores sandbox). Could help on huge-output workflows (log/data analysis) if the agent routes through it.

## 7. PARE-CLAUDE-MD — commit 48d36ad (2026-03-02)
- Type: one-shot skill. Rewrites CLAUDE.md/AGENTS.md removing "obvious" content (obviousness principle), prepends core principles. Bucket A (shrinks per-session memory). NO runtime overhead.
- Install: skill. Plugin/marketplace install needs git (would fail); manual copy to skills/ works (git-free, what I did). MINOR friction, one-shot.
- Test input: representative bloated CLAUDE.md (4,212 bytes / 104 lines; mostly obvious boilerplate + 5 genuine gotchas).
- MEASURED (startup probe, real session tokens):
  - no CLAUDE.md baseline: 20,657
  - bloated CLAUDE.md present: 21,945  (+1,288/session)
  - pared CLAUDE.md present:   21,365  (+708/session)
  - SAVING: 580 tokens/session (~45% of this file's overhead), PERMANENT/recurring every session.
- pare output: 4,212 -> 1,160 bytes (-72%). Kept ALL 5 non-obvious gotchas + principles; removed obvious structure/npm/coding-standards/generic-advice. High quality.
- One-time cost to run: ~$0.20, 6 turns. Saving accrues every session forever. Scales with how bloated your CLAUDE.md is (teams with 15-30KB files save thousands of tokens/session).
- Task correctness: N/A (transformation). Pared file is accurate + higher signal.
- Disadvantage axes:
  - Setup friction: MINOR (skill; manual copy git-free; one-shot)
  - Broke/hid a needed tool: No
  - Answer-quality degradation: No — IMPROVES signal (ETH Zurich: bloated context hurts perf). Minor risk: over-aggressive pare could drop something; skill asks for confirmation by default (review the diff).
  - Added latency: NONE at runtime (one-time transform; not active in normal sessions)
- VERDICT: Cleanest pure-win. Zero ongoing cost, permanent per-session saving, no downside. Only caveat: helps only if your CLAUDE.md is actually bloated.

## CROSS-CUTTING FINDINGS
- Claude Code 2.1 ALREADY defers MCP tool schemas natively (names only until used). This neutralizes most of mcp-compressor's premise and reduces MCP "bloat" generally.
- On agentic CODING tasks, output is dominated by code + tool-calls. Tools that compress PROSE (caveman) or tool-OUTPUT (squeez/context-mode/headroom) save little because the task's outputs are small/code-heavy. They shine on chat (caveman) or huge-log workflows (squeez/headroom/context-mode).
- Windows is a real filter: headroom BLOCKED (native build), squeez BLOCKED out-of-box (python3), caveman/ponytail plugin-install needs git (hooks fallback works), mcp-compressor/context-mode/pare fine.
- SIDE EFFECT: squeez wrote an always-on persona into REAL ~/.claude/CLAUDE.md (hardcodes $HOME/.claude, ignores config isolation) — invasive global change. Cleaned up.
- COST NOISE: cache-read vs cache-write pricing makes per-run cost vary ~20%; output tokens + turns are the stabler signals.
