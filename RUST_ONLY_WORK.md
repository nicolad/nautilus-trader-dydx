# Rust-only Work for dYdX Port

The following table and notes outline the Rust-centric tasks required to complete the
`nautilus-trader-dydx` port. These items are derived from the project's implementation plan
and provide a more detailed breakdown of the work ahead.

### "Rust-only" deliverables

| Layer | Concrete Rust deliverables | Notes / acceptance tests |
| --- | --- | --- |
| **1. Transport clients (Phase 1)** | *HTTP (REST) client*<br>• Hand-rolled `Reqwest` / `hyper` wrapper with HMAC-SHA256 signing helper<br>• Strongly-typed request / response structs for every dYdX v4 REST route (markets, orderbook snapshots, fills, funding, etc.)<br><br>*WebSocket client*<br>• `tokio-tungstenite` stream that yields an enum `MarketStreamMsg` covering order-book deltas, trades, tickers, heartbeats<br>• Automatic ping/pong/ reconnect with exponential back-off<br><br>*gRPC client*<br>• `tonic-build` code-gen from official proto files<br>• Wrapper that maps proto types ⇄ idiomatic Rust domain types (`OrderId`, `FeeRate`, …) | Criterion benches for each client (latency & throughput).<br>Property tests (`proptest`) for JSON <-> struct round-trips. |
| **2. Error & retry framework** | • Unified `enum DydxTransportError` implementing `thiserror::Error`<br>• A `RetryPolicy` trait with concrete `ExponentialBackoff` impl reused by all three clients | Must bubble up *only* typed errors to callers; no `anyhow` leaks. |
| **3. `dydx` crate API surface** | • Public façade module exporting<br>`HttpClient`, `WsClient`, `GrpcClient` + `DydxConfig` builder<br>• Feature flags: `rustls`/`native-tls`, `async-std` runtime | Zero `unsafe` in safe API. |
| **4. Python bridge (Phase 2)** | • `pyo3` + `maturin` bindings that expose:<br>`PyHttpClient`, `PyWsClient`, `PyMarketDataClient`<br>• Internal async executor management (either reuse Nautilus’ runtime via `tokio::runtime::Handle::current()` or spin a dedicated multi-thread) | Round-trip test from Python: subscribe to BTC-USD order-book and assert first delta < 200 ms from handshake. |
| **5. Instrument & metadata layer** | • `InstrumentProvider` in Rust: maps dYdX “markets” JSON → `Instrument` struct (symbol, base/quote, tick size, lot size, …).<br>• In-memory LRU cache + TTL invalidation. | Unit tests with a fixed JSON fixture for every market type. |
| **6. Market-data path** | • `DataClient` that stitches `WsClient` streaming + snapshot sync from `HttpClient` into a *single* coherent `BookEvent` feed (price-level upserts, checksums). | Fuzz with random insert/delete seeds; verify order-book checksum equals exchange checksum every N packets. |
| **7. Execution path** | • `ExecutionClient` (pure Rust, using gRPC)<br>  – Dedicated high-level methods:<br>  `send_short_term`, `send_long_term`, `send_conditional` (see *ORDER_REFACTOR_TODO*)<br>  – In-flight order tracker with  ✕<sup>timeouts</sup> &  ✅<sup>fills</sup><br>• Post-trade fee lookup (`GrpcClient::query_fee_rates`) | Integration test: place IOC order on test-net, assert ACK, FILL or CANCEL in < 1 s. |
| **8. Adapter glue (Phase 3)** | • Replace Python adapter plumbing with:<br>`RustInstrumentProvider`, `RustDataClient`, `RustExecutionClient` registered via Nautilus “service locator”.<br>• Conditional `#[cfg(feature = "full_rust")]` to compile old path out. | End-to-end soak test: 24 h test-net run with synthetic strategy; memory leak < 1 MB/h growth. |
| **9. Tooling & CI** | • `cargo xtask` for code-gen, proto re-gen, lint, bench<br>• GitHub Actions matrix: Linux/macOS, nightly/stable, `--release` benches | Benches must upload HTML flamegraphs to Workflow artifacts. |
| **10. Docs & examples** | • `docs/` mdbook chapter *“Building high-performance crypto adapters in Rust”*<br>• Example Jupyter notebook (via the pyo3 wheel) streaming live order book into pandas | Notebook must run under GitHub Codespaces. |

---

### Recommended development sequencing

1. **Sprint 1 (1–2 weeks)** – bootstrap `dydx` crate, implement REST `Time` + `Markets` endpoints, finish error design.
2. **Sprint 2** – WebSocket delta stream & order-book builder; release `v0.1.0-alpha`.
3. **Sprint 3** – gRPC order placement + fee lookup; load tests.
4. **Sprint 4** – pyo3 bridge, wire into existing Python `MarketDataClient`; latency comparison benchmark.
5. **Sprint 5** – port Instrument / Data / Execution clients; remove Python path under `full_rust` feature.
6. **Sprint 6** – order-submission refactor, soak tests, publish examples & docs; tag `v1.0.0`.

### Key design principles

* **Zero copy, zero allocation** where possible (e.g. borrow `&[u8]` WebSocket frames, parse with `serde_json::from_slice` into `Cow<'_, str>`).
* **Tokio everywhere** – keeps a single runtime for REST, WS and gRPC.
* **No global singletons** – pass `Arc<…>` configs; eases multi-exchange use.
* **Typed guarantees at compile-time** – separate new-types for `UsdPrice`, `BaseQty`, `QuoteQty` to avoid unit mix-ups.
* **Feature-flagged crates** so Nautilus users can opt-in only to integrations they need.

These items outline the Rust-focused path to complete the dYdX integration.
