# TODO for Rust Adapter Port

This file tracks tasks required to implement a Rust-based adapter for NautilusTrader. See the [Developer Guide](https://nautilustrader.io/docs/latest/developer_guide/adapters/) for additional details on building an adapter.

## Adapter Components
- **Instrument Provider**: supply instrument definitions using Rust traits matching the Python `InstrumentProvider` behaviour.
- **Data Client**: manage custom data feeds not strictly related to market data.
- **Market Data Client**: provide market data such as order book deltas and instrument status.
- **Execution Client**: handle order submission, modification, cancellation, and report generation.
- **Configuration**: define settings required by the Rust clients (API keys, base URLs, etc.).

## Porting Steps
1. Create Rust crates for the clients above, following existing Python interfaces.
2. Reuse strongly typed transport clients from Phase 1 for HTTP/WebSocket/gRPC.
3. Expose the Rust API to Python via `pyo3` to maintain compatibility.
4. Implement thorough tests to mirror behaviour of the Python adapter.

