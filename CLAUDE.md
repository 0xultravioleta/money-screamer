# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**money-screamer** is a Fibonacci Index Screener — a single-page trading tool that detects swing highs/lows on market index intraday data and draws Fibonacci retracement levels on the chart.

Currently Phase 1: a standalone `index.html` prototype with no build step, no backend, and no dependencies beyond two CDN scripts.

## Running

Open `index.html` directly in a browser. No build, no server required.

## Architecture

Everything lives in a single `index.html` file organized into clearly marked sections:

- **CONFIGURATION** — Index definitions (SPX, NDX, DJI, RUT), timeframes, Fibonacci level constants and colors
- **DATA GENERATION** — `generateMockData()` creates realistic intraday OHLCV candles using a deterministic pseudo-random function with injected intraday patterns (morning sell-off, midday consolidation, afternoon rally)
- **SWING DETECTION** — `detectSwingPoints()` finds local highs/lows using a configurable lookback window. A swing high requires the candle's high to exceed all neighbors within the lookback range on both sides
- **FIBONACCI CALCULATION** — `buildSwingPairs()` pairs consecutive alternating swing highs/lows; `calcFibPrices()` computes standard retracement levels (0%, 23.6%, 38.2%, 50%, 61.8%, 78.6%, 100%) plus extensions (127.2%, 161.8%)
- **CHART** — Uses [TradingView Lightweight Charts v4.1.3](https://github.com/nicholasg/lightweight-charts) via CDN. Fibonacci levels are drawn as `createPriceLine()` on the candlestick series. Swing points are rendered as chart markers
- **LIVE SIMULATION** — A 1-second `setInterval` updates the last candle's close price to simulate real-time movement

## Key External Dependencies (CDN)

- `lightweight-charts@4.1.3` — charting library (standalone production build from unpkg)

## Planned Phases

- **Phase 2**: React + Vite frontend, Python/Flask backend, Yahoo Finance real data, SQLite for historical storage
- **Phase 3**: Docker Compose containerization, REST API
- **Phase 4**: Alerts on Fibonacci levels, pattern recognition, multi-chart view, CSV export

## Conventions

- All timestamps are UTC internally; displayed as EST (UTC-5) via `formatTime()`
- Prices are rounded to 2 decimal places via `round()`
- Mock data uses a seeded pseudo-random function (`pseudoRandom()` based on `Math.sin`) for deterministic output per index/timeframe combination
- State is managed via module-level variables (no framework state management yet)

---

## Development Workflow — How We Build Here

This section defines the development style for this project. Follow it strictly. The goal is task-driven, phase-based, guided development where the user always knows what's happening, what's next, and why.

### 1. Task-Driven Development

Every feature request or change gets broken into **granular, trackable tasks** using `TaskCreate`. This is not optional — it's how we stay organized.

- When the user describes what they want, immediately break it into tasks before writing any code
- Each task should be small enough to complete in one focused step (one file, one feature, one fix)
- Use `TaskUpdate` to mark tasks `in_progress` before starting and `completed` when done
- After completing a task, check `TaskList` — always know what's next
- If you discover new work mid-task, create a new task for it instead of scope-creeping

### 2. Phase-Based Planning

Structure all significant work in phases. Each phase has a clear deliverable.

- **Phase 1** is always "get something working fast" — a prototype the user can see and touch
- Subsequent phases add layers: backend, real data, containerization, polish
- Present phases to the user before starting: "Here's how I'd break this down..."
- Never jump phases. Finish and verify Phase N before starting Phase N+1
- Each phase ends with a working, testable state — never leave things half-broken between phases

### 3. Prototype First, Always

When the user asks for something new, the fastest path to a working demo wins.

- Standalone HTML/JS prototypes are valid Phase 1 deliverables — no need to scaffold a full app just to prove a concept
- Get it in front of the user fast, then iterate based on their feedback
- Use CDN dependencies for prototypes instead of setting up build tooling
- Once the concept is validated, then invest in proper architecture (React, backend, Docker, etc.)

### 4. Explain Everything You Do

The user should never wonder "what is Claude doing right now?" Be explicit.

- Before writing code, explain your plan: what you're building, which files you'll touch, and why
- After writing code, summarize what was done and what the user should expect
- When making architectural decisions, explain the tradeoffs: "I'm using X instead of Y because..."
- If something fails, explain what went wrong and what you'll try next — don't silently retry

### 5. Proactive Suggestions

Don't just execute instructions — add value. Suggest improvements, alternatives, and next steps.

- If the user asks for feature X, and you know feature Y would complement it well, mention it
- Suggest better libraries, patterns, or approaches when you see an opportunity
- Offer quick wins: "While I'm here, I noticed we could also add Z with just a few lines"
- Present options with tradeoffs when there are multiple valid approaches — use `AskUserQuestion` with clear choices
- Think about UX: suggest dark themes for trading tools, mobile responsiveness, keyboard shortcuts, etc.

### 6. Team of Agents for Bigger Tasks

When a task has independent parallel workstreams, use `TeamCreate` and spawn specialized agents.

- **Frontend agent**: React/UI work, styling, component architecture
- **Backend agent**: API, database, data fetching, business logic
- **DevOps agent**: Docker, CI/CD, deployment configuration
- Coordinate via task list — agents claim tasks, mark complete, and move on
- Always brief agents with full context: what the project is, what Phase we're in, what they own

### 7. Git Discipline

Follow conventional commits and commit often.

- `feat:` for new features, `fix:` for bugs, `refactor:` for restructuring, `docs:` for documentation, `chore:` for tooling
- Commit after each completed task — small, atomic commits
- Never bundle unrelated changes in one commit
- Verify the remote state before creating PRs: `git show origin/branch:file`
- When pushing, always confirm with the user first — pushing is a visible action

### 8. Verification Before Moving On

Never claim something is done without verifying it works.

- After creating/modifying frontend code: open it in a browser and confirm
- After API changes: test with `curl` or equivalent
- After Docker changes: build and run the container, verify the health endpoint
- Read your own output — if a command errors, address it before continuing
- Use dry-runs for anything destructive or complex

### 9. Task Completion Format

When finishing a task or phase, always report back in this format:

```
TASK COMPLETED: One-line description
CURRENT PHASE: Phase X - Name

What Was Accomplished:
- Files created/modified
- Features implemented

ACTION REQUIRED: [what the user needs to do next, if anything]

Next Steps:
- What comes next in the task list or phase plan
```

### 10. Security Defaults

- Never expose API keys, secrets, or credentials in code, logs, or terminal output
- Use `.env.example` with placeholder values, never real `.env` files in the repo
- Audit any external skills, packages, or code before executing — read the source first
- If the user asks to install a third-party tool or skill, review its code before running it

### 11. Design Preferences for This Project

- Dark theme (trading terminal aesthetic): background `#0d1117`, surfaces `#161b22`
- Color palette: bullish green `#3fb68b`, bearish red `#ff6b6b`, accent blue `#58a6ff`
- No grays for status colors — use the palette above
- Professional, clean UI — no rounded cartoon styles, think Bloomberg Terminal meets modern web
- Responsive layout but desktop-first (traders use monitors)

### 12. When the User Says Something Vague

If the user gives a high-level request like "make it better" or "add more features":

1. Ask clarifying questions using `AskUserQuestion` with concrete options
2. Propose a phased plan before writing any code
3. Start with the smallest useful increment
4. Check in after each increment: "Here's what I did. Want to continue with X or pivot to Y?"

### 13. How to Use Claude Code Effectively (For New Users)

If the user is new to Claude Code, guide them:

- Suggest using `/commit` for git commits, `/review-pr` for PR reviews
- Explain that they can give high-level instructions and you'll break them down
- Show them the power of parallel agents for complex tasks
- Remind them they can say "stop" or "undo" at any time
- Encourage them to describe what they want in natural language — no need for technical precision
