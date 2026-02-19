# Money Screamer — Architecture Diagrams

## 1. System Architecture Overview

```mermaid
graph TB
    subgraph Phase1["Phase 1 — Standalone Prototype"]
        HTML["index.html<br/>Single File App"]
        LWC["TradingView<br/>Lightweight Charts"]
        MOCK["Mock Data<br/>Generator"]
        HTML --> LWC
        HTML --> MOCK
    end

    subgraph Phase2["Phase 2 — Full Stack"]
        subgraph Frontend["Frontend Container"]
            REACT["React + Vite"]
            COMP["Components"]
            HOOKS["Custom Hooks"]
            API_CLIENT["API Client"]
            REACT --> COMP
            REACT --> HOOKS
            HOOKS --> API_CLIENT
        end

        subgraph Backend["Backend Container"]
            FLASK["Flask API"]
            YF["yfinance<br/>Data Fetcher"]
            FIB_ENGINE["Fibonacci<br/>Engine"]
            SWING_ENGINE["Swing<br/>Detector"]
            SQLITE[("SQLite<br/>Cache DB")]
            FLASK --> YF
            FLASK --> FIB_ENGINE
            FLASK --> SWING_ENGINE
            FLASK --> SQLITE
        end

        API_CLIENT -->|"REST API"| FLASK
    end

    subgraph Phase3["Phase 3 — Containerized"]
        DOCKER["Docker Compose"]
        NGINX["nginx<br/>Static Server"]
        PYTHON["Python<br/>API Server"]
        VOL[("SQLite<br/>Volume")]
        DOCKER --> NGINX
        DOCKER --> PYTHON
        PYTHON --> VOL
    end

    subgraph External["External Services"]
        YAHOO["Yahoo Finance API"]
    end

    YF -->|"HTTP"| YAHOO

    style Phase1 fill:#1a1a2e,stroke:#58a6ff,color:#e6edf3
    style Phase2 fill:#16213e,stroke:#3fb68b,color:#e6edf3
    style Phase3 fill:#0f3460,stroke:#ff9800,color:#e6edf3
    style External fill:#1a1a2e,stroke:#ff6b6b,color:#e6edf3
    style SQLITE fill:#161b22,stroke:#58a6ff,color:#e6edf3
    style VOL fill:#161b22,stroke:#ff9800,color:#e6edf3
```

## 2. Data Pipeline

```mermaid
flowchart LR
    subgraph Source["Data Source"]
        YF["Yahoo Finance<br/>yfinance"]
    end

    subgraph Ingestion["Ingestion Layer"]
        FETCH["Fetch OHLCV<br/>Intraday Data"]
        VALIDATE["Validate &<br/>Normalize"]
        CACHE["Cache to<br/>SQLite"]
    end

    subgraph Processing["Processing Engine"]
        SWING["Swing<br/>Detection"]
        PAIR["Swing Pair<br/>Builder"]
        FIB["Fibonacci<br/>Calculator"]
        ZONE["Zone<br/>Classifier"]
    end

    subgraph Delivery["API Delivery"]
        REST["REST<br/>Endpoints"]
        JSON["JSON<br/>Response"]
    end

    subgraph Rendering["Frontend Rendering"]
        CHART["Candlestick<br/>Chart"]
        LINES["Fibonacci<br/>Price Lines"]
        MARKERS["Swing<br/>Markers"]
        SIDEBAR["Sidebar<br/>Panels"]
    end

    YF --> FETCH --> VALIDATE --> CACHE
    CACHE --> SWING --> PAIR --> FIB --> ZONE
    ZONE --> REST --> JSON
    JSON --> CHART
    JSON --> LINES
    JSON --> MARKERS
    JSON --> SIDEBAR

    style Source fill:#1a1a2e,stroke:#ff6b6b,color:#e6edf3
    style Ingestion fill:#1a1a2e,stroke:#ff9800,color:#e6edf3
    style Processing fill:#1a1a2e,stroke:#4caf50,color:#e6edf3
    style Delivery fill:#1a1a2e,stroke:#58a6ff,color:#e6edf3
    style Rendering fill:#1a1a2e,stroke:#a371f7,color:#e6edf3
```

## 3. Swing Detection Algorithm

```mermaid
flowchart TD
    START(["Start: Receive<br/>Candle Array"]) --> INIT["Set i = lookback"]

    INIT --> CHECK{"i < length<br/>- lookback?"}
    CHECK -->|No| RETURN(["Return<br/>Swing Points"])

    CHECK -->|Yes| HIGH_CHECK["isHigh = true<br/>isLow = true"]
    HIGH_CHECK --> INNER_LOOP["j = 1"]

    INNER_LOOP --> J_CHECK{"j <= lookback?"}

    J_CHECK -->|Yes| TEST_HIGH{"candle[i].high ><br/>candle[i-j].high<br/>AND<br/>candle[i].high ><br/>candle[i+j].high?"}

    TEST_HIGH -->|No| FAIL_HIGH["isHigh = false"]
    TEST_HIGH -->|Yes| TEST_LOW

    FAIL_HIGH --> TEST_LOW{"candle[i].low <<br/>candle[i-j].low<br/>AND<br/>candle[i].low <<br/>candle[i+j].low?"}

    TEST_LOW -->|No| FAIL_LOW["isLow = false"]
    TEST_LOW -->|Yes| INC_J["j++"]
    FAIL_LOW --> INC_J

    INC_J --> J_CHECK

    J_CHECK -->|No| EMIT_CHECK{"isHigh OR<br/>isLow?"}

    EMIT_CHECK -->|"isHigh"| EMIT_SH["Emit Swing High<br/>price = candle[i].high"]
    EMIT_CHECK -->|"isLow"| EMIT_SL["Emit Swing Low<br/>price = candle[i].low"]
    EMIT_CHECK -->|"Both"| EMIT_BOTH["Emit Both<br/>SH + SL"]
    EMIT_CHECK -->|"Neither"| INC_I["i++"]

    EMIT_SH --> INC_I
    EMIT_SL --> INC_I
    EMIT_BOTH --> INC_I
    INC_I --> CHECK

    style START fill:#58a6ff,stroke:#58a6ff,color:#fff
    style RETURN fill:#3fb68b,stroke:#3fb68b,color:#fff
    style EMIT_SH fill:#ff6b6b,stroke:#ff6b6b,color:#fff
    style EMIT_SL fill:#3fb68b,stroke:#3fb68b,color:#fff
    style EMIT_BOTH fill:#ff9800,stroke:#ff9800,color:#fff
```

## 4. Fibonacci Calculation

```mermaid
flowchart LR
    subgraph Input
        SH["Swing High<br/>Price: H"]
        SL["Swing Low<br/>Price: L"]
    end

    subgraph Calculate["Calculate Levels"]
        DIFF["diff = H - L"]
        F0["0% = L"]
        F236["23.6% = L + diff × 0.236"]
        F382["38.2% = L + diff × 0.382"]
        F50["50% = L + diff × 0.5"]
        F618["61.8% = L + diff × 0.618"]
        F786["78.6% = L + diff × 0.786"]
        F100["100% = H"]
        F1272["127.2% = L + diff × 1.272"]
        F1618["161.8% = L + diff × 1.618"]
    end

    subgraph Output["Chart Output"]
        LINES["Horizontal<br/>Price Lines"]
        LABELS["Axis Labels<br/>with %"]
        ZONE["Current<br/>Price Zone"]
    end

    SH --> DIFF
    SL --> DIFF
    DIFF --> F0
    DIFF --> F236
    DIFF --> F382
    DIFF --> F50
    DIFF --> F618
    DIFF --> F786
    DIFF --> F100
    DIFF --> F1272
    DIFF --> F1618

    F0 --> LINES
    F236 --> LINES
    F382 --> LINES
    F50 --> LINES
    F618 --> LINES
    F786 --> LINES
    F100 --> LINES
    F1272 --> LINES
    F1618 --> LINES

    LINES --> LABELS
    LINES --> ZONE

    style F0 fill:#787B86,stroke:#787B86,color:#fff
    style F236 fill:#e91e63,stroke:#e91e63,color:#fff
    style F382 fill:#ff5722,stroke:#ff5722,color:#fff
    style F50 fill:#ff9800,stroke:#ff9800,color:#fff
    style F618 fill:#4caf50,stroke:#4caf50,color:#fff
    style F786 fill:#2196f3,stroke:#2196f3,color:#fff
    style F100 fill:#787B86,stroke:#787B86,color:#fff
    style F1272 fill:#9c27b0,stroke:#9c27b0,color:#fff
    style F1618 fill:#7c4dff,stroke:#7c4dff,color:#fff
```

## 5. React Component Architecture (Phase 2)

```mermaid
graph TD
    APP["App"]

    APP --> HEADER["Header"]
    APP --> MAIN["MainLayout"]
    APP --> FOOTER["StatusBar"]

    HEADER --> IDX_SEL["IndexSelector<br/><i>SPX NDX DJI RUT</i>"]
    HEADER --> TF_SEL["TimeframeSelector<br/><i>1m 5m 15m 1H</i>"]
    HEADER --> LIVE["LiveBadge"]

    MAIN --> CHART_AREA["ChartArea"]
    MAIN --> SIDEBAR["Sidebar"]

    CHART_AREA --> CHART["CandlestickChart<br/><i>lightweight-charts</i>"]
    CHART_AREA --> FIB_OVERLAY["FibonacciOverlay<br/><i>Price Lines</i>"]
    CHART_AREA --> SWING_MARKERS["SwingMarkers<br/><i>Arrows on chart</i>"]
    CHART_AREA --> VOLUME["VolumeHistogram"]

    SIDEBAR --> ZONE_PANEL["ZoneIndicator<br/><i>Current Fib zone</i>"]
    SIDEBAR --> FIB_PANEL["FibLevelsPanel<br/><i>Level list + prices</i>"]
    SIDEBAR --> SWING_LIST["SwingPointsList<br/><i>Clickable swings</i>"]
    SIDEBAR --> PAIR_NAV["PairNavigator<br/><i>Prev / Next</i>"]
    SIDEBAR --> SETTINGS["SettingsPanel<br/><i>Lookback, toggles</i>"]

    FOOTER --> PRICE_DISPLAY["PriceDisplay<br/><i>Current + change</i>"]
    FOOTER --> STATS["DayStats<br/><i>O H L Vol Swings</i>"]

    style APP fill:#58a6ff,stroke:#58a6ff,color:#fff
    style HEADER fill:#1c2128,stroke:#30363d,color:#e6edf3
    style MAIN fill:#1c2128,stroke:#30363d,color:#e6edf3
    style FOOTER fill:#1c2128,stroke:#30363d,color:#e6edf3
    style CHART fill:#161b22,stroke:#3fb68b,color:#e6edf3
    style FIB_OVERLAY fill:#161b22,stroke:#ff9800,color:#e6edf3
    style SWING_MARKERS fill:#161b22,stroke:#ff6b6b,color:#e6edf3
```

## 6. API Sequence — Loading an Index

```mermaid
sequenceDiagram
    actor User
    participant React as React Frontend
    participant Flask as Flask API
    participant Cache as SQLite Cache
    participant Yahoo as Yahoo Finance

    User->>React: Select "NDX" index

    React->>Flask: GET /api/candles/NDX?tf=5m

    Flask->>Cache: Query cached data
    alt Cache Hit (fresh data)
        Cache-->>Flask: Return cached OHLCV
    else Cache Miss or Stale
        Flask->>Yahoo: Fetch intraday data
        Yahoo-->>Flask: OHLCV response
        Flask->>Cache: Store/update cache
    end

    Flask->>Flask: detectSwingPoints(candles, lookback)
    Flask->>Flask: buildSwingPairs(swings)
    Flask->>Flask: calcFibLevels(pairs)

    Flask-->>React: JSON Response
    Note over Flask,React: { candles, volumes,<br/>swings, pairs,<br/>fibLevels, currentZone }

    React->>React: Update chart data
    React->>React: Draw Fibonacci lines
    React->>React: Place swing markers
    React->>React: Update sidebar panels

    React-->>User: Chart rendered with Fibonacci
```

## 7. Docker Infrastructure (Phase 3)

```mermaid
graph TB
    subgraph Docker["Docker Compose Network"]
        subgraph FE["frontend container"]
            NGINX["nginx:alpine"]
            STATIC["Static Build<br/>/usr/share/nginx/html"]
            NGINX_CONF["nginx.conf<br/>proxy /api → backend"]
            NGINX --> STATIC
            NGINX --> NGINX_CONF
        end

        subgraph BE["backend container"]
            PYTHON["Python 3.11<br/>slim"]
            FLASK_APP["Flask App<br/>:5000"]
            GUNICORN["Gunicorn<br/>WSGI Server"]
            PYTHON --> GUNICORN --> FLASK_APP
        end

        subgraph Storage["Persistent Storage"]
            DB_VOL[("SQLite Volume<br/>/data/screamer.db")]
        end

        NGINX_CONF -->|"/api/* proxy_pass"| FLASK_APP
        FLASK_APP --> DB_VOL
    end

    USER["User Browser<br/>localhost:3000"] --> NGINX
    YAHOO["Yahoo Finance<br/>API"] --> FLASK_APP

    style Docker fill:#0d1117,stroke:#30363d,color:#e6edf3
    style FE fill:#161b22,stroke:#58a6ff,color:#e6edf3
    style BE fill:#161b22,stroke:#3fb68b,color:#e6edf3
    style Storage fill:#161b22,stroke:#ff9800,color:#e6edf3
    style USER fill:#58a6ff,stroke:#58a6ff,color:#fff
    style YAHOO fill:#ff6b6b,stroke:#ff6b6b,color:#fff
```

## 8. State Management Flow (Phase 2)

```mermaid
stateDiagram-v2
    [*] --> Idle: App Loaded

    Idle --> FetchingData: User selects index/timeframe
    FetchingData --> ProcessingSwings: Data received
    ProcessingSwings --> CalculatingFib: Swings detected
    CalculatingFib --> RenderingChart: Fib levels computed
    RenderingChart --> LiveUpdating: Chart rendered

    LiveUpdating --> LiveUpdating: Price tick (1s interval)
    LiveUpdating --> FetchingData: User changes index
    LiveUpdating --> RecalculatingFib: User changes lookback
    LiveUpdating --> TogglingDisplay: User toggles setting

    RecalculatingFib --> ProcessingSwings: Re-detect with new lookback
    TogglingDisplay --> RenderingChart: Re-render with new settings

    LiveUpdating --> NavigatingPairs: User clicks prev/next
    NavigatingPairs --> RenderingChart: Redraw Fib for selected pair

    state FetchingData {
        [*] --> CheckCache
        CheckCache --> ReturnCached: Cache fresh
        CheckCache --> FetchYahoo: Cache stale
        FetchYahoo --> UpdateCache
        UpdateCache --> ReturnCached
    }
```

## 9. Fibonacci Confluence Detection (Phase 4)

```mermaid
flowchart TD
    subgraph Pairs["All Swing Pairs"]
        P1["Pair 1<br/>SH: 5891 → SL: 5812"]
        P2["Pair 2<br/>SL: 5812 → SH: 5867"]
        P3["Pair 3<br/>SH: 5867 → SL: 5834"]
    end

    subgraph FibCalc["Fibonacci Levels per Pair"]
        F1["Pair 1: 61.8% = 5,860.84"]
        F2["Pair 2: 38.2% = 5,833.02"]
        F3["Pair 3: 50.0% = 5,850.50"]
        F4["Pair 1: 50.0% = 5,851.50"]
        F5["Pair 2: 61.8% = 5,846.00"]
    end

    subgraph Confluence["Confluence Zones"]
        CZ1["STRONG ZONE<br/>5,849 - 5,852<br/>Pair 1 (50%) + Pair 3 (50%)<br/>2 overlapping levels"]
        CZ2["MEDIUM ZONE<br/>5,859 - 5,862<br/>Pair 1 (61.8%)<br/>1 level near resistance"]
    end

    subgraph Signal["Trading Signal"]
        STRONG["HIGH CONFLUENCE<br/>Strong Support/Resistance<br/>Multiple Fib levels cluster"]
        WEAK["LOW CONFLUENCE<br/>Single Fib level<br/>Less reliable zone"]
    end

    P1 --> F1
    P1 --> F4
    P2 --> F2
    P2 --> F5
    P3 --> F3

    F4 --> CZ1
    F3 --> CZ1
    F1 --> CZ2

    CZ1 --> STRONG
    CZ2 --> WEAK

    style STRONG fill:#3fb68b,stroke:#3fb68b,color:#fff
    style WEAK fill:#ff9800,stroke:#ff9800,color:#fff
    style CZ1 fill:#1a1a2e,stroke:#3fb68b,color:#e6edf3
    style CZ2 fill:#1a1a2e,stroke:#ff9800,color:#e6edf3
```
