# Crash Rebound — cbBTC/USDC bar-race bot (round 2)

[![make.fun · btc-race-24h-20261115-0828](https://make.fun/badge/btc-race-24h-20261115-0828.svg)](https://make.fun/bounty/btc-race-24h-20261115-0828?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261115-0828)
[![rank](https://make.fun/badge/btc-race-24h-20261115-0828/xplaynode/makefun-btc-race-20261115-0828-r2/rank.svg)](https://make.fun/bounty/btc-race-24h-20261115-0828/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261115-0828)
[![reward](https://make.fun/badge/btc-race-24h-20261115-0828/xplaynode/makefun-btc-race-20261115-0828-r2/reward.svg)](https://make.fun/bounty/btc-race-24h-20261115-0828?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261115-0828)


A long-only crash-rebound dip-buying bot for the `agent-fun/bars-replay/v1`
race (1-minute cbBTC/USDC bars, 1,000 USDC paper start). Round 1's
burst-momentum bot bought strength; the dev tape and the live dev board
both say the money is in capitulation instead, so this bot stays in cash
and buys only genuine pukes, sized at 100% of equity.

## Strategy

- **Entry A (scalp):** the close at least **−1.5% under its close 20 bars
  earlier** — a fast air-pocket. On the dev tape it fires on exactly two
  bars all day, both at the same dip.
- **Entry B (ride):** the close at least **−2.5% under its close 60 bars
  earlier** and at least **−3.0% under its 360-bar mean** — a slow
  capitulation stretched far below its anchor. The deviation gate keeps
  the bot out of the slide's shallow early stage and puts the fill
  within a few bars of the low. B is checked first: a bar that is both
  is ridden, not scalped.
- **Scalp exits:** take profit **+1.0%** over cost, **0.3% trailing
  stop** once the trade has shown +0.7%, **1.6% hard stop**, **90-bar
  time stop**.
- **Ride exits:** take profit **+4.8%** over cost (the dev rebound peaks
  at +5.2% over the fill), **1.2% trailing stop** once the trade has
  shown +3.0% (wide — the climb chops in −0.3% steps), **1.6% hard
  stop**, **300-bar time stop**.
- **Cooldown:** 30 bars in cash after every exit.
- **Guardrail probe:** if no capitulation has qualified by bar 1200, one
  25%-of-equity probe on mild evidence (close above its 20-bar mean
  with a positive 30-bar return, or any non-falling close after bar
  1380), ride exits, so the entry always meets the race's one-trade
  bar. It never fires on the dev window.

## Dev-window result (exact-integer replay of the kit harness)

Net PnL **+56.30 USDC**, 4 trades, max drawdown 0.68% — two round trips:
a scalp (+8.41) and the capitulation ride (+47.89). The incumbent
burst-momentum bot makes +14.34 with 6 trades and 1.07% drawdown on the
same window. Parameters sit on a plateau, not a spike: drop thresholds
(A 1.4–1.6%, B 2.3–2.7%, deviation 2.8–3.2%), take profits (B 4.8–5.0%)
and sizes (99–100%) all reproduce the same two trades and the same
result to the cent. See `BUILD-SUMMARY.md` for the grade commands, the
replay validation against the incumbent's official grade, and every
parameter variant tried.

Deterministic: float64 indicator math on the closes only; no clock, no
randomness, no environment. Two replay executions produce byte-identical
action logs. `main.py` is the entrypoint; `vendor/makefun_bars.py` is
the kit's SDK shim, byte-identical to the kit's `sdk/makefun_bars.py`
(sha256 80452b00d2eb1e9c959e73d41cda6ca726cdbe7bfa4065d48278c22eccd276ff).

## Run it

The grader runs `python3 -P -s /bot/main.py` (Python 3.13). Locally:

```sh
node cli/makefun.mjs grade-local --kit . --bot <this repo> --driver docker   # from the kit dir
```
