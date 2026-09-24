# AGENTS.md — Opencode 安装指南（极简）

> **本仓路线声明（2026-09-25 起，路线 A）**：专注 **OpenCode + 双 ComputerUse（PRIMARY ashhart + FALLBACK macgui）**。
> 已弃用 `oh-my-openagent / OMO` 路线，不再新装 OMO；文档中保留的 OMO 章节仅为历史归档。
> 双 ComputerUse 的完整安装与验收以 [`OpenCode_Dual_ComputerUse_Install_Guide_REVISED.md`](./OpenCode_Dual_ComputerUse_Install_Guide_REVISED.md) 为准。

本仓库仅用于：指导 Agent 在用户机器安装、配置并验证 **Opencode**。  
官方来源优先：
- https://github.com/anomalyco/opencode
- https://opencode.ai/docs

**配套文档**：
- [`SKILLS_GUIDE.md`](./SKILLS_GUIDE.md) — 技能（Skills）安装与使用完整指南
- [`OpenCode_Dual_ComputerUse_Install_Guide_REVISED.md`](./OpenCode_Dual_ComputerUse_Install_Guide_REVISED.md) — 双 ComputerUse 安装、验收与排障（PRIMARY + FALLBACK），Verified Baseline 详见该文档

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

## 9. 插件接入：oh-my-openagent（⚠️ 已弃用归档，路线 A 不再新装）

> 路线 A 下不要新装 OMO。本节仅保留卸载/回滚说明，供历史环境清理使用。

历史包名说明（归档）：`oh-my-openagent` 为现名，`oh-my-opencode` 为旧名。卸载时两者都要检查。

官方参考（归档）：
- https://github.com/code-yeongyu/oh-my-openagent
- [安装指南（上游 dev 分支）](https://github.com/code-yeongyu/oh-my-openagent/blob/dev/docs/guide/installation.md)

### 9.1 前置条件（归档，不再执行新装）

```bash
opencode --version
```

历史要求 OpenCode `>= 1.4.0`；仅供查阅，不再作为新装依据。

### 9.2 安装命令（归档，路线 A 禁止新装）

```bash
# 路线 A 下禁止执行以下新装命令，仅归档备查：
# bunx oh-my-openagent install
# bunx oh-my-openagent install --no-tui --platform=opencode --claude=no --openai=no --gemini=no --copilot=yes
```

### 9.3 配置与验证（历史环境检查用）

```bash
# OpenCode 主配置中的 plugin 数组应包含 oh-my-openagent（可带版本后缀）
jq '.plugin' ~/.config/opencode/opencode.json

# OMO 统一配置：Agent/Category 路由在 [opencode] 中
ls ~/.omo/omo.jsonc

# 检查插件诊断和解析后的 Agent 模型
bunx oh-my-openagent doctor
opencode debug agent explore

# 再次验证 opencode 可用
which opencode
opencode --version
```

### 9.4 认证（按需）

```bash
opencode auth login
```

按提示配置你实际使用的提供商（Anthropic/OpenAI/Google/GitHub 等）。

### 9.5 回滚/卸载（路线 A 下 OMO 清理仍用本节）

先备份 `opencode.json`，只从 `plugin` 数组移除 `oh-my-openagent` 条目（包括 `@latest` 等版本后缀）及遗留的 `oh-my-opencode` 条目；保留其他插件、Provider、MCP 和 Agent 配置。保存后完全退出并重启 OpenCode，再检查解析配置。

`~/.omo/omo.jsonc` 可能同时包含 `[opencode]`、`[codex]`、`[native]` 等平台配置，**不要直接删除整个文件**。只在确认不再被其他 OMO 平台使用时，备份后移除相关配置。`bunx oh-my-openagent uninstall` 当前只清理 Codex Light 的受管状态，不用于卸载 OpenCode 插件。

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

### 10.7 oh-my-openagent 内置技能（⚠️ 已弃用归档）

> 路线 A 不再配置 OMO 内置技能。以下仅归档备查：历史插件曾提供内置技能，名称随版本变化；当时需用 `opencode debug skill` + 会话 `<available_skills>` 双重核对，并按当前 OMO schema 配置 `~/.omo/omo.jsonc` 的 `[opencode]`。新环境不要再启用。

### 10.8 Agent 输出补充要求（技能操作时）

除第 7 节要求外，额外输出：
1. 技能文件路径与内容摘要
2. `name` 是否与目录名匹配
3. 技能是否在 Agent 的 `<available_skills>` 中可见
4. 结论（成功/失败）与下一步

## 11. 踩坑记录与经验总结

> 以下经验来自实际配置过程，务必遵守以避免重复踩坑。

### 11.1 配置修改后必须重启 opencode（最重要）

**坑**：`opencode.json`、`instructions`、MCP/Agent 配置均在 OpenCode 启动时加载（历史 OMO 的 `~/.omo/omo.jsonc` 亦如此）；运行中的进程不能仅靠文件修改来证明新模型已生效。

**表现**：修改了 `[opencode].agents` 或 `[opencode].categories` 后，旧会话仍使用旧模型，看起来像“改了没用”。

**正确做法**：
1. 修改配置文件
2. **完全退出 opencode 进程**
3. 重新启动 opencode
4. 用 `opencode debug agent <name>` 检查解析配置，再通过实际调用或 DB 确认运行时模型

**验证命令（查 DB 确认实际使用的模型）**：

```bash
sqlite3 ~/.local/share/opencode/opencode.db \
  "SELECT json_extract(data,'$.model.providerID'), json_extract(data,'$.model.modelID'), json_extract(data,'$.agent') FROM message WHERE json_extract(data,'$.agent') IS NOT NULL ORDER BY time_created DESC LIMIT 10;"
```

**教训**：不要把"配置没生效"误判为"平台故障"或"模型不可用"。先确认 opencode 是否重启过。

### 11.2 GitHub Copilot 模型兼容性（并非全部可用，有效期标注）

**坑**：GitHub Copilot 模型目录中列出的模型，并非全部能通过 opencode 正常调用。部分模型返回空响应或直接崩溃。

> **有效期**：下述验证仅对 `2026-09-25 / opencode 1.18.32 / auth.json 有 Copilot oauth` 有效。每次选型前必须重跑验证命令，不得直接沿用旧名单。

**2026-09 本机已验证**：`github-copilot/claude-opus-5`、`github-copilot/claude-fable-5`、`github-copilot/gpt-5.6-sol`、`github-copilot/gpt-5.6-terra`、`github-copilot/gemini-3.8-flash` 均曾通过 `opencode run --model ... "只回复 OK"` 返回 `OK`。这只证明当时的简单文本调用，不保证所有工具调用、配额或其他机器可用。

每次验证命令：

```bash
opencode models github-copilot
opencode run --model github-copilot/claude-opus-5 "只回复 OK"
opencode auth list
```

**历史记录（2026-03）**：部分早期 Gemini Copilot 模型曾返回空响应，不能据此断言所有 Gemini 模型永久不可用，也不能把旧价格乘数当作现价。目录中列出模型不等于实际可调用；选型前先查 `opencode models github-copilot`，再对目标模型做实际调用。

### 11.3 模型选型原则（⚠️ OMO 历史归档，路线 A 仅备查）

OMO 的模型选择是**按 Agent/Category 的 fallback chain**，并非单一全局默认模型。先确认插件版本、已认证 Provider 及候选模型，再决定是否显式覆盖；固定旧模型会阻止后续自动选择新版候选。

以本机 OMO `4.19.4`、GitHub Copilot 为主要 Provider 的配置为例：Sisyphus 使用 `claude-opus-5`，Hephaestus/Deep 使用 `gpt-5.6-sol`，Explore/Librarian 使用 `claude-haiku-4.5`。这些是版本相关的本机配置，不应照搬到其他版本或订阅环境；以当前插件模型需求链和实际调用结果为准。

### 11.4 移除 Provider 的完整清单

移除某个 Provider 时，不要只删 `opencode.json` 中的 provider 配置，还需要：

1. ✅ 删除 `opencode.json` → `provider` 中对应的整个 section
2. ✅ 删除 `opencode.json` → `plugin` 中关联的认证插件（如 `opencode-antigravity-auth`）
3. ✅ 删除 `~/.zshrc`（或 `~/.bashrc`）中关联的环境变量（如 `ANTHROPIC_API_KEY`）
4. ✅ 更新 `~/.omo/omo.jsonc` 的 `[opencode]` 中所有引用该 Provider 的 Agent/Category 模型
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
2. **区分解析与运行**：`opencode debug agent <name>` 查看新进程解析结果，DB/实际调用确认运行时模型（见 11.1）
3. **区分"模型不可用"和"配置未加载"**：先排除配置问题，再怀疑模型问题
4. **测试模型可用性**：单独用一个简单 prompt 测试目标模型，确认模型本身能响应
5. **不要急于下结论**：特别是不要过早诊断为"平台级故障"，大多数情况是配置层面的问题
