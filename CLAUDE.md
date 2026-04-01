# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

Happy Coder 是一个为 Claude Code 和 Codex 提供移动端和 Web 客户端的开源项目。它允许用户从任何地方通过端到端加密的方式控制 AI 编码助手。

### 核心组件

项目采用 monorepo 架构，包含以下主要包：

- **[happy-app](packages/happy-app/)** - React Native (Expo) 移动客户端和 Web 应用
- **[happy-cli](packages/happy-cli/)** - Claude Code 和 Codex 的命令行包装器
- **[happy-agent](packages/happy-agent/)** - 远程代理控制 CLI
- **[happy-server](packages/happy-server/)** - 加密同步后端服务器
- **[happy-wire](packages/happy-wire/)** - 共享的消息协议类型和 Zod schemas

## 常用命令

### 根级别命令

```bash
# 运行 CLI
yarn cli

# 运行 Web 应用
yarn web

# 发布版本 (维护者)
yarn release

# 安装依赖后自动运行
yarn postinstall
```

### CLI 开发 (packages/happy-cli)

```bash
# 构建项目
yarn workspace happy-coder build

# 类型检查
yarn workspace happy-coder typecheck

# 运行测试
yarn workspace happy-coder test

# 开发模式运行
yarn workspace happy-coder dev

# 使用本地服务器开发
yarn workspace happy-coder dev:local-server

# 启动稳定版本守护进程
yarn workspace happy-coder stable:daemon:start
yarn workspace happy-coder stable:daemon:stop
yarn workspace happy-coder stable:daemon:status

# 启动开发版本守护进程
yarn workspace happy-coder dev:daemon:start
yarn workspace happy-coder dev:daemon:stop
yarn workspace happy-coder dev:daemon:status

# 创建开发版本符号链接
yarn workspace happy-coder link:dev
```

### App 开发 (packages/happy-app)

```bash
# 启动 Expo 开发服务器
yarn workspace happy-app start

# 运行 iOS
yarn workspace happy-app ios

# 运行 Android
yarn workspace happy-app android

# 运行 Web
yarn workspace happy-app web

# 类型检查
yarn workspace happy-app typecheck

# 生成原生目录
yarn workspace happy-app prebuild

# 部署 OTA 更新
yarn workspace happy-app ota

# macOS Desktop (Tauri)
yarn workspace happy-app tauri:dev
yarn workspace happy-app tauri:build:dev
yarn workspace happy-app tauri:build:preview
yarn workspace happy-app tauri:build:production
```

### Server 开发 (packages/happy-server)

```bash
# 启动服务器
yarn workspace happy-server start

# 开发模式
yarn workspace happy-server dev

# 类型检查
yarn workspace happy-server build

# 运行测试
yarn workspace happy-server test

# 数据库迁移
yarn workspace happy-server migrate

# 生成 Prisma 客户端
yarn workspace happy-server generate

# 启动本地 PostgreSQL
yarn workspace happy-server db

# 启动本地 Redis
yarn workspace happy-server redis
```

### Agent 开发 (packages/happy-agent)

```bash
# 开发模式
yarn workspace @slopus/agent dev

# 构建
yarn workspace @slopus/agent build

# 类型检查
yarn workspace @slopus/agent typecheck
```

### Wire 包开发 (packages/happy-wire)

```bash
# 构建
yarn workspace @slopus/happy-wire build

# 类型检查
yarn workspace @slopus/happy-wire typecheck

# 运行测试
yarn workspace @slopus/happy-wire test
```

## 架构概览

### 整体架构

Happy Coder 采用分布式架构，主要组件通过 WebSocket 和端到端加密进行通信：

1. **CLI (happy-cli)** - 在用户计算机上运行，包装 Claude Code/Codex
2. **App (happy-app)** - 移动端和 Web 客户端，提供远程控制界面
3. **Server (happy-server)** - 中继服务器，处理加密消息转发和会话管理
4. **Agent (happy-agent)** - CLI 工具，用于远程创建和管理会话
5. **Wire (happy-wire)** - 共享类型定义，确保各组件间类型安全

### CLI 架构 (packages/happy-cli)

```
src/
├── api/              # 服务器通信和加密
│   ├── api.ts        # 主要 API 客户端
│   ├── apiSession.ts # WebSocket 会话客户端
│   ├── auth.ts       # 认证流程
│   └── encryption.ts # 端到端加密
├── claude/           # Claude Code 集成
│   ├── loop.ts       # 主控制循环
│   ├── claudeSdk.ts  # SDK 集成
│   └── types.ts      # 消息类型定义
├── ui/               # 用户界面组件
│   ├── logger.ts     # 日志系统
│   └── start.ts      # 应用启动
└── daemon/           # 守护进程逻辑
```

关键设计决策：
- **双模式运行**：interactive（终端）和 remote（移动端控制）
- **端到端加密**：所有通信使用 TweetNaCl 加密
- **会话持久化**：支持跨重启恢复会话
- **文件日志**：避免干扰 Claude 终端 UI

### App 架构 (packages/happy-app)

```
sources/
├── app/              # Expo Router 屏幕和路由
├── auth/             # QR 码认证逻辑
├── components/       # 可复用 UI 组件
├── sync/             # 实时同步引擎和加密
└── utils/            # 工具函数
```

关键特性：
- **React Native + Expo** SDK 54
- **Unistyles** 跨平台样式系统
- **Socket.io** 实时 WebSocket 通信
- **libsodium** 端到端加密
- **LiveKit** 实时语音通信
- **国际化支持**：使用 `t()` 函数处理所有用户可见字符串

### Server 架构 (packages/happy-server)

```
sources/
├── app/              # 应用入口点
│   ├── api.ts        # API 服务器设置
│   └── timeout.ts    # 超时处理
├── apps/             # 应用目录
│   └── api/          # API 服务器应用
│       └── routes/   # API 路由
├── modules/          # 可复用模块
├── services/         # 核心服务
└── storage/          # 数据库和存储工具
```

技术栈：
- **Fastify 5** Web 框架
- **Prisma** ORM + PostgreSQL
- **Socket.io** 实时通信
- **Redis** 缓存和发布/订阅
- **Zod** 输入验证

## 开发规范

### 通用规则

1. **TypeScript 严格模式** - 所有包都启用严格模式
2. **使用 Yarn** - 不使用 npm
3. **4 空格缩进** - 不是 2 空格
4. **路径别名** - 使用 `@/` 前缀导入
5. **最小修改原则** - 只修改必要的代码

### CLI 特定规范

- 避免创建小函数/getter/setter
- 避免过多的 if 语句
- **所有导入必须在文件顶部**
- 优先使用命名导出
- 使用 Zod 进行类型验证

### App 特定规范

- **始终使用 `t()` 函数**处理用户可见字符串
- 新字符串必须添加到所有语言文件
- 检查 `common` 部分避免重复翻译
- 使用 `ItemList` 组件作为主要容器
- 始终应用 layout width constraints
- 从不使用 `Alert` 模块，使用 `@/modal`
- 页面始终使用 `memo` 包装
- 样式放在文件末尾

### Server 特定规范

- 优先使用接口而非类型
- 避免使用枚举，使用 map
- 操作必须是幂等的
- **不要自己创建数据库迁移**
- 使用 `inTx` 包装数据库操作
- 功能性编程模式，避免类

## 工作流程

### 开始新任务

1. 阅读相关包的 `CLAUDE.md` 文件
2. 检查现有代码模式
3. 运行类型检查：`yarn workspace <package> typecheck`
4. 运行测试：`yarn workspace <package> test`

### CLI 开发流程

1. 构建项目：`yarn workspace happy-coder build`
2. 启动开发守护进程：`yarn workspace happy-coder dev:daemon:start`
3. 查看日志：`tail -f ~/.happy-dev/logs/*.log`
4. 测试更改

### App 开发流程

1. 启动开发服务器：`yarn workspace happy-app start`
2. 在模拟器或设备上运行
3. 更新 CHANGELOG.md
4. 运行：`npx tsx sources/scripts/parseChangelog.ts`
5. 类型检查：`yarn workspace happy-app typecheck`

### Server 开发流程

1. 启动本地服务：`yarn workspace happy-server db` 和 `yarn workspace happy-server redis`
2. 启动服务器：`yarn workspace happy-server dev`
3. 查看日志：`tail -f .logs/*.log`
4. **不要创建迁移**，只运行 `yarn workspace happy-server generate`

## 调试技巧

### CLI 调试

```bash
# 检查守护进程状态
yarn workspace happy-coder dev:daemon:status

# 查看最新日志
ls -lt ~/.happy-dev/logs/ | head -5

# 监控会话创建
tail -f ~/.happy-dev/logs/*.log | grep -i session

# 检查认证流程
tail -f ~/.happy-dev/logs/*.log | grep -i auth
```

### Server 调试

```bash
# 检查当前时间（日志使用本地时间）
date

# 检查最新日志文件
ls -la .logs/*.log | tail -5

# 监控错误
tail -100 .logs/*.log | grep -E "(error|Error|ERROR)"

# 跟踪会话创建
tail -500 .logs/*.log | grep "new-session"

# 检查 socket 连接
tail -200 .logs/*.log | grep -E "(websocket|Socket.*connected)"
```

### App 调试

- 使用 Expo 开发菜单查看错误
- 检查 React Native Debugger
- 查看设备日志

## 重要文件位置

- **CLI 配置**：`~/.happy/` (stable) 或 `~/.happy-dev/` (dev)
- **CLI 日志**：`~/.happy/logs/` 或 `~/.happy-dev/logs/`
- **Server 日志**：`.logs/`
- **Prisma schema**：`packages/happy-server/prisma/schema.prisma`

## 版本管理

项目支持稳定版本和开发版本并行运行：

- **稳定版本**：使用 `~/.happy/` 数据目录
- **开发版本**：使用 `~/.happy-dev/` 数据目录

两者完全隔离，可同时运行。

## 发布流程

维护者发布新版本：

```bash
# 从 repo 根目录
yarn release

# 或发布特定包
yarn workspace happy-coder release
yarn workspace @slopus/happy-wire release
```

## 相关资源

- [文档网站](https://happy.engineering/docs/)
- [GitHub 仓库](https://github.com/slopus/happy)
- [Discord 社区](https://discord.gg/fX9WBAhyfD)
