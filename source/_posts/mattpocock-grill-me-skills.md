---
title: Matt Pocock（grill-me）系列 Skill 工程化路径
date: 2026-09-29 21:48:51
updated: 2026-09-29 21:48:51
tags:
  - Skill
  - Matt Pocock
  - grill-me
  - Agent
  - 工程化
categories:
  - 教程
cover: /img/cover-mattpocock-grill-me-skills.png
top_img: false
description: 按 Matt Pocock 的思路，把项目从对齐想法、落实 Spec、拆分 Ticket，到实现与审核，拆成一套可复用的 Skill 工程化路径。
keywords: Matt Pocock,grill-me,Skill,grill-with-docs,to-spec,to-ticket,implement,code-review
---

{% note info %}
整理自得到大脑笔记《mattpocock（grill-me）系列 skill》。核心是一套文档驱动的工程化路径，配合若干辅助 Skill。
{% endnote %}

## 核心设计

按 Matt 的想法，常规项目工程化大致有这样一个顺序（非常重要），每一步对应一个 Skill：

1. **对齐想法** — `/grill-with-docs`：跟用户对齐想法，并沉淀共识文档
2. **落实文档** — `/to-spec`：把想法落实成设计文档（上一步只是共识想法，这一步才是 Spec）
3. **拆分任务** — `/to-ticket`：把设计文档拆分成可以独立完成的多个任务（ticket）
4. **完成任务** — `/implement`：新开 session 调用它实现拆分好的指定 ticket，比如 `/implement 01`
5. **审核代码** — `/code-review`：审核代码有没有按设计完成、是否符合规范（`/implement` 会自动调用）

我认为这是 Matt 系列 Skill 最核心的设计，一定要理解这条工程化路径。其它很多 Skill 也都是围绕它做辅助。

工程化核心步骤有了，但要让 Agent 真正守住流程，还需要一套大家都遵守的标准文档。因此在这 5 步真正开始之前，还应该先设计好标准，于是就有了 `/setup-matt-pocock-skills`。我把它称作工程化路径的**步骤 0**：项目刚刚建立、新建了文件夹之后，**第一步就执行它**。

1. **设置标准** — `/setup-matt-pocock-skills`：初始化每个步骤都应遵守的标准文档，确定术语，配置 issue tracker、文档布局等

## 使用细节

记住上面几个 Skill，已经能应付大多数项目。下面补充一些灵活用法和注意事项。

- 这套 Skill 依赖文档驱动，基本不依赖上下文，因此拆分的 ticket 都可以在新的 session 中实现，不用再喂上下文；换 Agent 实现也几乎无痛。但 Matt 建议：**从 `/grill-with-docs` 到 `/to-ticket`（步骤 1–3）尽量在一个上下文中完成**。
- `grill-me` / `grill-with-docs` 有时问得非常多。调用时可以告诉它：优缺点十分分明的选项让它自行决定，不用事事来问。提问前你给出的信息也要足够、有效，否则它会没完没了地问。Matt 专门出过视频讲这个问题：[B 站视频](https://www.bilibili.com/video/BV1zn396mEfz)（挺重要）。
- 有人会觉得 `/grill-with-docs` 和 `/to-spec` 都是把想法落到文档，功能重复。其实文档类型不同：前者是拷问用户达成的共识文档，比较零散；后者是把所有共识整理总结后的真正设计文档。
- 工程化 0–5 步**不是强制的**，可按项目大小调整。小项目可以在 `/grill-with-docs` 之后直接跳到 `/implement`，跳过 `/to-spec` 和 `/to-ticket`，或只跳过 `/to-ticket`。
- 为什么有了 Spec 还要拆成 ticket？这是 Matt 的核心思想之一。现在大模型虽然常宣传 1M 上下文，但真正的 smart zone 往往不到 200K（例如 GPT 支持 1M，但 Codex 默认设置还不到 300K）。上下文变大后注意力会下降。通过拆分任务，可以把每个 ticket 需要的上下文控制在 smart zone 以内（**每个 ticket 都推荐在新的 session 中实现**），同时也更好控制模块化进展。
- `/to-ticket` 拆出的 ticket 会包含依赖关系：有些要等前面某个 ticket 完成才能开始；无依赖的可以多个 session 并行。
- `/code-review` 会开两个 subagent 并行审查：一个看规范、是否有 bug；一个看是否偏离设计目标，很有用。
- `/implement` 跟你直接让它开干的区别是：会自动调用 `/tdd` 实现，并自动调用 `/code-review` 审查。因此步骤 5 一般是自动完成的，你也可以随时手动调用审查刚写的代码。
- `/implement-spec` 可以按依赖关系和顺序，用 subagent 自动实现一组 Spec 拆分出的 ticket。这是较新的官方能力；之前 ticket 只能用户自己开 session 实现。
- `/ask-matt` 非常有用，相当于 Matt 系列 Skill 的帮助入口。上面说的这些，用它问通常能得到更细的解释：用法、是否适用、推荐流程等都可以问。

## 其它常用 Skill

- `/improve-codebase-architecture`：架构优化，排查重复代码，给模块化建议。多数结论仍需要再走一轮从 `grill-me` / `grill-with-docs` 开始的工程化 1–5 步；简单整改也可以直接让 AI 改。我一般在它结束后，给每个候选 fork 一个 session，在各自 session 里走后续流程。
- `/wayfinder`：对较大项目很有用。有些地方自己设计时都没想清楚，直接 grilling 反而容易让文档走偏。用它引导走一遍规划：现在能定的生成可直接实现的 ticket；还定不了的生成决策 ticket（后续还要探索，不能直接写代码）。决策 ticket 可用 `/wayfinder TICKET_ID/TICKET_PATH` 推进。
- `/codebase-design`：设计模块 / 接口时推荐用；实现 ticket 时也可能按需自动调用。
- `/prototype`：先不真正写代码，根据描述快速做出几个备选原型让你选。
- `/writing-for-agents`：把你想让 Agent 遵守的内容写进相关文档。比自己手改更合适——给 Agent 读的文档并不是越详细越好，它会按适合 Agent 的方式组织，并尽量省 token。
- `/handoff`：把当前 session 内容生成文档，别的 session 读了就能继续干。走了前面 0–5 步的项目几乎不太需要它。
- `/wait-what`：把太啰嗦的 Agent 回复简化成说人话。实测直接让 Agent 用大白话重讲，效果往往也不错。
- `/research`：启动 subagent 调研，不占主 session 上下文。提示词本身不复杂，核心是刨根问底找来源，不轻信二手说法，调研准确度还不错。
- `/teach`：让它教你理解项目，会生成可互动的网页，还有问答题加深理解。

## 表格总览

| Skill | 功能 | 备注 |
| --- | --- | --- |
| `/setup-matt-pocock-skills` | 配置 issue tracker、文档布局 | 工程化步骤 0，一切的开始 |
| `/grill-with-docs` | 对齐想法并沉淀文档 | 工程化步骤 1 |
| `/to-spec` | 想法落实成设计文档 | 工程化步骤 2 |
| `/to-ticket` | 拆成较小 ticket，可单独完成 | 工程化步骤 3 |
| `/implement` | 用 `/tdd` 实现计划，并用 `/code-review` 检查 | 工程化步骤 4；可跳过 Spec 和 ticket 直接实现 |
| `/code-review` | 检查是否按规范和设计文档执行，调用 2 个 subagent | 工程化步骤 5 |
| `/implement-spec` | 对 Spec 对应的所有 ticket 按依赖顺序在 worktree 自动实现 | |
| `/tdd` | 用 Test-Driven Development 方法开发 | |
| `/codebase-design` | 设计原则，规范代码形状与深度模块设计 | `/tdd` 会自动调用；设计模块 / 接口时建议主动调用 |
| `/prototype` | 做原型 | |
| `/research` | 后台 agent 调研，不占主 session | |
| `/improve-codebase-architecture` | 代码库健康巡检，HTML 可视化报告 | 下一步常对候选做 grilling → spec → impl |
| `/handoff` | 把当前工作交给另一个 Agent Session | |
| `/teach` | 生成 HTML 引导课程 | 学开源仓库也可用 |
| `/wait-what` | 把啰嗦回复简化成人话 | |
| `/writing-for-agents` | 写给 Agent 看的文档，减少无效上下文 | |
| `/wayfinder` | 为大项目规划决策 ticket | |
| `/domain-modeling` | 规范事物叫什么、指什么，沉淀在 `CONTEXT.md` | |
| `/triage` | 面向工单的 AI 自动分诊与处理 | 多人协作才更需要 |
| `/retro` | 复盘前面几个 Session，看下次能否做得更好 | 主要针对 Agent，而不是项目 |

## 视频推荐

这个视频覆盖了上文很大一部分内容，推荐看：

[Matt Pocock：十分钟讲完 25 个 Skills | mattpocock/skills v1.2.3（中英字幕）](https://www.bilibili.com/video/BV1zUh56RE8k/?vd_source=32468b8f1b914911548382372c3e1778)
