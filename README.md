# realtime-translation-monorepo
A monorepo for real-time translation widget (frontend) and its backend service (Node.js/Express)
# 🚀 实时语音转写与翻译 Monorepo

![Project Status](https://img.shields.io/badge/status-active-brightgreen.svg)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/Li-Zhuo-Ran/realtime-translation-monorepo/main.yml?branch=main)](https://github.com/Li-Zhuo-Ran/realtime-translation-monorepo/actions)
<!-- 如果你未来有CI/CD或部署状态，可以在这里添加徽章 -->

## 📚 项目简介

`realtime-translation-monorepo` 是一个集成了科大讯飞实时语音转写和翻译功能的 Monorepo 项目。它包含了后端签名服务和前端语音浮窗小部件，旨在为网页应用提供便捷的实时多语言语音交互能力，特别是为学院官方网站等场景提供增强的可访问性和用户体验。

## ✨ 主要功能

*   **后端签名服务 (`servers/signa-server`)**:
    *   提供安全的鉴权签名 (`signa`)，用于访问科大讯飞的实时语音转写和机器翻译API。
    *   处理跨域资源共享 (CORS)。
*   **前端浮窗小部件 (`packages/frontend-widget`)**:
    *   **实时语音输入**: 浏览器内直接通过麦克风采集语音。
    *   **实时转写**: 将麦克风捕获的英文语音内容即时转换为文本。
    *   **实时翻译**: 将转写后的英文文本实时翻译为中文。
    *   **UI 浮窗展示**: 以可拖动、简洁的浮窗形式在网页上同步展示转写和翻译结果。
    *   **SDK 集成**: 深度整合科大讯飞 Web 端 SDK (`RecorderManager`) 进行音频处理和数据传输。

## ⚙️ 技术栈

### 核心技术

*   **Monorepo 管理**: npm Workspaces (或 pnpm Workspaces)
*   **实时通信**: WebSocket
*   **语音处理**: 科大讯飞 Web 端 SDK (基于 Web Audio API, Web Worker, AudioWorklet)

### 后端 (`servers/signa-server`)

*   **语言**: Node.js
*   **框架**: Express.js
*   **加密**: Node.js `crypto` 模块 (用于 HMAC-SHA1 和 MD5)
*   **中间件**: `cors` (处理跨域)

### 前端 (`packages/frontend-widget`)

*   **语言**: HTML, CSS, JavaScript (ES6+)
*   **SDK**: 科大讯飞 Web 端实时语音转写与翻译 SDK
*   **开发服务器**: `http-server` (用于本地开发和演示)

## 📦 项目结构

realtime-translation-monorepo/
├── packages/                               # 前端小部件相关代码
│   └── frontend-widget/
│       ├── dist/                           # 编译和分发产物（科大讯飞SDK核心文件）
│       │   ├── index.cjs.js
│       │   ├── index.esm.js
│       │   ├── index.umd.js
│       │   ├── index.d.ts
│       │   ├── processor.worker.js
│       │   └── processor.worklet.js
│       ├── app.js                          # 前端核心业务逻辑，SDK调用和UI交互
│       ├── index.html                      # 前端示例页面，用于演示和测试
│       └── package.json                    # 前端小部件独立的依赖和脚本
│       └── README.md                       # 前端小部件的详细说明
├── servers/                                # 后端服务相关代码
│   └── signa-server/
│       └── signa-server.js                 # 提供科大讯飞API签名的后端服务
│       └── package.json                    # 后端服务的依赖和脚本
│       └── README.md                       # 后端服务的详细说明
├── package.json                            # Monorepo 根配置：定义工作区、通用脚本和共享依赖
├── package-lock.json                       # Monorepo 依赖锁定文件
└── README.md                               # 项目主 README 文件


## 🚀 快速启动 (本地开发)

按照以下步骤在本地启动整个实时语音转写与翻译系统。

### 1. 前期准备

1.  **Node.js**: 确保您的机器上安装了 Node.js (推荐 v18 或更高版本)。
2.  **Git**: 确保已安装 Git。
3.  **科大讯飞 API 凭证**:
    *   前往 [科大讯飞开放平台](https://www.xfyun.cn/) 注册账号。
    *   创建一个新的应用，并开通**实时语音转写**和**机器翻译**服务。
    *   获取您的 **AppID**, **APIKey**。请注意，实时转写服务在 WebSocket 连接时使用 `AppID` 和 `APIKey` 生成 `signa`，而翻译服务可能需要 `APISecret`，请参照科大讯飞最新文档。本项目 `signa-server.js` 使用 `AppID` 和 `APIKey`。

### 2. 克隆仓库

```bash
git clone https://github.com/Li-Zhuo-Ran/realtime-translation-monorepo.git
cd realtime-translation-monorepo
