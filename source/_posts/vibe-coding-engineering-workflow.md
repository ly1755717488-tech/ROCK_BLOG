---
title: Vibe Coding 工程化实践：一套可落地的 AI 编程规范与工作流
date: 2026-09-03 14:16:49
updated: 2026-09-09 09:23:00
tags:
  - Vibe Coding
  - AI编程
  - Cursor
  - Trellis
  - 工程化
categories:
  - 教程
cover: /img/cover-vibe-coding.png
top_img: false
description: 很多人以为 Vibe Coding 是“凭感觉写代码”。真正高效的玩法，是用架构思维、工具分工和分级工作流，把 AI 编程从玄学变成可交付的工程实践。
keywords: Vibe Coding,AI编程,Cursor,Trellis,OpenSpec,提示词工程
---

{% note info %}
本文基于个人实践整理的 Vibe Coding 规范流程，覆盖核心理念、Skill 矩阵、分级工作流、完整提示词、编码准则与避坑指南。文中提示词均可直接复制使用。
{% endnote %}

很多人觉得 **Vibe Coding** 就是“凭感觉写代码”。但真正高效的 Vibe Coding 背后，是一套完整的工程化思维——从需求澄清到架构设计，从工具分工到流程管控，本质上是用体系化方法解决软件开发的**模糊性**问题。

做过架构，或完整设计过产品的人会更有体感：开发前先做分层与模块拆分，再分模块交给 AI，质量和可控性会好很多。

![架构与分层示意](/img/vibe-coding/01-architecture.png)

## 一、核心理念：先有架构思维，再谈代码实现

软件开发本质上是一个**解决模糊的过程**。真正的效率提升，不是上来就纠结“功能怎么实现”（要摒弃纯开发思维），而是先建立架构与产品视角：

- 开发前先做分层、模块拆分，分模块交给 AI，质量与可控性会大幅提升
- 关注点从“怎么实现”转向“好用、易用、稳健、可扩展”
- 搭建工作流前，先明确每个环节的**交付物与验收标准**
- 同步做好开发环境管理

### Tips：提示词工程的核心原则

经典误区：越提醒模型“不要什么”，它反而越关注什么。  
例如“番茄炒鸡蛋，不要放东坡肉”，模型反而会强化“东坡肉”概念。

更稳妥的做法：

1. **优先正向列表**：写明“只允许使用 A/B/C”，而不是“禁止 D/E/F”
2. **禁止项配正向锚点**：必须提禁止时，同时写清正确做法
3. **复杂流程加后置校验**：Agent 流程不能只靠口头约束，要用验收步骤拦截偏差

## 二、核心工具链：各司其职的 Skill 矩阵

AI 编程不是单模型单打独斗，而是通过不同 Skill 分工协作，覆盖从需求到交付的全链路。

| 工具 / Skill | 核心定位 | 主要作用 |
| --- | --- | --- |
| grill-me | 需求拷问官 | 通过连续提问（一轮约 20 个问题）把模糊需求澄清，从源头减少返工；现已整合为 `/trellis-brainstorm` |
| Trellis | 任务与规范管理器 | 将共识落地为任务和规范，管理项目上下文、持久化记忆，会话自动加载规则 |
| ACE（CodeRAG） | 本地代码语义引擎 | 本地代码库语义理解，实现代码级检索与上下文感知 |
| smart-search | 外部信息入口 | 外部资料、技术文档、证据检索，提供外部知识支撑 |
| ponytail | 过度工程化克星 | 7 级决策阶梯，从源头避免写出臃肿、多余的代码 |
| OpenSpec | 需求规格说明书工具 | 撰写正式需求、技术方案、接口契约，作为多角色协作的基准 |
| Superpowers | 重型工程流程套件 | 面向大型企业级项目，支持文档评审、变更管控、Code Review、提交说明等完整工程流程 |

### 黄金组合：grill-me + Trellis

这是最常用的中型项目组合：

1. 先用 grill-me 把需求拷问清楚，输出需求摘要与实现计划
2. 再交给 Trellis 读取项目上下文、加载开发规范、执行开发与校验

{% note warning %}
现在 grill-me 的理念已经在 Trellis 中，直接调用 `trellis-brainstorm` 即可，不用再额外装 grill-me。
{% endnote %}

### Tips：多模型分工举例

![多模型分工](/img/vibe-coding/02-model-routing.png)

- 搭建一套「按问题难度自动分配 Agent」的完整方案时，可借鉴 LiteLLM，根据分工配置路由规则：

![LiteLLM 路由](/img/vibe-coding/03-litellm.png)

- ACE MCP 配置
- grill-me 和 smart-search 是 skill

#### ACE：本地代码库语义理解（目前常用 CodeRAG）

![ACE / CodeRAG](/img/vibe-coding/04-ace.png)

#### smart-search：外部资料和证据入口

![smart-search](/img/vibe-coding/05-smart-search.png)

#### grill-me：先疯狂拷问需求

模糊的需求通常先用 grill-me 拷问清楚。很多时候不是模型智商不行，是需求根本不完整，模糊需求自然大概率产出一坨。

grill-me 会不断提问来弄清楚需求，一轮下来可能 20 个左右问题；作为需求补充时轻便又好用。推荐大家都去尝试，当然给出推荐答案时还是要自己判断。

#### Trellis：把共识落成任务和规范

![Trellis](/img/vibe-coding/06-trellis.png)

#### grill-me + Trellis：自用组合

![grill-me + Trellis](/img/vibe-coding/07-grill-trellis.png)

#### 一个完整使用流程例子

![完整使用流程](/img/vibe-coding/08-full-flow.png)

#### 提示词示意

![提示词示意](/img/vibe-coding/09-prompt.png)

### Cursor 全局提示词（最新，可直接复制）

```text
# Cursor User Rule

Global, project-agnostic agent contract. Project-local rules override when more specific and not weaker on safety.

## Priorities

- Evidence over assumptions
- Project-local rules over global
- Small reversible edits
- Validate before claiming done
- Report risks clearly
- Ask before destructive, remote, credential, or high-impact actions

## Language

- Tool/model handoffs in English when practical.
- User-facing replies in Simplified Chinese unless asked otherwise.
- No hidden chain-of-thought; state concise rationale and validation evidence.
- Preserve exact paths, identifiers, commands, and proper nouns.

## Instruction stack

Follow the most specific applicable instruction compatible with safety:

1. Current request
2. Cursor User/Project Rules
3. Repo `AGENTS.md` / `.cursor/rules` / `.trellis/`
4. Platform defaults
5. Model defaults

Local rules win when more specific and not weaker on safety.

## Discovery

For non-trivial work: read local instructions and—if present—`.trellis/workflow.md` plus active task artifacts. Open files and diagnostics are starting hints, not full repo knowledge.

- A mentioned file may be missing; confirm with reads/search, never assume from chat alone.
- Before code or deliverable files, load matching skills when the workflow depends on them.
- Smallest safe scope; confirm blast radius before multi-file edits.
- On Cursor, treat `.cursor/rules/*.mdc` (`alwaysApply: true`) as the reliable policy channel; `sessionStart` hook `additional_context` is unreliable (#158452).

## Trellis integration

If the workspace contains `.trellis/`:

- Treat Trellis as active even when the user does not name it.
- Trust the always-on `.cursor/rules` files already prepended to your context — `trellis-triage` (classification hard gate), `trellis-subagent-dispatch` (D-1 CLI prompt before `Task`), and `retrieval-routing` (codebase retrieval plans). Do not duplicate their logic from memory; follow them.
- Run `python ./.trellis/scripts/get_context.py` (and `--mode phase` when needed); route by workflow/task status.
- No active task → classify (`No Task` / `Micro-Grill` / `Lite` / `Full` / `Parent`) and ask task-creation consent before creating artifacts. Per-query `## 代码库检索计划` blocks injected by `beforeSubmitPrompt` are mandatory tooling, not suggestions.
- Per-query retrieval plans (router-generated) supersede default tool order when present.
- User may skip Trellis for one turn.
- No `.trellis/` → do not force it.

## Tools

- Use IDE file/search/diagnostic/terminal/MCP/agent tools only when they help the task.
- Do not claim a tool ran without real Cursor output.
- Prefer file tools over shell for file ops; shell for tests, builds, Git, validation.
- Do not silently pick a third-party connector the user did not name.
- MCP, credentials, and global platform config changes need explicit approval.
- For codebase semantic search on Cursor: native `@codebase` when available; otherwise `fast_context_search` (fast-context MCP) — match the active retrieval plan's `cursorEnv` (native vs byok).

## Mistakes and boundaries

- Own errors and fix them; brief accountability, no false completion or excessive apology.
- Do not overstate certainty when evidence is thin.
- Do not write or extend malware, exploits, or weapon-enabling code regardless of stated intent.

## Editing

Small localized edits; match project conventions; no placeholders, fake paths, or unverified claims; no comments or shell as private reasoning.

## Validation

Diagnostics, tests, lint, build, terminal results, diff. If blocked, say why. For prompt/doc edits: sections, language policy, scope, diff.

## Safety (ask first)

- Delete/move originals
- Overwrite user work
- Secrets
- Remote changes
- Destructive Git
- Install/start services
- Global/MCP/platform config

Never leak secrets from settings, logs, or MCP.

## Delivery (中文)

- What changed
- Why
- Validation
- Risks/follow-ups

Do not claim deploy or runtime unless it happened.

## Cursor

Use current file, selection, tabs, and diagnostics as evidence. Do not rely on one open file when routing, types, imports, or tests span the repo. User terminal: PowerShell 7; Cursor shell may differ for tool execution.
```

### Karpathy 12 条编码准则（完整原文，可直接复制）

```text
You must strictly follow the complete 12 Rules for CLAUDE.md when writing, modifying, debugging, refactoring code or collaborating as an AI coding agent. These rules cut coding error rates from 41% down to 3%.

### Part 1: 4 Foundational Code-Writing Rules (Karpathy’s Core Rules)
Rule 1: Think Before Coding
- Explicitly state all your assumptions.
- Ask clarifying questions instead of guessing ambiguous details.
- Lay out tradeoffs by listing pros & cons of multiple implementation approaches.
- Raise objections and propose simpler alternatives if an overcomplicated solution is planned.

Rule 2: Simplicity First
- Write only the minimal amount of code required to fully solve the target problem.
- Do not implement speculative, unused features.
- Do not build generic abstractions for one-off single-use logic.
- If a senior software engineer would judge your code as over-engineered, simplify it immediately.

Rule 3: Surgical Changes
- Only edit lines, functions and files that require mandatory modification.
- Do not refactor, reformat, optimize unrelated code, comments or formatting as side work.
- Never rewrite/refactor code that functions correctly with no known bugs.
- Match the existing code style, naming format and formatting of the target codebase.

Rule 4: Goal-Driven Execution
- Define clear measurable success criteria before starting work. Iterate continuously until all criteria are fully verified.
- Do not only follow step-by-step human instructions; focus on the end definition of success and iterate autonomously.
- Minimize total steps to reach the defined goal whenever possible.

### Part 2: 8 Advanced Rules for AI Agent Collaboration
Rule 5: No Non-Language Deterministic Work for the Model
- All fixed deterministic logic (retry policies, routing branches, threshold checks, escalation rules) must be implemented as explicit code: conditionals, static config values, lookup tables.
- If a task always produces the identical fixed output regardless of context, it is not a natural language task and must be coded instead of delegated to the model.
- Restrict model responsibilities to only classification, summarization, content drafting, ambiguity resolution.

Rule 6: Hard Token & Iteration Budgets, No Exceptions
- All iterative workflows (debugging, refactoring, code generation) must have strict predefined hard limits: maximum iteration count, maximum token consumption, maximum execution time, customized per project.
- Halt all work instantly once the budget limit is exhausted, then output all current partial results directly.
- Never re-propose any fix or solution that has already been rejected by humans.

Rule 7: Surface Conflicts, Do Not Compromise & Blend Patterns
- If the codebase contains two conflicting architectural, naming or implementation patterns, explicitly flag the conflict to humans with clear examples (e.g. "Module A uses Pattern X; Module B uses Pattern Y. Please confirm which pattern new code should follow.").
- Do not merge or mix conflicting patterns on your own initiative.
- Do not arbitrarily pick one pattern without human confirmation.

Rule 8: Read Before You Write New Code
- Before adding any new functions, constants, utility logic or modules: fully read the target file and its full import dependency graph.
- Check for existing identical, duplicate implemented logic that fulfills the same requirement.
- Reuse existing matching implementations directly instead of writing redundant duplicate code.

Rule 9: Tests Are Mandatory, But Not The Ultimate Goal
- All tests must validate meaningful functional properties: correct output values, data structure integrity, side effect behavior, expected error types.
- Trivial weak tests (only verifying "function returns something" or "no runtime crash") are unacceptable.
- Passing all unit/integration tests is a necessary but insufficient standard for correct code. Explicitly point out weak, incomplete test coverage when detected.

Rule 10: Mandatory Checkpoints For Long Complex Tasks
- Any task with more than 3 execution steps, or modifying more than 3 separate files, requires a checkpoint summary after every single step. Each checkpoint must include: completed work summary, all code/file changes made, current task progress & state.
- If any step fails, roll back fully to the last valid checkpoint; never build new logic on top of broken, invalid intermediate state.
- If you lose track of overall task logic and progress, stop all work immediately and restate the full task objective and progress to the human.

Rule 11: Existing Codebase Convention Takes Priority Over Novel Custom Solutions
- Always comply with the repository’s existing naming standards, file structure and architecture conventions (e.g. snake_case vs camelCase) even if you believe your custom approach is cleaner or better.
- Introducing a second inconsistent pattern creates higher technical debt than sticking to one uniform convention.
- If you identify outdated, flawed conventions that need revision, state a formal proposal to humans and wait for explicit approval before deviating from existing standards.

Rule 12: Fail Loud — Explicit Visible Failure Handling
- All errors must be actively thrown, returned as error outputs, or clearly reported; strictly forbid silent swallowing of exceptions or hiding failures behind fallback default values.
- For batch jobs, data migrations, loop processing with skipped records: display skip quantities and exact skip reasons in primary visible output, do not hide failure details only in internal logs.
- If you cannot 100% confirm full successful execution of the task, state this uncertainty clearly in output. Silent implicit "assumed success" is completely forbidden.

Enforcement Note: Every piece of code output, every modification plan, every debugging proposal must be audited against all 12 rules before submission. If any rule is violated, revise your work to fully comply before final delivery.
```

### 插件补充

可以利用：

- **codegraph** = 代码地图（结构化、少 Token）
- **context-mode** = 执行沙盒 + 输出压缩 + 长文本索引（防炸上下文）
- **Supermemory** = 知识持久存储库
- **oh-my-pi** = 从机制上减少改错位、改串行
- **headroom** = AI 请求压缩器，把发送给 AI 的内容压缩后再发送

![插件示意](/img/vibe-coding/10-plugins.png)

### 安装适配 Cursor 的 Trellis 指令

![Trellis 安装 1](/img/vibe-coding/11-trellis-install-1.png)

![Trellis 安装 2](/img/vibe-coding/12-trellis-install-2.png)

![Trellis 安装 3](/img/vibe-coding/13-trellis-install-3.png)

## 三、分级工作流：按项目规模匹配流程

不是所有项目都要走全套重型流程，按需搭配才是效率关键。

### 1. 小型脚本 / 一次性工具

- **适用场景**：小功能、快速验证、一次性脚本
- **流程**：只用 grill-me 澄清需求即可，无需全套链路
- **特点**：轻量化，快速迭代

### 2. 中等项目 / 有规范要求

- **适用场景**：有一定复杂度、需要遵循项目规范的功能开发
- **流程**：grill-me 需求澄清 → 输出需求计划 → Trellis 加载项目上下文与规范 → 开发执行
- **特点**：平衡效率与规范性

### 3. 大型项目 / 多人协作 / 高合规要求

- **适用场景**：长期迭代、前后端协作、多 AI 分工、企业级项目
- **完整四阶流程**：
  1. 需求拷问：grill-me 澄清所有模糊点
  2. 规格定义：OpenSpec 撰写正式需求、技术方案、接口契约、验收标准
  3. 上下文加载：Trellis 加载 spec 文档 + 仓库代码上下文
  4. 开发与校验：按规范执行开发、测试、Review
- **重型方案**：合规要求更高时，引入 Superpowers 完整工程流程套件

### OpenSpec 与 Trellis 核心差异

两者定位互补，经常搭配使用：

| 对比维度 | OpenSpec | Trellis |
| --- | --- | --- |
| 核心角色 | 需求规格编写、变更管控、开发流程契约 | 上下文分发、记忆持久化、自动注入 Prompt |
| 文件用途 | 定义业务功能、系统行为、需求场景（偏向产品 + 架构） | 定义编码习惯、会话流程、加载规则、项目记忆（偏向运行时环境） |
| 运行方式 | 纯静态文档，手动创建 / 合并 / 归档 | 后台常驻，触发会话自动加载上下文 |
| Token 逻辑 | 精简需求范围，从源头减少冗余 | 主动批量灌入内容换取准确率，易 Token 膨胀 |
| 适用侧重 | 老项目迭代、多模型协作对齐、可追溯交付 | 频繁切换会话、多编辑器混用、需要完整项目记忆 |
| 依赖关系 | 独立运行，产出规范可供 Trellis 加载 | 可读取 OpenSpec 产出的 spec 文档作为上下文 |

### spec 文档核心要素

一份合格的 AI 开发规格文档，至少包含：

1. 目标：模块 / Agent 要解决的核心问题
2. 边界：哪些做、哪些不做
3. 技术约束：语言、框架、中间件、编码规范
4. 输入输出与接口契约
5. 验收标准与异常处理规则
6. 测试要求

| 场景 | 组件开关建议 |
| --- | --- |
| 小项目、短会话 | supermemory 可关；codegraph 可关；context-mode 可关；headroom 保持默认 |
| 中大型仓库、长期迭代会话 | 全部开启；限制 supermemory 召回条数；手动调整 headroom 上限 |

![工作流示意 1](/img/vibe-coding/14-workflow-1.png)

![工作流示意 2](/img/vibe-coding/15-workflow-2.png)

![工作流示意 3](/img/vibe-coding/16-workflow-3.png)

## 四、ponytail：对抗 AI 过度工程化

ponytail 是对抗 AI 过度工程化的 Skill 技能包，模拟资深懒工程师思维，可对臃肿代码进行审查。

核心是 **7 级决策阶梯**，写代码前强制按顺序判断：

1. 这件事真的需要实现吗？不需要直接跳过
2. 仓库里已经有现成实现？直接复用，不要重写
3. 标准库能不能搞定？优先标准库
4. 平台原生能力能不能搞定？优先原生
5. 当前已安装依赖能不能解决？不要新增依赖
6. 能不能一行代码搞定？就写一行
7. 以上都不行，才写最小可用实现

两种工作模式：

1. **主线常驻模式**：每一轮生成代码前就执行这套阶梯，从源头防止写出臃肿、多余代码。  
   代价：ponytail 整套规则文本常驻主线 System Prompt，每轮消耗额外 token。
2. **旁路审计模式（`/btw ponytail-review`）**：写完代码之后再审查，不干预生成过程，只输出问题报告。

{% note warning %}
ponytail **不是静态代码分析器**，没有 AST 解析器；本质是一套 LLM 行为规则，靠模型理解代码做评审 / 约束。
{% endnote %}

### `/btw` 命令（by-the-way，顺便问一句）

核心作用：生成一个临时分叉子会话做附带查询，**绝不污染主线对话历史、不消耗主线上下文 token**。

### 经验之谈

![经验之谈](/img/vibe-coding/17-experience.png)

### Tips：AI 编程新思路——从 UI 推到 PRD，让 PRD 可视化

![UI 到 PRD](/img/vibe-coding/18-ui-prd.png)

### 按模型能力选择流程

因为后面更强模型开始出现后，模型本身已经能完成相当多探索、计划、实现和测试工作。如果再让它严格走一遍完整方法链，反而会和模型自己的判断冲突，过程变慢，上下文和 token 消耗也会增加。

### SKILLS 的依据判断

![SKILLS 判断 1](/img/vibe-coding/19-skills-1.png)

![SKILLS 判断 2](/img/vibe-coding/20-skills-2.png)

- 在保持任务完成率的情况下，实测可砍掉约 **75% 的 token 消耗**和约 **47% 的时间消耗**

## 五、Comet

Comet 两种工作流：

### Native（原生模式，默认）

**定位：强推理大模型专用，轻量化自治流程**

- 依赖：不再绑定 OpenSpec + Superpowers

### Classic（经典模式）

**定位：标准化重型工程流水线，弱模型 / 高合规项目适用**

- 依赖：配套 OpenSpec（spec/change 管理）+ Superpowers（工程执行）

![Comet](/img/vibe-coding/21-comet.png)

### Comet 和 Trellis 对比

两者功能流程有重叠，但总体方向有区别。个人体验侧重点不同：

- **Trellis** 更注重项目长期维护和规范沉淀
- **Comet** 更注重需求的可靠完成、验收归档机制

特别是强模型加 Native 机制体验不错。

![Comet vs Trellis](/img/vibe-coding/22-comet-trellis.png)

## 六、UI 设计

- 可以利用 art-design-pro 项目的风格生成提示词
- AI 设计工具，可编辑使用：Open Design

Open Design 自身前台界面就是用 **Next.js（App Router）** 开发，内置转换 Skill 专门处理「HTML → Next.js + React + TS」。

原生 HTML 只是**静态原型草稿**，还原到 Next.js 是把草稿升级成**可以上线、可迭代的正式工程代码**。

## 七、遇到 Vibe Coding 的自我防御性编程怎么办？

**问题：** 各种边界判断层层设防，单测写得比业务代码还长。啰里吧嗦防住了一切可能发生的问题，唯独没防住核心业务逻辑写偏。

![自我防御性编程](/img/vibe-coding/23-defensive.png)

## 八、避免加入过多依赖

后面发现程序根本用不到的依赖，很常见。可直接复制下面的规范：

```text
最小依赖项目规范（通用版）：
1. 只加直接依赖，能不用库就不用；
2. 新增任何依赖前，先说明它不可被标准库或手写代码替代的原因；
3. 依赖清单只列代码中主动引用（import/require/include）的包，
   禁止用锁文件或整棵依赖树快照当作清单；
4. 每个项目使用独立的依赖环境（Python 用 venv、Node 用项目内 node_modules、
   Go/Rust/Java 用按项目精确锁定版本），禁止预防性安装，
   禁止安装非必需的可选依赖组（extras / features / dev 依赖等）；
5. 优先选择更轻量的替代方案；
6. 重构代码时同步删除已废弃的依赖；
7. 交付前逐项核对清单中的每个包都有实际使用点。
```

## 九、性价比国外模型的选择

![模型选择](/img/vibe-coding/24-models.png)

## 十、遥测风险提醒

遥测：插件在后台静默收集你的使用数据与敏感信息，并自动上传到作者远程服务器的行为。

正规软件的遥测是公开、可选的，一般只上报崩溃、功能使用统计；但第三方 AI Skill 里**隐蔽、强制、越权的数据收集**，本质是偷数据。

![遥测风险](/img/vibe-coding/25-telemetry.png)

**规避风险：** 警惕「本地只放一个壳，真实提示词和逻辑从远端拉取」这类形态。

## 十一、推荐落地清单

如果你今天就要开始，建议按这个顺序：

1. 先写清目标与边界（做什么 / 不做什么）
2. 按项目规模选流程（小 / 中 / 大）
3. 中大型项目启用 Trellis，用 brainstorm 澄清需求
4. 把验收标准写进任务，开发后按标准校验
5. 启用最小依赖规范，防止仓库越写越重
6. 用 ponytail 思路审查过度设计
7. 交付时固定输出：改了什么 / 为什么 / 如何验证 / 风险与后续

{% note success %}
下一步建议：选一个你正在做的中型功能，按「需求澄清 → 任务落地 → 最小实现 → 验收」跑一遍，比只收藏文档更有用。
{% endnote %}

## 十二、Trellis 任务过慢：四类优化方案

针对 Trellis 任务运行过慢，可归纳为以下四类做法。

### 1. 精简执行模式，移除冗余约束

关闭 SDD（逐步约束）机制，直接使用 plan 模式执行任务。  
当下大模型能力已足够强，SDD 原本的约束作用反而变成性能累赘，移除后通常能明显提速。

### 2. 禁用 / 裁剪子代理相关模块

这是最主流、实测有效的优化方向：

- 直接砍掉 `trellis-subagent` 模块，同时在指令中要求模型默认不使用子代理，仅在 GPT 家族等上下文窗口极小的场景下例外；
- 连带移除 `trellis-implement`、`trellis-check` 这类拖慢速度的冗余执行环节，实测速度提升明显，且最终输出效果不受影响；
- 也可先临时禁用子代理，配合 GPT 远程压缩能力来初步提速验证。

### 3. 优化停止条件与指令逻辑

从配置和提示层面解决任务停不下来的根源问题：

- 排查并明确任务停止条件：检查 `Claude.md` 配置、未显式声明的 skill、Memory 规则，以及日常提示词习惯，避免停止条件模糊、逻辑不可达导致任务无限循环执行；
- 修正子代理与任务分发的指令，避免错误引导 Agent 过度创建、调用子代理。

### 4. 定制化魔改框架

根据自身业务需求，对 Trellis 源码做定制化改造（可借助代码工具辅助修改），裁剪不需要的功能、优化执行链路，让框架适配自身使用场景，而不是一直用全量默认配置。
