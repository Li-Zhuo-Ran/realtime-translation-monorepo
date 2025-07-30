# 实时语音翻译后端签名服务 (@realtime-translation/signa-server)

![Node.js Version](https://img.shields.io/badge/node-v18%2B-brightgreen.svg)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](../../LICENSE)

## 📚 项目简介

这是 `realtime-translation-monorepo` 项目中的一个后端服务，专门用于生成科大讯飞（iFLYTEK）实时语音转写和机器翻译API所需的鉴权签名 (`signa`)。由于科大讯飞的 API 鉴权通常需要敏感的 `APIKey`，直接在前端暴露是不安全的。此服务充当代理，安全地在服务器端生成签名，并将其提供给前端。

## ✨ 主要功能

*   **安全鉴权签名**: 根据科大讯飞的鉴权规则（HmacSHA1(MD5(appid + ts), api_key)）生成 `signa`。
*   **时间戳 (`ts`) 生成**: 生成当前时间戳，作为签名的一部分。
*   **跨域支持**: 配置了 CORS，允许前端应用从不同域名安全地请求签名。
*   **翻译参数提供**: 除了鉴权信息，还提供固定的翻译相关参数（如源语言、目标语言、翻译类型等）。

## ⚙️ 技术栈

*   **语言**: JavaScript (Node.js)
*   **Web 框架**: Express.js
*   **加密**: Node.js 内置 `crypto` 模块
*   **中间件**: `cors` (跨域资源共享), `body-parser` (虽然本项目主要处理GET请求，但作为一个Web服务常见依赖)

## 🚀 快速启动 (本地开发)

在 Monorepo 根目录中，推荐使用 `npm run start-all` 一键启动前端和后端。
如果你想单独启动或开发此后端服务：

1.  **导航到此服务目录**:
    ```bash
    cd servers/signa-server
    ```

2.  **安装依赖**:
    如果你在 Monorepo 根目录 (`realtime-translation-monorepo/`) 已经运行过 `npm install`，则此步骤通常不需要重复，因为依赖会被统一管理。
    如果首次单独启动或确保依赖最新：
    ```bash
    npm install
    ```

3.  **配置 API 凭证**:
    打开 `signa-server.js` 文件，找到以下变量并替换为你在科大讯飞开放平台获得的实际 `AppID` 和 `APIKey`。
    ```javascript
    // servers/signa-server/signa-server.js
    const YOUR_APPID = 'YOUR_ACTUAL_APPID';       // <-- 请务必替换！
    const YOUR_APIKEY = 'YOUR_ACTUAL_APIKEY';     // <-- 请务必替换！
    ```
    **重要提示**: 这些凭证是您访问科大讯飞 API 的关键，请务必保密，不要泄露！

4.  **启动服务**:
    *   **开发模式 (推荐)**: 自动重启服务
        ```bash
        npm run dev
        ```
        这需要你全局或本地安装 `nodemon` (`npm install -g nodemon` 或 `npm install nodemon`)。
    *   **生产模式 / 普通模式**:
        ```bash
        npm start
        ```

    服务将默认监听 `http://localhost:3000`。你会在控制台看到类似以下输出：
    ```
    Signa server listening at http://localhost:3000
    Frontend will request: http://localhost:3000/generate-signa
    Remember to replace YOUR_APPID and YOUR_APIKEY in signa-server.js!
    ```

5.  **测试接口**:
    在浏览器或使用 Postman/cURL 访问 `http://localhost:3000/generate-signa`，你将收到一个 JSON 响应，其中包含 `appid`, `ts`, `signa` 以及翻译相关参数。

## 部署到生产环境

将此后端服务部署到生产环境需要将其放置在您的服务器上，并配置适当的进程管理和网络访问。

1.  **文件上传**: 将整个 `servers/signa-server` 目录上传到您的服务器。
2.  **安装依赖**: 在服务器上该目录下运行 `npm install`。
3.  **配置 API 凭证**: 确保 `signa-server.js` 中的 `YOUR_APPID` 和 `YOUR_APIKEY` 已被正确替换为生产环境的凭证。
4.  **配置 CORS**: 如果前端部署在与后端不同的域名下，请务必在 `signa-server.js` 中精确配置 `cors` 中间件的 `origin` 选项，将其设置为您的前端应用所运行的域名，例如：
    ```javascript
    app.use(cors({
        origin: 'https://your-frontend-domain.com', // 替换为你的前端域名
        methods: ['GET'], // 根据实际需求，只允许GET请求
        allowedHeaders: ['Content-Type']
    }));
    ```
5.  **进程管理**: 使用 `pm2`、`systemd` 或 Docker 等工具启动并管理 `signa-server.js` 进程，确保其在后台持续运行，并在服务器重启后自动恢复。
    *   **使用 PM2 示例**:
        ```bash
        # 安装pm2 (如果未安装)
        npm install -g pm2
        # 在 servers/signa-server 目录下启动服务
        pm2 start signa-server.js --name realtime-translation-signa-server
        # 查看服务状态
        pm2 list
        # 设置开机自启
        pm2 save
        pm2 startup
        ```
6.  **防火墙配置**: 确保服务器防火墙开放了您后端服务监听的端口（默认为 3000）。
7.  **HTTPS (强烈推荐)**: 如果您的前端网站使用 HTTPS，强烈建议为后端签名服务也配置 HTTPS。您可以通过反向代理（如 Nginx 或 Apache）在前端终止 SSL，并将请求转发到后端的 HTTP 端口。

## ⚠️ 安全提示

*   **API 凭证保护**: `APIKey` 和 `APISecret` 是您在科大讯飞的账户凭证。此服务的主要目的是将它们安全地保存在服务器端，避免在前端代码中直接暴露。
*   **CORS 配置**: 在生产环境中，`cors` 的 `origin` 选项必须设置为您的前端应用可信的域名，切勿使用 `*`。
*   **错误处理**: 生产环境中的后端服务应包含更健壮的错误处理和日志记录机制。

## 🤝 贡献

欢迎对本服务进行改进和贡献！如果您有任何问题或建议，请提交 Issue 或 Pull Request 到 Monorepo 的主仓库。

## 📄 许可证

本项目采用 MIT 许可证，详情请参阅 Monorepo 根目录下的 `LICENSE` 文件。
