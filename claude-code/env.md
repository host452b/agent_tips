# Claude Code 环境变量参考

> **数据来源**：[`https://code.claude.com/docs/en/env-vars`](https://code.claude.com/docs/en/env-vars)（官方权威，单页 246 个标识符的总表）外加 `settings`、`hooks`、`model-config`、`monitoring-usage`、`amazon-bedrock`、`google-vertex-ai`、`microsoft-foundry`、`claude-platform-on-aws`、`llm-gateway` 等子页面交叉验证。
> **采集日期**：2026-05-12。
> **覆盖版本**：以 `code.claude.com/docs` 当前文档为准；个别变量标注引入版本（如 `v2.1.94+`、`v2.1.111+`）。
> **使用方式**：可通过 shell 直接 `export`，或写入 `~/.claude/settings.json` 的 `env` 字段；后者对每个 session 生效，并会强制传给子进程。

---

## 1. 速查总表（仅列变量名 + 一句话作用 + 状态）

| 变量名 | 一句话作用 | 状态 |
|---|---|---|
| **认证与订阅** | | |
| `ANTHROPIC_API_KEY` | 用 API key 鉴权，覆盖 Claude.ai 订阅 | documented |
| `ANTHROPIC_AUTH_TOKEN` | 自定义 `Authorization: Bearer …` | documented |
| `CLAUDE_CODE_OAUTH_TOKEN` | OAuth access token，优先于 keychain | documented |
| `CLAUDE_CODE_OAUTH_REFRESH_TOKEN` | OAuth refresh token，免浏览器登陆 | documented |
| `CLAUDE_CODE_OAUTH_SCOPES` | 与 refresh token 配套的 scope | documented |
| `CLAUDE_CONFIG_DIR` | 覆盖配置目录（默认 `~/.claude`） | documented |
| **API & 网络** | | |
| `ANTHROPIC_BASE_URL` | 自定义 API 端点（走 proxy/gateway） | documented |
| `ANTHROPIC_CUSTOM_HEADERS` | 注入自定义 HTTP 头 | documented |
| `ANTHROPIC_BETAS` | 注入 `anthropic-beta` 头值 | documented |
| `API_TIMEOUT_MS` | API 请求超时 ms，默认 `600000` | documented |
| `CLAUDE_CODE_MAX_RETRIES` | 重试次数，默认 `10` | documented |
| `CLAUDE_CODE_DISABLE_NONSTREAMING_FALLBACK` | 关闭流式失败时的非流式回退 | documented |
| `CLAUDE_CODE_PROXY_RESOLVES_HOSTS` | 让 proxy 负责 DNS 解析 | documented |
| `CLAUDE_CODE_EXTRA_BODY` | 注入每个请求 body 的 JSON 字段 | documented |
| `CLAUDE_CODE_ATTRIBUTION_HEADER` | `0` 关闭归因头，提高 gateway 缓存命中 | documented |
| `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS` | 剥离 beta 头/字段，兼容旧 gateway | documented |
| `CLAUDE_CODE_ENABLE_FINE_GRAINED_TOOL_STREAMING` | 工具输入边生成边流式 | documented |
| `ENABLE_TOOL_SEARCH` | 第三方 base URL 下启用 MCP 工具搜索 | documented |
| `CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY` | 从 gateway `/v1/models` 填充 `/model` | documented |
| `CLAUDE_ENABLE_BYTE_WATCHDOG` | 字节级 idle watchdog 开关 | documented |
| `CLAUDE_ENABLE_STREAM_WATCHDOG` | event 级 idle watchdog 开关 | documented |
| `CLAUDE_STREAM_IDLE_TIMEOUT_MS` | watchdog 超时阈值 | documented |
| `HTTP_PROXY` / `HTTPS_PROXY` / `NO_PROXY` | 标准代理变量 | documented |
| **TLS / mTLS** | | |
| `CLAUDE_CODE_CERT_STORE` | CA 证书来源（`bundled,system`） | documented |
| `CLAUDE_CODE_CLIENT_CERT` | mTLS 客户端证书路径 | documented |
| `CLAUDE_CODE_CLIENT_KEY` | mTLS 客户端私钥路径 | documented |
| `CLAUDE_CODE_CLIENT_KEY_PASSPHRASE` | 加密私钥的口令 | documented |
| `NODE_EXTRA_CA_CERTS` | Node 标准额外 CA（OTLP 也用） | documented |
| **Amazon Bedrock** | | |
| `CLAUDE_CODE_USE_BEDROCK` | 启用 Bedrock provider | documented |
| `CLAUDE_CODE_SKIP_BEDROCK_AUTH` | 跳过 AWS 鉴权（由 gateway 签名） | documented |
| `ANTHROPIC_BEDROCK_BASE_URL` | 覆盖 Bedrock 端点 | documented |
| `ANTHROPIC_BEDROCK_SERVICE_TIER` | `default/flex/priority` | documented |
| `AWS_BEARER_TOKEN_BEDROCK` | Bedrock API-key 鉴权 | documented |
| `ANTHROPIC_SMALL_FAST_MODEL_AWS_REGION` | Haiku 类模型的 AWS 区域覆盖 | documented |
| `CLAUDE_CODE_USE_MANTLE` | 启用 Mantle 端点（v2.1.94+） | documented |
| `CLAUDE_CODE_SKIP_MANTLE_AUTH` | 跳过 Mantle 鉴权 | documented |
| `ANTHROPIC_BEDROCK_MANTLE_BASE_URL` | 覆盖 Mantle 端点 | documented |
| **Claude Platform on AWS** | | |
| `CLAUDE_CODE_USE_ANTHROPIC_AWS` | 走 Anthropic 运营的 AWS 端点 | documented |
| `CLAUDE_CODE_SKIP_ANTHROPIC_AWS_AUTH` | 跳过 SigV4（由 gateway 签名） | documented |
| `ANTHROPIC_AWS_API_KEY` | workspace API key（优先于 SigV4） | documented |
| `ANTHROPIC_AWS_WORKSPACE_ID` | 必填 workspace id | documented |
| `ANTHROPIC_AWS_BASE_URL` | 覆盖端点 | documented |
| **Google Vertex** | | |
| `CLAUDE_CODE_USE_VERTEX` | 启用 Vertex provider（向导 v2.1.98+） | documented |
| `CLAUDE_CODE_SKIP_VERTEX_AUTH` | 跳过 Google 鉴权 | documented |
| `ANTHROPIC_VERTEX_BASE_URL` | 覆盖 Vertex 端点 | documented |
| `ANTHROPIC_VERTEX_PROJECT_ID` | GCP 项目 id（可被标准 GCP 变量覆盖） | documented |
| `GCLOUD_PROJECT` / `GOOGLE_CLOUD_PROJECT` | 标准 GCP 项目变量 | documented |
| `GOOGLE_APPLICATION_CREDENTIALS` | ADC 路径；v2.1.121+ 支持 WIF | documented |
| `CLOUD_ML_REGION` | Vertex 区域 | documented |
| `VERTEX_REGION_CLAUDE_3_5_HAIKU` | Vertex 模型粒度区域覆盖 | documented |
| `VERTEX_REGION_CLAUDE_3_5_SONNET` | 同上 | documented |
| `VERTEX_REGION_CLAUDE_3_7_SONNET` | 同上 | documented |
| `VERTEX_REGION_CLAUDE_4_0_OPUS` | 同上 | documented |
| `VERTEX_REGION_CLAUDE_4_0_SONNET` | 同上 | documented |
| `VERTEX_REGION_CLAUDE_4_1_OPUS` | 同上 | documented |
| `VERTEX_REGION_CLAUDE_4_5_OPUS` | 同上 | documented |
| `VERTEX_REGION_CLAUDE_4_5_SONNET` | 同上 | documented |
| `VERTEX_REGION_CLAUDE_4_6_OPUS` | 同上 | documented |
| `VERTEX_REGION_CLAUDE_4_6_SONNET` | 同上 | documented |
| `VERTEX_REGION_CLAUDE_4_7_OPUS` | 同上（要求 v2.1.111+） | documented |
| `VERTEX_REGION_CLAUDE_HAIKU_4_5` | 同上 | documented |
| **Microsoft Foundry** | | |
| `CLAUDE_CODE_USE_FOUNDRY` | 启用 Foundry provider | documented |
| `CLAUDE_CODE_SKIP_FOUNDRY_AUTH` | 跳过 Azure 鉴权 | documented |
| `ANTHROPIC_FOUNDRY_API_KEY` | Foundry API key | documented |
| `ANTHROPIC_FOUNDRY_RESOURCE` | Foundry resource 名称 | documented |
| `ANTHROPIC_FOUNDRY_BASE_URL` | Foundry 完整 base URL | documented |
| **Provider 路由** | | |
| `CLAUDE_CODE_PROVIDER_MANAGED_BY_HOST` | 平台嵌入：屏蔽用户 settings 里的 provider 字段 | documented |
| **模型配置** | | |
| `ANTHROPIC_MODEL` | 当前模型 | documented |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | `opus` 解析到哪个具体模型 | documented |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | `sonnet` 解析到哪个 | documented |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | `haiku`/后台任务用哪个 | documented |
| `ANTHROPIC_SMALL_FAST_MODEL` | **已弃用** → `ANTHROPIC_DEFAULT_HAIKU_MODEL` | deprecated |
| `CLAUDE_CODE_SUBAGENT_MODEL` | 子 agent 模型 | documented |
| `ANTHROPIC_CUSTOM_MODEL_OPTION` | 给 `/model` 加自定义条目 | documented |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_NAME` | 自定义条目显示名 | documented |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_DESCRIPTION` | 自定义条目描述 | documented |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_SUPPORTED_CAPABILITIES` | 自定义条目能力位 | documented |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_NAME` / `_DESCRIPTION` / `_SUPPORTED_CAPABILITIES` | 钉死 Opus 的显示与能力 | documented |
| `ANTHROPIC_DEFAULT_SONNET_MODEL_NAME` / `_DESCRIPTION` / `_SUPPORTED_CAPABILITIES` | 钉死 Sonnet | documented |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_NAME` / `_DESCRIPTION` / `_SUPPORTED_CAPABILITIES` | 钉死 Haiku | documented |
| `CLAUDE_CODE_DISABLE_LEGACY_MODEL_REMAP` | 禁止把旧 Opus 自动重映射到当前 Opus | documented |
| `CLAUDE_CODE_DISABLE_1M_CONTEXT` | 隐藏 1M 上下文变体 | documented |
| `FALLBACK_FOR_ALL_PRIMARY_MODELS` | 任何主力模型 overload 后都触发 fallback | documented |
| **Effort / Thinking** | | |
| `CLAUDE_CODE_EFFORT_LEVEL` | `low/medium/high/xhigh/max/auto`；`max` 时持久化 | documented |
| `CLAUDE_EFFORT` | **只读**：Bash/hook 子进程里被注入当前 effort | documented |
| `MAX_THINKING_TOKENS` | thinking 预算上限；adaptive 模型默认无效 | documented |
| `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING` | 关闭 4.6 系的 adaptive，回到固定预算；**对 Opus 4.7 无效** | documented |
| `CLAUDE_CODE_DISABLE_THINKING` | 强制关 thinking（比 `MAX_THINKING_TOKENS=0` 更直接） | documented |
| `DISABLE_INTERLEAVED_THINKING` | 跳过 interleaved thinking beta 头 | documented |
| **上下文 & 压缩** | | |
| `CLAUDE_CODE_MAX_OUTPUT_TOKENS` | 每次请求最大输出 tokens | documented |
| `CLAUDE_CODE_MAX_CONTEXT_TOKENS` | 覆盖上下文窗口大小，**需同时设 `DISABLE_COMPACT=1`** | documented |
| `CLAUDE_CODE_AUTO_COMPACT_WINDOW` | 自动压缩用的容量 | documented |
| `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` | 触发自动压缩的百分比（1-100，默认 ~95） | documented |
| `DISABLE_COMPACT` | 全关压缩（含手动 `/compact`） | documented |
| `DISABLE_AUTO_COMPACT` | 只关自动压缩 | documented |
| `CLAUDE_CODE_FILE_READ_MAX_OUTPUT_TOKENS` | 文件读取的 token 上限 | documented |
| **CLAUDE.md / 记忆** | | |
| `CLAUDE_CODE_DISABLE_CLAUDE_MDS` | 禁加载所有 CLAUDE.md | documented |
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY` | 关 / 强制开 auto memory | documented |
| `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD` | 也加载 `--add-dir` 目录里的 CLAUDE.md | documented |
| **Bash / Shell 工具** | | |
| `BASH_DEFAULT_TIMEOUT_MS` | bash 默认超时（`120000`） | documented |
| `BASH_MAX_TIMEOUT_MS` | bash 最大超时（`600000`） | documented |
| `BASH_MAX_OUTPUT_LENGTH` | 输出超此长度落盘后只给路径+预览 | documented |
| `CLAUDE_CODE_SHELL` | 覆盖 shell 自动检测 | documented |
| `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR` | 每条命令后回到原 cwd | documented |
| `CLAUDECODE` | **只读**：Bash/tmux 子进程里被注入 `1` | documented |
| `CLAUDE_CODE_SHELL_PREFIX` | 给所有外壳命令套一层包装 | documented |
| `CLAUDE_CODE_USE_POWERSHELL_TOOL` | 控制 PowerShell 工具 | documented |
| `CLAUDE_CODE_GIT_BASH_PATH` | Windows 下 Git Bash 路径 | documented |
| `CLAUDE_CODE_TMPDIR` | 覆盖临时目录 | documented |
| `CLAUDE_CODE_TMUX_TRUECOLOR` | tmux 中开 24-bit 色 | documented |
| **文件操作 / 搜索** | | |
| `CLAUDE_CODE_GLOB_HIDDEN` | Glob 是否包含点文件（默认 `true`） | documented |
| `CLAUDE_CODE_GLOB_NO_IGNORE` | Glob 是否忽略 `.gitignore`（默认 `true`） | documented |
| `CLAUDE_CODE_GLOB_TIMEOUT_SECONDS` | Glob 超时（`20`，WSL `60`） | documented |
| `CLAUDE_CODE_DISABLE_ATTACHMENTS` | 关闭 `@` 文件附件 | documented |
| `CLAUDE_CODE_DISABLE_FILE_CHECKPOINTING` | 关闭 `/rewind` checkpoint | documented |
| `CLAUDE_CODE_PERFORCE_MODE` | Perforce 感知写保护 | documented |
| `USE_BUILTIN_RIPGREP` | `0` 改用系统 `rg` | documented |
| `CLAUDE_CODE_USE_NATIVE_FILE_SEARCH` | 走 Node fs 而非 ripgrep 发现命令/skill | documented |
| **工具 / MCP / 并发** | | |
| `CLAUDE_CODE_MAX_TOOL_USE_CONCURRENCY` | 并行只读工具/子 agent 数（默认 `10`） | documented |
| `CLAUDE_CODE_MCP_ALLOWLIST_ENV` | stdio MCP 用最小安全 env | documented |
| `MCP_TIMEOUT` | MCP server 启动超时 ms（`30000`） | documented |
| `MCP_TOOL_TIMEOUT` | MCP 工具执行超时 ms（~28 小时） | documented |
| `MCP_CONNECT_TIMEOUT_MS` | 首次查询等待 MCP 连接的时间（`5000`） | documented |
| `MCP_CONNECTION_NONBLOCKING` | `-p` 模式跳过 MCP 等待 | documented |
| `MCP_CLIENT_SECRET` | MCP OAuth client secret | documented |
| `MCP_OAUTH_CALLBACK_PORT` | 固定 OAuth callback 端口 | documented |
| `MCP_SERVER_CONNECTION_BATCH_SIZE` | stdio MCP 启动并发批量（`3`） | documented |
| `MCP_REMOTE_SERVER_CONNECTION_BATCH_SIZE` | 远端 MCP 启动并发（`20`） | documented |
| `MAX_MCP_OUTPUT_TOKENS` | MCP 工具响应 token 上限（`25000`） | documented |
| `ENABLE_CLAUDEAI_MCP_SERVERS` | claude.ai MCP server 开关 | documented |
| **Skill** | | |
| `CLAUDE_CODE_DISABLE_POLICY_SKILLS` | 跳过系统级 skill | documented |
| `SLASH_COMMAND_TOOL_CHAR_BUDGET` | Skill 元数据字符预算 | documented |
| **Agent / 子 agent / 后台任务** | | |
| `CLAUDE_CODE_DISABLE_AGENT_VIEW` | 关闭后台 agent 与 agent 视图 | documented |
| `CLAUDE_AUTO_BACKGROUND_TASKS` | 强制开自动后台化（**不是 `CLAUDE_CODE_AUTO_BACKGROUND_TASKS`**） | documented |
| `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` | 关全部后台任务功能 | documented |
| `CLAUDE_ASYNC_AGENT_STALL_TIMEOUT_MS` | 后台 subagent 停滞超时（`600000`） | documented |
| `CLAUDE_AGENT_SDK_DISABLE_BUILTIN_AGENTS` | `-p`：禁内置子 agent | documented |
| `CLAUDE_CODE_FORK_SUBAGENT` | 启用 forked subagent（继承上下文） | documented |
| `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` | agent teams | experimental |
| `CLAUDE_CODE_TEAM_NAME` | **只读**：team 成员上自动设的 team 名 | documented |
| `CLAUDE_AGENT_SDK_MCP_NO_PREFIX` | SDK：跳过 `mcp__<server>__` 前缀 | documented |
| `TASK_MAX_OUTPUT_LENGTH` | subagent 输出字符上限（默认 `32000`，最大 `160000`） | documented |
| **定时任务** | | |
| `CLAUDE_CODE_DISABLE_CRON` | 禁 `/loop` 与 cron 工具 | documented |
| **Session 管理** | | |
| `CLAUDE_CODE_SESSION_ID` | **只读**：当前 session id | documented |
| `CLAUDE_CODE_REMOTE` | **只读**：是否在云 session | documented |
| `CLAUDE_CODE_REMOTE_SESSION_ID` | **只读**：云 session id | documented |
| `CCR_FORCE_BUNDLE` | `claude --remote` 强制打包本地仓库上传 | documented |
| `CLAUDE_REMOTE_CONTROL_SESSION_NAME_PREFIX` | 远程会话名前缀 | documented |
| `CLAUDE_CODE_TASK_LIST_ID` | 跨 session 共享 task list | documented |
| `CLAUDE_CODE_RESUME_INTERRUPTED_TURN` | SDK：自动恢复中断的回合 | documented |
| `CLAUDE_CODE_EXIT_AFTER_STOP_DELAY` | 闲置后多少 ms 自动退出（SDK） | documented |
| `CLAUDE_CODE_MAX_TURNS` | 回合数上限 | documented |
| `CLAUDE_CODE_SKIP_PROMPT_HISTORY` | 跳过 prompt 历史与 transcript | documented |
| `CLAUDE_CODE_SESSIONEND_HOOKS_TIMEOUT_MS` | SessionEnd hooks 预算 | documented |
| **Hooks（注入到 hook 命令）** | | |
| `CLAUDE_PROJECT_DIR` | hook 命令里替换为项目根 | documented |
| `CLAUDE_PLUGIN_ROOT` | hook 命令里替换为插件目录 | documented |
| `CLAUDE_PLUGIN_DATA` | hook 命令里替换为插件持久数据目录 | documented |
| `CLAUDE_ENV_FILE` | hook 可写入：下一条 Bash 命令前 source | documented |
| **插件** | | |
| `CLAUDE_CODE_PLUGIN_CACHE_DIR` | 插件根目录（默认 `~/.claude/plugins`） | documented |
| `CLAUDE_CODE_PLUGIN_GIT_TIMEOUT_MS` | 插件 git 操作超时（`120000`） | documented |
| `CLAUDE_CODE_PLUGIN_KEEP_MARKETPLACE_ON_FAILURE` | git pull 失败时保留 marketplace 缓存 | documented |
| `CLAUDE_CODE_PLUGIN_SEED_DIR` | 只读插件 seed 目录列表 | documented |
| `CLAUDE_CODE_DISABLE_OFFICIAL_MARKETPLACE_AUTOINSTALL` | 跳过首次官方 marketplace 自动添加 | documented |
| `CLAUDE_CODE_ENABLE_BACKGROUND_PLUGIN_REFRESH` | `-p` 中回合边界刷新插件 | documented |
| `CLAUDE_CODE_SYNC_PLUGIN_INSTALL` | `-p` 等待插件安装完成 | documented |
| `CLAUDE_CODE_SYNC_PLUGIN_INSTALL_TIMEOUT_MS` | 上一项的等待上限 | documented |
| `FORCE_AUTOUPDATE_PLUGINS` | 强制插件自动更新 | documented |
| **IDE 集成** | | |
| `CLAUDE_CODE_AUTO_CONNECT_IDE` | 自动连接 IDE 开关 | documented |
| `CLAUDE_CODE_IDE_HOST_OVERRIDE` | 覆盖 IDE 连接主机（WSL→Windows 路由） | documented |
| `CLAUDE_CODE_IDE_SKIP_AUTO_INSTALL` | 跳过 IDE 扩展自动安装 | documented |
| `CLAUDE_CODE_IDE_SKIP_VALID_CHECK` | 跳过 IDE lockfile 校验 | documented |
| **UI / 终端渲染** | | |
| `CLAUDE_CODE_NO_FLICKER` | 全屏渲染（减闪烁，长会话内存平） | documented |
| `CLAUDE_CODE_DISABLE_ALTERNATE_SCREEN` | 强制经典主屏渲染 | documented |
| `CLAUDE_CODE_DISABLE_MOUSE` | 关全屏鼠标跟踪 | documented |
| `CLAUDE_CODE_DISABLE_VIRTUAL_SCROLL` | 全屏每条都渲染（修空白滚动 bug） | documented |
| `CLAUDE_CODE_SCROLL_SPEED` | 滚轮速度 1-20 | documented |
| `CLAUDE_CODE_FORCE_SYNC_OUTPUT` | DEC 2026 同步输出 | documented |
| `CLAUDE_CODE_NATIVE_CURSOR` | 用终端原生光标 | documented |
| `CLAUDE_CODE_DISABLE_TERMINAL_TITLE` | 关终端 title 自动更新 | documented |
| `CLAUDE_CODE_HIDE_CWD` | 启动 logo 不显示 cwd | documented |
| `CLAUDE_CODE_ACCESSIBILITY` | 屏幕放大器友好 | documented |
| `CLAUDE_CODE_SYNTAX_HIGHLIGHT` | diff 语法高亮开关 | documented |
| `IS_DEMO` | 演示模式 | documented |
| **输出模式 / 系统提示词** | | |
| `CLAUDE_CODE_DISABLE_FAST_MODE` | 关 fast mode | documented |
| `CLAUDE_CODE_SIMPLE` | 最小系统提示，仅 Bash/Read/Edit | documented |
| `CLAUDE_CODE_SIMPLE_SYSTEM_PROMPT` | **仅 Opus 4.7**：缩短系统提示 | documented |
| `CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS` | 系统提示去掉 git workflow | documented |
| `CLAUDE_CODE_NEW_INIT` | `/init` 走交互式 setup | documented |
| `CLAUDE_CODE_ENABLE_AWAY_SUMMARY` | session recap 开关 | documented |
| `CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION` | 灰色 prompt 预测开关 | documented |
| `CLAUDE_CODE_ENABLE_TASKS` | `-p` 模式启用任务跟踪 | documented |
| **Prompt Caching** | | |
| `DISABLE_PROMPT_CACHING` | 全部模型关缓存（最高优先级） | documented |
| `DISABLE_PROMPT_CACHING_HAIKU` | Haiku 关缓存 | documented |
| `DISABLE_PROMPT_CACHING_OPUS` | Opus 关缓存 | documented |
| `DISABLE_PROMPT_CACHING_SONNET` | Sonnet 关缓存 | documented |
| `ENABLE_PROMPT_CACHING_1H` | 请求 1 小时 TTL（订阅自动开） | documented |
| `ENABLE_PROMPT_CACHING_1H_BEDROCK` | **已弃用** → `ENABLE_PROMPT_CACHING_1H` | deprecated |
| `FORCE_PROMPT_CACHING_5M` | 强制 5 分钟 TTL | documented |
| **结构化输出** | | |
| `MAX_STRUCTURED_OUTPUT_RETRIES` | `-p --json-schema` 校验失败重试次数（`5`） | documented |
| **Telemetry / 关闭项** | | |
| `CLAUDE_CODE_ENABLE_TELEMETRY` | 启用 OTel（OTel 系列变量的前置） | documented |
| `DISABLE_TELEMETRY` | 关匿名 telemetry | documented |
| `DO_NOT_TRACK` | 跨工具标准 opt-out（等价上一项） | documented |
| `DISABLE_ERROR_REPORTING` | 关错误上报 | documented |
| `DISABLE_AUTOUPDATER` | 关自动更新检查 | documented |
| `DISABLE_UPDATES` | 连手动 `claude update` 都禁 | documented |
| `DISABLE_FEEDBACK_COMMAND` | 关 `/feedback` | documented |
| `DISABLE_BUG_COMMAND` | 上一项的**别名** | documented |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | 上面四项的合集 | documented |
| `CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY` | 关 session 质量问卷 | documented |
| `CLAUDE_CODE_PACKAGE_MANAGER_AUTO_UPDATE` | 自动 brew/winget 升级 | documented |
| `DISABLE_GROWTHBOOK` | 关 GrowthBook feature flag | documented |
| `DISABLE_COST_WARNINGS` | 关成本警告 | documented |
| **隐藏命令（企业/托管）** | | |
| `DISABLE_DOCTOR_COMMAND` | 隐藏 `/doctor` | documented |
| `DISABLE_EXTRA_USAGE_COMMAND` | 隐藏 `/extra-usage` | documented |
| `DISABLE_INSTALL_GITHUB_APP_COMMAND` | 隐藏 `/install-github-app` | documented |
| `DISABLE_LOGIN_COMMAND` | 隐藏 `/login` | documented |
| `DISABLE_LOGOUT_COMMAND` | 隐藏 `/logout` | documented |
| `DISABLE_UPGRADE_COMMAND` | 隐藏 `/upgrade` | documented |
| `DISABLE_INSTALLATION_CHECKS` | 关装机告警 | documented |
| **OTel（Claude Code 自家）** | | |
| `CLAUDE_CODE_OTEL_FLUSH_TIMEOUT_MS` | flush span 超时（`5000`） | documented |
| `CLAUDE_CODE_OTEL_SHUTDOWN_TIMEOUT_MS` | 关闭超时（`2000`） | documented |
| `CLAUDE_CODE_OTEL_HEADERS_HELPER_DEBOUNCE_MS` | 动态 header 刷新（`1740000`） | documented |
| `CLAUDE_CODE_API_KEY_HELPER_TTL_MS` | `apiKeyHelper` 凭证刷新间隔 | documented |
| `OTEL_LOG_USER_PROMPTS` | trace 中是否含用户 prompt 文本 | documented |
| `OTEL_LOG_TOOL_CONTENT` | span 是否含工具输入输出内容 | documented |
| `OTEL_LOG_TOOL_DETAILS` | span 是否含工具参数等细节 | documented |
| `OTEL_LOG_RAW_API_BODIES` | trace 含原始请求体（可 `file:<dir>`） | documented |
| `OTEL_METRICS_INCLUDE_ACCOUNT_UUID` | metrics 是否含账号 uuid | documented |
| `OTEL_METRICS_INCLUDE_SESSION_ID` | metrics 是否含 session id | documented |
| `OTEL_METRICS_INCLUDE_VERSION` | metrics 是否含版本 | documented |
| **OTel SDK 透传** | | |
| `OTEL_METRICS_EXPORTER` | `otlp/prometheus/console/none` | documented |
| `OTEL_LOGS_EXPORTER` | 日志 exporter | documented |
| `OTEL_TRACES_EXPORTER` | 追踪 exporter（tracing beta 必填） | documented |
| `OTEL_EXPORTER_OTLP_PROTOCOL` | `grpc / http/protobuf / http/json` | documented |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | OTLP 端点 | documented |
| `OTEL_EXPORTER_OTLP_HEADERS` | OTLP 鉴权头 | documented |
| `OTEL_EXPORTER_OTLP_METRICS_ENDPOINT` / `_PROTOCOL` / `_TEMPORALITY_PREFERENCE` | metrics 信号专用 | documented |
| `OTEL_EXPORTER_OTLP_LOGS_ENDPOINT` / `_PROTOCOL` | logs 信号专用 | documented |
| `OTEL_EXPORTER_OTLP_TRACES_ENDPOINT` / `_PROTOCOL` | traces 信号专用 | documented |
| `OTEL_EXPORTER_OTLP_CERTIFICATE` / `_CLIENT_CERTIFICATE` / `_CLIENT_KEY` / `_METRICS_CLIENT_KEY` | gRPC mTLS | documented |
| `OTEL_METRIC_EXPORT_INTERVAL` | metrics 推送间隔（`60000`） | documented |
| `OTEL_LOGS_EXPORT_INTERVAL` | logs 推送间隔（`5000`） | documented |
| `OTEL_TRACES_EXPORT_INTERVAL` | traces 推送间隔 | documented |
| `OTEL_RESOURCE_ATTRIBUTES` | OTel 资源属性 | documented |
| **Tracing beta** | | |
| `CLAUDE_CODE_ENHANCED_TELEMETRY_BETA` | span 追踪开关 | experimental |
| `ENABLE_ENHANCED_TELEMETRY_BETA` | 上一项的**别名** | experimental |
| `ENABLE_BETA_TRACING_DETAILED` | 详细 span | experimental |
| `BETA_TRACING_ENDPOINT` | 详细 span endpoint | experimental |
| `TRACEPARENT` | **W3C trace 上下文**：Bash 子进程注入；SDK/`-p` 读 | documented |
| `TRACESTATE` | 与 `TRACEPARENT` 配套 | documented |
| **调试 / 日志** | | |
| `CLAUDE_CODE_DEBUG_LOGS_DIR` | 调试日志路径（**单独设没用，还得 `--debug`**） | documented |
| `CLAUDE_CODE_DEBUG_LOG_LEVEL` | `verbose/debug/info/warn/error` | documented |
| **子进程安全** | | |
| `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB` | 把 env 清洗后再传给子进程 | documented |
| `CLAUDE_CODE_SCRIPT_CAPS` | 每 session 限制特定脚本调用次数（依赖上一项） | documented |

---

## 2. 重要变量详解 + 默认值

### 2.1 思考预算（Effort & Thinking）

| 变量 | 默认值 | 取值 | 作用要点 |
|---|---|---|---|
| `CLAUDE_CODE_EFFORT_LEVEL` | 模型默认 | `low/medium/high/xhigh/max/auto` | 优先级高于 `/effort` 和 `effortLevel` 设置；用环境变量设的 `max` 会在 session 间持久化（其他来源只是 session 内）。 |
| `MAX_THINKING_TOKENS` | 模型相关 | 整数；`0` = 禁用 | 上限 = 模型最大输出 tokens − 1；**adaptive reasoning 模型默认会忽略**它，要起作用得同时设 `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING=1`。 |
| `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING` | — | `1` | 让 Opus 4.6 / Sonnet 4.6 退回固定预算；**对 Opus 4.7 完全无效**（4.7 始终 adaptive）。 |
| `CLAUDE_CODE_DISABLE_THINKING` | — | `1` | 强制关掉 extended thinking，比 `MAX_THINKING_TOKENS=0` 更直接。 |
| `DISABLE_INTERLEAVED_THINKING` | — | `1` | 跳过 interleaved-thinking beta 头，用于 gateway/provider 不支持时。 |

### 2.2 API & 网络（高频调参项）

| 变量 | 默认值 | 作用要点 |
|---|---|---|
| `API_TIMEOUT_MS` | `600000`（10 分钟） | 大于 `2147483647` 会溢出导致立刻失败。 |
| `CLAUDE_CODE_MAX_RETRIES` | `10` | API 失败重试次数。 |
| `ANTHROPIC_BASE_URL` | — | 改到非官方 base 时，MCP `tool_search` 默认会被禁；如 gateway 转发 `tool_reference`，再加 `ENABLE_TOOL_SEARCH=true`。 |
| `CLAUDE_CODE_EXTRA_BODY` | — | JSON 对象会合并进每次请求体，用来传 Claude Code 没暴露的 provider 专属字段。 |
| `CLAUDE_CODE_ATTRIBUTION_HEADER` | `1` | 系统 prompt 开头的归因块。对 gateway 缓存命中率有影响；对 Anthropic 官方缓存无影响。 |
| `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS` | — | 剥离 `anthropic-beta` 头和 `defer_loading` / `eager_input_streaming` 等 beta 字段；走会拒绝它们的 gateway 时用。 |
| `CLAUDE_CODE_DISABLE_NONSTREAMING_FALLBACK` | — | 流式失败后默认会走非流式重试一次。某些 proxy 会导致工具被重复执行，关闭它。 |
| `CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY` | off | 从 gateway `/v1/models` 拉模型列表填到 `/model`，会过滤为 `claude*` / `anthropic*`。要求 Anthropic Messages 格式，引入版本 `v2.1.129+`。 |
| `CLAUDE_ENABLE_BYTE_WATCHDOG` | on（仅 Anthropic API） | 字节级 idle watchdog；超过 `CLAUDE_STREAM_IDLE_TIMEOUT_MS`（最低 5 分钟）没新字节就断。 |

### 2.3 上下文窗口（最容易踩坑）

- `CLAUDE_CODE_MAX_CONTEXT_TOKENS` **只有同时设了 `DISABLE_COMPACT=1` 才生效**，单独设无效。
- 增大 `CLAUDE_CODE_MAX_OUTPUT_TOKENS` 会减少自动压缩前的有效上下文。
- `CLAUDE_CODE_AUTO_COMPACT_WINDOW` 是「自动压缩计算用的容量」，被截断到模型实际窗口；`CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` 是「达到容量百分之多少触发」。
- `DISABLE_COMPACT` 关全部（含手动 `/compact`），`DISABLE_AUTO_COMPACT` 只关自动。

### 2.4 后台任务、子 agent、effort 传递

- **正确变量名是 `CLAUDE_AUTO_BACKGROUND_TASKS`，不是 `CLAUDE_CODE_AUTO_BACKGROUND_TASKS`**。许多第三方清单写错。
- `CLAUDE_CODE_FORK_SUBAGENT=1` 启用 fork 子 agent，让 `/fork` 真正派生一个继承上下文的 subagent，而不是别名到 `/branch`。
- `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` 开 agent teams（实验性）；team 成员上会自动注入 `CLAUDE_CODE_TEAM_NAME`（只读）。
- `CLAUDE_CODE_DISABLE_AGENT_VIEW=1` 关掉所有后台 agent / agent 视图。
- `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS=1` 关 `run_in_background`、自动后台化、Ctrl+B 等所有后台功能。
- `CLAUDE_EFFORT` 是 Claude Code 注入到 Bash/hook 子进程的**只读**变量，反映当前回合的 effort level；仅当模型支持 effort 时才注入。

### 2.5 Prompt Caching（影响成本）

- `ENABLE_PROMPT_CACHING_1H=1`：把 cache TTL 从默认 5 分钟拉到 1 小时。
  - 适用于 API key、Bedrock、Vertex、Foundry、Claude Platform on AWS。
  - 订阅用户自动用 1 小时 TTL。
  - 1 小时写入按更高费率计费——**不是免费 buff**。
- `FORCE_PROMPT_CACHING_5M=1` 强制走 5 分钟，覆盖上面那个。
- `DISABLE_PROMPT_CACHING=1` 全局关；按模型粒度可用 `DISABLE_PROMPT_CACHING_OPUS/SONNET/HAIKU`。
- `ENABLE_PROMPT_CACHING_1H_BEDROCK` **已废弃**，统一用 `ENABLE_PROMPT_CACHING_1H`。

### 2.6 UI 渲染（长会话稳定性）

- `CLAUDE_CODE_NO_FLICKER=1` 走 fullscreen renderer（研究预览），减少闪烁，长会话内存增长更平。等价 `tui` 设置 / `/tui fullscreen`。
- `CLAUDE_CODE_DISABLE_ALTERNATE_SCREEN=1` 强制传统主屏渲染——**优先级高于 `CLAUDE_CODE_NO_FLICKER` 和 `tui` 设置**。
- `CLAUDE_CODE_DISABLE_VIRTUAL_SCROLL=1` 解决 fullscreen 模式下偶尔出现的空白滚动 bug，代价是渲染所有消息。

### 2.7 Hooks 注入（写 hook 脚本必看）

| 变量 | 说明 |
|---|---|
| `CLAUDE_PROJECT_DIR` | hook 的 `command`/`args` 中**字面替换**为项目根目录 |
| `CLAUDE_PLUGIN_ROOT` | 插件安装目录（每次更新会变） |
| `CLAUDE_PLUGIN_DATA` | 插件持久数据目录（更新不会丢） |
| `CLAUDE_ENV_FILE` | hook 可写文件：`SessionStart`/`Setup`/`CwdChanged`/`FileChanged` hook 在里面 `echo 'export X=1'`，下一条 Bash 命令前会 source。 |

### 2.8 调试日志

- `CLAUDE_CODE_DEBUG_LOGS_DIR` **单独设没有任何效果**，得配合 `--debug` 或 `/debug`；或直接 `--debug-file <path>` 一步到位。

### 2.9 子进程 env scrub

`CLAUDE_CODE_SCRIPT_CAPS`（JSON 限制特定脚本调用次数）**只有 `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB=1` 也设置时才生效**。匹配是子串匹配，`xargs` / `find -exec` 的 fan-out 检测不到，属于纵深防御而非硬保证。

---

## 3. 只读 / 注入型变量（不要自己设）

这些是 Claude Code **写给子进程或 hook 用**的，不应由用户手工设：

- `CLAUDECODE`（Bash/tmux 子进程标记 `=1`）
- `CLAUDE_CODE_SESSION_ID`、`CLAUDE_CODE_REMOTE`、`CLAUDE_CODE_REMOTE_SESSION_ID`、`CLAUDE_CODE_TEAM_NAME`
- `CLAUDE_EFFORT`（Bash/hook 子进程中反映当前 effort level）
- `TRACEPARENT` / `TRACESTATE`（W3C trace 上下文；Bash 子进程注入，SDK/`-p` 会从入参读取）
- `CLAUDE_PROJECT_DIR`、`CLAUDE_PLUGIN_ROOT`、`CLAUDE_PLUGIN_DATA`（hook 命令字符串里替换，不是常规 env）

---

## 4. 常见误传 vs 官方拼写

| 社区/旧文中常见错名 | 官方正确名 | 备注 |
|---|---|---|
| `CLAUDE_CODE_AUTO_BACKGROUND_TASKS` | `CLAUDE_AUTO_BACKGROUND_TASKS` | 没有 `_CODE_` |
| `CLAUDE_CODE_BASH_MAINTAIN_PROJECT_WORKING_DIR` | `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR` | 没有 `_CODE_` |
| `CLAUDE_CODE_CODE_ACCESSIBILITY` | `CLAUDE_CODE_ACCESSIBILITY` | 单 `CODE` |
| `ANTHROPIC_OAUTH_TOKEN` / `…REFRESH_TOKEN` / `…SCOPES` | `CLAUDE_CODE_OAUTH_TOKEN` / `…REFRESH_TOKEN` / `…SCOPES` | 前缀不同 |
| `CLAUDE_CODE_DEBUG_FILE` | `CLAUDE_CODE_DEBUG_LOGS_DIR` | CLI flag 是 `--debug-file`，环境变量名不同 |
| `ANTHROPIC_TIMEOUT_MS` | `API_TIMEOUT_MS` | 前缀不是 `ANTHROPIC_` |
| `CLAUDE_CODE_TELEMETRY_ENABLED` | `CLAUDE_CODE_ENABLE_TELEMETRY` | 后缀顺序不同 |
| `CLAUDE_CODE_USE_PWSH` | `CLAUDE_CODE_USE_POWERSHELL_TOOL` | 老命名已不存在 |

并且以下变量在很多第三方清单/AI 自动汇总中出现过，但**官方 env-vars 页面里搜不到**，应视为虚构：`CLAUDE_CODE_TOKEN_COUNTER_OVERRIDE`、`CLAUDE_CODE_SYSTEMD_NOTIFY`、`CLAUDE_CODE_TEMP_DIR`（注意：`CLAUDE_CODE_TMPDIR` 才存在）、`CLAUDE_CODE_VERTEX_REGION`、`CLOUD_METADATA_ADDR`、`OTLP_ENDPOINT`、`OTLP_HEADERS`、`OTEL_SDK_DISABLED`。

---

## 5. 推荐配置模板（写入 `~/.claude/settings.json`）

### 5.1 基础稳态

```json
{
  "env": {
    "CLAUDE_CODE_EFFORT_LEVEL": "high",
    "CLAUDE_CODE_NO_FLICKER": "1",
    "CLAUDE_CODE_MAX_RETRIES": "20",
    "API_TIMEOUT_MS": "1200000"
  }
}
```

### 5.2 喜欢用多 agent / fork

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1",
    "CLAUDE_CODE_FORK_SUBAGENT": "1",
    "CLAUDE_AUTO_BACKGROUND_TASKS": "1"
  }
}
```

### 5.3 走代理 / LiteLLM / gateway，被 beta 头拒

```json
{
  "env": {
    "CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS": "1",
    "CLAUDE_CODE_DISABLE_NONSTREAMING_FALLBACK": "1"
  }
}
```

### 5.4 想固定高 thinking 预算（Opus 4.6 / Sonnet 4.6）

```json
{
  "env": {
    "CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING": "1",
    "MAX_THINKING_TOKENS": "64000"
  }
}
```

> **注意**：对 Opus 4.7 无效；4.7 始终是 adaptive，思考预算由模型自己定。

### 5.5 企业/合规：极简 telemetry

```json
{
  "env": {
    "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1"
  }
}
```

等价 `DISABLE_AUTOUPDATER + DISABLE_FEEDBACK_COMMAND + DISABLE_ERROR_REPORTING + DISABLE_TELEMETRY`。

---

## 6. 版本相关注意点

- `VERTEX_REGION_CLAUDE_4_7_OPUS`：要求 Claude Code `v2.1.111+`。
- Bedrock 配置向导：`v2.1.94+`。
- Mantle 端点：`v2.1.94+`。
- Vertex 配置向导：`v2.1.98+`。
- Vertex 的 X.509 Workload Identity Federation：`v2.1.121+`。
- Gateway model discovery：`v2.1.129+`。
- Opus 4.7：一般要求 `v2.1.111+`。

env-vars 页面本身只在少数变量上打了最低版本标记，绝大部分变量没有显式版本号。

---

## 7. 一些行为细节备忘

- env-vars 页面只有一个 H2「Environment variables」+ 一张总表（约 246 个标识符），上面的分类是本文件做的归类。
- `settings.json` 的 `env` 字段每个 session 都生效，并会**强制传给子进程**，所以 `CLAUDE_CODE_MCP_ALLOWLIST_ENV` 与 `CLAUDE_CODE_SCRIPT_CAPS` 这类「为子进程定 policy」的变量适合写在这里。
- Bedrock / Foundry 优先级高于 Claude Platform on AWS；Bedrock + Mantle 可并存（按模型 id 路由）。
- Bedrock 与 Vertex 在 pinned 模型不可用时会启动 fallback；**Foundry 不会，直接报错**——团队部署务必显式 pin 模型版本。
- `CLAUDE_CODE_PROVIDER_MANAGED_BY_HOST` 会让 `settings.json` 里的 provider 字段被忽略（`USE_BEDROCK`、`ANTHROPIC_BASE_URL`、`ANTHROPIC_API_KEY` 等），但 CLI flag 和命令行 env 仍然生效；并会跳过 Bedrock/Vertex/Foundry 的自动 telemetry 退订。
