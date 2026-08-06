# pi-timing

Timing widget for [pi](https://pi.dev) — shows how long each turn actually took, and how much of your session was real work vs idle waiting.

## Features

- **Per-turn generation time** — wall-clock of each LLM stream (thinking + reply), with an estimated thinking split (`思考≈`)
- **Tool time** — wall-clock execution; parallel tools counted once
- **Cumulative active time** — Σ(generation) + Σ(tools), excluding idle; survives restarts via per-turn `timing-snapshot` entries
- **Session span** — wall-clock from first to last message (includes idle), shown first
- **History recompute** — restores totals from the session file on restart; `/tree` branches include pre-jump history (ancestor chain)

## Install

```bash
pi install git:github.com/adamcjm/pi-timing
```

Or copy `extensions/timing` into `~/.pi/agent/extensions/` (or `.pi/extensions/` for a project).

## Usage

- `/reload` to activate after installing
- `/timing` — toggle the widget
- `/timing list` — show the last 20 turns with details

## Widget example

Idle:

```
⏱ 会话跨度 10h31m  ·  累计活跃 36m26s (14轮)  ·  上次回复 30.4s (生成 25.2s · 思考≈25.2s · 工具 3.0s)
```

While working (live-updating):

```
⏱ 会话跨度 1h02m · 累计活跃 12.4s (3轮) · 本轮已耗时 5m37s · 生成 1.7s · 工具运行中 5m35s
```

## How it works

- `message_start` / `message_end` events for generation time (thinking end anchored at the stream's `thinking_end` event)
- `tool_execution_start` / `tool_execution_end` for tool wall-clock (parallel groups counted once)
- A `timing-snapshot` custom entry is appended after each final reply so restarts restore exact values instead of timestamp-delta estimates
- `session_start` / `session_tree` recompute history from the session JSONL (ancestor chain for `/tree` branches)

## License

MIT
