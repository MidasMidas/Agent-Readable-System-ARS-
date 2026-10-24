# Agent Readable System (ARS)

### A Shared Language for Agents, Applications, and Operating Systems

**让系统对 AI 可读、可操作、可验证，让用户始终掌握控制权。**

> GUI makes systems accessible to people.  
> ARS makes systems readable and actionable for agents.

**建议仓库名称：`agent-readable-system`**

Agent Readable System（ARS）旨在探索一套跨操作系统的架构标准，让 AI Agent 通过统一、结构化、受权限控制的方式读取系统与应用状态、执行操作并验证结果。

ARS 可以通过现有操作系统上的适配层、应用框架桥接和运行时实现，也可以进一步集成到操作系统框架与系统服务中。推进 ARS 不要求先构建一个全新的操作系统。本文描述拟议架构，不代表已经发布的标准或可用于生产的实现。

## 1. 项目愿景

计算机操作系统的交互方式经历了长期演进。

### 命令行时代（CLI）

```text
Human → Command → Operating System
```

人类通过输入命令操作计算机。

这种方式高效、精确，但要求用户了解计算机命令及其参数。

### 图形界面时代（GUI）

```text
Human → GUI → Operating System
```

图形用户界面通过窗口、图标、按钮和菜单，将复杂的系统操作转化为直观的视觉交互。

GUI 极大降低了计算机的使用门槛。

然而，GUI 的设计对象始终是人类。

### 人工智能时代（Agent）

随着人工智能的发展，用户开始通过自然语言向 AI 表达自己的意图。

当前主流方案通常采用以下方式：

```text
Human
  ↓
AI Agent
  ↓
Screenshot / UI Automation / Registered API
  ↓
Application
  ↓
Operating System
```

AI 需要通过屏幕截图、视觉识别、模拟点击，或者调用应用开发者专门注册的功能，才能操作应用程序。

这些方法能够解决部分问题，但并不是面向 AI 原生设计的操作系统交互架构。

**Agent Readable System (ARS) 希望改变这一点。**

我们的目标是：

```text
                 Human
                   │
             Natural Language
                   │
                   ▼
                AI Agent
                   │
                   ▼
           ARS Agent Interface
                   │
                   ▼
             Operating System
                   │
            ┌──────┴──────┐
            ▼             ▼
      System Services  Applications
```

AI 应当成为操作系统正式支持的操作主体。

GUI 不再是 AI 操作设备时必须经过的中间层。

---

## 2. 核心理念

### 2.1 AI 不应该被迫模拟人类

传统 GUI 通过图像向人类传递信息。

AI 并不一定需要通过像素理解界面。

例如，一个按钮在应用内部可能具有以下属性：

```json
{
  "role": "button",
  "label": "加入购物车",
  "enabled": true,
  "actions": ["click"]
}
```

既然应用的 UI 框架已经掌握这些信息，操作系统就应该能够通过标准接口将其提供给 AI。

AI 不需要先截图，再通过视觉模型识别按钮的位置。

对于具有结构化语义的界面，可以直接查询并执行操作。

### 2.2 AI 和 GUI 应当是平等的交互方式

未来的操作系统应当同时支持：

**Human Interface**

- 图形界面
- 触摸操作
- 鼠标与键盘
- 手势交互

**ARS Agent Interface**

- 应用发现
- 结构化状态查询
- 控件操作
- 系统能力调用
- 状态变化通知
- 操作结果验证

两种接口应当共享同一套应用状态和业务逻辑。

AI 执行的点击操作，应尽可能触发与人类点击相同的应用行为。

用户可以随时通过 GUI 查看、修改或接管 AI 的操作。

### 2.3 开发者不应该为每一种 AI 重复适配

目前，一些 AI 集成方式要求开发者主动注册业务功能。

例如：

```text
searchProduct()
addToCart()
changeQuantity()
submitOrder()
```

这种方式能够提供高效、明确的业务操作，但无法自动覆盖应用中尚未开放的功能。

Agent Readable System (ARS) 希望提供另一种基础能力：

**只要应用通过 UI 框架暴露了可访问的控件语义，AI 就应该能够通过操作系统的统一接口发现并操作这些控件。**

开发者不需要为了不同 AI 模型重复编写基础交互接口。

对于标准控件，应尽可能实现零额外适配。

---

## 3. 系统架构

### 3.1 统一术语

| 术语 | 定义与职责 |
|---|---|
| Agent Readable System（ARS） | 项目名称，以及拟议的跨操作系统架构标准整体 |
| ARS Protocol | 定义应用发现、系统能力、UI 语义、动作执行、状态事件与授权交互的协议 |
| ARS Runtime | 实现协议的运行时，负责平台桥接、状态同步、操作调度、权限执行与审计 |
| ARS Agent Interface | 面向 AI Agent 的统一交互入口，通过 ARS Protocol 与 ARS Runtime 交互 |

### 3.2 分层架构

```text
┌──────────────────────────────────────┐
│                User                  │
│      Voice / Text / Gesture          │
└──────────────────┬───────────────────┘
                   │
                   ▼
┌──────────────────────────────────────┐
│              AI Agent                │
│                                      │
│  Intent Understanding                │
│  Task Planning                       │
│  Decision Making                     │
│  Result Verification                 │
└──────────────────┬───────────────────┘
                   │
                   ▼
┌──────────────────────────────────────┐
│         ARS Agent Interface       │
│                                      │
│  Application Discovery               │
│  System Capability API               │
│  UI Semantics API                    │
│  Action Execution API                │
│  State & Event API                   │
│  Permission & Authorization          │
└──────────────────┬───────────────────┘
                   │
                   ▼
┌──────────────────────────────────────┐
│             ARS Runtime              │
│  Platform Adapters / State Sync       │
│  Action Dispatch / Authorization      │
└──────────────────┬───────────────────┘
                   │
                   ▼
┌──────────────────────────────────────┐
│          Operating System            │
│                                      │
│  Process / File / Device Management  │
│  Application Runtime                 │
│  UI Framework                        │
│  Security & Permissions              │
└──────────────────┬───────────────────┘
                   │
           ┌───────┴────────┐
           ▼                ▼
     Applications      System Services
```

系统的核心是 **ARS Agent Interface**。

它是面向不同模型和智能体实现的统一入口。ARS Protocol 定义交互契约，ARS Runtime 通过平台适配器执行请求，并遵守操作系统与应用的安全边界。

ARS Runtime 可以作为现有系统上的适配层实现，也可以与 UI 框架、应用接口或系统服务深度集成。不同部署方式可以并存。

不同 AI 模型可以通过相同的协议使用各平台已开放并授权的能力。跨平台统一的是交互契约，不保证各平台功能完全一致。

---

## 4. 系统能力与应用能力

Agent Readable System (ARS) 将操作能力划分为两个层次。

### 4.1 系统能力（System Capabilities）

操作系统已经管理的功能，应直接通过原生接口提供给 AI。

例如：

- 文件管理
- 应用启动与切换
- 进程管理
- 音量调整
- 网络连接
- 蓝牙设备管理
- 系统通知
- 权限管理

例如用户说：

> 将音量调到 30%。

AI 不应该需要打开设置界面、寻找音量滑块，再模拟拖动。

它应该直接调用经过授权的系统能力。

```text
User Intent
    ↓
AI Agent
    ↓
System Capability API
    ↓
Audio Service
```

### 4.2 应用能力（Application Capabilities）

应用程序内部的业务功能与操作系统基础能力不同。

例如：

- 修改购物车数量
- 发送聊天消息
- 编辑照片
- 创建日程
- 修改游戏设置

操作系统通常不知道这些业务功能的内部实现。

因此，Agent Readable System (ARS) 采用两种互补方式。

**方式 A：原生业务能力接口**

应用可以主动提供高层业务操作。

适合明确、复杂或需要事务保障的功能。

**方式 B：运行时 UI 语义接口**

当应用没有提供业务接口时，AI 可以查询当前界面中已经暴露的控件，并通过标准操作接口与应用交互。

这允许 AI 使用许多未专门适配 AI 的应用功能。

两种方式并存，不要求所有开发者都实现额外的 AI 业务 API。

---

## 5. ARS Protocol

ARS Protocol 是本项目计划设计的统一协议，涵盖系统能力、应用能力与 UI 语义交互。下文展示其中 UI 交互部分的概念设计。

它定义 AI 如何读取和操作应用当前提供的交互界面。

### 5.1 基础控件结构

```json
{
  "id": "element_42",
  "role": "button",
  "label": "+",
  "enabled": true,
  "actions": ["click"],
  "parent": "cart_item_3"
}
```

基础属性包括：

| 属性 | 说明 |
|---|---|
| id | 当前 UI 会话中的控件标识 |
| role | 控件类型 |
| label | 控件名称 |
| value | 当前值 |
| state | 当前状态 |
| actions | 支持的操作 |
| parent | 父节点 |
| children | 子节点 |

控件标识可以由运行时生成，不要求开发者手动注册。

标识的有效范围及生命周期需要由协议明确规定。

### 5.2 基础操作

第一阶段计划支持：

```text
listApplications()
launchApplication(appId)

getCurrentUI()
findElements(query)

executeAction(elementId, action)
setText(elementId, text)

observeChanges()
cancelOperation(operationId)
```

这些接口为概念设计，并非现有 Android API 的完整定义。

### 5.3 按需读取

系统不需要提前扫描所有应用的全部页面。

当 AI 需要操作某个应用时：

1. 启动应用。
2. 查询当前已创建且可访问的 UI。
3. 获取结构化控件信息。
4. 根据用户目标选择操作。
5. 执行操作。
6. 重新查询状态并验证结果。

对于尚未创建的页面控件，AI 需要先导航到相应页面。

### 5.4 UI 状态版本

为了减少错误操作，协议应提供 UI 状态版本或等效的一致性机制。

例如：

```json
{
  "uiVersion": 105,
  "elementId": "element_42",
  "action": "click"
}
```

当目标控件已经失效或界面状态发生冲突时，系统可以拒绝操作并要求 AI 重新查询。

---

## 6. 开发者兼容原则

Agent Readable System (ARS) 的目标不是增加开发者的工作量。

**开发者应当能够继续使用正常的 UI 开发方式。**

例如：

```java
Button button = new Button(context);

button.setText("加入购物车");

button.setOnClickListener(v -> addToCart());
```

对于这种标准控件，UI 框架已经具有控件类型、文本和可执行动作等基础信息。

ARS Runtime 应尽可能自动提供对应的机器可读表示。

### 兼容性原则

| UI 技术 | 预期支持方式 |
|---|---|
| 原生标准控件 | 自动读取基础语义 |
| 声明式 UI | 通过语义树桥接 |
| HTML / WebView | 通过可访问性语义桥接 |
| Flutter 等跨平台框架 | 通过框架语义桥接 |
| 自绘 Canvas | 取决于是否暴露语义节点 |

对于完全没有暴露语义的自绘界面，系统不保证自动识别其内部交互元素。

本项目不会以识别所有自绘按钮作为第一阶段目标。

长期而言，当 AI 成为重要的应用入口时，开发者将有动力改善应用的机器可访问性。

但这种生态变化并非必然，也不能作为兼容性承诺。

---

## 7. Human-in-the-Loop

ARS 将 AI 作为受控的交互主体，同时保留 GUI 与用户直接操作的完整地位。

GUI 仍然承担重要作用：

- 用户查看 AI 的操作结果
- 用户修改 AI 的错误判断
- 用户手动完成复杂操作
- 用户确认敏感操作
- 用户随时接管设备

### 基本原则

**Human Control Has Priority**

当用户与 AI 同时操作同一界面时，系统应当优先保护用户的直接操作。

ARS Runtime 需要支持：

- 操作取消
- 状态重新同步
- 冲突检测
- 人工接管
- 任务恢复

AI 不能假设界面状态始终由自己独占。取消任务应停止尚未执行的后续动作，不能被误认为可以自动撤回已经完成的外部操作。

---

## 8. 安全与权限

AI 获得系统级操作能力，也意味着更高的安全要求。

Agent Readable System (ARS) 不应允许 AI 无限制操作设备。

基本安全设计包括：

**最小权限原则**

AI 只能访问当前任务所需的系统和应用能力。

**用户授权**

发送消息、删除数据、提交订单等操作需要根据风险和授权范围进行控制。

**可信确认**

支付等高风险操作应在可信的系统或服务边界进行确认，不能仅依赖按钮名称判断风险。

**操作审计**

系统应当能够记录关键操作、授权结果及执行状态，同时保护用户隐私。

**不信任应用语义**

应用提供的控件标签和描述可能不准确，甚至具有误导性。

AI 不应将第三方应用的界面文本视为高优先级指令，也不能据此扩大用户授权。

ARS Runtime 应在实际执行边界校验主体与授权范围，支持授权撤销，并与平台安全机制共同工作。读取界面的权限不自动包含执行所有动作的权限。

---

## 9. 技术边界

Agent Readable System (ARS) 不承诺 AI 能够可靠执行任意应用的全部功能。

需要明确以下限制：

1. 操作系统无法自动理解任意应用内部的私有业务逻辑。
2. 未创建的控件无法直接通过当前 UI 查询获得。
3. 自绘界面不一定提供可访问的控件语义。
4. 控件可点击不代表系统知道其业务后果。
5. UI 操作成功不代表业务操作一定成功。
6. 不同操作系统具有不同的安全和应用隔离机制。
7. 统一协议不能让平台未开放的能力自动变得可用。
8. 状态版本机制有助于检测冲突，但本身不保证业务操作具有原子性。

因此，项目目标是：

**为 AI 提供标准化、低开销、可授权的原生交互能力，并持续提高兼容性和可靠性。**

而不是保证 AI 可以无条件操作所有软件。

---

## 10. 开发路线

以下阶段为拟议研究路线，尚不表示已完成，也不承诺具体发布日期。

### Phase 1 — Proof of Concept

基于现有 Android Accessibility API 构建原型。

目标：

- 获取当前应用 UI 树
- 转换为统一数据结构
- 执行标准控件操作
- 验证操作结果
- 测试陌生应用

这一阶段用于验证协议设计，不代表最终系统依赖第三方辅助功能架构。

### Phase 2 — ARS Protocol

设计与操作系统无关的标准协议。

重点包括：

- 应用发现
- UI 语义
- 控件身份
- 操作执行
- 状态同步
- 权限与安全
- 人机操作权管理

### Phase 3 — AOSP Integration

基于 Android Open Source Project 开发系统级原型。

研究将 ARS Runtime 和 ARS Agent Interface 集成到 Android Framework 和系统服务中的可行性。

这一阶段用于验证深度集成的价值，不要求所有 ARS 实现都采用定制操作系统。

重点验证系统级集成相对于现有 Accessibility 方案的实际收益。

### Phase 4 — Cross-Platform Research

探索在不同平台实现统一协议的可能性：

- Android
- Linux Desktop
- Windows
- 其他支持相应接口的系统

跨平台协议不意味着可以绕过各平台的安全和权限限制。

---

## 11. 项目与传统 AI 自动化的区别

| 传统 AI 自动化 | Agent Readable System (ARS) |
|---|---|
| AI 作为第三方应用运行 | AI 作为可识别、可授权的操作主体 |
| 利用已有自动化接口 | 设计原生 ARS Agent Interface |
| 可能依赖截图与坐标 | 优先使用结构化状态和动作 |
| 侧重完成自动化任务 | 侧重跨操作系统的交互架构标准 |
| 受现有接口能力限制 | 可以通过系统级设计扩展能力 |
| 人工接管由工具自行实现 | 将人机协作纳入系统设计 |

上表描述 ARS 的设计重点；现有自动化方案也可能具备结构化状态、安全控制和人工接管能力。

本项目并不认为现有 Accessibility、自动化工具或应用业务接口没有价值。

相反，它们可以作为构建下一代 ARS Agent Interface 的技术基础。

---

## 12. 长期愿景

未来，用户可能不再需要学习每个应用的操作方式。

用户只需要表达目标：

> 帮我整理今天收到的文件。

> 将购物车里的咖啡数量改成两件。

> 把这张照片调整得更明亮。

> 查找明天下午有空的会议时间。

AI 负责理解意图，并通过操作系统提供的标准接口调用系统或应用能力。

GUI 仍然存在，但不再是所有操作的必经路径。

最终，我们希望实现：

```text
Human Intent
      ↓
   AI Agent
      ↓
ARS Agent Interface
      ↓
Operating System
      ↓
Applications & Devices
```

### 核心主张

**操作系统不应该只为人类提供交互接口，也应该为 AI 提供原生、统一、受控的交互接口。**

GUI 让人类更容易使用计算机。

ARS Agent Interface 应当让 AI 更自然地使用计算机。

---

## 13. 项目状态

**Status: Concept / Early Research**

目前项目处于架构构想与技术研究阶段。

尚未完成正式发布的 ARS Protocol、可供生产使用的 ARS Runtime 或稳定的 ARS Agent Interface。

本文描述的是设计目标和拟议架构，不代表所有能力已经实现。

欢迎对操作系统架构、UI 语义、应用兼容性、安全模型和 AI Agent 交互协议提出讨论与建议。

---

## License

许可证待定。

---

**Agent Readable System (ARS)**

*Make systems readable. Make actions verifiable. Keep humans in control.*
