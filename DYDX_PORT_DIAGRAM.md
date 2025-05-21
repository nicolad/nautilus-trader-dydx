# dYdX Port Architecture Diagram

Below is a conceptual diagram showing how the dYdX port for Nautilus Trader is intended to work. It illustrates the three main phases from the implementation plan and the flow between Python and Rust components.

```mermaid
flowchart TD
    subgraph Phase1["Phase 1 – Typed Transport Clients"]
        http_client["Rust HTTP Client"]
        ws_client["Rust WebSocket Client"]
        grpc_client["Rust gRPC Client"]
    end

    subgraph Phase2["Phase 2 – Python Bridge"]
        py_bridge["pyo3 Bridge"]
        md_client["MarketDataClient (Rust)"]
    end

    subgraph Phase3["Phase 3 – Full Adapter in Rust"]
        instr_provider["InstrumentProvider"]
        data_client["DataClient"]
        exec_client["ExecutionClient"]
    end

    PythonApp[("Nautilus Trader (Python)")]
    Exchange[("dYdX Exchange")]

    PythonApp --> py_bridge
    py_bridge --> http_client
    py_bridge --> ws_client
    py_bridge --> grpc_client
    http_client --> Exchange
    ws_client --> Exchange
    grpc_client --> Exchange

    py_bridge --> md_client
    md_client --> Exchange

    instr_provider -.-> PythonApp
    data_client -.-> PythonApp
    exec_client -.-> PythonApp
    instr_provider --> Exchange
    data_client --> Exchange
    exec_client --> Exchange
```

This diagram summarizes the planned progression:

1. **Phase 1** introduces typed transport clients in Rust for HTTP, WebSocket, and gRPC communication with dYdX.
2. **Phase 2** exposes these clients to Python via `pyo3`, routing market data through the Rust implementation.
3. **Phase 3** completes the port by implementing all adapter components in Rust, enabling Nautilus Trader to interact with dYdX end-to-end without Python in the critical path.
