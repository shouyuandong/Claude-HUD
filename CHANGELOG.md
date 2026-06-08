# Change Log

All notable changes to the "claude-hud" extension will be documented in this file.

## 0.1.1 - 2026-06-09

### Added

- **Pre-fill 20 K-line candles on open** — Candlestick chart now immediately shows 20 historical candles using deterministic PRNG, instead of appearing one-by-one
- **OHLC precision** — Open/High/Low/Close values now display with 1 decimal place throughout (data layer and frontend)
- **Burst Rate candlestick bias** — K-line direction now influenced by token consumption rate: high consumption pushes price up, low consumption keeps it flat, with realistic ±5% clamp
- **Chinese stock K-line colors** — Auto-detects VS Code locale, red for up and green for down (Chinese convention)
- **Deterministic PRNG** — Candlestick simulation uses sine-hash math instead of Math.random (reliable across ticks)

### Changed

- **Simplified tracer ball physics** — Replaced complex rate→position math with a simple kick-and-gravity model. Activity kicks the ball upward with fixed velocity so it always reaches the same peak (h*20%). Higher token rate just reduces gravity so the ball floats longer before settling. Uses minimal per-frame state (`_vy`, `_activity`)
- **Cleaned up stale variable references** — Removed lingering `ratio` and `prevY` references after physics rewrite
- **Realistic price simulation** — Migrated from absolute math (±64% swings) to percentage-based volatility (~1.2%, clamped ±5%) with mean reversion toward 100
- **Y-axis scaling** — Switched from wick-inclusive (high/low) to body-only (open/close) range with 15% padding, preventing candle compression from spike wicks
- **Limit to 20 candles** — K-line chart now only shows last 20 candles via slice(-20)
- **Cleaned up debug logging** — Removed 9 `[HUD DEBUG]` console.log lines from dataProvider

## 0.1.0 - 2026-06-09

### Added

- Initial release
- **Token Flow** — Real-time token rate visualization (matrix rain / candlestick / tracer-ball)
- **Context Window** — Progress bar showing current usage vs limit
- **Token Usage** — IN/OUT token breakdown with proportional bar
- **Agent Status** — Multi-agent tree with tasks, progress, and token counts
- **History Chart** — Token usage history over 24h or 7d
- **Plan Mode Badge** — Indicates when Claude is in Plan Mode
- **Drag & Reorder** — Freely reorder modules by dragging headers
- **Auto Theme** — Automatically follows VS Code light / dark theme
- **Cost Tracking** — Token cost estimation by model
- **Session Timer** — Running session duration display
- **Multi-language** — Auto / English / 中文 / 日本語 UI support
- **Module Configuration** — Per-module visibility via `claudeHud.modules`
- **Status Bar** — Quick token usage overview in VS Code status bar
