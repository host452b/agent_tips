# cursor-agent CLI Skills 教程

> **覆盖**：Cursor 2.4（2026-01-22 发布）后的 Skills 系统（`cursor-agent` CLI **原生支持**）、Rules、AGENTS.md、MCP 配置；安装/查看/禁用；与 `npx skills` 的关系；2026 最佳实践。
> **数据来源**：[`cursor.com/docs/skills`](https://cursor.com/docs/skills)、`/docs/rules`、`/docs/cli/using`、`/docs/cli/reference/configuration`、`/changelog/2-4`、`/blog/agent-best-practices`，外加 cursor 论坛关于 Skills + Rules 统一化的讨论帖。
> **采集日期**：2026-05-12。

---

## 1. cursor-agent 有 Skills 吗？——**有（2.4 起）**

是的。Cursor **2.4**（2026-01-22）changelog 明确写：
> "Cursor now supports Agent Skills in the editor and CLI."

即 IDE 和 `cursor-agent` 命令行都吃 Skill。系统设计是 Claude Code Skills 的 1:1 类比：

- `SKILL.md` + YAML frontmatter
- 多个装载根目录
- `name` + `description` 必填
- 还可以**直接读取 `.claude/skills/` 这种 legacy 路径**，做跨工具复用

⚠️ **版本要求**：`cursor-agent --version` 必须 ≥ 2.4。早期版本静默忽略 `~/.cursor/skills/` 等目录。

---

## 2. Cursor 的扩展面（多套并存）

### 2.1 Skills（推荐用，2.4+）

装载根（**Cursor 自家的**）：

| 路径 | 作用域 |
|---|---|
| `.agents/skills/` | 项目 |
| `.cursor/skills/` | 项目 |
| `~/.agents/skills/` | 用户（全局）|
| `~/.cursor/skills/` | 用户（全局）|
| `.claude/skills/`、`.codex/skills/`（以及 `~/` 同名） | **legacy / 跨工具兼容** |

Frontmatter：

| 字段 | 必需 | 说明 |
|---|---|---|
| `name` | ✅ | 小写字母 + 数字 + 连字符 |
| `description` | ✅ | WHAT + WHEN to use（模型据此决定相关性）|
| `paths` | 否 | glob，把 skill scope 到匹配文件 |
| `license` | 否 | 名称或捆绑文件引用 |
| `compatibility` | 否 | 环境要求（依赖包、网络）|
| `metadata` | 否 | 任意 KV |
| `disable-model-invocation` | 否 | `true` → 只接受用户 `/skill-name` |

Skill 可以带 `scripts/` 子目录；正文里相对路径引用，调用时执行。**嵌套在项目子目录里的 skill 自动 scoped 到那棵子树**。

### 2.2 Rules（仍然支持，但 2.4 推荐转 Skills）

`.cursor/rules/*.mdc` 四种生效方式：

| 模式 | frontmatter |
|---|---|
| **Always**（全程注入）| `alwaysApply: true` |
| **Auto Attached**（glob 触发）| `alwaysApply: false` + `globs: "src/**/*.tsx"` |
| **Agent Requested**（描述驱动）| `alwaysApply: false` + 有 `description:` |
| **Manual**（`@my-rule` 触发）| 都没有 |

可用字段：`description`（string）、`globs`（string 或 array）、`alwaysApply`（boolean）。也可以是无 frontmatter 的 `.md`（纯散文）。位置：

- Project Rules：`.cursor/rules/`（commit 到 git）
- User Rules：全局
- Team Rules：Cursor dashboard 推送

**官方建议**：动态 rule 和 slash command **迁移到 Skills**（提供了 `/migrate-to-skills` 命令）。Rule 适合做「永远生效的护栏」或「按路径触发的风格指南」。

### 2.3 AGENTS.md

CLI 文档明文：**`cursor-agent` 把 `AGENTS.md` 和 `CLAUDE.md` 作为 rule 加载到 `.cursor/rules` 旁边**。位置：项目根，纯 markdown。

### 2.4 `.cursorrules`（legacy）

向后兼容仍工作，但所有新工具都指向 `.cursor/rules/*.mdc` 或 `AGENTS.md`。**不要给新项目用**。

### 2.5 MCP server

CLI 文档显式：**`cursor-agent` 自动检测 `mcp.json` 并复用 IDE 配的同一批 MCP**。所以 MCP 在 CLI 是**一等公民**（不是只有 IDE 能用，旧资料里有时会写错）。

### 2.6 `cli-config.json`

- 全局：`~/.cursor/cli-config.json`（macOS/Linux）/ `%USERPROFILE%\.cursor\cli-config.json`（Windows）。可被 `CURSOR_CONFIG_DIR` 覆盖；Linux/BSD 上认 `XDG_CONFIG_HOME`。
- 项目：`<repo>/.cursor/cli.json`——**只放 permissions**。
- 字段：`version`（1）、`editor.vimMode`、`permissions.allow[]`、`permissions.deny[]`、`model`、`hasChangedDefaultModel`、`network.useHttp1ForAgent`、`attribution.*`。
- 纯 JSON 无注释；自修复缺字段；坏文件改名为 `.bad` 并重置。
- **没有 `skills` / `rules` 键**——它们是磁盘上的文件，不是 JSON 里的字段。

---

## 3. 安装 / 启用 skill

### 3.1 手动

```bash
mkdir -p ~/.cursor/skills/summarize-changes
$EDITOR ~/.cursor/skills/summarize-changes/SKILL.md
```

项目级：

```bash
mkdir -p .cursor/skills/summarize-changes
$EDITOR .cursor/skills/summarize-changes/SKILL.md
```

### 3.2 IDE UI 加 GitHub Skill

IDE 里：**Settings → Rules → Project Rules → Add Rule → Remote Rule (GitHub)**，粘贴 repo URL。2.4 起这个 UI 也管 skill。

### 3.3 `npx skills`（Vercel Labs 第三方，**对 cursor 友好**）

`npx skills` 在 supported-agents 列表里包含 `cursor`。

```bash
npx skills add some-org/repo -a cursor      # 项目级 → ./.cursor/skills/
npx skills add some-org/repo -a cursor -g   # 全局 → ~/.cursor/skills/
npx skills add some-org/repo -a '*'         # 所有 agent
```

**特殊优势**：Cursor 2.4 显式读 `~/.claude/skills/` 与 `.claude/skills/`——所以**用 `npx skills add owner/repo`（即默认 `-a claude-code`）装到 Claude Code 目录的 skill，Cursor 也能直接载**，前提是 frontmatter 字段 Cursor 认识（`name`、`description`、`paths`/`globs`、`disable-model-invocation`）。Anthropic 私有字段会被静默忽略。这是**目前跨工具复用 skill 最无痛的路径**。

### 3.4 没有 marketplace

Cursor 没有官方 skill marketplace。社区索引（lobehub、agensi 等）是**目录**，不是 registry。

---

## 4. 查看当前 agent 可见的 skill / rule

**CLI 没有 `cursor-agent rules list` / `--list-rules`**。可用路径：

| 方法 | 说明 |
|---|---|
| IDE | Settings → Rules，2.4+ 在 "Agent Decides" 区也列出 skill |
| 文件系统 | `ls .cursor/rules ~/.cursor/skills .cursor/skills ~/.agents/skills .agents/skills .claude/skills` |
| AGENTS.md | 直接 `cat AGENTS.md`、`cat CLAUDE.md` |
| 问 agent | 在会话里直接问「what rules and skills are loaded」——它看得到自己的注入内容 |

---

## 5. 禁用

### 5.1 单个 skill：让模型看不到自动调用入口

frontmatter 加：

```yaml
disable-model-invocation: true
```

变成「只能 `/skill-name` 显式调」的状态。

### 5.2 单条 rule

- `alwaysApply: false` + 删除 `globs` + 模糊化 description → 不被 auto-attach
- 或直接删/改名 `.mdc` 文件

### 5.3 项目级整体关

删/移走 `.cursor/rules/`、`.cursor/skills/`、`AGENTS.md`、`CLAUDE.md`。

### 5.4 CLI flag 关？

**没有 `--no-rules` / `--no-skills`**。published CLI reference 里不存在。要「干净 session」只能：

- 移走文件
- 或 `cursor-agent --worktree --workspace <empty-dir>`（git worktree 干净状态）

---

## 6. `npx skills` 与 cursor-agent 关系

完全可用，**而且最优雅**：因为 Cursor 2.4 同时读 `~/.cursor/skills/` 与 `~/.claude/skills/`，任何一个目录里的 skill 都能用。最佳实践：

- 跨 Claude Code 与 cursor 复用：装到 `~/.claude/skills/`（`npx skills add owner/repo -g` 默认 agent 是 claude-code）
- 只给 cursor：`npx skills add owner/repo -a cursor -g`

Cursor 加载时只挑 frontmatter 里它认识的字段（`name`/`description`/`paths`/`globs`/`disable-model-invocation`），其它字段忽略不报错。

---

## 7. 2026 最佳实践

1. **过程类工作用 Skills，不用 Rules**——Cursor 2.4 官方建议如此。Rules 只留给「永远生效的护栏」和「按 glob 触发的风格指南」。
2. **repo 根放一份 `AGENTS.md`** 当跨工具基线（Codex / Cursor / Claude Code 同时读）。≤200 行。
3. **项目 skill 放 `.cursor/skills/`，全局 skill 放 `~/.cursor/skills/`**——和 IDE UI 列表保持同步。
4. **跨 Claude Code 复用的 skill 放 `~/.claude/skills/`**——Cursor 通过 legacy 路径别名加载，零移植。
5. **Rule 配 `globs:` 限制作用域**，别 `alwaysApply: true` 套全项目——上下文预算被 always-on rule 吃光是 #1 翻车原因。
6. **可重复执行的动作打包为 skill 里的 `scripts/foo.sh`**，别每次都口述。
7. **`.cursor/cli.json` 提交到 git**（只放 permissions），让 CI 跟人用同一份 allowlist。
8. **`/migrate-to-skills`** 把存量动态 rule 和 ad-hoc 槽位命令迁移过去。

---

## 8. 坑

- **2.4 之前没有 Skills**。`cursor-agent --version` 必须 ≥ 2.4（2026-01-22）。
- **CLI 没有 lister**。审清单只能问 agent 或读文件系统。
- **没有 `--no-rules` 逃生**。要无 rule session 只能改文件或 worktree。
- **两个 skill 根 + 两个作用域 = 命名冲突未文档化**。Cursor 没公开 `~/.cursor/skills/foo/` 与 `.cursor/skills/foo/` 的优先级——**避免跨作用域同名**。
- **`alwaysApply: true` 无 `globs` 是上下文杀手**。每条 prompt 都会注入这条 rule。
- **`cli-config.json` 是纯 JSON，不带注释**。坏掉会被改名 `.bad` 并重置——**别在里面放 secret**。
- **`.cursor/cli.json` 项目级只控 permissions**，model / vim mode / attribution 都得在全局文件里——团队复现要靠 commit rules/skills/AGENTS.md，不是 `cli.json`。
- **`AGENTS.md` 和 `.cursor/rules` 都生效**——同一条规则在两边都写是上下文双倍消耗，是常见的自食恶果。一条规则一个家。
