# TODO for Rust Adapter Port

This file tracks tasks required to implement a Rust-based adapter for NautilusTrader. See the [Developer Guide](https://nautilustrader.io/docs/latest/developer_guide/adapters/) for additional details on building an adapter.

## Adapter Components
- **Instrument Provider**: supply instrument definitions using Rust traits matching the Python `InstrumentProvider` behaviour.
- **Data Client**: manage custom data feeds not strictly related to market data.
- **Market Data Client**: provide market data such as order book deltas and instrument status.
- **Execution Client**: handle order submission, modification, cancellation, and report generation.
- **Configuration**: define settings required by the Rust clients (API keys, base URLs, etc.).

## dYdX API Notes
- The existing adapter already includes three transport clients:
  - **HTTP** for historical data and instrument metadata.
  - **WebSocket** for real-time market data streams.
  - **gRPC** for submitting orders and querying fee rates.
  These clients will be replaced with typed Rust equivalents during the port.

## Step-by-Step Porting Guide

1. **Set Up the Project**
   - Create a new Cargo workspace or `dydx` crate to host the Rust code.
   - Add dependencies such as `tokio`, `pyo3`, and serialization libraries.
   - Define a shared configuration module for API keys and base URLs.

2. **Implement Typed Transport Clients**
   - **HTTP Client**
     - Map REST endpoints for instrument metadata and historical data.
     - Implement typed request/response structs and authentication helpers.
     - Write unit tests covering error handling and edge cases.
   - **WebSocket Client**
     - Connect to the streaming endpoint and manage subscriptions.
     - Deserialize order book deltas and instrument status updates.
     - Provide a clean API for consumers to register callbacks.
   - **gRPC Client**
     - Compile the official proto files and generate Rust types.
     - Implement order submission and fee rate requests with retries.
     - Add tests to verify request formatting and response parsing.

3. **Expose Rust to Python**
   - Use `pyo3` (via `maturin` or `setuptools-rust`) to build Python bindings.
   - Wrap each client in a Python-friendly interface mirroring the existing adapter.
   - Validate the bindings with simple scripts that fetch data and submit orders.

4. **Reimplement Adapter Components in Rust**
   - Port the `InstrumentProvider`, `DataClient`, `MarketDataClient`, and `ExecutionClient` using Rust traits.
   - Ensure method names and behaviours match the Python versions for drop-in compatibility.
   - Integrate the transport clients from step 2 and the configuration module.

5. **Testing and Examples**
   - Mirror the Python adapter's behaviour with comprehensive unit and integration tests.
   - Provide example strategies that load instruments, stream market data, and place orders using the new Rust implementation.
   - Document environment variables and configuration files required for credentials.

Following these steps will gradually replace the current Python adapter with a fully typed Rust implementation while maintaining compatibility for existing Nautilus Trader users.

