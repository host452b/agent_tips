# OpenAI Codex CLI 环境变量参考

> **数据来源**：[`developers.openai.com/codex/*`](https://developers.openai.com/codex/) 系列官方文档，外加 [`github.com/openai/codex`](https://github.com/openai/codex) 仓库源码（特别是 `codex-rs/` 子目录）逐文件验证。
> **采集日期**：2026-05-12。
> **覆盖版本**：`@openai/codex` Rust 重写线，最新 release `rust-v0.130.0`（2026-05）。
> **核心提醒**：**Codex 不沿用 `CLAUDE_CODE_*` → `CODEX_*` 的命名映射**。配置主要走 `~/.codex/config.toml`、profile、或 `-c key=value` flag；真正的「环境变量」面集合很小，大部分以 `CODEX_*` 开头的标识符是**仓库内部常量**（沙箱握手、telemetry、app-server 托管配置），不是给终端用户调的开关。

---

## 1. 数据来源

| URL / 路径 | 内容 |
|---|---|
| [`developers.openai.com/codex/config-reference`](https://developers.openai.com/codex/config-reference) | `config.toml` 的权威字段表 |
| [`developers.openai.com/codex/config-basic`](https://developers.openai.com/codex/config-basic) | 入门配置（无 env 列表） |
| [`developers.openai.com/codex/config-advanced`](https://developers.openai.com/codex/config-advanced) | `shell_environment_policy`、自定义 provider、OTEL |
| [`developers.openai.com/codex/auth`](https://developers.openai.com/codex/auth) | 鉴权：`CODEX_HOME`、`CODEX_CA_CERTIFICATE`、`SSL_CERT_FILE`、`forced_login_method` |
| [`developers.openai.com/codex/cli`](https://developers.openai.com/codex/cli) | CLI 概览（env 列表稀疏） |
| [`developers.openai.com/codex/cli/reference`](https://developers.openai.com/codex/cli/reference) | CLI flag 参考；提到 `OPENAI_API_KEY`、`CODEX_HOME`、`RUST_LOG` |
| [`developers.openai.com/codex/agent-approvals-security`](https://developers.openai.com/codex/agent-approvals-security) | 沙箱与审批策略（来自 config/flag，不是 env） |
| `github.com/openai/codex/blob/main/docs/install.md` | 文档化 `RUST_LOG` |
| `github.com/openai/codex/blob/main/AGENTS.md` | 文档化 `CODEX_SANDBOX` / `CODEX_SANDBOX_NETWORK_DISABLED` |
| `codex-rs/login/src/auth/manager.rs` | 鉴权常量：`OPENAI_API_KEY_ENV_VAR`、`CODEX_API_KEY_ENV_VAR`、`CODEX_ACCESS_TOKEN_ENV_VAR`、`REFRESH_TOKEN_URL_OVERRIDE_ENV_VAR`、`REVOKE_TOKEN_URL_OVERRIDE_ENV_VAR` |
| `codex-rs/login/src/auth/default_client.rs` | `CODEX_INTERNAL_ORIGINATOR_OVERRIDE`、`CODEX_SANDBOX` |
| `codex-rs/core/src/spawn.rs` | `CODEX_SANDBOX_NETWORK_DISABLED`、`CODEX_SANDBOX` |
| `codex-rs/codex-client/src/custom_ca.rs` | `CODEX_CA_CERTIFICATE`、`SSL_CERT_FILE` |
| `codex-rs/core/src/config/mod.rs` | `CODEX_HOME`、`CODEX_OSS_PORT`、`CODEX_OSS_BASE_URL` |
| `codex-rs/utils/home-dir/src/lib.rs` | `CODEX_HOME` 定义 |
| `codex-rs/app-server/src/config_manager.rs` | `CODEX_APP_SERVER_MANAGED_CONFIG_PATH`、`CODEX_APP_SERVER_DISABLE_MANAGED_CONFIG` |
| `codex-rs/tui/src/session_log.rs` | `CODEX_TUI_RECORD_SESSION`、`CODEX_TUI_SESSION_LOG_PATH` |
| `codex-rs/tui/src/tui/keyboard_modes.rs` | `CODEX_TUI_DISABLE_KEYBOARD_ENHANCEMENT` |
| `codex-rs/codex-mcp/src/mcp/mod.rs` | `CODEX_CONNECTORS_TOKEN` |
| `codex-rs/exec-server/src/remote.rs` | `CODEX_EXEC_SERVER_REMOTE_BEARER_TOKEN` |
| `codex-rs/core/src/arc_monitor.rs` | `CODEX_ARC_MONITOR_ENDPOINT_OVERRIDE`、`CODEX_ARC_MONITOR_TOKEN` |
| `codex-rs/scripts/start-codex-exec.sh` | `CODEX_REMOTE_EXEC_SERVER_LOCAL_PORT` 等脚本变量 |
| `codex-rs/arg0/src/lib.rs` | 加载 `~/.codex/.env`，**显式拒绝** `CODEX_*` 前缀的 key |
| `codex-cli/bin/codex.js` | npm 包装器注入 `CODEX_MANAGED_BY_NPM` / `CODEX_MANAGED_BY_BUN` |
| `scripts/install/install.sh` | `CODEX_INSTALL_DIR` |

---

## 2. 速查总表

| 变量 | 默认值 | 类别 | 状态 | 适用范围 |
|---|---|---|---|---|
| `OPENAI_API_KEY` | — | 鉴权 | documented | 用 API key 登录时必填 |
| `CODEX_API_KEY` | — | 鉴权 | inferred-from-code | 只有 `codex_api_key_env_enabled=true` 才被读 |
| `CODEX_ACCESS_TOKEN` | — | 鉴权 | inferred-from-code | agent-identity 模式 |
| `CODEX_HOME` | `~/.codex` | 配置 | documented | 改配置/凭证/session/日志根目录 |
| `CODEX_CA_CERTIFICATE` | — | 鉴权 | documented | 企业代理 mTLS CA bundle |
| `SSL_CERT_FILE` | — | 鉴权 | documented | 上一项的 fallback |
| `CODEX_REFRESH_TOKEN_URL_OVERRIDE` | — | 鉴权 | inferred-from-code | ChatGPT 登录 token 刷新端点 |
| `CODEX_REVOKE_TOKEN_URL_OVERRIDE` | — | 鉴权 | inferred-from-code | ChatGPT 登录 token 注销端点 |
| `CODEX_SANDBOX` | unset | 沙箱握手 | documented (AGENTS.md) | **Codex 写给子进程**，不是用户设 |
| `CODEX_SANDBOX_NETWORK_DISABLED` | unset | 沙箱握手 | documented (AGENTS.md) | **Codex 写给子进程** |
| `BAZEL_BWRAP` | — | 沙箱辅助 | inferred-from-code | Bazel 下 `bwrap` 路径 |
| `RUST_LOG` | TUI: `codex_core=info,codex_tui=info,codex_rmcp_client=info`；`codex exec`: `error` | 日志 | documented | 标准 tracing-subscriber 过滤器 |
| `CODEX_TUI_RECORD_SESSION` | unset | 日志 | inferred-from-code | TUI 会话记录开关 |
| `CODEX_TUI_SESSION_LOG_PATH` | `$CODEX_HOME/log/...` | 日志 | inferred-from-code | TUI 会话日志路径 |
| `CODEX_TUI_DISABLE_KEYBOARD_ENHANCEMENT` | unset | TUI | inferred-from-code | 关掉 crossterm 键盘增强 |
| `CODEX_OSS_PORT` | 自动发现 | OSS Provider | inferred-from-code | 本地 LM Studio/Ollama 端口覆盖 |
| `CODEX_OSS_BASE_URL` | 由端口推导 | OSS Provider | inferred-from-code | 本地 OSS provider base URL |
| `CODEX_APP_SERVER_MANAGED_CONFIG_PATH` | — | app server | inferred-from-code | 组织托管配置路径 |
| `CODEX_APP_SERVER_DISABLE_MANAGED_CONFIG` | — | app server | inferred-from-code | 关掉托管配置 |
| `CODEX_EXEC_SERVER_REMOTE_BEARER_TOKEN` | — | exec server | inferred-from-code | 远端 codex-exec 注册 bearer |
| `CODEX_EXEC_SERVER_URL` | — | exec server | inferred-from-code (脚本) | 客户端指向已有 exec server |
| `CODEX_REMOTE_EXEC_SERVER_LOCAL_PORT` | `8765` | exec server 脚本 | inferred-from-code (脚本) | 启动脚本本地端口 |
| `CODEX_REMOTE_EXEC_SERVER_START_TIMEOUT_SECONDS` | `15` | exec server 脚本 | inferred-from-code (脚本) | 启动脚本超时 |
| `CODEX_CONNECTORS_TOKEN` | — | Connectors | inferred-from-code | Codex Apps/Connectors MCP 鉴权 |
| `CODEX_INTERNAL_ORIGINATOR_OVERRIDE` | `codex_cli_rs` | telemetry | inferred-from-code | 嵌入到其他产品时改 user-agent / OTEL originator |
| `CODEX_ARC_MONITOR_ENDPOINT_OVERRIDE` | — | telemetry | inferred-from-code | 内部 ARC monitor 端点 |
| `CODEX_ARC_MONITOR_TOKEN` | — | telemetry | inferred-from-code | 内部 ARC monitor token |
| `CODEX_MANAGED_BY_NPM` | unset | 安装包装 | inferred-from-code | npm 包装器自动设为 `1`，用于安装渠道 telemetry |
| `CODEX_MANAGED_BY_BUN` | unset | 安装包装 | inferred-from-code | bun 检测时自动设 |
| `CODEX_INSTALL_DIR` | `$HOME/.local/bin` | 安装脚本 | inferred-from-code | `scripts/install/install.sh` 安装目标 |

### 2.1 OS / 第三方变量（被 Codex 读取，但不是 Codex 专属）

| 变量 | 影响 |
|---|---|
| `EDITOR` / `VISUAL` | 外部编辑器选择（`VISUAL` 优先） |
| `SHELL` / `COMSPEC` | shell escalation 客户端使用的默认 shell |
| `TMPDIR` | `sandbox_workspace_write.exclude_tmpdir_env_var=true` 时从 writable roots 排除 |
| `NO_COLOR` | debug-client 输出抑制 ANSI 色 |
| `WSL_DISTRO_NAME` / `WSL_INTEROP` | 启用 WSL 特定代码分支 |
| `TERM_PROGRAM` / `TMUX` / `TMUX_PANE` | 终端/剪贴板启发判断 |
| `OTEL_EXPORTER_OTLP_TIMEOUT` | OTEL feature 下 OTLP exporter 标准超时 |
| `HTTP_PROXY` / `HTTPS_PROXY` / `NO_PROXY` | 标准代理变量 |

---

## 3. 真正面向用户的环境变量（详细说明）

### 3.1 鉴权

#### `OPENAI_API_KEY`
- **默认**：无。
- **效果**：当 `model_provider = "openai"`（默认）或任何 `env_key = "OPENAI_API_KEY"` 的 provider 用 API-key 登录时必填。
- **配置等价**：可通过 `~/.codex/config.toml` 的 `model_providers.openai.env_key` 改名（**变量名本身是可配置的**），但默认就叫 `OPENAI_API_KEY`。
- **来源**：`config-reference`、`codex-rs/login/src/auth/manager.rs#L465`。

#### `CODEX_API_KEY`
- **默认**：无。
- **效果**：备选 API-key 名；只有 `codex_api_key_env_enabled=true` 时才被读。
- **状态**：源码中是稳定常量 `CODEX_API_KEY_ENV_VAR`，但官网未明文文档。
- **来源**：`codex-rs/login/src/auth/manager.rs#L466,#L731`。

#### `CODEX_ACCESS_TOKEN`
- **默认**：无。
- **效果**：set 后 Codex 切到 agent-identity auth，不再走 ChatGPT/API-key 路径。
- **状态**：源码常量；面向特定企业场景，官网无明文。
- **来源**：`codex-rs/login/src/auth/manager.rs#L467,#L757`。

#### `CODEX_HOME`
- **默认**：`~/.codex`。
- **效果**：Codex 的「家」——`auth.json`、`config.toml`、session 文件、日志都在这里。**改它等同于改整套配置位置**。
- **注意**：set 之后该目录必须存在。
- **来源**：`developers.openai.com/codex/auth`、`config-reference`、`codex-rs/utils/home-dir/src/lib.rs`。

#### `CODEX_CA_CERTIFICATE`
- **默认**：无。
- **效果**：额外 PEM CA bundle 路径，用于企业 TLS 代理。覆盖所有 HTTPS/WSS 请求，**包括登录流程**。
- **来源**：`developers.openai.com/codex/auth`、`codex-rs/codex-client/src/custom_ca.rs#L61`。

#### `SSL_CERT_FILE`
- **默认**：无。
- **效果**：上一项的 fallback——`CODEX_CA_CERTIFICATE` 不设时，Codex 用它作 CA bundle。
- **来源**：`developers.openai.com/codex/auth`、`codex-rs/codex-client/src/custom_ca.rs#L62`。

#### `CODEX_REFRESH_TOKEN_URL_OVERRIDE` / `CODEX_REVOKE_TOKEN_URL_OVERRIDE`
- ChatGPT 登录的 token 刷新 / 注销端点 URL 覆盖。
- 状态：源码常量（`REFRESH_TOKEN_URL_OVERRIDE_ENV_VAR` / `REVOKE_TOKEN_URL_OVERRIDE_ENV_VAR`），主要用于内部/特殊部署场景。

### 3.2 沙箱握手（不是用户设的开关）

> **AGENTS.md 明确指出贡献者不得修改这两个变量的行为**——它们是 Codex **写给被沙箱化的子进程**的标记，方便下游工具检测自己在沙箱里。

#### `CODEX_SANDBOX`
- macOS 上 Codex 通过 Seatbelt spawn 子进程时设为 `seatbelt`；子代码可据此分支。
- Codex 自身**不读它来改变自己的行为**——唯一消费者是给下游工具用的 `is_sandboxed()` helper。

#### `CODEX_SANDBOX_NETWORK_DISABLED`
- 当 Codex 阻断子进程的网络时设为 `1`。被 Codex 自家的 shell 工具 / 沙箱 helper 用来检测拒绝。
- **不是给用户手动设的**——设了也不会改变 Codex 自己的网络策略，那个由 `sandbox_workspace_write.network_access = false` 控制。

### 3.3 OSS Provider（本地模型）

#### `CODEX_OSS_PORT`
- 覆盖本地 OSS provider（LM Studio / Ollama）发现的端口。
- 来源：`codex-rs/core/src/config/mod.rs#L479`。

#### `CODEX_OSS_BASE_URL`
- 覆盖 OSS provider 的 base URL；不设时由端口推导。
- 来源：`codex-rs/core/src/config/mod.rs#L486`。

### 3.4 日志 / 追踪

#### `RUST_LOG`
- 默认：TUI 用 `codex_core=info,codex_tui=info,codex_rmcp_client=info`；`codex exec` 用 `error`。
- 效果：标准 `tracing-subscriber` 过滤指令；可逐 target 设级别。
- 想换日志**位置**用 `config.toml` 的 `log_dir`，跟 `RUST_LOG` 各管一面。
- 来源：`github.com/openai/codex/blob/main/docs/install.md`。

#### `CODEX_TUI_RECORD_SESSION` / `CODEX_TUI_SESSION_LOG_PATH`
- 启用 TUI 会话日志，及覆盖输出路径（默认在 `$CODEX_HOME/log/...`）。
- 来源：`codex-rs/tui/src/session_log.rs`。

#### `CODEX_TUI_DISABLE_KEYBOARD_ENHANCEMENT`
- 关掉 crossterm 的键盘增强 flag，规避部分终端兼容性问题。
- 来源：`codex-rs/tui/src/tui/keyboard_modes.rs#L16`。

### 3.5 app server / exec server / 远程执行

只有用 `codex-app-server`、远程模式的 `codex-exec`，或 `scripts/start-codex-exec.sh` 才相关：

- `CODEX_APP_SERVER_MANAGED_CONFIG_PATH`：组织托管的配置文件路径。
- `CODEX_APP_SERVER_DISABLE_MANAGED_CONFIG`：禁用托管配置。
- `CODEX_EXEC_SERVER_REMOTE_BEARER_TOKEN`：远端 codex-exec server 的注册 bearer。
- `CODEX_EXEC_SERVER_URL`：客户端指向已有的远端 exec server。
- `CODEX_REMOTE_EXEC_SERVER_LOCAL_PORT`（默认 `8765`）、`CODEX_REMOTE_EXEC_SERVER_START_TIMEOUT_SECONDS`（默认 `15`）：脚本变量。

### 3.6 Codex Apps / Connectors

#### `CODEX_CONNECTORS_TOKEN`
- 给 Codex Apps/Connectors MCP server（实验性 `features.apps`）的 bearer。
- 来源：`codex-rs/codex-mcp/src/mcp/mod.rs#L47,#L394`。

### 3.7 Telemetry / 内部

| 变量 | 默认 | 作用 |
|---|---|---|
| `CODEX_INTERNAL_ORIGINATOR_OVERRIDE` | `codex_cli_rs` | 把 user-agent 和 OTEL 里的 originator 改名；只用于 Codex 被嵌入其他 OpenAI 产品时 |
| `CODEX_ARC_MONITOR_ENDPOINT_OVERRIDE` | — | 内部 ARC monitor 服务端点 |
| `CODEX_ARC_MONITOR_TOKEN` | — | 内部 ARC monitor token |

### 3.8 安装包装变量

| 变量 | 默认 | 作用 |
|---|---|---|
| `CODEX_MANAGED_BY_NPM` | unset | npm 包装器启动 binary 时自动设为 `1`，用于安装渠道 telemetry |
| `CODEX_MANAGED_BY_BUN` | unset | bun 包管理器场景自动设 |
| `CODEX_INSTALL_DIR` | `$HOME/.local/bin` | 安装脚本 `scripts/install/install.sh` 放 binary 的目录 |

---

## 4. 容易被误以为是环境变量的 `config.toml` 字段

这是本文件**最重要的一节**——很多第三方清单（包括早期 AI 生成的对比表）把这些 TOML 字段写成了 `CODEX_*` 环境变量名，全是假的。

| TOML key | 类别 | 作用 |
|---|---|---|
| `model` | 顶层 | 当前模型 id（`gpt-5.5` 等） |
| `model_provider` | 顶层 | provider id（默认 `openai`） |
| `model_reasoning_effort` | 顶层 | `minimal | low | medium | high | xhigh`，**没有 `max`** |
| `model_reasoning_summary` | 顶层 | `auto / concise / detailed / none` |
| `model_verbosity` | 顶层 | `low / medium / high` |
| `approval_policy` | 顶层 | `untrusted / on-request / never / granular` |
| `sandbox_mode` | 顶层 | `read-only / workspace-write / danger-full-access` |
| `sandbox_workspace_write.network_access` | 表 | workspace-write 沙箱里是否允许出网 |
| `sandbox_workspace_write.exclude_tmpdir_env_var` | 表 | 是否把 `$TMPDIR` 从可写根目录排除 |
| `shell_environment_policy.inherit` | 表 | `all / core / none`，Codex 传给子进程的 env 范围 |
| `shell_environment_policy.set` | 表 | 给子进程显式注入 env |
| `openai_base_url` | 顶层 | 内置 openai provider 的 base URL——**Codex 不读 `OPENAI_BASE_URL` 环境变量！**要走代理用这个 TOML 字段或 `-c openai_base_url=...` |
| `model_providers.<id>.base_url` | 表 | 自定义 provider 的 base URL |
| `model_providers.<id>.env_key` | 表 | 该 provider 用哪个**环境变量名**取 API key（默认 `OPENAI_API_KEY`） |
| `model_providers.<id>.env_http_headers` | 表 | header → 环境变量名 的映射 |
| `mcp_servers.<id>.env` | 表 | 给 stdio MCP server 的静态 env |
| `mcp_servers.<id>.env_vars` | 表 | 转发给 stdio MCP server 的 env 白名单 |
| `mcp_servers.<id>.bearer_token_env_var` | 表 | HTTP MCP server bearer token 来自哪个**用户自定义**环境变量名 |
| `mcp_servers.<id>.env_http_headers` | 表 | HTTP MCP server header→env 映射 |
| `forced_login_method` | 顶层 | `chatgpt / api` |
| `forced_chatgpt_workspace_id` | 顶层 | 强制 ChatGPT 登录到指定 workspace UUID |
| `cli_auth_credentials_store` | 顶层 | `file / keyring / auto` |
| `chatgpt_base_url` | 顶层 | 覆盖 ChatGPT 登录 URL |
| `otel.*` | 表 | OTEL exporter / endpoint / headers / TLS；header 中可用 `${VAR}` 插值引用任意环境变量名 |
| `log_dir` | 顶层 | Codex 日志目录（默认 `$CODEX_HOME/log`） |
| `sqlite_home` | 顶层 | 状态 DB 目录 |
| `history.persistence` / `history.max_bytes` | 表 | 会话历史保留 |
| `analytics.enabled` | 表 | 分析总开关 |

**Codex CLI 的正确写法举例**：

```toml
# ~/.codex/config.toml
model_reasoning_effort = "high"
model_reasoning_summary = "auto"
sandbox_mode = "workspace-write"
approval_policy = "on-request"

[sandbox_workspace_write]
network_access = false

[shell_environment_policy]
inherit = "core"
```

或用命令行：

```bash
codex -c model_reasoning_effort=high -c sandbox_mode=workspace-write
```

---

## 5. 常见误传

| 错误说法 | 实际情况 |
|---|---|
| `CODEX_MODEL=…` 设当前模型 | **不存在**。改 `~/.codex/config.toml` 的 `model` 字段，或 `-c model=…` |
| `CODEX_MODEL_PROVIDER`、`CODEX_APPROVAL_POLICY`、`CODEX_SANDBOX_MODE`、`CODEX_OPENAI_API_KEY`、`CODEX_DEBUG` 等 | **均不存在**。Codex 不沿用 `CLAUDE_CODE_*` → `CODEX_*` 命名规则 |
| `OPENAI_BASE_URL` 改 Codex 的 OpenAI 端点 | **Codex 不读这个**。改 `openai_base_url` TOML 字段或 `-c openai_base_url=...` |
| `model_reasoning_effort = "max"` | **不存在**。Codex 的合法值是 `minimal/low/medium/high/xhigh` |
| `ANTHROPIC_API_KEY` 给 Codex 配 Anthropic | 默认不读；只有通过 `model_providers.<id>.env_key = "ANTHROPIC_API_KEY"` 手动接进来才行 |
| 在 `~/.codex/.env` 里写 `CODEX_FOO=1` | `codex-rs/arg0/src/lib.rs` 出于安全考虑**显式过滤** `CODEX_*` 前缀的 key，从这个文件里设无效。要么走真实 shell env，要么用 `config.toml` |

---

## 6. `OPENAI_ALLOWED_DOMAINS` 这种「灰区」变量

部分 Codex 早期资料提到 `OPENAI_ALLOWED_DOMAINS`，它**只被 `codex-cli/scripts/run_in_container.sh`**（一个 Docker 容器化沙箱脚本）使用，不是 Codex CLI binary 本身的环境变量。除非你直接用那个脚本，否则可以忽略。

旧的 TypeScript 版 Codex CLI（`codex-cli/` 子目录下，在 Rust 重写之前的版本）曾有 `OPENAI_TIMEOUT_MS`、`CODEX_QUIET_MODE` 等变量，**Rust binary 已不再支持**。

---

## 7. 推荐配置形态

环境变量层（写入 shell 或 `~/.codex/.env`，注意后者会过滤 `CODEX_*`）：

```bash
export OPENAI_API_KEY="sk-..."
export CODEX_HOME="$HOME/.codex"           # 改根目录
export CODEX_CA_CERTIFICATE="$HOME/certs/corp.pem"   # 企业 CA
export RUST_LOG="codex_core=debug,codex_tui=info"    # 调试
```

行为/模型层（写 `~/.codex/config.toml`）：

```toml
model = "gpt-5.5"
model_reasoning_effort = "high"
model_reasoning_summary = "auto"
sandbox_mode = "workspace-write"
approval_policy = "on-request"

[sandbox_workspace_write]
network_access = false

[shell_environment_policy]
inherit = "core"

[history]
persistence = "save-all"
max_bytes = 5_000_000
```

---

## 8. 版本备忘

- 本文档对应 `openai/codex` 仓库主线 `main`（release `rust-v0.130.0`，2026-05）。
- Codex CLI 演进很快，老版本（TypeScript 时代）env 表与此文档不兼容。
- 如果遇到本文档没有的 `CODEX_*` 变量，先在 `codex-rs/` 下 `rg "CODEX_"` 搜源码常量定义，再判断是否为内部用途。
