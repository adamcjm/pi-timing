# pi-timing

> [English Docs](README.md)

<p align="center">
  <img src="https://raw.githubusercontent.com/adamcjm/pi-timing/main/docs/demo.gif" alt="pi-timing demo" width="720">
</p>

[pi](https://pi.dev) 的计时组件 —— 显示每次回复实际花了多长时间，以及你的会话里有多少是真正干活、多少是在空等。

## 功能

- **每次生成耗时** — 每次 LLM 流式的墙钟（思考 + 回复），含思考时间估算（`思考≈`）
- **工具耗时** — 工具执行墙钟，并行工具按组只计一次
- **累计活跃时间** — Σ(生成) + Σ(工具)，不含空闲；通过每轮 `timing-snapshot` 条目跨重启持久化
- **会话跨度** — 首条消息到末条消息的墙钟（含空闲），显示在最前
- **历史重算** — 重启时从会话文件恢复累计；`/tree` 分支包含跳转点之前的历史（祖先链）
- **多语言** — widget 语言自动跟随你的消息语言（中/英），初始取系统 locale；可用 `/timing lang` 强制

## 安装

```bash
pi install npm:pi-timing
```

或者把 `extensions/timing` 拷贝到 `~/.pi/agent/extensions/`（或项目级 `.pi/extensions/`）。

安装后执行 `/reload`（或重启 pi）生效。

## 使用方法

生效后，widget 显示在输入框上方。输入任意消息并回车 —— widget 立即开始工作（生成中实时计时，回复完成后显示最终拆解）。

Widget 示例（中文显示）：

```
⏱ 会话跨度 10h31m  ·  累计活跃 36m26s (14轮)  ·  上次回复 30.4s (生成 25.2s · 思考≈25.2s · 工具 3.0s)
```

Widget 示例（英文显示——消息语言为英文时自动切换，或 `/timing lang en` 强制）：

```
⏱ elapsed 10h31m  ·  active time 36m26s (14 turns)  ·  last reply 30.4s (gen 25.2s · think≈25.2s · tool 3.0s)
```

工作中（实时跳秒，中文显示）：

```
⏱ 会话跨度 1h02m · 累计活跃 12.4s (3轮) · 本轮已耗时 5m37s · 生成 1.7s · 工具运行中 5m35s
```

工作中（实时跳秒，英文显示）：

```
⏱ elapsed 1h02m · active time 12.4s (3 turns) · turn 5m37s · gen 1.7s · tool running 5m35s
```

## 斜杠命令

所有命令在 pi 输入框输入，回车执行。

### `/timing` — 开关 widget

```bash
/timing
```

- 执行一次隐藏 widget，再执行一次显示。
- 隐藏期间计时器照常运行，不会丢失任何数据。
- 无确认提示，widget 直接消失/出现。

### `/timing list` — 逐轮明细列表

```bash
/timing list
```

在 widget 内显示最近 20 轮，最新在前（中文显示）：

```
#1 生成 575ms  思考≈9ms  工具 9ms
#2 生成 776ms  思考≈700ms  工具 0ms
```

英文显示：

```
#1 gen 575ms think≈9ms tool 9ms
#2 gen 776ms think≈700ms tool 0ms
```

再次执行 `/timing list` 隐藏列表。

### `/timing lang zh|en|auto` — widget 语言

```bash
/timing lang zh     # 强制中文
/timing lang en     # 强制英文
/timing lang auto   # 恢复自动（默认）
```

- `auto`（默认）：初始取系统 locale，之后跟随你的消息语言（CJK 占比检测）。
- `auto` 模式下，斜杠命令和过短的消息不会触发切换，避免误切。
- 执行后屏幕底部会显示确认通知（如 `timing language: en`）。
- 设置为内存态：重启 pi 后恢复 `auto`。

## 术语对照

| 中文 | English |
|---|---|
| 会话跨度 | elapsed（墙钟，含空闲） |
| 累计活跃 | active time（Σ生成 + Σ工具，不含空闲） |
| 本轮已耗时 | turn |
| 生成 / 生成中 | gen / generating |
| 思考≈ | think≈（思考时间估算） |
| 工具 / 工具运行中 | tool / tool running |
| 上次回复 / 上次生成 | last reply / last gen |

## 时间格式

- `<1s` → `500ms`
- `<60s` → `1.5s`
- `<60m` → `1m05s`
- `<24h` → `1h05m`
- `>=24h` → `1d03h`

## 实现原理

- `message_start` / `message_end` 事件计算生成耗时（思考结束点锚定流式的 `thinking_end` 事件）
- `tool_execution_start` / `tool_execution_end` 计算工具墙钟（并行组只计一次）
- 每轮最终回复后追加 `timing-snapshot` custom 条目，重启时恢复精确值（而非时间戳差估算）
- `session_start` / `session_tree` 从会话 JSONL 重算历史（`/tree` 分支用祖先链）
- `input` 事件检测消息语言（CJK 占比）自动切换 widget 语言；初始值取系统 locale（`Intl`）

## 开发

仓库结构：

```
pi-timing/
├── package.json              # pi-package 清单（pi install 用）
├── extensions/timing/index.ts  # 扩展本体
└── README.md / README.zh-CN.md
```

本地开发用软链接（改源码 → `/reload` 立即生效）：

```bash
ln -s ~/.pi/dev/pi-timing/extensions/timing ~/.pi/agent/extensions/timing
```

如果之前用 `pi install` 装过包，先移除避免重复加载：

```bash
pi remove npm:pi-timing
```

## 许可证

MIT
