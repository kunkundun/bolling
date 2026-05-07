# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Single-file crypto trading dashboard (`untitled.html`) — a client-side HTML/JS application that displays Binance futures data with Bollinger Band analytics and multi-timeframe trading signals.

## How to use

Open `untitled.html` directly in a browser. No build step, no server required. All data comes from public Binance REST APIs (`api.binance.com`, `fapi.binance.com`).

## Architecture

All logic lives in one `<script>` block. Key subsystems:

- **Bollinger Band strategy** (`generateEventStreamFromClosedKlines`, `buildTradesFromClosedKlines`): Uses 20-period BB (2σ). Entry on close crossing back inside bands after being outside. Exit on either stop-loss (1.5% from entry) or reversion to middle band.
- **Multi-timeframe signal detection** (`refreshAllPeriodsEvents`): Runs the strategy across `TIMEFRAMES` array (`5m` through `1w`) and surfaces the 3 most recent signals in an alert panel.
- **Charting**: Lightweight Charts v4.1.1 with candlestick series + BB upper/lower/middle + MA60. Markers painted at entry signals.
- **Period reminder filtering**: Each timeframe can be independently muted via a drawer UI; muted timeframes are excluded from alerts.
- **Sound**: Web Audio API oscillator beeps on new signals (requires user interaction first per browser autoplay policy).
- **Theme**: CSS custom properties with `dark-mode` class toggle, persisted to `localStorage`.

## API endpoints used

| Endpoint | Purpose |
|---|---|
| `api.binance.com/api/v3/klines` | OHLCV for chart, BB calculation, trading signals |
| `api.binance.com/api/v3/ticker/24hr` | Current price + 24h change |
| `fapi.binance.com/fapi/v1/premiumIndex` | Funding rate |

## Refresh intervals

- Price: every 4s
- Funding rate: every 12s
- Kline data (chart): every 10s
- Bollinger cards: every 30s
- Multi-period events: every 60s
- Trade log: every 20s

## State storage

All user preferences in `localStorage`:
- `kun_theme_mode` — `"light"` / `"dark"`
- `kun_sound_enabled` — `"true"` / `"false"`
- `kun_period_reminder` — JSON object mapping timeframe → boolean
