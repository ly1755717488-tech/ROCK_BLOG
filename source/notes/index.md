---
title: 随笔
date: 2026-09-02 15:30:00
updated: 2026-10-06 22:36:40
type: "notes"
comments: false
---

<div class="notes-board">

<article class="note-card note-card--essay">
  <header class="note-card__head">
    <span class="note-card__badge">随笔</span>
    <time class="note-card__time">2026-10-06</time>
  </header>
  <h2 class="note-card__title">CC 规划、CX 写码：按能力分工更省钱也更快</h2>
  <div class="note-card__body">
    <p>Claude Code（CC）和 Codex（CX）擅长的面不一样：CC 偏规划、前端、全栈；CX 偏审代码、后端。底层、C/C++、编译器、解码器这类，两边都不行。</p>
    <h3>节奏、成本和排版</h3>
    <p>CC 走 Claude API，单价贵，但适合拿来做规划和拆问题。真正写代码交给 Codex，整体花不了多少钱。</p>
    <p>CC 最大的好处是让你保持很快的节奏去思考、推进。Codex 别的都还行，就是想太久，排版也不太利于沟通。</p>
    <h3>权限和并行</h3>
    <p>两边最大的差别是权限。CC 能做更放开的分析，抓包、软件分析之类，商业场景更对口。CX 权限绕不过去，去掉这一层，Claude API 和 gpt-5.x-codex API 能力其实差不多。CX 这边得绕半天，还没法做到百分之百抓数据分析。</p>
    <p>软件设计上还有一点：CC 直接多模型并行，所以快；CX 一个模型串行，所以慢。CC 读文件用 Haiku（不管你选的是 Sonnet 还是 Opus 4.5），分析才用你自选的模型。</p>
    <h3>代码质量和推荐用法</h3>
    <p>同样的应用场景里，CX 写出来的代码更规范，Bug 也更少。</p>
    <p>比较好用的办法：用 CC 做规划、分析，再指导 CX 写代码，直接让 CX 执行。成本上划算，能力上也对得上，整体更高效。</p>
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
