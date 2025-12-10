opencode
===

AI 驱动的终端编码助手，直接在命令行中协助开发任务

## 补充说明

**opencode命令** 是一个强大的 AI 编码助手，专为终端环境设计。它利用大型语言模型（LLM）的能力，直接在命令行中帮助开发者完成各种软件工程任务，包括代码分析、功能开发、Bug 修复、代码重构等。

opencode 提供了直观的文本用户界面（TUI），支持多种工作模式、自定义配置、以及与主流 LLM 提供商的无缝集成。

### 核心特性

* **双模式工作流**：Plan 模式用于规划和分析，Build 模式用于实际修改代码
* **智能文件引用**：使用 `@` 符号进行模糊搜索并引用项目文件
* **可视化辅助**：支持拖放图片到终端，AI 可分析图像并作为参考
* **撤销/重做**：完整的操作历史管理，可随时回退或恢复更改
* **对话分享**：生成可分享的对话链接，便于团队协作
* **项目感知**：通过 `AGENTS.md` 文件理解项目结构和编码规范

---

## CLI（命令行界面）和 TUI（终端用户界面）

* CLI（命令行界面）和 TUI（终端用户界面）在 OpenCode 中代表了两种主要的交互方式：

### TUI（终端用户界面）
TUI 是 OpenCode 提供的交互式终端界面，用于使用大型语言模型（LLM）处理项目。

* 默认启动： 默认情况下，OpenCode CLI 在不带任何参数运行时会启动 TUI。
* 交互性： TUI 是一个完整的交互界面，用户可以在其中输入消息（提示），使用斜杠命令（例如 /connect, /undo, /share）快速执行操作，以及使用键盘快捷键（例如 ctrl+x 作为 Leader key）。
* 用户体验： 在 TUI 中，用户可以通过 @ 符号进行模糊文件搜索并将文件内容添加到对话中，或者使用 ! 运行 shell 命令，其输出会作为工具结果添加到对话中。
* 架构： TUI 充当连接到 OpenCode 服务器的客户端。OpenCode 1.0 版本对 TUI 进行了彻底重写，从基于 go+bubbletea 转移到使用 zig+solidjs 编写的内部框架 OpenTUI。

### CLI（命令行界面）
OpenCode CLI 是执行程序和非交互式任务的工具。
* 程序化交互： CLI 接受命令，允许用户以程序化的方式与 OpenCode 交互。
* 非交互模式： CLI 允许通过直接传递提示来在非交互模式下运行 OpenCode。这种模式对于脚本编写、自动化或在不需要启动完整 TUI 的情况下快速获得答案非常有用。
* 命令功能： CLI 包含多种命令，例如：
  * tui：启动 OpenCode 终端用户界面。
  * agent：管理 OpenCode 的代理。
  * auth：管理提供商的凭据和登录。
  * models：列出所有可用的模型。
  * run：在非交互模式下运行 OpenCode，直接传递提示。
  * serve：启动一个无头（headless）OpenCode 服务器，以进行 API 访问。

简而言之，TUI 提供了一个交互式、富界面的环境（类似终端中的应用程序），是日常开发工作的默认选择；而 CLI 则用于启动 TUI 或执行程序化、非交互式的任务和管理命令。

## Build（构建）代理和 Plan（计划）

 TUI（终端用户界面）Build（构建）代理和 Plan（计划）代理是两个内置的主要代理（Primary agents），它们旨在用于不同的开发工作流，主要区别在于它们对工具（尤其是修改代码和执行系统命令的工具）的访问权限上

<img width="678" height="553" alt="image" src="https://github.com/user-attachments/assets/90b25c93-ef3e-4cf9-9f8f-b78560ce339d" />

## 快速入门

### 前置要求

使用 opencode 前需要准备：

1. **终端**

2. **LLM 提供商的 API Key**
   - 支持 OpenAI、Anthropic、Google、OpenRouter 等多家提供商
   - 推荐新手使用 [OpenCode Zen](https://opencode.ai/auth) 快速开始

### 安装

**推荐方式（自动安装脚本）**：

```shell
curl -fsSL https://opencode.ai/install | bash
```

**使用 Node.js 包管理器**：

```shell
# npm
npm install -g opencode-ai

# pnpm
pnpm install -g opencode-ai

# Yarn
yarn global add opencode-ai

# Bun
bun install -g opencode-ai
```

**使用 Homebrew（macOS/Linux）**：

```shell
brew install opencode
```

**验证安装**：

```shell
opencode --version
```

### 首次配置

**1. 配置 API Key**

启动 opencode 后，使用 `/connect` 命令配置 LLM 提供商：

```shell
# 启动 opencode
opencode

# 在 TUI 中运行
/connect
```

选择提供商（推荐新手选择 `opencode`），然后：
- 访问 [opencode.ai/auth](https://opencode.ai/auth) 获取 API Key
- 在提示符处粘贴 API Key

**2. 初始化项目**

进入项目目录并初始化：

```shell
cd /path/to/your/project
opencode
```

在 TUI 中运行：

```shell
/init
```

这会让 opencode 分析项目并生成 `AGENTS.md` 文件，帮助它理解项目结构和编码模式。

> **提示**：建议将 `AGENTS.md` 文件提交到 Git 仓库。

**3. 开始使用**

现在可以向 opencode 提问或请求帮助了！

---

## 使用指南

### 基本语法

```shell
# 启动交互式 TUI
opencode [项目路径]

# 非交互模式（直接执行）
opencode run "你的问题或任务"

# 继续上次会话
opencode --continue

# 使用特定模型
opencode --model anthropic/claude-sonnet-4-20250514
```

### 常用选项

| 选项 | 简写 | 说明 |
|------|------|------|
| `--help` | `-h` | 显示帮助信息 |
| `--version` | `-v` | 显示版本号 |
| `--continue` | `-c` | 继续上次会话 |
| `--session` | `-s` | 指定会话 ID 继续 |
| `--prompt` | `-p` | 指定初始提示词 |
| `--model` | `-m` | 指定使用的模型（格式：`提供商/模型名`） |
| `--print-logs` | | 将日志输出到 stderr |
| `--log-level` | | 设置日志级别（DEBUG, INFO, WARN, ERROR） |

### TUI 内命令

在 opencode 的交互界面中，可以使用以下命令：

| 命令 | 功能 |
|------|------|
| `/connect` | 配置 LLM 提供商 API Key |
| `/init` | 分析项目并创建 AGENTS.md |
| `/undo` | 撤销上一次的代码更改 |
| `/redo` | 重做被撤销的更改 |
| `/share` | 生成可分享的对话链接 |
| `Tab` | 在 Plan 和 Build 模式间切换 |

### 常见工作流

#### 1. 询问和理解代码

使用 `@` 符号引用文件（支持模糊搜索）：

```shell
# 询问特定文件的功能
@src/auth/index.ts 这个文件是如何处理身份验证的？

# 理解多个文件的关系
@api/routes @middleware/auth 这两个模块是如何协同工作的？
```

#### 2. 添加新功能（推荐工作流）

**步骤 1：切换到 Plan 模式**

按 `Tab` 键切换到 Plan 模式（右下角会显示 "Plan"）。

**步骤 2：描述需求**

```text
我想实现一个软删除功能。当用户删除笔记时，不是真正删除，
而是在数据库中标记为已删除。然后创建一个"回收站"页面，
显示所有被标记为删除的笔记。用户可以从回收站恢复笔记或
永久删除它们。
```

**步骤 3：迭代优化计划**

根据 opencode 给出的计划，你可以：

```text
# 提供设计参考
[拖放设计图到终端] 请参考这个设计图来实现回收站页面

# 补充细节
请在删除按钮旁边添加确认对话框

# 修正计划
不要使用 Modal 组件，使用 Drawer 组件代替
```

**步骤 4：执行实现**

确认计划后，按 `Tab` 切换回 Build 模式：

```text
计划看起来不错！请按照这个方案实施。
```

#### 3. 直接修改代码

对于简单直接的更改，可以在 Build 模式下直接描述：

```text
@src/routes/settings.ts
请参考 @src/routes/notes.ts 中的身份验证实现方式，
在 settings 路由中添加相同的身份验证逻辑。
```

#### 4. 撤销和重做

如果对结果不满意：

```shell
# 撤销最近的更改
/undo

# 可以多次撤销
/undo
/undo

# 重做被撤销的更改
/redo
```

---

## 实用示例

### Bug 修复工作流

```shell
# 1. 启动 opencode
opencode

# 2. 描述问题（带上相关文件）
@src/api/users.ts 用户登录时偶尔会返回 500 错误，
请帮我分析可能的原因并修复。

# 3. 如果需要查看日志
能否检查一下错误日志？日志文件在 logs/app.log

# 4. 确认修复方案后
好的，请实施这个修复方案，并添加相应的错误处理。
```

### 代码重构

```shell
# 1. 先用 Plan 模式分析
# 按 Tab 切换到 Plan 模式

@src/utils/helpers.ts 这个文件的函数太多了，
请分析一下如何将其重构成多个模块，保持单一职责原则。

# 2. 评估计划
这个拆分方案看起来合理。请确保所有导入路径都相应更新。

# 3. 切换到 Build 模式执行
# 按 Tab 切换回 Build 模式
请执行重构，并运行测试确保没有破坏现有功能。
```

### 性能优化

```shell
@components/DataTable.tsx 这个组件在处理大量数据时
渲染很慢，请分析性能瓶颈并提供优化建议。

# 提供更多上下文
数据量通常在 1000-5000 行之间，包含 10 列。
用户需要能够排序和筛选。
```

### 编写测试

```shell
@src/services/payment.ts 请为这个支付服务模块编写
完整的单元测试，覆盖正常流程和各种异常情况。

# 指定测试框架
我们使用 Jest 作为测试框架，请遵循现有的测试模式
@tests/services/user.test.ts
```

### 生成文档

```shell
@src/api/*.ts 请分析这些 API 端点，并生成
OpenAPI/Swagger 格式的 API 文档。

# 或者为函数添加注释
@src/utils/encryption.ts 请为这个加密工具模块
的所有导出函数添加 JSDoc 注释。
```

---

## 工作模式详解

### Build 模式（默认）

**用途**：日常开发工作，完整的文件操作和命令执行权限。

**可用工具**：
- ✅ 读取文件（read）
- ✅ 编辑文件（edit）
- ✅ 创建文件（write）
- ✅ 执行命令（bash）
- ✅ 搜索代码（grep, glob）
- ✅ 应用补丁（patch）
- ✅ 网络请求（webfetch）

**适用场景**：
- 实现新功能
- 修复 Bug
- 代码重构
- 编写测试
- 生成文档

### Plan 模式

**用途**：纯分析和规划，不会修改任何代码。

**禁用工具**：
- ❌ 编辑文件（edit）
- ❌ 创建文件（write）
- ❌ 应用补丁（patch）
- ❌ 执行命令（bash）

**保留工具**：
- ✅ 读取文件（read）
- ✅ 搜索代码（grep, glob）

**适用场景**：
- 理解陌生代码
- 制定实现方案
- 评估技术选型
- 代码审查
- 架构分析

### 模式切换

在 TUI 中按 `Tab` 键即可在 Plan 和 Build 模式间切换，右下角会显示当前模式。

---

## 高级配置

### 配置文件位置

opencode 支持两级配置：

- **全局配置**：`~/.config/opencode/opencode.json`
- **项目配置**：`<项目根目录>/opencode.json`

项目配置会覆盖全局配置。

### 配置示例

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "anthropic/claude-sonnet-4-20250514",
  "agent": {
    "build": {
      "model": "anthropic/claude-sonnet-4-20250514",
      "temperature": 0.3
    },
    "plan": {
      "model": "anthropic/claude-haiku-4-20250514",
      "temperature": 0.1
    }
  },
  "theme": "opencode",
  "keybinds": {
    "submit": ["ctrl+enter"],
    "switch_mode": ["tab"]
  }
}
```

### 自定义工作模式

创建 `.opencode/mode/review.md`：

```markdown
---
model: anthropic/claude-sonnet-4-20250514
temperature: 0.1
tools:
  write: false
  edit: false
  bash: false
---

你处于代码审查模式。专注于：

- 代码质量和最佳实践
- 潜在的 Bug 和边界情况
- 性能影响
- 安全性考虑

提供建设性反馈，但不直接修改代码。
```

### 管理 LLM 提供商

```shell
# 查看已配置的提供商
opencode auth list

# 添加新提供商
opencode auth login

# 移除提供商
opencode auth logout

# 查看可用模型
opencode models
```

### 非交互式使用

适合脚本和自动化场景：

```shell
# 直接执行单个任务
opencode run "解释 React 中的 useEffect 钩子"

# 附加文件
opencode run -f screenshot.png "这个设计图应该如何实现？"

# 继续特定会话
opencode run -s <session-id> "继续上次的讨论"

# 生成可分享链接
opencode run --share "创建一个 TODO 应用"

# 输出 JSON 格式（便于解析）
opencode run --format json "列出项目中所有的 API 端点"
```

### 启动无头服务

```shell
# 启动 HTTP API 服务
opencode serve --port 4096

# 在另一个终端连接到服务（避免 MCP 服务器冷启动）
opencode run --attach http://localhost:4096 "你的任务"
```

---

## 数据存储

### 日志文件

**位置**：
- macOS/Linux：`~/.local/share/opencode/log/`
- Windows：`%USERPROFILE%\.local\share\opencode\log\`

**特点**：
- 日志文件按时间戳命名（如 `2025-12-10T123456.log`）
- 保留最近 10 个日志文件
- 使用 `--log-level DEBUG` 获取详细日志

### 应用数据

**位置**：
- macOS/Linux：`~/.local/share/opencode/`
- Windows：`%USERPROFILE%\.local\share\opencode\`

**包含**：
- `auth.json` - API Key 和认证令牌
- `log/` - 应用日志
- `project/` - 项目会话数据
  - Git 仓库：`./<项目标识>/storage/`
  - 非 Git 目录：`./global/storage/`

---

## 故障排除

### OpenCode 无法启动

**症状**：运行 `opencode` 命令后无响应或崩溃。

**解决方案**：
1. 查看日志文件查找错误信息
2. 使用 `--print-logs` 在终端查看输出：
   ```shell
   opencode --print-logs
   ```
3. 升级到最新版本：
   ```shell
   opencode upgrade
   ```

### 身份验证失败

**症状**：提示 API Key 无效或认证错误。

**解决方案**：
1. 在 TUI 中重新认证：
   ```shell
   /connect
   ```
2. 检查 API Key 是否有效（访问提供商控制台确认）
3. 确认网络能访问提供商 API
4. 检查环境变量或 `.env` 文件中是否有冲突的配置

### 模型不可用 (ProviderModelNotFoundError)

**症状**：提示找不到指定的模型。

**原因**：模型名称格式错误或模型不存在。

**解决方案**：
1. 检查模型名称格式是否正确：`<提供商ID>/<模型ID>`
   - ✅ 正确：`openai/gpt-4.1`
   - ✅ 正确：`anthropic/claude-sonnet-4-20250514`
   - ✅ 正确：`openrouter/google/gemini-2.5-flash`
   - ❌ 错误：`gpt-4.1`（缺少提供商前缀）

2. 查看可用模型列表：
   ```shell
   opencode models
   ```

3. 确认已认证相应的提供商

### 提供商初始化错误 (ProviderInitError)

**症状**：启动时提示提供商初始化失败。

**原因**：配置文件损坏或格式错误。

**解决方案**：
1. 参考[提供商配置指南](https://opencode.ai/docs/providers)检查配置
2. 如果问题持续，清除配置并重新设置：
   ```shell
   rm -rf ~/.local/share/opencode
   ```
3. 重新启动 opencode 并使用 `/connect` 重新认证

### API 调用错误 (AI_APICallError)

**症状**：请求模型时出现 API 错误。

**原因**：提供商包（OpenAI、Anthropic 等）版本过旧。

**解决方案**：
1. 清除提供商包缓存：
   ```shell
   rm -rf ~/.cache/opencode
   ```
2. 重启 opencode，它会自动下载最新版本的提供商包

### Linux 复制/粘贴不工作

**症状**：在 Linux 上无法复制或粘贴内容。

**原因**：缺少剪贴板工具。

**解决方案**：

**X11 系统**：
```shell
# Debian/Ubuntu
sudo apt install -y xclip
# 或
sudo apt install -y xsel
```

**Wayland 系统**：
```shell
sudo apt install -y wl-clipboard
```

**无头环境（Headless）**：
```shell
sudo apt install -y xvfb
# 启动虚拟显示
Xvfb :99 -screen 0 1024x768x24 > /dev/null 2>&1 &
export DISPLAY=:99.0
```

### 性能问题

**症状**：响应缓慢或处理大型项目时卡顿。

**建议**：
1. 使用 `.gitignore` 排除不必要的文件（`node_modules`、构建产物等）
2. 在 Plan 模式下进行大规模分析（更快的模型）
3. 使用无头服务模式避免重复的 MCP 服务器启动：
   ```shell
   # 终端 1
   opencode serve

   # 终端 2
   opencode run --attach http://localhost:4096 "你的任务"
   ```

---

## 集成与扩展

### GitHub Actions 集成

opencode 可以作为 GitHub Actions 工作流的一部分：

```shell
# 安装 GitHub Agent
opencode github install
```

这会设置必要的 GitHub Actions 工作流。[详细文档](https://opencode.ai/docs/github)

### GitLab 集成

类似地，opencode 也支持 GitLab CI/CD。[详细文档](https://opencode.ai/docs/gitlab)

### MCP 服务器

opencode 支持模型上下文协议（MCP）服务器，可以扩展额外的工具和能力。[详细文档](https://opencode.ai/docs/mcp-servers)

### IDE 集成

虽然 opencode 主要设计为终端工具，但也可以通过 LSP 或插件与编辑器集成。[详细文档](https://opencode.ai/docs/ide)

### 自定义命令

在项目中创建 `.opencode/commands/` 目录，添加自定义命令：

```markdown
<!-- .opencode/commands/review.md -->
---
description: 执行代码审查
---

请审查最近的代码更改，重点关注：
- 潜在的 Bug
- 性能问题
- 安全漏洞
- 代码风格一致性
```

使用时运行：
```shell
/review
```

---

## 使用建议

### 最佳实践

1. **提供充分的上下文**
   - 使用 `@` 引用相关文件
   - 说明项目背景和目标
   - 提供示例和参考

2. **先规划后实施**
   - 复杂任务先用 Plan 模式制定方案
   - 确认计划后再切换到 Build 模式
   - 逐步实施，而非一次性大改

3. **善用撤销功能**
   - 不确定的更改可以先尝试
   - 随时用 `/undo` 回退
   - 保持小步迭代

4. **维护 AGENTS.md**
   - 定期运行 `/init` 更新项目分析
   - 将 `AGENTS.md` 提交到版本控制
   - 团队成员共享同一个 `AGENTS.md`

5. **选择合适的模型**
   - 简单任务使用快速模型（Haiku、Gemini Flash）
   - 复杂任务使用强大模型（Sonnet、GPT-4）
   - 在配置中为不同模式设置不同模型

### 提示词技巧

**明确具体**：
```text
❌ 优化这个函数
✅ 优化 @src/utils/sort.ts 中的 quickSort 函数，
   减少递归深度，并添加针对小数组的优化
```

**提供约束**：
```text
✅ 添加用户登录功能，使用 JWT 认证，
   session 有效期 7 天，支持记住我功能
```

**参考现有代码**：
```text
✅ 参考 @components/UserCard.tsx 的样式，
   创建一个类似的 ProductCard 组件
```

---

## 安全与隐私

### 数据处理

- **本地优先**：代码在本地处理，仅发送必要内容到 LLM
- **不自动分享**：对话默认不会分享，除非主动使用 `/share`
- **敏感信息保护**：API Key 加密存储在本地

### 安全实践

- 避免将包含密钥的文件（`.env`、`credentials.json`）提交到版本控制
- 定期轮换 API Key
- 在 `.gitignore` 中排除敏感文件
- 审查 opencode 的更改，不要盲目接受所有建议

---

## 资源链接

- **官方网站**：[opencode.ai](https://opencode.ai)
- **完整文档**：[opencode.ai/docs](https://opencode.ai/docs)
- **GitHub 仓库**：[github.com/sst/opencode](https://github.com/sst/opencode)
- **问题反馈**：[github.com/sst/opencode/issues](https://github.com/sst/opencode/issues)
- **Discord 社区**：[opencode.ai/discord](https://opencode.ai/discord)
- **获取 API Key**：[opencode.ai/auth](https://opencode.ai/auth)

---

## 扩展阅读

- [主题定制](/docs/themes) - 个性化 TUI 外观
- [快捷键配置](/docs/keybinds) - 自定义键盘快捷键
- [代码格式化器](/docs/formatters) - 配置代码格式化工具
- [自定义命令](/docs/commands) - 创建项目特定命令
- [LLM 提供商](/docs/providers) - 配置不同的 AI 模型提供商
- [OpenCode SDK](/docs/sdk) - 使用 SDK 构建自定义集成
