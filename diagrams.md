
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
