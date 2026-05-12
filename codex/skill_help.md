# Codex CLI Skills 教程

> **覆盖**：Codex CLI 的 Skill 系统（**是的，原生有！**）、AGENTS.md、MCP、Plugins、Subagents 等扩展点的安装/查看/禁用。
> **数据来源**：[`developers.openai.com/codex/skills`](https://developers.openai.com/codex/skills)、`/codex/guides/agents-md`、`/codex/mcp`、`/codex/plugins`、`/codex/subagents`、`/codex/config-basic`、`/codex/config-reference`、`/codex/cli/reference`、`/codex/cli/slash-commands`；[`openai/codex`](https://github.com/openai/codex) 仓库 `codex-rs/skills/`、`codex-rs/core-skills/src/loader.rs`、`.codex/skills/`。
> **采集日期**：2026-05-12 / 对应 `@openai/codex` `rust-v0.130.0`。

---

## 1. Codex 有 Skills 吗？——**有，原生系统**

是的。Codex CLI 自 v0.130.0 时已有**一等公民 Skills 系统**，设计与 Claude Code 高度对齐：

- 同样的文件名约定：`SKILL.md`（loader 常量 `SKILLS_FILENAME = "SKILL.md"`）
- 同样的 YAML frontmatter：`name` + `description` 必填
- 同样的 progressive disclosure：模型先看 metadata（约 8 KB 总预算），调用时才载正文
- 同样的多层装载根（user / project / system）
- 自带 `/skills` 槽位命令、`$skill-name` mention 语法、`[[skills.config]]` TOML 表
- 仓库 `codex-rs/skills/src/assets/samples/` 直接 ship 了 `skill-creator`、`skill-installer`、`plugin-creator`、`imagegen`、`openai-docs` 几个示范 skill

---

## 2. Skill 结构

跟 Claude Code 几乎一样：

```
my-skill/
├── SKILL.md           # 必需，YAML frontmatter + 正文
├── scripts/           # 可选，脚本
├── references/        # 可选，按需参考
├── assets/            # 可选，模板/数据
└── agents/
    └── openai.yaml    # 可选，Codex 特定 UI metadata
                       # display_name, short_description, icons,
                       # policy.allow_implicit_invocation,
                       # dependencies.tools
```

### 2.1 Frontmatter

按 [agentskills.io 开放规范](https://agentskills.io/specification) 必填 `name` + `description`，其它可选。Codex 特有的 `agents/openai.yaml` 是给 UI 用的 metadata（图标、显示名、隐式调用策略、依赖工具声明）。

---

## 3. Skill 装载位置（specificity 高的胜出）

| 优先级 | 名称 | 路径 |
|---|---|---|
| 1 | **repo local** | `./.agents/skills/`（从 cwd 向上走到 repo root）|
| 2 | **repo root** | `$REPO_ROOT/.agents/skills/` |
| 3 | **user** | `$HOME/.agents/skills/` 与 `$CODEX_HOME/skills/`（默认 `~/.codex/skills/`）|
| 4 | **admin** | `/etc/codex/skills/` |
| 5 | **system** | 仓库 bundled（`skill-creator`、`skill-installer`、`plugin-creator`、`imagegen`、`openai-docs`）|

注意 Codex 同时认 `~/.agents/skills/`（开放规范路径）和 `~/.codex/skills/`（Codex 私有路径）——前者是为了和 `npx skills`、其它 agent 跨工具复用。

---

## 4. 安装 / 启用

### 4.1 手动

```bash
mkdir -p ~/.codex/skills/my-skill
$EDITOR ~/.codex/skills/my-skill/SKILL.md
```

或项目级：

```bash
mkdir -p ./.agents/skills/my-skill
$EDITOR ./.agents/skills/my-skill/SKILL.md
```

### 4.2 用内置 `skill-installer` skill（在 Codex 会话里）

```text
$skill-installer install <github-org/repo>
```

`skill-installer` 是 Codex 自带的 system skill，能拉远程仓库并装到默认位置。

### 4.3 用 `npx skills`（Vercel Labs 第三方，**支持 Codex**）

`skills` npm 包的 supported-agents 列表里**明确写了 `codex`**。

```bash
npx skills add some-org/repo -a codex       # 项目级 → ./.agents/skills/
npx skills add some-org/repo -a codex -g    # 全局 → ~/.codex/skills/
npx skills add some-org/repo -a '*'         # 同时装到所有 agent
```

`-g` 写到 `~/.codex/skills/` 或 `~/.agents/skills/`——两个都是 Codex 装载路径。默认 symlink，`--copy` 拷贝。**不修改 `config.toml`**——纯放文件。

### 4.4 还有个独立的 `codex-skills` 包

`npm view codex-skills` → `codex-skills` v2.0.0（jMerta 第三方），装到 `~/.agents/skills/`——也是 Codex 认的根。但**不是 OpenAI 官方**，注意分辨。

### 4.5 MCP server（不是 skill 但常和 skill 配套）

```bash
codex mcp add <server-name> -- <command> [args...]
```

或直接编辑 `~/.codex/config.toml`：

```toml
[mcp_servers.context7]
command = "npx"
args = ["-y", "@upstash/context7-mcp"]
env_vars = ["LOCAL_TOKEN"]
[mcp_servers.context7.env]
MY_ENV_VAR = "MY_ENV_VALUE"

[mcp_servers.figma]
url = "https://mcp.figma.com/mcp"
bearer_token_env_var = "FIGMA_OAUTH_TOKEN"
http_headers = { "X-Figma-Region" = "us-east-1" }
```

常用可选 key：`enabled`、`required`、`startup_timeout_sec`、`tool_timeout_sec`、`enabled_tools`、`disabled_tools`。

### 4.6 AGENTS.md

层级加载（高 → 低）：

```
~/.codex/AGENTS.override.md
~/.codex/AGENTS.md
<from git root down to cwd>:
  each level: AGENTS.override.md → AGENTS.md → project_doc_fallback_filenames
```

各层按空行拼接；离 cwd 近的覆盖远的。硬上限 `project_doc_max_bytes = 32 KiB`。`/init` 在当前目录脚手架一份 `AGENTS.md`。

### 4.7 Plugins（meta-bundler）

Plugin 可以同时打包 **skills + apps + MCP servers**。用 `/plugins` 槽位命令或 Codex 应用的 Plugin Directory 管理。这是给团队分发整套环境最快的方式。

### 4.8 Profiles

`[profiles.<name>]` 表覆盖顶层字段（`model`、`web_search`、`personality`、`sandbox_mode`、`approval_policy` 等）。激活：

```bash
codex --profile review
```

或在 `config.toml` 顶层 `profile = "review"` 设默认。

### 4.9 Subagents

`~/.codex/agents/` 或 `.codex/agents/` 下放 agent 文件，定义 `model`、sandbox、instructions、per-skill enable/disable。并发上限：`agents.max_threads` 默认 6，`agents.max_depth` 默认 1。

### 4.10 Features 开关

```bash
codex features enable multi_agent
codex features disable web_search
codex features                          # 列出
```

写到 `config.toml` 的 `[features]` 表。

---

## 5. 查看当前 agent 可见的扩展

| 看什么 | 怎么看 |
|---|---|
| Skills | 会话内 `/skills`（TUI） |
| Plugins | 会话内 `/plugins`（TUI） |
| MCP servers + 工具 | 会话内 `/mcp`；CLI `codex mcp`（list/add/remove/authenticate）|
| Feature flags | `codex features` |
| AGENTS.md 链 | 没有专用 dump 命令；最简单：会话里问 agent 「Summarize current instructions」 |
| Subagents | `/agent`（TUI）|

**没有** `codex doctor` / `codex config show` / `--print-config`——这是 Codex 对比 Claude Code 的一个明显 ergonomic 缺口。要审 config 只能直接读 `~/.codex/config.toml` + `.codex/config.toml`。

---

## 6. 禁用 / 关闭扩展

### 6.1 单个 Skill

写到 `~/.codex/config.toml`：

```toml
[[skills.config]]
path = "/absolute/path/to/skill"     # 或：name = "skill-name"
enabled = false
```

### 6.2 MCP server

```toml
[mcp_servers.figma]
enabled = false    # 不删，先停用
```

或直接删 `[mcp_servers.<id>]` 块。

### 6.3 AGENTS.md

- 删 / 重命名文件
- 或在更高层放 `AGENTS.override.md` 覆盖

### 6.4 Plugin

在 plugin 配置里设 `enabled = false`——保留安装但停用。

### 6.5 Profile

省略 `--profile` 或切到别的。

### 6.6 Feature

```bash
codex features disable <feature>
```

---

## 7. `npx skills` 与 Codex 关系

**已确认兼容**。`skills` v1.5.6（vercel-labs）的 supported-agents 包含 `codex`，`-a codex` 写到 `.agents/skills/`（项目）或 `~/.codex/skills/`（全局）——都是 Codex 文档化的装载根。

```bash
npx skills add owner/repo -a codex          # 项目
npx skills add owner/repo -a codex -g       # 全局
npx skills add owner/repo -a '*'            # 所有支持的 agent
npx skills remove some-skill -a codex
npx skills list                              # 只列 npx skills 装过的
```

Codex 内置 loader 不知道 `npx skills` 的存在——它只看到 `~/.codex/skills/foo/SKILL.md` 就加载。所以**装完后用 Codex 自己的 `/skills` 验证**，看是否进了 listing。

---

## 8. 2026 最佳实践

1. **repo 根放一份 AGENTS.md** 当跨工具基线（Codex / Cursor / Claude Code 都读）。控制在 200 行以内。
2. **`AGENTS.override.md` 只放本地临时覆盖**，不 commit。
3. **重复用的「过程 + 脚本」从 AGENTS.md 抽出来变 skill**。skill 适合：有顺序的步骤 + 附带脚本/资源。
4. **skill 优于 MCP server**——只要工作是 prompt + script 就用 skill（零进程）。MCP 留给需要长连接 + auth 的（Figma、GitHub）。
5. **profile 用来切换 model / sandbox / approval-policy 套件**（如 `review` profile 设 `sandbox_mode = "read-only"`），**别**用 profile 切换 skill 集合——那是 `[[skills.config]]` 该干的。
6. **`description` 控制在 ~200 字以内、明确 scope**。Codex 整体 skill listing 预算 ~8 KB，太啰嗦会被截断，隐式调用准确率下降。
7. **团队分发用 Plugin**——一个安装搞定 skills + MCP 注册 + app connection。
8. **新 skill 先扔 `./.agents/skills/` 沙箱测**，验证行为后再 promote 到 `~/.codex/skills/` 全局。
9. **conflict 避免**：同名 skill 不要分散在多层。Codex specificity 规则会让近的覆盖远的——很容易自己踩自己。

---

## 9. 坑

- `developers.openai.com/codex/` 的 URL 路径**不直观**（如 `/codex/guides/agents-md`、`/codex/cli/reference`、`/codex/cli/slash-commands`）。猜路径会 404，从 `/codex/` 用侧栏走。
- **`~/.codex/.env` 过滤 `CODEX_*` 变量**这条没在公开文档里见到（之前 env.md 里的描述是从源码 `codex-rs/arg0/src/lib.rs` 看到的）。设置 env 用真的 shell env。
- **`CODEX_CONNECTORS_TOKEN`** 在公开文档里没找到——Apps/Connectors 的 auth 走 Plugin 安装 UI，不是单一 bearer token。
- **没有 `codex doctor` / `codex config show`**——审 config 只能直接读 TOML + 用 TUI 的 `/skills`、`/plugins`、`/mcp`、`/agent`。
- npm 上 `codex` 是高度过载的关键词，第三方 `codex-skills` **不是 OpenAI**——OpenAI 官方只发 `@openai/codex`。
- 我**没有跑通**真实的 plugin install end-to-end，文档描述与实际 UI 可能有差。
