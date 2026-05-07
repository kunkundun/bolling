# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 在此仓库中工作时提供指引。

## 项目概览

单文件加密货币交易仪表盘 (`untitled.html`) — 一个纯前端 HTML/JS 应用，展示币安合约数据，包含布林带分析和多周期交易信号。

## 使用方式

直接在浏览器中打开 `untitled.html`。无需构建步骤，无需服务器。所有数据来自币安公开 REST API (`api.binance.com`、`fapi.binance.com`)。

## 架构

所有逻辑位于单个 `<script>` 块中。关键子系统：

- **布林带策略** (`generateEventStreamFromClosedKlines`, `buildTradesFromClosedKlines`)：使用 20 周期布林带（2σ）。价格突破布林带外之后回落至上/下轨之内时入场。止损（入场价的 1.5%）或回归中轨时离场。
- **多周期信号检测** (`refreshAllPeriodsEvents`)：在 `TIMEFRAMES` 数组（`5m` 到 `1w`）上运行策略，将最近 3 条信号展示在提醒面板中。
- **图表**：Lightweight Charts v4.1.1，包含 K 线序列 + 布林带上/中/下轨 + MA60。入场点标记在 K 线图上。
- **周期提醒过滤**：每个周期可通过抽屉面板独立静音，被静音的周期不会出现在提醒中。
- **声音**：Web Audio API 振荡器在新信号出现时发出提示音（需用户首次交互以符合浏览器自动播放策略）。
- **主题**：通过 CSS 自定义属性配合 `dark-mode` class 切换，偏好存储在 `localStorage`。

## 使用的 API 端点

| 端点 | 用途 |
|---|---|
| `api.binance.com/api/v3/klines` | OHLCV 数据，用于图表、布林带计算、交易信号 |
| `api.binance.com/api/v3/ticker/24hr` | 当前价格和 24 小时涨跌幅 |
| `fapi.binance.com/fapi/v1/premiumIndex` | 资金费率 |

## 刷新频率

- 价格：每 4 秒
- 资金费率：每 12 秒
- K 线数据（图表）：每 10 秒
- 布林带卡片：每 30 秒
- 多周期事件：每 60 秒
- 交易日志：每 20 秒

## 状态存储

所有用户偏好存储在 `localStorage` 中：
- `kun_theme_mode` — `"light"` / `"dark"`
- `kun_sound_enabled` — `"true"` / `"false"`
- `kun_period_reminder` — JSON 对象，timeframe → boolean
