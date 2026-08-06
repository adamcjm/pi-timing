# pi-timing

> [中文文档 (Chinese)](README.zh-CN.md)

Timing widget for [pi](https://pi.dev) — shows how long each turn actually took, and how much of your session was real work vs idle waiting.

## Features

- **Per-turn generation time** — wall-clock of each LLM stream (thinking + reply), with an estimated thinking split (`think≈`)
- **Tool time** — wall-clock execution; parallel tools counted once
- **Cumulative active time** — Σ(generation) + Σ(tools), excluding idle; survives restarts via per-turn `timing-snapshot` entries
- **Session span** — wall-clock from first to last message (includes idle), shown first
- **History recompute** — restores totals from the session file on restart; `/tree` branches include pre-jump history (ancestor chain)
- **i18n** — widget language follows your message language automatically (zh/en), starts from your system locale; force with `/timing lang`

## Install

```bash
pi install git:github.com/adamcjm/pi-timing
```

Or copy `extensions/timing` into `~/.pi/agent/extensions/` (or `.pi/extensions/` for a project).

## Usage

After installing, run `/reload` (or restart pi) to activate. The widget appears above the input editor.

### Commands

| Command | Action |
|---|---|
| `/timing` | Toggle the widget on/off |
| `/timing list` | Toggle the detail list (last 20 turns) |
| `/timing lang zh` | Force Chinese |
| `/timing lang en` | Force English |
| `/timing lang auto` | Back to automatic (follows your message language) |

### Widget example

Idle:

```
⏱ 会话跨度 10h31m  ·  累计活跃 36m26s (14轮)  ·  上次回复 30.4s (生成 25.2s · 思考≈25.2s · 工具 3.0s)
⏱ span 10h31m  ·  active 36m26s (14 turns)  ·  last reply 30.4s (gen 25.2s · think≈25.2s · tool 3.0s)
```

While working (live-updating):

```
⏱ 会话跨度 1h02m · 累计活跃 12.4s (3轮) · 本轮已耗时 5m37s · 生成 1.7s · 工具运行中 5m35s
```

### Terminology

| 中文 | English |
|---|---|
| 会话跨度 | span (wall-clock, includes idle) |
| 累计活跃 | active (Σ gen + Σ tool, no idle) |
| 本轮已耗时 | this turn |
| 生成 / 生成中 | gen / generating |
| 思考≈ | think≈ (estimated thinking time) |
| 工具 / 工具运行中 | tool / tool running |
| 上次回复 / 上次生成 | last reply / last gen |

## Time formats

- `<1s` → `500ms`
- `<60s` → `1.5s`
- `<60m` → `1m05s`
- `<24h` → `1h05m`
- `>=24h` → `1d03h`

## How it works

- `message_start` / `message_end` events for generation time (thinking end anchored at the stream's `thinking_end` event)
- `tool_execution_start` / `tool_execution_end` for tool wall-clock (parallel groups counted once)
- A `timing-snapshot` custom entry is appended after each final reply so restarts restore exact values instead of timestamp-delta estimates
- `session_start` / `session_tree` recompute history from the session JSONL (ancestor chain for `/tree` branches)
- `input` event detects the language of your messages (CJK ratio) and switches the widget language; `Intl` system locale is the initial value

## Development

Repository layout:

```
pi-timing/
├── package.json              # pi-package manifest (pi install)
├── extensions/timing/index.ts  # the extension
└── README.md / README.zh-CN.md
```

Local development with a symlink (edit → `/reload` → immediate effect):

```bash
ln -s ~/.pi/dev/pi-timing/extensions/timing ~/.pi/agent/extensions/timing
```

If you installed the package via `pi install`, remove it first to avoid double-loading:

```bash
pi remove git:github.com/adamcjm/pi-timing
```

## License

MIT
