# OpenCode Setup

这个仓库是 [OpenCode](https://github.com/anomalyco/opencode) 的**运维控制仓库**。

你不需要手动执行任何安装或配置命令。只需在 OpenCode 中打开这个仓库，用自然语言告诉 Agent 你想做什么，它会自动读取仓库中的指令文件来完成工作。

## 它能做什么

在 OpenCode 终端中，你可以直接说：

| 你说的话 | Agent 会做的事 |
|---|---|
| "帮我安装 opencode" | 根据你的操作系统选择合适的方式安装，并自动验证 |
| "升级到最新版" | 检测当前版本，执行升级，验证结果 |
| "装一下 oh-my-opencode 插件" | 安装插件、写入配置、验证插件可用 |
| "把插件卸了" | 从配置中移除插件，清理相关文件 |
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
| `~/.config/opencode/instructions/*.md` | 全局行为设置 — 如语言偏好、输出格式等 |
| `~/.config/opencode/oh-my-opencode.json` | Agent 与模型配置 — 定义各 Agent 使用的 AI 模型 |
| `~/.config/opencode/opencode.json` | OpenCode 主配置 — Provider、插件列表等 |

你不需要直接编辑这些文件。需要修改时，告诉 Agent 就行。

## 前置条件

- 已安装 [OpenCode](https://github.com/anomalyco/opencode)（`brew install anomalyco/tap/opencode`）
- macOS / Linux / Windows

## 如果你是第一次

1. 安装 OpenCode：`brew install anomalyco/tap/opencode`
2. 克隆这个仓库，在仓库目录下启动 `opencode`
3. 输入："帮我检查一下环境" — Agent 会完成剩下的事

## 相关链接

- [OpenCode 官方仓库](https://github.com/anomalyco/opencode)
- [OpenCode 文档](https://opencode.ai/docs)
- [oh-my-opencode 插件](https://github.com/code-yeongyu/oh-my-openagent)
