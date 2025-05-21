# OKX Port Documentation

This document outlines a proposed plan to port Nautilus Trader to support the OKX exchange. The steps mirror the approach for the existing dYdX port (see PR #1951 in the main repository) and expand with additional details.

## Overview
- **Objective**: Provide a full-featured adapter for the OKX exchange using the same architecture as the dYdX port.
- **Approach**: Implement typed transport clients in Rust, expose them through `pyo3` to Python, and gradually replace the current Python OKX adapter.

## Step-by-Step Process

### 1. Gather Requirements
1. Review the existing dYdX adapter implementation and the architecture defined in [PR #1951](https://github.com/nautechsystems/nautilus_trader/pull/1951).
2. Document OKX-specific API endpoints, authentication methods, and WebSocket channels.
3. Identify any differences from dYdX in order types, rate limits, and market data feeds.

### 2. Set Up Rust Crates
1. Create new Rust crates for `okx_http`, `okx_ws`, and `okx_grpc` based on the dYdX transport clients.
   - Each crate should expose a lightweight asynchronous API built on `reqwest` and `tokio`.
   - Include reusable error types so failures are handled consistently across the adapter.
2. Implement strongly typed request/response structs matching the OKX API.
   - Model WebSocket channels as enum variants with precise payload structs.
   - Provide conversion helpers to map between OKX order formats and the internal Nautilus Trader types.
3. Provide exhaustive unit tests and benchmarks for each client.
   - Use mocked servers to ensure deterministic test behaviour.
   - Benchmark the core serialization and network routines to catch regressions early.

### 3. Expose Rust APIs to Python
1. Use `pyo3` to generate Python bindings for the Rust transport clients.
2. Ensure the build system compiles these bindings as part of the Python package.
3. Validate the bindings with example scripts that exercise REST and WebSocket calls.

### 3a. Rust Port Guidelines
1. Organize each crate with a clear module hierarchy separating models, clients, and utilities.
2. Favor asynchronous tasks to maximize throughput when streaming market data or submitting orders.
3. Where possible, mirror the method names and semantics of the existing Python adapter so the higher-level logic remains unchanged.
4. Document each public function with examples to guide future contributors.

### 4. Implement Adapter Components in Rust
1. Recreate `InstrumentProvider`, `MarketDataClient`, `DataClient`, and `ExecutionClient` in Rust, mirroring the dYdX port structure.
2. Integrate the typed transport clients from step 2.
3. Provide configuration management for API keys, base URLs, and timeouts.

### 5. Bridge to Python
1. Replace the current Python OKX adapter methods with calls to the Rust implementations via the `pyo3` bindings.
2. Maintain compatibility with existing Nautilus Trader interfaces to minimize breaking changes.
3. Add integration tests ensuring orders and market data flow correctly through the new adapter.

### 6. Update Examples and Documentation
1. Provide Python examples demonstrating how to configure and use the OKX adapter.
2. Document environment variables or configuration files required for credentials.
3. Outline troubleshooting steps for common issues (connection drops, order rejections, etc.).

### 7. Final Testing and Deployment
1. Run soak tests to monitor stability under real market conditions.
2. Collect latency metrics and compare them with the existing Python adapter.
3. Publish the adapter on PyPI (or the relevant package index) and update Nautilus Trader docs to reflect OKX support.

## Notes
- The full port requires careful review of OKX's API rate limits and order handling specifics.
- Following the pattern from the dYdX port ensures consistency across exchanges and simplifies future maintenance.
- Additional pull requests may refine logging, metrics, or optional features such as advanced order types.

