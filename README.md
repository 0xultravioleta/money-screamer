# Money Screamer

A real-time Fibonacci Index Screener that automatically detects swing highs and swing lows throughout the trading day and draws Fibonacci retracement levels on every swing pair. Built for traders who use Fibonacci analysis on major market indices.

## The Idea

Fibonacci retracement is one of the most widely used tools in technical analysis. Traders draw Fibonacci lines between swing highs and swing lows to identify potential support/resistance levels and reversal zones. The problem? Doing this manually throughout the day is tedious — you have to constantly identify new swings and redraw lines as price action develops.

**Money Screamer automates this entire process.** It:

- Scans intraday price action in real-time
- Detects every swing high and swing low using a configurable lookback algorithm
- Automatically draws Fibonacci retracement levels (0%, 23.6%, 38.2%, 50%, 61.8%, 78.6%, 100%) between each swing pair
- Shows Fibonacci extensions (127.2%, 161.8%) for breakout targets
- Tells you exactly which Fibonacci zone the current price is sitting in
- Lets you navigate between all detected swing pairs throughout the day

## Current State — Phase 1 (Standalone Prototype)

A fully functional single-file prototype (`index.html`) that runs directly in any browser with zero setup.

### Features

- **4 Major Indices**: S&P 500 (SPX), NASDAQ 100 (NDX), Dow Jones (DJI), Russell 2000 (RUT)
- **4 Timeframes**: 1-minute, 5-minute, 15-minute, 1-hour candles
- **Automatic Swing Detection**: Finds swing highs and swing lows using a lookback window algorithm (configurable from 2 to 10 candles)
- **Full Fibonacci Suite**: Standard retracements (0%, 23.6%, 38.2%, 50%, 61.8%, 78.6%, 100%) plus extensions (127.2%, 161.8%)
- **Fibonacci Lines on Chart**: Color-coded dashed lines drawn directly on the candlestick chart at each Fibonacci level
- **Swing Pair Navigation**: Browse through all detected swing high/low pairs with prev/next controls
- **Current Zone Indicator**: Shows which two Fibonacci levels the price is currently between
- **Show All Zones**: Toggle to overlay Fibonacci levels from every swing pair simultaneously
- **Live Price Simulation**: Real-time price updates every second with a pulsing live indicator
- **Volume Bars**: Histogram volume display at the bottom of the chart
- **Swing Point Markers**: Red arrows for swing highs, green arrows for swing lows directly on candles
- **Professional Dark Theme**: Trading terminal aesthetic with TradingView Lightweight Charts

### How to Run

```bash
# Just open the file
open index.html
# Or on Windows
start index.html
```

No dependencies to install. No build step. No server needed.

### Screenshot Layout

```
+----------------------------------------------------------+
|  FIBSCREENER  | [SPX] [NDX] [DJI] [RUT] | 1m 5m 15m 1H  |
+----------------------------------------------------------+
|                                    | CURRENT PRICE ZONE   |
|                                    | 50% - 61.8%          |
|    Candlestick Chart               | 5,837.77 - 5,860.47  |
|    with Fibonacci lines            |                      |
|    and swing markers               | FIBONACCI LEVELS     |
|         SH ▼                       | ██ 100%    5,891.23  |
|    -------- 61.8% --------         | ██  78.6%  5,874.15  |
|    -------- 50.0% --------         | ██  61.8%  5,860.47  |
|         SL ▲                       | ██  50%    5,849.12  |
|    -------- 23.6% --------         | ██  38.2%  5,837.77  |
|                                    | ██  23.6%  5,827.89  |
|   ▁▃▅▂▁▃▆▅▃▂▁▃▅▇▅▃ (volume)      | ██   0%    5,812.01  |
+----------------------------------------------------------+
|  SPX 5,856.32  +12.45 (+0.21%)  |  O H L Vol Swings    |
+----------------------------------------------------------+
```

### Tech Stack (Phase 1)

| Component | Technology |
|-----------|-----------|
| Chart | [TradingView Lightweight Charts v4.1.3](https://github.com/nicholasg/lightweight-charts) |
| UI | Vanilla HTML/CSS/JS |
| Data | Mock data with realistic intraday patterns |
| Hosting | Static file (no server) |

## Roadmap

### Phase 2 — Real Data + Full Stack App

Replace mock data with real market data and build a proper application architecture.

**Backend (Python + Flask)**
- REST API serving real intraday OHLCV data via [yfinance](https://github.com/ranaroussi/yfinance) (free, no API key)
- SQLite database for caching historical data and reducing API calls
- Server-side swing detection and Fibonacci calculation endpoints
- Endpoints: `GET /api/indices`, `GET /api/candles/:symbol`, `GET /api/fibonacci/:symbol`

**Frontend (React + Vite)**
- Migrate the prototype to a React single-page application
- Component architecture: `<Chart>`, `<FibPanel>`, `<SwingList>`, `<IndexSelector>`, `<Settings>`
- State management with React hooks (useState/useContext)
- Keep the same dark theme and TradingView Lightweight Charts integration
- API client layer to fetch from the Flask backend

**Data Flow**
```
Yahoo Finance API → Flask Backend → SQLite Cache
                         ↓
                    REST API JSON
                         ↓
              React Frontend → Chart
```

### Phase 3 — Docker + Production Ready

Containerize everything for consistent local development and easy deployment.

**Docker Compose Setup**
- `frontend` container: Node.js build → nginx serving static files
- `backend` container: Python/Flask API with SQLite volume mount
- Single `docker-compose up` to run the entire stack
- Health check endpoints for both services
- Environment variable configuration via `.env.example`

**Additional Features**
- Auto-refresh data on configurable intervals (1min, 5min, etc.)
- Date picker to load historical trading days
- Multiple indices on screen simultaneously (split view)
- Export Fibonacci analysis to CSV/JSON

### Phase 4 — Advanced Analysis (Future)

- **Fibonacci Confluence Zones**: Highlight price levels where multiple Fibonacci retracements from different swing pairs overlap — these are the strongest support/resistance areas
- **Alert System**: Desktop notifications when price enters a key Fibonacci zone (61.8%, 78.6%)
- **Pattern Recognition**: Detect common Fibonacci-based patterns (golden pocket entries, extension targets)
- **Multi-Chart Dashboard**: View 4 indices simultaneously in a grid layout
- **WebSocket Live Data**: Replace polling with real-time streaming for sub-second updates
- **Backtesting Module**: Test how price has historically reacted to Fibonacci levels on each index

## Swing Detection Algorithm

The swing detection uses a **lookback window** approach:

- A **Swing High** is a candle whose high price is strictly greater than the highs of all `N` candles to its left AND `N` candles to its right
- A **Swing Low** is a candle whose low price is strictly less than the lows of all `N` candles to its left AND `N` candles to its right
- `N` (lookback) is configurable from 2 to 10 — lower values detect more swings, higher values only catch major swings

Once swings are detected, they are paired into consecutive high-low or low-high pairs. Fibonacci retracement levels are calculated for each pair:

```
Fibonacci Level = Swing Low + (Swing High - Swing Low) × Ratio

Ratios: 0%, 23.6%, 38.2%, 50%, 61.8%, 78.6%, 100%
Extensions: 127.2%, 161.8%
```

## Project Structure

```
money-screamer/
├── index.html      # Phase 1 standalone prototype (fully functional)
├── CLAUDE.md       # Development workflow guide for Claude Code
└── README.md       # This file
```

After Phase 2:
```
money-screamer/
├── index.html              # Phase 1 prototype (preserved)
├── frontend/               # React + Vite app
│   ├── src/
│   │   ├── components/     # Chart, FibPanel, SwingList, etc.
│   │   ├── hooks/          # Custom hooks for data fetching
│   │   ├── services/       # API client
│   │   └── App.jsx
│   ├── package.json
│   └── vite.config.js
├── backend/                # Python Flask API
│   ├── app.py              # Flask routes
│   ├── models.py           # SQLite models
│   ├── services/           # yfinance fetcher, Fibonacci calculator
│   └── requirements.txt
├── docker-compose.yml
├── CLAUDE.md
└── README.md
```

## Built With

This project was built using [Claude Code](https://claude.ai/code) — from initial prototype to full-stack application, developed through conversational AI-driven development with task tracking, phased planning, and parallel agent teams.

## License

MIT
