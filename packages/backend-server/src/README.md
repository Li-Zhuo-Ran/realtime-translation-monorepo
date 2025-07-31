# 实时语音翻译后端签名服务 (@realtime-translation/signa-server)

![Node.js Version](https://img.shields.io/badge/node-v18%2B-brightgreen.svg)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](../../LICENSE)

## 📚 项目简介

这是 `realtime-translation-monorepo` 项目中的一个后端服务，专门用于生成科大讯飞（iFLYTEK）实时语音转写和机器翻译API所需的鉴权签名 (`signa`)。由于科大讯飞的 API 鉴权通常需要敏感的 `APIKey`，直接在前端暴露是不安全的。此服务充当代理，安全地在服务器端生成签名，并将其提供给前端。

# 实时语音转写与翻译 - 后端签名服务

此服务负责根据科大讯飞的鉴权规则生成必要的 `signa` 参数，并提供翻译相关的配置信息，供前端应用调用。它旨在将敏感的 `APIKey` 和 `AppID` 安全地保存在服务器端，避免直接暴露给客户端。

## ✨ 主要功能

* **安全鉴权签名**: 根据科大讯飞的鉴权规则（`HmacSHA1(MD5(appid + ts), api_key)`）生成 `signa`。
* **时间戳 (`ts`) 生成**: 生成当前时间戳，作为签名的一部分。
* **跨域支持**: 配置了 CORS，允许前端应用从不同域名安全地请求签名。
* **翻译参数提供**: 除了鉴权信息，还提供固定的翻译相关参数（如源语言、目标语言、翻译类型等）。

## ⚙️ 技术栈

* **语言**: JavaScript (Node.js)
* **Web 框架**: Express.js
* **加密**: Node.js 内置 `crypto` 模块
* **中间件**: `cors` (跨域资源共享)
* **进程管理 (生产环境推荐)**: PM2

## 🚀 快速启动 (本地开发)

在 Monorepo 根目录中，如果 `package.json` 配置了 `start-all` 脚本，推荐使用 `npm run start-all` 一键启动前端和后端。

如果你想单独启动或开发此后端服务，请遵循以下步骤：

1.  **导航到此服务目录**:
    ```bash
    cd servers/signa-server
    ```
2.  **安装依赖**:
    如果你在 Monorepo 根目录 (`realtime-translation-monorepo/`) 已经运行过 `npm install` (或 `pnpm install`)，则此步骤通常不需要重复，因为依赖会被统一管理。
    如果首次单独启动或确保依赖最新，请在此目录运行：
    ```bash
    npm install
    ```

3.  **配置 API 凭证**:
    打开 `signa-server.js` 文件，找到以下变量。**在生产环境中，强烈建议使用环境变量而非直接修改代码文件**。
    在本地开发时，你可以直接修改它们进行测试：
    ```javascript
    // servers/signa-server/signa-server.js
    const YOUR_APPID = 'YOUR_ACTUAL_APPID';    // <-- 请替换为你在科大讯飞控制台获取的真实 AppID！
    const YOUR_APIKEY = 'YOUR_ACTUAL_APIKEY';  // <-- 请替换为你在科大讯飞控制台获取的真实 APIKey！
    ```
    **重要提示**: 这些凭证是您访问科大讯飞 API 的关键，请务必保密，不要泄露！

4.  **启动服务**:
    * **开发模式 (推荐)**: 如果你希望在文件更改时自动重启服务，可以使用此命令。这通常需要你全局或本地安装 `nodemon` (`npm install -g nodemon` 或作为开发依赖安装并在 `package.json` 中配置脚本)。
        ```bash
        npm run dev # 假设 package.json 中有 "dev": "nodemon signa-server.js"
        ```
    * **生产模式 / 普通模式**: 启动服务，不带自动重启功能。
        ```bash
        npm start # 假设 package.json 中有 "start": "node signa-server.js"
        ```

    服务将默认监听 `http://localhost:3000`。你会在控制台看到类似以下输出：
    ```
    Signa server listening at http://localhost:3000
    Frontend will request: http://localhost:3000/generate-signa
    Remember to replace YOUR_APPID and YOUR_APIKEY in signa-server.js!
    ```

5.  **测试接口**:
    在浏览器中访问 `http://localhost:3000/generate-signa`，或使用 `curl`、Postman 等工具进行测试：
    ```bash
    curl http://localhost:3000/generate-signa
    ```
    你将收到一个 JSON 响应，其中包含 `appid`, `ts`, `signa` 以及翻译相关参数。

## 部署到生产环境

将此后端服务部署到生产环境需要将其放置在您的服务器上，并配置适当的 Node.js 运行环境、进程管理和网络访问。

1.  **准备服务器环境**:
    * **安装 Node.js**: 推荐使用 [NVM (Node Version Manager)](https://github.com/nvm-sh/nvm) 来安装和管理 Node.js 版本。
        ```bash
        curl -o- [https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh](https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh) | bash
        # (安装后重启终端或执行 source ~/.bashrc)
        nvm install --lts # 安装最新LTS版本
        nvm use --lts
        ```
    * **安装 PM2**: 这是一个强大的 Node.js 进程管理器，确保服务在后台持续运行并在崩溃时自动重启。
        ```bash
        npm install -g pm2
        ```

2.  **文件上传**:
    将整个 `servers/signa-server` 目录上传到您的服务器的合适位置（例如 `/var/www/your_project/servers/signa-server/` 或 `/home/user/your_project/servers/signa-server/`）。
    推荐使用 Git 进行部署：
    ```bash
    # 在服务器上
    git clone your_repository_url /path/to/your_project
    cd /path/to/your_project/servers/signa-server
    ```

3.  **安装依赖**:
    在服务器上导航到 `servers/signa-server` 目录下运行：
    ```bash
    npm install
    ```

4.  **配置 API 凭证 (生产环境最佳实践)**:
    **强烈建议通过环境变量来管理您的 `YOUR_APPID` 和 `YOUR_APIKEY`**，而非直接修改 `signa-server.js` 文件。这增加了安全性，并方便在不同环境中切换凭证。
    * **在 `signa-server.js` 中使用环境变量**:
        ```javascript
        const YOUR_APPID = process.env.XF_APPID || 'YOUR_DEFAULT_APPID'; // 生产环境应设置 XF_APPID
        const YOUR_APIKEY = process.env.XF_APIKEY || 'YOUR_DEFAULT_APIKEY'; // 生产环境应设置 XF_APIKEY
        ```
    * **通过 PM2 启动时设置环境变量**:
        ```bash
        pm2 start signa-server.js --name "realtime-translation-signa-server" --env production -- XF_APPID=YOUR_ACTUAL_APPID XF_APIKEY=YOUR_ACTUAL_APIKEY
        ```
        或者，在服务器的启动脚本或系统级环境变量中设置。

5.  **配置 CORS**:
    如果前端部署在与后端不同的域名下（这在生产环境中很常见），请务必在 `signa-server.js` 中精确配置 `cors` 中间件的 `origin` 选项，将其设置为您的前端应用所运行的域名。**切勿在生产环境中使用 `origin: '*' `**。
    ```javascript
    app.use(cors({
        origin: '[https://your-frontend-domain.com](https://your-frontend-domain.com)', // <-- 替换为你的前端生产域名，例如 [https://realtime-translation.com](https://realtime-translation.com)
        methods: ['GET'], // 根据实际需求，只允许GET请求
        allowedHeaders: ['Content-Type']
    }));
    ```

6.  **进程管理 (使用 PM2)**:
    在 `servers/signa-server` 目录下启动服务并由 PM2 管理：
    ```bash
    # 在 servers/signa-server 目录下启动服务
    pm2 start signa-server.js --name realtime-translation-signa-server
    # 查看服务状态
    pm2 list
    # 配置开机自启 (按提示执行命令)
    pm2 save
    pm2 startup
    ```
    这将确保服务在服务器重启后自动启动，并提供日志管理和监控功能。

7.  **防火墙配置**:
    确保服务器防火墙开放了您后端服务监听的端口（默认为 `3000`）。
    * **UFW (Ubuntu/Debian)**:
        ```bash
        sudo ufw allow 3000/tcp
        sudo ufw enable
        ```
    * **firewalld (CentOS/RHEL)**:
        ```bash
        sudo firewall-cmd --zone=public --add-port=3000/tcp --permanent
        sudo firewall-cmd --reload
        ```

8.  **反向代理 (强烈推荐)**:
    为了提供更好的安全性、性能（如 SSL 终止、缓存）和可管理性，强烈建议使用 Nginx 或 Apache 作为反向代理，将外部请求转发到您的 Node.js 服务。
    以下是 **Nginx 示例配置**（通常位于 `/etc/nginx/sites-available/your_backend_api.conf`）：
    ```nginx
    server {
        listen 80;
        listen [::]:80;
        server_name api.your-domain.com; # <-- 替换为您的后端 API 域名

        # 如果需要 HTTPS (强烈推荐)，请取消注释并配置 SSL 证书
        # listen 443 ssl;
        # listen [::]:443 ssl;
        # ssl_certificate /etc/letsencrypt/live/[api.your-domain.com/fullchain.pem](https://api.your-domain.com/fullchain.pem);
        # ssl_certificate_key /etc/letsencrypt/live/[api.your-domain.com/privkey.pem](https://api.your-domain.com/privkey.pem);
        # include /etc/letsencrypt/options-ssl-nginx.conf;
        # ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

        location / {
            proxy_pass http://localhost:3000; # Node.js 服务在服务器内部监听的端口
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme; # 用于在 Node.js 中获取请求协议 (http/https)
        }

        # 可选：如果希望将日志记录到特定文件
        access_log /var/log/nginx/api.your-domain.com_access.log;
        error_log /var/log/nginx/api.your-domain.com_error.log;
    }
    ```
    配置完成后，激活站点并重启 Nginx：
    ```bash
    sudo ln -s /etc/nginx/sites-available/your_backend_api.conf /etc/nginx/sites-enabled/
    sudo nginx -t # 测试 Nginx 配置语法
    sudo systemctl restart nginx
    ```
    **重要**: 如果您使用反向代理，前端应用中请求签名接口的 URL 需要更新为反向代理的地址（例如 `https://api.your-domain.com/generate-signa`），而不是 `http://localhost:3000/generate-signa`。

## ⚠️ 安全提示

* **API 凭证保护**: `APIKey` 和 `AppID` 是您在科大讯飞的账户凭证。此服务的主要目的是将它们安全地保存在服务器端，避免在前端代码中直接暴露。**在任何情况下都不要将它们硬编码在公共可访问的前端代码中。**
* **环境变量**: 始终通过环境变量或安全的秘密管理服务（如 AWS Secrets Manager, HashiCorp Vault）来管理敏感凭证，而不是直接写入代码。
* **CORS 配置**: 在生产环境中，`cors` 的 `origin` 选项必须精确设置为您的前端应用可信的域名（例如 `https://your-frontend-domain.com`），切勿使用 `*`。
* **HTTPS**: 对于任何生产环境的应用，强烈建议为所有流量（包括前端和后端 API）配置 HTTPS，以加密数据传输，保护用户隐私和数据完整性。
* **错误处理与日志**: 生产环境中的后端服务应包含更健壮的错误处理、输入验证和日志记录机制。日志应集中管理，方便监控和故障排查。
* **依赖更新**: 定期更新所有 Node.js 依赖，以获取安全补丁和性能改进。
* **最小权限原则**: 为 Node.js 进程运行的用户账户分配最小必要的权限。

## 🤝 贡献

欢迎对本服务进行改进和贡献！如果您有任何问题或建议，请提交 Issue 或 Pull Request 到 Monorepo 的主仓库。

## 📄 许可证

本项目采用 MIT 许可证，详情请参阅 Monorepo 根目录下的 `LICENSE` 文件。
