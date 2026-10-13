# Zipline Data Dependencies

Zipline ships no market data. Backtests read from a **local, pre-ingested data bundle**,
so the core engine makes no network calls during a run. Network access happens only at
ingest time, and only for some bundles.

## 1. Price data: data bundles

- Bundles are registered in `src/zipline/data/bundles/` (`quandl.py`, `csvdir.py`, plus the machinery in `core.py`).
- `zipline ingest -b <bundle>` runs the bundle's ingest function and writes the data to local storage, under the zipline root (`~/.zipline` by default, or `ZIPLINE_ROOT`). Each ingestion is stored in a timestamped directory:
  - `daily_equities.bcolz`: daily OHLCV bars
  - `minute_equities.bcolz`: minute bars
  - `adjustments.sqlite`: splits, mergers and dividends
  - `assets-<version>.sqlite`: asset metadata (symbols, exchanges, dates)
- `zipline run` and `run_algorithm(bundle=...)` load the latest ingestion via `bundles.load()`. Old ingestions can be removed with `zipline clean`.

| Bundle | Source | Network? |
|---|---|---|
| `quandl` | Quandl WIKI prices, downloaded with `requests.get` (needs `QUANDL_API_KEY`) | Yes, at ingest only |
| `csvdir` | Your own CSV files (daily/minute) in a local directory | No |
| Custom bundle | Anything you write and register (files, database, API) | Depends on your code |

## 2. Benchmark returns

No benchmark is downloaded in this fork. `BenchmarkSpec.resolve` (`src/zipline/utils/run_algo.py`) uses, in order:

1. a `benchmark_returns` pandas Series passed in
2. `--benchmark-file`: a local CSV, read by `get_benchmark_returns_from_file` (`data/benchmarks.py`)
3. `--benchmark-sid` / `--benchmark-symbol`: an asset in the ingested bundle
4. `--no-benchmark`: a series of zero returns
5. otherwise warnings are logged, and the algorithm must call `set_benchmark()` in `initialize`, or the run fails with `NoBenchmark`

## 3. Trading calendars

Provided by the external `exchange_calendars` package. Calendars are generated from rules that ship with it, so no download is needed.

## 4. Other sources

- `src/zipline/sources/requests_csv.py` can fetch CSVs by URL for custom data sources. It is a network call, and only used if you use it.
- `run_algorithm(..., data=...)` accepts in-memory pandas data (used heavily in tests), with no bundle needed.
- Pipeline data (`EquityPricing`) comes from the same bundle readers. Extra datasets (for example fundamentals) must be loaded by a custom pipeline loader.

## Summary

| Need | Where it comes from | Network |
|---|---|---|
| Prices, volume | Ingested bundle | Only at ingest, for `quandl` or custom bundles |
| Splits/dividends, asset metadata | Ingested bundle | Same |
| Benchmark | Your Series, a local CSV, or a bundle asset | No |
| Calendars | `exchange_calendars` | No |

For a fully offline setup: use `csvdir` (or a custom bundle) plus `--benchmark-file` or `--no-benchmark`.
