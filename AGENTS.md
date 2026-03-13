# AGENTS.md — Opencode 安装指南（极简）

本仓库仅用于：指导 Agent 在用户机器安装、配置并验证 **Opencode**。  
官方来源优先：
- https://github.com/anomalyco/opencode
- https://opencode.ai/docs

**配套文档**：
- [`SKILLS_GUIDE.md`](./SKILLS_GUIDE.md) — 技能（Skills）安装与使用完整指南

## 1. 执行原则

1. 只选 **一种** 安装方式，不要混装。  
2. 每次安装后必须做验证。  
3. 若发现旧版（尤其 0.1.x），先卸载再安装。  
4. 若本文与官方 README 冲突，以官方为准。

## 2. 安装前检查（必做）

```bash
which opencode || true
opencode --version || true
```

记录：操作系统、Shell、使用的包管理器。

## 3. 安装命令（按平台）

### macOS / Linux（推荐）

```bash
brew install anomalyco/tap/opencode
```

备选：

```bash
brew install opencode
```

### 通用（Node）

```bash
npm i -g opencode-ai@latest
```

### Windows

```bash
scoop install opencode
# 或
choco install opencode
```

### 一键脚本（可选）

```bash
curl -fsSL https://opencode.ai/install | bash
```

## 4. 脚本安装目录优先级

1. `OPENCODE_INSTALL_DIR`  
2. `XDG_BIN_DIR`  
3. `$HOME/bin`  
4. `$HOME/.opencode/bin`

示例：

```bash
OPENCODE_INSTALL_DIR=/usr/local/bin curl -fsSL https://opencode.ai/install | bash
XDG_BIN_DIR=$HOME/.local/bin curl -fsSL https://opencode.ai/install | bash
```

## 5. 安装后验证（必做）

```bash
which opencode
opencode --version
```

需要满足：
- `which opencode` 有有效路径
- `opencode --version` 正常输出版本号

## 6. 失败处理（最小闭环）

- **PATH 未生效**：重开终端，或 `source ~/.zshrc` / `source ~/.bashrc`  
- **多版本冲突**：`which -a opencode`，卸载旧版后重装  
- **权限问题**：避免 `sudo npm -g`，优先 brew/scoop/choco

## 7. Agent 输出格式（必须）

每次任务输出必须包含：
1. 采用的安装方式与完整命令  
2. `which opencode` 输出  
3. `opencode --version` 输出  
4. 结论（成功/失败）与下一步

## 8. 规则优先级

若存在以下文件，先遵循它们：
- `.cursor/rules/**`
- `.cursorrules`
- `.github/copilot-instructions.md`

优先级：
1. 用户当前指令
2. Cursor/Copilot 规则
3. 本 AGENTS.md

## 9. 插件接入：oh-my-openagent（可选）

说明：仓库名是 `oh-my-openagent`，但安装命令与插件名仍为 `oh-my-opencode`。

官方参考：
- https://github.com/code-yeongyu/oh-my-openagent
- 安装指南（上游原文）：
  `https://raw.githubusercontent.com/code-yeongyu/oh-my-opencode/refs/heads/dev/docs/guide/installation.md`

### 9.1 前置条件

```bash
opencode --version
```

建议版本：`1.0.150+`（来自插件安装文档）。

### 9.2 安装命令

交互式（人工操作）：

```bash
bunx oh-my-opencode install
# 或
npx oh-my-opencode install
```

非交互（Agent 推荐）：

```bash
bunx oh-my-opencode install --no-tui --claude=<yes|no|max20> --openai=<yes|no> --gemini=<yes|no> --copilot=<yes|no>
```

### 9.3 配置与验证

```bash
# 插件应写入 opencode 主配置
cat ~/.config/opencode/opencode.json

# 期望包含："oh-my-opencode"

# 再次验证 opencode 可用
which opencode
opencode --version
```

进入 `opencode` 后，可在会话中尝试：`ultrawork`（或 `ulw`）验证插件工作流。

### 9.4 认证（按需）

```bash
opencode auth login
```

按提示配置你实际使用的提供商（Anthropic/OpenAI/Google/GitHub 等）。

### 9.5 回滚/卸载

```bash
# 从 plugin 数组移除 oh-my-opencode
jq '.plugin = [.plugin[] | select(. != "oh-my-opencode")]' ~/.config/opencode/opencode.json > /tmp/oc.json && mv /tmp/oc.json ~/.config/opencode/opencode.json

# 可选：删除插件配置
rm -f ~/.config/opencode/oh-my-opencode.json ~/.config/opencode/oh-my-opencode.jsonc
rm -f .opencode/oh-my-opencode.json .opencode/oh-my-opencode.jsonc
```

### 9.6 Agent 输出补充要求（安装插件时）

除第 7 节要求外，额外输出：
1. 使用的插件安装命令（完整）
2. `~/.config/opencode/opencode.json` 中 `plugin` 关键片段
3. 插件接入结论（成功/失败）与下一步

## 10. 技能管理（Skills）

> 完整指南见 [`SKILLS_GUIDE.md`](./SKILLS_GUIDE.md)，以下为快速参考。

### 10.1 核心概念

技能是 OpenCode 的可重用指令机制，以 `SKILL.md` 文件定义。**放文件即安装，无需 CLI 命令**。

### 10.2 安装技能（快速步骤）

```bash
# 项目级技能
mkdir -p .opencode/skills/my-skill
# 在目录中创建 SKILL.md（含 name + description frontmatter）

# 全局技能
mkdir -p ~/.config/opencode/skills/my-skill
# 同上
```

### 10.3 SKILL.md 最小格式

```markdown
---
name: my-skill
description: 这个技能做什么的简短描述
---

# 技能内容

给 Agent 看的详细指令...
```

**名称规则**：kebab-case，1-64 字符，正则 `^[a-z0-9]+(-[a-z0-9]+)*$`，必须与目录名一致。

### 10.4 搜索位置（按优先级）

1. `.opencode/skills/<name>/SKILL.md` — 项目级
2. `~/.config/opencode/skills/<name>/SKILL.md` — 全局
3. `.claude/skills/` / `.agents/skills/` — 兼容格式（项目和全局）

### 10.5 跨 Agent 统一管理（符号链接）

推荐以 `.claude/skills/` 为**单一源**，通过 symlink 同步到其他 Agent 路径，避免重复复制：

```bash
# 建立符号链接（相对路径，保证可移植）
mkdir -p .opencode/skills .agents/skills
ln -s ../../.claude/skills/my-skill .opencode/skills/my-skill
ln -s ../../.claude/skills/my-skill .agents/skills/my-skill
```

效果：修改只在 `.claude/skills/` 下进行一次，OpenCode（`.opencode/skills/`）、Claude Code（`.claude/skills/`）、其他 Agent（`.agents/skills/`）都能同时发现同一份技能。

**去重说明**：同一技能通过 symlink 出现在多个目录时，OpenCode 按 name 去重，不会重复加载。最终保留优先级最高的版本（`.opencode/skills/` > `.claude/skills/` > `.agents/skills/`）。日志中的 `"duplicate skill name"` 警告属正常现象。

> 详细说明见 [`SKILLS_GUIDE.md` § 2.4](./SKILLS_GUIDE.md)。

### 10.6 权限控制

在 `opencode.json` 中配置：

```json
{
  "permission": {
    "skill": {
      "*": "allow",
      "internal-*": "deny",
      "experimental-*": "ask"
    }
  }
}
```

### 10.7 oh-my-opencode 内置技能

安装 oh-my-opencode 插件后自动获得：`git-master`、`playwright`、`dev-browser`、`frontend-ui-ux` 等。

可在 `oh-my-opencode.json` 中禁用：

```json
{
  "disabled_skills": ["playwright-cli", "agent-browser"]
}
```

### 10.8 Agent 输出补充要求（技能操作时）

除第 7 节要求外，额外输出：
1. 技能文件路径与内容摘要
2. `name` 是否与目录名匹配
3. 技能是否在 Agent 的 `<available_skills>` 中可见
4. 结论（成功/失败）与下一步

## 11. 踩坑记录与经验总结

> 以下经验来自实际配置过程，务必遵守以避免重复踩坑。

### 11.1 配置修改后必须重启 opencode（最重要）

**坑**：oh-my-opencode 插件配置（`oh-my-opencode.json`）在 opencode 启动时**只加载一次**，运行期间修改配置文件**不会生效**。没有 file-watching 或 hot-reload 机制。

**表现**：修改了 `oh-my-opencode.json` 中的模型配置后，Agent 仍然使用旧模型，看起来像"改了没用"。

**正确做法**：
1. 修改配置文件
2. **完全退出 opencode 进程**
3. 重新启动 opencode
4. 通过 DB 或实际调用验证新配置已生效

**验证命令（查 DB 确认实际使用的模型）**：

```bash
sqlite3 ~/.local/share/opencode/opencode.db \
  "SELECT json_extract(data,'$.model.providerID'), json_extract(data,'$.model.modelID'), json_extract(data,'$.agent') FROM message WHERE json_extract(data,'$.agent') IS NOT NULL ORDER BY time_created DESC LIMIT 10;"
```

**教训**：不要把"配置没生效"误判为"平台故障"或"模型不可用"。先确认 opencode 是否重启过。

### 11.2 GitHub Copilot 模型兼容性（并非全部可用）

**坑**：GitHub Copilot 模型目录中列出的模型，并非全部能通过 opencode 正常调用。部分模型返回空响应或直接崩溃。

**已验证可用（截至 2026-03，opencode v1.2.25）**：

| 模型 | 价格乘数 | 状态 |
|---|---|---|
| `claude-opus-4.6` | 3x / 30x(fast) | ✅ 可用 |
| `claude-sonnet-4.6` | 1x | ✅ 可用 |
| `claude-sonnet-4.5` | 1x | ✅ 可用 |
| `claude-haiku-4.5` | 0.33x | ✅ 可用 |
| `gpt-5.4` | 1x | ✅ 可用 |
| `gpt-5.3-codex` | 1x | ✅ 可用 |
| `gpt-5-mini` | 0x（包含） | ✅ 可用（v1.2.25 修复）|
| `gpt-4.1` | 0x（包含） | ✅ 可用（v1.2.25 修复）|
| `grok-code-fast-1` | 0.25x | ✅ 可用 |
| `gemini-2.5-pro` | 1x | ✅ 可用（v1.2.25 修复）|
| `gemini-3-flash-preview` | 0.33x | ✅ 可用（v1.2.25 修复）|
| `gemini-3.1-pro-preview` | 1x | ✅ 可用（v1.2.25 修复）|
| `gemini-3-pro-preview` | 1x | ✅ 可用（v1.2.25 修复）|

**已验证不可用**：

| 模型 | 问题 |
|---|---|
| `gemini-2.5-flash` | Model not found |
| `gpt-5.2` | 模型过旧，不推荐 |

**注意**：v1.2.25 之前，Gemini 系列通过 GitHub Copilot provider 调用会全部返回空响应。升级到 v1.2.25 后已修复，Gemini 系列现在可正常使用。

### 11.3 模型选型原则

**Claude 模型 vs GPT 模型的 prompt 差异**：
- Claude 模型适合 mechanics-driven prompts（详细的流程指令，约 1,100 行）
- GPT 模型适合 principle-driven prompts（原则性指令，约 300 行）
- **Sisyphus（主 Agent）只有 Claude prompt，没有 GPT prompt，不要给 Sisyphus 分配 GPT 模型**

**推荐配置思路**：
- 主力编排（sisyphus, metis）→ Claude Opus（最强 Claude，机制驱动）
- 规划/审阅（oracle, prometheus, momus）→ GPT-5.4（推理能力强，原则驱动）
- 视觉/前端（visual-engineering, multimodal-looker）→ Gemini 3 Pro（视觉理解原生优势）
- 文档检索（librarian）→ Gemini 3 Flash（快速信息检索）
- 代码 grep（explore, quick）→ grok-code-fast-1（最快最便宜，专为编码设计）
- 编码任务（hephaestus, deep）→ gpt-5.3-codex（Codex 专为深度编码设计）
- 通用轻量（atlas, writing, unspecified-low）→ Claude Sonnet / Haiku

**当前生效配置（oh-my-opencode.json，截至 2026-03）**：

```json
{
  "agents": {
    "sisyphus":          { "model": "github-copilot/claude-opus-4.6",       "variant": "high" },
    "metis":             { "model": "github-copilot/claude-opus-4.6",       "variant": "high" },
    "oracle":            { "model": "github-copilot/gpt-5.4",               "variant": "high" },
    "prometheus":        { "model": "github-copilot/gpt-5.4",               "variant": "high" },
    "momus":             { "model": "github-copilot/gpt-5.4",               "variant": "xhigh" },
    "explore":           { "model": "github-copilot/grok-code-fast-1" },
    "librarian":         { "model": "github-copilot/gemini-3-flash-preview" },
    "multimodal-looker": { "model": "github-copilot/gemini-3-pro-preview" },
    "atlas":             { "model": "github-copilot/claude-sonnet-4.6" }
  },
  "categories": {
    "visual-engineering": { "model": "github-copilot/gemini-3-pro-preview",  "variant": "high" },
    "ultrabrain":         { "model": "github-copilot/claude-opus-4.6",       "variant": "high" },
    "artistry":           { "model": "github-copilot/claude-opus-4.6",       "variant": "high" },
    "deep":               { "model": "github-copilot/gpt-5.3-codex",         "variant": "high" },
    "quick":              { "model": "github-copilot/grok-code-fast-1" },
    "unspecified-low":    { "model": "github-copilot/claude-haiku-4.5" },
    "unspecified-high":   { "model": "github-copilot/gpt-5.4",               "variant": "high" },
    "writing":            { "model": "github-copilot/claude-sonnet-4.6" }
  }
}
```

### 11.4 移除 Provider 的完整清单

移除某个 Provider 时，不要只删 `opencode.json` 中的 provider 配置，还需要：

1. ✅ 删除 `opencode.json` → `provider` 中对应的整个 section
2. ✅ 删除 `opencode.json` → `plugin` 中关联的认证插件（如 `opencode-antigravity-auth`）
3. ✅ 删除 `~/.zshrc`（或 `~/.bashrc`）中关联的环境变量（如 `ANTHROPIC_API_KEY`）
4. ✅ 更新 `oh-my-opencode.json` 中所有引用该 Provider 模型的配置
5. ✅ 重启 opencode 使变更生效

**切记**：不要用 `sudo npm -g` 来解决权限问题。

### 11.5 全局指令机制（instructions）

opencode 原生支持 `instructions` 配置，用于向所有 Agent 的 system prompt 注入全局指令。

**配置方法**：

1. 在 `opencode.json` 中添加顶层字段：

```json
{
  "instructions": [
    "~/.config/opencode/instructions/*.md"
  ]
}
```

2. 在 `~/.config/opencode/instructions/` 目录中创建 `.md` 文件，每个文件的内容会被注入到所有 Agent 的 prompt 中。

**适用场景**：
- 全局语言设置（如要求所有 Agent 使用中文）
- 统一输出格式要求
- 全局行为约束

**注意**：支持 glob 模式，方便按需添加多个指令文件而无需逐个修改 `opencode.json`。

### 11.6 排查问题的正确姿势

遇到 Agent 行为异常时，按以下顺序排查：

1. **确认配置是否生效**：opencode 是否在修改配置后重启过？（见 11.1）
2. **查 DB 验证实际使用的模型**：不要靠猜测，用 SQL 查实际调用记录（见 11.1 验证命令）
3. **区分"模型不可用"和"配置未加载"**：先排除配置问题，再怀疑模型问题
4. **测试模型可用性**：单独用一个简单 prompt 测试目标模型，确认模型本身能响应
5. **不要急于下结论**：特别是不要过早诊断为"平台级故障"，大多数情况是配置层面的问题
