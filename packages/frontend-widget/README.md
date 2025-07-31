# 语音转写与翻译前端小部件 (@realtime-translation/frontend-widget)

这是一个基于科大讯飞实时语音转写与翻译 Web 端 SDK 官方 Demo 改造的前端小部件。它是 Monorepo `Li-Zhuo-Ran/realtime-translation-monorepo` 中的一个子项目，旨在提供一个开箱即用的浏览器端解决方案，用于实现实时麦克风语音转写和多语言翻译功能。

## 🚀 功能特性

*   **实时语音转写**: 采集麦克风音频，实时将语音内容转写为文本 (源语言：英文)。
*   **实时语音翻译**: 在转写的同时，将源语言内容实时翻译为目标语言 (目标语言：中文)。
*   **浏览器兼容**: 适用于现代浏览器。
*   **模块化设计**: 将核心功能封装为独立的前端组件，便于集成。
*   **鉴权与通信**: 负责与后端签名服务 (`signa-server.js`) 进行鉴权，并建立与科大讯飞实时转写服务的 WebSocket 连接。
*   **UI 浮窗展示**: 在前端界面中以简洁的浮窗同时显示转写和翻译结果。

## 📦 项目结构
```
realtime-translation-monorepo/
├── packages/
│   └── frontend-widget/
│       ├── dist/                         # 编译产物目录
│       │   ├── index.cjs.js              # CommonJS 模块格式
│       │   ├── index.esm.js              # ES Module 模块格式
│       │   ├── index.umd.js              # UMD (Universal Module Definition) 模块格式，适用于浏览器直接引入
│       │   ├── index.d.ts                # TypeScript 类型声明文件
│       │   ├── processor.worker.js       # Web Worker 文件，用于音频处理
│       │   └── processor.worklet.js      # AudioWorklet 文件，用于更底层的音频处理
│       ├── app.js                        # 核心客户端逻辑，负责UI交互、WebSocket通信和SDK调用
│       ├── index.html                    # 示例页面，演示如何集成和使用小部件
│       ├── package.json                  # 小部件的依赖管理和构建脚本
│       └── README.md                     # 本说明文件
├── servers/
│   └── signa-server/
│       └── signa-server.js               # 后端签名服务
└── package.json                          # Monorepo 根 package.json
└── package-lock.json                     # Monorepo 根 lock 文件
```

## 🛠️ 如何使用

### 前置条件

在运行此前端小部件之前，请确保以下条件已满足：

1.  **Monorepo 依赖安装**:
    * 请在 Monorepo 根目录 (`realtime-translation-monorepo/`) 下运行 `npm install` (或 `pnpm install`)，以安装所有子项目的依赖。此步骤也会安装开发服务器 `http-server` (如果其被声明为开发依赖)，用于本地运行。
2.  **前端构建 (适用于生产环境或需要最新构建产物时)**:
    * 在 Monorepo 根目录 (`realtime-translation-monorepo/`) 下，通常需要运行一个构建命令来生成前端小部件的优化文件。请查阅 `package.json` 中的 `scripts` 部分，例如运行 `npm run build` 或 `pnpm run build`。构建成功后，生成的文件应位于 `packages/frontend-widget/dist/` 目录下。
3.  **后端签名服务运行中**:
    * 在 Monorepo 根目录 (`realtime-translation-monorepo/`) 下，通过以下命令启动签名服务：
        ```bash
        npm run start-server
        ```
    * 请确保该服务正在 `http://localhost:3000` 端口上运行，或者根据实际部署情况，确保前端能够正确访问到后端服务的地址和端口。
4.  **麦克风权限**:
    * 浏览器将请求访问你的麦克风。请务必授予权限，否则语音输入功能将无法使用。

### 运行步骤 (本地开发/测试环境)

1.  **导航到前端小部件目录**:
    ```bash
    cd packages/frontend-widget
    ```
2.  **启动前端小部件开发服务器**:
    * 在 `packages/frontend-widget/` 目录下，运行以下命令启动本地开发服务器：
        ```bash
        npm start
        ```
    * 你将看到类似 `Starting up http-server, serving .` 的输出，并提示访问 `http://localhost:8081`。
3.  **打开浏览器**:
    * 在浏览器中访问 `http://localhost:8081/index.html`。

### 使用小部件

1.  **开始实时转写与翻译**:
    * 页面加载后，点击浮窗上的 "开始" 按钮。
    * 浏览器会提示你授权麦克风。
    * 授权后，对着麦克风说话，转写和翻译结果将实时显示在浮窗中。
    * 点击 "停止" 按钮可以结束转写和翻译会话。
    * 浮窗支持拖动和调整大小，以适应您的操作习惯。

### 生产环境部署说明

对于生产环境部署，建议采用更健壮的方案，而不是直接使用 `http-server`。

1.  **构建前端文件**:
    * 确保已在 Monorepo 根目录执行了前端构建命令（例如 `npm run build`），生成优化的静态文件到 `packages/frontend-widget/dist/` 目录。
2.  **部署静态文件**:
    * 将 `packages/frontend-widget/` 目录下的 `index.html` 和 `app.js` 文件，以及 `packages/frontend-widget/dist/` 目录下的所有构建产物（例如 `index.umd.js`）部署到以下任一静态文件服务器：
        * **Nginx / Apache**: 这是最常见和推荐的方式，配置 Nginx 或 Apache 指向您的前端文件目录。
        * **对象存储 + CDN**: 例如 AWS S3, Google Cloud Storage 配合 CloudFront 或 Cloudflare，提供高可用性和全球分发能力。
        * **其他静态网站托管服务**: 如 Vercel, Netlify 等。
3.  **后端服务部署**:
    * 将 `servers/signa-server/` 目录下的 Node.js 后端服务部署到服务器。建议使用 `pm2` 或 `forever` 等进程管理器确保服务持续运行，并使用 Nginx 或 Apache 作为反向代理，将前端请求转发到后端服务端口（默认为 `3000`）。
    * **重要**: 确保前端（`index.html` 或 `app.js` 中）引用的后端服务地址是生产环境实际的地址，而不是 `http://localhost:3000`。可能需要在前端代码中配置环境变量或根据部署环境动态调整。

## 核心实现说明 (`app.js` 概述)

`app.js` 是连接 UI、科大讯飞 SDK 和后端服务的核心逻辑文件。它主要负责：

*   **DOM 操作**: 获取按钮和结果显示区域的引用。
*   **`RecorderManager` 初始化**: 使用科大讯飞 SDK 提供的 `RecorderManager` 类来管理麦克风音频的采集。`RecorderManager` 依赖 `processor.worker.js` 和 `processor.worklet.js` 进行音频处理。
*   **后端签名请求**: 通过 `fetch` 请求 `http://localhost:3000/generate-signa` 获取鉴权参数 (`appid`, `ts`, `signa` 等) 以及翻译相关参数 (`lang`, `transType`, `targetLang`, `pd`)。
*   **WebSocket 连接**: 使用获取到的参数构建 WebSocket URL，并与 `wss://rtasr.xfyun.cn/v1/ws` 建立连接。
*   **音频数据传输**: `RecorderManager` 捕获的音频帧数据（`onFrameRecorded` 回调）被实时通过 WebSocket 发送到科大讯飞服务。
*   **结果处理**: 监听 WebSocket 的 `onmessage` 事件，接收并解析科大讯讯飞返回的 JSON 数据，将其中的转写 (`src`) 和翻译 (`dst`) 结果分别更新到 UI。
*   **错误处理与资源清理**: 健全的错误处理机制和在停止/错误时清理 WebSocket 和 RecorderManager 实例的逻辑。

## ⚠️ 注意事项

*   **文件路径**: `app.js` 中 `RecorderManager` 的实例化路径 `new window.RecorderManager("./")` 非常重要，它指定了 `processor.worker.js` 和 `processor.worklet.js` 的相对加载路径。请确保这些文件与 `index.umd.js` 一起放置在 `dist` 目录下，并能被正确访问到。
*   **后端服务**: 此前端部件高度依赖于 `signa-server.js` 提供的签名服务。如果没有后端提供签名，前端将无法与科大讯飞服务建立连接。
*   **浏览器兼容性**: 某些旧版浏览器可能不支持 `AudioWorkletNode`，这会使 `RecorderManager` 退化到使用 `ScriptProcessorNode`。`app.js` 中的逻辑已考虑到这一点。
*   **隐私**: 麦克风访问涉及用户隐私，请确保在使用时遵守相关法律法规，并明确告知用户。

## 🤝 贡献

欢迎对本小部件进行改进和贡献！如果您有任何问题或建议，请提交 Issue 或 Pull Request。
