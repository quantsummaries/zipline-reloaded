# Zipline Reloaded Learning Roadmap

> **Version**: written against branch `yzeng` at HEAD `d2242707` (2026-10-08).

Entry-points and recommended reading order for learning this codebase.
All paths below are relative to the repo root `zipline-reloaded/`.

## Table of Contents

- [1. Run the minimal example first](#1-run-the-minimal-example-first)
- [2. Understand the core concepts](#2-understand-the-core-concepts)
- [3. Inspect backtest results without ingesting a bundle](#3-inspect-backtest-results-without-ingesting-a-bundle)
- [4. Escalate through the examples (one new concept per file)](#4-escalate-through-the-examples-one-new-concept-per-file)
- [5. Prerequisite before running anything live](#5-prerequisite-before-running-anything-live)
- [Suggested order](#suggested-order)

## 1. Run the minimal example first

- [ ] `src/zipline/examples/buyapple.py` — 50-line "hello world". Demonstrates
      `initialize` / `handle_data` / `context` / `data` / `order` / commission &
      slippage setup / `analyze` plotting.
- [ ] `src/zipline/examples/buyapple.ipynb` — same example as a notebook, uses
      the `%%zipline` cell magic.

CLI form:

```bash
zipline run -f src/zipline/examples/buyapple.py \
  --start 2014-01-01 --end 2014-11-01 \
  -o out.pickle --bundle <your-bundle>
```

[⬆ 回到目录](#table-of-contents)

## 2. Understand the core concepts

- [ ] `docs/source/beginner-tutorial.rst` — written tutorial: what `context` is,
      what `data` contains per bar, how the event stream drives the backtest.
- [ ] `docs/notebooks/tutorial.ipynb` — interactive version of the same tutorial
      (214 KB; recommended to open and run cell-by-cell).
- [ ] Quickstart: https://github.com/quantsummaries/zipline-reloaded
- [ ] Getting started guide: https://zipline.ml4trading.io/beginner-tutorial

[⬆ 回到目录](#table-of-contents)

## 3. Inspect backtest results without ingesting a bundle

Old `quantopian-quandl` bundle is retired; until a local bundle is ingested
(see `docs/source/bundles.rst`), study pre-computed results:

- [ ] `docs/notebooks/buyapple_out.pickle` (276 KB) — full result DataFrame of
      buyapple backtest.
- [ ] `docs/notebooks/perf_dma.pickle` (328 KB) — dual-moving-average results.

Key columns to learn: `portfolio_value`, `positions`, `orders`,
`transactions`, `returns`.

[⬆ 回到目录](#table-of-contents)

## 4. Escalate through the examples (one new concept per file)

All in `src/zipline/examples/`:

- [ ] `buy_and_hold.py` — stripped-down version, no commission/slippage.
- [ ] `dual_moving_average.py` — SMA golden/death cross; introduces
      `zipline.transforms` / `schedule_function`.
- [ ] `dual_ema_talib.py` — EMA via TA-Lib.
- [ ] `momentum_pipeline.py` — Pipeline API (factor-based screening, zipline's
      biggest subsystem).
- [ ] `olmar.py` — online mean-reversion strategy; more advanced.

Reference: https://github.com/stefan-jansen/zipline-reloaded/tree/main/src/zipline/examples

[⬆ 回到目录](#table-of-contents)

## 5. Prerequisite before running anything live

- [ ] Ingest a data bundle — read `docs/source/bundles.rst` and set up a
      supported bundle (e.g. Quandl/Alpha Vantage/custom). The bundled
      `quantopian-quandl` source is no longer available upstream.

[⬆ 回到目录](#table-of-contents)

## Suggested order

1. Open `docs/notebooks/tutorial.ipynb` and read through.
2. Re-run `buyapple.py` end-to-end.
3. Then read `dual_moving_average.py` and `momentum_pipeline.py`.

[⬆ 回到目录](#table-of-contents)
