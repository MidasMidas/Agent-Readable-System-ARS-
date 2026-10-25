# Agent Readable System (ARS)

## 架构设计方案 v0.2（讨论草案）

> **定位**：ARS 是一套由操作系统、构建工具与应用框架共同支持的原生 Agent 交互标准。开发者保持现有 API 与 UI 开发方式；构建阶段自动提取静态交互元数据，运行阶段自动注册动态交互实例，操作系统向 Agent 提供统一、安全的发现、查询、调用与状态接口。
>
> **状态**：Proposal / Draft，尚未实现或验证跨平台兼容性。

## 1. 背景与目标

人类通过 GUI/CLI 与操作系统及应用交互；AI Agent 也需要一个正式、结构化、无需截图或模拟坐标的交互入口。现有应用通常已经通过 Web API、按钮、输入框、菜单等对象定义了对外交互行为。ARS 的目标是利用这些已有入口，避免应用开发者为 AI 再注册一套业务工具。

### 1.1 设计原则

1. **开发者近乎无感知**：正常开发应用，不逐项注册 AI Tool；接入主要由框架、构建工具和系统完成。
2. **不暴露全部内部方法**：仅纳入应用已有的外部交互入口（API、可交互 UI 控件等），不把任意 `public` 方法视为外部能力。
3. **机器可读但不替 AI 理解业务**：提供包/类/方法/控件名称、参数、类型、状态与可选注释；AI 根据上下文选择入口。
4. **静态与动态分离**：构建期描述“可能存在的能力”，运行期描述“当前存在的实例”，实时状态描述“现在能否操作”。
5. **统一协议，不强制统一内部实现**：Spring、Qt、GTK、Android 等可采用不同的提取和执行方式。
6. **系统是唯一受信任的 ARS 发现与调用入口**：Agent 不得直接读取应用 Manifest 作为权威能力源，也不得绕过系统服务调用 ARS。
7. **Agent 注册准入与凭证校验**：Agent 须经可信注册中心审核并获取可验证凭证；系统定期同步批准、撤销和信任信息。
8. **身份、设备授权、应用授权分离**：获准接入 ARS 不等于有权操作任意应用；每次发现与调用分别授权。
9. **标准独立于具体 AI 模型与操作系统**：Linux 参考实现只是第一步。

### 1.2 非目标

- 不要求 Linux 内核理解业务方法语义。
- 不要求发现所有私有函数或跨进程直接执行任意内存地址。
- 不把截图、OCR 或鼠标坐标模拟作为核心协议。
- 不保证未适配的旧应用自动获得完整 ARS 支持。
- 不将 JSON 文件视为动态对象必须采用的存储形式。

## 2. 总体架构

```text
 Agent Developer / Operator                    Application Developer
       │ 注册/审核/发证                              │ 原有开发方式
       ▼                                           ▼
┌────────────────────────┐              ┌──────────────────────────┐
│ ARS Agent Registration │              │ ARS Build Integration    │
│ Center (可信注册中心)  │              │ 静态 Manifest / 桥接代码 │
│ 身份审核 / 发证 / 撤销  │              └────────────┬─────────────┘
└───────────┬────────────┘                           │ 安装时可信导入
            │ 签名信任数据/批准名单/撤销信息         ▼
            │ 周期同步及必要时在线验证    ┌──────────────────────────┐
            └───────────────────────────►│ Local ARS System Service │
                                        │ 受保护能力索引/注册表    │
 Agent ──凭证+持有证明──►[身份验证]────►│ 发现授权/调用授权/审计   │
          查询/调用                       └────────────┬─────────────┘
                                                    │ 可信 IPC
                                      ┌─────────────┴────────────┐
                                      ▼                          ▼
                                 API Adapter                 UI Adapter
                                 (Spring)                    (Qt 等)
                                      │                          │
                                 既有 API 逻辑               既有 UI 逻辑

注：注册中心可为官方、企业或设备受信任的本地机构；不强制全球单点服务。
```

### 2.1 组件职责

| 组件 | 职责 |
| --- | --- |
| **ARS Build Integration** | 从现有路由、UI 元数据生成静态 Manifest；必要时生成桥接代码。 |
| **ARS Framework Adapter** | 在应用进程中注册动态实例、提供实时状态并执行原有交互路径。 |
| **ARS Agent Registration Center** | 验证 Agent 身份及发行方、审批准入、签发或认可凭证、发布批准和撤销信息。 |
| **ARS System Service** | 可信导入静态清单，维护受保护索引与运行时注册表；验证 Agent 身份、权限、凭证和撤销状态；路由并审计调用。 |
| **ARS Client / Agent** | 持有效凭证并证明其持有权，通过系统 API 查询和调用获准端点。 |
| **Existing Application** | 继续使用原有业务逻辑、UI 及业务鉴权，不额外编写 AI Tool。 |

### 2.2 两条独立信任链

- **Provider（被操作应用）信任链**：安装包来源/完整性 → 受保护 Manifest 导入 → 运行进程身份和 IPC 绑定 → 动态实例注册。不能仅凭进程自报 `appId` 接受注册。
- **Agent（调用者）信任链**：注册中心审批 → 凭证签发 → 本地系统验证发行者/签名/持有证明/有效期/撤销状态 → 设备用户授权 → 目标应用业务鉴权。

**系统注册表是权威查询源。** 安装包中的 JSON 仅为构建产物，不能作为授权凭据。即使攻击者读取或复制 Manifest，也不能因此获得 ARS 调用权；但元数据保密仍需安装包和文件访问控制。受保护索引不得被普通 Agent 直接读取。

## 3. 三层数据模型

### 3.1 静态能力清单（Capability Manifest）

构建或打包时自动生成，随应用版本发布。用于描述应用**可能提供**的外部交互端点和 UI 模板。可以是安装包内一个或多个 JSON 文件，或者等价的结构化编码。

示例：

```json
{
  "arsVersion": "0.1",
  "appId": "com.example.shop",
  "appVersion": "1.0.0",
  "endpoints": [
    {
      "endpointId": "com.example.shop.cart.addProduct",
      "kind": "api.http",
      "source": {
        "package": "com.example.shop.cart",
        "class": "CartController",
        "method": "addProduct"
      },
      "transport": {
        "method": "POST",
        "path": "/cart/add"
      },
      "parameters": [
        { "name": "productId", "type": "string", "required": true }
      ],
      "returnType": "boolean"
    }
  ],
  "uiTemplates": [
    {
      "templateId": "product.detail",
      "controls": [
        { "localId": "quantity", "kind": "ui.input.number", "operations": ["read", "write"] },
        { "localId": "addToCart", "kind": "ui.button", "operations": ["invoke"] }
      ]
    }
  ]
}
```

**限制**：静态清单不保证端点在任意时刻可调用；编译期不能可靠识别全部动态生成的控件。注释、参数名可能因语言、编译选项和构建方式而缺失，必须标注缺失而不是猜测。

### 3.2 运行时交互注册表（Interaction Registry）

应用框架在窗口、页面、控件或 API 实例创建/销毁时，通过运行时桥接层维护当前有效的实例。注册表默认位于内存，由 ARS System Service 在完成 Provider 身份验证后管理或索引；普通 Agent 只能通过受授权的系统查询接口读取。

```json
{
  "providerId": "com.example.shop:pid-4210",
  "instanceId": "page-1042",
  "templateId": "product.detail",
  "generation": 7,
  "context": { "productId": "coffee-123" },
  "controls": [
    { "instanceId": "page-1042.quantity", "localId": "quantity" },
    { "instanceId": "page-1042.addToCart", "localId": "addToCart" }
  ]
}
```

- `instanceId` 用于区分同一模板的多个实例；不能把可复用 UI 对象的 ID 当成永久业务 ID。
- `generation` / `stateVersion` 用于检测旧引用与并发修改。
- 应用异常退出时，系统按进程连接生命周期自动清理注册表；不能仅依赖应用发送删除事件。
- 运行时生成、没有静态模板的对象，可注册完整的动态端点描述。

### 3.3 实时状态（Live State）

状态按需读取或订阅，不要求每次输入字符都写磁盘文件。

```json
{
  "instanceId": "page-1042.quantity",
  "stateVersion": 12,
  "value": 2,
  "enabled": true,
  "visible": true,
  "operations": ["read", "write"]
}
```

运行时事件示例：

```json
{ "event": "REGISTER", "instanceId": "page-1042", "templateId": "product.detail" }
```

```json
{ "event": "UPDATE", "instanceId": "page-1042.quantity", "stateVersion": 12, "changes": { "value": 2 } }
```

```json
{ "event": "UNREGISTER", "instanceId": "page-1042" }
```

动态数据通过 IPC 消息传递，**可以序列化为 JSON，但不必创建磁盘 JSON 文件**。必要时可保存诊断快照，不作为权威状态源。

## 4. 端点类型与统一操作

| 端点类型 | 自动发现来源 | 典型操作 | 调用注意事项 |
| --- | --- | --- | --- |
| `api.http` | Web 框架路由表 / 构建元数据 | `invoke` | 保留原有认证、参数转换、校验和拦截器。 |
| `ui.button` | UI 框架控件树 / 运行时对象 | `invoke` | 在正确 UI 线程触发原有处理流程。 |
| `ui.input.*` | UI 框架控件树 | `read`、`write` | 处理校验、焦点、变更事件及状态版本。 |
| `ui.checkbox` | UI 框架控件树 | `read`、`write` | 保留原有事件语义。 |
| `ui.menu` | UI 框架控件树 | `list`、`invoke` | 只暴露当前允许访问的菜单项。 |

ARS 统一**发现、描述、寻址、授权和调用信封**，但不强迫所有端点拥有完全相同的业务行为。

## 5. 最小协议草案

以下操作必须通过已认证的 ARS 会话调用；认证上下文由系统 IPC/会话建立，不信任请求 JSON 中自报的 Agent ID。

```text
discoverApps()
listEndpoints(appId, filters)
describeEndpoint(endpointId)
listInstances(providerId, filters)
readState(instanceId)
invoke(targetId, operation, arguments, expectedStateVersion?)
subscribe(targetId, eventTypes)
```

调用示例：

```json
{
  "requestId": "req-0001",
  "targetId": "page-1042.quantity",
  "operation": "write",
  "arguments": { "value": 2 },
  "expectedStateVersion": 11
}
```

响应示例：

```json
{
  "requestId": "req-0001",
  "status": "success",
  "result": { "value": 2, "stateVersion": 12 }
}
```

建议的错误类别：`NOT_FOUND`、`STALE_INSTANCE`、`STATE_CONFLICT`、`INVALID_ARGUMENT`、`PERMISSION_DENIED`、`TIMEOUT`、`PROVIDER_UNAVAILABLE`、`EXECUTION_FAILED`。异步操作应返回可跟踪的操作 ID，不能把“请求已接收”当作“业务已完成”。

## 6. Agent 注册中心、凭证与安全边界

### 6.1 Agent 准入流程

1. Agent 开发者/运营方向受信任的 **ARS Agent Registration Center** 申请注册，提交 Agent 标识、发行方信息、签名公钥/证书及必要的安全声明。
2. 注册中心执行身份审核和准入审批，生成唯一注册标识与批准记录；批准记录可限定用途、发行者、版本或部署范围。
3. 注册中心签发或认可可验证的 **Agent Identity Credential**，或将 Agent 公钥/身份写入签名批准列表。
4. Agent 安装或启动时，本地 ARS System Service 验证发行者信任链、凭证有效期、Agent 身份与当前进程/安装包的绑定关系，并通过挑战响应等机制验证密钥持有权。
5. 通过准入验证后，Agent 仍须取得**当前设备/用户对具体应用与操作的授权**；Agent 注册获批不授予全局调用权。
6. Agent 使用受认证的系统会话进行 `discoverApps`、`listEndpoints`、`readState`、`invoke` 等操作。每次查询与调用分别进行授权判断。

### 6.2 凭证模型（示意，不代表最终线协议）

**身份凭证的逻辑字段**：

```json
{
  "credentialType": "AgentIdentity",
  "agentId": "com.example.shopping-agent",
  "issuer": "ars-trust.example",
  "publicKeyId": "key-001",
  "audience": "ars-system",
  "notBefore": "2026-10-25T00:00:00Z",
  "expiresAt": "2026-11-25T00:00:00Z"
}
```

实际凭证必须有经过验证的数字签名或等价可信机制，且 Agent 应证明持有对应私钥；上例不是可直接使用的安全 Token。私钥应优先使用系统密钥库、TPM、硬件保护或平台提供的应用密钥隔离机制。

**本地授权记录**（由本机用户/管理员授权，不应由 Agent 自行声明）：

```json
{
  "agentId": "com.example.shopping-agent",
  "deviceScope": "current-user-session",
  "targetApp": "com.example.shop",
  "permissions": ["cart.read", "cart.addProduct"],
  "expiresAt": "2026-10-26T00:00:00Z"
}
```

**三层决策**：注册中心决定是否准入；操作系统和用户决定本设备上允许发现/调用的范围；目标 App 仍执行登录、会话、业务鉴权与敏感操作校验。

### 6.3 本地同步、验证与撤销

- ARS System Service **定期从可信注册中心同步**经过签名验证的批准列表、撤销列表/状态、信任锚与策略版本；周期由平台安全策略决定，不在核心协议中写死。
- 本地可使用签名凭证进行离线密码学验证，但**不能仅因 Agent 出现在缓存批准列表就放行**：还需验证签名、发行者、有效期、目标受众、密钥持有证明、进程身份绑定和授权范围。
- 支持在线即时查询、主动撤销推送、短期凭证和本地撤销缓存，缩短两次同步之间的风险窗口。
- 同步结果带签名、版本/序列号、有效时间和防回滚保护，避免攻击者重放旧批准列表。
- 定义缓存过期与注册中心不可达时的 **fail-closed / 受限离线模式**：高风险操作默认拒绝；低风险操作是否继续由系统策略明确决定，不允许无限期信任过期名单。
- 每次调用再次检查撤销和本地授权；对于长时间运行的任务，应在关键阶段重新验证授权或允许撤销中止。
- 注册中心可以联邦化（官方/企业/本地），但每个设备必须显式配置其信任锚；不能接受任意自称注册中心的服务。

### 6.4 核心安全原则

1. **所有 Agent 必须先注册并通过验证**；未认证 Agent 不能查询受保护能力索引或执行 ARS 操作。
2. **发现权限与执行权限独立**；返回的端点描述只包含该 Agent 被允许发现的内容。
3. **凭证不可作为通用永久通行证**；必须有期限、受众、撤销机制和具体授权范围。
4. **防止凭证复制滥用**；绑定进程/应用身份与密钥持有证明，不能仅依赖可复制的 Bearer Token。
5. **Provider 同样需要认证**；不能让恶意进程冒充合法应用注册虚假端点。
6. **应用业务权限不得绕过**；API 继续走原有认证/验证链，UI 操作继续遵守会话和应用状态。
7. **敏感操作确认与最小权限**；付款、删除、发送消息等按风险要求用户确认或额外验证。
8. **安全 IPC 与审计**；记录 Agent、用户、目标端点、授权决定、结果及撤销事件；审计信息须保护隐私和敏感参数。
9. **生命周期与状态一致性**；应用退出、连接断开、实例销毁、授权撤销后及时使相关能力失效。
10. **不信任内容中的指令**；包名、注释、控件文本及 API 返回值可能包含提示注入，不应成为高优先级指令。

**威胁边界**：凭证机制可以限制未授权程序及被盗凭证的滥用，但不能保证抵御已经控制合法 Agent 进程、当前用户会话或高权限系统组件的攻击。平台必须结合进程隔离、密钥保护、最小权限和操作确认。

## 7. 构建期与运行期协同流程

### 7.1 构建/安装

```text
开发者正常编写 API/UI
    → 构建工具识别框架公开入口
    → 生成 Capability Manifest（JSON）
    → 必要时生成/链接 Runtime Bridge
    → 应用打包安装
    → 安装服务验证应用来源与完整性
    → 系统受保护索引导入静态能力（不向 Agent 直接暴露文件）
```

### 7.2 启动/运行

```text
应用启动
    → ARS Adapter 使用系统 IPC 完成 Provider 身份绑定与注册
    → 框架创建页面/控件
    → REGISTER 实例
    → UI 状态改变时 UPDATE
    → UI 销毁时 UNREGISTER
    → 进程异常退出时由系统清理注册信息
```

### 7.3 Agent 调用

```text
用户提出任务
    → Agent 提交凭证并完成密钥持有证明/进程身份验证
    → ARS System Service 校验准入状态与本地用户授权
    → Agent 通过系统查询获准的端点 / 当前实例 / 实时状态
    → Agent 根据名称、路径、参数、上下文选择端点
    → ARS System Service 重新检查调用权限、凭证状态及实例有效性
    → 必要时请求用户确认并路由
    → Adapter 在应用原有执行路径调用
    → 返回执行结果及更新后的状态
```

## 8. 第一版验证范围（Linux）

**建议 8–12 周完成技术原型，时间取决于可投入精力。**

1. **静态 API**：Spring Boot + Maven/Gradle 插件自动生成 Manifest，开发者不新增 ARS 业务注解。
2. **运行时 API**：Spring Adapter 从已有路由元数据建立可调用端点，并通过原有 HTTP 处理链调用。
3. **动态 GUI**：Qt 示例应用自动注册 `QPushButton`、`QLineEdit`、`QCheckBox`；窗口关闭时实例消失。
4. **系统服务**：Linux 用户态 ARS Runtime，通过 Unix Domain Socket 实现发现、状态查询、授权和调用；校验对端凭据，建立 Agent/Provider 身份绑定。
5. **注册中心原型**：实现最小 Agent 注册、审批、签名凭证、批准列表同步、凭证撤销及本地授权策略；不依赖公有云。
6. **客户端**：先使用 CLI，再接入 AI Agent；核心协议不依赖模型工具调用格式。
7. **一致性测试**：新增 API 自动发现、动态页面生成/销毁、崩溃清理、权限拒绝、并发状态冲突、无凭证拒绝、伪造凭证拒绝、过期/撤销凭证拒绝、跨应用越权拒绝、离线同步过期策略。

**原型成功标准**：业务开发者只编写普通 API 和 UI；在启用 ARS 的构建工具与框架环境中，构建和运行后无需逐项注册 AI Tool，即可由**经过注册审批且持有效凭证并获本地授权**的独立 Client 发现、描述和调用已有交互入口，并得到真实结果。

## 9. 建议的规范与仓库结构

```text
agent-readable-system/
├── README.md
├── ARCHITECTURE.md
├── ROADMAP.md
├── specs/
│   ├── ARS-001-manifest.md
│   ├── ARS-002-runtime-registry.md
│   ├── ARS-003-state-lifecycle.md
│   ├── ARS-004-invocation.md
│   ├── ARS-005-agent-identity-registration.md
│   ├── ARS-006-authorization-credential.md
│   └── ARS-007-trust-sync-revocation.md
├── schemas/
│   ├── manifest.schema.json
│   ├── endpoint.schema.json
│   └── invocation.schema.json
├── registration-center/
├── runtime/
├── adapters/
│   ├── spring/
│   └── qt/
├── examples/
└── tests/conformance/
```

## 10. 尚待研究的问题

- 静态 UI 模板能否从不同 GUI 框架的构建流程稳定提取？哪些只能运行时发现？
- 对没有语义名称、参数名或源码注释的端点，如何标准化缺失信息？
- Web API 的用户身份、Cookie、Token 与 Agent 身份如何正确关联？
- UI 实例是否应仅在当前用户会话中可见？多窗口、多用户、多设备如何隔离？
- 状态变化的事件粒度、订阅背压和快照一致性如何定义？
- 怎样允许开发者用现有权限策略排除敏感入口，同时保持接入无感知？
- 不同框架适配器的符合性测试如何保持一致？
- 注册中心的审批主体和信任根如何建立？如何支持企业私有中心与跨中心互信？
- 不同平台怎样将 Agent 凭证绑定到不可伪造的安装包/进程身份？
- 凭证与授权的有效期、同步频率、撤销延迟及离线降级策略如何量化？
- 同一用户下多个 Agent 如何隔离密钥、权限、日志和会话？

---

**一句话总结**：**构建期自动生成静态能力清单；运行时由框架自动注册动态实例与状态；系统维护受保护的能力索引；Agent 先经注册中心审批并持凭证通过身份验证，再按设备/用户/应用授权进行受控发现与调用。**
