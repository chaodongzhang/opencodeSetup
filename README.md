# OpenCode Setup

> **路线声明（2026-09-25 起，路线 A）**：本仓专注 **OpenCode + 双 ComputerUse（PRIMARY ashhart + FALLBACK macgui）**，不再新装 `oh-my-openagent / OMO`。历史 OMO 内容仅归档备查。

这个仓库是 [OpenCode](https://github.com/anomalyco/opencode) 的**运维控制仓库**。

你不需要手动执行任何安装或配置命令。只需在 OpenCode 中打开这个仓库，用自然语言告诉 Agent 你想做什么，它会自动读取仓库中的指令文件来完成工作。

## 它能做什么

在 OpenCode 终端中，你可以直接说：

| 你说的话 | Agent 会做的事 |
|---|---|
| "帮我安装 opencode" | 根据你的操作系统选择合适的方式安装，并自动验证 |
| "升级到最新版" | 检测当前版本，执行升级，验证结果 |
| "装一下双 ComputerUse" | 按 Dual 指南安装 PRIMARY + FALLBACK，验证 Calculator 真机操作 |
| "把 OMO 插件卸了" | 从 OpenCode 主配置移除历史 OMO 插件残留（归档操作，不再新装） |
| "检查一下版本" | 输出 opencode 版本、插件状态、Agent 配置 |
| "把语言改成英文" | 修改 instructions 中的语言设置 |

这些操作不需要你记住任何命令 — Agent 会读取仓库中的指令文件，按照预定义的流程自动执行。

## 工作原理

```
你（自然语言）──▶ OpenCode Agent ──▶ 读取仓库指令 ──▶ 自动执行 ──▶ 输出结果
```

Agent 在执行任务时会自动参考以下文件：

| 文件 | 作用 |
|---|---|
| `AGENTS.md` | 核心指令 — 定义安装流程、验证步骤、失败处理、插件管理的完整规则 |
| `OpenCode_Dual_ComputerUse_Install_Guide_REVISED.md` | 双 ComputerUse 安装/验收/排障 runbook（PRIMARY + FALLBACK），Verified Baseline 以该文件为准 |
| `SKILLS_GUIDE.md` | 技能安装与使用指南 |
| `~/.config/opencode/instructions/*.md` | 全局行为设置 — 如语言偏好、输出格式等 |
| `~/.config/opencode/opencode.json` | OpenCode 主配置 — Provider、插件列表、MCP、Agent 等 |

你不需要直接编辑这些文件。需要修改时，告诉 Agent 就行。

## 前置条件

- 已安装 [OpenCode](https://github.com/anomalyco/opencode)（`brew install anomalyco/tap/opencode`，本机已验证 `1.18.32`）
- macOS（双 ComputerUse 主战场）/ Linux / Windows
- 双 ComputerUse 另需：`node/npm`、Xcode CLT（FALLBACK Swift helper 构建用）、macOS 辅助功能 + 屏幕录制授权（人工在系统设置中勾选，不绕过 TCC）
- 详见 Dual 指南 `Phase 0` 环境调查清单

## 如果你是第一次

1. 安装 OpenCode：`brew install anomalyco/tap/opencode`
2. 克隆这个仓库，在仓库目录下启动 `opencode`
3. 输入："帮我检查一下环境" — Agent 会完成剩下的事

## 相关链接

- [OpenCode 官方仓库](https://github.com/anomalyco/opencode)
- [OpenCode 文档](https://opencode.ai/docs)
- [oh-my-openagent 插件（历史归档，路线 A 不再新装）](https://github.com/code-yeongyu/oh-my-openagent)
