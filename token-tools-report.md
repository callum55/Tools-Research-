# Claude Code Token-Saving Tools — Test Report

**Goal:** Find out which of seven "token-saving" tools actually reduce day-to-day Claude Code cost on Windows, by how much, and what they cost you in hassle or quality.

**Date:** 29 June 2026 · **Platform:** Windows 11, Claude Code 2.1.195, model Opus 4.8 (claude-opus-4-8)

---

## How to read this (plain English)

Every Claude Code session pays for tokens in three ways:

- **A — Startup overhead:** a fixed block of text (system prompt + every tool definition + your CLAUDE.md) sent at the start. Paid every session.
- **B — Tool-output tokens:** the text your commands and file-reads dump into the conversation.
- **C — Response tokens:** how wordy Claude's own replies and generated code are.

Each tool attacks a different one of these. **That's the whole story of why some helped and some didn't:** our real-world test was a *coding* task, and coding output is mostly code + tool calls — which most of these tools deliberately don't touch.

### The test
- **Baseline:** a clean, isolated Claude Code profile with **no extra tools**.
- **Startup overhead (A):** measured directly — the exact token count of a fresh session. Deterministic.
- **End-to-end (B+C):** one identical real task (fix a bug + add a function + run tests) run **3× per tool**; we report the average and range of output tokens and cost. Kept separate from the startup number, never blended.

### The labelled baseline

| Baseline (no tools) | Value |
|---|---|
| **Startup overhead (A)** | **20,657 tokens** |
| End-to-end output (avg, 3 runs) | 1,630 tokens (range 1,564–1,755) |
| End-to-end cost (avg) | $0.161 (range $0.139–$0.173) |
| Task result | ✅ correct (4/4 tests) |

> **Cost figures wobble ~20% run-to-run** because Claude prices "cached" vs "fresh" context differently. Output-tokens and turn-count are the steadier signals; treat single-dollar deltas as rough.

---

## Results table

| Tool (version) | Startup Δ vs baseline | End-to-end cost (avg + range) | Output Δ | Setup friction | Broke a tool? | Quality loss? | Added latency |
|---|---|---|---|---|---|---|---|
| **Baseline** | — (20,657) | $0.161 ($0.139–0.173) | — (1,630) | — | — | — | — |
| **caveman** v1.9.0 (`25d22f8`) | **+1,140 (+5.5%)** | $0.161 (~0%) | +9% | Significant (plugin needs git; hooks work) | No | No | Negligible |
| **headroom** v0.28 (`aea3c35`) | **Not measurable** | **Could not run** | — | **Blocking (Windows)** | N/A | N/A | N/A |
| **mcp-compressor** (`d664676`) | **~0 normal / −28% schema-heavy** | (not an e2e tool) | — | Minor | Risk (indirection) | No | Modest |
| **ponytail** v4.8.4 (`bc9ee94`) | +1,712 (+8.3%) | **$0.127 (−21%; $0.094–0.145)** | **−8%** | Significant (plugin git) / Minor (hooks) | No | No | None (faster) |
| **squeez** v1.32.1 (`752179e`) | ~0 | $0.181 (**+12%**) | −3% | **Blocking out-of-box (Windows)** | No | No | Yes (+~15s) |
| **context-mode** v1.0.169 (`4dedadc`) | — | $0.161 (~0%) | +5% | Minor (runs on Windows) | No | No | Minor |
| **pare-claude-md** (`48d36ad`) | **−580/session** (one-time) | N/A (one-shot) | — | Minor | No | No (improves) | None |

---

## One sentence per tool

- **caveman** — Makes Claude "talk like a caveman" to cut wordiness; real for chat, but on coding it saved nothing here (code/tool-output dominate and it leaves those alone) while adding ~800 tokens of rules every turn — **a wash for a coding team.**
- **headroom** — The most-hyped tool (54k stars), but its compression engine is a native module with **no Windows build**; it would not install or run, so it fails your "must work on Windows" bar outright.
- **mcp-compressor** — Installs cleanly and shrinks MCP tool definitions, but **Claude Code 2.1 already hides MCP tools until they're needed**, so on normal work it saved ~0; only helps if you run many always-on MCP tools.
- **ponytail** — Tells Claude to "do the simplest thing that works," which made it explore less and finish in fewer steps (8 vs 9 turns) with ~8% less output at lower cost — **the only tool that genuinely saved money on real coding, with the task still correct.**
- **squeez** — Compresses big noisy command logs, but **won't install on Windows out of the box** (needs a working `python3` you don't have), had nothing to compress on a small task, added per-command latency, and silently edited your global CLAUDE.md.
- **context-mode** — The best-behaved "compressor" on Windows (it installs and runs), but on a small coding task the agent never used its sandbox, so it just added overhead that washed out to **neutral** — it only pays off on huge-output workflows.
- **pare-claude-md** — A one-time cleanup that rewrote a bloated CLAUDE.md down 72% and saved ~580 tokens **every future session** with no ongoing cost and better signal — **pure upside if your CLAUDE.md is bloated.**

---

## Bottom line — what to keep (ranked, plain English)

**1. ✅ pare-claude-md — keep and use now.**
The only no-downside win. Run it once on each bloated CLAUDE.md/AGENTS.md your team has; every session afterward is cheaper and cleaner, forever, at zero ongoing cost. The bigger your CLAUDE.md, the more you save (our modest 4 KB file saved ~580 tokens/session; a typical 20 KB file saves several thousand). Review the diff before saving — that's the only care needed.

**2. ✅ ponytail — keep if your team mostly writes code.**
The only tool that actually lowered the cost of a real coding task (~8% less output, fewer back-and-forth turns, ~20% cheaper on average) while still completing the work correctly. Install via its hook files (the one-click plugin install needs `git` on PATH, which this machine lacks). Low risk, real upside for coding-heavy use.

**3. 🟡 caveman — only if you use Claude mostly for chat/Q&A, not coding.**
Its 65% savings are real for prose, but invisible on coding work, where it was a slight net cost here. Skip it for a coding team.

**4. 🟡 context-mode — keep only if you do huge-output work** (analyzing massive logs/data dumps). It's harmless on normal coding (neutral cost) but only delivers its headline savings on giant tool outputs. Not worth the setup for everyday coding.

**5. 🟡 mcp-compressor — only if you run many always-on MCP servers.** Claude Code already does most of what it offers. If your team loads lots of MCP tools and feels the weight, it can help; otherwise it's redundant.

**6. ❌ squeez — skip on Windows.** Doesn't install out of the box (needs `python3`), added latency, and edited your global config behind your back. Reconsider only on Mac/Linux with log-heavy workflows.

**7. ❌ headroom — skip on Windows.** Can't run at all here. Revisit only if your team moves to Mac/Linux.

### The two facts that explain everything
- **Claude Code 2.1 already defers MCP tool definitions** (loads names, fetches details on demand). That alone neutralizes most "MCP bloat" tools.
- **On coding work, the tokens live in code and tool output**, which the prose/output compressors deliberately don't touch — so they help chat and log-analysis far more than coding.

**If you do one thing:** run **pare-claude-md** on your CLAUDE.md files, and add **ponytail** if you're a coding team. Those two are the real wins; the rest are situational or don't work on Windows.

---

## Appendix — method notes & caveats
- All tools pinned to exact commits (above). Results will drift as they update.
- Tested headless via `claude -p --output-format json` in isolated `CLAUDE_CONFIG_DIR` profiles; tokens read from the API usage block (real billed tokens).
- **headroom** could not run, so its figures are vendor-reported estimates only, not measured.
- **mcp-compressor** is a startup-overhead tool, so it was measured directly (raw vs wrapped MCP), not via the end-to-end task.
- **squeez** required manual Windows fixes (a `python3` shim + hand-wired bash paths) just to run at all — a normal user would not do this; "blocking" reflects the out-of-box experience.
- Cost variance (~20%) comes from cache pricing; where a tool's cost delta is within that band, treat it as "neutral."
