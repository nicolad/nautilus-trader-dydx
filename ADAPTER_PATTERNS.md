# Adapter Patterns Extracted from Repo

The repository mentions adapter-related patterns primarily in its planning and TODO documents. Key references include:

- **Keeping the Python adapter** while implementing typed Rust clients for HTTP, WebSocket, and gRPC. This is part of Phase 1 in the implementation plan.
- **Routing the MarketDataClient** through the Rust implementation so the current Python adapter transparently calls Rust code.
- **Full Adapter in Rust**, re-implementing InstrumentProvider, DataClient, and ExecutionClient entirely in Rust, and refactoring order submission methods.
- TODO instructions highlighting adapter components: Instrument Provider, Data Client, Market Data Client, Execution Client, and Configuration.
- Porting steps referencing reuse of typed transport clients, exposing the Rust API to Python via pyo3, and implementing tests mirroring Python adapter behavior.

The documentation does not mention Binance or Bybit specifically, but the repository describes a phased approach to porting adapter functionality from Python to Rust.

