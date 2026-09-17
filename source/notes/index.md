---
title: 随笔
date: 2026-09-02 15:30:00
updated: 2026-09-17 11:37:00
type: "notes"
comments: false
---

<div class="notes-board">

<article class="note-card note-card--essay">
  <header class="note-card__head">
    <span class="note-card__badge">随笔</span>
    <time class="note-card__time">2026-09-17</time>
  </header>
  <h2 class="note-card__title">大模型 Agent 工程实践：务实、复盘与底层思考</h2>
  <div class="note-card__body">
    <p>深耕大模型 Agent 工程落地，愈发明白：能走得远的开发者，靠的是极致的务实、深度的复盘和扎实的底层功底，而不是花哨的架构设计和跟风式开发。</p>

    <h3>一、务实落地：拒绝架构空想，优先最小闭环</h3>
    <p>务实是大模型应用开发工程师最核心的底色，能让人在复杂的 AI 工程迭代里走得更远、落地更多成果。</p>
    <p>很多人做 Agent 开发时，容易陷入无限复杂的架构幻想：盲目堆多智能体分层、复杂任务编排、各类花哨组件，沉迷于看似优雅、宏大的技术方案，却忽略最核心的落地问题。这种脱离实际的空想只会拖慢节奏，方案难落地、迭代也停住。</p>
    <p>工程务实的原则很简单：优先最小可用闭环，先跑通核心链路。用最简单、最轻量的方案解决真实业务问题，严控过度设计。</p>
    <p>能解决问题的简单方案，永远优于优雅但落不了地的宏大设计。少堆无效抽象，聚焦核心能力闭环，才是 Agent 迭代该有的节奏。</p>

    <h3>二、换位思考：以 Agent 视角调试，用日志定位本质问题</h3>
    <p>Agent 工程有一套独特的调试思维：像你的 Agent 一样思考。</p>
    <p>多数开发者习惯站在人类视角揣测模型逻辑、预判执行行为，这也是很多问题拖很久的原因。更稳妥的做法是完全代入 Agent 视角，还原完整执行链路：它拿到了什么上下文？工具返回了哪些信息？可读取的日志范围是什么？又会在哪些环节产生错误推理与幻觉？</p>
    <p>先明确一点：<strong>Harness 是 Agent 的执行沙盒与运行环境</strong>。日常开发里，绝大多数 Agent 执行失败，根因往往是 Harness 配置异常、上下文传递丢失、工具返回结果不规范等工程漏洞，而不是模型本身能力不够。</p>
    <p>解决这类问题的核心抓手，是完整留存执行日志。用日志复现每一轮思考、工具调用、结果接收的输入输出，定位幻觉、报错、逻辑偏离发生在哪一步，再做精准迭代。</p>

    <h3>三、双向共生：人机互相观察，构建协同 Agent 闭环</h3>
    <p>人机协作的核心是双向观察与成长，这也是协同 Agent 区别于自主 Agent 的关键特质。</p>
    <p><strong>人类观察 Agent，是为了迭代优化。</strong>通过完整执行轨迹，捕捉能力短板、边界 case、执行漏洞，再针对性改提示词、适配工具、完善 Harness 评测与调试体系，让 Agent 持续进化。</p>
    <p><strong>Agent 观察人类，是为了学习适配。</strong>它可以记录人类的操作习惯、决策偏好、纠错逻辑，借助轨迹学习、Few-Shot 示例、人类偏好微调等方式，更贴合使用习惯，提高执行精准度。</p>
    <p>一观一学、一调一优，构成协同 Agent 的人机闭环迭代。</p>

    <h3>四、独立思考：第一性原理，拒绝跟风复刻</h3>
    <p>做 Agent 工程，始终要谨记：Lead, don't follow。</p>
    <p>不要盲目复刻市面上现成的 Agent 框架与方案，也不要被流行架构范式裹挟。用第一性原理拆问题：剥掉固化经验假设，回到任务最本质的需求——这个任务真正需要的核心能力是什么？</p>
    <p>以常见的代码 Agent 为例，抛开花哨包装，底层逻辑其实很清楚：读取代码仓库信息 → 规划改动方案 → 在沙盒执行操作 → 验证结果。</p>
    <p>基于底层需求重新设计适配方案，而不是无脑套用通用多 Agent 模板，才能做出贴合业务、稳定高效的产品，从「跟随复刻」走向「引领创新」。</p>

    <h3>五、扎根底层：筑牢技术根基，站在巨人肩膀迭代</h3>
    <p>很多人以为 Agent 开发只是写提示词、调模型。底层技术功底，才决定工程上限。</p>
    <p>操作系统、容器沙盒、Git、编译器、网络、数据库等知识，直接决定 Harness 环境的稳定性和能力边界。要做可用、稳定的代码 Agent，就得懂容器与进程隔离，才能做安全沙盒执行；懂编译与运行机制，才能让 Agent 自主完成编译、单元测试、文件变更管理。没有底层支撑，上层设计都是空中楼阁。</p>
    <p>高效迭代也不靠闭门造车。站在巨人肩膀上，研读 OpenAI、Anthropic、Cursor、Grok Bot 等团队的公开工程文档与实战经验，吸收成熟踩坑与最优实践，少从零盲目试错，开发效率会高很多。</p>

    <h3>结语</h3>
    <p>Agent 工程的核心心法：拒绝空想跟风，坚持务实落地；以日志为抓手，打通人机双向协同迭代；深耕底层根基，用第一性原理拆问题、做设计——不复刻成品，只创造价值。</p>
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
