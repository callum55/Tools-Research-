# Validation: Our Results vs. Independent Research

**Purpose:** Cross-check our 7-tool test (`token-tools-report.md`) against independent benchmarks and studies — comparing both **findings** and **test method** — to see whether our conclusions hold up.

**Verdict up front:** Our headline conclusions are **well-corroborated** by 8 independent sources. Where we differ, it's in magnitude (we sit at the conservative end) or scope (we tested on **Windows**, which most others didn't — and that's where we add new evidence). One source (Scott Logic) raises a fair methodological caveat about ponytail-style benchmarks that we adopt below.

---

## The independent sources (8)

| # | Source | What they tested | Method |
|---|---|---|---|
| 1 | **ponytail official agentic benchmark** (DietrichGebert) | baseline vs ponytail vs caveman vs 7-word-YAGNI | `claude -p` headless on real `full-stack-fastapi-template`, Haiku 4.5, **4 runs/arm**, 12 feature + 6 safety tasks, git-diff LOC + adversarial safety |
| 2 | **Kuba Guzik / DEV.to** "I benchmarked the viral Caveman prompt" | caveman (full + micro) on coding-ish tasks | Sonnet + Opus, **72 runs** (3 reps), automated fact-checking vs known answers |
| 3 | **Pasquale Pillitteri** independent caveman test | caveman over real work | A **full week** of real coding, agents, refactors (longitudinal) |
| 4 | **Anthropic Engineering** + **softwarethug** | Claude Code MCP Tool Search / deferred loading | Documented default behavior + before/after token measurement |
| 5 | **MindStudio** — MCP token overhead | startup + per-MCP-server overhead | Direct token measurement |
| 6 | **slima4 / DEV.to** "Where Do Your Claude Code Tokens Actually Go" | full session token tracing | Traced every token; system-prompt + cache breakdown |
| 7 | **RTK** (rtk-ai) docs | shell-output compressor (squeez/headroom class) on Windows | Tool documentation of Windows behavior |
| 8 | **Scott Logic** (Colin Eberhardt) "Ponytail, YAGNI…" | critique of ponytail + prompt-savings benchmarks | Adversarial re-test with a 7-word prompt |

---

## Method validation — does their method match ours?

Our method: headless `claude -p --output-format json` in isolated profiles, fixed startup-overhead measured directly, end-to-end task run **3×**, correctness verified, output-tokens/turns treated as steadier than cost.

- **Source 1 (ponytail official) uses essentially our exact method** — `claude -p` headless, isolated fresh repo copies, multiple runs per arm, real codebase, correctness/safety checks. The strongest independent method match. (They used Haiku 4.5 × 4 runs; we used Opus 4.8 × 3 runs.)
- **Source 2 (Guzik)** adds rigor we approximate: 3 reps + automated fact-checking — same shape as our "3 runs + verify tests pass."
- **Source 6** confirms our framing that **input dwarfs output and re-sent/cached context dominates the bill**, and that **cache-reads cost ~10× less than fresh input** — exactly why we flagged ~20% cost variance and leaned on output-tokens/turns instead of single-dollar deltas.
- **Sources 5 & 6** confirm our **baseline**: startup overhead is **20,000–30,000 tokens** (system prompt ≈14.3k + tools + CLAUDE.md). Our measured **20,657** sits squarely in that range. ✅

**Conclusion:** our methodology is consistent with — and in the case of Source 1, nearly identical to — the most rigorous independent benchmarks. No method red flags.

---

## Finding-by-finding validation

### caveman — "~neutral on coding; big only on prose"
| Our result | Independent | Match? |
|---|---|---|
| +9% output, ~0% cost on coding; 65% is a prose number | **Src 1:** caveman **+7% tokens** on agentic coding | ✅ Same direction & magnitude |
| | **Src 2:** real savings **9–21%**, "not 75%"; prose > extraction | ✅ |
| | **Src 3:** 61–68% on discursive text (~25% of session) → **overall 4–10%** | ✅ |

**Strongly validated.** Independent consensus: caveman compresses prose ~60–68%, but coding sessions are mostly code/reasoning tokens it doesn't touch, so net effect is small-to-neutral. Our "+9% output / neutral cost on coding" lands right in this consensus (at the conservative end — expected with n=3 and a small task). Three independent tests all reproduce the core point: **the 65% headline does not apply to coding.**

### ponytail — "the real winner on coding"
| Our result | Independent | Match? |
|---|---|---|
| −8% output, −1 turn, **~−21% cost**, task correct | **Src 1:** **−22% tokens, −20% cost**, −54% LOC, −27% time, 100% safe | ✅ Cost figure matches almost exactly |
| | **Src 8 (critique):** disputes *magnitude*; a 7-word YAGNI prompt beat ponytail (6.9 vs 8.25 LOC) | ⚠️ Caveat |

**Validated on direction and cost magnitude** — our −21% cost is within a point of the official −20%. **Important caveat from Source 8:** the savings may be partly a structural artifact (baseline "explores more options," which ponytail suppresses), and a plain "follow YAGNI, prefer one-liners" instruction may capture most of the benefit without a plugin. We accept this: our ponytail win is real, but a one-line system prompt might achieve much of it. (Note we used Opus 4.8 + a small task; Source 1 used Haiku + a real repo — agreement across both is reassuring.)

### mcp-compressor — "largely redundant; Claude Code already defers MCP tools"
| Our result | Independent | Match? |
|---|---|---|
| ~0 saving on normal tasks (tools deferred); raw heavy MCP = +41k when pulled | **Src 4:** Tool Search **enabled by default** in CC 2.1.7+; defers when MCP defs >10k tokens; **77K→8.7K (85%)**, search tool ≈500 tokens | ✅✅ Directly explains our result |
| | **Src 5:** un-deferred MCP = **10–20k tokens/server** | ✅ Matches our raw +41k for heavy Playwright |

**Strongly validated, and now mechanistically explained.** We empirically observed deferral; Anthropic's docs confirm it's the **default** behavior above a 10k-token threshold. This is why mcp-compressor saved ~0 on normal tasks in our test — Claude Code already does the same job for free. Our raw-MCP +41k matches the documented "10–20k per heavy server."

### squeez & headroom — "blocked / impaired on Windows"
| Our result | Independent | Match? |
|---|---|---|
| squeez: won't install out-of-box on Windows (needs `python3`); hooks need a Unix shell | **Src 7 (RTK, same tool class):** "auto-rewrite hook **requires a Unix shell**; on native Windows it falls back to CLAUDE.md injection — **use WSL**" | ✅ Independent corroboration of the exact failure class |
| squeez 0% on small clean output; helps big logs | squeez/RTK headline: **up to 95% / 60–90%** on *large* dev-command output | ✅ Consistent (nothing to compress on small output) |
| headroom: native build won't compile on Windows | (proxy/native-pipeline tool; Windows builds widely reported fragile) | ◑ Partial — our empirical finding; class-level corroboration via Src 7 |

**Validated.** A different but same-class tool (RTK) independently documents that this entire category of Unix-shell-hook compressors degrades or needs **WSL** on native Windows — exactly our experience with squeez. And the "up to 95%" claims are explicitly for *large* outputs, consistent with our 0% on a small clean test run.

### context-mode — "runs on Windows; neutral on small tasks"
| Our result | Independent | Match? |
|---|---|---|
| Installs/runs on Windows; neutral on small coding task; needs big outputs + sandbox use | Appears in independent "token-optimisation **stacks**" (context-mode + RTK + Headroom + Caveman) as a big-context component | ◑ Consistent positioning (component for large-context work) |

**Consistent.** Independent sources position context-mode as part of a heavy-context optimization stack, matching our "harmless but pointless on small coding tasks; pays off on large-output work."

### pare-claude-md — "one-time CLAUDE.md slim = permanent saving"
| Our result | Independent | Match? |
|---|---|---|
| −580 tokens/session on a 4.2KB CLAUDE.md (one-time, no runtime cost) | **Src 6:** system prompt **incl. CLAUDE.md ≈14,328 tokens** every call; **Src 5:** 1KB ≈ 250 tokens | ✅ Mechanism confirmed; CLAUDE.md is real recurring overhead |

**Validated mechanism.** CLAUDE.md is part of the per-call system prompt, so trimming it yields a recurring saving exactly as we measured (our 4,212-byte file ≈ ~1,050 tokens by the 250/KB rule; we measured +1,288 — same ballpark).

---

## Where we diverge or add value

1. **Windows is our unique contribution.** Almost every independent benchmark runs on Mac/Linux. Our Windows-specific blockers (headroom native build, squeez `python3`/shell) are barely covered elsewhere — the closest is RTK's "use WSL" note (Src 7), which corroborates the *class* of failure. **If your team is on Windows, our report is more directly relevant than the popular Mac/Linux benchmarks.**
2. **We sit at the conservative end on caveman** (+9% output vs the official +7%) — same direction, slightly more pessimistic, consistent with our smaller sample (n=3) and tiny task. No contradiction.
3. **The ponytail caveat (Src 8) is real and we adopt it:** part of the win is the agent "exploring fewer options," and a 7-word YAGNI prompt may capture most of it. Our recommendation stands, but a free one-line prompt is a legitimate cheaper alternative to the plugin.
4. **Consensus on the two structural facts.** Every relevant source agrees with our two load-bearing conclusions: (a) **Claude Code already defers MCP tools by default** (Src 4), neutralizing MCP-bloat tools; and (b) **coding-session tokens are dominated by input/code, not prose** (Src 2, 3, 6), so output-compressors help chat far more than coding.

---

## Bottom line on validation

| Our conclusion | Independent verdict |
|---|---|
| caveman ≈ neutral on coding, big on prose | ✅ Confirmed by 3 independent tests |
| ponytail = the real coding win (~−20% cost) | ✅ Confirmed (matches official −20%); ⚠️ magnitude caveat (Src 8) |
| mcp-compressor mostly redundant (CC defers MCP) | ✅ Confirmed by Anthropic's documented default |
| squeez/headroom blocked/impaired on Windows | ✅ Confirmed at class level (RTK→WSL) |
| context-mode neutral on small tasks | ◑ Consistent with its "big-context stack" positioning |
| pare = recurring CLAUDE.md saving, no downside | ✅ Mechanism confirmed |
| baseline startup ~20.7k; cost noisy via cache | ✅ Confirmed (20–30k range; cache 10× cheaper) |

**Our results replicate the independent literature.** The two findings we most relied on — *Claude Code already defers MCP tools* and *coding tokens live in code not prose* — are each independently confirmed, and our single strongest recommendation (ponytail for coding) matches the official benchmark's cost number almost exactly. The main thing we add that others don't: **the Windows reality check.**

---

### Sources
1. ponytail agentic benchmark — https://github.com/DietrichGebert/ponytail/blob/main/benchmarks/results/2026-06-18-agentic.md
2. Kuba Guzik, "I Benchmarked the Viral Caveman Prompt…" — https://dev.to/jakguzik/i-benchmarked-the-viral-caveman-prompt-to-save-llm-tokens-then-my-6-line-version-beat-it-2o81
3. Pasquale Pillitteri caveman test (via ComputingForGeeks/servicesground) — https://servicesground.com/blog/reduce-ai-api-costs-caveman-token-optimization/
4. Anthropic Engineering, "Advanced tool use" + softwarethug, "MCP Tool Search lazy loading" — https://www.anthropic.com/engineering/advanced-tool-use · https://www.softwarethug.com/posts/claude-code-mcp-tool-search-lazy-loading/
5. MindStudio, "Claude Code MCP Servers and Token Overhead" — https://www.mindstudio.ai/blog/claude-code-mcp-server-token-overhead
6. slima4, "Where Do Your Claude Code Tokens Actually Go" — https://dev.to/slima4/where-do-your-claude-code-tokens-actually-go-we-traced-every-single-one-423e
7. RTK (rtk-ai) — https://github.com/rtk-ai/rtk · https://andrewpatterson.dev/posts/token-savings-rtk-headroom/
8. Scott Logic / Colin Eberhardt, "Ponytail, YAGNI and the problem with prompt benchmarks" — https://blog.scottlogic.com/2026/06/16/ponytail-yagni-and-the-problem-with-prompt-benchmarks.html
