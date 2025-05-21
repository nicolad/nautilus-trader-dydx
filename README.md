# nautilus-trader-dydx

This repository provides code and documentation for integrating dYdX with Nautilus Trader.

## Implementation Plan

Below is a high-level TODO list for the planned work:

- **Phase 1 – Core Typed Transport Clients**
  - Implement strongly typed Rust clients for HTTP (REST), WebSocket, and gRPC.
  - Add exhaustive tests and benchmarks.
  - Keep the existing Python adapter in place.
- **Phase 2 – Python Bridge & Market-Data Path**
  - Expose the Rust clients to Python using a lightweight `pyo3` layer.
  - Route the `MarketDataClient` so the current Python adapter transparently calls the Rust implementation.
  - Users should see lower latency with no application level changes.
- **Phase 3 – Full Adapter in Rust**
  - Re-implement `InstrumentProvider`, `DataClient`, and `ExecutionClient` entirely in Rust.
  - Refactor order submission with dedicated methods for short-term, long-term, and conditional orders.
  - Run soak and latency tests and publish example strategies and docs.
