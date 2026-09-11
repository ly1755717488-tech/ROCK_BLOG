---
title: 随笔
date: 2026-09-02 15:30:00
updated: 2026-09-11 10:08:00
type: "notes"
comments: false
---

<div class="notes-board">

<article class="note-card note-card--essay">
  <header class="note-card__head">
    <span class="note-card__badge">随笔</span>
    <time class="note-card__time">2026-09-08</time>
  </header>
  <h2 class="note-card__title">关于垂直逆向复刻网站 Skills 有感：Agent 解决的是什么</h2>
  <div class="note-card__body">
    <p class="note-card__lead">Agent 解决的是理解和判断。</p>
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
