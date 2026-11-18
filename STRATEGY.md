# Strategy: cbBTC two-rung dip ladder

## Idea

cbBTC mean-reverts after sharp slides on the one-to-two-hour scale, but a
slide that is still falling eats a full-size dip buy alive. This bot buys in
two rungs and scales out in two steps, so a shallow dip costs little and a
deep dip gets the full size near the bottom:

- **Rung 1:** trailing 60-minute return at or below -110 bps: buy 40% of
  cash.
- **Rung 2:** trailing 120-minute return at or below -210 bps: top the
  position up to 100% of cash (either as the first buy, or as an add-on
  while rung 1 is held).
- **Scale out:** sell half the position at +180 bps over the average cost
  (fees included), the rest at +200 bps.
- **Safety nets:** trailing stop 150 bps below the peak close since entry,
  hard stop 200 bps below the average cost, max hold 480 bars.
- **Guardrail:** if no buy has ever filled by bar 1425, buy 25% of cash,
  so the entry still clears the >= 1 trade bar on a quiet day.
- A buy the market refuses is retried while its signal holds; a sell the
  market refuses is retried while its exit holds. Max slippage 300 bps on
  every order (never binds on 1-minute data; it only guards a gap open).

## Why these numbers

Swept with a local fill-model simulator (4 bps fee, 2 bps slippage,
constant-product impact against each bar's open depth) on TWO frozen
24-hour windows and picked the config by its worst window, not its best:

| config | window 16-17 Nov | window 17-18 Nov |
|--------|-----------------:|-----------------:|
| single-rung dip -100 bps / TP +120 (round-3 shape) | +$27.22 | -$16.38 |
| deep-dip only, 120m -210 / TP +250 | -$0.82 (guardrail only) | +$15.25 |
| **this ladder** | **+$14.57** | **+$17.70** |

The plateau around the chosen point is wide: rung-1 fraction 0.30-0.40,
first take 170-190 bps, second take 180-220 bps, trail 120-180 bps and
deep-dip threshold -200 to -220 bps all stay above +$11 on the worst
window. The hard stop never triggers on either window; it is a safety net
for the race window, not fitted to it. Max hold 480 matters: cutting it to
360 drops the 17-18 Nov window to +$5.77 because the big trade needs the
room to ride its rally.

## Risks

- Two windows are two samples. A race day with no -210 bps 120-minute
  slide trades only rung 1 and the guardrail; a slide that keeps falling
  past the top-up hits the -200 bps stop.
- Max drawdown is small on both windows (0.7%) because positions are
  short-lived and the stop caps the loss per trade at ~200 bps plus costs.
- Deterministic: integer arithmetic only, no randomness, no clock.
