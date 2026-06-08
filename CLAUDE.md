# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm install          # Install dependencies
npm run compile      # Compile TypeScript (tsc -p ./)
npm run watch        # Watch mode (tsc -watch)
```

- To test: launch VS Code extension host via F5 (no test framework configured)
- Output goes to `out/`

## Architecture

A VS Code extension that reads Claude Code's real-time session data from disk (`~/.claude/`) and renders it as a HUD in the sidebar Activity Bar.

### Data Flow (200ms tick loop)

```
[Claude Code JSONL on disk]
        ↓ reads
  DataProvider.tickOnce()        ← called every 200ms from extension.ts
        ↓ produces
  HUDData snapshot
        ↓ pushed via postMessage
  HUDPanelProvider (webview)     ← renders canvas bars, charts, agent tree
  StatusBarManager               ← text: "12.4k/200k (6%)"
```

### Key Files

| File | Role |
|---|---|
| [src/extension.ts](src/extension.ts) | Extension entry — wires up all components, starts the 200ms tick loop |
| [src/dataProvider.ts](src/dataProvider.ts) | Reads & parses `~/.claude/sessions/*.json` and `~/.claude/projects/**/*.jsonl` to extract tokens, agents, plan mode, context |
| [src/hudPanel.ts](src/hudPanel.ts) | WebviewViewProvider — serves HTML, handles config/theme messaging, drag-reorder modules |
| [src/statusBar.ts](src/statusBar.ts) | VS Code status bar item (token count + color-coded percentage) |
| [src/configManager.ts](src/configManager.ts) | Read/write `claudeHud.modules` settings, emit change events |
| [src/historyStore.ts](src/historyStore.ts) | Persist hourly/daily token history in VS Code globalState |
| [src/types.ts](src/types.ts) | All shared interfaces (`HUDData`, `AgentStatus`, `CandleData`, `WebviewMessage`, etc.) |
| [src/webview/script.js](src/webview/script.js) | Frontend — canvas rendering (burst bars, candlestick, history chart), drag-reorder, settings overlay, theme detection |
| [src/webview/styles.css](src/webview/styles.css) | Zinc/Emerald palette, spring physics (`cubic-bezier(0.16, 1, 0.3, 1)`), light/dark themes |

### Data Source (Claude Code on Disk)

- **Session metadata**: `~/.claude/sessions/<pid>.json` — contains `sessionId`, `cwd`, `pid`, `startedAt`
- **Conversation logs**: `~/.claude/projects/<sanitized-cwd>/<sessionId>.jsonl` — JSONL with `user` / `assistant` messages containing `usage.input_tokens`, `usage.output_tokens`, `model` name
- **Subagents**: `~/.claude/projects/<sanitized-cwd>/<sessionId>/<sessionId>/subagents/agent-*.jsonl`
- **Plan mode**: Detected from `attachment` entries with type `plan_mode` / `plan_mode_exit` in the JSONL

### Key Behaviors

- **Model detection**: Scans JSONL backwards for `message.model` to pick context window limits (200K for Claude, 1M for DeepSeek/Gemini)
- **Incremental reads**: Only reads new bytes since last tick via `fs.readSync` with offset
- **Fallback sessions**: When no active PID found, uses most recent `.jsonl` for the current project
- **Provider-aware context estimation**: Anthropic API (cache values additive) vs DeepSeek/Gemini (input_tokens includes cache)
- **Candlestick tracking**: Every 20 ticks (~4s) produces a candle (O/H/L/C) for the K-line view
- **Theme sync**: VS Code `onDidChangeActiveColorTheme` propagated to webview as `themeChanged` message

### Module System

Six toggleable modules controlled via `claudeHud.modules` in VS Code settings:
`agentStatus`, `tokenFlow` (burst/candle visualization), `contextWindow`, `tokenUsage` (IN/OUT breakdown), `historyChart` (24h/7d), `sessionTime`.

## Model Context Window Reference

- Claude (Opus 4.x, Sonnet 4.x, Haiku 4.x, 3.5): 200K tokens
- DeepSeek (v4-flash, v3, r1): 1M tokens
- Gemini (2.5, 2.0): 1M tokens
