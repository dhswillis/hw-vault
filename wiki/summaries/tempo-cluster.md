---
created: 2026-04-10
updated: 2026-04-10
type: summary
sources:
  - raw-sources/trading/tempo/tempo_rules_v3_complete.md
  - raw-sources/trading/tempo/tempo_rules_v2_complete.md
  - raw-sources/trading/tempo/TEMPO_V14_CORRECTIONS.md
  - raw-sources/trading/tempo/TEMPO_IFVG_BUILD_SPEC.md
  - raw-sources/trading/tempo/TEMPO_IFVG_RESEARCH.md
  - raw-sources/trading/tempo/TEMPO_IFVG_AUDIT_REPORT.docx
  - raw-sources/trading/tempo/IFVG_TP_Optimization_Report.docx
  - raw-sources/trading/tempo/LUMI_STRATEGY_SPEC.md
  - raw-sources/trading/tempo/The_Unified_Brain_Architecture.docx
  - raw-sources/trading/tempo/Unified_Brain_Options_and_Growth.docx
  - raw-sources/trading/tempo/AIO_System_Blueprint.docx
  - raw-sources/trading/tempo/AI_Trading_Show_Blueprint.docx
  - raw-sources/trading/tempo/Tempo_IBKR_Migration_Plan.docx
  - raw-sources/trading/tempo/TEMPO_PROJECT_STATE.md
related:
  - wiki/concepts/ifvg.md
  - wiki/concepts/smt.md
  - wiki/concepts/bos-fvg.md
  - wiki/entities/tempo-methodology.md
  - wiki/entities/nova-sable-brains.md
  - wiki/entities/tempo-trading-system.md
  - wiki/syntheses/bos-fvg-claim-vs-reality.md
  - wiki/syntheses/tempo-three-layers.md
tags: [trading, tempo, cluster-summary, ifvg, unified-brain]
---

# Tempo Cluster — 14 Sources

Everything in `raw-sources/trading/tempo/`. The cluster spans three distinct layers that **must not be conflated**:

1. **Layer 1 — Tempo the educator's methodology** (canonical — 88% WR on 400+ real trades)
2. **Layer 2 — Harrison's mining/backtest interpretations** (repeatedly contaminated — mostly suspect)
3. **Layer 3 — Current implementation** (v14/v24 on NinjaTrader sim — unproven)

See [[tempo-three-layers]] for how they relate.

## Layer 1 — Tempo methodology (canonical)

### `tempo_rules_v3_complete.md` — 281 trade recaps (Oct 2024 → Jan 2026)
The most authoritative single source for what Tempo actually teaches. 257k words analyzed across 281 transcriptions. Captures the full methodology evolution across 5 phases (Foundation → Integration → Adaptation → Refinement → Mastery).

**Key rules:**
- **Entry:** [[ifvg|IFVG]] (Inverted Fair Value Gap) + [[dol-framework|DOL sweep]] + [[smt|SMT divergence]] → inverse entry on the reversal
- **Optimal gap size:** 8–15 pts NQ (5–6 risky, 20–30 strong, skip >30)
- **Entry trigger:** candle CLOSE through the gap (not wick)
- **Setup quality:** A+ needs 7+ confluences; minimum for funded accounts = 3+
- **SMT is PRIMARY confluence** since Nov 2025 — without SMT, WR drops from ~80% to ~60-65%
- **Trim philosophy V3:** 55% at TP1 (+17pts), 30% at TP2 (+35pts), 15% runner — "base hit mentality, $200-300/account"
- **Stops:** Soft stops 90% of the time (watch for candle CLOSE past stop, not wick). Hard stops only for funded account drawdown rules.
- **Break-even:** at +17pts profit, non-negotiable. Never BE below 10pts.
- **Risk:** 0.4–0.6% per trade ($200-300 on $50k), NOT 1%
- **Session:** primary window 9:30–11:00 AM ET, exit at 11. Wait 2–5 min after open.
- **Max 2 losses/day** then done. After daily target hit, lock all funded accounts.
- **Copy trading:** 30+ prop firm accounts by Jan 2026. $42k single day achieved 2026-01-29.

**Reported performance (Tempo's own, late 2025–Jan 2026):** 88% WR across 400+ trades, avg win 30-50 pts, avg loss 15-20 pts.

See [[ifvg]], [[smt]], [[dol-framework]], [[bpr]], [[setup-quality-grading]] for concept extractions.

### `tempo_rules_v2_complete.md`
Earlier extraction from 90 transcriptions. Superseded by v3 but useful for tracking how Tempo's methodology evolved. Key change from v2 → v3: SMT elevated to primary, trimming became more conservative, soft stops became default.

### `TEMPO_PROJECT_STATE.md` — the master "read this first" doc
Project state snapshot (2026-02-09) by Harrison. Critical warnings to Claude:
1. **Do NOT invent rules.** Only use what Tempo actually teaches.
2. **Follow Option 4 architecture** — NOT a monolithic QC algo.
3. **Real rules live in 324 trade recap videos** — 281 transcribed, 43 remaining.
4. **SMT = Smart Money Technique** (NQ vs ES), NOT "Supply Manipulation Test."

Contains: 324 video pipeline (Twelve Labs + Whisper), Discord education channel inventory (42 premium videos, 18 YouTube classes), V1/V2 QC backtest history (V1 blew up -92% in 2022 bear market, V2 circuit breaker fired permanently), 12 "lessons learned" including the circuit-breaker-must-have-recovery rule.

## Layer 2 — Mining / backtests (suspect)

### `TEMPO_IFVG_RESEARCH.md`, `TEMPO_IFVG_BUILD_SPEC.md`
The IFVG build spec and supporting research. **Critical:** per `TEMPO_V14_CORRECTIONS.md`, the build spec's signal detection sequence is WRONG — the actual Tempo pattern has the FVG forming BEFORE the sweep (momentum gap), not after. See correction detail below.

### `TEMPO_IFVG_AUDIT_REPORT.docx`, `IFVG_TP_Optimization_Report.docx`
IFVG-family mining and TP optimization results. Must be read through the lens of the [[v10i-look-ahead-bug]] and [[be-trail-mechanism]] caveats.

### `LUMI_STRATEGY_SPEC.md`
Small spec (2KB) for the Lumi engine. Per v14 corrections audit: 35 trades / 29 days, 20% WR, -0.228 avg R, net NEGATIVE on the audit period. Adding Lumi to the portfolio HURTS it (pts/day drops from +26.4 to +23.0, max DD worsens).

## Layer 3 — Implementation & current state

### `TEMPO_V14_CORRECTIONS.md` — critical implementation corrections (2026-03-20)
Four critical corrections to the v14 C# code:

1. **Signal detection sequence (WRONG in build spec):**
   - WRONG: sweep → new FVG forms → candle closes through it
   - CORRECT: momentum gap FVG forms → price sweeps key level → price reverses back through the **pre-existing** FVG → entry at **FVG edge** (not bar close)
2. **Stop placement:** use opposite IFVG edge (soft stop), NOT fixed 20-30pt stop. Emergency hard stop = sweep wick ± 1pt as safety net only.
3. **Entry price:** FVG edge exactly — use 30s bars for precise detection. Previous code entered at bar close (often 13+ pts past the FVG edge).
4. **Emergency stop wrong-side bug:** for 30 fade trades, emergency stop was on the PROFIT side of entry — effectively disabled. Fixed with `max(wick+1, entry+3)` for shorts / `min(wick-1, entry-3)` for longs.

**London session diagnosis:** 0% WR consistently. Root cause: 100% runner with BE at +17pts + soft stop at IFVG edge (~5-7pt gaps) = soft stop catches every trade before it can run. London breakout entirely negative; London fade "works" only via EOD runner dependence (3 EOD trades = 247% of total PnL). **Recommendation: skip London in production** unless structurally rebuilt.

**Combined portfolio (2026-03-20, 29 days):** US M60 + London fade = 163 trades, 62% WR, +31.9 pts/day, Calmar 4.8. **BUT:** 4 EOD trades = 139% of total PnL. Non-EOD = -10.3 pts/day. Both strategies net negative without runners at flatten time. Fragile.

### `The_Unified_Brain_Architecture.docx` — THE CURRENT PROJECT STATE
This is the most recent authoritative doc (2026-04). It replaces three earlier planning docs (The Partnership, AI Trading Show Blueprint, AIO System Blueprint). **The project has pivoted dramatically.**

**Critical honesty disclosure at the top of the doc:**
> "NOTHING from the backtesting research (V7-V11) is validated for live trading. BOS_FVG does not work. V7/V8 was killed by look-ahead bias. The V10t clean results are approximately breakeven after commissions. The research journal documents what was TESTED, not what WORKS. v24 (IFVG MTF cascade) is the ONLY strategy currently on NinjaTrader sim. It has not yet proven itself."

This directly contradicts the V2 mining report's claim of BOS_FVG at 63.5% WR as "the best signal in the dataset." See [[bos-fvg-claim-vs-reality]] for the reconciliation.

**The new architecture — Unified Brain:**
- Two AI researchers: **[[nova-sable-brains|Nova and Sable]]** (DeepSeek R1-Distill-Qwen-14B via Ollama)
- Persistent memory: **Mem0** (semantic) + SQLite (structured) + weekly LoRA fine-tuning
- Risk Officer: hard-coded Python, no AI, no flexibility, non-negotiable limits
- Weekly tournament with external AI advisor judging (Claude / Grok / ChatGPT APIs)
- Phase progression: research-only → paper → micro live (1 MNQ) → scale
- Live YouTube show pipeline (ElevenLabs + BocaLive avatars + FFmpeg compositor)
- Monthly cost: $220–650

**Anti-stupidity rules (hard-coded):** NEVER test multi-TF candle alignment (look-ahead trap). NEVER test break-even stops (dead across 11 configs). ALWAYS walk-forward validate (4 quarters positive OOS). ALWAYS apply realistic costs (0.5pt slippage + 0.5pt commission per side).

**Dead strategies (baked into Mem0 seed):** london_breakout, VP_POC_retest, fib_retracement, failed_breakout, sweep_reversal, double_BOS_momentum, VWAP_mean_reversion.

### `Unified_Brain_Options_and_Growth.docx`, `AIO_System_Blueprint.docx`, `AI_Trading_Show_Blueprint.docx`
Earlier versions of the Unified Brain thinking. All superseded by `The_Unified_Brain_Architecture.docx`. Cross-link only.

### `Tempo_IBKR_Migration_Plan.docx`
Migration plan from current NinjaTrader execution to IBKR via `ib_insync`. Part of the Unified Brain Phase 2 buildout (weeks 4-5, paper trading).

## Cross-cluster connections

- **[[v5-strategy-bug-audit]]** (earlier ingest) documented the NinjaTrader V5 bugs that explained live/backtest divergence in the SF portfolio. The V14 corrections doc is the same class of problem a generation later — the C# code implementing the wrong entry logic.
- **[[comprehensive-mining-report-v2]]** (earlier ingest) claimed BOS_FVG as "the core signal" at 63.5% WR. The Unified Brain doc now says "BOS_FVG does not work" as the definitive current position. See [[bos-fvg-claim-vs-reality]].
- **[[tempo-context-engine-spec]]** (earlier ingest) described the Bayesian belief engine planning. That work exists and is "already built and working" per the Unified Brain doc — 46-feature market state vector, Bayesian session classifier at 68.6% accuracy.
