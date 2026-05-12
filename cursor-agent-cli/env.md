# cursor-agent CLI 环境变量参考

> **数据来源**：[`cursor.com/docs`](https://cursor.com/docs) 下的 CLI 子目录与 agent 子目录。
> **采集日期**：2026-05-12。
> **覆盖版本**：Cursor 官方文档无版本号标记，反映 2026 年 5 月当前 release。
> **结论先行**：Cursor 没有一个集中的「CLI 环境变量」页面。CLI 主要靠 flag 和 `cli-config.json` 配置；**真正写在 CLI 文档里的环境变量只有 7 个**：1 个鉴权、2 个配置路径、4 个网络代理/CA。另外有 5 个 `CURSOR_*` 是给 IDE agent 的沙箱用的，CLI 是否一致注入官方未明示。

---

## 1. 数据来源（已验证可达的官方页面）

| URL | 内容 |
|---|---|
| [`cursor.com/docs/cli/reference/configuration`](https://cursor.com/docs/cli/reference/configuration) | `CURSOR_CONFIG_DIR`、`XDG_CONFIG_HOME`、代理/CA |
| [`cursor.com/docs/cli/reference/parameters`](https://cursor.com/docs/cli/reference/parameters) | `CURSOR_API_KEY` ↔ `--api-key` |
| [`cursor.com/docs/cli/reference/authentication`](https://cursor.com/docs/cli/reference/authentication) | `CURSOR_API_KEY`「Option 1: Environment variable (recommended)」 |
| [`cursor.com/docs/cli/headless`](https://cursor.com/docs/cli/headless) | headless 脚本场景：`CURSOR_API_KEY` |
| [`cursor.com/docs/cli/github-actions`](https://cursor.com/docs/cli/github-actions) | GitHub Actions：只提 `CURSOR_API_KEY` |
| [`cursor.com/docs/cli/overview`](https://cursor.com/docs/cli/overview) | 概览（未列 env） |
| [`cursor.com/docs/cli/using`](https://cursor.com/docs/cli/using) | 使用说明（未列 env） |
| [`cursor.com/docs/agent/tools/terminal`](https://cursor.com/docs/agent/tools/terminal) | **IDE agent 的沙箱**：`CURSOR_SANDBOX` 等，**不专门针对 CLI** |

返回 404 / 308 跳转到 `/docs` 根的页面：`/docs/cli`、`/docs/cli/installation`、`/docs/cli/troubleshooting`、`/docs/cli/automations`、`/docs/cli/sandbox`、`/docs/cli/reference/shell-mode`、`/docs/agent/sandboxing`。

---

## 2. 速查总表

### 2.1 CLI 文档中**正式写出**的环境变量（7 个）

| 变量 | 默认值 | 类别 | 状态 | 来源 |
|---|---|---|---|---|
| `CURSOR_API_KEY` | — | 鉴权 | documented | authentication / parameters / headless / github-actions |
| `CURSOR_CONFIG_DIR` | 平台默认目录 | 配置路径 | documented | configuration |
| `XDG_CONFIG_HOME` | — | 配置路径 | documented | configuration |
| `HTTP_PROXY` | — | 网络 | documented | configuration |
| `HTTPS_PROXY` | — | 网络 | documented | configuration |
| `NODE_USE_ENV_PROXY` | — | 网络 | documented | configuration |
| `NODE_EXTRA_CA_CERTS` | — | TLS | documented | configuration |

### 2.2 IDE agent 终端沙箱里被注入的变量（CLI 没明示，但同源）

| 变量 | 类别 | 状态 | 来源 |
|---|---|---|---|
| `CURSOR_SANDBOX` | 沙箱握手 | documented (IDE agent) | agent/tools/terminal |
| `CURSOR_ORIG_UID` | 沙箱握手 (macOS/Linux) | documented (IDE agent) | agent/tools/terminal |
| `CURSOR_ORIG_GID` | 沙箱握手 (macOS/Linux) | documented (IDE agent) | agent/tools/terminal |
| `CURSOR_SANDBOX_LANDLOCK_STATUS` | 沙箱握手 (Linux) | documented (IDE agent) | agent/tools/terminal |
| `CURSOR_AGENT` | 检测 agent 运行中 | documented (IDE agent) / inferred (CLI) | agent/tools/terminal + 论坛 bug 反馈 |

---

## 3. 详细说明

### 3.1 `CURSOR_API_KEY`
- **默认**：无。
- **效果**：等价于 `--api-key <key>` 命令行参数。在自动化、脚本、CI/CD 中推荐用环境变量传，避免命令行历史里留 key。
- **使用场景**：headless 模式、GitHub Actions。
- **来源**：[`authentication`](https://cursor.com/docs/cli/reference/authentication) / [`parameters`](https://cursor.com/docs/cli/reference/parameters) / [`headless`](https://cursor.com/docs/cli/headless) / [`github-actions`](https://cursor.com/docs/cli/github-actions)。

### 3.2 配置文件路径

#### `CURSOR_CONFIG_DIR`
- **默认**：平台默认（`~/.config/cursor/`、`%APPDATA%\cursor\` 等）。
- **效果**：覆盖 CLI 的 `cli-config.json` 所在目录。
- **来源**：[`configuration`](https://cursor.com/docs/cli/reference/configuration)。

#### `XDG_CONFIG_HOME`
- **效果**：Linux/BSD 上若设置，CLI 会读 `$XDG_CONFIG_HOME/cursor/cli-config.json`。是 XDG Base Directory 标准约定。
- **来源**：同上。

### 3.3 网络代理 / CA

| 变量 | 作用 |
|---|---|
| `HTTP_PROXY` | HTTP 流量走指定代理 |
| `HTTPS_PROXY` | HTTPS 流量走指定代理 |
| `NODE_USE_ENV_PROXY` | 让内嵌的 Node.js 运行时识别 `HTTP_PROXY` / `HTTPS_PROXY`（一般设 `1`） |
| `NODE_EXTRA_CA_CERTS` | 额外的 PEM 信任 CA 证书路径（企业代理做 SSL 解密时常用） |

这 4 个本质上是标准 OS/Node 约定，但 Cursor 在文档里**明文承诺支持**。

### 3.4 沙箱握手变量（IDE agent 视角，CLI 未明示）

> **重要**：这些变量出现在 [`agent/tools/terminal`](https://cursor.com/docs/agent/tools/terminal)（IDE 内置 agent 的终端工具说明），CLI 文档里**没有**复述。CLI 是否也会注入它们，Cursor 没有官方答复。

| 变量 | 取值 | 作用 |
|---|---|---|
| `CURSOR_SANDBOX` | macOS：`"seatbelt"`；Linux/Windows：`"native"`；未沙箱：unset | 子进程检测自己是否在沙箱里 |
| `CURSOR_ORIG_UID` | 数字 | 启动 Cursor 的用户 UID（在沙箱身份切换前捕获） |
| `CURSOR_ORIG_GID` | 数字 | 同上，GID |
| `CURSOR_SANDBOX_LANDLOCK_STATUS` | `fully_enforced` / `bubblewrap` | Linux 上当前沙箱后端 |

#### `CURSOR_AGENT`
- 文档中只在 troubleshooting 段落里出现：「可以在 shell rc 里测它，给 agent 跑命令时跳过花哨的 prompt 主题」。
- 论坛报告（[forum.cursor.com/t/132427](https://forum.cursor.com/t/cursor-cli-is-not-setting-cursor-agent-1-environment-variable-while-executing-bash-commands/132427)）反映 **CLI 并不总是注入这个变量**——属于「期望但不保证」的状态。
- 行为：**IDE agent 注入；CLI 行为不稳定**。

---

## 4. Cursor CLI **没有**官方文档的常见诉求

| 想做的事 | 官方有无 env 支持 | 现实做法 |
|---|---|---|
| 关掉 telemetry / `DO_NOT_TRACK` | **没找到** | 没有官方 opt-out env；只能用网络层封禁 |
| 模型选择（环境变量） | **没找到** | 用 `--model` flag 或 `cli-config.json` |
| 调高日志级别 / debug 模式 | **没找到** | 没有官方 env；只能用 `--help` 里的 verbose flag（如有） |
| 工作区/项目路径覆盖 | **没找到**（CLI 层面） | 只有 `CURSOR_CONFIG_DIR` 改配置目录 |
| API 端点覆盖（自建网关） | **没找到** | 文档里没有「base URL」类 env |
| 沙箱模式开关（CLI 层） | **没找到** | 文档只展示 IDE agent 的沙箱说明 |
| 子 agent / 多 agent 控制 | **没找到** | 没有 env |

> 如果你看到第三方清单里写了 `CURSOR_TELEMETRY=0`、`CURSOR_BASE_URL=…`、`CURSOR_MODEL=…`、`CURSOR_DEBUG=…` 之类——**在 2026-05 的官方文档里都不存在**，多半是从其他 CLI（如 Claude Code、Codex）类比出来的，别照搬。

---

## 5. 推荐用法

### 5.1 Headless / CI

```bash
export CURSOR_API_KEY="..."
cursor-agent --print "fix the failing test"
```

### 5.2 走企业代理

```bash
export HTTP_PROXY="http://proxy.corp:8080"
export HTTPS_PROXY="http://proxy.corp:8080"
export NODE_USE_ENV_PROXY=1
export NODE_EXTRA_CA_CERTS="/etc/ssl/corp-bundle.pem"
cursor-agent ...
```

### 5.3 多账号 / 隔离配置

```bash
CURSOR_CONFIG_DIR="$HOME/.cursor-work" cursor-agent ...
CURSOR_CONFIG_DIR="$HOME/.cursor-personal" cursor-agent ...
```

或走 XDG 习惯：

```bash
XDG_CONFIG_HOME="$HOME/.config-work" cursor-agent ...
```

### 5.4 shell prompt 适配（避免 agent 跑命令时被花哨 prompt 干扰）

```bash
# .bashrc / .zshrc
if [[ -n "$CURSOR_AGENT" ]]; then
  PS1='$ '   # 简化 prompt
fi
```

注意：Cursor 论坛确认 CLI 注入 `CURSOR_AGENT` 行为不稳定，把这个当 **best-effort** 检测；如果要稳，再叠加判断 `$CURSOR_SANDBOX` 是否非空。

---

## 6. 与 Claude Code / Codex 对比的差异

| 维度 | cursor-agent CLI | Claude Code | Codex |
|---|---|---|---|
| 主配置形式 | flag + `cli-config.json` | `settings.json` 的 `env` + env vars | `config.toml` + `-c key=value` + env vars |
| 文档化 env 数量 | ~7 个（含通用代理） | ~250 个 | 用户面 ~7 个（其余是仓库内部常量） |
| Telemetry opt-out env | **无官方支持** | `DO_NOT_TRACK` / `DISABLE_TELEMETRY` / `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | 走 `config.toml` 的 `analytics.enabled` |
| 模型选择 env | **无** | `ANTHROPIC_MODEL` 等多个 | TOML 字段 `model` |
| Base URL 覆盖 env | **无** | `ANTHROPIC_BASE_URL` | TOML 字段 `openai_base_url` |
| 调试 env | **无** | `CLAUDE_CODE_DEBUG_LOGS_DIR` + `--debug` | `RUST_LOG` |
| 沙箱握手 env（写给子进程） | `CURSOR_SANDBOX` 等（IDE 视角） | `CLAUDECODE`、`CLAUDE_CODE_SESSION_ID` 等 | `CODEX_SANDBOX`、`CODEX_SANDBOX_NETWORK_DISABLED` |

> **关键判断**：Cursor 的 CLI 把绝大多数配置藏在 `cli-config.json` 与 flag 里，环境变量面非常窄；不要按其他 CLI 的命名习惯去猜。

---

## 7. 备注

- Cursor 文档无版本号标签，本表反映 2026-05-12 抓取时的当前 release。
- 若 Cursor 后续推出 `CURSOR_BASE_URL` / `CURSOR_MODEL` 之类的 env，先在官方 docs 里逐字搜索确认，再加进本表。
- IDE agent 与 CLI 同源但表面不同。沙箱握手类变量在 IDE agent 文档里明确，但 CLI 没有复述，使用时务必实测（用 `cursor-agent --print "echo $CURSOR_SANDBOX"` 之类）。
