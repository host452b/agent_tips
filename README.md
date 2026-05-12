# agent_tips

主流 agent CLI 的**环境变量参考**，每个 CLI 一份，全部按官方文档/源码逐条核验。

| CLI | 文件 | 文档化变量 | 核心提醒 |
|---|---|---|---|
| Claude Code | [`claude-code/env.md`](./claude-code/env.md) | ~250 | 来自 `code.claude.com/docs` 单页主表 + 多个子页面 |
| OpenAI Codex CLI | [`codex/env.md`](./codex/env.md) | ~7 用户面 + 仓库内部常量 | 主体走 `~/.codex/config.toml`，**不**沿用 `CLAUDE_CODE_*` → `CODEX_*` 命名 |
| cursor-agent CLI | [`cursor-agent-cli/env.md`](./cursor-agent-cli/env.md) | 7 | Cursor 官方没单独的 env 参考页，配置主要靠 flag 与 `cli-config.json` |

## 为什么有这个仓库

社区里关于 agent CLI 环境变量的清单经常**夹带虚构、写错前缀、或者把一种 CLI 的命名规则套到另一种上**。本仓库的做法：

1. 每个变量都标了 **`source` 链接**（精确到官方文档 URL，必要时到仓库源码行号）。
2. 每个变量都标了 **`status`**：`documented` / `experimental` / `deprecated` / `inferred-from-code`。
3. 显式列出常见误传名（如 `CLAUDE_CODE_AUTO_BACKGROUND_TASKS` 实为 `CLAUDE_AUTO_BACKGROUND_TASKS`、`OPENAI_BASE_URL` Codex 不读、`model_reasoning_effort` 没有 `max` 这种合法值）。
4. 把 **「容易误以为是环境变量、其实是 config 字段」** 单列一节（Codex 尤其需要）。

## 文件结构

```
agent_tips/
├── README.md
├── claude-code/
│   └── env.md
├── codex/
│   └── env.md
└── cursor-agent-cli/
    └── env.md
```

每个 `env.md` 的统一结构：

1. **数据来源** — 抓取的官方页面/源码文件清单
2. **速查总表** — 变量名 + 一句话作用 + 状态
3. **重要变量详解** — 默认值、行为细节、相互依赖
4. **常见误传 / config vs env 对照** — 纠正社区常见错误
5. **推荐配置模板** — 几种典型场景的写法
6. **版本备忘** — 哪些变量有最低版本要求

## 采集口径

- **采集日期**：2026-05-12
- **Claude Code**：以 [`code.claude.com/docs/en/env-vars`](https://code.claude.com/docs/en/env-vars) 单页总表为准，交叉验证 `settings` / `hooks` / `model-config` / `monitoring-usage` / `amazon-bedrock` / `google-vertex-ai` / `microsoft-foundry` / `claude-platform-on-aws` / `llm-gateway` 子页面。
- **Codex**：以 [`developers.openai.com/codex/config-reference`](https://developers.openai.com/codex/config-reference) 与 [`openai/codex`](https://github.com/openai/codex) 仓库源码（`codex-rs/` 子目录）为准；版本对应 `rust-v0.130.0`。
- **cursor-agent**：以 [`cursor.com/docs/cli/*`](https://cursor.com/docs/cli) 路径下的页面 + [`cursor.com/docs/agent/tools/terminal`](https://cursor.com/docs/agent/tools/terminal) 为准。

## 阅读建议

- 想**调一个具体行为**：先看 CLI 的「速查总表」用 `Ctrl-F` 找关键词，再回到「详解」看默认值与坑。
- 想**搭一份 `settings.json` / `config.toml`**：直接看每份文档末尾的「推荐配置模板」。
- 看到第三方清单里的变量名，**先在对应 CLI 的「常见误传」表里搜一下**——很多名字是错的。

## 维护

环境变量集合演进很快（特别是 Claude Code 与 Codex）。重新采集时应：

1. 重新 fetch 第 4 节列出的官方页面。
2. diff 对照本文档表格，记录新增/废弃/重命名。
3. 更新各 `env.md` 顶部的「采集日期」与「覆盖版本」字段。

不要从社区博文反向回填表格——容易把不存在的名字写回来。
