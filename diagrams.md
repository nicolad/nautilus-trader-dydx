
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

```md
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
```


Below are the seven Mermaid diagrams wrapped in Markdown code-blocks, ready to paste straight into any renderer that supports Mermaid.

### 1 · Package / module overview

```mermaid
graph TD
    subgraph nautilus_trader.adapters
        A[dydx]
    end

    A --> B[dydx.config]
    A --> C[dydx.factories]
    A --> D[dydx.common.enums]
    A --> E[dydx.providers]
    A --> F[dydx.data]
    A --> G[dydx.execution]
```

### 2 · Config module

```mermaid
classDiagram
    class DydxAdapterConfig {
        +DydxRESTConfig rest
        +DydxWebSocketConfig ws
        +bool demo
        +str account_id
        +validate()
    }

    class DydxRESTConfig {
        +str api_key
        +str api_secret
        +str passphrase
        +str endpoint
    }

    class DydxWebSocketConfig {
        +str endpoint
        +list~str~ channels
        +int ping_interval
    }

    DydxAdapterConfig *-- DydxRESTConfig : embeds
    DydxAdapterConfig *-- DydxWebSocketConfig : embeds
```

### 3 · Factories module

```mermaid
classDiagram
    class DydxInstrumentFactory {
        +create_from_rest(json)
        +create_from_ws(json)
    }

    class DydxOrderFactory {
        +create_limit(side, price, size)
        +create_market(side, size)
        +create_cancel(order_id)
    }

    class DydxAccountFactory {
        +create_from_rest(json)
        +create_from_ws(json)
    }

    DydxInstrumentFactory <|-- BaseInstrumentFactory
    DydxOrderFactory <|-- BaseOrderFactory
    DydxAccountFactory <|-- BaseAccountFactory
```

### 4 · Enums module

```mermaid
classDiagram
    class DydxSide {
        <<enumeration>>
        +BUY
        +SELL
    }

    class DydxOrderType {
        <<enumeration>>
        +LIMIT
        +MARKET
        +STOP_LIMIT
        +STOP_MARKET
    }

    class DydxOrderStatus {
        <<enumeration>>
        +OPEN
        +FILLED
        +PARTIALLY_FILLED
        +CANCELED
        +REJECTED
    }
```

### 5 · Providers module

```mermaid
classDiagram
    class DydxRESTMarketDataProvider {
        +get_ticker(instrument)
        +get_order_book(instrument, depth)
        +get_trades(instrument, since)
    }

    class DydxWebSocketMarketDataProvider {
        +subscribe_ticker(instrument)
        +subscribe_order_book(instrument, depth)
        +subscribe_trades(instrument)
        +disconnect()
    }

    class MarketDataEvent {
        +datetime ts
        +str type
        +dict payload
    }

    DydxRESTMarketDataProvider --> MarketDataEvent : emits
    DydxWebSocketMarketDataProvider --> MarketDataEvent : emits
```

### 6 · Data module

```mermaid
classDiagram
    class DydxInstrument {
        +str symbol
        +str base
        +str quote
        +int price_precision
        +int size_precision
    }

    class DydxTrade {
        +datetime ts
        +float price
        +float size
        +DydxSide side
    }

    class DydxOrderBookSnapshot {
        +datetime ts
        +list~Level~ bids
        +list~Level~ asks
    }

    class Level {
        +float price
        +float size
    }

    DydxOrderBookSnapshot "*" --> Level : contains
    DydxTrade --> DydxSide : uses
```

### 7 · Execution module

```mermaid
classDiagram
    class DydxExecutionService {
        +send(order_request)
        +cancel(order_id)
        +replace(order_id, new_qty, new_price)
        +ws_handler(message)
    }

    class DydxOrderRequest {
        +str client_id
        +DydxOrderType type
        +DydxSide side
        +float qty
        +float price
    }

    class DydxOrderResponse {
        +str exchange_order_id
        +DydxOrderStatus status
        +str reason
    }

    DydxExecutionService --> DydxOrderRequest : submits
    DydxExecutionService --> DydxOrderResponse : returns
    DydxOrderResponse --> DydxOrderStatus : conveys
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

```mermaid
graph TD
    %% ────── dYdX adapter entry-point ──────
    EP["nautilus_trader.adapters.dydx<br/>__init__.py<br/>(entry-point)"]

    EP --> F1[DYDXLiveDataClientFactory]
    EP --> F2[DYDXLiveExecClientFactory]
    EP --> C1[DYDXDataClientConfig]
    EP --> C2[DYDXExecClientConfig]
    EP --> P[DYDXInstrumentProvider]

    %% ────── Factories build clients ──────
    subgraph "Factory build-time graph"
        F1 --> DC[DYDXDataClient]
        F2 --> EC[DYDXExecutionClient]

        DC --> HC[DYDXHttpClient]
        DC --> WS[DYDXWebsocketClient]

        EC --> HC
        EC --> WS
        EC --> GRPC[DYDXAccountGRPCAPI]

        DC --> P
        EC --> P
    end

    %% ────── Typical live script wiring ──────
    MM["examples/live/dydx/<br/>dydx_market_maker.py"] --> TN[TradingNode]
    TN -->|add_data_client_factory("DYDX", …)| F1
    TN -->|add_exec_client_factory("DYDX", …)| F2
```
##### 1. File-/Module-level map

```mermaid
flowchart TD
    subgraph "nautilus_trader.adapters.dydx"
        A[__init__.py<br/>(re-exports)]
        B[config.py<br/>DydxConfig]
        C[common/*<br/>constants & helpers]
        D[endpoints/*<br/>(URL builders)]
        E[http/*<br/>HTTP client]
        F[websocket/*<br/>WebSocket client]
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

##### 2. Runtime initialisation

```mermaid
sequenceDiagram
    participant Strategy
    participant Adapter as "DydxExchangeAdapter"
    participant HTTP    as "HttpClient"
    participant WS      as "WebSocketClient"
    participant DXREST  as "dYdX REST API"
    participant DXWS    as "dYdX WS API"

    Strategy  ->> Adapter: connect()
    Adapter   ->> HTTP:   start()
    Adapter   ->> WS:     start()
    HTTP      ->> DXREST: GET /accounts
    DXREST    -->> HTTP:  balances
    HTTP      -->> Adapter: balances normalised
    WS        ->> DXWS:   SUBSCRIBE trades, orders
    DXWS      -->> WS:    stream frames
    WS        -->> Adapter: envelopes (raw)
    Adapter   -->> Strategy: domain events via bus
```

##### 3. Live data & order-flow

```mermaid
flowchart LR
    subgraph "dYdX"
        A1[WebSocket stream] -->|JSON frames| B1[Deserializer]
        A2[REST endpoint]
    end

    B1 --> C[Normalizer<br/>(schemas/*)]
    A2 -.-> C

    C --> D[MessageBus]
    D --> E[Strategy]
    E --> F[OrderIntent]
    F --> G[OrderRouter<br/>(execution.py)]
    G --> H[HttpClient]
    G --> I[WebSocketClient]
    H -->|REST place order| A2
    I -->|WS private| A1
    G --> D
```


