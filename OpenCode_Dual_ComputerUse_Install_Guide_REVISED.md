# OpenCode 双 Computer Use 安装与配置指令

## 目标

请在当前 macOS 电脑上，完整安装、配置、测试以下两套 OpenCode Computer Use 方案，并形成“主 Computer Use + 原生 fallback”的架构。

```text
OpenCode / Sisyphus
        │
        ├── PRIMARY
        │      computer_use
        │      computer_use_browser
        │            │
        │            ▼
        │     ashhart/ComputerUse
        │            │
        │            ▼
        │     Codex / ChatGPT
        │     Computer Use runtime
        │
        └── FALLBACK
               macgui_*
                  │
                  ▼
        virajshoor/opencode-computer-use
                  │
                  ▼
        macOS native Accessibility /
        CoreGraphics / CGEvent
```

## Verified Baseline（已验证基线）

以下基线来自 **2026-09-24** 的完整安装与真实 GUI 验收。它用于判断未来 upstream 是否发生变化，不代表以后必须永久固定在这些版本。

```text
Date: 2026-09-24

OpenCode:
1.18.32
Config schema:
verified against OpenCode 1.18.32 current schema

PRIMARY:
ashhart/ComputerUse
e98236cca6e15d5152a285afa0a63970f0b0339b

FALLBACK:
virajshoor/opencode-computer-use
3df2b09b3c62341fa447fe524b64df169e9ef6b2
```

安装或升级时必须记录两个 checkout 的实际 `git rev-parse HEAD`。

如果当前 upstream HEAD 与上述 verified revision 不同：

1. 不要自动回退到旧 commit；
2. 把当前 revision 标记为 **UNVERIFIED UPGRADE**；
3. 重新执行 PRIMARY、FALLBACK、routing、权限、cleanup 的完整验收；
4. 只有完整验收通过后，才能把新的 revision 作为新的 verified baseline；
5. 不要把旧版本 workaround 无条件套用到新版本。

---

## Compatibility Notes（仅针对上述已验证基线）

以下都是 **2026-09-24 + 上述 commit + OpenCode 1.18.32** 下实际观察到的兼容性现象。它们是排障线索，不是永久事实。未来版本应先重新验证，再决定 workaround 是否仍然需要。

### PRIMARY screenshot MIME

在已验证基线上，部分 ChatGPT/Codex runtime screenshot attachment 出现：

```text
实际内容：JPEG
声明 mimeType：image/png
```

OpenCode 严格校验真实文件签名时可能拒绝该附件。

遇到时：

1. 记录原始错误；
2. 不修改第三方源码来制造通过；
3. 改用 AX 文本路径完成 GUI 操作与结果验证；
4. 后续 upstream revision 若已修复，不再沿用该 workaround。

### FALLBACK 多显示器负坐标

在已验证基线及部分多显示器布局中观察到：

- `macgui_computer_move` / `drag` 可处理负坐标；
- `macgui_computer_click` 在负 X/Y 副屏上可能发生坐标重映射异常；
- 把窗口移动到主屏非负坐标区域后 click 正常。

这是当前 revision 的兼容性记录，不应假定未来版本仍然存在。

### FALLBACK TextEdit AX 测试

已验证 revision 的 `npm test` 对 TextEdit AX 结构存在特定假设，例如期待 `AXTextArea`。新版或中文 macOS 的 TextEdit AX tree 可能不同，因此官方脚本可能失败，即使 MCP、权限、截图和 AX 读取实际正常。

不要为了让测试变绿而修改 upstream test。应保留原始失败，并通过独立 Calculator GUI 验收判断核心能力。

### Browser 验证范围

`Chrome native host / extension configured` 与 `computer_use_browser 已完成真实浏览器控制验收` 是两个不同状态。

必须分别报告：

```text
Browser setup: CONFIGURED / NOT CONFIGURED
Browser live Computer Use: PASS / NOT TESTED / FAIL
```

除非任务明确要求 Browser live acceptance，否则桌面双 Computer Use 的最终 PASS 不要求浏览器 live test；但不得把 `Chrome native host ✅` 描述成浏览器控制已经通过真实验收。

---

## 核心原则

1. `ashhart/ComputerUse` 是 PRIMARY Computer Use。
2. `virajshoor/opencode-computer-use` 是 FALLBACK。
3. 第二套 MCP Server 必须命名为 `macgui`，不要命名为 `computer-use`。
4. 不允许把 `ashhart/ComputerUse` 同时作为 OpenCode Plugin 和 MCP 安装，避免 duplicate tools。
5. `ashhart/ComputerUse` 只使用 OpenCode Plugin 集成。
6. `virajshoor/opencode-computer-use` 只作为 MCP Server 集成。
7. 两套 GUI backend 绝对禁止并发操作真实桌面。
8. 同一个 GUI workflow 中不要在两套 backend 之间随意来回切换。
9. 默认优先 PRIMARY；只有 PRIMARY 无法连接、runtime 出错、API 缺失或明显失败时，才允许切换 FALLBACK。
10. 不修改、不删除现有其他 OpenCode Agent、Plugin、MCP、模型配置。
11. 不要修改当前正在开发的项目仓库。
12. 不要执行任何 git push、远端写入或无关系统修改。
13. macOS Accessibility / Screen Recording 等需要人工授权的步骤，不得试图绕过安全机制；可以自动打开对应设置页面并明确告诉用户需要勾选什么。

---

# Phase 0：环境与配置调查

先不要安装任何东西。

检查并记录：

```bash
opencode --version
which opencode

node --version
npm --version
python3 --version
xcode-select -p
xcodebuild -version
```

检查：

- 当前 OpenCode 属于哪一代配置格式。
- 当前实际使用的 OpenCode config 路径。
- 是否存在：

```text
~/.config/opencode/opencode.json
~/.config/opencode/opencode.jsonc
$XDG_CONFIG_HOME/opencode/...
```

- 当前已有 plugins。
- 当前已有 MCP servers。
- 当前 Agent 配置。
- 是否安装 Oh My OpenAgent / Sisyphus。
- 当前是否存在 Computer Use 相关 Plugin/MCP，避免重复安装。

如果 OpenCode 当前版本支持相关命令，也检查：

```bash
opencode mcp list
```

不要根据旧文档盲目决定配置结构。

OpenCode 不同版本的配置 schema 可能变化，因此必须根据“当前实际版本 + 当前配置文件 schema + 当前 OpenCode 官方行为”选择正确写法。不要把 OpenCode 1.18.x 当前使用的 `agent` / `permission` object schema 简称为“V2 schema”；报告时应写成类似 `verified against OpenCode 1.18.32 current schema`。

在修改任何 OpenCode 配置文件之前：

1. 创建 timestamped backup。
2. 验证当前 JSON/JSONC 可以正常解析。
3. 修改后再次验证。
4. 禁止覆盖整个配置文件，只做最小增量修改。

---

# Phase 1：检查 Codex / ChatGPT Computer Use runtime

检查机器上是否已有 ChatGPT 或 Codex Desktop，以及 `ashhart/ComputerUse` 所需的 vendor runtime。

不要假定路径一定存在。

检查 Computer Use 是否已经在 ChatGPT/Codex Desktop 中启用。

如果缺少 runtime：

- 不要伪造。
- 不要下载来源不明的 runtime。
- 不要复制认证 token。
- 不要修改 private IPC、安全策略或 browser profile。
- 明确说明缺少的组件。

如果 runtime 已存在，则继续。

---

# Phase 2：安装 PRIMARY —— ashhart/ComputerUse

项目：

```text
https://github.com/ashhart/ComputerUse
```

把第三方工具安装在稳定、不会被临时删除的位置，例如：

```text
~/.local/share/opencode-tools/ComputerUse
```

如果目录不存在则创建。

如果已经安装：

- 检查 remote。
- 检查 working tree。
- 不要直接破坏已有修改。
- 如果是干净 checkout，可以安全更新。
- 如果存在本地修改，先报告，不要覆盖。

全新安装时：

```bash
git clone https://github.com/ashhart/ComputerUse.git \
  ~/.local/share/opencode-tools/ComputerUse

cd ~/.local/share/opencode-tools/ComputerUse
git rev-parse HEAD
```

记录实际 revision，并与本文 `Verified Baseline` 比较。

对于全新、干净 checkout，如果仓库存在有效 `package-lock.json`，优先使用可重复安装方式：

```bash
npm ci
```

如果 upstream 当前 README 明确改为其他依赖安装流程，则以当前 upstream 为准，并在报告中记录差异。不要为了沿用本 runbook 而覆盖新的官方安装方式。

优先采用项目当前官方针对 OpenCode 的安装入口：

```bash
./bin/computer-use install --opencode
```

不要使用：

```text
ashhart ComputerUse MCP
```

因为我们要的是 OpenCode Plugin 路径。

确认最终产生类似：

```text
~/.config/opencode/plugins/computer-use.js
```

或当前 `XDG_CONFIG_HOME` 对应位置。

该目录中的 `computer-use.js` 会被 OpenCode 自动发现。不要再把同一个本地 Plugin 手动加入 `opencode.json` 的 `plugin` 数组，否则可能造成重复加载或后续维护混乱。

确认 loader 指向稳定 checkout：

```text
~/.local/share/opencode-tools/ComputerUse
```

因为该项目的 OpenCode loader 依赖 checkout，需要保留该目录。

确保：

```text
~/.local/bin
```

在 `PATH` 中。

如果需要修改 shell config：

- 先检查现有配置；
- 不要重复添加同一行；
- 只做最小修改。

---

# Phase 3：PRIMARY 权限检查

运行项目当前版本提供的权限与状态检查命令，例如：

```bash
./bin/computer-use permissions
./bin/computer-use permissions --screen
./bin/computer-use status
./bin/computer-use verify
```

当前版本中，两条权限命令用途不同：

- `permissions`：请求或打开 Accessibility 授权页面；
- `permissions --screen`：请求或打开 Screen Recording 授权页面。

不要只运行其中一条。

如果当前版本命令不同，以项目当前 README / CLI help 为准，不要强行执行已不存在的旧命令。

需要：

```text
Accessibility
Screen Recording
```

如果需要人工授权：

1. 自动打开正确的 System Settings 页面；
2. 停止进一步 GUI 测试；
3. 清楚列出需要启用的项目；
4. 不要绕过 macOS TCC。

权限完成以后再继续验证。

---

# Phase 4：安装 FALLBACK —— virajshoor/opencode-computer-use

项目：

```text
https://github.com/virajshoor/opencode-computer-use
```

安装到：

```text
~/.local/share/opencode-tools/opencode-computer-use
```

全新安装时：

```bash
git clone https://github.com/virajshoor/opencode-computer-use.git \
  ~/.local/share/opencode-tools/opencode-computer-use

cd ~/.local/share/opencode-tools/opencode-computer-use
git rev-parse HEAD

npm ci
mkdir -p bin
npm run build
```

记录实际 revision，并与本文 `Verified Baseline` 比较。

如果未来 upstream 移除/变更 lockfile 或明确要求其他依赖安装流程，以当前 upstream README 为准；不要强行使用 `npm ci`。

当前上游仓库的全新 checkout 可能没有预建 `bin/` 目录，而 `build:sdk` 会直接把 Swift helper 写到 `bin/computeruse`。如果未先创建目录，链接阶段可能出现：

```text
ld: open() failed, errno=2 (No such file or directory) for 'bin/computeruse'
```

创建 `bin/` 只是补齐构建输出目录，不是修改第三方源码。

确认：

```text
dist/index.js
bin/computeruse
```

正确生成。

运行项目自己的测试（如果当前版本提供）：

```bash
npm test
```

不要因为测试失败就自动修改项目源码；先判断是环境、权限还是项目本身问题。

---

# Phase 5：把第二套注册成 `macgui` MCP

非常重要：

MCP Server 的 OpenCode 名称必须是：

```text
macgui
```

不要使用：

```text
computer-use
computer_use
```

根据当前 OpenCode 版本使用正确 schema。

目标效果是 OpenCode 最终看到类似：

```text
macgui_computer_permissions
macgui_computer_screenshot
macgui_computer_screeninfo
macgui_computer_click
macgui_computer_move
macgui_computer_drag
macgui_computer_scroll
macgui_computer_type
macgui_computer_key
macgui_computer_list_windows
macgui_computer_read_screen
...
```

MCP command 必须使用可靠的绝对路径，不依赖当前项目 cwd。

大意应该指向：

```text
node
~/.local/share/opencode-tools/opencode-computer-use/dist/index.js
```

但必须使用当前机器实际解析出来的 Node 绝对路径，例如通过：

```bash
which node
```

确认。

不要假定：

```text
/usr/local/bin/node
/opt/homebrew/bin/node
```

配置完成以后：

```bash
opencode mcp list
```

或当前版本等效命令确认 `macgui` connected。

---

# Phase 6：避免工具选择混乱

我们的设计不是让 PRIMARY 和 FALLBACK 平级竞争。

建立以下明确 routing policy：

## PRIMARY

```text
ashhart ComputerUse
```

工具：

```text
computer_use
computer_use_browser
computer_use_browser_setup
computer_use_reset
computer_use_stop
```

用途：

- 普通桌面 Computer Use
- 长程 GUI workflow
- 持久 app state
- Screenshot / AX
- Chrome/browser workflow
- 多步骤 GUI 操作

## FALLBACK

```text
macgui_*
```

用途：

- PRIMARY runtime 无法启动
- PRIMARY 报 runtime/API error
- PRIMARY 无法访问目标 App
- PRIMARY Computer Use API 暂时失效
- 需要底层 macOS AX / CGEvent 操作
- 需要直接检查 Accessibility tree
- 需要独立于 Codex runtime 排障

禁止：

```text
PRIMARY 与 FALLBACK 同时控制 GUI
```

尤其禁止两个 subagent 同时：

```text
移动鼠标
点击
输入键盘
切换 frontmost app
拖动
滚动
```

GUI Computer Use 必须串行执行。

---

# Phase 7：Agent 隔离

检查当前 OpenCode + Sisyphus 是否适合建立一个专门的 fallback subagent。

如果当前版本支持稳定的 agent-specific tool configuration，并且不会破坏 Oh My OpenAgent/Sisyphus：

创建一个清晰命名的 subagent，例如：

```text
gui-fallback
```

职责仅限：

```text
当 PRIMARY Computer Use 不可用时，
使用 macgui_* 完成或诊断 macOS GUI 操作。
```

让它：

```text
仅在 OpenCode 审批后允许执行 macgui_*
```

同时尽量减少无关工具。

如果当前 OpenCode 支持让 Sisyphus 默认隐藏 `macgui_*`，但 `gui-fallback` 可以访问，则采用这种配置。

理想状态：

```text
Sisyphus
│
├── computer_use
├── computer_use_browser
│
└── task → gui-fallback
               │
               └── macgui_*
```

这样避免把大约 17 个 fallback GUI MCP schema 长期放进主 Agent 的有效工具集合。

但是：

如果当前 OpenCode/Oh My OpenAgent 的 agent routing 机制不适合安全实现这个隔离，不要强行 hack。

此时允许：

```text
Sisyphus 同时看到 macgui_*
```

但必须在 Agent instruction 中明确：

```text
macgui_* 仅为 fallback，不是与 computer_use 平级的默认工具。
```

不要修改 Oh My OpenAgent 项目源码。

## `gui-fallback` 的正确调用方式

`gui-fallback` 是 `mode: "subagent"`，不要直接执行：

```bash
opencode run --agent gui-fallback "..."
```

OpenCode 会提示该 Agent 不是 primary agent，并回退到默认主 Agent；这不能证明 fallback routing 正常。

正确方式是在新的 Sisyphus/主 Agent 会话中通过 `task → gui-fallback` 委派，并明确本轮禁止 PRIMARY。`macgui_*: ask` 需要交互式 OpenCode 会话呈现审批；headless `opencode run` 可能自动拒绝权限请求，因此不应把这种拒绝误判为 MCP 故障。

---

# Phase 8：权限策略

不要把 GUI 工具全部设置成无条件自动允许。

Computer Use 涉及真实桌面操作。

优先使用 OpenCode 当前版本原生的 permission/permissions 机制。

## PRIMARY

保留 `ashhart/ComputerUse` 自己与 OpenCode permission UI 的审批机制。

## FALLBACK

对于：

```text
macgui_*
```

至少不要通过配置绕过已有的 OpenCode 安全审批体系。

如果 OpenCode 当前版本支持 wildcard permissions，可以使用合适的：

```text
macgui_*
```

规则。

但是不要为了“方便”把高风险 GUI、AppleScript 等能力永久 unrestricted。

尤其注意：

```text
computer_applescript
```

属于高风险能力。

不要给它比其他 fallback 工具更宽松的权限。

## 已验证的最小权限与 Agent 配置

OpenCode `1.18.x` 可采用以下结构。只能把对应字段最小增量合并到现有配置，不得用整个示例覆盖用户的 Plugin、MCP、Agent、Provider 或模型配置。必须把示例中的 Node 和用户目录替换为当前机器实际绝对路径，并把 `model` 替换为当前已经认证且验证可调用的模型：

```json
{
  "mcp": {
    "macgui": {
      "type": "local",
      "command": [
        "/actual/absolute/path/to/node",
        "/Users/USER/.local/share/opencode-tools/opencode-computer-use/dist/index.js"
      ],
      "enabled": true
    }
  },
  "agent": {
    "gui-fallback": {
      "description": "仅当 PRIMARY Computer Use 不可用时使用 macgui_*。",
      "mode": "subagent",
      "model": "provider/verified-model",
      "prompt": "仅在 PRIMARY 无法连接、runtime/API 出错、无法访问目标 App，或上级明确指定 fallback 时使用 macgui_*。GUI 是串行资源，绝不与 computer_use 或其他 GUI 子代理并发。",
      "permission": {
        "*": "deny",
        "macgui_*": "ask"
      }
    }
  },
  "permission": {
    "macgui_*": "deny"
  }
}
```

实际效果：

- 全局 `macgui_*: deny` 阻止 Sisyphus 和其他普通 Agent 执行 fallback 工具；
- `gui-fallback` 中后置的 `macgui_*: ask` 覆盖全局 deny，并保留 OpenCode 审批；
- 当前 OpenCode 不能保证从主 Agent 的 tool schema 中真正隐藏 MCP 工具，可靠的隔离边界是执行权限；
- 不要使用已弃用的 `agent.tools` 实现隔离；
- `gui-fallback` 应显式指定已验证可用的模型，避免继承到过期 OAuth 或不可用 Provider。

修改 MCP、Agent、Plugin 或 instructions 后，必须完全退出并重新启动 OpenCode。配置在进程启动时加载，不要用旧 session 判断新配置是否生效。

---

# Phase 9：PRIMARY 独立验收

重启一个新的 OpenCode session，让 Plugin 真正重新加载。

不要只检查文件存在。

首先列出实际注册工具，确认：

```text
computer_use
computer_use_browser
computer_use_browser_setup
computer_use_reset
computer_use_stop
```

没有出现第二份 ashhart MCP duplicate tools。

然后使用 PRIMARY 完成一个最小真实测试。

推荐 Calculator：

```text
打开 Calculator
计算 1 + 2
验证结果为 3
```

要求验证：

1. Computer Use runtime 成功连接。
2. Calculator 可以被获取。
3. 可以看到 UI state。
4. 可以执行 GUI action。
5. 可以读取/截图验证结果。
6. turn 结束后 capture/sharing 能正常释放。
7. 没有调用 `macgui_*`。

优先使用 `getAXState()` 的文本结果验证 Calculator，不要依赖固定语言的按钮 Description。应优先按稳定控件 ID 查找，例如：

```text
AllClear
One
Add
Two
Equals
```

在本文 `Verified Baseline` 对应环境中曾观察到“内容实际为 JPEG、`mimeType` 却声明为 `image/png`”的 screenshot attachment。未来 revision 应先重新验证，不要假定该问题仍然存在。若当前环境实际复现，OpenCode 可能因严格校验真实文件签名而拒绝该附件。遇到此问题时：

1. 记录为上游 MIME 兼容问题；
2. 不要自动修改第三方源码；
3. 改用纯 AX 文本路径完成 Calculator 操作和结果验证；
4. 仍需关闭 Calculator，并调用 `computer_use_stop` 释放 runtime。

## Browser 状态与验收范围

如果已经安装 Chrome native host / extension，必须把“配置状态”和“真实控制验收”分开记录：

```text
Browser setup:
CONFIGURED / NOT CONFIGURED

Browser live Computer Use:
PASS / NOT TESTED / FAIL
```

`Chrome native host`、扩展或 setup 检查通过，只能证明 browser integration 已配置，不能证明 `computer_use_browser` 已完成真实网页控制。

如果本次任务没有明确要求 Browser live acceptance，可以记录为 `NOT TESTED`；这不影响桌面 PRIMARY/FALLBACK 的 PASS。但最终报告不得把 setup 成功描述成 browser live test 成功。

如果项目当前提供：

```bash
npm run test:opencode
```

也执行官方 OpenCode integration test。

官方完整测试可能硬编码英文 Calculator Description（例如 `All Clear`），在中文系统上会失败。先判断 AX 中是否存在稳定 ID（例如 `AllClear`），并以独立真实 GUI 测试作为最终验收依据。不得修改测试来制造通过。

---

# Phase 10：FALLBACK 独立验收

这一轮不要调用 PRIMARY。

只调用：

```text
macgui_*
```

执行 Calculator 或 TextEdit 最小测试：

```text
读取 screen / AX
→ 找到目标控件
→ click
→ type / key
→ 再次读取 screen 或 screenshot
→ 验证结果
```

验证：

- screenshot 正常；
- Accessibility 正常；
- click 正常；
- keyboard 正常；
- window/app 查询正常。

## 多显示器坐标检查

在点击前先调用 `macgui_computer_screeninfo` 和 `macgui_computer_list_windows`。

如果目标窗口位于负 X/Y 坐标的副屏：

1. 用 `macgui_computer_drag` 把窗口移动到主屏非负坐标区域；
2. 重新读取窗口 bounds、AX tree 和 screenshot；
3. 从新鲜 AX 数据计算按钮中心坐标；
4. 再执行 click，不要复用移动前的坐标。

本文 `Verified Baseline` 的部分多显示器布局中曾观察到：`macgui_computer_move` 和 `drag` 能正确处理负坐标，但 `macgui_computer_click` 可能错误重映射负坐标。只有在当前 revision 实际复现时，才采用“先拖到主屏再点击”的 workaround，并把现象记录为当前 revision 的上游兼容性限制。

## 官方测试兼容性

本文 `Verified Baseline` 对应 revision 的 `npm test` 会驱动 TextEdit，并对 AX tree 存在特定结构假设（例如 `AXTextArea`）。新版、中文系统或未来 revision 的行为可能不同。只有当前测试实际出现相同失败时，才按以下兼容性流程处理。

发生这种情况时：

- 记录已经通过的测试阶段和原始失败输出；
- 不要修改上游测试或源码来制造通过；
- 使用 Calculator 完成独立 screenshot、AX、click、keyboard、window/app 查询验收；
- 只有真实 GUI 验收也失败时，才把 FALLBACK 判定为安装失败。

测试结束后不要留下无关文档或窗口状态。

无论测试成功还是失败，都必须清理：

1. `macgui_computer_list_apps` 检查 Calculator 和 TextEdit；
2. `macgui_computer_list_windows` 检查残留窗口；
3. 使用 `macgui_computer_app` 的 quit 操作退出；
4. 再次确认两个 App 均无进程和窗口。

不要使用 `computer_applescript` 做常规清理。

---

# Phase 11：主备切换验收

不要真的破坏 Codex runtime。

通过明确指定方式模拟 PRIMARY 不可用：

```text
本轮禁止使用 computer_use，只允许使用 gui-fallback/macgui
```

让系统确认 fallback routing 可以成立。

该模拟应在交互式主 Agent 会话中通过 `task → gui-fallback` 执行，不要使用 `opencode run --agent gui-fallback`。验收时确认父 Agent 未调用任何 `computer_use*`，子 Agent 未调用 PRIMARY。

再恢复 PRIMARY。

最后确认正常默认路径仍然是：

```text
computer_use
```

而不是：

```text
macgui_*
```

恢复测试结束后调用 `computer_use_stop`，确保 capture/runtime 已释放。

---

# Phase 12：并发安全

检查现有 Sisyphus / Agent instructions。

增加一条非常明确的长期规则：

```text
GUI computer control is a serialized resource.

Never run computer_use and macgui_* concurrently.

Never allow two subagents to control the physical desktop,
mouse, keyboard, focused application, or GUI at the same time.

Use ashhart ComputerUse as primary.

Use macgui only as fallback after the primary path fails or is explicitly unavailable.
```

如果有适合放置 OpenCode 全局 Agent guidance 的已有配置文件，采用最小修改加入。

不要在每个项目目录重复添加这些规则。

优先放在全局配置/全局 Agent 指令中。

---

# Phase 13：最终检查

## Baseline / upgrade 状态

执行：

```bash
git -C ~/.local/share/opencode-tools/ComputerUse rev-parse HEAD
git -C ~/.local/share/opencode-tools/opencode-computer-use rev-parse HEAD
```

把结果写入最终报告。

如果任一 revision 不等于本文 `Verified Baseline`：

```text
Baseline status: UNVERIFIED UPGRADE
```

并确认已经对该新 revision 完整重新执行 Phase 9、10、11 和冲突/cleanup 检查。只有重新验收全部通过后，最终状态才可以是 PASS。


最终必须检查：

## PRIMARY

```text
ashhart checkout 存在
依赖安装正常
OpenCode plugin 存在
plugin 指向正确 checkout
Computer Use runtime 可连接
permissions 正常
computer_use 工具正常
```

## FALLBACK

```text
virajshoor checkout 存在
npm build 成功
Swift helper 存在
MCP server 名称 = macgui
OpenCode 可以连接 MCP
macgui_* 工具可用
```

## 冲突检查

确认不存在：

```text
ashhart OpenCode Plugin
+
ashhart MCP
```

这种重复。

确认 MCP 中不存在另一个无用的：

```text
computer-use
```

旧配置。

但如果发现它属于用户原有配置，不要直接删除；先判断来源，并在最终报告中说明。

确认没有相同 Tool Name collision。

确认不存在 GUI parallel execution configuration。

---

# Phase 14：不要提前结束

负责完成：

```text
调查
→ 安装
→ 配置
→ build
→ permissions diagnosis
→ OpenCode integration
→ PRIMARY test
→ FALLBACK test
→ routing test
→ config validation
→ final report
```

能自动完成的全部自己完成。

不要每一步都停下来询问。

只有以下情况才需要人工介入：

```text
macOS Accessibility 授权
macOS Screen Recording 授权
ChatGPT/Codex Desktop 中启用 Computer Use
Chrome 官方扩展安装
任何必须由 macOS 用户亲自确认的安全权限
```

出现这种情况时，只告诉用户具体需要点击什么；不要重新把整个任务交还给用户。

---

# 推荐执行模型

主 Agent 应使用当前 Oh My OpenAgent 已验证可用、且与 Sisyphus prompt 兼容的模型。不要在通用安装指南中硬编码某个可能下线、改名或与当前 Agent prompt 不兼容的模型。

`gui-fallback` 必须显式 pin 到当前已认证、经过简单调用验证的模型。不要假定默认 Provider、默认小模型或历史 OAuth 凭据仍然有效。

推荐先使用正常 reasoning 档位；只有在以下复杂情况中再临时提高推理强度：

适用场景包括：

- 复杂 `opencode.json/jsonc`
- 多个已有 MCP / Plugin
- Oh My OpenAgent 配置冲突
- Plugin / MCP tool registration 异常
- Codex Computer Use runtime 连接异常
- macOS TCC 权限状态异常
- 当前 OpenCode 版本与配置 schema 无法快速确认
- fallback subagent 模型继承或鉴权异常

不建议一开始就使用 `xhigh`。

---

# 最终报告格式

完成后给出：

```text
Computer Use Integration Report

OpenCode version:
Config schema (verified against version):
Config path:

PRIMARY — ashhart/ComputerUse
Status:
Install path:
Git revision:
Baseline status: VERIFIED / UNVERIFIED UPGRADE
Plugin path:
Runtime:
Permissions:
Registered tools:
Desktop integration test:
Browser setup:
Browser live Computer Use:
Result:

FALLBACK — virajshoor/opencode-computer-use
Status:
Install path:
Git revision:
Baseline status: VERIFIED / UNVERIFIED UPGRADE
MCP name:
MCP command:
Registered tools:
Integration test:
Result:

Routing
Primary:
Fallback:
Concurrent GUI control:
Fallback agent:
Permission policy:

Config changes:
Backups created:

Remaining manual actions:
Known limitations:

FINAL STATUS:
PASS / CONDITIONAL PASS / FAIL
```

本 runbook 的默认 FINAL STATUS 评估范围是 **桌面双 Computer Use 集成**。Browser live acceptance 只有在任务明确要求时才属于 PASS 必选项；否则必须如实记录 `Browser live Computer Use: NOT TESTED`。

只有在以下全部满足时才能给：

```text
PASS
```

- PRIMARY 真正完成一次 GUI 测试；
- FALLBACK 真正完成一次独立 GUI 测试；
- OpenCode 重启后工具仍然存在；
- 没有 duplicate ashhart tools；
- `macgui` MCP 正常连接；
- 主备 routing 明确；
- 没有并发 GUI 操作；
- 配置文件仍然有效；
- 两个 checkout 的实际 Git revision 已记录；若属于 `UNVERIFIED UPGRADE`，已经重新执行完整验收。

如果因为 macOS 人工权限尚未完成导致无法真实测试：

```text
CONDITIONAL PASS
```

并精确列出唯一剩余的人工步骤。

不要因为“安装成功”就把状态写成 PASS。
