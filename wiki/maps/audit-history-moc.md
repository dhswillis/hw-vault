---
created: 2026-04-11
updated: 2026-04-11
type: moc
topic: Audit History
tags: [moc, audits, methodology, canonical]
related:
  - wiki/concepts/invalidation-rules.md
  - wiki/concepts/bar-sim-trailing-bug.md
  - wiki/concepts/v10i-look-ahead-bug.md
  - wiki/summaries/audits-cluster.md
---

# Audit History — Map of Content

> The chronological catalog of every audit run on the Tempo trading research corpus. Grouped by outcome: **bugs found** vs **clean passes**. Each audit links to what it invalidated and the rule(s) it produced.

## Orientation

The Tempo research has been through **seven major audits** in 2026. Each one found a different class of contamination. The pattern that matters: **each audit catches a different bug**. An earlier clean-looking audit does NOT protect a signal against later audits that test for different problems. The V2 mining report survived the V10i look-ahead audit — and still got killed by bar-sim trailing two months later.

**The master ruleset distilled from all of these:** [[invalidation-rules]]. Read it before citing any number from pre-April 2026 research.

## ⚠️ Audits that found bugs (sorted by date)

These are the audits where the research was wrong and had to be corrected. The invalidation chain flows forward from each one.

### 2026-02-18 — Look-Ahead Audit (foundation audit)

**Source:** [[look-ahead-audit]]
**Scope:** Systematic look-ahead check on V10-era mining pipeline.
**Bug found:** [[v10i-look-ahead-bug|V10i alignment look-ahead]] — `get_alignment()` consulted unclosed higher-TF bars; `close` field was future data.
**Magnitude:** A 91.1% WR config on 2,035 trades dropped to **37.6% WR on 340 trades** after the fix. 0/13 green months after. Entire "sniper tier" edge was look-ahead.
**Rule produced:** [[invalidation-rules|Rule 2]] — do not consult unclosed HTF bars.
**Invalidated:** all V10a–V10n alignment research; V1 312-day mining report "sniper tiers" (97.9% / 98.8% / 90.6% WR).

### 2026-03-07 — Tick Data Quality Audit

**Source:** MEMORY.md "FULL AUDIT (2026-03-07)" section
**Scope:** Databento tick data sanity check.
**Bug found:** 601,050 bad ticks (0.6%) across 306/312 days — prices ~200–500 (likely MNQ bleed). 26.5% of US 1-min bars contaminated.
**Magnitude:** Context engine and `clean_backtest.py` were NOT affected (symbol filter). **Doji break scripts WERE affected** — loaded all ticks without symbol filter. The false doji break "edge" was partly this data contamination.
**Rule produced:** Safety filter `df = df[(df['price'] >= 10000) & (df['price'] <= 30000)]` in `micro_structure.py:170` and `clean_backtest.py:140`.
**Invalidated:** doji break strategy all sessions (also had phantom B/E bug — double contamination).

### 2026-03-08 — BOS_FVG 10+pt Tick-Based Audit (the careful one)

**Source:** [[bos-fvg-10pt-audit]]
**Scope:** Tick-level audit of the 10+pt static T3R BOS_FVG variant with strict statistical rigor.
**Bugs found:** Three distinct contamination sources:
1. **P/D filter was 100% look-ahead** — original filter used full-day H/L (future data) producing +0.241 avgR; rolling P/D with clean data produces +0.031 avgR — zero separation
2. **Stacked FVG filter** captures something real structurally but fails permutation p = 0.118
3. **Multiple-comparison artifacts** — 25 filter combos tested, every 95% bootstrap CI spans zero after Bonferroni
**Magnitude:** Baseline +0.015 avgR (essentially zero). Best "clean" config is ~+0.15 avgR and still not tradeable after commissions + slippage.
**Rules produced:** [[invalidation-rules|Rules 5, 6, 7, 8]] — P/D up-to-signal-time only, walk-forward ≠ edge, multiple comparison correction, outlier month check.
**Invalidated:** every BOS_FVG variant that cited P/D alignment as a filter; "best filter" claims from mining grid sweeps without Bonferroni correction.
**Status:** **Still valid** — was tick-level and statistically rigorous. This was the first real warning that BOS_FVG had no edge, and was unfortunately not propagated to the NQ Playbook / Research Journal until the April 10 audit.

### 2026-03-20 — Tempo V14 Corrections Audit

**Source:** [[tempo-v14-corrections]]
**Scope:** Audit of the v14 C# NinjaTrader implementation against the canonical Tempo methodology.
**Bugs found:** 4 implementation bugs:
1. **Signal detection sequence wrong** — code waits for FVG AFTER sweep; real Tempo pattern has FVG forming BEFORE sweep (momentum gap)
2. **Stop placement wrong** — fixed 20-30pt stop; real Tempo uses opposite IFVG edge as soft stop
3. **Entry price wrong** — bar close, often 13+ pts past FVG edge; real Tempo enters at FVG edge exactly
4. **Emergency stop on wrong side** — for 30 fade trades, emergency stop was on profit side, effectively disabled
**Magnitude:** After corrections, US sessions 70.5% WR / +24.5 pts/day. London 0% WR — **structural issue**, soft stop catches every trade before +17pt BE trigger.
**Rule produced:** Skip London for IFVG-style strategies unless soft stop is restructured.
**Invalidated:** the v14 build spec (pre-correction); London component of the Tempo portfolio.

### 2026-03-23 — V5 Strategy Bug Audit

**Source:** [[v5-strategy-bug-audit]]
**Scope:** Live vs backtest divergence investigation on V5 NinjaTrader SF portfolio strategies.
**Bugs found:** 7 bugs:
1. **VWAP double-counting** (CRITICAL) — `Calculate.OnEachTick` accumulator fires per-tick in live, per-bar in backtest → different VWAP values
2. **No protective stop orders on the exchange** (CRITICAL) — stops managed in code only, zero crash protection
3. **Chart stats tag collision** (HIGH) — both NQ strategies use `"V5Stats"`, LF1170 overwrites VWPM stats
4. Profit factor displays 0.00 on all-winners
5. Virtual P&L diverges from actual fills — no `OnExecutionUpdate` reconciliation
6. Session-flat uses wrong bar price — includes overnight gap
7. Side establishment paused during active trade (VWPM only)
**Magnitude:** ES VWPM **44.4% live WR vs 66% backtest**. Bug 1 is the primary cause.
**Rule produced:** [[invalidation-rules|Rule 9]] — NinjaTrader OnEachTick bar-dependent accumulators need bar gating.
**Invalidated:** all V5 SF portfolio live trading results until fixes are deployed.

### 2026-04-10 — BOS_FVG Failure Consolidated (the canonical audit)

**Source:** [[bos-fvg-failure-consolidated]]
**Scope:** Paired bar-vs-tick trailing replay on BOS_FVG across a 60-day Databento NQ sample.
**Bug found:** [[bar-sim-trailing-bug|Bar-sim trailing bug]] — trailing-stop simulators on OHLC bars systematically inflate results by +0.29 R/trade on BOS_FVG-class signals because the path reconstruction favors the winner.
**Magnitude:** Bar-sim +0.359 avgR → tick-replay **+0.001 avgR.** Essentially zero. **+0.29 R/trade inflation.**
**Rules produced:** [[invalidation-rules|Rules 1, 12, 14]] — do not trail on bars, tick audits supersede bar results, NautilusTrader cross-check was an early warning sign.
**Invalidated (headlines):**
- `BOS_FVG_RESEARCH.md` — +0.354 avgR / Calmar 151 / +13.37 R/d
- `clean_backtest.py` BOS_FVG — +0.895 avgR / Calmar 237
- V10z 15S trailing — +1.141 avgR / Calmar 263.6
- V10z 4-signal portfolio — +0.880 avgR / Calmar 94.4
- V2 comprehensive mining report BOS_FVG — 63.5% WR / +1.354 R/T (V2 fixed alignment look-ahead but still ran bar-sim)
- `nq-playbook` flagship numbers (all of them)
**Status:** **Authoritative current position.** Every BOS_FVG trailing-stop claim going forward must either reconcile against this audit or produce its own tick-level validation.

## Audits that didn't catch the biggest bug

These audits ran, cleaned up smaller issues, and **missed** the bar-sim trailing bug that was hiding in plain sight. The lesson is not that they were wrong — they were correct for what they tested. The lesson is that **no single audit is a clean bill of health**.

### Backtest Integrity Audit V2 (February 2026)

**Source:** `raw-sources/trading/audits/BACKTEST_INTEGRITY_AUDIT_V2.docx`
**Scope:** Broader sweep of the simulation pipeline after the V10i fix.
**What it cleaned:** Various simulator bugs, intra-bar ordering tightening (the "conservative re-check" fix that only catches 0.5% of trades).
**What it missed:** The bar-sim trailing bug requires paired bar-vs-tick replay to identify. The audit was run entirely at the bar level — it checked "is the bar simulator internally consistent?" (yes) instead of "does the bar simulator agree with tick replay?" (no, by +0.29 R/trade).
**Invalidation status:** The audit's "clean" numbers for trailing strategies are suspect. The audit remains valid for what it actually tested (signal-side cleanup).

### V8 Full Audit Report

**Source:** `raw-sources/trading/audits/V8_Full_Audit_Report.md`
**Scope:** V8 mining era audit — phantom B/E bug, numpy bool bug.
**What it cleaned:** `simulate_outcomes()` B/E fill logic, `rej_delta_aligned` string-to-bool conversion.
**What it missed:** Bar-sim trailing bug. V8 was pre-trailing; most V8 results used fixed targets. But any V8 result that used trailing inherited the contamination.
**Rule produced:** [[invalidation-rules|Rule 4]] — phantom B/E fills in `micro_structure.py`.

### V9 Full Audit Report

**Source:** `raw-sources/trading/audits/V9_Full_Audit_Report.md`
**Scope:** V9 improvements over V8.
**What it cleaned:** Builds on V8 fixes. Additional simulator cleanups.
**What it missed:** Same as V8 — bar-sim trailing bug not caught.

### BOS_FVG Audit Findings (early historical)

**Source:** `raw-sources/trading/audits/BOS_FVG_AUDIT_FINDINGS.md`
**Scope:** Early BOS_FVG analysis. Historical.
**Status:** Superseded by the March 10pt audit and the April consolidated audit.

## Audit-to-rule map

| Rule | Audit(s) that produced it |
|---|---|
| [[invalidation-rules\|Rule 1]] — Do not trail on bars | [[bos-fvg-failure-consolidated]] (Apr 2026) |
| [[invalidation-rules\|Rule 2]] — Do not consult unclosed HTF bars | [[look-ahead-audit]] (Feb 2026) |
| [[invalidation-rules\|Rule 3]] — BE stops kill Model 2 edge | BOS_FVG_RESEARCH BE analysis + V10t |
| [[invalidation-rules\|Rule 4]] — Phantom B/E fills | V8 Full Audit Report |
| [[invalidation-rules\|Rule 5]] — P/D must be up-to-signal-time | [[bos-fvg-10pt-audit]] §1 |
| [[invalidation-rules\|Rule 6]] — WF pass ≠ edge | [[bos-fvg-failure-consolidated]] |
| [[invalidation-rules\|Rule 7]] — Multiple comparison correction | [[bos-fvg-10pt-audit]] §6 |
| [[invalidation-rules\|Rule 8]] — Outlier months | [[bos-fvg-10pt-audit]] §5 |
| [[invalidation-rules\|Rule 9]] — NT OnEachTick bar gating | [[v5-strategy-bug-audit]] |
| [[invalidation-rules\|Rule 10]] — Min risk floor | [[comprehensive-mining-report-v2]] |
| [[invalidation-rules\|Rule 11]] — Start sim at idx+1 | [[comprehensive-mining-report-v2]] |
| [[invalidation-rules\|Rule 12]] — Tick supersedes bar | [[bos-fvg-failure-consolidated]] |
| [[invalidation-rules\|Rule 13]] — Forward-link banners | (methodology rule — every audit) |
| [[invalidation-rules\|Rule 14]] — Nautilus cross-check | [[bos-fvg-failure-consolidated]] |
| [[invalidation-rules\|Rule 15]] — Anti-stupidity rules | [[unified-brain-architecture]] |

## The invalidation chain visualized

```
V1 mining (312d, Feb 2026) ────┐
V2 mining (34k trades) ────────┤
BOS_FVG_RESEARCH.md ───────────┤
clean_backtest.py outputs ─────┼──► BOS_FVG_FAILURE_CONSOLIDATED (2026-04-10)
NQ_PLAYBOOK flagship (V10z) ───┤       │
Research Journal narrative ────┤       │
Mining Analysis grid ──────────┘       │
                                       ▼
                              BOS_FVG = DEAD
                              Wiki concept tagged dead-strategy
                              All summary banners updated
                              Rules 1, 12, 14 codified
```

## Related MOCs

- **Tempo research MOC** *(to be created)* — organized by strategy component (IFVG, SMT, DOL, sweep) with the evolution timeline
- **BOS_FVG saga MOC** *(to be created)* — the full invalidation story as a case study
- **[[tempo-three-layers]]** — the framework separating what Tempo teaches from what Harrison's mining found from what's actually implemented

## Open questions

- Is there a subset of BOS_FVG at tick resolution that does have edge? (The 10+pt static audit says no; no subset tested survives Bonferroni. But the signal space is large and un-exhaustively searched.)
- Do the other "V10z 4-signal portfolio" legs (vol_spike_FVG, ORB_breakout, pre_gap_fill) have any edge at tick resolution? They inherit the bar-sim bug proportionally but haven't been individually re-audited.
- Which of the Tempo IFVG numbers in [[tempo-ifvg-research]] are safe from the bar-sim bug? IFVG uses a close-through soft stop (Rule 1 exempt), so most of its backtest results should be structurally clean — but this needs explicit verification.
