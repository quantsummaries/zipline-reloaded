# Zipline Reloaded — Maintenance TODO

This file tracks **work to do on the codebase itself**: bugs, missing tests,
stale docs, upstream features to port, performance fixes.

The **learning roadmap** (for someone coming to the codebase fresh) lives in
[`CODE_LEARN.md`](CODE_LEARN.md). Project-wide conventions:
[`../PROJECT-CONVENTIONS.md`](../PROJECT-CONVENTIONS.md).

## Bugs / behavior fixes

- [ ]

## Test coverage gaps

- [ ]

## Documentation

- [x] read DEPENDENCIES.md
- [ ] read CODE_OVERVIEW.md
- [ ] read CODE_LEARN.md
- [ ] use pyreverse to generate dependency diagrams
- [ ] use pdoc to generate html documentation
- [ ] Understanding TradingAlgorithm.__init__ and handle_data / before_trading_start is the fastest way to map the rest of the system.
- [ ] Run the AST call-graph builder per subpackage and merge results into this file
- [ ] Document class-level call relationships inside algorithm.TradingAlgorithm (most user-visible behavior lives there)
