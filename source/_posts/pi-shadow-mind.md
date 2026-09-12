---
title: Pi Shadow Mind：代码审计者身份与并行认知核心
date: 2026-08-17 15:30:00
updated: 2026-09-12 11:37:00
tags:
  - Pi
  - Shadow Mind
  - Agent
  - 代码审计
  - AgentSession
categories:
  - 教程
cover: /img/cover-pi-shadow-mind.png
top_img: false
description: Pi Shadow Mind 让多个专业化认知核心与主 Agent 并行工作：架构审阅、事实核验、文档维护、完成度检查可持续介入，实现与审阅发生在同一轮。
keywords: Pi Shadow Mind,Shadow Mind,代码审计,AgentSession,并行认知核心,report_to_main
---

{% note info %}
一句话：主 Agent 持续推进实现；若干 Shadow Mind 按固定职责并行审阅、核验或维护，在返工成本变高之前介入。
{% endnote %}

## 项目介绍

Pi Shadow Mind 让多个专业化认知核心与主 Agent 并行工作。每个 Shadow Mind 都有一项持续、稳定的职责，例如架构审阅、正确性检查、文档维护、项目事实核验，或任何由你定义的任务。

主 Agent 负责持续推进；其他认知核心独立审阅决策、核验事实、维护相关文件，并在错误演变成高昂返工之前介入。

> 让实现与审阅发生在同一轮工作中。

## 一个 Agent，多项独立职责

| 认知核心 | 职责 |
| --- | --- |
| 架构审阅 | 在编码过程中发现上帝组件、职责错位、模块边界缺失和脆弱的扩展点 |
| 项目事实核验 | 对照真实仓库检查结论，发现模型编造的 API、文件、约束和实现细节 |
| 文档维护 | 跟踪实现变化，让架构说明、设计决策和使用文档保持同步 |
| 完成度审阅 | 在主 Agent 宣布完成前，独立检查结果是否真正满足任务要求 |

这些职责由用户定义、持续存在，可以独立决定何时检查、行动或汇报；主 Agent 不会临时把它们当作一次性子任务派出去。

## Shadow 不只审阅，也可以工作

Shadow Mind 可以保持只读，只向主 Agent 汇报发现；也可以获得额外工具，独立负责另一条任务线。

主 Agent 写代码时，另一个 Shadow 可以同步维护文档、更新架构决策，或处理独立文件。工具权限按每个 Shadow 单独配置，每个认知核心只拿到职责真正需要的能力。

```text
主 Agent                 Architecture Shadow
实现功能                 审阅模块边界

主 Agent                 Documentation Shadow
编写代码                 维护设计文档
```

审阅只是职责之一。Shadow Mind 可以观察、核验、维护，也可以直接构建。

{% note tip %}
默认配置下，这个 Shadow 只读：在实现过程中并行审阅架构，并向主 Agent 报告具体问题，不接管主任务。
{% endnote %}

## 发现问题之后怎么交接

发现问题后，Shadow 回报给主 Agent，由主 Agent 接着处理，不会直接改主任务。

流程大致是：

1. Shadow 调用 `report_to_main({ content: "..." })`，当前 Shadow 会话立刻结束。
2. 报告进入短暂聚合窗口（默认约 400ms），多个 Shadow 的意见可合并成一份。
3. 插件生成可见的 `shadow-report`，再按主 Agent 状态投递：
   - 主 Agent 仍在运行：用 `steer`，插到下一个认知边界；
   - 主 Agent 空闲：用 `follow-up`，触发补充或纠正。
4. 主 Agent 看到报告后，自己决定改代码、解释，还是忽略。
5. 主 Agent 回复的 `turn_end` 可能再次唤醒 Shadow，形成审阅链。用户发来新消息后进入新 epoch，旧的 Shadow 与未投递报告失效。

![Shadow 回报主 Agent 的交接流程](/img/pi-shadow-mind/02-session-vs-rpc.png)

{% note warning %}
要点：Shadow 只指出问题；真正怎么修，仍由主 Agent 执行。超时、中止或用户开了新一轮时，未投递的结果会丢掉，不会硬插进新任务。
{% endnote %}

## 与临时对话 / RPC 模式对比

Pi 把 `AgentSession` 做成可被扩展调用的运行时原语；Cursor 把它收进产品内核，只对外暴露受控边界。因此这套 Shadow Mind 目前只能在 Pi 上落地。

| 项目 | Pi Shadow Mind（当前这套） | `/btw` RPC 模式 |
| --- | --- | --- |
| 隔离层级 | 进程内对象隔离（`AgentSession`） | OS 子进程 `pi --mode rpc` |
| 进程数量 | 始终只有 1 个 pi 主进程 | 每开一个子任务就新建操作系统进程 |
| 开销 | 轻量；主要是内存对象与 LLM API 调用 | 更重；进程创建销毁有操作系统开销 |
| 工具权限 | 每个 Session 的工具白名单在主进程管控 | 子进程独立环境，经 IPC 传指令 |
| 生命周期 | 跑完 `dispose()` 销毁对象 | 操作系统进程退出后回收资源 |

## 什么时候触发 Shadow

主 Agent 的 `turn_end` 之后，插件先做硬控调度，再决定是否初始化 Shadow：

1. **心跳抽签**：按 `heartbeat_probability` 做概率判定。
2. **过滤候选**：去掉未启用、模型不匹配、已有 Shadow 会话在跑的项。
3. **激活抽签**：对剩余候选再按 `activation_probability` 判定。
4. **并发裁剪**：超出并发槽位则本轮不启动。

通过硬控的 Shadow 才会初始化临时会话，输入包括：

- **净化轨迹**：主会话的精简版，不是完整上下文
- 公共协议、Shadow 定义、kickoff 启动指令

初始化之后，模型再按自身职责判断内容：无关则返回 `NOT_RELEVANT`；有关才真正干活。

![Shadow 触发与硬控调度流程](/img/pi-shadow-mind/01-architecture-shadow.png)

{% note info %}
概率与并发决定「这一轮要不要开这个 Shadow」；kickoff 决定「开了之后要不要真干」。输入是净化轨迹，不是主会话全文。
{% endnote %}

## 总结：提示词管什么，代码管什么

![提示词与代码的职责边界](/img/pi-shadow-mind/04-summary.png)

**提示词管什么**

- 每个 Shadow 的 Markdown 正文：关注什么、何时介入、说话风格
- 公共 kickoff 协议：先判是否相关；无关返回 `NOT_RELEVANT`；有结果则 `report_to_main`

**代码管什么（提示词替代不了）**

- 主 Agent `turn_end` 后的心跳、激活概率、并发上限
- 轨迹净化：去掉内部思考块，工具结果只留摘要
- 独立临时 `AgentSession`、工具白名单、超时与 abort
- 把 `report_to_main` 注回主会话，以及 `/shadow` 等管理命令

一句话收束：职责定义写在提示词里；何时启动、怎么隔离、如何投递，必须由代码硬控。
