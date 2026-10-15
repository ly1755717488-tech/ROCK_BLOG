---
title: 随笔
date: 2026-09-02 15:30:00
updated: 2026-10-15 09:38:34
type: "notes"
comments: false
---

<div class="notes-board">

<article class="note-card note-card--essay">
  <header class="note-card__head">
    <span class="note-card__badge">随笔</span>
    <time class="note-card__time">2026-10-15</time>
  </header>
  <h2 class="note-card__title">模型体感：Astra Pro 与 Opus 5.5 Max，还有抽卡</h2>
  <div class="note-card__body">
    <p>这是别人的经验：最近在打算法比赛，聊到模型时谈到——Astra Pro 和 Opus 5.5 Max 属于同一档，但前者更擅长突破，后者更擅长分析。</p>
    <p>用下来的一点体会：给 Astra Pro 的提示词越少越好，给多了反而限制它的发挥。其次，本质上是抽卡——可能抽 10 次，才有 1 次的 Pro 特别聪明。</p>
  </div>
</article>

<article class="note-card note-card--essay">
  <header class="note-card__head">
    <span class="note-card__badge">随笔</span>
    <time class="note-card__time">2026-10-15</time>
  </header>
  <h2 class="note-card__title">Claude 拆任务、Codex 写代码：一条 shell 编排链路</h2>
  <div class="note-card__body">
    <p>GPT 的代码审查能力暂时还没有 Claude 强，所以可以把两者拼起来用：让 Claude 负责想和拆，让 Codex 负责写。</p>
    <h3>架构链路</h3>
    <p>用户需求 → Claude 思考、拆解任务 → 输出 shell 指令 → 本地执行 shell → 调用 Codex → Codex 生成代码</p>
    <p>串起来的关键是中间那层 shell：Claude 的产出不是直接代码，而是一串可执行的指令，由本地 shell 落下去跑，再把写码这一步交给 Codex。规划和生成各用各自更擅长的模型，链路本身走本地命令，衔接清楚也好排查。</p>
  </div>
</article>

<article class="note-card note-card--essay">
  <header class="note-card__head">
    <span class="note-card__badge">随笔</span>
    <time class="note-card__time">2026-10-14</time>
  </header>
  <h2 class="note-card__title">支付后积分 / 虚拟产品怎么保证到账</h2>
  <div class="note-card__body">
    <p>核心思路是最终一致性：状态机 + 消息队列 + 幂等 + 补偿重试。</p>
    <ol>
      <li><strong>回调只落库</strong>：收到微信支付回调、校验成功后，先在数据库事务里把订单更新为已支付，并在同一事务内发出积分发放消息（MQ）。</li>
      <li><strong>异步发货 + 幂等</strong>：消费 MQ 再执行发积分 / 发虚拟商品；消费端做好幂等，防止重复回调导致重复发放。</li>
      <li><strong>失败补偿</strong>：消息消费失败进死信队列，由定时任务补偿重试。</li>
      <li><strong>每日对账</strong>：定时对账任务比对本地订单和微信账单，查漏补缺，补发缺失的积分。</li>
    </ol>
    <p>关键约束：不能在回调接口里同步执行复杂业务，超时会导致微信反复回调。</p>
  </div>
</article>

<article class="note-card note-card--essay">
  <header class="note-card__head">
    <span class="note-card__badge">随笔</span>
    <time class="note-card__time">2026-10-12</time>
  </header>
  <h2 class="note-card__title">Harness：给大模型套上的那根缰绳</h2>
  <div class="note-card__body">
    <p>这里说的是 Agent 大模型方向的 Harness Engineering，不是 DevOps 那个 Harness 平台。一句话公式：<strong>Agent = Model + Harness</strong>。</p>
    <p>Harness 直译是马具、缰绳。大模型本身是一匹能力很强但不受控的马，Harness 就是包裹在大模型之外、让 Agent 稳定安全可控运行的整套运行时基础设施，不属于模型本身。</p>
    <h3>包含的核心组件</h3>
    <ol>
      <li><strong>状态管理</strong>：保存对话上下文、任务状态、子任务进度，LangGraph 里的 State 就是 Harness 的一部分。</li>
      <li><strong>工具调度层</strong>：工具调用、参数校验、工具执行、结果解析，控制模型能调用哪些工具。</li>
      <li><strong>Guardrails（护栏）</strong>：输入输出校验，过滤违规内容、做权限控制，限制模型不能做的操作（禁止删库、高危操作），Allow / Deny / Ask 策略。</li>
      <li><strong>上下文管理 Resolver</strong>：按需加载记忆、RAG 上下文，做上下文压缩，防止 prompt 超长溢出。</li>
      <li><strong>Hook 钩子</strong>：在模型调用前 / 调用后拦截，做日志、埋点、校验、限流、缓存。</li>
      <li><strong>Agent 循环控制</strong>：限制最大迭代次数，防止无限循环，并判断终止条件。</li>
      <li><strong>观测与成本控制</strong>：Token 统计、耗时监控、重试策略、异常兜底。</li>
    </ol>
    <p>核心目标是约束大模型，解决 Agent 落地的痛点：无限循环、上下文爆炸、越权操作、不可控输出、成本不可控。</p>
    <p>通俗理解：模型负责思考推理，Harness 负责管控、调度、安全、资源、流程，给模型套上缰绳。</p>
  </div>
</article>

<article class="note-card note-card--essay">
  <header class="note-card__head">
    <span class="note-card__badge">随笔</span>
    <time class="note-card__time">2026-10-12</time>
  </header>
  <h2 class="note-card__title">混合检索都分组排序去重了，为什么还要再精排</h2>
  <div class="note-card__body">
    <ol>
      <li><strong>召回阶段追求高召回率，牺牲精度</strong>：BM25 和向量检索是粗筛，目标是尽量把相关文档捞出来，宁可多召回不能漏，允许混入一些相关性弱的文档。粗排打分逻辑比较简单，对语义的深层理解不足。</li>
      <li><strong>粗排是两个不同的分数体系</strong>：向量相似度和 BM25 关键词分数是两套独立逻辑，融合只能做简单加权或 RRF，很难精细判断语义相关性。</li>
      <li><strong>Re-rank 模型（交叉编码器 Cross-Encoder）能力更强</strong>：精排的输入是「问题 + 单条文档」成对输入，深度判断文档和用户问题的真实语义匹配程度，比 Embedding 模型理解更强，能精准区分「看起来像、但无关」的文档。</li>
      <li><strong>资源成本可控</strong>：召回阶段一次性召回几十条，数量少；精排只对这几十条跑，开销可控，不用全库计算。</li>
    </ol>
    <p>一句话：粗召回负责「不漏掉相关文档」，Re-rank 精排负责「挑出最相关的文档」，提升最终答案质量。</p>
  </div>
</article>

<article class="note-card note-card--essay">
  <header class="note-card__head">
    <span class="note-card__badge">随笔</span>
    <time class="note-card__time">2026-10-12</time>
  </header>
  <h2 class="note-card__title">PostgreSQL 物化视图：把查询结果存成一张表</h2>
  <div class="note-card__body">
    <h3>先分清两种视图</h3>
    <p><strong>普通视图</strong>：像一个「查询快捷方式」。比如你经常要算「每个部门有多少人」，每次都得写很长的 SQL 去统计。普通视图就是把这段 SQL 存起来，你查视图时数据库才临时算一遍，把结果给你。它本身不存数据，只存公式。</p>
    <p><strong>物化视图</strong>：它不是快捷方式，而是把查询结果实实在在存成一张真实的表格。相当于提前把「每个部门人数」算好，把结果表存在硬盘里；以后查它直接读存好的结果，不用再算一遍。</p>
    <h3>特点</h3>
    <p>数据不是实时更新的。部门来了新员工，物化视图里的数字不会自动变，得手动「刷新」一下，它才会重新算一遍并保存最新结果。</p>
    <h3>什么时候用</h3>
    <p>那些经常查、计算又很慢、且数据不用实时更新的统计报表，用它速度特别快，属于「用存储空间换查询时间」。</p>
  </div>
</article>

<article class="note-card note-card--essay">
  <header class="note-card__head">
    <span class="note-card__badge">随笔</span>
    <time class="note-card__time">2026-10-12</time>
  </header>
  <h2 class="note-card__title">消息队列怎么保证任务不丢、任务一致</h2>
  <div class="note-card__body">
    <p>从生产端、存储端、消费端三个环节全链路保障。</p>
    <h3>生产端：保证消息成功入队</h3>
    <ul>
      <li><strong>生产端确认机制</strong>：发送后等待队列 ACK 响应，失败则重试；支持事务消息或半消息机制。</li>
      <li><strong>落库兜底</strong>：先把任务写入数据库，再异步投递，定时扫描补偿未入队的消息。</li>
    </ul>
    <h3>存储端：保证消息持久化不丢</h3>
    <ul>
      <li>开启队列 / 中间件的持久化机制，如 Redis RDB + AOF、RabbitMQ 持久化、Kafka 副本机制。</li>
      <li>集群部署、多副本冗余，避免单点故障导致数据丢失。</li>
    </ul>
    <h3>消费端：保证任务被成功执行</h3>
    <ul>
      <li><strong>手动 ACK</strong>：业务逻辑执行完成后再确认消费，失败则不 ACK，超时自动重入队。</li>
      <li><strong>幂等设计</strong>：任务携带唯一 ID，消费前校验是否已执行，避免重复执行导致数据不一致。</li>
      <li><strong>死信队列</strong>：多次重试失败的任务进入死信队列，人工兜底处理，不阻塞正常流程。</li>
    </ul>
  </div>
</article>

<article class="note-card note-card--essay">
  <header class="note-card__head">
    <span class="note-card__badge">随笔</span>
    <time class="note-card__time">2026-10-12</time>
  </header>
  <h2 class="note-card__title">后端几个常见机制：原子操作、行级锁、幂等与验签</h2>
  <div class="note-card__body">
    <h3>Redis Lua 脚本：多步操作打包成原子操作</h3>
    <p>需要把多步 Redis 操作打包成「原子操作」、中间不能被别的请求打断时，就用 Lua 脚本。Lua 是一种轻量小巧的脚本语言，常被嵌入到其他软件里运行。放在 Redis 里用的核心价值就是原子性：一整段脚本一口气执行完，中间不会被其他命令打断，要么全成功，要么全失败。</p>
    <h3>MySQL 行级锁</h3>
    <p>行级锁是 MySQL 里的细粒度锁，只锁住你当前要修改的那一行数据，其他行完全不受影响。</p>
    <h3>幂等怎么实现</h3>
    <p>最通用的是「唯一 ID + 去重表」：给每次操作生成一个全局唯一流水号（比如支付订单号），操作前先查去重表，已存在就直接返回成功，不存在则执行业务并写入去重表。支付、下单这类所有新增操作都适用。</p>
    <h3>支付回调验签流程</h3>
    <p>验签是支付安全的核心，用来防止伪造回调。</p>
    <ol>
      <li><strong>收报文</strong>：接收微信支付回调的全部参数和签名字段。</li>
      <li><strong>排参数</strong>：剔除签名字段，剩余参数按名称 ASCII 码从小到大排序。</li>
      <li><strong>拼字符串</strong>：排序后按键值拼接，末尾拼上商户密钥。</li>
      <li><strong>算签名</strong>：用约定算法（MD5 / HMAC-SHA256）计算本地签名。</li>
      <li><strong>双校验</strong>：先比对签名是否一致，再校验订单号、金额和本地一致，全部通过才处理业务。</li>
    </ol>
  </div>
</article>

<article class="note-card note-card--essay">
  <header class="note-card__head">
    <span class="note-card__badge">随笔</span>
    <time class="note-card__time">2026-10-12</time>
  </header>
  <h2 class="note-card__title">AI 输出如何保证结构化，以及为什么用 Markdown 而不是 JSON</h2>
  <div class="note-card__body">
    <h3>如何保证结构化</h3>
    <ol>
      <li><strong>Prompt 强约束</strong>：系统 Prompt 里明确规定输出格式、结构、层级，要求必须按 Markdown 语法输出，禁止自由格式。</li>
      <li><strong>输出校验</strong>：输出后做格式校验，检查标题、列表、表格等结构是否完整，不符合就自动重生成。</li>
      <li><strong>结构化解析</strong>：前端用 Markdown 解析器统一渲染，保证不同模型、不同场景的展示样式一致。</li>
      <li><strong>关键信息抽取</strong>：需要业务字段的场景，额外加一个信息抽取环节，把关键信息提取成结构化 JSON，供业务系统流转。</li>
    </ol>
    <h3>为什么选 Markdown 不选 JSON</h3>
    <ol>
      <li><strong>大模型友好</strong>：Markdown 是自然语言的结构化，大模型生成准确率远高于 JSON，不容易出现语法错误、字段缺失。</li>
      <li><strong>可读性强</strong>：生成过程中可预览、可调试，人工审核成本低；JSON 的嵌套结构肉眼难读。</li>
      <li><strong>渲染兼容</strong>：前端直接渲染成富文本，支持标题、列表、表格、代码块；JSON 要自定义映射才能渲染。</li>
      <li><strong>容错性高</strong>：Markdown 的小瑕疵不影响阅读；JSON 多一个逗号、少一个引号就解析失败。</li>
      <li><strong>适用场景广</strong>：问答、文档、教程这类内容天然适合 Markdown；只有纯机器对接的接口才用 JSON。</li>
    </ol>
  </div>
</article>

<article class="note-card note-card--essay">
  <header class="note-card__head">
    <span class="note-card__badge">随笔</span>
    <time class="note-card__time">2026-10-12</time>
  </header>
  <h2 class="note-card__title">SSE 与 WebSocket：单向推和双向通</h2>
  <div class="note-card__body">
    <h3>SSE（Server-Sent Events）</h3>
    <ul>
      <li>底层是普通 HTTP 长连接，单向通信：只能服务器推数据给客户端；客户端要发消息得单独再发一个 HTTP 请求。</li>
      <li>浏览器自带断线自动重连，只能传文本。</li>
      <li>适合：AI 流式输出、消息通知、行情推送——只需要后端不断推、客户端不用频繁发消息的场景。</li>
    </ul>
    <h3>WebSocket</h3>
    <ul>
      <li>独立协议，由 HTTP 握手升级而来；全双工双向通信，客户端和服务端可以随时互相发消息。</li>
      <li>支持文本和二进制数据；断线重连、心跳检测要自己手写。</li>
      <li>适合：在线聊天、协同文档、在线游戏——双向高频实时交互的场景。</li>
    </ul>
    <p>一句话区分：只需要服务器单向推消息用 SSE；双方都要随时互相发消息，用 WebSocket。</p>
    <h3>为什么 WebSocket 失败要降级成轮询</h3>
    <p>轮询不会跟着一起失败，因为 WebSocket 连不上不等于网络不通。长连接走不通，短请求照样能发；这时候降级成轮询，数据照样更新，只是实时性差一点。</p>
    <ul>
      <li>若是完全断网、服务器宕机、域名解析失败，WebSocket 和 HTTP 会一起失效，轮询也没用，直接提示网络异常即可。</li>
      <li>但八成以上的连接异常是「长连接保不住、协议不支持」这类问题，网没断，降级轮询就能兜底。</li>
    </ul>
    <p>降级的本质是分层兜底：不追求 100% 可用，而是按优先级层层退让。</p>
    <ol>
      <li>最优：WebSocket 长连接，实时性最好、最省资源。</li>
      <li>兜底：HTTP 长轮询，实时性稍差，但兼容性最强，绝大多数网络环境都能用。</li>
      <li>最终兜底：都不行就提示网络异常，并给一个手动重试按钮。</li>
    </ol>
    <h3>消息边界：到底谁负责切分</h3>
    <p>SSE 常被说成「有边界」，但那个 `\n\n` 只是字节流里的几个字符，不是协议层面给的边界。往下几层看就清楚了：</p>
    <table>
      <thead>
        <tr><th>层</th><th>边界</th><th>应用要不要自己切</th></tr>
      </thead>
      <tbody>
        <tr><td>TCP</td><td>无，只有有序字节流</td><td>必须</td></tr>
        <tr><td>HTTP/2 DATA 帧</td><td>有帧边界，但 body 对应用仍是字节流</td><td>必须</td></tr>
        <tr><td><strong>SSE</strong></td><td>自定义 <code>\n\n</code> 分界——但它只是字节流里的字符</td><td>必须</td></tr>
        <tr><td>WebSocket</td><td>有真正的帧 / 消息边界，整条消息交付</td><td>不用</td></tr>
      </tbody>
    </table>
    <p>记三句：SSE 是「话怎么说」，HTTP 是「信封怎么写」，TCP 是「快递怎么把东西可靠送到」。三层各管一段，谁也不替谁划消息边界。</p>
  </div>
</article>

<article class="note-card note-card--essay">
  <header class="note-card__head">
    <span class="note-card__badge">随笔</span>
    <time class="note-card__time">2026-10-11</time>
  </header>
  <h2 class="note-card__title">Claude Code 的核心循环：TAOR</h2>
  <div class="note-card__body">
    <p>Claude Code 干活的主线是一个 Agent 闭环，缩写叫 TAOR：Think → Act → Observe → Repeat。四步一轮，转到达标为止。</p>
    <h3>四步拆开看</h3>
    <ol>
      <li><strong>Think（思考）</strong>：模型拿到当前上下文，理解任务目标，规划下一步动作，从可用工具里挑一个（读文件、编辑代码、跑 shell、执行测试等），输出工具调用指令。</li>
      <li><strong>Act（执行动作）</strong>：后端沙箱执行选中的工具，比如读取源码、改文件、跑单元测试。所有代码和命令都在隔离沙箱里跑。</li>
      <li><strong>Observe（观测结果）</strong>：捕获工具返回的输出，代码内容、报错、stdout 日志，统统塞回上下文。</li>
      <li><strong>Repeat（判断循环）</strong>：模型看着观测结果，判断任务做完没有。没做完就回到 Think 再走一轮；满足退出条件，收工。</li>
    </ol>
    <p>拆到底就是：模型只管想和决策，工具在沙箱里干活，结果回到上下文，再判断要不要继续。所谓 Agent 能力，大半是把这套循环的每一步做扎实。</p>
  </div>
</article>

<article class="note-card note-card--essay">
  <header class="note-card__head">
    <span class="note-card__badge">随笔</span>
    <time class="note-card__time">2026-10-06</time>
  </header>
  <h2 class="note-card__title">CC 规划、CX 写码：体感差在 Harness，不在模型</h2>
  <div class="note-card__body">
    <p>Claude Code（CC）偏规划、前端、全栈；Codex（CX）偏审代码、后端。底层、C/C++、编译器、解码器，两边都不行。</p>
    <p>这些体感差异，绝大多数来自两家官方 Harness（智能体运行时编排）的设计，模型本身的能力倾向只占一小部分。模型是大脑，Harness 是神经、骨骼和工作制度：它决定流程、工具、权限边界、任务怎么分、输出怎么校验。</p>
    <p>行业里有句共识：Harness 才是产品力，模型只是原材料。Anthropic 偏向灵活、高效、开发者主导、多模型协同；OpenAI 偏向规范、安全、一致性优先、单模型全链路。</p>
    <h3>节奏、成本和排版</h3>
    <p>CC 走 Claude API，单价贵，但适合规划和拆问题。真正写代码交给 Codex，整体花不了多少钱。CC 能让你保持很快的节奏去思考、推进；Codex 别的都还行，就是想太久，排版也不太利于沟通。</p>
    <p>快慢很大一块是调度策略。CC 内置任务复杂度分类和模型路由：读文件、摘要、简单检索走 Haiku（不管你选的是 Sonnet 还是 Opus 4.5），架构规划、深度分析才用自选大模型，子任务还能并行派发。CX 是单主模型串行流水线，思考、写码、工具、自检都绑在同一个模型上，再加多轮自检和思维链展开，优先保证准确性，等待就长。</p>
    <h3>权限和并行</h3>
    <p>两边最大的差别是权限。CC 的运行时动态审批更灵活，系统调用、抓包、软件深度分析这类放行阈值低，偏向开发者全权主导，商业分析更对口。CX 是显式策略引擎加强沙箱，文件修改、网络访问每一步都要过策略，很多操作必须人工确认，绕不过去。CX 这边得绕半天，还没法做到百分之百抓数据分析。</p>
    <p>如果说「去掉权限，Claude API 和 gpt-5.x-codex API 基本没区别」——更准确的说法是：不用官方客户端、直接调原始模型 API，差异会缩小到文风和能力倾向；只要用官方客户端，Harness 带来的流程、权限、编排差异依然很大。</p>
    <h3>代码质量和推荐用法</h3>
    <p>CC 规划强，既有 Claude 擅长长文本推理、架构拆解的一面，也有 Harness 强制「先规划后执行」：先出方案、拆步骤，再动手。CX 代码更规范、Bug 更少，既有 GPT 在语法和工程规范上的积累，也有写完之后自动 lint、格式、静态检查这道质量门禁。</p>
    <p>比较好用的办法：用 CC 做规划、分析，再指导 CX 写代码并直接执行。本质是两套 Harness 互补——CC 吃快速规划和多模型调度，CX 吃规范输出和质量校验。成本划算，能力也对得上，比单工具走完全程更高效。</p>
  </div>
</article>

<article class="note-card note-card--essay">
  <header class="note-card__head">
    <span class="note-card__badge">随笔</span>
    <time class="note-card__time">2026-09-30</time>
  </header>
  <h2 class="note-card__title">理解机制，就懂这些 Prompt 怪现象</h2>
  <div class="note-card__body">
    <p>如果理解注意力机制，就不难理解：指令后面加一堆感叹号起不到强调作用，反而会稀释注意力；prompt 重复两遍可以提升性能，重复五遍却会降低表现。</p>
    <h3>注意力机制</h3>
    <p>大模型读文字时会分配「关注度」，优先关注重要 token。</p>
    <ul>
      <li>一堆 <code>！！！！！</code> 会占用注意力资源，把真正的核心指令冲淡，模型分不清重点；</li>
      <li>重复 2 遍 prompt：相当于提醒模型这件事很重要，效果变好；</li>
      <li>重复 5 遍：冗余太多，注意力被反复消耗，反而变差。</li>
    </ul>
    <h3>Tokenizer 分词器</h3>
    <p>如果理解了 Tokenizer 原理，看到模型算出 <code>9.11 &gt; 9.9</code>、或说 <code>strawberry</code> 里只有 2 个 r，就不会大惊小怪。</p>
    <p>模型不是按人类理解的「单词 / 数字」阅读，而是切成一个个小块（token）。</p>
    <ol>
      <li><code>9.11</code> 会被切为 <code>9</code>、<code>.</code>、<code>11</code>；<code>9.9</code> 切为 <code>9</code>、<code>.</code>、<code>9</code>。模型是对 token 编码做运算，不是做数学计算，所以会出现人类看来很蠢的大小比较错误。</li>
      <li><code>strawberry</code> 分词时可能被切成片段，模型不是直接「读取全部字母」，所以数 r 会数错。</li>
    </ol>
    <p>简单说：模型不是认字，是认碎片。</p>
    <h3>KV 缓存（KV 矩阵）</h3>
    <p>如果知道每次预测都要综合计算前面所有上下文的 KV 矩阵，就能明白上下文管理有多重要，也明白：模型一旦拒绝要求，继续说服往往没用。</p>
    <p>模型每一轮回答，都会把上文信息编码保存。</p>
    <ul>
      <li>上下文越长，KV 占用越大；前面已经判定拒绝（安全判定），这个结论已经写入上下文 KV。后面再劝说、软磨硬泡，模型依然带着前面的判定，基本不会改主意。</li>
      <li>所以长对话要精简历史、清理无效旧对话，这就是上下文管理。</li>
    </ul>
    <h3>过拟合与思考标记</h3>
    <p>如果了解过拟合，就更容易理解：为什么强行塞入思考标记，有时反而会得到更随机的回复。</p>
    <p>过拟合指模型训练时死记硬背样本，泛化变弱。<code>&lt;think&gt;</code> 是 DeepSeek 这类推理模型的内部思考标记。当你强制写入它，会触发进入推理模式，这种模式下更容易发散、不稳定，随机性更强。</p>
    <h3>语义激活</h3>
    <p>如果理解语义激活，就能明白：prompt 里用正面要求，往往比负向写法效果更好。</p>
    <p>模型根据词语关联激活对应语义。</p>
    <ul>
      <li>负面写法：「不要犯错、不要编造、不要遗漏」——会先激活「犯错、编造、遗漏」这些词的语义，反而更容易踩坑；</li>
      <li>正面写法：「请严谨、引用原文、逐条输出」——直接激活你想要的行为，效果更好。</li>
    </ul>
  </div>
</article>

<article class="note-card note-card--essay">
  <header class="note-card__head">
    <span class="note-card__badge">随笔</span>
    <time class="note-card__time">2026-09-30</time>
  </header>
  <h2 class="note-card__title">跟 AI 协作摸到的四个小技巧</h2>
  <div class="note-card__body">
    <p>这次做项目又攒了几条亲测有用的经验，记下来。</p>
    <h3>1. 修不好就引导，结尾加「第一性原理」</h3>
    <p>Bug 老是修不好时，别干骂。好好跟它聊，帮它理思路，但最后一定要补一句：</p>
    <p><strong>请你从第一性原理的角度出发，捋清楚刚才的问题症结为我修复它。</strong></p>
    <p>这句话真有点魔法，像按了重置键，把它从混乱里拉回来，逼它从打地基开始重新看整栋楼的结构。这样给出来的方案通常会好很多。好好说话总比骂人管用，对 AI 也成立。</p>
    <h3>2. 收尾做对抗性检查</h3>
    <p>项目收尾时，很多问题自己日常用不到、测不出来，除非经验特别丰富。AI 往往比你见过更多坑——直接让它对这个项目做<strong>对抗性检查</strong>，专门用来拆台。它真能挖出一些设计上的逻辑错误。</p>
    <h3>3. 前端难搞就让它接管浏览器</h3>
    <p>前端问题不好解决时，多让它接管浏览器做开发调试。很多时候是你描述不准、观察不细，或者不太熟那些专业名词，开发效果就会打折扣。直接要求它接管浏览器，边看边调。</p>
    <p>反重力的浏览器调试很好用，腾讯出品的 BrowseSkills 也不错，还有真神 Codex。用哪个丰俭由人。</p>
    <h3>4. 交货前用动态执行验脚本，别只看静态检查</h3>
    <p>要求模型交货前写脚本多做测试。文件一庞大，AI 往往只能审查局部——单看每段都正确无误，上下组合起来就错了。重构时尤其危险：任何重构都不能只靠静态语法检查，必须靠<strong>动态沙箱执行</strong>验证运行时引用完整性。</p>
    <p>该死的 Gemini 就给我犯过三四次这种低级错误：修 Bug 修着修着，把脚本注入页面整段弄丢了。</p>
  </div>
</article>

<article class="note-card note-card--essay">
  <header class="note-card__head">
    <span class="note-card__badge">随笔</span>
    <time class="note-card__time">2026-09-18</time>
  </header>
  <h2 class="note-card__title">打野节点思路</h2>
  <div class="note-card__body">
    <p>访问：<code>https://gist.github.com</code></p>
    <p>搜索常用的节点协议链接：<code>ss://</code>、<code>vless://</code> 等，选择 <code>Recently updated</code>，如图所示：</p>
    <p><img src="https://cdn3.ldstatic.com/original/4X/f/5/e/f5e386ebb5abff092c3c2b34e3649bce552feb01.png" alt="Gist 搜索示意"></p>
    <h3>GitHub 中 Gist 的作用</h3>
    <p>你写了一小段代码、一段文字、一个 JSON 配置，不想专门新建一个完整项目仓库，只想快速存起来，生成一个链接发给别人看，这就用 Gist。</p>
    <h3>Raw 链接</h3>
    <p>就是 Gist 里面文件的「纯原始文本直链」，不带网页 UI、没有按钮、没有广告，浏览器打开直接就是文件本身的内容，方便程序 / 脚本读取。</p>
    <h3>功能：订阅节点分发</h3>
    <p>写个脚本，直接通过 git 就可以<strong>定时把最新节点列表自动上传更新到 Gist</strong>。【节点经常会失效、掉线，需要替换新节点】，Gist 会给你一个固定不变的 raw 原始链接。</p>
  </div>
</article>

<article class="note-card note-card--essay">
  <header class="note-card__head">
    <span class="note-card__badge">随笔</span>
    <time class="note-card__time">2026-09-16</time>
  </header>
  <h2 class="note-card__title">提交前门禁：Gitleaks 与 SonarQube</h2>
  <div class="note-card__body">
    <ul>
      <li><strong>pre-commit 钩子</strong>：Git 的内置机制，在代码提交（commit）之前会自动触发预设脚本，用来做提交前检查。</li>
      <li><strong>Gitleaks</strong>：专门的敏感信息扫描工具，检测代码里有没有硬编码的密钥、密码、Token、API Key 等。</li>
      <li><strong>组合作用</strong>：每次提交代码前自动跑 Gitleaks 扫描，一旦发现 AI 生成的代码里夹带了敏感信息，直接阻止提交，从源头防止密钥泄露到代码库。</li>
    </ul>
    <p><strong>SonarQube</strong> 是面向代码质量与安全的静态分析平台，支持 40+ 编程语言。核心价值是对所有代码（人工编写 / AI 生成）执行一致的自动化校验，拦截 Bug、安全漏洞、代码坏味道，从源头避免技术债务累积。</p>
  </div>
</article>

<article class="note-card note-card--memo">
  <header class="note-card__head">
    <span class="note-card__badge">小记</span>
    <time class="note-card__time">2026-09-15</time>
  </header>
  <h2 class="note-card__title">大模型选型逻辑</h2>
  <div class="note-card__body">
    <p class="note-card__lead">从业务约束出发，优先级：<strong>能力需求 → 成本 → 延迟 → 隐私合规 → 运维负担</strong></p>
  </div>
</article>

<article class="note-card note-card--essay">
  <header class="note-card__head">
    <span class="note-card__badge">随笔</span>
    <time class="note-card__time">2026-09-15</time>
  </header>
  <h2 class="note-card__title">上下文压缩：弱模型压缩质量不行怎么办</h2>
  <div class="note-card__body">
    <p>上下文压缩本质是：保留关键信息，删减冗余。模型能力不足时，不要完全依赖大模型做压缩，走<strong>混合方案</strong>：</p>
    <ol>
      <li>
        <p><strong>非 LLM 前置压缩（优先）</strong></p>
        <ul>
          <li>规则 / 检索：按时间过滤、去重、移除无关日志、固定模板；</li>
          <li>关键词抽取、摘要提取用轻量模型，甚至传统 NLP；</li>
          <li>按重要性打分，直接丢弃低权重片段，不交给大模型。</li>
        </ul>
      </li>
      <li>
        <p><strong>拆分压缩任务，不要一次性压缩超长文本</strong></p>
        <p>分块局部摘要，再合并局部摘要，避免一次性超长输入压垮弱模型。</p>
      </li>
      <li>
        <p><strong>改变压缩策略：不做全文摘要，做信息抽取</strong></p>
        <p>不要求模型写总结，改为结构化抽取：<code>问题、关键结论、待办、冲突点</code>，结构化输出更容易稳住质量。</p>
      </li>
      <li>
        <p><strong>增加校验层</strong></p>
        <p>压缩完成后做轻量校验：检查是否丢失关键实体 / 编号；丢失则保留原文片段。</p>
      </li>
      <li>
        <p><strong>兜底策略</strong></p>
        <p>压缩失败 / 丢失关键信息时，回退原始上下文，牺牲 token 换正确性。</p>
      </li>
    </ol>
    <p><strong>核心思路：</strong>把重压缩工作交给检索 + 传统手段，弱模型只做结构化提炼，而不是让弱模型做全文浓缩。</p>
  </div>
</article>

<article class="note-card note-card--essay">
  <header class="note-card__head">
    <span class="note-card__badge">随笔</span>
    <time class="note-card__time">2026-09-15</time>
  </header>
  <h2 class="note-card__title">AI 调用工具失败怎么解决</h2>
  <div class="note-card__body">
    <p>工具调用失败常见类型：格式错误、参数错误、函数幻觉、超时、业务权限报错。分层处理：</p>
    <ol>
      <li>
        <p><strong>第一层：校验拦截（前置）</strong></p>
        <p>用 JSON Schema / Zod 校验模型输出的工具调用参数，格式不对直接判失败，不调用真实函数。</p>
      </li>
      <li>
        <p><strong>第二层：重试机制</strong></p>
        <p>区分可重试错误（网络超时、5xx）和不可重试（参数非法、权限不足）；可重试做有限次数重试 + 退避；参数错误不要重试。</p>
      </li>
      <li>
        <p><strong>第三层：错误回传给大模型，让模型自修正</strong></p>
        <p>把 error message、错误原因返回模型，让它重新生成工具调用；加提示约束：不要编造不存在的参数 / 函数。</p>
      </li>
      <li>
        <p><strong>第四层：兜底降级</strong></p>
        <p>多次调用失败后，停止工具调用，改用自然语言回答；或交给人工介入。</p>
      </li>
      <li>
        <p><strong>第五层：观测与优化</strong></p>
        <p>记失败日志，统计幻觉、格式错误高频 case，优化 system prompt；必要时用小模型做工具调用专用微调。</p>
      </li>
    </ol>
  </div>
</article>

<article class="note-card note-card--essay">
  <header class="note-card__head">
    <span class="note-card__badge">随笔</span>
    <time class="note-card__time">2026-09-15</time>
  </header>
  <h2 class="note-card__title">Prompt 相关观察：冷启动锚点与文字边界</h2>
  <div class="note-card__body">
    <h3>冷启动锚点</h3>
    <p><code>系统提示词 → 第一条用户消息 → 模型第一轮回复</code></p>
    <p>第一条用户消息是紧跟系统指令后的第一个用户输入，会直接塑造模型第一轮推理的基础心智状态。</p>
    <p>举你美妆例子对比：</p>
    <ul>
      <li>
        <p>首句 1：「脸过敏了！烂脸了！」（负面维权、情绪激烈）</p>
        <p>模型捕捉到紧急负面诉求，更容易放宽售后尺度，话术偏向安抚、让步；</p>
      </li>
      <li>
        <p>首句 2：「请问这款面霜早晚怎么涂抹？」（中性技术咨询）</p>
        <p>模型识别普通咨询场景，严格死守售后规则，一点不会松口提补偿。</p>
      </li>
    </ul>
    <p>哪怕系统提示词一模一样，初始锚点不同，输出策略直接分化。</p>
    <p>新增固定首条默认用户消息（比如统一预置：“我使用产品出现问题，需要售后咨询”）</p>
    <h3>文字边界标识</h3>
    <p>人为给 AI 画好分界线，告诉它哪部分是参考依据</p>
    <p>例子：把「用户提问」当成「知识库官方规则」（致命幻觉，业务事故）</p>
    <p>这是最危险的情况，直接篡改知识库本意。</p>
    <p>还是同一段无分隔文本：</p>
    <blockquote>
      <p>过敏可以全额退吗？7 天内未拆封可无理由退货，使用后皮肤过敏凭医院诊断可退 80% 货款……</p>
    </blockquote>
    <p>模型错误判定：</p>
    <p>将问句 <code>过敏可以全额退吗？</code> 识别为知识库里面写的官方条款。</p>
    <p>模型错误输出示例：</p>
    <blockquote>
      <p>根据售后规则，过敏可以办理全额退款；同时 7 天内未拆封商品也支持无理由退货，过敏情况最高可退 80%。</p>
    </blockquote>
    <p>优化后标准化包裹分隔知识库内容：</p>
    <blockquote>
      <p>用户问题：过敏可以全额退吗？</p>
      <p>【知识库参考片段 1】7 天内未拆封可无理由退货</p>
      <p>【知识库参考片段 2】使用后皮肤过敏凭医院诊断可退 80% 货款，拆封超 15 天不予售后</p>
      <p>清晰边界让模型精准识别参考依据，严格按库内内容作答，大幅减少幻觉。</p>
    </blockquote>
  </div>
</article>

<article class="note-card note-card--essay">
  <header class="note-card__head">
    <span class="note-card__badge">随笔</span>
    <time class="note-card__time">2026-09-15</time>
  </header>
  <h2 class="note-card__title">软件开发设计</h2>
  <div class="note-card__body">
    <p>软件开发设计：</p>
    <p>交互部分：ui交互、数据交互、逻辑交互、基础设施：数据权限管理，行为权限管理、报错系统</p>
    <p>部署发布：域名管理、私有化部署</p>
    <p>运营维护：监控日志、数据管理和分析、资源管理</p>
    <p>软件的本质：只是数据收集、复制、转化和传播</p>
    <ul>
      <li>任何一个有价值的生产系统，其实它的核心一定是来自于后端</li>
      <li>前端只是让人更容易理解这个数据而已</li>
      <li>维护：数据量、权限、性能、安全、并发、集成</li>
      <li>合格的产品：可靠、可扩展、安全、商业闭环、可观测性（最重要）</li>
    </ul>
  </div>
</article>

<article class="note-card note-card--memo">
  <header class="note-card__head">
    <span class="note-card__badge">小记</span>
    <time class="note-card__time">2026-09-14</time>
  </header>
  <h2 class="note-card__title">回调不等于数据同步</h2>
  <div class="note-card__body">
    <p class="note-card__lead">回调是「有人变了，快来查一下」的提醒；真正的数据同步，是你收到提醒之后，主动去企微拉最新资料，再写入 ERP。</p>
  </div>
</article>

<article class="note-card note-card--memo">
  <header class="note-card__head">
    <span class="note-card__badge">小记</span>
    <time class="note-card__time">2026-09-11</time>
  </header>
  <h2 class="note-card__title">正则化 vs 归一化：核心区别</h2>
  <div class="note-card__body">
    <p class="note-card__lead">归一化：预处理，改输入数据的数值分布，解决数值尺度问题；正则化：训练约束，防止模型过拟合，提升泛化能力。</p>
    <div class="note-table-wrap">
      <table>
        <thead>
          <tr>
            <th>对比维度</th>
            <th>归一化（Normalization / Standardization）</th>
            <th>正则化（Regularization）</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td><strong>核心目的</strong></td>
            <td>统一特征取值范围，消除量纲影响，<strong>加速收敛、保证梯度稳定</strong></td>
            <td>抑制过拟合，降低模型在训练集上的过度拟合，<strong>提升泛化能力</strong></td>
          </tr>
          <tr>
            <td><strong>作用阶段</strong></td>
            <td><strong>数据预处理阶段</strong>，训练之前就对原始特征做变换</td>
            <td><strong>模型训练阶段</strong>，在损失 / 参数更新过程施加约束</td>
          </tr>
          <tr>
            <td><strong>作用对象</strong></td>
            <td><strong>输入特征 X</strong>（样本数据）</td>
            <td><strong>模型参数 W、网络结构、标签、样本</strong>（模型本身）</td>
          </tr>
          <tr>
            <td><strong>常见方法</strong></td>
            <td>Min-Max 归一化、Z-score 标准化、BN（批量归一化，是层内激活归一化）</td>
            <td>L1/L2、Weight Decay、Dropout、早停、数据增强、标签平滑</td>
          </tr>
          <tr>
            <td><strong>是否改变数据含义</strong></td>
            <td>保留特征相对大小，只是缩放数值</td>
            <td>不修改原始输入数据，约束模型学习行为</td>
          </tr>
          <tr>
            <td><strong>过拟合关系</strong></td>
            <td><strong>不能直接防过拟合</strong>；BN 有轻微正则副作用，但不是它的本职工作</td>
            <td><strong>核心目标就是对抗过拟合</strong></td>
          </tr>
        </tbody>
      </table>
    </div>
    <div class="note-table-wrap">
      <table>
        <thead>
          <tr>
            <th>维度</th>
            <th>传统机器学习正则</th>
            <th>深度学习正则</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td>模型规模</td>
            <td>参数少，主要控制特征权重大小（L1/L2）</td>
            <td>参数海量，除了权重惩罚，还要约束激活、网络连接、训练过程</td>
          </tr>
          <tr>
            <td>主流方法</td>
            <td>L1、L2、早停</td>
            <td>Weight Decay、Dropout、BN、标签平滑、数据增强</td>
          </tr>
          <tr>
            <td>优化器影响</td>
            <td>几乎无差别</td>
            <td>Weight Decay 在 AdamW 才是正确实现，Adam+L2 会有偏差</td>
          </tr>
          <tr>
            <td>作用对象</td>
            <td>模型参数</td>
            <td>参数、神经元连接、特征分布、标签、样本</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</article>

<article class="note-card note-card--essay">
  <header class="note-card__head">
    <span class="note-card__badge">随笔</span>
    <time class="note-card__time">2026-09-08</time>
  </header>
  <h2 class="note-card__title">关于垂直逆向复刻网站 Skills 有感：Agent 解决的是什么</h2>
  <div class="note-card__body">
    <p class="note-card__lead">Agent 解决的是理解和判断。</p>
    <p>复刻网页的焚决：<a href="https://github.com/boyang-hu/website-rebuild-skill" target="_blank" rel="noopener noreferrer">boyang-hu/website-rebuild-skill</a></p>
    <ol>
      <li><strong>站点特异的理解：</strong>每个 URL 的栈、bundle 形态、怪癖都不同，要当场读证据再决策。</li>
      <li><strong>长流程编排：</strong>按检查清单选脚本、读 reference、看输出、决定下一步——像现场工头。</li>
      <li><strong>把理解落成产物：</strong>笔记、计划、工程接线、（必要时）重构代码、差异登记。</li>
      <li><strong>在规矩内处理例外：</strong>门红归因（报错分析）、该不该登记偏差、何时停下来问你——脚本只会红/绿。</li>
    </ol>
    <p>补充一点：每个 URL 的技术栈、打包产物、隐藏坑都不一样，不能依赖过往经验硬套；必须先抓取、解析当前这个 URL 的现场证据，再动态选择对应的处理路径。里面的工具是通用的，但怎么调用、参数怎么配、走哪条分支，由当前站点现场证据决定。</p>
    <aside class="note-card__pin">【最值钱】在 Skills 中——人（或长期项目）：发现坑 → 定方法 → 固化脚本与门。</aside>
  </div>
</article>

<article class="note-card note-card--essay">
  <header class="note-card__head">
    <span class="note-card__badge">随笔</span>
    <time class="note-card__time">2026-09-07</time>
  </header>
  <h2 class="note-card__title">MinIO 和 RustFS 对比</h2>
  <div class="note-card__body">
    <ul>
      <li>MinIO：中大文件、备份归档、普通业务上传体验最好；海量碎文件是短板。</li>
      <li>RustFS：S3 全兼容，专门优化了海量小对象、list 遍历，元数据去中心化，AI / 数据湖场景优势明显；但 RC 版本，大文件场景能力和 MinIO 差距不大。</li>
    </ul>
    <h3>简单选型口诀</h3>
    <ol>
      <li>你的业务：备份、视频、大附件、普通业务图片，文件总数不冲亿 → MinIO 很香。</li>
      <li>你的业务：AI 训练、Iceberg 数据湖，产生亿级大量小碎片文件，同时不想 AGPL 许可证约束 → 优先评估 RustFS。</li>
    </ol>
    <h3>对象存储是什么</h3>
    <p>对象存储是扁平化的，没有真实目录。<br>核心本质：以对象为最小单位，每个对象包含三部分：数据本体、自定义元数据、唯一键（Key）。采用扁平化命名空间，没有真实的目录结构，所谓文件夹只是 Key 的前缀模拟。</p>
  </div>
</article>

<article class="note-card note-card--memo">
  <header class="note-card__head">
    <span class="note-card__badge">小记</span>
    <time class="note-card__time">2026-09-06</time>
  </header>
  <h2 class="note-card__title">AI 时代真正该练的两件事</h2>
  <div class="note-card__body">
    <p>AI 时代，提升的是解决问题的思路，以及验证答案的能力。<br>模型可以很快给出方案，判断对不对、能不能落地，仍要靠自己。</p>
  </div>
</article>

<article class="note-card note-card--essay">
  <header class="note-card__head">
    <span class="note-card__badge">随笔</span>
    <time class="note-card__time">2026-09-05</time>
  </header>
  <h2 class="note-card__title">观测：当前 AI 大模型主要在做什么</h2>
  <div class="note-card__body">
    <p class="note-card__lead">把最近用大模型的感受整理如下。就日常接触来看，它的核心能力大致落在这四类。</p>
    <h3>1. 路由分路</h3>
    <p>先判断当前请求该怎么处理：直接回答、检索资料、调用工具、编写代码，还是继续追问澄清。<br>分流准确，后续步骤才容易做对。</p>
    <h3>2. 语义分析</h3>
    <p>包括理解、拆分、分类与判断。<br>把自然语言整理成可执行的信息结构：问题是什么、还缺哪些约束、优先处理哪一步。</p>
    <h3>3. 总结归纳与分析</h3>
    <p>文章提炼、观点整理，以及按既有风格进行的写作，都属于这一层。<br>操作电脑也可以看成同类能力：先明确目标，再把操作步骤整理成脚本或命令并执行。<br>任务卡住时，就换路径继续试。例如分析网站的接口与页面结构，找到可以自动化完成的方式。</p>
    <p>这里还有一层长期价值，可以叫复利工程：先借助 AI 找出一条可行路径，整理成文档、脚本、规则或示例，供之后的人和 AI 继续使用。单次探索会消耗时间，留下来的材料才能持续产生价值。</p>
    <h3>4. 多模态</h3>
    <p>覆盖视觉理解、语音理解与视频理解。<br>处理视频时，常见做法是先分帧、提取字幕，再做内容总结与串联。输入形式变了，核心仍是拆分信息并压缩成可用结论。</p>
    <aside class="note-card__pin note-card__pin--soft">简要归纳：大模型帮助人更快拆解问题、选择路径，并把一次偶然跑通的经验，整理成下次还能继续用的方法。</aside>
  </div>
</article>

</div>
