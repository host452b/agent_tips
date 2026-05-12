# Quick-to-Use：不计 token，最大性能一键命令

> **前提假设**：token 不要钱、API 不限速；目标是把 agent 的**推理深度、上下文长度、并行度、超时容忍**都开到极限。
> **注意**：部分变量是实验性 / 危险开关。每条命令底下都标了关键 trade-off，**别盲抄**。
> **不在这里调的事**：Claude.ai 订阅自带 1h cache、自动 telemetry——这些跟 agent 性能无关。

---

## 1. Claude Code（环境变量最丰富）

### 1.1 一键命令（Opus 4.7 / 1M context，半自动）

```bash
CLAUDE_CODE_EFFORT_LEVEL=max \
CLAUDE_CODE_MAX_TOOL_USE_CONCURRENCY=20 \
CLAUDE_CODE_MAX_RETRIES=20 \
CLAUDE_CODE_FORK_SUBAGENT=1 \
CLAUDE_AUTO_BACKGROUND_TASKS=1 \
CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1 \
API_TIMEOUT_MS=1800000 \
BASH_DEFAULT_TIMEOUT_MS=600000 \
BASH_MAX_TIMEOUT_MS=1800000 \
BASH_MAX_OUTPUT_LENGTH=200000 \
MAX_MCP_OUTPUT_TOKENS=100000 \
TASK_MAX_OUTPUT_LENGTH=160000 \
CLAUDE_CODE_FILE_READ_MAX_OUTPUT_TOKENS=200000 \
MCP_TIMEOUT=120000 \
MCP_CONNECT_TIMEOUT_MS=15000 \
CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=98 \
ENABLE_PROMPT_CACHING_1H=1 \
FALLBACK_FOR_ALL_PRIMARY_MODELS=1 \
CLAUDE_CODE_NO_FLICKER=1 \
DISABLE_COST_WARNINGS=1 \
DISABLE_AUTOUPDATER=1 \
claude --model 'claude-opus-4-7[1m]'
```

**逐行作用速查**（每条对应上面命令里的一行）：

- `CLAUDE_CODE_EFFORT_LEVEL=max` — **推理 effort 拉满**。用 env 设的 `max` 会跨 session 持久化（在 `/effort` 里切到 `max` 只是本次 session）。
- `CLAUDE_CODE_MAX_TOOL_USE_CONCURRENCY=20` — **并行只读工具与子 agent 数 10 → 20**，让 Glob/Read/Grep 类工具一次性多开几路。
- `CLAUDE_CODE_MAX_RETRIES=20` — **API 失败重试次数 10 → 20**，应对偶发 5xx 与限流。
- `CLAUDE_CODE_FORK_SUBAGENT=1` — **启用 fork 子 agent**：`/fork` 真正派生一个继承当前完整上下文的 subagent（默认仅别名到 `/branch`）。
- `CLAUDE_AUTO_BACKGROUND_TASKS=1` — **长任务（~2 分钟以上）自动后台化**。注意官方拼写**没有 `_CODE_` 中缀**——很多旧文档写错。
- `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` — **启用 agent teams（实验性）**，多 agent 协同。
- `API_TIMEOUT_MS=1800000` — **单次 API 请求超时 10 分钟 → 30 分钟**，给深度思考留时间。
- `BASH_DEFAULT_TIMEOUT_MS=600000` — **bash 命令默认超时 2 分钟 → 10 分钟**。
- `BASH_MAX_TIMEOUT_MS=1800000` — **bash 命令最大超时 10 分钟 → 30 分钟**（模型能主动请求的上限）。
- `BASH_MAX_OUTPUT_LENGTH=200000` — **bash 输出超 20 万字符才落盘**，否则全文塞进上下文给模型看。
- `MAX_MCP_OUTPUT_TOKENS=100000` — **MCP 工具响应 token 上限 25K → 100K**（官方在 10K 以上就开始警告，但我们不在意 token）。
- `TASK_MAX_OUTPUT_LENGTH=160000` — **子 agent 输出字符上限拉到文档最大值**（默认 32K，最大 160K）。
- `CLAUDE_CODE_FILE_READ_MAX_OUTPUT_TOKENS=200000` — **Read 工具的 token 上限**，能一次性读完大文件不被截断。
- `MCP_TIMEOUT=120000` — **MCP server 启动超时 30 秒 → 2 分钟**，给慢启动的 server 留余地。
- `MCP_CONNECT_TIMEOUT_MS=15000` — **首次查询前等 MCP 连接的时间 5 秒 → 15 秒**，确保慢的 MCP server 也能加进首批工具列表。
- `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=98` — **自动压缩触发点从 ~95% 推到 98%**，让上下文更晚被压。
- `ENABLE_PROMPT_CACHING_1H=1` — **prompt cache TTL 从 5 分钟拉到 1 小时**，跨工具调用之间缓存复用率更高（写入费率会更高，我们不在意）。
- `FALLBACK_FOR_ALL_PRIMARY_MODELS=1` — **任何主力模型 overload 后都触发 `--fallback-model`**（默认只 Opus 触发），减少卡顿。
- `CLAUDE_CODE_NO_FLICKER=1` — **fullscreen renderer**，长会话减少闪烁、内存增长曲线更平。
- `DISABLE_COST_WARNINGS=1` — **关掉成本告警弹窗**，因为我们不在意。
- `DISABLE_AUTOUPDATER=1` — **session 内不去查更新**，避免长跑期间被插件/CLI 自动更新打断。
- `claude --model 'claude-opus-4-7[1m]'` — **显式选 Opus 4.7 的 1M 上下文变体**（不加 `[1m]` 后缀默认是 200K 窗口）。

> 想看每条的 trade-off / 不开它的影响，看下面 §1.4。

### 1.2 一键命令（**全自动 / 无人值守 / 危险**）

在上一条命令前再叠加权限绕过，**只在沙箱、CI、容器内用**：

```bash
… 上面那一长串 … \
claude --model 'claude-opus-4-7[1m]' --dangerously-skip-permissions
```

`--dangerously-skip-permissions` 让 Claude Code 不再就工具调用问你，搭配大并发 + 大超时 + max effort 可以达到「贴脸输出」级吞吐。代价：写错文件、改坏系统的风险全部由你承担。

### 1.3 如果跑 Opus 4.6 / Sonnet 4.6（adaptive thinking 可关）

4.7 永远 adaptive 思考，预算由模型自定；**4.6 系**才能用固定预算。给 4.6 配套追加：

```bash
CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING=1 \
MAX_THINKING_TOKENS=128000 \
… 其他和 1.1 一样 … \
claude --model 'claude-opus-4-6[1m]'
```

注意 `MAX_THINKING_TOKENS` 上限是模型 max output tokens − 1，128K 足够吃满。

### 1.4 每条变量做什么、为什么

| 变量 | 作用 | trade-off |
|---|---|---|
| `CLAUDE_CODE_EFFORT_LEVEL=max` | 推理 effort 拉满；通过 env 设的 `max` 会跨 session 持久化 | 单 turn 延迟和 token 都飙 |
| `CLAUDE_CODE_MAX_TOOL_USE_CONCURRENCY=20` | 只读工具与子 agent 并行数从 10→20 | 容易撞 MCP server 限流 |
| `CLAUDE_CODE_MAX_RETRIES=20` | API 失败重试从 10→20 | 真挂的时候要等更久才报错 |
| `CLAUDE_CODE_FORK_SUBAGENT=1` | `/fork` 启用上下文继承的子 agent | 多并发 fork 烧大量 token |
| `CLAUDE_AUTO_BACKGROUND_TASKS=1` | 长任务自动后台化（注意**不是 `CLAUDE_CODE_AUTO_*`**） | 后台任务多了状态难追 |
| `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` | 启用 agent teams | 实验性；接口可能变 |
| `API_TIMEOUT_MS=1800000` | 单次 API 请求 30 分钟超时 | 真有问题会拖很久 |
| `BASH_DEFAULT_TIMEOUT_MS=600000` / `BASH_MAX_TIMEOUT_MS=1800000` | bash 默认 10 分钟、上限 30 分钟 | 误入死循环会等到上限才回来 |
| `BASH_MAX_OUTPUT_LENGTH=200000` | bash 输出超 20 万字符再落盘 | 大 log 直接灌进上下文 |
| `MAX_MCP_OUTPUT_TOKENS=100000` | MCP 工具响应上限拉到 10 万 token（默认 25K，文档在 10K 以上就警告） | 一个工具就能塞满半个上下文 |
| `TASK_MAX_OUTPUT_LENGTH=160000` | 子 agent 输出上限拉到文档最大值 | 子 agent 调用一次就是一大块上下文 |
| `CLAUDE_CODE_FILE_READ_MAX_OUTPUT_TOKENS=200000` | 文件读取 token 上限 | 大文件会一次性进 context |
| `MCP_TIMEOUT=120000` | MCP server 启动超时 2 分钟 | 启动慢时不会假报错 |
| `MCP_CONNECT_TIMEOUT_MS=15000` | 首次查询前等 MCP 连接更久 | 启动慢 3 倍 |
| `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=98` | 上下文到 98% 才触发自动压缩（默认 ~95） | 越晚压缩越容易撞墙 |
| `ENABLE_PROMPT_CACHING_1H=1` | cache TTL 5 分钟→1 小时 | 1h cache **写入费率更高**——题目说不在意，所以开 |
| `FALLBACK_FOR_ALL_PRIMARY_MODELS=1` | 任何主力模型 overload 都触发 fallback | 偶尔会切到次优模型 |
| `CLAUDE_CODE_NO_FLICKER=1` | fullscreen renderer，长会话内存平 | 与某些终端不兼容（不行就换 `CLAUDE_CODE_DISABLE_ALTERNATE_SCREEN=1`） |
| `DISABLE_COST_WARNINGS=1` | 不弹成本告警 | 主动放弃成本可见性 |
| `DISABLE_AUTOUPDATER=1` | session 内不查更新 | 长跑期间不会被插件刷新打断 |
| `--model 'claude-opus-4-7[1m]'` | 显式用 1M context Opus 4.7 | 1M 上下文 token 费率更高 |

### 1.5 故意**没开**的那些

| 不开的变量 | 原因 |
|---|---|
| `DISABLE_COMPACT=1` | 全关压缩；上下文到顶就直接 fail，不值得 |
| `CLAUDE_CODE_MAX_OUTPUT_TOKENS` 大调 | 增大 = 减少自动压缩前的有效上下文，反而吃亏 |
| `CLAUDE_CODE_SIMPLE` / `CLAUDE_CODE_SIMPLE_SYSTEM_PROMPT` | 这俩是「精简」开关，会减少 agent 能力 |
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY` | 关 auto memory = 失去跨 session 记忆 |
| `CLAUDE_CODE_DISABLE_CLAUDE_MDS` | 关 CLAUDE.md = 失去项目级 instruction |
| `CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS` | 关 git workflow 指令 = 写 commit 质量下降 |
| `DISABLE_PROMPT_CACHING*` | 关缓存反而每次都重算，纯负贡献 |
| `DISABLE_TELEMETRY` 之类 | 跟性能无关 |

---

## 2. OpenAI Codex CLI（env 面窄，性能靠 `-c` flag）

Codex 的「性能」开关绝大部分是 `~/.codex/config.toml` 字段，可以临时用 `-c key=value` 注入。**真正的 env 变量基本只能做 CA / 日志 / 鉴权**。

### 2.1 一键命令（半自动，最大推理）

```bash
RUST_LOG=info \
OPENAI_API_KEY="$OPENAI_API_KEY" \
codex \
  -c model_reasoning_effort=xhigh \
  -c model_reasoning_summary=detailed \
  -c model_verbosity=high \
  -c approval_policy=on-request \
  -c sandbox_mode=workspace-write \
  -c sandbox_workspace_write.network_access=true \
  -c shell_environment_policy.inherit=all
```

### 2.2 一键命令（**全自动 / 无人值守 / 危险**）

```bash
RUST_LOG=warn \
OPENAI_API_KEY="$OPENAI_API_KEY" \
codex \
  -c model_reasoning_effort=xhigh \
  -c model_reasoning_summary=detailed \
  -c model_verbosity=high \
  -c approval_policy=never \
  -c sandbox_mode=danger-full-access \
  -c shell_environment_policy.inherit=all
```

`approval_policy=never` 让 Codex 不再就单步动作问你；`sandbox_mode=danger-full-access` 等于关掉沙箱。**只在容器/CTF/能整体回滚的环境里跑**。

### 2.3 每条 flag 做什么

| `-c` 字段 | 作用 | trade-off |
|---|---|---|
| `model_reasoning_effort=xhigh` | Codex 推理 effort 顶档；**没有 `max`** | 单 turn 延迟显著上升 |
| `model_reasoning_summary=detailed` | 推理摘要详尽 | 输出更长 |
| `model_verbosity=high` | 输出 verbosity 拉满 | 多废话 |
| `approval_policy=on-request` | Codex 主动请求时才拦 | 比 `never` 安全 |
| `approval_policy=never` | 完全不拦 | **危险** |
| `sandbox_mode=workspace-write` | 沙箱允许写工作区 | 出 workspace 仍受限 |
| `sandbox_mode=danger-full-access` | 沙箱完全开放 | **危险** |
| `sandbox_workspace_write.network_access=true` | workspace-write 沙箱允许出网 | 沙箱形同虚设 |
| `shell_environment_policy.inherit=all` | 子进程继承全部宿主 env | 工具能拿到你的全部 secret |

### 2.4 真正的 env 部分

| 变量 | 作用 |
|---|---|
| `OPENAI_API_KEY` | API key 鉴权（除非用 ChatGPT 登录） |
| `RUST_LOG` | 日志级别（`info` / `debug` / `codex_core=debug,codex_tui=warn`） |
| `CODEX_CA_CERTIFICATE` | 企业 TLS 代理 CA bundle（性能不相关，但常忘） |
| `CODEX_HOME` | 改配置/凭证/session/日志根目录 |

### 2.5 选模型

Codex 用 `model` 字段或 `-c model=…`：

```bash
codex -c model=gpt-5.5 -c model_reasoning_effort=xhigh ...
```

模型名以你 `~/.codex/config.toml` 里 `model_providers` 段下的实际可用列表为准——这部分 Codex 不会自动透露。

### 2.6 持久化到 `~/.codex/config.toml`

如果你每次都要这套配置，直接写进 TOML，命令行就只剩 `codex` 一个词：

```toml
model_reasoning_effort = "xhigh"
model_reasoning_summary = "detailed"
model_verbosity = "high"
approval_policy = "on-request"
sandbox_mode = "workspace-write"

[sandbox_workspace_write]
network_access = true

[shell_environment_policy]
inherit = "all"
```

---

## 3. cursor-agent CLI（**实话实说，没什么可调**）

cursor-agent 没有面向「性能」的环境变量。`CURSOR_API_KEY` 是鉴权用的，其他 4 个是代理/CA，没有 `effort`、没有 `model`、没有 `concurrency` 这种环境变量。要拿最大性能，只能靠 **flag** 和 `cli-config.json`。

### 3.1 一键命令

```bash
CURSOR_API_KEY="$CURSOR_API_KEY" \
cursor-agent \
  --model <你账号能拿到的最强模型> \
  --print "<你的任务>"
```

`--print` 是 headless / 非交互。模型名取决于你 Cursor 账号的可用列表（GPT-5 / Claude Opus / Sonnet 都可能在）。

### 3.2 想压榨更多

| 来源 | 做法 |
|---|---|
| `cli-config.json` | 调默认模型、默认 system prompt；Cursor 文档列了字段 |
| `--api-key <key>` | 等价 `CURSOR_API_KEY`，可在脚本里临时换号 |
| GitHub Actions | 把 `CURSOR_API_KEY` 灌进去就行 |

### 3.3 不要试图

| 假命名 | 现实 |
|---|---|
| `CURSOR_MODEL=…` | **不存在**，用 `--model` flag |
| `CURSOR_EFFORT=…` | **不存在** |
| `CURSOR_MAX_CONCURRENCY=…` | **不存在** |
| `CURSOR_DEBUG=1` | **不存在**，官方文档无 debug env |
| `CURSOR_BASE_URL=…` | **不存在** |

如果你看到第三方教程写这些，那是从别的 CLI 类比出来的，**别照搬**。

---

## 4. 三者对比（不计成本、求性能极限）

| 维度 | Claude Code | Codex CLI | cursor-agent CLI |
|---|---|---|---|
| 性能调节面 | env 变量 + flag | `-c` flag + TOML（env 很少） | 几乎只有 flag |
| effort / reasoning 档位 | `low/medium/high/xhigh/max/auto` | `minimal/low/medium/high/xhigh` | 无 |
| thinking 预算手动控制 | 4.6 系：`MAX_THINKING_TOKENS` + `…DISABLE_ADAPTIVE_THINKING=1`；**4.7 系：不可控** | 由 `model_reasoning_effort` 推算，不能直接指定 token | 无 |
| 上下文窗口 | 模型 + `[1m]` 后缀 / `CLAUDE_CODE_MAX_CONTEXT_TOKENS`（依赖 `DISABLE_COMPACT`） | 由模型决定 | 由模型决定 |
| 并行度调节 | `CLAUDE_CODE_MAX_TOOL_USE_CONCURRENCY` | 无对应字段 | 无 |
| 工具/Bash 超时 | `BASH_*_TIMEOUT_MS` / `MCP_*` 一套 | 由 TOML 的 sandbox 段控制 | 无 |
| 无人值守开关 | `--dangerously-skip-permissions` | `approval_policy=never` + `sandbox_mode=danger-full-access` | （Cursor agent 本身在沙箱中跑，无 CLI 层等价开关） |
| 缓存 TTL | `ENABLE_PROMPT_CACHING_1H=1` | 不可见 | 不可见 |
| 后台并发 | `CLAUDE_AUTO_BACKGROUND_TASKS=1` + `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` | 无 | 无 |

---

## 5. 一句话总结

- **Claude Code 是「性能旋钮」最多的**——env 变量就能改 effort、并行度、超时、缓存、后台、agent teams。
- **Codex 把性能旋钮做成 TOML / `-c` flag**——env 那一面基本只够鉴权和日志。
- **cursor-agent 几乎没旋钮**——能调的就是 `--model` 和 `cli-config.json`，其他靠 Cursor 服务端的默认策略。

如果你的目标是「同一份配置无脑灌进环境就吃满性能」，**Claude Code 是唯一支持这种用法的**。Codex 至少要写一份 `config.toml`，cursor-agent 则需要每条命令带 flag。
