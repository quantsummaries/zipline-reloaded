# Zipline Function Dependencies

> **Version**: written against branch `yzeng` at HEAD `d2242707` (2026-10-08).

This document maps in-package function calls in `src/zipline/`. It is
intended as a navigation aid for reviewing implementation changes: a change
to a lower-level function may affect every caller listed beneath it.

Zipline is large (176 `.py` files across 9 subpackages). The first pass
below covers **top-level modules only** (`api.py`, `algorithm.py`,
`protocol.py`, `extensions.py`, `errors.py`, `currency.py`, `country.py`,
`__main__.py`). The function graph is intentionally thin because most
top-level modules export classes, constants, or re-exports rather than
top-level functions. A subpackage-level dependency tree follows, and
per-subpackage function graphs are listed as future work.

Dependencies below are direct calls between zipline functions. Calls into
NumPy, pandas, SciPy, pytz, and **empyrical-reloaded** are not listed. The
graph follows the checked-in source on branch `yzeng` and may need updating
when functions are added or refactored.

## Table of Contents
- [Top-level function dependency layers](#top-level-function-dependency-layers)
  - [Level 0 — foundational functions](#level-0--foundational-functions)
  - [Level 1 — CLI entry point](#level-1--cli-entry-point)
  - [Modules with no top-level function defs](#modules-with-no-top-level-function-defs)
- [Reverse index (top-level)](#reverse-index-top-level)
- [Subpackage dependency overview](#subpackage-dependency-overview)
  - [Subpackage size map](#subpackage-size-map)
- [Cross-repo edges (informational)](#cross-repo-edges-informational)
- [Key observations](#key-observations)
- [Future work](#future-work)

## Top-level function dependency layers
Levels indicate the direction of dependency: higher-level functions rely
on lower-level functions.

### Level 0 — foundational functions

These functions do not call another zipline top-level function directly.

1. `zipline.extensions`: `clear`, `create_registry`, `get_registry`,
   `load`, `parse_extension_arg`, `register`, `unregister`,
   `update_namespace`.
2. `zipline.country`: `code`.
3. `zipline.__main__` (CLI subcommand handlers): `bundles`, `clean`,
   `extract_option_object`, `ingest`, `ipython_only`, `run`,
   `zipline_magic`.

### Level 1 — CLI entry point

1. `zipline.__main__.main` → `zipline.extensions.create_args`.

`main` is the single top-level caller; it dispatches to the Level-0 CLI
subcommands above via click / argparse.

### Modules with no top-level function defs

The following modules are mostly classes, constants, or re-exports and
therefore contribute no function-level edges:

- `api.py` — re-exports public algorithm-decorator API
  (`order`, `record`, `schedule_function`, etc.).
- `algorithm.py` (84 KB) — the `TradingAlgorithm` class and its methods.
  Top-level helpers are thin.
- `protocol.py` — re-exports `BarData`, `InnerPosition` from `_protocol`.
- `errors.py` (24 KB) — exception classes; uses
  `utils.memoize.lazyval` for lazy attributes.
- `currency.py` — currency constants.

[⬆ 回到目录](#table-of-contents)

## Reverse index (top-level)
- `create_args` ← `__main__.main`.

[⬆ 回到目录](#table-of-contents)

## Subpackage dependency overview
Zipline's real architecture lives in its subpackages. The tree below is a
rough import-dependency map: arrows point "imported by".

```text
zipline.assets            Asset / Equity / Future / SID model
   ▲
   │  imported by:
   │
zipline.protocol          BarData, Position, Transaction
   ▲
   │
zipline.utils              shared plumbing (34 .py files)
   ├── calendar_utils, events, date_utils, math_utils,
   ├── input_validation, preprocess, cache, memoize,
   ├── cli (Date / Timestamp), run_algo, compat,
   ├── security_list, numpy_utils
   ▲
   │  used by everything below
   │
zipline.finance            order execution, slippage, commission,
   ├── blotter, controls, execution, slippage,
   ├── cancel_policy, asset_restrictions, metrics
   ▲
   │
zipline.gens               simulation drivers
   ├── tradesimulation (AlgorithmSimulator),
   ├── sim_engine (MinuteSimulationClock)
   ▲
   │
zipline.data               data layer (25 .py files)
   ├── bundles, data_portal, history_loader,
   ├── resampler, us_equity_pricing, minute_bars,
   └── etc.
   ▲
   │
zipline.pipeline           factor / classifier / factor pipeline (44 .py files)
   ├── engine, domain, data, factors, classifiers,
   ├── filters, term, graph, loaders
   ▲
   │
zipline.algorithm          TradingAlgorithm class (top-level 84 KB)
   ▲
   │  imports: assets, finance.blotter, finance.controls,
   │          finance.metrics, gens.tradesimulation,
   │          gens.sim_engine, pipeline, pipeline.engine,
   │          sources.*, utils.*
   │
zipline.__main__           CLI entry point
   └── imports: data.bundles, utils.calendar_utils,
                utils.cli, utils.run_algo, extensions

zipline.sources            benchmark / CSV data sources
zipline.testing            pytest fixtures
zipline.resources           packaged data
```

### Subpackage size map

| Subpackage | `.py` files | Role |
|---|---:|---|
| `pipeline` | 44 | Factor/classifier/filter pipeline engine |
| `utils` | 34 | Shared plumbing |
| `data` | 25 | Data portal, bundles, bar readers |
| `finance` | 21 | Blotter, execution, slippage, commission, metrics |
| `assets` | 9 | Asset / SID model |
| `testing` | 9 | Fixtures |
| `gens` | 4 | Simulation drivers |
| `sources` | 4 | Benchmark / CSV sources |
| `lib` | 5 | Cython helpers |

[⬆ 回到目录](#table-of-contents)

## Cross-repo edges (informational)
- `zipline.finance.metrics` calls into **empyrical-reloaded** for
  post-backtest statistics (sharpe, drawdown, beta, etc.).
- `pyfolio-reloaded.utils.extract_rets_pos_txn_from_zipline` consumes the
  DataFrame produced by a zipline backtest.
- `alphalens-reloaded` can ingest zipline pipeline outputs.

[⬆ 回到目录](#table-of-contents)

## Key observations
- **`utils/` is the foundation.** Every other subpackage imports from it.
  A change in `utils.calendar_utils`, `utils.input_validation`, or
  `utils.preprocess` has the widest blast radius.
- **`algorithm.py` is the composition root.** It imports assets, finance,
  gens, pipeline, sources, and utils directly. Understanding
  `TradingAlgorithm.__init__` and `handle_data` / `before_trading_start`
  is the fastest way to map the rest of the system.
- **`finance/` and `data/` are largely independent** — finance models order
  execution; data models bar retrieval; both are bridged through
  `algorithm.py`.
- **`pipeline/` is the largest subpackage** and is largely self-contained;
  it plugs into the algorithm via `pipeline.engine` and
  `pipeline.domain`.
- **Top-level function graphs are not informative** — most top-level work
  is done inside class methods. Future passes should run the AST walker
  per subpackage (`finance/`, `data/`, `pipeline/`, `utils/`) rather than
  at the top level.

[⬆ 回到目录](#table-of-contents)

## Future work
- [ ] Run the AST call-graph builder per subpackage and merge results
      into this file.
- [ ] Document class-level call relationships inside
      `algorithm.TradingAlgorithm` (most user-visible behavior lives
      there).
- [ ] Add a section for `finance.metrics` — the point where zipline
      crosses into empyrical.

[⬆ 回到目录](#table-of-contents)