# Orderflow Transcript Mining Findings

**Generated: 2026-02-17**
**Sources: 7 transcript folders, ~30 substantial transcripts analyzed**
**Focus: Quantifiable/codeable rules for backtester integration**

---

## Table of Contents
- [A. Entry Confirmation Techniques](#a-entry-confirmation-techniques)
- [B. Stop Placement Methods](#b-stop-placement-methods)
- [C. FVG/Imbalance Trading](#c-fvgimbalance-trading)
- [D. Orderflow Patterns](#d-orderflow-patterns)
- [E. Rejection/Confirmation Patterns](#e-rejectionconfirmation-patterns)
- [F. Session/Time-of-Day Insights](#f-sessiontime-of-day-insights)
- [G. Risk Management Insights](#g-risk-management-insights)

---

## A. Entry Confirmation Techniques

### A1. CLC Rule (Context-Location-Confirmation) -- Carmine Rosato
The foundational framework for entry decisions across all orderflow methods.

**Rules (codeable):**
1. **Context** = Determine trend direction (higher highs/higher lows = uptrend, lower lows/lower highs = downtrend)
2. **Location** = Price must be at a significant level (S/D zone, support/resistance, VWAP, POC)
3. **Confirmation** = Volume/orderflow confirms reaction at location

> "Context, Location, Confirmation. Whenever you hear me say context what I really mean is what is the trend... Point number two is location, is it at a level of interest... Point number three is confirmation, are buyers willing to buy at my level, are buyers trapped, are sellers willing to sell at my level of interest or are sellers trapped." -- Carmine Rosato

**Codeable rule:** Entry = (trend direction confirmed on HTF) AND (price at predefined zone) AND (volume anomaly detected at zone)

### A2. Volume Cluster Pullback Entry -- Trader Dale
Trade pullbacks to areas where heavy volume was previously traded (within trends or rejections).

**Rules:**
1. Identify significant volume cluster (heavy volume area on 30min footprint using volume cells view)
2. Volume cluster must be within a trend or a rejection
3. Price must move away from cluster (at least 1-2 full footprints)
4. Wait for pullback to cluster
5. Enter at first touch in trend direction

> "You look for significant volume cluster... heavy volumes traded in that area and you should look for those volume clusters either within a trend or in a rejection... you wait for pullback and when the price hits this heavy volume area you enter long." -- Trader Dale

**Codeable rule:** Entry when price returns to high-volume node after moving away by >= 2 bars, direction = prior trend direction

### A3. POC Pullback Entry -- Trader Dale
Trade the first pullback to a daily Point of Control.

**Rules:**
1. Identify daily POC (widest part of volume profile)
2. Price must move away from POC establishing directional bias
3. Wait for first pullback to POC (first test only)
4. Enter at POC or at beginning of heavy volume zone (alternative method)

> "The way I trade this is I place the level a little bit higher to the beginning of that heavy volume zone... the reason is that I started to notice that I'm missing many trades. When I was trading from the point of control from that level exactly I was missing a lot of trades." -- Trader Dale

**Alternative entry rule:** Enter at beginning of heavy volume zone rather than exact POC line (catches more trades)

**Do NOT trade POC in rotation:**
> "When you notice that the market is going sideways... you don't want to trade the strategy of point of control pullbacks... point of control in a rotation is usually somewhere around the center of the rotation... the point of control is actually a nice place where to take your profit." -- Trader Dale

### A4. POC Reversal Trade -- Trader Dale
When a POC trade fails, trade the same level from the opposite direction.

**Rules:**
1. POC pullback trade taken and fails (price shoots through)
2. Wait for pullback back to same POC level
3. Enter in NEW direction (opposite of original trade)

> "If point of control fails, then you take a reversal trade... you extend this line a bit. Wait for the pullback and at the same level, the same point of control, you enter a reversal trade." -- Trader Dale

**Best case:** Price shoots through POC fast with no reaction (e.g. during macro news) then pulls back

### A5. Entry in "The Now" -- Carmine Rosato
Do NOT wait for candle close. Enter when volume signal appears in real time.

> "I will never click the buy or sell button strictly off of a Candlestick chart... the time frame that I use to confirm my trades... is the now time frame reading the volume and reading the orderflow... I'm getting in long when I see the buying activity I'm not waiting for the market to rally up." -- Carmine Rosato

**Codeable implication:** Signal generation should be tick-based or sub-bar, not wait for bar close. Entry at first detection of absorption/volume anomaly, not at bar completion.

### A6. Stacked Imbalance Zone Entry -- Michael Valtos
Trade pullback into stacked imbalance zones.

**Rules:**
1. Identify stacked imbalance (3+ consecutive price levels with 4:1 ratio imbalance)
2. Draw zone from stacked imbalance levels
3. Wait for pullback into zone
4. Enter in direction of original imbalance (buying imbalance = long on pullback, selling imbalance = short)
5. Stop loss: just beyond the zone (if price closes on other side, exit)

> "A stacked imbalance is when you have three or more imbalances stacked neatly on top of each other... markets have memory, liquidity has memory so when a market comes back those levels it can give you a good opportunity to either get long or get short." -- Michael Valtos

> "All you got to do is you just throw in a bid here right around the 74 you know a stop at 73 and look what happens... it rallies from 74 all the way up to 94 in about 10 15 minutes." -- Michael Valtos

**Codeable rule:** Zone = price range of 3+ stacked imbalances (4:1 ratio, min 10 contracts). Entry = limit order at zone edge. Stop = 1 tick beyond zone.

### A7. Trigger Point + Delta Unwind Entry -- Axia Futures
Enter on delta unwind after trigger point fails to sustain.

**Rules:**
1. Identify trigger point (trend line break with volume + volatility + price confirmation)
2. If market fails to sustain below trigger, and returns above it
3. Initiate longs above trigger point when delta starts unwinding
4. Stop: below the base of aggressive buying (tight, ~4 pips in FX)

> "Once we got above that 11403... that's where we initiate the Longs... stop limit orders we're looking to buy now and hold why because we're going to get a Delta unwind... if we entering on the break of that 11404 where we stopping out... simply 114 figure 11399 we're talking about four Pips here guys four Pips for a Delta unwind." -- Axia Futures

**Codeable rule:** After trigger break, if price returns above trigger level within N bars, AND cumulative delta flips, enter long with stop at trigger level.

### A8. Exhaustion Print Entry -- Michael Valtos
Enter reversal trades when exhaustion prints appear at swing highs/lows.

**Rules:**
1. Identify exhaustion: very low volume (1-9 contracts for ES, adjustable per market) at the high or low of a bar
2. Must appear at swing high/low or HOD/LOD (not in the middle of a range)
3. Enter in reversal direction after exhaustion confirmed
4. Threshold is market-specific (9 for most, 25-50 for high-volume markets like 10yr)

> "An exhaustion print to me is a sign that the last buyer has bought in an up move or the last seller has sold in a down move." -- Michael Valtos

> "Tom DeMark... said something that always resonated with me... a move is over when the last buyer has bought." -- Michael Valtos

**Codeable rule:** If volume_at_extreme <= threshold AND at swing_high/low, signal reversal. For ES: threshold = 9. For bonds: threshold = 25.

---

## B. Stop Placement Methods

### B1. Stop in Low Volume Area Behind Heavy Volume Barrier -- Trader Dale
The universal stop placement rule from volume profile trading.

> "Stop loss needs to go in low volume area where nobody really was interested in trading... you want to place the stop behind that resistance because if the price goes past the resistance, there's no way of telling where it will go next." -- Trader Dale

**Codeable rule:** Stop = first low-volume node beyond the high-volume zone being traded. If heavy volume zone is support, stop below the low-volume gap beneath it.

### B2. Stop Beyond Stacked Imbalance Zone -- Michael Valtos
For stacked imbalance trades, stop is 1 tick beyond the zone.

> "If the price closes on the other side of the stacked imbalance zone... exit." -- Michael Valtos (from session summary)

**Codeable rule:** Stop = zone_low - 1 tick (for longs) or zone_high + 1 tick (for shorts)

### B3. Stop at Trade Invalidation Point -- Axia Futures / Carmine Rosato
Stop where the thesis is invalidated, not at an arbitrary distance.

> "You don't need 40 pip stops you don't need 50 pip stops what you need is a strategy that gives you the precise access point to access a trade opportunity with risk that invalidates the trade." -- Axia Futures

> "I have a onepoint stop loss because I'm trading at an area... if I'm short right here and I'm viewing this activity as actual sellers in the market for me to be wrong on this my stop loss is above this High." -- Carmine Rosato

**Codeable rule:** Stop = just beyond the volume anomaly / absorption level that triggered the entry. For Carmine's approach on ES: typically 1-2 points.

### B4. Stop Below Absorption / Large Volume Node -- Carmine Rosato
Place stop under the large passive order that signals your entry.

> "Placing my stop loss under this massive sell order which is technically a buyer cuz the Market's going to Rally I'm putting my stop loss directly under this very large volume." -- Carmine Rosato

**Codeable rule:** Stop = price_of_absorption_node - buffer (1-2 ticks)

---

## C. FVG/Imbalance Trading

### C1. Diagonal Imbalance Detection -- Trader Dale
Footprint imbalances are read DIAGONALLY, not horizontally.

**Rules:**
- Compare bid at price N to ask at price N+1 (diagonal comparison)
- If ratio >= 3:1 (300%), mark as imbalance
- Blue/highlighted = imbalance

> "You read the footprints diagonally. So for example here you compare 18 to zero... And if one of the numbers is three times or more bigger than the other number which you compare it to, then it gets highlighted... 300% which means I look for values which are three times or more bigger." -- Trader Dale

**Codeable rule:** imbalance = bid[price_N] / ask[price_N+1] >= 3.0 OR ask[price_N+1] / bid[price_N] >= 3.0

### C2. Stacked Imbalance Definition -- Trader Dale / Michael Valtos
Three or more consecutive imbalances stacked vertically.

> "This is called stacked imbalance where three imbalances are at top of each other... it shows strength of sellers, right? Because here sellers are dominating." -- Trader Dale

> "A stacked imbalance is when you have three or more imbalances stacked neatly on top of each other." -- Michael Valtos

**Codeable rule:**
```
if consecutive_imbalances_same_direction >= 3:
    mark_stacked_imbalance(direction, price_range)
```

### C3. Imbalance Ratio and Minimum Volume -- Michael Valtos
Standard ratio is 4:1 with minimum volume of 10 contracts.

> "People generally use a ratio of about four to one some people use three to one some people use five to one... sort of the industry standard if there is one is about four to one." -- Michael Valtos

> "One thing you know depending on the market you're trading you may want to use a minimum volume for the imbalance... I use 10... even though I'm using four to one if I have one against five technically that's an imbalance of over four to one but doesn't meet the minimum volume requirement." -- Michael Valtos

**Codeable parameters:**
- `imbalance_ratio`: 4.0 (default), range 3.0-6.0
- `min_imbalance_volume`: 10 contracts (filter noise on thin markets)

### C4. Buying Imbalances on Red Candles = Supply/Absorption -- Michael Valtos
Counter-intuitive but critical: buying imbalances appearing on bearish candles indicate passive sellers absorbing aggressive buying.

> "When you have a buying imbalance on red candles... what you actually have coming into the market is Supply... that Supply is absorbing whatever aggressive buying is going on... it's a sign of absorption you have sellers that are absorbing the aggressive buying." -- Michael Valtos

**Codeable rule:** If buying_imbalance AND candle_bearish: signal = "absorption/supply" (bearish despite buying)

### C5. Selling Imbalances on Green Candles = Support/Absorption -- Michael Valtos
Selling imbalances on bullish candles indicate passive buyers absorbing aggressive selling.

> "Selling imbalances into green candles... strong passive buying here on the bid side... passive bids are absorbing that aggressive selling that's what's causing this selling imbalance... buyers are stepping up to absorb whatever aggressive selling." -- Michael Valtos

**Codeable rule:** If selling_imbalance AND candle_bullish: signal = "support/absorption" (bullish despite selling)

### C6. Imbalance Zeros = Stops/Sweeps -- Axia Futures
Zeros on one side of the footprint indicate stop runs or aggressive sweeps.

> "Notice what happens on the break... he's so intent on getting filled that he goes to the next price and the next price he doesn't let sellers step in and sell and that's why we've got the zeros there. Now whenever we see a zero we call this an imbalance." -- Axia Futures

**Codeable rule:** If volume_at_price_one_side == 0 AND volume_other_side > threshold: mark as sweep/stop_run

---

## D. Orderflow Patterns

### D1. Absorption Pattern -- Trader Dale / Bookmap
Heavy volumes on BOTH bid AND ask at the same price level(s). Buying or selling pressure is being absorbed.

**Detection rules:**
1. Unusually large volume on both bid AND ask at the same price level
2. Price stops moving (stalls)
3. Eventually, the absorbed side runs out of fuel and price reverses

> "Absorption basically tells you that either buying or selling pressure is being absorbed... you will see huge volumes both on bid and ask... heavy volumes in this example it is telling us that buyers were pushing the price upwards and we see their aggressive orders on ask. But at the same time at this price level sellers started to sell aggressively as well." -- Trader Dale

> "You don't have exact values for each instrument what the unusually huge volumes are because every session is different... but if you look at the chart and look at what the usual or standard volume is... then you'll know that when you see something like this that this is unusually huge." -- Trader Dale

**Codeable rule:** absorption = (bid_volume > 2 * avg_bid_volume) AND (ask_volume > 2 * avg_ask_volume) AND price_stalling

### D2. Delta Divergence -- Trader Dale / Axia Futures / Michael Valtos
Price moving in one direction while delta moves in the opposite direction.

> "The divergence between price and delta is that we are looking at bullish footprint and the delta is negative... basically price goes up, delta goes down. That's the divergence." -- Trader Dale

> "If price is rising and delta falling then buyers are passive in the move higher... sellers are in control there's more people actively selling as the price rises rather than buyers actually hitting into the offers." -- Axia Futures

**Codeable rule:**
```
bearish_divergence = price_making_higher_highs AND cumulative_delta_making_lower_highs
bullish_divergence = price_making_lower_lows AND cumulative_delta_making_higher_lows
```

### D3. Delta Reversal Pattern -- Michael Valtos
At swing points, look for large max/min delta that reverses to near zero within the same bar.

**Detection rules:**
1. At swing high: bar shows large positive max delta (e.g., +500) but closes near zero or negative
2. At swing low: bar shows large negative min delta (e.g., -500) but closes near zero or positive
3. Visual: long wick on delta candle with small body
4. Must be at a reversal point (swing high/low, HOD/LOD), NOT in middle of range

> "For a Buy Signal... you're looking for a new low being made... you're going to look at the Delta the internal Max and Min Delta and you're going to compare it to the final Deltas... you're looking for a big negative Delta that reverses to near zero." -- Michael Valtos

> "Really what you're looking for are those big long wicks above or below with a small body... like this one here 728 Max Delta got all the way back down to 19." -- Michael Valtos

**Codeable rule:**
```
buy_signal = (min_delta < -threshold) AND (close_delta > -50) AND at_swing_low
sell_signal = (max_delta > threshold) AND (close_delta < 50) AND at_swing_high
# threshold = relative to recent bar delta history
# band of +/- 50 treated as neutral for liquid markets like ES
```

### D4. Stopping Volume -- Michael Valtos
Heavy volume concentrated at 2-3 price levels at the high or low of a bar, representing 25%+ of bar's total volume.

> "587 on top, 287 beneath it... this bar is trading just under 4000 contracts... and literally a quarter of your volume is just happening in these three price levels right in here at the high... that's what stopping volume looks like." -- Michael Valtos

> "If it's a small number between one and zero so 0.48... that's telling me stopping volume... if it was a very high number like 58 then it's price exhaustion." -- Michael Valtos

**Codeable rule:**
```
stopping_volume = (volume_at_top_3_prices / total_bar_volume) >= 0.25 AND at_bar_extreme
# Ratio 0-1 = stopping volume (concentrated at extreme)
# High ratio = price exhaustion (distributed, nothing at extreme)
```

### D5. Trapped Traders -- Michael Valtos
Two types of traps: stop-triggered and absorption-based.

**Type 1: Stop-triggered trap**
- Price sweeps through a level triggering stops
- No follow-through after the sweep
- Sweep appears as stacked buying/selling imbalances at extreme with zero counter-trade

**Type 2: Absorption trap**
- Aggressive buying/selling hits a passive wall (iceberg/limit order)
- Price cannot advance despite heavy aggressive volume
- Traders realize they're trapped and reverse, adding momentum to the reversal

> "500 contracts of aggressive buying and Market can't move higher what's the trade? To the short side." -- Michael Valtos

> "If these 2000 contracts... keep buying it thinking it's going higher and at some point they're going to realize that Market's not going any higher and they're going to turn sellers and get out and that's going to add pressure to the market." -- Michael Valtos

**Codeable rule for no-follow-through trap:**
```
trap_signal = (strong_buying_imbalance at extreme) AND (next_bar_delta_reversal) AND (no_price_continuation)
# Previous bar: max_delta = very positive, min_delta = 0
# Next bar: max_delta = 0, min_delta = very negative
```

### D6. Key Auction Reversal -- Axia Futures
Equal and opposite delta switch at an extreme, signaling trend reversal.

**Detection rules:**
1. Volatile candle in trend direction with high volume and high delta
2. Next candle: similar volume, delta flips to equal-and-opposite
3. Price confirms reversal direction

> "A key auction reversal the underlying characteristics... we get this volatile candle to the upside with high volume and high Delta, we then get the switch... an equal and opposite similar volume, the Delta is similar amount to the previous one but negative... and then we get a price reaction." -- Axia Futures

**Codeable rule:**
```
key_auction_reversal = (
    bar_N.delta > threshold AND bar_N.volume > avg_volume * 1.5
    AND bar_N1.delta < -threshold AND bar_N1.volume > avg_volume * 1.5
    AND abs(bar_N.delta + bar_N1.delta) < threshold * 0.3  # roughly equal and opposite
)
```

### D7. Initiative Drive / Open Drive -- Axia Futures
Strong directional move off the open with imbalances confirming the break.

> "What we can see is that just after the break a buyer absorbs a thousand four hundred but he also lifts a thousand five hundred... at one price we've transacted three thousand in the favor of the buyer... he goes to the next price and the next price... and that's why we've got the zeros there." -- Axia Futures

**Pattern:** Break of initial balance high/low with:
- Imbalances (zeros on counter side)
- 2-3x average volume at break point
- Continuation expected; trade pullback to break level

### D8. Volume Tail / Failed Breakout -- Carmine Rosato
Dissipating volume on a breakout indicates failure.

> "As the market breaks out the volume dissipates and the volume gets weaker... this volume tail shows that nobody cares about the market above this resistance level if no buyers care about this move then sellers are going to be forced to lower their prices." -- Carmine Rosato

**Codeable rule:** If breakout_candle AND volume_decreasing_per_tick_into_breakout: signal = "failed breakout" (fade)

### D9. Elephant/Iceberg Orders -- Axia Futures
Large bid/offer that reloads when hit, signaling institutional defense.

> "Look at the 400 lot orders on the bid in the oil... market came down took his four hundred lots but he reloaded for another 330... we call this an elephant we call this an iceberg order... once the market realized that elephant was there we always had the sellers back off." -- Axia Futures

**Not directly codeable from historical data** but the effect shows as stopping volume with consistent bid-side absorption at a single price level.

---

## E. Rejection/Confirmation Patterns

### E1. Support-to-Resistance Confirmation -- Axia Futures
Support must show INTERACTION (multiple tests with volume), not just a line.

**Rules:**
1. Support requires 12+ tests with buying interaction
2. Break must show range expansion + volume spike + price confirmation
3. ALWAYS wait for retest (don't sell the break aggressively)
4. On retest: look for key auction reversal (sellers engaging where buyers previously were)

> "So 12 attempts at this area... 12 attempts to try and break lower the market then sprung back... we can see this market has a significant area of support here not because I say so not because of the lines but because of the interaction that took place." -- Axia Futures

> "Every good breakout strategy always gives a signal... there's always a signal on the footprint that something is coming." -- Axia Futures

**Codeable rule:** Support = price level tested >= N times with volume interaction. Break = volume > 2x average + range expansion + close beyond level.

### E2. Trigger Point Confirmation -- Axia Futures
Three requirements for a valid trigger/breakout:

> "A trigger has three rules: one we need volume... two we need volatility... three we need price confirmation... we see those three things we know we've now got a trigger in the market." -- Axia Futures

**Codeable rule:**
```
valid_trigger = (
    volume_at_break > 2 * rolling_avg_volume
    AND range_at_break > 1.5 * avg_range
    AND close_beyond_trigger_level
)
```

### E3. Aggressive Selling With No Reward = Reversal Signal -- Carmine Rosato
The core reversal signal: aggressive volume in one direction with no price follow-through.

> "What I am looking for is unusual activity or outliers in my data... if a lot of selling is coming in at the low of the day... an auction you would see it continue moving lower. Now... despite all of this selling down here the market actually bounced... these sellers at the low became trapped." -- Carmine Rosato

> "Tons of effort but no reward by those sellers and this is the only option for the market to do is then move up." -- Carmine Rosato

**Codeable rule:**
```
reversal_signal = (
    delta_at_price > outlier_threshold  # e.g., > 2 std dev from mean
    AND price_direction != delta_direction  # price didn't follow
    AND at_significant_level  # HOD, LOD, S/D zone
)
```

### E4. Head and Shoulders + Delta Divergence -- Axia Futures
Classic pattern enhanced by delta divergence has higher win rate.

> "If you play every single head and shoulders it's got a reasonably low win rate but if you play a head and shoulders that has now got cumulative delta dropping i.e sellers are now in control you boost your win rate." -- Axia Futures

**Codeable rule:** Standard H&S pattern detection + cumulative_delta_declining through formation = higher probability signal

### E5. VWAP Setups -- Trader Dale
Two modes: Rotation (trade deviation touches) and Trend (trade pullback to first deviation).

**Rotation setup:**
- Market rotating around VWAP
- Go long at lower deviation, short at upper deviation
- Take profit at VWAP (center/magnet)

**Trend setup:**
- Market trending away from VWAP
- Trade pullback to first standard deviation
- Enter in trend direction

> "Wait 5 hours for daily VWAP to develop." -- Trader Dale (from session summary)

**Codeable rule:** If price touches VWAP +/- 1 std dev AND trend confirmed: enter in trend direction.

---

## F. Session/Time-of-Day Insights

### F1. Cash Open = Highest Volume/Volatility -- Multiple Sources
The cash open (9:30 ET for US equities, 2:30 GMT for European) provides the highest volume and best setups.

> "Each one of these is obviously a five minute rotation... the market opens and we shoot down... what we're looking for was the break of that initial balance high." -- Axia Futures

### F2. Initial Balance (First Hour) -- Axia Futures
The first hour establishes the initial balance range. Breaks of IB high/low are significant.

> "The initial balance seven until eight o'clock UK time for European equities... it gives you the first hour before the cash open." -- Axia Futures

**Codeable rule:** IB_high and IB_low = high/low of first 60 minutes. Break above IB_high or below IB_low with volume = continuation signal.

### F3. Wait 5 Hours for Daily VWAP -- Trader Dale
Daily VWAP needs time to develop before it becomes meaningful.

### F4. U.S. Session Preferred for Orderflow Signals -- Michael Valtos
Overnight sessions have less volume, making orderflow signals less reliable.

> "I find it better during the U.S hours rather than during the night session... you just don't get enough movement in my opinion for the night session for a Delta reversal." -- Michael Valtos

> "I prefer to look at them the stacked imbalances during the U.S sessions as opposed to the overnight sessions where there's less volume." -- Michael Valtos

### F5. Post-Close Volume Anomalies -- Michael Valtos
Large volume appearing after cash close can signal overnight/weekend direction.

> "When I see size like that coming in after the close it sort of portends what's going to happen... that big volume come in sort of post cash close which is something you don't normally see." -- Michael Valtos

### F6. Gap Close Trades -- Axia Futures
Gap close becomes a "live trade" only once price returns inside the initial balance.

> "The gap close is always on until the gap is closed but the trade doesn't become live until you get back realistically inside the initial balance." -- Axia Futures

---

## G. Risk Management Insights

### G1. Take Profit Before Next Heavy Volume Zone -- Trader Dale
Exit before the next potential support/resistance barrier.

> "You want to take the profit before the price reaches heavy volume zone... the rule for the stop loss is stop loss goes behind a barrier and take profit goes before a heavy volume barrier." -- Trader Dale

**Codeable rule:** TP = next high-volume zone in trade direction - buffer

### G2. Targets: HOD, LOD, VWAP, Next S/D Zone -- Carmine Rosato
Specific, predetermined exit targets rather than trailing.

> "Majority of my exits come from higher days, lower days, VWAP, the next supply or demand zone or the next support resistance." -- Carmine Rosato

**Codeable rule:** TP candidates = [HOD, LOD, VWAP, nearest_SD_zone, nearest_VP_zone]. Select closest in trade direction.

### G3. Minimum Risk:Reward of 1:1, Target 2:1+ -- Trader Dale / Carmine Rosato

> "My rule for this is that I want to trade at least with risk reward ratio one... that's the minimum." -- Trader Dale

> "I'm always maintaining a 2 to 1 risk of reward or better." -- Carmine Rosato

**Codeable filter:** Only take trades where (distance_to_TP / distance_to_SL) >= 1.0 (minimum), prefer >= 2.0

### G4. Tight Stops From Precise Orderflow Entries -- Carmine Rosato
Orderflow precision allows much tighter stops than chart-based entries.

> "If I got short at the lower oval my stop loss would had to be about six points versus me getting short at the upper oval my stop loss only being a one point." -- Carmine Rosato

**Implication for backtester:** With orderflow confirmation, stop distances can be 1-2 points on ES rather than 5-7 points. This dramatically improves R:R.

### G5. One Strategy Is Enough -- Axia Futures
Don't try to master everything. One reliable pattern traded consistently beats 10 half-learned patterns.

> "You only need one okay you only need one strategy to be successful you only need to identify one strategy... make money out of one strategy with leverage to really be successful." -- Axia Futures

### G6. Don't Preempt Delta Reversals -- Axia Futures
Stay with the trend until the trigger point is actually invalidated.

> "Never preempt a Delta reversal... stay with the trend stay with the momentum if it comes back above that trigger point then we get cracken." -- Axia Futures

### G7. Trace Filter Settings -- Trader Dale
Filter small orders to see only institutional activity.

> "I set the filter to show only trades which are bigger than 25 contracts... recently I had to change it to 35 because with 25 it just gave me too many signals per day." -- Trader Dale

**Codeable rule:** Filter trades below threshold. ES: 25-35 contracts. Adjust per instrument.

### G8. Trade Only First Test of Levels -- Trader Dale
First touch has highest probability of reaction.

> "You enter the trade at first touch... you also want to trade only the first test. That's at least how I trade this... they have higher probability of a successful reaction." -- Trader Dale

**Codeable rule:** Mark levels as "spent" after first test. Do not re-trade same level.

---

## Source Attribution Summary

| Educator | Source Folder | Key Contributions |
|----------|--------------|-------------------|
| **Trader Dale** | trader-dale/ | Volume clusters, POC trading, absorption, delta divergence, imbalance ratios (3:1), stop/TP placement, VWAP setups, trace filter |
| **Axia Futures** | axia-futures/ | Key auction reversal, delta unwind, trigger points, support-to-resistance, cumulative delta divergence/confirmation, initiative drives, footprint strategies |
| **Michael Valtos** | orderflows-michael-valtos/ | Stacked imbalances (4:1), exhaustion prints, stopping volume, delta reversals, trapped traders, absorption via delta, inverse imbalances |
| **Carmine Rosato** | carmine-rosato-orderflow/ & carmine-rosato-supply-demand/ | CLC Rule, entry in "the now", footprint delta analysis, volume tails, supply/demand zones, tight stop methodology |
| **Bruce Pringle (Bookmap)** | bookmap/ | Liquidity visualization, sweep mechanics, book flipping, absorption/exhaustion at micro level, historical order book context |

---

## Priority Codeable Patterns for Backtester

### Tier 1 -- Directly Implementable
1. **Stacked imbalance zones** (4:1 ratio, 3+ levels, min 10 vol) as entry zones on pullback
2. **Exhaustion prints** (volume <= threshold at extreme) as reversal signals
3. **Delta divergence** (price vs cumulative delta direction mismatch)
4. **Volume cluster pullback** (heavy volume node + pullback = entry)
5. **POC pullback** (first test only, skip in rotation)
6. **Stop in low-volume area** behind heavy volume barrier

### Tier 2 -- Requires More Complex Logic
7. **Absorption detection** (high volume both sides + price stall)
8. **Key auction reversal** (equal-and-opposite delta switch)
9. **Stopping volume** (25%+ of bar volume at 2-3 price levels at extreme)
10. **Trapped traders / no follow-through** (aggressive volume + opposite delta next bar)
11. **Support-to-resistance flip** (N tests + break + retest)

### Tier 3 -- Requires Real-Time Data
12. **Delta unwind** after trigger point (needs sub-bar execution)
13. **Entry in "the now"** (tick-level entry, not bar-close)
14. **Elephant/iceberg detection** (reloading limit orders)
15. **Volume tail / failed breakout** (dissipating volume into breakout)
