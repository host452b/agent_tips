# Claude Code Skills 教程

> **覆盖**：Skill 文件结构、装载顺序、安装/启用方法（含 `npx skills`）、运行时调用、查看清单、禁用方式、2026 年最佳实践。
> **数据来源**：[`code.claude.com/docs/en/skills`](https://code.claude.com/docs/en/skills)、`/plugins`、`/discover-plugins`、`/settings`，[`agentskills.io/specification`](https://agentskills.io/specification)，本机 `claude` 二进制中 strings 反推的 loader 代码，本地 `~/.claude/skills/` 与 `superpowers` 插件实例，`npx -y skills --help`（v1.5.6）实测。
> **采集日期**：2026-05-12。

---

## 1. Skill 是什么

一个 Skill 是**一个目录**，里面必须有一份 `SKILL.md`（loader 严格匹配 `/^skill\.md$/i`，但在文件系统大小写敏感时只认 `SKILL.md` 大写形式优先）。

```
my-skill/
├── SKILL.md           # 必需。YAML frontmatter + Markdown 正文
├── scripts/           # 可选，Claude 可执行的脚本
├── references/        # 可选，按需加载的参考文档
├── assets/            # 可选，模板/图片/数据
└── …                  # 其它任意文件
```

### 1.1 Frontmatter

按 [agentskills.io 开放规范](https://agentskills.io/specification)，**只有 `name` 和 `description` 是强制的**。Claude Code 自己更宽松——`name` 没有也能用（用目录名兜底），但 `description` **强烈推荐**：模型靠它决定何时调用 Skill。

Claude Code 完整字段（开放规范的超集）：

| 字段 | 必需 | 说明 |
|---|---|---|
| `name` | spec 必需；CC 可省 | 小写字母/数字/连字符；≤64 字符；与目录名一致 |
| `description` | 推荐 | **WHEN to use**，不是 WHAT it does。与 `when_to_use` 合并截断到 1536 字符 |
| `when_to_use` | 否 | 附加触发条件，追加到 description |
| `argument-hint` | 否 | 自动补全提示，如 `[issue-number]` |
| `arguments` | 否 | 命名位置参数，用于 `$name` 模板替换 |
| `disable-model-invocation` | 否 | `true` → 模型不可调，仅用户 `/skill-name` |
| `user-invocable` | 否 | `false` → `/` 菜单不显示，仅模型可调 |
| `allowed-tools` | 否 | 此 Skill 激活时预批准的工具 |
| `model` | 否 | 覆盖本 turn 的模型 |
| `effort` | 否 | `low/medium/high/xhigh/max` |
| `context` | 否 | `fork` 让 Skill 在子 agent 上下文里跑 |
| `agent` | 否 | 配 `context: fork` 用的 subagent 类型 |
| `hooks` | 否 | Skill 作用域内的 hooks |
| `paths` | 否 | glob 模式；只有匹配文件被触碰时自动加载（条件式 Skill） |
| `shell` | 否 | `bash`（默认）或 `powershell`，决定 `` !`cmd` `` 用哪个 shell |
| `license` / `compatibility` / `metadata` | 否 | 开放规范字段，CC 接受但大多不处理 |

实战范例（来自本机 `superpowers/using-superpowers/SKILL.md`）：

```yaml
---
name: using-superpowers
description: Use when starting any conversation - establishes how to find and use skills, requiring Skill tool invocation before ANY response including clarifying questions
---
```

### 1.2 正文

普通 Markdown，外加两个扩展：

- **动态上下文**：`` !`<shell command>` `` 内联，或 ` ```!` 围栏块。命令在 Skill 内容送给模型**之前**执行，stdout 替换原位。这是**预处理**，不是工具调用。
- **字符串替换**：`$ARGUMENTS`、`$ARGUMENTS[N]`、`$N`、`arguments:` 里定义的命名 `$arg`、`${CLAUDE_SESSION_ID}`、`${CLAUDE_EFFORT}`、`${CLAUDE_SKILL_DIR}`。

### 1.3 生命周期

```
Session 启动:
  Loader 扫描所有 skill 根目录 → 按 realpath 去重 →
  解析每份 SKILL.md frontmatter →
  把 {name + description + when_to_use} 注入系统提示词
  (单 skill 1536 字符上限) → 这叫 "skill listing"
  ⚠️ Skill 正文还没装载
  
有 `paths:` 的 skill 暂不激活，等模型碰到匹配文件再触发

会话中:
  Claude 决定调用 Skill → Skill 工具调用
  或用户 `/skill-name` →
  正文渲染（替换变量 + 跑 `!`cmd`）→ 作为一条消息进入对话
  Skill 内容会留在上下文里整个 session

自动压缩时:
  每个 skill 最近一次调用的内容会被重附在压缩摘要后
  单 skill 前 5000 token，总预算 25000 token，溢出从旧的开始丢
```

---

## 2. Skill 装载位置与优先级

`claude` 二进制 loader 反推出的精确顺序（**从高到低优先**，同名时后加载覆盖先加载，**插件 skill 例外——它们带命名空间永不冲突**）：

| # | 名称 | 路径 | 开关 |
|---|---|---|---|
| 1 | policy / 托管 | `<managed_dir>/skills/` | `CLAUDE_CODE_DISABLE_POLICY_SKILLS=1` 跳过 |
| 2 | user | `~/.claude/skills/` | `projectSettings` + `skillsLocked` 控制 |
| 3 | project | `<repo>/.claude/skills/` | `projectSettings` 控制 |
| 4 | --add-dir 目录 | `<每个 --add-dir>/.claude/skills/` | 默认参与 |
| 5 | legacy 命令 | `.claude/commands/*.md` | 仍工作，同名时 skill 胜出 |
| 6 | 插件 skill | `<plugin>/skills/<name>/SKILL.md` | 命名空间 `plugin-name:skill-name` |

**`--bare` / `CLAUDE_CODE_SIMPLE=1`**：跳过 user 与 policy 层；只加载 `--add-dir` 里的 skills；插件 skill 必须 `--plugin-dir` 显式带上。`/skill-name` 仍可触发。

---

## 3. 安装 / 启用方式

### 3.1 手动（最直接）

```bash
mkdir -p ~/.claude/skills/summarize-changes
$EDITOR ~/.claude/skills/summarize-changes/SKILL.md
```

`~/.claude/skills/`、`<repo>/.claude/skills/`、`--add-dir` 目录是被 watch 的——增删改即时生效，不用重启。**新建顶层 skills 目录**才需要重启。

### 3.2 插件 marketplace（`/plugin` 命令）

```text
/plugin marketplace add anthropics/claude-code           # owner/repo / git URL / 本地路径 / 远程 .json
/plugin install commit-commands@anthropics-claude-code   # 也可走 UI
/plugin disable commit-commands@anthropics-claude-code
/plugin uninstall commit-commands@anthropics-claude-code
/reload-plugins                                          # 不重启就生效
```

CLI 等价：

```bash
claude plugin install <name> --scope user|project|local
claude plugin disable <name>
claude plugin uninstall <name>
claude plugin marketplace add|list|remove|update
claude plugin validate <path>
claude plugin details <name>
```

官方 marketplace `claude-plugins-official` 默认已加。Demo 版 `claude-code-plugins`（`anthropics/claude-code` 仓库本身）要手动加。**临时**使用：`claude --plugin-dir ./my-plugin` 或 `claude --plugin-url https://…/my-plugin.zip`。

团队级安装走 `.claude/settings.json` 的 `extraKnownMarketplaces` + `enabledPlugins`。

### 3.3 `npx skills` —— **Vercel Labs 第三方，不是 Anthropic 官方**

- `npm view skills` → `skills` v1.5.6，作者 `rauchg`（Guillermo Rauch）+ `quuu`，仓库 `github.com/vercel-labs/skills`。
- `npm view @anthropic-ai/skills` **→ 404**。Anthropic 没有发布同名包。
- 价值：**跨多个 agent CLI 一键铺 skill**，不是与 Claude 深度集成。

子命令（来自实测 `npx -y skills --help` v1.5.6）：

```
skills add <package>        # 别名 a；支持 owner/repo、Git URL、本地路径
skills remove [skills]      # 别名 rm；不带参数走交互
skills list                 # 别名 ls
skills find [query]
skills update [skills…]     # 别名 upgrade
skills init [name]          # 生成 <name>/SKILL.md 模板
skills experimental_install # 从 skills-lock.json 恢复
skills experimental_sync    # 从 node_modules 同步到 agent 目录

主要 flag:
  -g, --global              # 装到用户级目录（CC 为 ~/.claude/skills）
  -p, --project             # 装到项目级（默认）
  -a, --agent <agents>      # 目标 agent，"claude-code cursor codex" 或 "*"
  -s, --skill <skills>      # 指定 skill 名或 "*"
  -l, --list                # 干跑预览
  --copy                    # 拷贝代替默认的 symlink
  --all                     # 等价 --skill '*' --agent '*' -y
  -y                        # 非交互
```

**它把文件写到哪**：
- 项目作用域（默认）：`./.claude/skills/<skill>/`（其它 agent：`./.cursor/skills/`、`./.codex/skills/`）
- 全局（`-g`）：`~/.claude/skills/<skill>/`
- **默认是 symlink** 到 `node_modules/<pkg>/skills/<name>/`；`--copy` 改成拷贝。
- **不改 `settings.json`**——纯粹移动文件。卸载时撤销 symlink/拷贝。

换句话说：`npx skills` 能做的事手动也能做。它的卖点是**跨 agent 批量装**和 `skills-lock.json` 复现。

```bash
# 典型用法
npx skills add some-org/awesome-skills            # 装到当前项目 .claude/skills/
npx skills add some-org/awesome-skills -g -a '*'  # 全局装到所有支持的 agent
npx skills list                                    # 列出 npx skills 自己装过的
npx skills remove awesome-skills                   # 卸载
```

---

## 4. 运行时调用

两条路：

1. **模型调用**（通过内建 `Skill` 工具）：session 开始时模型只看到每个 skill 的 `{name, description, when_to_use}`（不是正文）。模型决定要用时 `Skill(<name>, args)`，正文才被注入对话。
   - `disable-model-invocation: true` 的 skill **不在模型看到的清单里**，模型根本调不到。

2. **用户调用**：在 prompt 里键入 `/skill-name [args]`。不受 `disable-model-invocation` 限制。`user-invocable: false` 让它从 `/` 菜单消失，但模型仍可调。

`/init`、`/review`、`/security-review` 等内建命令其实就是 Skill 实现的。

**custom commands 与 skills 已合并**：`.claude/commands/deploy.md` 和 `.claude/skills/deploy/SKILL.md` 都产生 `/deploy`。同名时 skill 胜出。

---

## 5. 查看当前 agent 可见的 skill

没有 `claude skills list` 子命令。可用路径：

| 方法 | 说明 |
|---|---|
| `/skills`（会话内） | 交互式 skill 菜单。空格切状态，回车保存（写到 `.claude/settings.local.json` 的 `skillOverrides`）|
| 直接问 Claude | 「列出当前可见的 skill」——它会读自己的 skill listing 给你 |
| `/doctor` | 报告 skill listing 上下文预算是否溢出、哪些被截断 |
| 文件系统 | `ls ~/.claude/skills/ <repo>/.claude/skills/ ~/.claude/plugins/cache/*/<plugin>/skills/` |
| 插件 skill | `claude plugin details <plugin>` 看组件清单与预计 token 成本；或 `/plugin` 的 **Installed** 标签 |
| `npx skills` | `npx skills list` 只列**它自己装过的** |

`claude --help` 里**没有** `--list-skills` 之类 flag。

---

## 6. 禁用 skill（4 个粒度，从粗到细）

| 粒度 | 手段 |
|---|---|
| **本 session 全部关** | `claude --disable-slash-commands`（help 里写着 "Disable all skills"）；或在 `/permissions` 里 deny `Skill` 工具 |
| **只关 policy / 托管层** | `CLAUDE_CODE_DISABLE_POLICY_SKILLS=1`（二进制中已验证；只影响 policy 层，user/project/插件仍生效）|
| **禁止 skill 里的 `!`cmd`` 执行** | `"disableSkillShellExecution": true`（写在 settings；适合企业 managed）。每个 `` !`cmd` `` 被替换为 `[shell command execution disabled by policy]`。**bundled 与 managed skill 例外**。|
| **特定 skill** | `/permissions` 用 `Skill(name)` 匹配规则允许/拒绝。`Skill(commit)` 允许；`Skill(deploy *)` 拒绝 |
| **从 settings 控可见性** | `skillOverrides` 字段，值：`on` / `name-only`（隐藏 description）/ `user-invocable-only`（隐藏对模型）/ `off`（全隐）。**不影响插件 skill** |
| **skill 自己声明** | frontmatter 加 `disable-model-invocation: true` 或 `user-invocable: false` |
| **整个插件** | `/plugin disable <plugin>@<marketplace>` 或 `claude plugin uninstall …`。`/plugin marketplace remove` 还会清掉该 marketplace 下所有插件 |
| **不信任的项目 skill** | 不接受 workspace trust 弹窗；或对该 skill 名加 `skillOverrides: off` |

> **2026-05 现状提醒**（issue `anthropics/claude-code#14920`）：要**单独**禁用某个**插件里的某个 skill**（不整插件 disable），目前还没有 UI 开关——只能用 `/permissions` 写一条 `Skill(plugin:name *)` deny 规则。

---

## 7. 2026 年最佳实践

来自 `superpowers/writing-skills/SKILL.md` 与官方 anthropic-best-practices 文档。

### 7.1 写 skill 的规则

- **`description` 写 WHEN，不要写 WHAT**。开头用 "Use when…"。如果 description 里塞了流程总结，Claude 会读到一半觉得「我懂了」直接跳过正文。
  - 反例：`description: Use when executing plans - dispatches subagent per task with code review between tasks`
  - 正例：`description: Use when executing implementation plans with independent tasks in the current session`
- **`SKILL.md` 控制在 500 行 / 5000 token 以内**。重量级参考放 `references/`，脚本放 `scripts/`，模板放 `assets/`——progressive disclosure 是 skill 存在的全部理由。
- **token 预算**：getting-started 类 < 150 words；高频加载 skill < 200 words；其它 < 500 words。**「上下文窗口是公共资源」**。
- **命名动词先行、active voice、gerund 友好**：`creating-skills`、`condition-based-waiting`、`using-git-worktrees`——不要 `skill-creation` / `async-test-helpers`。CSO（Claude Search Optimization）靠 description + 正文关键词。
- **一个好例子 > 五个中等例子**。不要为了「全面」翻成 5 种语言。
- **交叉引用，别 `@`-include**。用散文 `**REQUIRED BACKGROUND:** Use superpowers:test-driven-development`，别用 `@path/to/file.md`——后者会在 parse 时强制加载，烧上下文。
- **`<SUBAGENT-STOP>` 模式**：顶层 XML 标签告诉作为 subagent 派遣的实例直接跳过本 skill。例（`using-superpowers/SKILL.md`）：

  ```
  <SUBAGENT-STOP>
  If you were dispatched as a subagent to execute a specific task, skip this skill.
  </SUBAGENT-STOP>
  ```

- **`<EXTREMELY-IMPORTANT>` 框架** + Red Flags 表 + Rationalizations vs Reality 表——superpowers 风格的「防模型自己说服自己绕过规则」武装。
- **rigid vs flexible** 明确标注。TDD / debugging / verification-before-completion 是 rigid（按字面执行）；模式/参考类是 flexible（按情境改编）。

### 7.2 目录结构（与开放规范对齐）

```
skill-name/
  SKILL.md           # 必需，frontmatter + 正文
  references/        # 按需参考（REFERENCE.md、FORMS.md 等）
  scripts/           # 可跑代码；正文里 ${CLAUDE_SKILL_DIR}/scripts/foo.py
  assets/            # 模板、数据、图片
  examples/          # 标准 input/output 对照（约定俗成）
```

### 7.3 Skill 的 TDD（superpowers 铁律）

1. **RED**：选一个真实压力场景，**不带这个 skill** 派 subagent 跑一遍，记录它实际做了什么、用了哪些 rationalization 来绕过。
2. **GREEN**：写**最小**的 skill 解决那些具体失败。
3. **REFACTOR**：再测，堵新漏洞，再测——直到防弹。每轮迭代往 Rationalizations 表里加新条目。

### 7.4 反模式

- description 写「这个 skill 做了 X 然后 Y 然后 Z」——Claude 跳过正文。
- skill 只是包装一个工具（「Bash skill：用 Bash 跑命令」）——不增加领域知识或工作流纪律，直接删。
- skill 没有「when to use」——对模型不可见。
- 叙事化（「2025-10-03 session 我们发现……」）替代可复用参考。
- 多语言稀释（`example.js` + `example.py` + `example.go`）——写一个最好的。
- 流程图用 `step1` / `helper2` 这种泛标签。
- `@references/big-file.md` 强加载——无条件烧上下文。

---

## 8. 我没能完全验证的事

- `CLAUDE_CODE_DISABLE_POLICY_SKILLS` 没有在 Anthropic 公开文档中**明文**记载。它是从 `claude` 二进制里 strings 反推 + loader 代码路径验证的——**真实有效，但属于未文档化的逃生通道**。
- managed/policy 目录的 per-OS 实际路径由内部 helper 决定，本次没有逐 OS 枚举。
- `skillResolution` 这个 setting 在某些汇总页出现，但在主 `/settings` 页正文里没见到——可能是 2026-05 之后的更新，使用前确认。
- 没有真跑 `npx skills add <real-repo>`，symlink-vs-copy 默认行为来自 README/help 文本而非实测。
- 插件 skill 单独禁用（不整插件 disable）目前**没有官方一键 UI**——只能 `/permissions` 写 `Skill(plugin:name *)` deny 规则。
