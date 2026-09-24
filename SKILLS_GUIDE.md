# OpenCode 技能（Skills）安装与使用完整指南

> 官方文档：https://opencode.ai/docs/skills/  
> 源代码：https://github.com/anomalyco/opencode/blob/dev/packages/opencode/src/skill/skill.ts

## 1. 核心概念

OpenCode 的 **Skills（技能）** 是可重用的指令文件，以 `SKILL.md` 格式定义。Agent 在运行时通过原生 `skill` 工具按需加载技能内容到自己的上下文中。

**关键特性**：
- **无需安装命令** — 把文件放到正确目录，OpenCode 自动发现
- **去中心化** — 没有官方 marketplace，技能通过文件、插件或远程 URL 分发
- **兼容 Claude Code** — 支持 `.claude/skills/` 和 `.agents/skills/` 目录
- **权限控制** — 细粒度的 allow/deny/ask 权限，支持通配符模式

## 2. 安装方式（放置文件即安装）

### 2.1 支持的目录位置（按优先级从高到低）

| 优先级 | 路径 | 作用域 |
|--------|------|--------|
| 1 | `.opencode/skills/<name>/SKILL.md` | 项目级（OpenCode 原生） |
| 2 | `~/.config/opencode/skills/<name>/SKILL.md` | 全局（OpenCode 原生） |
| 3 | `.claude/skills/<name>/SKILL.md` | 项目级（Claude Code 兼容） |
| 4 | `~/.claude/skills/<name>/SKILL.md` | 全局（Claude Code 兼容） |
| 5 | `.agents/skills/<name>/SKILL.md` | 项目级（通用 Agent 兼容） |
| 6 | `~/.agents/skills/<name>/SKILL.md` | 全局（通用 Agent 兼容） |

### 2.2 发现机制

OpenCode 从当前工作目录**向上遍历**到 Git worktree 根目录，加载沿途所有匹配的 `skills/*/SKILL.md`。同名技能按优先级去重，高优先级覆盖低优先级。

全局定义同时从 `~/.config/opencode/skills/`、`~/.claude/skills/`、`~/.agents/skills/` 加载。

### 2.3 安装步骤

以创建一个名为 `my-skill` 的项目级技能为例：

```bash
# 1. 创建技能目录
mkdir -p .opencode/skills/my-skill

# 2. 创建 SKILL.md 文件
cat > .opencode/skills/my-skill/SKILL.md << 'EOF'
---
name: my-skill
description: 这个技能做什么的简短描述
---

# 技能内容

这里是给 Agent 看的详细指令...
EOF

# 3. 重启 opencode（技能在启动时发现，运行中不会热加载）
```

全局安装则将文件放到 `~/.config/opencode/skills/my-skill/SKILL.md`。

### 2.4 跨 Agent 统一管理（符号链接策略）

不同编程 Agent 扫描不同的技能目录（`.opencode/skills/`、`.claude/skills/`、`.agents/skills/`）。如果为每个 Agent 复制一份技能文件，维护成本高且容易不一致。

**推荐方案**：选择一个目录作为**单一源（Single Source of Truth）**，其他目录通过符号链接（symlink）指向它。

#### 推荐结构

以 `.claude/skills/` 为单一源（兼容性最广），通过 symlink 让 OpenCode 和其他 Agent 都能发现：

```
project/
├── .claude/skills/                          ← 单一源（实际文件存放处）
│   └── my-skill/
│       ├── SKILL.md
│       └── references/
├── .opencode/skills/                        ← symlink → .claude/skills/
│   └── my-skill → ../../.claude/skills/my-skill
└── .agents/skills/                          ← symlink → .claude/skills/
    └── my-skill → ../../.claude/skills/my-skill
```

#### 操作命令

```bash
# 1. 确保技能文件在 .claude/skills/ 下
mkdir -p .claude/skills/my-skill
# （创建 SKILL.md 和 references/ 等）

# 2. 创建 symlink 目录
mkdir -p .opencode/skills .agents/skills

# 3. 建立符号链接（使用相对路径，确保项目可移植）
ln -s ../../.claude/skills/my-skill .opencode/skills/my-skill
ln -s ../../.claude/skills/my-skill .agents/skills/my-skill
```

#### 新增技能时

每次在 `.claude/skills/` 下新增技能后，只需补两条 symlink：

```bash
ln -s ../../.claude/skills/new-skill .opencode/skills/new-skill
ln -s ../../.claude/skills/new-skill .agents/skills/new-skill
```

#### 为什么选 `.claude/skills/` 作为单一源

| 考量 | 说明 |
|------|------|
| 兼容性最广 | Claude Code、Cursor、Windsurf 等主流 Agent 原生支持 |
| OpenCode 兼容 | OpenCode 将 `.claude/skills/` 作为兼容路径扫描 |
| 社区惯例 | 大多数开源项目已使用 `.claude/` 目录存放 Agent 配置 |
| 迁移成本低 | 已有 `.claude/skills/` 的项目无需移动文件 |

> **注意**：符号链接使用**相对路径**（如 `../../.claude/skills/my-skill`），不要用绝对路径，否则项目移动后链接会断。

#### 去重机制（重要）

使用 symlink 后，同一个技能会被 OpenCode 在多个目录中发现。**不会导致重复加载**，但需要了解其去重行为：

**OpenCode 原生层**（`skill.ts`）：
- 技能存储在 `Record<string, Info>` 中，以 `name` 为 key
- 同名技能后发现的**覆盖**先发现的，并输出 `"duplicate skill name"` 警告日志
- 扫描顺序（后者覆盖前者）：全局兼容路径 → 项目级兼容路径 → `.opencode/skills/` → 配置路径 → 远程 URL

**oh-my-openagent 插件层**：
- 按 scope 优先级去重，高优先级覆盖低优先级：

| scope | 优先级 | 对应路径 |
|-------|--------|---------|
| `builtin` | 1（最低） | 插件内置技能 |
| `config` | 2 | `[opencode].skills` 中定义的技能 |
| `user` | 3 | `~/.claude/skills/`（全局） |
| `opencode` | 4 | `~/.config/opencode/skills/`（全局） |
| `project` | 5 | `.claude/skills/`、`.agents/skills/`（项目级） |
| `opencode-project` | 6（最高） | `.opencode/skills/`（项目级） |

**对 symlink 方案的实际影响**：

| 路径 | scope | 结果 |
|------|-------|------|
| `.claude/skills/my-skill/` | project (5) | 先被扫描 |
| `.agents/skills/my-skill/` → symlink | project (5) | 同 scope，被忽略 |
| `.opencode/skills/my-skill/` → symlink | opencode-project (6) | **最终保留此版本** |

- ✅ 技能内容完全相同（symlink 指向同一文件），功能无影响
- ⚠️ 可能出现 `"duplicate skill name"` 警告；确认最终保留的是预期技能版本
- ✅ 同名技能最终按优先级保留一个版本；加载开销仍以实际插件版本为准

## 3. SKILL.md 文件格式

### 3.1 最小要求

```markdown
---
name: my-skill
description: 这个技能做什么的简短描述
---

# 技能内容

这里是给 Agent 看的详细指令...
```

### 3.2 完整字段

```yaml
---
# === 必需字段 ===
name: my-skill                         # 1-64 字符，kebab-case，必须与目录名一致
description: 技能描述                    # 1-1024 字符，足够具体供 Agent 选择

# === 可选字段（OpenCode 原生）===
license: MIT                           # 许可证标识符
compatibility: opencode                # 兼容性标记
metadata:                              # 自定义元数据（字符串键值对）
  author: your-name
  workflow: github

# === OMO 扩展字段（使用前核对当前版本）===
model: github-copilot/claude-opus-5    # 技能委派模型
agent: custom-agent                    # 委派 Agent
subtask: true                          # 作为子任务执行
argument-hint: "用法提示"                # 命令提示
allowed-tools:                         # 工具限制
  - read
  - grep
# mcp: ...                             # 技能级 MCP，需按当前 schema 配置
---
```

上半部分为 OpenCode 原生技能字段；下半部分是 OMO 技能加载器支持的扩展字段，具体功能和 MCP 配置形状取决于插件版本。通过 `opencode debug skill` 与实际委派检查加载结果，不要把 OpenCode 主配置的 `agent`、`mcp` 结构原样复制进 frontmatter。

### 3.3 名称规则

```
正则: ^[a-z0-9]+(-[a-z0-9]+)*$

✅ 合法: git-release, code-review, my-skill-v2
❌ 非法: GitRelease, git--release, -my-skill, my_skill
```

**名称必须与包含 `SKILL.md` 的目录名完全一致。**

### 3.4 描述规范

- 长度：1-1024 字符（推荐 50-150 字）
- 风格：第三人称，足够具体供 Agent 判断何时使用
- 示例：`"Create consistent releases and changelogs from merged PRs"`

## 4. 目录结构

```
my-skill/
├── SKILL.md              # 核心指令（必需，建议 < 5,000 字）
├── references/           # 详细参考文档（可选，Agent 按需加载）
│   ├── api-docs.md
│   └── changelog-guide.md
├── scripts/              # 可执行脚本（可选，不自动加载）
│   └── generate-notes.sh
└── assets/               # 模板和静态文件（可选，不自动加载）
    └── template.json
```

- `SKILL.md` 中可以通过 `@references/api-docs.md` 格式引用子文件
- `references/` 中的文件会被 Agent 按需读取
- `scripts/` 和 `assets/` 不会被自动加载到 Agent 上下文

## 5. 完整示例

### 5.1 基础示例：Git 发布

```
.opencode/skills/git-release/SKILL.md
```

```markdown
---
name: git-release
description: Create consistent releases and changelogs
license: MIT
compatibility: opencode
metadata:
  audience: maintainers
  workflow: github
---

## What I do
- Draft release notes from merged PRs
- Propose a version bump
- Provide a copy-pasteable `gh release create` command

## When to use me
Use this when you are preparing a tagged release.
Ask clarifying questions if the target versioning scheme is unclear.
```

### 5.2 浏览器自动化技能示例

```markdown
---
name: browser-automation
description: Use when automating browser interactions in a project with an approved browser tool.
---

# Browser Automation Skill

Use the browser tool configured for this project to automate browser interactions.

## Capabilities
- Navigate to URLs and take screenshots
- Fill forms and click elements
- Extract data from web pages
- Test web application flows
```

此示例只描述技能指令，不会自动安装或授权浏览器工具。OMO 的技能加载器还支持特定的扩展字段（包括技能级 MCP）；使用前应核对当前版本的 schema、审批权限与实际加载结果，不要把原生 SKILL.md 示例当作 MCP 安装步骤。

### 5.3 高级示例：GitHub Triage（来自 oh-my-openagent 仓库）

```markdown
---
name: github-triage
description: "读取 GitHub 问题和 PR。分析所有开放项目并输出有证据支持的报告。从不改变 GitHub 状态 - 仅报告。"
---

# GitHub Triage - 只读分析器

<role>
读取 GitHub 开放问题/PR，分类，为每项生成后台子代理分析。
</role>

## 架构

| 规则 | 值 |
|-----|-----|
| Category | `quick` |
| Execution | `run_in_background=true` |
| 并行度 | 所有项目同步 |
| Output | `/tmp/{YYYYMMDD-HHmmss}/issue-{N}.md` |
```

## 6. 权限控制

### 6.1 全局权限（opencode.json）

```json
{
  "permission": {
    "skill": {
      "*": "allow",
      "pr-review": "allow",
      "internal-*": "deny",
      "experimental-*": "ask"
    }
  }
}
```

| 权限 | 行为 |
|------|------|
| `allow` | 技能立即加载 |
| `deny` | 技能对 Agent 隐藏，拒绝访问 |
| `ask` | 加载前询问用户批准 |

模式支持通配符：`internal-*` 匹配 `internal-docs`、`internal-tools` 等。

### 6.2 按 Agent 覆盖权限（opencode.json）

```json
{
  "agent": {
    "plan": {
      "permission": {
        "skill": {
          "internal-*": "allow"
        }
      }
    }
  }
}
```

### 6.3 按 Agent 拒绝技能加载

对不需要技能的 Agent 拒绝加载所有技能：

```json
{
  "agent": {
    "plan": {
      "permission": {
        "skill": {
          "*": "deny"
        }
      }
    }
  }
}
```

此配置限制加载权限；是否在 `<available_skills>` 中显示还取决于当前 OpenCode 版本，需用 `opencode debug agent plan` 和实际会话确认。

## 7. 远程技能源（高级）

OpenCode 支持从 URL 拉取技能集合。

### 7.1 配置

```json
{
  "skills": {
    "urls": ["https://example.com/.well-known/skills/"]
  }
}
```

### 7.2 远程端格式

远程端需在基 URL 下提供 `index.json`：

```json
{
  "skills": [
    {
      "name": "agents-sdk",
      "description": "Build AI agents on Cloudflare Workers",
      "files": ["SKILL.md", "references/callable.md"]
    }
  ]
}
```

OpenCode 会从相同基 URL 下载每个文件：
- `https://example.com/.well-known/skills/agents-sdk/SKILL.md`
- `https://example.com/.well-known/skills/agents-sdk/references/callable.md`

**本地缓存位置**：`~/.cache/opencode/skills/<skill-name>/`

## 8. oh-my-openagent 插件扩展（⚠️ 已弃用归档，路线 A 不再配置）

> 路线 A 下本节仅归档备查，不再新装/新配 OMO。以下内容有效期止于 `2026-09-25` 前的历史 OMO 环境，使用前必须按当前上游 schema 重核。

oh-my-openagent 历史上可提供内置技能，但技能名单随版本变化。`opencode debug skill` 可查看 CLI 可发现的技能，却不一定列出插件注入的全部内置技能；当时需同时查看会话中的 `<available_skills>` 和当前插件列表，再加载或禁用对应名称。

### 8.1 内置技能（历史名单，不再维护）

| 技能名称 | 作用 | 触发关键词 |
|---------|------|----------|
| `git-master` | Git 提交、变基、历史搜索 | commit, rebase, squash, blame |
| `frontend` | 前端与 UI/UX 工作 | UI/UX 任务、样式 |
| `visual-qa` | 页面与组件视觉验收 | 截图、响应式检查 |

### 8.2 在 OMO 统一配置中禁用技能

```jsonc
{
  "[opencode]": {
    "disabled_skills": ["visual-qa"]
  }
}
```

文件位置为 `~/.omo/omo.jsonc`（项目级为 `.omo/omo.jsonc`）。只禁用本机实际存在的技能；其他技能配置字段需按当前 OMO schema 核对。OpenCode 原生技能权限仍在 `opencode.json` 的 `permission.skill` 中设置。

若要跨 OMO 平台禁用技能，可将 `disabled_skills` 放在统一配置顶层；上例放在 `[opencode]`，只针对 OpenCode。

### 8.3 通过 `load_skills` 在任务委派中使用

```typescript
// UI 实现
task({
  category: "visual-engineering",
  load_skills: ["frontend", "visual-qa"],
  prompt: "实现响应式仪表板"
})

// Git 操作
task({
  category: "quick",
  load_skills: ["git-master"],
  prompt: "整理提交历史"
})
```

## 9. Agent 如何使用技能

### 9.1 技能在工具描述中列出

OpenCode 将可用技能嵌入 `skill` 工具的描述中：

```xml
<available_skills>
  <skill>
    <name>git-release</name>
    <description>Create consistent releases and changelogs</description>
  </skill>
</available_skills>
```

### 9.2 Agent 按需加载

Agent 通过调用 `skill` 工具加载技能内容：

```
skill({ name: "git-release" })
```

返回 SKILL.md 的完整内容，Agent 据此执行任务。

## 10. 故障排查

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| 技能不显示 | 文件名不是 `SKILL.md`（大小写敏感） | 确保文件名全部大写：`SKILL.md` |
| 加载失败 | 缺少 frontmatter | 检查 YAML 包含 `name` 和 `description` |
| 名称不匹配 | name 与目录名不一致 | 确保 frontmatter 中的 `name` 与目录名完全一致 |
| 重复名称警告 | 多个位置有同名技能 | 高优先级位置的技能会覆盖低优先级；使用唯一名称 |
| Agent 看不到技能 | 权限设为 `deny` | 检查 `opencode.json` 中的 `permission.skill` 配置 |
| 远程技能未下载 | URL 或 index.json 格式错误 | 验证 `.well-known/skills/index.json` 存在且格式正确 |
| 修改技能后不生效 | OpenCode 未重启 | 完全退出并重启 opencode |

## 11. 源代码参考

| 文件 | 功能 | 链接 |
|------|------|------|
| skill.ts | 核心加载和管理 | [GitHub](https://github.com/anomalyco/opencode/blob/dev/packages/opencode/src/skill/skill.ts) |
| discovery.ts | 远程技能下载 | [GitHub](https://github.com/anomalyco/opencode/blob/dev/packages/opencode/src/skill/discovery.ts) |
| tool/skill.ts | Skill 工具实现 | [GitHub](https://github.com/anomalyco/opencode/blob/dev/packages/opencode/src/tool/skill.ts) |
| 官方文档 | Skills 完整指南 | [opencode.ai/docs/skills](https://opencode.ai/docs/skills/) |

## 12. 一句话总结

> **OpenCode 的技能安装 = 在正确目录创建 `<name>/SKILL.md` 文件**。没有 `install` 命令，没有包管理器，没有注册表 — 放文件即用，OpenCode 自动发现。
