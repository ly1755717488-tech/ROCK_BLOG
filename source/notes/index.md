---
title: 随笔
date: 2026-09-02 15:30:00
updated: 2026-09-17 11:30:00
type: "notes"
comments: false
---

<div class="notes-board">

<article class="note-card note-card--essay">
  <header class="note-card__head">
    <span class="note-card__badge">随笔</span>
    <time class="note-card__time">2026-09-17</time>
  </header>
  <h2 class="note-card__title">做 Agent：务实、日志与第一性原理</h2>
  <div class="note-card__body">
    <h3>务实优先</h3>
    <p>务实的品质可以让大模型应用开发工程师走得更远、做得更多。务实让开发者不困于「过度冗长且孤立」的思考里，更容易找到简单有效的方案。</p>
    <p>很多做 Agent 的人容易陷入「无限复杂架构幻想」：多 Agent 分层、复杂编排、各种花哨组件。务实思路：优先最小可用闭环，先跑通核心链路，用最简单方案解决真实问题，而不是一上来堆砌复杂抽象。<strong>能解决问题的简单方案 &gt; 优雅但无法落地的宏大设计</strong>，这和「非必要不写单测、严控过度设计」的工程约束完全对齐。</p>

    <h3>像 Agent 一样思考</h3>
    <p>借助执行日志，观察上下文工程的设计缺陷，观察 Harness 的运行漏洞——这是 Agent 工程里很独特的调试思维。</p>
    <p>不要站在人类视角猜模型会怎么想；代入 Agent 视角：它拿到什么上下文、工具返回什么信息、它能看到哪些日志、它会做哪些错误推理。</p>
    <p><strong>Harness</strong> 就是 Agent 的执行沙盒 / 执行环境。绝大多数 Agent 失败，问题往往不在模型本身，而在 Harness 环境、上下文传递、工具返回结果的缺陷。</p>
    <p>核心手段：完整留存执行日志，复现 Agent 每一步的输入输出，定位它在哪一步产生幻觉、错误判断。</p>

    <h3>双向观察：人看 Agent，Agent 看人</h3>
    <p>人类观察 Agent 的行为很重要，Agent 观察人类的行为也很重要。双向观察，对应「协同 Agent」的人机闭环：</p>
    <ul>
      <li><strong>人看 Agent</strong>：人类观测 Agent 的执行轨迹，识别它的弱点、边界 case，用来迭代提示词、工具、Harness（评测、调试）。</li>
      <li><strong>Agent 看人</strong>：Agent 观察人类操作、决策偏好、纠错行为，做行为模仿、偏好学习，从人类反馈里持续优化（比如轨迹学习、few-shot、偏好微调）。</li>
    </ul>

    <h3>Lead, don't follow</h3>
    <p>从第一性原理思考事情的本质，去发现和构建新的东西。不要跟风复刻市面上现成 Agent 框架。剥离行业流行的方案假设，回到底层：这个任务真正需要什么能力？</p>
    <p>比如代码 Agent 的本质：读取代码 → 规划改动 → 在沙盒执行 → 验证结果。基于底层需求重新设计，而不是直接套别人的多 Agent 模板。</p>

    <h3>不要轻易放弃底层技术</h3>
    <p>底层技术会极大拓宽视角，要学会站在巨人的肩膀上。Agent 不只是写 prompt。操作系统、容器、沙盒、git、编译器、网络、数据库这些底层知识，决定 Harness 能做到什么程度。</p>
    <p>比如代码 Agent 要能在隔离环境编译、跑单元测试、管理文件变更；不懂容器 / 进程隔离，就做不出稳定的执行环境。站在巨人肩膀：读懂 OpenAI、Anthropic、Cursor / Grok Bot 这些团队公开工程文章，吸收他们踩过的坑，而不是从零盲试。</p>

    <p><strong>一句话总结：</strong>做 Agent 工程，拒绝空想和跟风；以日志为抓手，双向观察人机交互；扎根底层技术，优先务实落地，用第一性原理推导方案，而不是复制别人的成品。</p>
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
