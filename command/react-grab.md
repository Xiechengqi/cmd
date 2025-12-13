# React Grab 项目架构详解

> https://github.com/aidenybai/react-grab

## 项目概览

React Grab 是一个浏览器工具，允许用户通过快捷键（默认 CMD+C）选择页面元素，并将元素的上下文信息发送给编码代理（如 Cursor Agent）进行处理。

## 整体架构

```
┌─────────────────────────────────────────────────────────────────┐
│                        浏览器环境                                │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │            React Grab 核心 (react-grab)                   │ │
│  │  - 元素检测和选择                                          │ │
│  │  - UI 渲染 (SolidJS)                                       │ │
│  │  - 事件处理                                                │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          │                                      │
│                          ▼                                      │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │         Agent Manager (agent.ts)                         │ │
│  │  - 会话管理                                                │ │
│  │  - 流式处理                                                │ │
│  │  - 状态管理                                                │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          │                                      │
│                          ▼                                      │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │      Provider (如 @react-grab/cursor/client.ts)         │ │
│  │  - HTTP 请求封装                                           │ │
│  │  - SSE 流处理                                              │ │
│  │  - 连接管理                                                │ │
│  └──────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                          │
                          │ HTTPS/HTTP
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│                        服务器环境                                │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │    React Grab Server (@react-grab/cursor/server.ts)     │ │
│  │  - HTTP 服务器 (Hono)                                    │ │
│  │  - SSE 流式响应                                           │ │
│  │  - 请求路由                                               │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          │                                      │
│                          ▼                                      │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │         cursor-agent CLI                                  │ │
│  │  - 执行实际代码修改                                        │ │
│  │  - 流式输出 JSON                                          │ │
│  └──────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## 核心组件详解

### 1. React Grab 核心 (`packages/react-grab/src/core.tsx`)

**职责：**
- 初始化和管理整个 React Grab 系统
- 处理键盘和鼠标事件
- 检测和选择 DOM 元素
- 渲染 UI 覆盖层（选择框、标签等）
- 管理 Agent 集成

**关键流程：**

#### 初始化流程

```typescript
// 1. 用户加载页面，执行 init()
const api = init(options);

// 2. 创建 Agent Manager
const agentManager = createAgentManager(agentOptions);

// 3. 如果配置了 Agent Provider，注册它
if (hasAgentProvider()) {
  api.setAgent({ provider, storage: sessionStorage });
}
```

#### 激活流程（用户按 CMD+C）

```typescript
// 1. 监听键盘事件
window.addEventListener("keydown", (event) => {
  // 2. 检测是否按下 CMD+C (或 Ctrl+C)
  if (isTargetKeyCombination(event, options)) {
    // 3. 设置定时器，按住一定时间后激活
    holdTimerId = setTimeout(() => {
      activateRenderer(); // 激活渲染器
    }, keyHoldDuration);
  }
});
```

#### 元素选择流程

```typescript
// 1. 鼠标移动时检测元素
window.addEventListener("mousemove", (event) => {
  const element = getElementAtPosition(event.clientX, event.clientY);
  setDetectedElement(element);
});

// 2. 点击元素时
handlePointerUp(clientX, clientY) {
  const element = getElementAtPosition(clientX, clientY);
  
  // 3. 如果有 Agent Provider，进入输入模式
  if (hasAgentProvider()) {
    setIsInputMode(true);
    setFrozenElement(element);
  } else {
    // 4. 否则直接复制到剪贴板
    copySingleElementToClipboard(element);
  }
}
```

### 2. Agent Manager (`packages/react-grab/src/agent.ts`)

**职责：**
- 管理多个 Agent 会话
- 处理流式响应
- 管理会话状态和持久化
- 处理重试、中止、撤销等操作

**关键数据结构：**

```typescript
interface AgentSession {
  id: string;                    // 会话 ID
  context: AgentContext;         // 上下文（元素内容、提示等）
  lastStatus: string;            // 最后状态消息
  isStreaming: boolean;          // 是否正在流式传输
  createdAt: number;              // 创建时间
  position: { x: number; y: number }; // UI 位置
  selectionBounds?: OverlayBounds;    // 元素边界
  error?: string;                // 错误信息
}
```

**启动会话流程：**

```typescript
const startSession = async (params: StartSessionParams) => {
  // 1. 生成元素上下文
  const content = await generateSnippet([element], { maxLines: Infinity });
  
  // 2. 创建 AgentContext
  const context: AgentContext = {
    content,        // 元素的 HTML/代码片段
    prompt,         // 用户输入的提示
    options: {...}, // Agent 选项（如模型、工作目录）
    sessionId,      // 会话 ID（用于后续对话）
  };
  
  // 3. 创建会话
  const session = createSession(context, position, selectionBounds, ...);
  
  // 4. 调用 Provider 的 send 方法
  const streamIterator = agentOptions.provider.send(
    contextWithSessionId,
    abortController.signal,
  );
  
  // 5. 处理流式响应
  void executeSessionStream(session, streamIterator);
};
```

**流式处理流程：**

```typescript
const executeSessionStream = async (session, streamIterator) => {
  // 1. 迭代流式响应
  for await (const status of streamIterator) {
    // 2. 更新会话状态
    const updatedSession = updateSession(session, { lastStatus: status });
    setSessions(prev => new Map(prev).set(session.id, updatedSession));
    
    // 3. 触发状态回调
    agentOptions?.onStatus?.(status, updatedSession);
  }
  
  // 4. 完成时触发回调
  agentOptions?.onComplete?.(completedSession, element);
};
```

### 3. Cursor Provider 客户端 (`packages/provider-cursor/src/client.ts`)

**职责：**
- 封装与服务器的 HTTP 通信
- 处理 Server-Sent Events (SSE) 流
- 管理连接状态
- 提供统一的 Agent Provider 接口

**关键方法：**

#### `send` - 发送请求到服务器

```typescript
send: async function* (context: CursorAgentContext, signal: AbortSignal) {
  // 1. 构建请求
  const response = await fetch(`${serverUrl}/agent`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(context), // 包含 content, prompt, options, sessionId
    signal, // 用于取消请求
  });
  
  // 2. 处理 SSE 流
  const iterator = streamSSE(response.body, signal);
  
  // 3. 迭代流式响应
  for await (const status of iterator) {
    yield status; // 返回状态消息给 Agent Manager
  }
}
```

#### `streamSSE` - 解析 SSE 流

```typescript
// 从 @react-grab/utils/client
// SSE 格式: "data: status message\n\n"
// 解析并提取 data 字段的内容
```

### 4. Cursor Server (`packages/provider-cursor/src/server.ts`)

**职责：**
- 接收来自浏览器的 HTTP 请求
- 调用 cursor-agent CLI
- 将 CLI 输出转换为 SSE 流
- 管理会话和进程

**请求处理流程：**

```typescript
app.post("/agent", async (context) => {
  const body = await context.req.json<CursorAgentContext>();
  const { content, prompt, options, sessionId } = body;
  
  // 1. 判断是否是后续对话
  const cursorChatId = sessionId ? cursorSessionMap.get(sessionId) : undefined;
  const isFollowUp = Boolean(cursorChatId);
  
  // 2. 构建用户提示
  const userPrompt = isFollowUp ? prompt : `${prompt}\n\n${content}`;
  
  // 3. 返回 SSE 流
  return streamSSE(context, async (stream) => {
    // 4. 构建 cursor-agent 命令参数
    const cursorAgentArgs = [
      "--print",
      "--output-format", "stream-json",
      "--force",
    ];
    
    if (isFollowUp && cursorChatId) {
      cursorAgentArgs.push("--resume", cursorChatId);
    }
    
    // 5. 启动 cursor-agent 进程
    const cursorProcess = execa("cursor-agent", cursorAgentArgs, {
      stdin: "pipe",
      stdout: "pipe",
      stderr: "pipe",
      cwd: workspacePath,
    });
    
    // 6. 写入用户提示到 stdin
    cursorProcess.stdin.write(userPrompt);
    cursorProcess.stdin.end();
    
    // 7. 读取 stdout，解析 JSON 流
    cursorProcess.stdout.on("data", async (chunk: Buffer) => {
      const event = parseStreamLine(chunk.toString());
      
      // 8. 根据事件类型发送 SSE
      switch (event.type) {
        case "assistant":
          await stream.writeSSE({ data: textContent, event: "status" });
          break;
        case "result":
          if (event.subtype === "success") {
            await stream.writeSSE({ data: "Completed successfully", event: "status" });
          }
          break;
      }
    });
  });
});
```

## 完整调用流程

### 场景：用户选择元素并调用 Agent

```
1. 用户操作
   └─> 按 CMD+C 激活 React Grab
   └─> 鼠标移动到元素上
   └─> 点击元素
   └─> 输入提示："Change the button color to blue"
   └─> 按 Enter 提交

2. React Grab 核心 (core.tsx)
   └─> handleInputSubmit()
       └─> 获取元素和提示
       └─> agentManager.startSession({
             element,
             prompt: "Change the button color to blue",
             position: { x, y },
             selectionBounds: bounds,
           })

3. Agent Manager (agent.ts)
   └─> startSession()
       └─> generateSnippet([element]) // 生成元素代码片段
       └─> 创建 AgentContext {
             content: "<button>Click me</button>",
             prompt: "Change the button color to blue",
             options: { model: "claude-3-opus" },
           }
       └─> provider.send(context, abortSignal)

4. Cursor Provider 客户端 (client.ts)
   └─> streamFromServer()
       └─> fetch("https://prompts.xiechengqi.top/agent", {
             method: "POST",
             body: JSON.stringify({
               content: "<button>Click me</button>",
               prompt: "Change the button color to blue",
               options: { model: "claude-3-opus" },
             }),
           })
       └─> 解析 SSE 流
       └─> yield "Thinking…"
       └─> yield "Analyzing code..."
       └─> yield "Making changes..."
       └─> yield "Completed successfully"

5. React Grab Server (server.ts)
   └─> POST /agent
       └─> 解析请求体
       └─> 构建 cursor-agent 命令
       └─> execa("cursor-agent", ["--print", "--output-format", "stream-json"])
       └─> 写入提示到 stdin
       └─> 读取 stdout，解析 JSON
       └─> 发送 SSE 事件

6. cursor-agent CLI
   └─> 接收提示和代码上下文
   └─> 调用 AI 模型
   └─> 生成代码修改
   └─> 应用修改到文件系统
   └─> 输出流式 JSON 状态

7. 响应流回浏览器
   └─> Agent Manager 接收状态更新
   └─> 更新 UI 显示进度
   └─> 完成时显示结果
```

## 数据流详解

### AgentContext 结构

```typescript
interface AgentContext {
  content: string;      // 元素的 HTML/代码片段
  prompt: string;       // 用户输入的提示
  options?: {           // Agent 特定选项
    model?: string;     // AI 模型
    workspace?: string;  // 工作目录
  };
  sessionId?: string;    // 会话 ID（用于后续对话）
}
```

### SSE 消息格式

服务器发送的 SSE 消息：

```
event: status
data: Thinking…

event: status
data: Analyzing code...

event: status
data: Making changes...

event: status
data: Completed successfully

event: done
data:
```

### cursor-agent 输出格式

cursor-agent CLI 输出 JSON 流：

```json
{"type":"system","subtype":"init"}
{"type":"thinking","message":{"role":"thinking","content":[{"type":"text","text":"Analyzing..."}]}}
{"type":"assistant","message":{"role":"assistant","content":[{"type":"text","text":"I'll change the button color to blue."}]}}
{"type":"result","subtype":"success","result":"Changes applied successfully","session_id":"chat-123"}
```

服务器解析这些 JSON 并转换为 SSE 事件。

## 关键设计模式

### 1. Provider 模式

Agent Provider 是一个接口，允许不同的后端实现：

```typescript
interface AgentProvider {
  send: (context, signal) => AsyncIterable<string>;
  resume?: (sessionId, signal, storage) => AsyncIterable<string>;
  abort?: (sessionId) => Promise<void>;
  checkConnection?: () => Promise<boolean>;
}
```

这使得可以轻松支持不同的 Agent（Cursor、Claude Code、OpenCode 等）。

### 2. 流式处理

使用 AsyncGenerator 实现流式处理：

```typescript
async function* streamFromServer(...) {
  const response = await fetch(...);
  const iterator = streamSSE(response.body);
  
  for await (const status of iterator) {
    yield status; // 逐步返回状态
  }
}
```

### 3. 会话管理

会话存储在 `sessionStorage` 中，支持：
- 页面刷新后恢复会话
- 多标签页共享（如果使用 localStorage）
- 错误重试

### 4. 事件驱动

使用 SolidJS 的响应式系统：
- 状态变化自动触发 UI 更新
- 事件监听器管理用户交互
- 清理函数确保无内存泄漏

## 浏览器端关键代码路径

### 元素选择 → Agent 调用

```typescript
// core.tsx:1209
const handleInputSubmit = () => {
  const element = frozenElement() || targetElement();
  const prompt = inputText().trim();
  
  if (hasAgentProvider() && prompt) {
    // 调用 Agent Manager
    agentManager.startSession({
      element,
      prompt,
      position: { x: labelPositionX, y: currentY },
      selectionBounds: bounds,
      sessionId: currentReplySessionId ?? undefined,
    });
  }
};
```

### Agent Manager → Provider

```typescript
// agent.ts:311
const streamIterator = agentOptions.provider.send(
  contextWithSessionId,
  abortController.signal,
);
void executeSessionStream(session, streamIterator);
```

### Provider → HTTP 请求

```typescript
// client.ts:52
const response = await fetch(`${serverUrl}/agent`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify(context),
  signal,
});
```

## 服务器端关键代码路径

### HTTP 请求 → cursor-agent

```typescript
// server.ts:107
cursorProcess = execa("cursor-agent", cursorAgentArgs, {
  stdin: "pipe",
  stdout: "pipe",
  stderr: "pipe",
  cwd: workspacePath,
});

cursorProcess.stdin.write(userPrompt);
```

### cursor-agent 输出 → SSE

```typescript
// server.ts:166
cursorProcess.stdout.on("data", async (chunk: Buffer) => {
  const event = parseStreamLine(chunk.toString());
  
  if (event.type === "assistant") {
    await stream.writeSSE({ data: textContent, event: "status" });
  }
});
```

## 总结

React Grab 的架构采用了清晰的分层设计：

1. **浏览器层**：React Grab 核心处理用户交互和 UI
2. **管理层**：Agent Manager 管理会话和状态
3. **通信层**：Provider 封装 HTTP/SSE 通信
4. **服务器层**：Server 接收请求并调用 CLI
5. **执行层**：cursor-agent CLI 执行实际代码修改

这种设计使得：
- ✅ 关注点分离
- ✅ 易于扩展（支持新的 Agent）
- ✅ 流式处理提供实时反馈
- ✅ 会话管理支持复杂交互
- ✅ 错误处理和重试机制完善
