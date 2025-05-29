
<!-- 1 · Component & Dependency Architecture -->
```mermaid
graph TD
    subgraph Trading Node
        TN[TradingNode]
    end

    subgraph Factories
        DCF[DYDXLiveDataClientFactory]
        ECF[DYDXLiveExecClientFactory]
    end

    subgraph High-level Clients
        DC[DYDXDataClient]
        EC[DYDXExecutionClient]
    end

    subgraph Low-level Connectivity
        WS[DYDXWebSocketClient]
        HTTP[DYDXHttpClient]
        GRPC[DYDXAccountGRPCAPI]
    end

    subgraph Utilities
        IP[DYDXInstrumentProvider]
    end

    %% edges
    TN -- uses --> DCF
    TN -- uses --> ECF
    DCF -- builds --> DC
    ECF -- builds --> EC
    DC -- WebSocket --> WS
    DC -- HTTP --> HTTP
    EC -- HTTP --> HTTP
    EC -- gRPC --> GRPC
    IP -- instruments --> TN
```

<!-- 2 · Order Classification & Life-cycle -->
```mermaid
flowchart TD
    A[New Order] -->|default| B[Short-term<br>(in-block)]
    A -->|DYDXOrderTags<br>is_short_term_order = false| C[Long-term]

    %% short-term branch
    B -->|optional: num_blocks_open| B1[Expires after N blocks]
    B --> B2[Committed only<br>fill & expiry]

    %% long-term branch
    C --> D{Conditional?}
    D -->|Yes&nbsp;(STOP_*)| C1[Long-term<br>Conditional]
    D -->|No| C2[Long-term<br>Regular]

    %% terminal
    B1 & B2 & C1 & C2 --> E[Filled / Cancelled / Expired]
```

<!-- 3 · End-to-End Order Submission Sequence -->
```mermaid
sequenceDiagram
    participant S as Strategy / Algo
    participant N as TradingNode
    participant EC as DYDXExecutionClient
    participant HTTP as DYDXHttpClient
    participant CHAIN as dYdX Chain

    S->>N: create LimitOrder(...)
    N->>EC: submit(order)
    EC->>HTTP: POST /api/v4/orders
    HTTP->>CHAIN: broadcast tx
    CHAIN-->>HTTP: tx receipt
    HTTP-->>EC: order accepted
    EC-->>N: state = WORKING
    CHAIN-->>EC: fill event (WebSocket)
    EC-->>N: state = FILLED
```


```mermaid
classDiagram
    %% Core node and its configuration
    class TradingNode {
        +run()
        +dispose()
        +add_data_client_factory()
        +add_exec_client_factory()
    }
    class TradingNodeConfig
    TradingNode --> TradingNodeConfig

    class LoggingConfig
    TradingNodeConfig --> LoggingConfig

    class LiveExecEngineConfig
    TradingNodeConfig --> LiveExecEngineConfig

    class CacheConfig
    TradingNodeConfig --> CacheConfig

    %% Connectivity: data + execution
    class DYDXDataClientConfig
    class DYDXExecClientConfig
    TradingNodeConfig --> DYDXDataClientConfig
    TradingNodeConfig --> DYDXExecClientConfig

    %% Factories registered on the node
    class DYDXLiveDataClientFactory
    class DYDXLiveExecClientFactory
    TradingNode --> DYDXLiveDataClientFactory
    TradingNode --> DYDXLiveExecClientFactory

    %% Trader and strategy stack
    class Trader {
        +add_strategy()
    }
    TradingNode --> Trader

    class VolatilityMarketMaker
    Trader --> VolatilityMarketMaker

    class VolatilityMarketMakerConfig
    VolatilityMarketMaker --> VolatilityMarketMakerConfig

    %% Strategy–specific identifiers
    class InstrumentId
    class BarType
    VolatilityMarketMakerConfig --> InstrumentId
    VolatilityMarketMakerConfig --> BarType

```
nautilus_trader/
└── adapters/
    └── dydx/
        ├── __init__.py          ◀─ ENTRY-POINT (re-exports everything below)
        ├── factories.py         ◀─ DYDXLiveDataClientFactory / DYDXLiveExecClientFactory
        ├── data.py              ◀─ DYDXDataClient
        ├── execution.py         ◀─ DYDXExecutionClient
        ├── websocket/client.py  ◀─ DYDXWebsocketClient
        ├── http/client.py       ◀─ DYDXHttpClient
        ├── grpc/account.py      ◀─ DYDXAccountGRPCAPI
        ├── providers.py         ◀─ DYDXInstrumentProvider
        └── … (configs, enums, etc.)
