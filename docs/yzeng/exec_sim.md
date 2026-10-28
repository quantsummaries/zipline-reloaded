# Transaction Cost Models in Zipline

Zipline splits transaction costs into **slippage** (`src/zipline/finance/slippage.py`)
and **commissions** (`src/zipline/finance/commission.py`). Configure them with
`set_slippage()` and `set_commission()`.

## Slippage models

- **`VolumeShareSlippage`** (default for equities)
  - Fill price is `price * (1 ± price_impact * volume_share**2)`, with `price_impact=0.1`.
  - An order can take at most 2.5% of the bar's volume (`volume_limit`). Any remainder stays open and fills in later bars.
- **`FixedSlippage(spread)`**: fills at `close ± spread/2`.
- **`FixedBasisPointsSlippage`**: fixed 5 bps by default, with fills limited to 10% of volume.
- **`VolatilityVolumeShare`**: market-impact model for futures. It scales with volatility and volume share, using per-root-symbol eta values.
- **`NoSlippage`**: fills at the bar's price.
- Futures default to a 5% volume limit.

## Commission models

- **`PerShare`** (default for equities): $0.001 per share, $0 minimum per trade.
- **`PerContract`** (default for futures): $0.85 per contract, plus an optional exchange fee.
- **`PerDollar`**: 0.15% of the dollar value traded.
- **`PerTrade`** / **`PerFutureTrade`**: flat fee per trade.
- **`NoCommission`**: no commission.

## Assumptions

- Orders fill at the next bar's close, with no look-ahead.
- Volume is the only liquidity constraint. There is no order book or bid/ask spread unless you use `FixedSlippage`.
- Partial fills carry across bars.
- There are no borrow costs or financing costs for shorts or leverage.
- Custom models can be written by subclassing `SlippageModel` or `CommissionModel`.
