# 日文文本翻译工具 - Mermaid 图形示例

> 本文档展示如何在 GitHub Markdown 中使用 Mermaid 图形。所有代码都可以直接在应用的 `/mermaid` 页面复制。

## 📐 系统架构

### 类图 - 系统类结构

```mermaid
classDiagram
    class Home {
        -string chineseText
        -string japaneseText
        -string chineseResult
        -string japaneseResult
        -boolean isTranslatingCN
        -boolean isTranslatingJP
        +handleTranslateCNToJP()
        +handleTranslateJPToCN()
        +handleCopyResult()
        +handleClearChinese()
        +handleClearJapanese()
    }

    class TranslationAPI {
        -AxiosInstance apiClient
        -string APP_ID
        +translateText(text, from, to)
    }

    class UIComponents {
        <<shadcn/ui>>
        +Card
        +Button
        +Textarea
        +Toast
        +Icons
    }

    class TranslationRequest {
        <<interface>>
        +string q
        +string from
        +string to
    }

    class TranslationResponse {
        <<interface>>
        +object data
        +number status
        +string msg
        +array trans_result
    }

    class Axios {
        +create(config)
        +post(url, data)
        +interceptors
    }

    Home --> TranslationAPI : 使用
    Home *-- UIComponents : 组合
    TranslationAPI ..> TranslationRequest : 使用
    TranslationAPI ..> TranslationResponse : 返回
    TranslationAPI --> Axios : 依赖
```

## 🔄 翻译流程

### 序列图 - 完整交互流程

```mermaid
sequenceDiagram
    actor 用户
    participant Home as Home组件
    participant API as TranslationAPI
    participant Axios as Axios客户端
    participant Server as 翻译API服务

    用户->>Home: 1. 输入文本
    用户->>Home: 2. 点击翻译按钮
    activate Home
    Home->>Home: 3. 验证输入
    Home->>API: 4. translateText(text, from, to)
    activate API
    API->>API: 5. 创建请求对象
    API->>Axios: 6. post(url, data)
    activate Axios
    Axios->>Axios: 7. 添加 X-App-Id 头
    Axios->>Server: 8. HTTP POST 请求
    activate Server
    Server->>Server: 9. 执行翻译
    Server-->>Axios: 10. 翻译结果
    deactivate Server
    Axios->>Axios: 11. 响应拦截器处理
    Axios-->>API: 12. 返回响应数据
    deactivate Axios
    API->>API: 13. 提取翻译文本
    API-->>Home: 14. 翻译文本
    deactivate API
    Home->>Home: 15. 更新结果状态
    Home-->>用户: 16. 显示翻译结果
    Home-->>用户: 17. 显示成功提示
    deactivate Home
```

## 🏗️ 系统分层

### 组件图 - 三层架构

```mermaid
graph TB
    subgraph 表示层["表示层 (Presentation Layer)"]
        Home["Home<br/>主页面组件<br/>• 翻译界面<br/>• 状态管理"]
        UI["UI Components<br/>shadcn/ui 组件<br/>• Card, Button<br/>• Textarea, Toast"]
        Icons["Icons<br/>Lucide React<br/>• Languages<br/>• Arrows, Copy"]
        Hooks["Hooks<br/>React Hooks<br/>• useState<br/>• useToast"]
    end

    subgraph 服务层["服务层 (Service Layer)"]
        API["TranslationAPI<br/>翻译服务封装<br/>translateText()"]
        Types["Type Definitions<br/>TypeScript 类型<br/>Request/Response"]
    end

    subgraph 网络层["网络层 (Network Layer)"]
        AxiosClient["Axios Client<br/>HTTP 客户端<br/>请求/响应拦截器"]
        ExtAPI["Translation API<br/>外部翻译服务<br/>200+ 语言支持"]
    end

    Home --> API
    Home --> UI
    Home --> Icons
    Home --> Hooks
    API --> Types
    API --> AxiosClient
    AxiosClient --> ExtAPI

    style 表示层 fill:#e3f2fd,stroke:#2196F3,stroke-width:3px
    style 服务层 fill:#e3f2fd,stroke:#2196F3,stroke-width:3px
    style 网络层 fill:#e3f2fd,stroke:#2196F3,stroke-width:3px
    style Home fill:#bbdefb,stroke:#2196F3,stroke-width:2px
    style API fill:#bbdefb,stroke:#2196F3,stroke-width:2px
    style AxiosClient fill:#bbdefb,stroke:#2196F3,stroke-width:2px
```

## 🔀 状态管理

### 状态图 - 应用状态转换

```mermaid
stateDiagram-v2
    [*] --> 空闲状态
    
    空闲状态 --> 输入中文: 用户输入中文
    空闲状态 --> 输入日文: 用户输入日文
    
    输入中文 --> 翻译中_CN: 点击翻译按钮
    输入日文 --> 翻译中_JP: 点击翻译按钮
    
    翻译中_CN --> 验证输入_CN: 开始验证
    翻译中_JP --> 验证输入_JP: 开始验证
    
    验证输入_CN --> 调用API_CN: 验证通过
    验证输入_CN --> 输入中文: 验证失败
    
    验证输入_JP --> 调用API_JP: 验证通过
    验证输入_JP --> 输入日文: 验证失败
    
    调用API_CN --> 等待响应_CN: 发送请求
    调用API_JP --> 等待响应_JP: 发送请求
    
    等待响应_CN --> 显示结果_CN: 成功
    等待响应_CN --> 显示错误_CN: 失败
    
    等待响应_JP --> 显示结果_JP: 成功
    等待响应_JP --> 显示错误_JP: 失败
    
    显示结果_CN --> 空闲状态: 完成
    显示结果_JP --> 空闲状态: 完成
    显示错误_CN --> 输入中文: 重试
    显示错误_JP --> 输入日文: 重试
    
    空闲状态 --> [*]: 退出
```

## 📊 业务流程

### 流程图 - 完整翻译流程

```mermaid
flowchart TD
    Start([开始]) --> Input{用户输入?}
    
    Input -->|中文| InputCN[输入中文文本]
    Input -->|日文| InputJP[输入日文文本]
    
    InputCN --> ClickCN[点击翻译为日文]
    InputJP --> ClickJP[点击翻译为中文]
    
    ClickCN --> ValidateCN{验证输入}
    ClickJP --> ValidateJP{验证输入}
    
    ValidateCN -->|为空| ErrorCN[显示错误提示]
    ValidateCN -->|有效| LoadingCN[显示加载状态]
    
    ValidateJP -->|为空| ErrorJP[显示错误提示]
    ValidateJP -->|有效| LoadingJP[显示加载状态]
    
    ErrorCN --> InputCN
    ErrorJP --> InputJP
    
    LoadingCN --> CallAPICN[调用 translateText<br/>from: zh, to: jp]
    LoadingJP --> CallAPIJP[调用 translateText<br/>from: jp, to: zh]
    
    CallAPICN --> RequestCN[创建请求对象]
    CallAPIJP --> RequestJP[创建请求对象]
    
    RequestCN --> AxiosCN[Axios POST 请求]
    RequestJP --> AxiosJP[Axios POST 请求]
    
    AxiosCN --> ServerCN[翻译 API 服务]
    AxiosJP --> ServerJP[翻译 API 服务]
    
    ServerCN --> ResponseCN{请求成功?}
    ServerJP --> ResponseJP{请求成功?}
    
    ResponseCN -->|成功| ShowResultCN[显示日文结果]
    ResponseCN -->|失败| ShowErrorCN[显示错误信息]
    
    ResponseJP -->|成功| ShowResultJP[显示中文结果]
    ResponseJP -->|失败| ShowErrorJP[显示错误信息]
    
    ShowResultCN --> ToastCN[显示成功提示]
    ShowResultJP --> ToastJP[显示成功提示]
    
    ShowErrorCN --> InputCN
    ShowErrorJP --> InputJP
    
    ToastCN --> Actions{用户操作?}
    ToastJP --> Actions
    
    Actions -->|复制| Copy[复制到剪贴板]
    Actions -->|清空| Clear[清空输入和结果]
    Actions -->|继续| Input
    
    Copy --> Actions
    Clear --> Input
    
    Actions -->|退出| End([结束])
    
    style Start fill:#4caf50,stroke:#2e7d32,stroke-width:3px,color:#fff
    style End fill:#f44336,stroke:#c62828,stroke-width:3px,color:#fff
    style Input fill:#2196F3,stroke:#1565c0,stroke-width:2px,color:#fff
    style ValidateCN fill:#ff9800,stroke:#e65100,stroke-width:2px,color:#fff
    style ValidateJP fill:#ff9800,stroke:#e65100,stroke-width:2px,color:#fff
    style ResponseCN fill:#ff9800,stroke:#e65100,stroke-width:2px,color:#fff
    style ResponseJP fill:#ff9800,stroke:#e65100,stroke-width:2px,color:#fff
    style Actions fill:#2196F3,stroke:#1565c0,stroke-width:2px,color:#fff
```

## 💾 数据模型

### ER图 - 数据实体关系（可选功能）

```mermaid
erDiagram
    USER ||--o{ TRANSLATION_HISTORY : creates
    USER {
        string user_id PK
        string session_id
        timestamp created_at
    }
    
    TRANSLATION_HISTORY {
        string id PK
        string user_id FK
        string source_text
        string target_text
        string source_lang
        string target_lang
        timestamp created_at
    }
    
    TRANSLATION_HISTORY ||--|| API_REQUEST : triggers
    API_REQUEST {
        string request_id PK
        string history_id FK
        string endpoint
        json request_body
        json response_body
        number status_code
        timestamp timestamp
    }
    
    API_REQUEST ||--|| API_RESPONSE : returns
    API_RESPONSE {
        string response_id PK
        string request_id FK
        json data
        number status
        string message
        array trans_result
    }
```

## 📝 使用说明

### 如何使用这些图形

1. **复制代码**
   - 访问应用的 `/mermaid` 页面
   - 选择需要的图形类型
   - 点击 "复制代码" 按钮

2. **粘贴到 Markdown**
   - 在 GitHub 仓库中创建或编辑 `.md` 文件
   - 粘贴复制的代码
   - 保存文件

3. **查看渲染效果**
   - GitHub 会自动渲染 Mermaid 图形
   - 图形会以精美的可视化形式展示

### 支持的文件类型

- ✅ README.md
- ✅ CONTRIBUTING.md
- ✅ docs/*.md
- ✅ Issue 描述
- ✅ Pull Request 描述
- ✅ Wiki 页面
- ✅ Discussions
- ✅ Gist

## 🔗 相关资源

- [应用主页](/) - 日文翻译工具
- [SVG 图形](/uml) - 交互式 SVG UML 图形
- [Mermaid 代码](/mermaid) - 可复制的 Mermaid 代码
- [Mermaid 官方文档](https://mermaid.js.org/)
- [GitHub Mermaid 支持](https://github.blog/2022-02-14-include-diagrams-markdown-files-mermaid/)

## 💡 提示

- 所有图形代码都可以在应用中直接复制
- 支持在 GitHub、GitLab、Notion 等平台使用
- 可以在 [Mermaid Live Editor](https://mermaid.live) 中预览和编辑
- 图形会自动适应主题（亮色/暗色模式）

---

**注意：** 本文档中的所有 Mermaid 图形在 GitHub 上会自动渲染。如果您在其他平台查看，可能需要相应的 Mermaid 支持。
