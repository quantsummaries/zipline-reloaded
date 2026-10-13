# Zipline Documentation

> **Version**: written against branch `yzeng` at HEAD `d2242707` (2026-10-08).

## Table of Contents
- [Review Addendum](#review-addendum)
- [Overview](#overview)
- [Package dependencies](#package-dependencies)
- [Module guide](#module-guide)
- [Primary workflow](#primary-workflow)
- [Package dependency layering (original sketch)](#package-dependency-layering-original-sketch)


## Review Addendum
_Originally written at HEAD on 2026-10-09 (branch `yzeng`)._

### Scope reviewed

- `src/zipline/__init__.py`, `__main__.py`, `api.py`, `algorithm.py`,
  `protocol.py`, `extensions.py`, `errors.py`, `currency.py`,
  `country.py`, `dispatch.py`, `zipline_warnings.py`
- Subpackages: `assets/`, `data/`, `finance/`, `gens/`, `lib/`,
  `pipeline/`, `sources/`, `utils/`, `testing/`
- `tests/` (smoke read)
- Related repos: `empyrical-reloaded` (called by `finance.metrics`),
  `pyfolio-reloaded` (consumes backtest result DataFrame)

### Findings (ordered by severity)

1. **Medium (stale doc)**: `zipline/CLAUDE.md` still says
   "Current Branch: v3.1.1" while the working branch is `yzeng`. Any AI
   assistant reading CLAUDE.md gets the wrong baseline.
   - Ref: `CLAUDE.md`.
2. **Medium (package layering is circular in practice)**: the original
   review sketch classifies `utils` as Level 0, but `utils` actually
   imports back from `algorithm`, `api`, `data`, `errors`, `extensions`,
   `finance`, `pipeline`, `sources`, and `zipline_warnings`. Treat this as
   a coupled core cluster, not a strict DAG.
   - Ref: `src/zipline/utils/`.
3. **Low (scope of this document)**: function-level call graph in
   `DEPENDENCIES.md` is top-level only. Most user-visible behavior lives
   inside `TradingAlgorithm` class methods and subpackages; a per-subpackage
   AST pass is needed for a complete map.
   - Ref: `DEPENDENCIES.md`.
4. **Low (optional dep)**: `pandas-datareader` (used for Fama-French /
   FRED) is incompatible with Python >= 3.12; the corresponding data
   sources degrade gracefully but should be noted in user-facing docs.

### API / behavior changes captured

- Public algorithm API surface is re-exported from `api.py`: `order`,
  `order_target`, `order_value`, `order_percent`, `get_open_orders`,
  `set_slippage`, `set_commission`, `set_long_only`, `set_do_not_order_list`,
  `schedule_function`, `record`, `attach_pipeline`, `pipeline_output`,
  etc.
- `__main__` CLI subcommands: `ingest`, `bundles`, `clean`, `run` (plus
  IPython magic `zipline_magic`).
- Extensions are registered via `extensions.register(...)` /
  `extensions.load(...)`; user `~/.zipline/extension.py` is auto-loaded.

### Documentation updates completed

- `DEPENDENCIES.md` — top-level function graph + subpackage-level
  dependency tree.
- `CODE_LEARN.md` — learning roadmap (migrated from TODO.md 2026-10-09).
- `TODO.md` — maintenance task skeleton.

[⬆ 回到目录](#table-of-contents)

## Overview
Zipline is an event-driven backtesting engine for equities. A user writes
a `TradingAlgorithm` subclass (or a `initialize` / `handle_data` pair),
instructs it via the `api` decorators, and runs it against a data bundle.
At each simulation minute / day, the engine:

1. Pulls the next bar from the data portal.
2. Updates blotter state and open orders.
3. Applies slippage / commission models.
4. Invokes `handle_data` with a `BarData` snapshot.
5. Records portfolio state and metrics.

The engine is large (176 `.py` files) and organized around a small number
of key objects: `TradingAlgorithm`, `SimulationBlotter`, `BarData`,
`DataPortal`, `AlgorithmSimulator`, `MinuteSimulationClock`, and
`Pipeline` / `PipelineEngine`.

[⬆ 回到目录](#table-of-contents)

## Package dependencies
- Runtime core: `numpy`, `pandas`, `scipy`, `pytz`, `trading-calendars`,
  `requests`, `ta-lib` (optional), `pandas-datareader` (optional).
- Sibling reloaded libs: **empyrical-reloaded** is called by
  `finance.metrics` for post-backtest statistics; **pyfolio-reloaded**
  consumes the resulting backtest DataFrame downstream.
- Build / distribution: Cython for `lib/` helpers, `setuptools_scm`.

[⬆ 回到目录](#table-of-contents)

## Module guide
### `api`

Re-export of the public decorator / function surface users call inside
their algorithm scripts. Thin; do not put logic here.

### `algorithm` (84 KB)

`TradingAlgorithm` — the composition root. Owns:

- Initialization (`initialize`, `before_trading_start`).
- Bar dispatch (`handle_data`).
- Order placement (delegates to blotter).
- Pipeline attachment (`attach_pipeline`, `pipeline_output`).
- Calendar / event scheduling (`schedule_function`).

Most "zipline behavior" questions are answered by reading class methods
on `TradingAlgorithm`.

### `finance`

Order-execution layer:

- `blotter.SimulationBlotter` — order / position state machine.
- `execution.*` — order types (Market, Limit, Stop, StopLimit, etc.).
- `slippage.*` — slippage models (VolumeShare, FixedBasisPoints, etc.).
- `commission.*` — commission models (PerShare, PerDollar, PerTrade).
- `cancel_policy.*` — `NeverCancel`, `EODCancel`.
- `asset_restrictions.*` — restricted-security lists.
- `controls.*` — risk controls.
- `metrics.*` — post-backtest metric tracking; calls empyrical.

### `data`

Data layer (25 `.py` files):

- `bundles` — ingest / registry for market-data bundles (e.g. `quandl`,
  `quantopian-quandl`).
- `data_portal.DataPortal` — unified read API the algorithm sees.
- `history_loader` — lookback-window construction.
- `resampler` — minute-to-day / day-to-week aggregation.
- `us_equity_pricing`, `minute_bars`, etc. — on-disk storage readers.

### `gens`

Simulation drivers:

- `tradesimulation.AlgorithmSimulator` — main loop.
- `sim_engine.MinuteSimulationClock` — event clock.

### `pipeline` (44 `.py` files)

Factor / classifier / filter pipeline. Mostly self-contained; plugs into
the algorithm through `pipeline.engine.PipelineEngine` and
`pipeline.domain`. Read this only after the core backtest loop is clear.

### `assets`

`Asset`, `Equity`, `Future`, SID mapping, asset restrictions.

### `utils` (34 `.py` files)

Shared plumbing. Highest-blast-radius subpackage. Includes
`calendar_utils`, `events`, `date_utils`, `math_utils`,
`input_validation`, `preprocess`, `cache`, `memoize`, `cli`,
`run_algo`, `compat`, `security_list`, `numpy_utils`.

### `sources`

Benchmark source (`benchmark_source.BenchmarkSource`) and CSV requests
source (`requests_csv.PandasRequestsCSV`).

### `errors` (24 KB)

Exception hierarchy. Uses `utils.memoize.lazyval` for lazy attributes.

### `extensions`

Extension registry. `~/.zipline/extension.py` is auto-loaded on startup.

### `protocol`

Re-export of `BarData` and `InnerPosition` from `_protocol`.

### `__main__`

CLI entry point. Subcommands: `ingest`, `bundles`, `clean`, `run`; plus
the IPython `%%zipline` magic.

[⬆ 回到目录](#table-of-contents)

## Primary workflow
1. **Ingest a bundle**: `zipline ingest -b <bundle>` populates the data
   store under `~/.zipline/data/`.
2. **Write an algorithm**: a script with `initialize(context)` and
   `handle_data(context, data)`, using `api.*` decorators.
3. **Run**: `zipline run -f my_algo.py --start 2020-01-01 --end
   2020-12-31 -b <bundle> -o out.pickle`.
4. **Post-process**: load `out.pickle` in pyfolio to render tear sheets.

The runtime chain inside `AlgorithmSimulator`:

```text
MinuteSimulationClock
   → DataPortal.get_history / get_spot_value
   → BarData snapshot
   → TradingAlgorithm.handle_data
   → SimulationBlotter.place_orders
   → Slippage / Commission models
   → MetricsTracker (→ empyrical for end-of-run stats)
```

[⬆ 回到目录](#table-of-contents)

## Package dependency layering (original sketch)
The sketch below was written before the full import graph was audited;
treat levels as a reading order, not a strict DAG — `utils` actually
imports back into the core cluster (see Findings #2).

### Level 0 — no zipline dependencies

1. `zipline.country`
2. `zipline.currency`
3. `zipline.extensions`
4. `zipline.protocol`
5. `zipline.zipline_warnings`
6. `zipline.api`

### Level 1 — depends on Level 0

7. `zipline.errors` → `zipline.utils`
8. `zipline.assets` → `zipline.errors`, `zipline.utils`
9. `zipline.lib` → `zipline.errors`, `zipline.utils`
10. `zipline.sources` → `zipline.assets`, `zipline.errors`
11. `zipline.finance` → `zipline.assets`, `zipline.errors`, `zipline.gens`, `zipline.utils`
12. `zipline.gens` → `zipline.finance`, `zipline.utils`
13. `zipline.data` → `zipline.assets`, `zipline.errors`, `zipline.gens`, `zipline.lib`, `zipline.utils`

### Level 2 — depends on Level 0–1

14. `zipline.algorithm` → `zipline.assets`, `zipline.errors`, `zipline.finance`, `zipline.gens`, `zipline.pipeline`, `zipline.sources`, `zipline.utils`, `zipline.zipline_warnings`
15. `zipline.pipeline` → `zipline.assets`, `zipline.country`, `zipline.currency`, `zipline.data`, `zipline.errors`, `zipline.lib`, `zipline.utils`
16. `zipline.utils` → `zipline.algorithm`, `zipline.api`, `zipline.data`, `zipline.errors`, `zipline.extensions`, `zipline.finance`, `zipline.pipeline`, `zipline.sources`, `zipline.zipline_warnings`

### Level 3 — integration and test packages

17. `zipline.__main__` → `zipline.data`, `zipline.extensions`, `zipline.utils`
18. `zipline.examples` → `zipline.api`, `zipline.finance`, `zipline.pipeline`
19. `zipline.dispatch`
20. `zipline.test_algorithms` → `zipline.algorithm`, `zipline.api`, `zipline.errors`, `zipline.finance`
21. `zipline.testing` → `zipline.algorithm`, `zipline.assets`, `zipline.data`, `zipline.dispatch`, `zipline.finance`, `zipline.lib`, `zipline.pipeline`, `zipline.utils`
22. `zipline._version`

### Key observations

- `zipline.algorithm`, `zipline.assets`, `zipline.data`, `zipline.errors`,
  `zipline.finance`, `zipline.gens`, `zipline.lib`, `zipline.pipeline`,
  `zipline.sources`, and `zipline.utils` form a coupled core cluster with
  mutual imports; they should be reviewed as a group rather than strictly
  linearly.
- `zipline.country`, `zipline.currency`, `zipline.extensions`,
  `zipline.protocol`, and `zipline.zipline_warnings` are low-level packages
  with no internal zipline package dependencies.
- `zipline.__main__`, `zipline.examples`, `zipline.test_algorithms`, and
  `zipline.testing` are later-stage integration or validation layers and
  belong near the end of the review sequence.

[⬆ 回到目录](#table-of-contents)