---
created: 2026-04-10
updated: 2026-04-10
type: concept
sources:
  - raw-sources/COMPREHENSIVE_MINING_REPORT.md
related:
  - wiki/concepts/mtf-alignment.md
  - wiki/concepts/bos-fvg.md
  - wiki/summaries/comprehensive-mining-report-v2.md
  - wiki/summaries/strategy-mining-report-312d.md
  - wiki/syntheses/mining-reports-v1-v2-reconciliation.md
tags: [trading, bug, look-ahead, critical, nq]
---

# V10i Look-Ahead Bug

A critical look-ahead bias in the V10i `get_alignment()` function that inflated WR across an entire era of NQ mining research. Documented in [[comprehensive-mining-report-v2|COMPREHENSIVE_MINING_REPORT V2]] after V2 cleanup revealed that some "edges" were pure lookahead.

## The bug

```python
# WRONG — picks the currently-open HTF bar
mask = tf_c.index <= candles_1m.index[entry_idx]
aligned = tf_c[mask].iloc[-1]['close'] > tf_c[mask].iloc[-1]['open']
```

The filter `tf_c.index <= candles_1m.index[entry_idx]` selects the higher-TF bar whose START time is ≤ the entry time — but that bar **hasn't closed yet**. Example:

- 5M bar timestamped `10:00` covers the window 10:00–10:05
- Entry happens at `10:02`
- The filter selects the 10:00 bar because `10:00 <= 10:02`
- The `close` field of this bar is the 10:05 close — **three minutes of future data**

For a 1H bar starting at `10:00` and an entry at `10:03`, you're using a close that's 57 minutes in the future.

## The fix

Only use bars whose **end** time ≤ entry time:

```python
# CORRECT — only fully closed bars
mask = tf_c.index + TF_DURATION[tf_label] <= entry_time
```

Or switch to a completed-bar 3-bar HH/HL trend pattern — what the V2 rerun did.

## Impact (measured)

From the V2 stress test:

| Strategy | Original n | Original WR | Fixed n | Fixed WR | Fixed R |
|---|---|---|---|---|---|
| FA ATR 1.5x + al≥5 + co≤0 | 2,035 | 91.1% | 340 | 37.6% | -110R |

- **Entry trigger count unchanged** (16,583 both versions) — the bug didn't add or remove signals
- **Only the alignment labels changed** — inflated alignment counts because future close ≠ past close
- **0/13 green months after fix** — the entire edge was look-ahead
- **Sniper tier configs with 100% WR evaporated** once the unclosed-bar bars were excluded

## What was contaminated

Any research using V10a–V10n alignment logic (and the V1 312d [[strategy-mining-report-312d|mining report]]) is **suspect**. The [[bos-fvg|BOS_FVG]] signal itself survives the cleanup (63.5% WR V2 vs 97.9% WR V1) — the signal is real, but its magnitude was massively overstated. See [[mining-reports-v1-v2-reconciliation]] for the full reconciliation.

## How to detect this class of bug

Red flags:
- **Headline WR > 90%** on a mechanical signal with no discretionary filter
- **"100% WR" tiers** on n ≥ 50
- **WR monotonically improves** with filter strictness well past where sample-size noise should stop rewarding you
- **Alignment counts match trigger counts** across buggy/fixed versions (bug affected *labels*, not signals)

Audit protocol: recompute the claimed edge on an out-of-sample period, with alignment computed *only* on bars whose end time is strictly less than entry time. If the edge vanishes, it was lookahead.

## Why "V10i" specifically

The bug was identified during the V10i mining variant run. All V10a through V10n-era alignment results are retroactively invalidated. V10o onward uses the fixed logic.
