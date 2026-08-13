# pi-timing

> [中文文档 (Chinese)](README.zh-CN.md)

<p align="center">
  <img src="https://raw.githubusercontent.com/adamcjm/pi-timing/main/docs/demo.gif" alt="pi-timing demo" width="720">
</p>

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
pi install npm:pi-timing
```

Or copy `extensions/timing` into `~/.pi/agent/extensions/` (or `.pi/extensions/` for a project).

After installing, run `/reload` (or restart pi) to activate.

## Usage

Once active, the widget appears above the input editor. Type any message and press Enter — the widget starts updating immediately (live counter while generating, then the final breakdown when the reply finishes).

Widget examples:

```
⏱ 会话跨度 10h31m  ·  累计活跃 36m26s (14轮)  ·  上次回复 30.4s (生成 25.2s · 思考≈25.2s · 工具 3.0s)
⏱ elapsed 10h31m  ·  active time 36m26s (14 turns)  ·  last reply 30.4s (gen 25.2s · think≈25.2s · tool 3.0s)
```

While working (live-updating, Chinese / English):

```
⏱ 会话跨度 1h02m · 累计活跃 12.4s (3轮) · 本轮已耗时 5m37s · 生成 1.7s · 工具运行中 5m35s
⏱ elapsed 1h02m · active time 12.4s (3 turns) · turn 5m37s · gen 1.7s · tool running 5m35s
```

## Slash commands

All commands are typed in the pi input editor and submitted with Enter.

### `/timing` — toggle the widget

```bash
/timing
```

- Run once to hide the widget, run again to show it.
- Timers keep running while hidden — nothing is lost.
- No confirmation is shown; the widget simply disappears/appears.

### `/timing list` — per-turn detail list

```bash
/timing list
```

Shows the last 20 turns, newest first, inside the widget (English / Chinese):

```
#1 gen 575ms think≈9ms tool 9ms
#2 gen 776ms think≈700ms tool 0ms
#1 生成 575ms  思考≈9ms  工具 9ms
#2 生成 776ms  思考≈700ms  工具 0ms
```

Run `/timing list` again to hide the list.

### `/timing lang zh|en|auto` — widget language

```bash
/timing lang zh     # force Chinese
/timing lang en     # force English
/timing lang auto   # back to automatic (default)
```

- `auto` (default): starts from your system locale, then follows the language of your messages (CJK ratio detection).
- In `auto` mode, slash commands and very short messages are ignored to avoid accidental switching.
- A confirmation notification is shown at the bottom of the screen (e.g. `timing language: en`).
- The setting is in-memory: it resets to `auto` on pi restart.

## Terminology

| 中文 | English |
|---|---|
| 会话跨度 | elapsed (wall-clock, includes idle) |
| 累计活跃 | active time (Σ gen + Σ tool, no idle) |
| 本轮已耗时 | turn |
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
pi remove npm:pi-timing
```

## License

MIT
