# Expanded Notes on Order Submission Refactor and Project Context

This summary consolidates the documentation found in the repository and expands upon the planned refactoring work for Nautilus Trader's dYdX integration.

## Order Submission Refactor
The file `ORDER_REFACTOR_TODO.md` outlines a plan to split the current monolithic `submit_order` method into specialized functions:

- `submit_short_term_order(...)` for low latency intraday trading.
- `submit_long_term_order(...)` for swing or position strategies, including checks for funding rates or rollover.
- `submit_conditional_order(...)` triggered by price thresholds or other signals.

The refactor will extract shared logic (serialization, validation, error handling) into helper functions and update unit tests for each method. Once complete, the old `submit_order` method will be deprecated.

## Implementation Roadmap
According to `README.md`, the integration with dYdX is progressing through three phases:

1. **Core Typed Transport Clients** – Rust implementations for HTTP, WebSocket, and gRPC with exhaustive tests.
2. **Python Bridge & Market-Data Path** – expose the Rust clients via `pyo3` and route the `MarketDataClient` through them.
3. **Full Adapter in Rust** – re-implement `InstrumentProvider`, `DataClient`, and `ExecutionClient`, including the order submission refactor described above.

## References and Links
- The repository links back to the main Nautilus Trader project on GitHub: <https://github.com/nautechsystems/nautilus_trader>.
- PR [`#2610`](https://github.com/nautechsystems/nautilus_trader/pull/2610) introduces an OKX port serving as a reference architecture.
- PR [`#1951`](https://github.com/nautechsystems/nautilus_trader/pull/1951) describes earlier porting work used as a template here.

These links provide context for how the dYdX port and the order submission refactor fit into the broader Nautilus Trader development roadmap.
