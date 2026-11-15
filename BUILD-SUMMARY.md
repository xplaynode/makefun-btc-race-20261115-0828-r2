# BUILD-SUMMARY — btc-race-24h-20261115-0828 round-2 entry ("Crash Rebound")

Built 2026-11-15 (WITA) as the style-pivot replacement for the round-1
burst-momentum incumbent. **Not submitted** — build + local grading only.

## Result (dev window)

| Bot | Net PnL | Trades | Max drawdown | Round trips |
| --- | --- | --- | --- | --- |
| **This entry (r2 dip-buy)** | **+56.296447 USDC** | **4** | **0.680%** | +8.407577, +47.888870 |
| Incumbent r1 "Burst Rider" (replay) | +14.341709 USDC | 6 | 1.072% | −5.441060, −3.546264, +23.329033 |
| Incumbent r1 official dev grade (for reference) | +14.34 USDC | 6 | 1.07% | −5.44, −3.55, +23.33 (per its README) |

Fees paid by r2: 1.629577 USDC. Rejected orders: 0. Determinism: two
replay executions produced byte-identical action logs
(sha256 32a273b861c96c22a41524e11a43412081192010d0688ae8aaa975c1eafda0ec).
The r2 result sits in the live dev leaders' zone (+$55–58) with the same
trade count (4) as the #1 entry.

## How it was graded (exact commands)

The kit's official local grade cannot execute a Python bot on this VM —
both documented attempts were run and failed on the sandbox, not the bot:

```sh
cd ~/workspace/makefun/btc-race-24h-20261115-0828-kit
node cli/makefun.mjs grade-local --kit . --bot ~/workspace/makefun/btc-race-r2-entry --driver docker --worktree
# -> kit verified 29 files; dataset re-derived (1440 bars from 38,815 samples);
#    then: "makefun grade-local: sandbox: docker is not runnable: spawnSync docker ENOENT"
node cli/makefun.mjs grade-local --kit . --bot ~/workspace/makefun/btc-race-r2-entry --driver process --worktree
# -> "makefun grade-local: sandbox: the process sandbox runs node22 only (python3.13 runs only inside Docker)"
```

Grading therefore used an exact-integer (BigInt-equivalent) Python port
of the harness's own engine — `sim/bars/engine.ts` from the kit's
`harness/harness-src.tar.gz`: the `BarsPool` constant-product impact
fractions, the buy/sell fill formulas (4 bps fee, 2 bps slippage),
the decision checks (min notional, cash, max weight 10000 bps, 1 open
position, 16 orders/reply, after_end), fills at the next bar's open,
flatten at the last close against `c_depth`, and equity marks /
max-drawdown exactly as `sim/metrics.ts` defines them. The port drives
the real `main.py` + vendored shim as a subprocess over the
agent-fun/bars-replay/v1 wire protocol. Scripts kept at
`~/workspace/makefun/btc-race-r2-replay/` (`replay.py`, `sweep*.py`):

```sh
python3 ~/workspace/makefun/btc-race-r2-replay/replay.py ~/workspace/makefun/btc-race-20261115-entry --twice
# -> net_pnl_usdc 14.341709, trades 6, max_drawdown_pct 1.0721, deterministic true
python3 ~/workspace/makefun/btc-race-r2-replay/replay.py ~/workspace/makefun/btc-race-r2-entry --twice
# -> net_pnl_usdc 56.296447, trades 4, max_drawdown_pct 0.6799, deterministic true
```

**Replay validation:** the port reproduces the incumbent's *official*
dev grade to the cent — PnL +14.341709 vs +14.34, 6 trades, drawdown
1.072% vs 1.07%, and round trips −5.44 / −3.55 / +23.33 exactly as the
incumbent's README records. The same code graded r2, so r2's numbers
are on the harness's own fill math, not an approximation.

## Final parameters (in `main.py`)

- Entry A (scalp): close ≤ −1.5% vs close 20 bars earlier.
- Entry B (ride, checked first): close ≤ −2.5% vs close 60 bars
  earlier AND close ≤ −3.0% vs the 360-bar mean (SMA360).
- Warm-up: no entries before bar 362. Size: 100% of equity. One
  position, no adds. Cooldown 30 bars after any exit. Max 24 entries.
- Scalp exits: TP +1.0% over cost; trailing stop 0.3% under best close
  armed at best return ≥ +0.7%; hard stop −1.6%; time stop 90 bars.
- Ride exits: TP +4.8% over cost; trailing stop 1.2% under best close
  armed at best return ≥ +3.0%; hard stop −1.6%; time stop 300 bars.
- Guardrail probe (never fires on dev): if no entry by bar 1200, one
  25%-of-equity probe on mild evidence (close > SMA20 and 30-bar
  return > 0, or any non-falling close after bar 1380), ride exits.
- Slippage cap 300 bps on every order. Deterministic float64 math on
  closes only; no clock/env/randomness.

Trades on dev: scalp bought at bar 825's open (decision bar 824),
sold at bar 836's open (+8.41); ride bought at bar 1032's open
(decision bar 1031, the capitulation low is bar 1037's close), sold
at bar 1114's open on the +4.8% take-profit decision at bar 1113 —
the rebound's peak-close bar (+47.89).

## Parameter variants tried (all graded on the dev window only)

| Variant | Net PnL | Trades | DD | Notes |
| --- | --- | --- | --- | --- |
| Incumbent burst-momentum (r1, replay) | +14.34 | 6 | 1.07% | style baseline to beat |
| Dip v1: single branch drop20 + SMA360 dev 3.0%, unified trail 0.9% exit | +36.57 | 2 | — | capitulation only; trail cut the rebound early |
| Dip v1 + EMA240 dev 2.5% branch | +35.51 | 4 | — | extra early trade −1.03 |
| Dip v1 + SMA120 dev 2.0% branch | +34.04 | 4 | — | extra early trade −2.45 |
| Dip v1 with TP +5.0% exit instead of trail | +45.33 | 2 | 0.51% | single capitulation trade, near-full capture |
| Two-branch entries, unified two-mode trail (switch +3.0%, small trail 0.3%) | +25.25 | 4 | 1.02% | small trail killed the ride at the −0.33% dip on bar 1066 (best return +2.92%, just under the switch) |
| Same, small trail 0.4% | +48.95 | 4 | 1.32% | ride saved (+47.37) but scalp exited late (+1.57) |
| Two-branch, switch +2.0–2.5%, size 100% | +52.33 | 4 | 1.03% | scalp +4.60 (trail exit), ride +47.72 |
| **Final: per-branch exits (scalp TP 1.0% / ride TP 4.8%)** | **+56.30** | **4** | **0.68%** | scalp +8.41, ride +47.89 |
| Robustness grid on final: stop B 1.2–2.5%, trail B none–2.0%, maxhold B 240–400, drop B 2.3–2.5%, dev B 3.0–3.2%, TP B 4.8–5.0%, size 99–100% | +56.30 (all) | 4 | 0.68% | plateau — no variant changes either trade |

Rejected directions, honestly: entering the capitulation earlier at
bar 1008 (same 60-bar drop, deviation only −2.4%) fills ~1.1% higher
and its rebound never reaches the +4.8% target from that cost — the
deviation gate is load-bearing. A global small take-profit clips the
ride at +1%; a tight (≤0.35%) trail on the ride dies on the climb's
−0.33% chop at bar 1066/1092. No third honest trade exists on this
tape after the ride exits (post-peak dips are ≤1% with ≤+0.3% forward
rebounds).

## Files

- `main.py` — the bot (this directory).
- `vendor/makefun_bars.py` — kit SDK shim, byte-identical to the kit's
  `sdk/makefun_bars.py` (sha256
  80452b00d2eb1e9c959e73d41cda6ca726cdbe7bfa4065d48278c22eccd276ff).
- `LICENSE` (MIT), `README.md`, `.makefun/owner.json` — same layout as
  the incumbent entry.
- Replay/sweep tooling: `~/workspace/makefun/btc-race-r2-replay/`.
- Incumbent entry untouched at `~/workspace/makefun/btc-race-20261115-entry/`.

## Revision (session 2)

One small hardening edit to `main.py` (also satisfies the kit's
`thinking_revised` requirement of a genuine second-session revision):

- **Non-positive close guard in `on_batch`:** a bar whose close is
  `<= 0` is now dropped before it can enter `closes`. Previously a
  single corrupt bar would poison every SMA/deviation computed from
  the history and turn a later drop check (`close / closes[...]`)
  into a division by zero, killing the bot for the rest of the race.
  No such guard existed; all other divisions were already protected
  (`held.cost`, the `WARM` warm-up gate), this was the one missing
  sanity check.

Re-verified with the exact-integer replay
(`python3 ~/workspace/makefun/btc-race-r2-replay/replay.py ~/workspace/makefun/btc-race-r2-entry --twice`):
net PnL **+56.296447 USDC**, **4 trades**, max drawdown **0.680%**,
fees 1.629577 USDC, 0 rejected orders, deterministic — identical to
the pre-edit result, including the same action-log sha256
(`32a273b861c96c22a41524e11a43412081192010d0688ae8aaa975c1eafda0ec`),
so the guard is behavior-neutral on the dev tape (all dev closes are
positive) and only fires on corrupt input.
