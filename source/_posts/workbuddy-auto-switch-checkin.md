---
title: WorkBuddy 自动切账号和自动签到
date: 2026-09-09 09:57:00
updated: 2026-09-09 09:57:00
tags:
  - WorkBuddy
  - Electron
  - CDP
  - 自动化
  - 外挂增强
categories:
  - 教程
cover: /img/cover-workbuddy.png
top_img: false
description: WorkBuddy 是 Electron 套壳网页。本文讲清 CDP 旁路注入思路，以及 WorkDaddy 如何用调试口实现自动切账号、自动签到等零侵入增强，而不靠 RPA 模拟点击。
keywords: WorkBuddy,WorkDaddy,Electron,CDP,自动签到,切账号
---

{% note info %}
口语交流：WorkBuddy 是 Electron，本质就是 Chromium + Node 封装出来的套壳网页，界面运行逻辑和浏览器页面一致。  
技术严谨描述：Electron 以 Chromium 负责网页 UI 渲染、内置 Node.js 提供系统能力，再加一层框架胶水实现桌面原生能力；所以可以用 CDP 旁路注入做外挂增强层，而不是 RPA 模拟点击。
{% endnote %}

## CDP（Chrome DevTools Protocol）

一句话：CDP 是 Chromium 内核对外暴露的一套双向远程控制 / 调试协议。你 F12 打开 DevTools，DevTools 本身就是用 CDP 和浏览器内核通信；Electron 应用因为底层是 Chromium，同样支持 CDP，只要启动时开启远程调试端口，外部程序就能连上它的渲染页面。

> 通信底层：WebSocket + JSON-RPC，外部程序发指令，Chromium 回结果，还能主动推送事件（网络请求、页面加载、JS 报错）。

### CDP 核心能力（对应 WorkBuddy 外挂增强层场景）

1. **DOM 读取与操纵**：直接读取页面 DOM 树、元素属性，修改 HTML；不需要截图图像识别，直接拿到页面真实数据。
2. **页面内执行 JS**：在 Electron 渲染进程里任意运行 JavaScript，读取页面全局变量、调用页面本身函数。
3. **网络流量监听 / 拦截 / 篡改**：捕获所有请求、响应，拿到接口返回数据；还能修改请求参数、返回结果（这个做外挂非常关键）。
4. **页面控制**：点击元素、输入文本、截图、监听页面事件。
5. **调试、性能采集**：断点调试、内存 / CPU 性能分析、捕获控制台日志。

## 功能介绍

- **方便切换账号**：每个 WorkBuddy 账号独立备份，点一下就切，再也不用每次扫码。
- **账号导入导出**：把全部账号备份加密导出，在另一台电脑安装 WorkDaddy 后一键导入，方便电脑之间迁移账号。
- **自动领每日积分**：打开面板即对全部账号静默签到，幂等缓存，不打断你。
- **暂存提示词**：输入框边上一键把草稿暂存到待发送队列——图片 / 文件 / 引用原样保留，择机发送。
- **切换精美主题**：内置毛玻璃官方主题，多套预设壁纸，支持自定义壁纸。
- **账号间会话迁移**：把 A 账号的整段历史会话一键复制到 B 账号，跨账号继续接龙。
- **强化决策弹窗**：WorkBuddy 让你选/确认时，弹窗来点而不是文本询问（省得打字）。
- **防止电脑休眠**：睡前任务未完成，开启防休眠模式，任务结束后自动切换成允许休眠。

不动官方安装包，基于 **CDP 注入**，WorkBuddy 版本升级也不受影响。

环境要求：macOS、Windows  
客户端支持：国内版 WorkBuddy、国际版 WorkBuddy AI

![WorkBuddy 增强面板示意](/img/workbuddy/01-overview.png)

## 核心原理：为什么能零侵入？

WorkBuddy 本质是套了壳的网页（Electron = Chromium + Node）。

Chrome 有个官方能力叫 CDP（Chrome DevTools Protocol）：开发者工具（F12）背后，就是靠这套协议在遥控页面。

WorkDaddy 做的事等价于：

> 不开 F12 窗口，用程序连上同一套协议，往页面里执行一段 JS。

关键一步：启动时多带一个参数：

```text
--remote-debugging-port=9222
```

含义：在本机 9222 开一个只给本机用的调试口。Chromium 提供远程调试启动参数，加上这个参数启动程序后，会在本机开放 9222 端口，暴露 CDP 的 WebSocket 服务；外部程序（Python / Node 脚本，比如 Playwright、Puppeteer）就可以连接这个端口，发送 CDP 指令。

安装包、签名、asar 都不改，只是启动方式变了。

## 一步一步：你双击 WorkDaddy 之后发生了什么

### 第 1 步：启动器把 WorkBuddy 开成可调试模式

正常双击官方图标 → 没有调试口 → 外人进不去。

WorkDaddy 的 launcher 会：

1. 找到本机 WorkBuddy 可执行文件
2. 用多一个调试参数的方式启动它
3. 同时拉起守护进程 `daemon.js`

此时：WorkBuddy 窗口照常出现，但后台多了一扇门。

### 第 2 步：守护进程当管家，连上那扇门

`daemon.js` 会做类似这样的事：

1. 问 `http://127.0.0.1:9222/json/list`——里面有哪些页面？
2. 拿到 WebSocket 地址，连上 CDP
3. 之后就能发命令，例如：
   - `Runtime.evaluate`：在页面里执行一段 JavaScript
   - `Page.reload`：刷新窗口（切换账号后常用）

类比：管家拿着对讲机，可以对店里喊：帮我执行这段代码。

### 第 3 步：把面板注入进页面（你看到机器人按钮）

管家读出 `inject.js` 的全部内容，通过 CDP 发：

`Runtime.evaluate` → 在 WorkBuddy 页面里跑这段 JS

`inject.js` 在页面里会：

1. 清掉旧面板（防止重复注入叠两个按钮）
2. `createElement` 做出右下角按钮和弹窗
3. 挂上点击事件

于是你感觉 WorkBuddy 多了功能，其实只是页面 DOM 里多了一块外来 HTML。

官方代码没被改；刷新/重连后，管家会再注入一次。

### 第 4 步：你点按钮时，面板不自己干重活

面板（inject）和管家（daemon）的分工：

```text
你点「切换账号」
    ↓
inject.js 用 fetch 访问本机
    http://127.0.0.1:47832/api/switch
    ↓
daemon.js 收到请求
    ↓
改本地账号备份文件（真正换登录态）
    ↓
通过 CDP 让 WorkBuddy 刷新页面
    ↓
你看到已经是另一个账号
```

为什么不让面板直接改账号文件？

- 页面里的 JS 权限有限，也不该碰敏感文件
- 敏感操作集中在本机 daemon，更安全、更可控
- API 只监听 `127.0.0.1`（本机），外网进不来

### 第 5 步：账号备份是怎么来的

登录信息本来就在 WorkBuddy 自己的本地文件里。

守护进程会：

1. 听 CDP 网络事件（登录/刷新 token 时）
2. 或盯文件变化（兜底）
3. 按账号 ID 复制一份到 WorkDaddy 的备份目录

切换账号 ≈：

> 把选中的那份备份写回 WorkBuddy 正在用的位置 → 再刷新界面。

这就是点一下切号、不用反复扫码的底层原因。

## 一张总图（对着看）

```text
你（鼠标）
  │
  ▼
WorkBuddy 窗口里的面板 (inject.js)  ←── CDP 注入进来的
  │  fetch 本机 API
  ▼
daemon.js（管家）
  ├── 读写账号备份（本地磁盘）
  ├── HTTP :47832（只给本机）
  └── CDP :9222（遥控 WorkBuddy 页面）
         │
         ▼
    WorkBuddy 渲染进程（看起来像网页）
```

## 各功能分别落在哪一层

| 你看到的功能 | 主要靠谁 | 原理一句话 |
| ------------ | -------- | ---------- |
| 右下角机器人 / 面板 UI | inject.js | 往页面塞 DOM |
| 主题 / 毛玻璃样式 | inject.js + theme-patches.js | 注入 CSS，改外观 |
| 账号列表 / 切换 / 导入导出 | daemon.js + lib.js | 管本地 JSON 备份 |
| 切换后立刻生效 | daemon 调 CDP Page.reload | 刷新渲染进程 |
| 自动签到、部分增强 | daemon API + 偶发 CDP | 后台代调，面板只负责展示 |
| 安装、保活、更新 | launcher / watchdog | 保证带调试口启动 + 管家还活着 |
