
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
##### 1. File-/Module-level map (flowchart TD)

```mermaid
flowchart TD
    subgraph "nautilus_trader.adapters.dydx"
        A[__init__.py<br/>(re-exports)]
        B[config.py<br/>DydxConfig]
        C[common/*<br/>constants & helpers]
        D[endpoints/*<br/>(URL builders)]
        E[http/*<br/>HTTP client]
        F[websocket/*<br/>WS client]
        G[schemas/*<br/>Pydantic ↔ core]
        H[data.py<br/>Market-data adapters]
        I[execution.py<br/>Order logic]
        J[factories.py<br/>DydxAdapterFactory]
        K[providers.py<br/>DydxExchangeAdapter]
    end

    A --> K
    J --> K
    K --> B
    K --> C
    K --> D
    K --> E
    K --> F
    K --> G
    K --> H
    K --> I
    D --> E
    D --> F
```

##### 2. Runtime initialisation (sequence diagram)

```mermaid
sequenceDiagram
    participant Strategy
    participant Adapter as DydxExchangeAdapter
    participant HTTP as HttpClient
    participant WS as WebSocketClient
    participant DXREST as dYdX REST API
    participant DXWS   as dYdX WS API

    Strategy->>Adapter: connect()
    Adapter->>HTTP: start()
    Adapter->>WS: start()
    HTTP->>DXREST: GET /accounts
    DXREST-->>HTTP: balances
    HTTP-->>Adapter: balances normalised
    WS->>DXWS: SUBSCRIBE trades,orders
    DXWS-->>WS: stream frames
    WS-->>Adapter: envelopes (raw)
    Adapter-->>Strategy: domain events via bus
```

##### 3. Live data / order-flow (left-to-right flowchart)

```mermaid
flowchart LR
    subgraph "dYdX"
        A1[WebSocket stream] -->|JSON frames| B1[Deserializer]
        A2[REST endpoint] ---.
    end

    B1 --> C[Normalizer<br/>(schemas/*)]
    A2 -.-> C

    C --> D[MessageBus]
    D --> E[Strategy]
    E --> F[OrderIntent]
    F --> G[OrderRouter<br/>(execution.py)]
    G --> H[HttpClient] & I[WebSocketClient]
    H -->|REST: place order| A2
    I -->|WS: private| A1
    D <-- G
```
