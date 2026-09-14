---
title: CipherTalk 项目总结：本地数据处理才是人格克隆的底座
date: 2026-09-14 16:00:00
updated: 2026-09-14 16:00:00
tags:
  - CipherTalk
  - 人格克隆
  - RAG
  - Zod
  - MaiBot
  - GraphStore
categories:
  - 教程
cover: /img/cover-ciphertalk.png
top_img: false
description: CipherTalk 把微信脏聊天变成可校验的中间表示，再让模型在预算与降级护栏里工作。本地预处理决定成本上限和效果下限；附 MaiBot 拟人化、记忆分层与 GraphStore 经验。
keywords: CipherTalk,人格克隆,Zod,RRF,SQLite向量,MaiBot,GraphStore,FTS
---

{% note info %}
一句话：CipherTalk 是本地微信数据平台 + 派生检索层 + 多形态 LLM 应用（建议 / Agent / 人格）。最值得学的不是某个 Prompt，而是把脏聊天变成可控中间表示，再让模型在预算与降级护栏里工作。
{% endnote %}

## 稳定大模型生产的一条经验

自我克隆时，对话会先生成一张 **card**，而不是直接吐出一段克隆自己的 Prompt；生成的 card 再用 Zod 做规范化检查。

链路可以写成：

**对话 → 生成 card → Zod 校验（架构兜底）→ 多场景渲染**

核心做法：用工程手段兜住大模型的不确定性，把 LLM 从「直接产出最终结果」收缩为「产出结构化中间数据」。最后一公里的渲染与拼接交给代码。

## 克隆自己：六步流水线

### 1. 读会话消息（先约 6000，不够再全量）

先拉最近约 6000 条作为原料；近期太少、提炼不出稳定人格时，再回退全量历史。

设计意图：分身更像最近的你，同时控制初始数据量和成本。

### 2. 本地加工：`buildPersonaCorpus` 生成预算内语料

这一步**纯本地代码，不调大模型**：

- 把零散单条消息合并成完整一问一答轮次
- 统计发言频次、筛有效内容
- 按「近期优先 + 历史抽样」裁出约 1.4 万字高质量语料

1.4 万字是后续模型的输入预算：信息够用，又不轻易撑爆上下文。

### 3. 聊天大模型：`extractPersona` 提炼说话风格

把语料交给模型，提取语言表达特征，产出两样东西：

- **风格卡（card）**：语气、用词、口头禅、回复长短、常用句式。Zod 负责定义 Schema 并校验，把不可靠 JSON 收成可存、可改、可渲染的 card
- **few-shot 示例**：固定 3～10 条精选对话，完整拼进系统提示词

对应实现：`personaLlm.ts`。

### 4. 分块提炼深层个人画像

风格与人设内容是两个维度。聊天总量大，无法一次塞进模型：

- 最多拆成 12 块，每块调 `extractProfileChunk` 抽个人信息、观点、经历、行事习惯
- 再 `mergeProfile` 合并去重，得到完整 **profile**（你是谁、你怎么想、你在意什么）

### 5. 写入 `personas` 表

风格卡 + 深层画像写入本地库，后续分身聊天直接读这份基底。

### 6. 本地规则抽取问答对（`persona_pairs`，可选向量化）

同样不调大模型：按规则抽出「客户问 → 我答」，写入 `persona_pairs`。可选对问句做语义嵌入；用户提问时先找最像的历史问题，再参考当时的答法。

## 本地对话预处理才是核心底座

在 CipherTalk 里，微信数据的本地预处理不是喂模型前的打扫。它是整套 AI 能力的底座：优先级高于模型选型和提示词调优。LLM 与嵌入模型是上层执行器；本地处理锁住成本上限、效果下限、运行稳定性，以及多场景复用。

### 真实链路：AI 能力都先过本地层

模型消费的从来不是原始微信消息，而是本地已结构化的中间语料：

1. **会话语义索引**：`chatSearchIndexService` 拉索引，按规则切片段、双高水位增量更新，再让嵌入模型向量化；模型只负责向量生成与余弦检索
2. **人格克隆**：本地完成读消息、语音补转写、`mergeTurns`、`buildPersonaCorpus`、`renderProfileChunks`，再交给 LLM 做风格提取、分块画像与合并
3. **问答对样本库**：邻接轮次规则生成问答对；可选只对问句向量化，运行时检索后注入提示词
4. **自我 / 好友克隆**：同一套语料加工底座，翻转角色位即可切换；群聊补语料、表情包收集也由本地规则完成

微信消息又碎又短、不是标准一问一答，所以必须先切段、合并轮次。否则单条嵌入语义稀薄、问答对全错位，模型能力无从发挥。

### 为什么优先级高于模型调优

**1. 成本闸门在本地**

token 与调用次数上限由本地硬规则锁定：

- 语义索引：首建最多 1500 条；单片段 600 字 / 15 条 / 20 分钟断段；单条嵌入 ≤1000 字；批量 64 条；高水位只补增量
- 风格克隆：语料封顶约 1.4 万字（近期连续 + 历史抽样）
- 深层画像：单块约 1 万字，最多 12 块，3 路并发，超出只留最近块
- 问答对：单会话最多 3000 对，问答各截约 160 字
- 语音转写：批量补转有上限，已转写走缓存

换更强模型不会自动降本；本地预算一放松，成本立刻失控。

**2. 效果下限由本地定**

- 不做 `mergeTurns`（3 分钟同人消息合并）→ few-shot 与问答对角色错位
- 单条直接嵌入、不按片段切块 → 语义稀薄，跨多轮召回失效
- 未做语音转写直接丢掉 → 风格侧写缺口语维度
- 不足 50 条可用文本直接拒绝克隆 → 效果必然失真时，不进入 LLM

切段、轮次合并、过滤、抽样决定效果地板；模型和提示词只能在这个地板上抬天花板。

**3. 稳定性靠本地编排**

- 语义索引：无新消息返回「已是最新」；旧向量维度不兼容则跳过
- 深层画像：单块失败跳过；合并失败降级为无画像，风格卡仍可保存
- 问答检索：向量失败回退字符 bigram
- 嵌入未配置：向量整体禁用，本地索引和规则问答对仍在

**4. 一次加工，多处消费**

同一套中间数据（`mergeTurns` 轮次、`message_index`）可组合支撑多条产品线：

- 语义片段 → Agent / 深度模式 / 分身聊天的长程记忆
- 预算语料 → 风格卡 + few-shot
- 全量分块 → 深层人物画像
- 邻接规则 → `persona_pairs` →「像我回复」的检索式 few-shot
- 角色位翻转 → 克隆自己与克隆好友共用加工链路

{% note tip %}
本地数据处理把「微信原始消息流」变成「模型可消费单元」，同时承担形态化、预算化、质量闸、复用化。LLM 与嵌入模型只是执行工具。
{% endnote %}

## 检索过程

![检索流程](/img/ciphertalk/01-retrieval.png)

![混合检索与融合](/img/ciphertalk/02-retrieval-mix.png)

### RRF 算法

RRF（Reciprocal Rank Fusion，倒数秩融合）把多路检索结果（向量语义、关键词全文 FTS 等）无偏合并成一份统一排序。

### 关键词 FTS 检索

写入的是本地**持久化搜索索引**（SQLite：`message_index` + FTS），关掉 App 通常还在，不是纯内存缓存。

按需懒建：需要搜某个会话时，才把该会话（通常先是最近一部分）做成可持久检索的索引，再做关键词搜索；不够再往更早加深，完整后新消息增量补进同一份索引。不是启动时全量建完。

### 交叉编码器重排（Cross-Encoder Re-ranking）

粗召回 + 精排：先用混合检索（可搭配 RRF）召回 Top-N（通常 50～200 条），再用 Cross-Encoder 把查询与每个候选成对打分并重排。精准度高于纯召回侧融合；推理更贵、更慢，所以只对少量候选重排。

## 轻量化本地向量存储

没有引入 Milvus、Chroma、Pinecone、pgvector，而是 SQLite 文件存向量，应用层内存算余弦相似度：文件存向量 + 应用层暴力检索。

| 用途 | 库文件 | 表与核心内容 |
| --- | --- | --- |
| 会话语义索引 | `chat_vectors.db`（cache 目录） | `message_chunks`：片段文本 + embedding 浮点向量 |
| 克隆问答检索 | `agent_personas.db` | `persona_pairs`：问答对，问句可选存 embedding（blob） |

### 实现要点

1. **存储层**：`better-sqlite3` 同步驱动，适合嵌入式。向量只是表里的浮点数组 / blob；SQLite 本身不做向量检索。
2. **检索层**：embedding API 生成向量写入库；检索时把候选载入内存，用 AI SDK 的 `cosineSimilarity` 逐条算、排 Top-K。会话语义由 `messageVectorService.ts` 封装，人设问答由 `personaPairStore.ts` 封装。
3. **定位**：所谓向量库就是两个 SQLite 文件里的向量字段，计算全在应用进程内。

单用户 / 单会话量级通常可控，暴力检索够用；优先轻量、易部署。很多本地 AI 工具、桌面知识库也是这个思路。

> 用 better-sqlite3 操作 SQLite，BLOB 存 embedding；检索时 SQL 捞当前会话全部向量到内存，循环算余弦、取 TopK。没有库侧向量索引，用数据量小换最低部署成本。

## 迁移二开与整体分层

![迁移二开示意](/img/ciphertalk/03-fork.png)

![整体分层](/img/ciphertalk/04-layers.png)

| 层 | 职责 |
| --- | --- |
| 数据源 | 本机微信加密库（密钥 + 路径），近实时靠文件 / WAL，不是协议爬聊天 |
| 派生索引 | FTS 消息索引、会话向量、人设与问答对——按需懒建、增量 |
| AI 能力 | 回复建议、深度 Agent、人格克隆、分身聊天、MCP / Skills |
| 进程隔离 | 重活进 utilityProcess，避免堵 UI；AI 依赖 asarUnpack |

多条产品线共用本地底座，聊天 LLM、嵌入、检索、结构化抽取职责分离；贵的调用放在预算和闸门之后。

| 能力 | 模式 | 本地预处理 | 模型角色 |
| --- | --- | --- | --- |
| 普通回复建议 | 单次 generateText | 短上下文 | 直接出建议 JSON |
| 深度建议 | ToolLoopAgent | 长上下文 + 工具 | 先检索再写 |
| 自我 / 好友克隆 | ETL + 多轮 LLM | 轮次 / 预算 / 切块 | 侧写 card、few-shot、profile |
| 「像我」 | 建议 + 人设 | card 渲染 + pairs 检索 | 模仿语气 |
| 语义检索 | embed + 余弦 / 混合 RRF | 切段、高水位 | 嵌入模型 |
| 关键词检索 | FTS | 懒建 message_index | 无 LLM |
| 外部知识 | MCP | — | Agent 调工具 |

## 能学到的生产级经验

1. **本地数据处理优先于调模型**  
   微信消息碎、连发、多媒体。轮次合并、字符预算、抽样、最少条数拒绝克隆；向量按片段而不是按条；问答对用规则切，不靠 LLM 切全库。效果下限和成本上限主要由预处理决定。

2. **懒建 + 增量，而不是启动全量索引**  
   FTS、会话向量都是「用到某会话再建 / 再加深」；完整后只追新（游标 / 高水位）。用第一次慢，换多数会话零成本。

3. **混合检索，不迷信单一向量**  
   semantic_search：向量 + FTS + RRF；无嵌入则关键词兜底；persona_pairs 向量失败用 bigram。专名靠关键词，语义靠向量，生产要有降级路径。

4. **结构化中间表示（JSON + Zod），再渲染 Prompt**  
   风格卡先落 `card_json`，再 `buildMyPersonaContext` / 分身 system 不同拼法；Zod 校验、coerce、默认值、重试。可编辑、可复用、可演进。

5. **人设分层：怎么说 / 是谁 / 怎么回过**  
   - card + 静态 fewShots：怎么说  
   - profile map-reduce：是谁、边界、反应  
   - persona_pairs：运行时相似「问 → 答」  
   与会话语义索引（聊过什么）分开。风格、知识、答法样本分库分检索。

6. **Agent 有边界**  
   工具超时、只读 MCP；深度模式可检索、不发送、不乱改记忆。生产 Agent = 能力开放 + 策略收缩。

7. **多供应商现实**  
   宽松 JSON、重试、能力探测；视觉能力、嵌入维度不符就跳过旧向量。桌面多模型场景，解析与兼容层和业务同等重要。

8. **进程与 UX**  
   克隆进度条；单块 profile 失败跳过；合并失败可无深层画像仍保存风格卡。部分成功优于全失败。

9. **产品能力正交可组合**  
   深度 / 像我 / 对方画像 / MCP / 向量化——开关独立。降低门槛，也方便二开只增强一条线。

10. **分清本机资产与可迁移资产**  
    按 `session_id` / `account_id` 的 `chat_vectors`、自克隆键，不等于可跨机话术包。做 ToB「经理赋能客服」时，要单独设计导出 Playbook + 外部 KB，不能指望拷库。

### 架构取舍

| 取舍 | 好处 | 代价 |
| --- | --- | --- |
| SQLite 存向量 + 暴力余弦 | 简单、无运维 | 规模大时要换专用向量库 |
| 按会话自克隆 | 对这个人语气准 | 无全局自我，迁移难 |
| Agent 自主调工具 | 灵活 | 不保证每次都查 KB，需提示 / 策略加强 |
| 深度绑定本机微信 | 真数据、近实时 | ETL / 跨机要另做 |

### 可带走的经验清单

1. 先定义可消费的数据单元（轮次、片段、问答对、card），再调模型
2. 预算与闸门写在代码里（字数、条数、块数、最少样本）
3. 检索多路 + 降级（向量 / 关键词 / 规则）
4. 结构化抽取 → 存储 → 多场景渲染 Prompt
5. 能力分层（风格 / 画像 / 答法 / 会话记忆 / 外部 KB）
6. 懒建增量控制成本
7. Agent 工具要策略（超时、只读、任务边界）
8. 失败可降级、进度可感知
9. 迁移与本机索引分开设计

## 检索聊天图片（一开双路）

![聊天图片双路检索](/img/ciphertalk/05-image-search.png)

Zod = 运行时校验 + 编译时类型推导 + 数据转换清洗 + Schema 组合复用 + 异步校验 + JSON Schema 导出。

## 附录：MaiBot 拟人化处理

核心目标：压低一问一答的机械感，复刻真人社群里克制、随性、碎片化的交流。不靠单一 Prompt，而是决策层、表达层、频率门控、社群学习、发送后处理叠加；关键创新是**双 LLM 拆分**。

![MaiBot 四大功能](/img/ciphertalk/06-maibot.png)

完整链路：

`接收群消息` → 前置频率门控 → Planner（要不要发言、要不要调工具）→ Replyer（口语化回复）→ 真人化后处理 → 模拟打字延迟发送

### 双 LLM 分工

| 层级 | 模型定位 | 核心职责 | 输出边界 |
| --- | --- | --- | --- |
| Planner（决策大脑） | 思考规划专用 LLM | 读群氛围、节奏；判定是否参与、调工具、查记忆 | 不能直接输出聊天内容；只返回动作：wait / 调用 reply |
| Replyer（表达出口） | 口语生成专用 LLM | 结合人格、语境、社群话术产出日常短句 | 唯一合法发言通道 |

Planner 可以全程静默，不触发 Replyer，从根源减少逐条刷屏。

### 1. 时机门控：不是每条消息都接话

**回复必要性评分 `reply_necessity.py`**

- 加分：@机器人、直接点名
- 减分：哈哈、草、666、哦哦这类无实质短句；近期发言过密则限流

**空闲退避 `idle_backoff.py`**：群冷清、机器人许久没互动，进一步降低主动触发。没人聊时不强行搭话。

Planner 行为准则：优先旁观倾听；仅在被提及或对话题有兴趣时参与，其余保持安静。

### 2. 表达风格层

基线：风格平淡简短，参考贴吧、知乎、微博日常口语；拒绝浮夸长句。

Replyer 约束：日常生活化口语；禁用括号神态旁白、@前缀、多余符号、结构化排版；可随机切 1～2 字极简敷衍或慵懒语气。靠 Prompt 前置约束，没有单独的反 Markdown 拦截模块。

### 3. 社群进化学习

- `expression_learner.py`：归纳什么话题适合什么语气，生成时随机注入
- `jargon_learner`：圈子梗、黑话入库，给 Planner 理解潜台词

### 4. 发送后处理

`process_llm_response_segments`：

1. 清掉中文括号神态标注
2. 长句按标点拆成多条短句分开发
3. 小概率错别字，再隔一会儿补发修正
4. `send_service` 加随机打字延迟

## 记忆分层

| 层 | 是什么 | 典型实现 |
| --- | --- | --- |
| 工作记忆 / 短期 | 当前会话最近上下文 | `_chat_history`，群约 40 / 私聊约 60 条，超了就裁 |
| 中期（聊天回想） | 被裁掉或稍早的聊天，压缩成回想再召回 | `mid_term_memory` |
| 长期 | 跨会话摘要、关系、Episode、检索 | A_memorix：向量 + 图谱双路，可 `query_memory` |
| 人物层 | 对这个人的结构化了解 | 人物画像，挂在长期记忆证据上 |

![记忆分层](/img/ciphertalk/07-memory.png)

### DualPathRetriever：向量 + 知识图谱

| 名词 | 通俗解释 |
| --- | --- |
| 稠密向量 Embedding | 把一句话压成一串数字，意思越近数字越靠拢 |
| FAISS | 向量高速检索 |
| BM25 / SQLite FTS5 | 关键词检索，补语义误差 |
| 知识图谱 | 三元组织成关系网，如「赖 xx，喜欢吃，甜品」 |
| Hop | 关系网跳转次数：1hop 直接关联，2hop 再跳一层 |
| RRF 加权融合 | 多路结果按权重合并排名 |
| PPR | 在关系图上扩散热度 |
| 后验图门控 | 过滤无效、矛盾的关系记忆 |

![双路径召回](/img/ciphertalk/08-dual-path.png)

**通道 1：向量语义（权重最高）**  
问句转向量，FAISS 查段落向量池和实体 / 关系向量池；命中关系时顺带拉出佐证原文。

**通道 2：BM25 关键词**  
jieba 分词 + SQLite FTS5，专名、昵称更稳。

**通道 3：图谱关系跳转**  
从问句抽种子实体，1～2 hop 扩边，带出可信度与佐证条数。

融合时：段落用加权 RRF（向量约 0.7、关键词约 0.3）；关系三元组按佐证条数和可信度打分；再用 alpha 平衡闲聊段落 vs 关系提问。可选 PPR 热度扩散，最后用后验图门控过滤矛盾与低可信条目，截 Top K 送给模型。

三者分工：jieba + FTS5 抓字面；自研稀疏矩阵图谱挖人际羁绊；FAISS 抓语义相近片段。SQLite 可以充当轻量向量库，但原生不行，必须靠扩展；单机小体量场景也替代不了 Milvus、Qdrant。

## GraphStore：基于 SciPy 稀疏矩阵的内存知识图谱

定位：**不是 Neo4j 这类图数据库，是自研内存图存储**。文本抽三元组 → 写入 GraphStore，和向量库联合检索，提供 PPR 显著性扩散、BFS 路径召回。

| 变量 | 作用 |
| --- | --- |
| `_nodes` + `_node_to_idx` | 实体名 ↔ 矩阵下标；名称统一 `lower().strip()` |
| `_adjacency` | SciPy 稀疏邻接矩阵，存拓扑权重 |
| `_edge_hash_map` | `(src_idx, dst_idx)` → 关系哈希；关系文本、时效等元数据放 SQLite |

持久化：`graph_adjacency.npz` + `graph_metadata.json`；边关系映射落地 SQLite，支持快照。

- **读多写少（查询、PPR）：CSR** —— 取邻居、矩阵乘法快
- **频繁增量随机写：LIL** —— 原位修改；批量后再转回 CSR/CSC
- 扩容：LIL 直接 `resize()`；CSR/CSC 用 `bmat` 拼零块，禁止全量稠密复制

边写入：同一 `(src,dst)` 批内重复边覆盖权重；增量模式 LIL 改 `A[i,j]`；批量模式构造 delta 再相加。配套 `update_edge_weight`、`deactivate_edges`（软删）、`delete_edges`。

PPR：出度归一化转移矩阵上幂迭代，给实体显著性打分。`find_paths` 用双向 BFS，限制深度与路径数。`connect_synonyms` 按相似度阈值加同义边。生命周期有衰减、修剪弱边、清理孤立节点。快照写入新目录后原子改指针，避免半残主索引。

**CSR**：按行打包非零值，适合取下游邻居和 PPR；改单个格子很慢。  
**LIL**：每行一个小列表，单点写入快、可扩容；矩阵乘法和批量遍历慢。

链路：

```text
文本/摘要 → 三元组抽取 → GraphStore（节点/边写入）
                        ↓
                向量库 + 图谱联合检索
                        ↓
            PPR扩散 / BFS路径 补充召回
```

### 可带走的 GraphStore 实现提示词

```text
请在本项目中实现一套轻量知识图谱存储与检索组件，架构对齐 MaiBot/A_Memorix 的 GraphStore 设计理念（可重新实现，不要整文件抄袭授权不明代码）。

# 目标
用「稀疏邻接矩阵存拓扑 + 外部库存完整 SPO 语义」实现进程内知识图谱，供 Agent/RAG 做实体跳边召回与可选 Personalized PageRank 重排。不要引入 Neo4j/TigerGraph 等图数据库。

# 核心拆分（必须遵守）
1. GraphStore（结构层）
   - 实体名 ↔ 整数下标映射（含 canonicalize，大小写/空白规范化）
   - SciPy 稀疏邻接矩阵存拓扑与边权（CSR/CSC 只读与批量；LIL 增量写）
   - edge_hash_map: (src_idx, dst_idx) -> Set[relation_hash]
   - 三种模式：BATCH / INCREMENTAL / READ_ONLY，模式切换时自动转换矩阵格式
   - API：add_nodes / add_edges / get_neighbors / get_in_neighbors / get_weighted_neighbors / get_relation_hashes_for_edge
   - 持久化：原子快照；邻接矩阵用 scipy.sparse.save_npz/load_npz；节点映射与属性用 JSON；edge_hash_map 独立落盘或 SQLite 镜像

2. Metadata/RelationStore（语义层，可用 SQLite）
   - 存完整三元组：relation_hash, subject, predicate, object, confidence, source_paragraph/evidence, active/frozen 等
   - GraphStore 不存谓词字符串；结构边只通过 relation_hash 回查 SPO

3. 写入管线
   - 上游（LLM 抽取或导入）产出 entities + relations[{subject,predicate,object}]
   - 先 upsert 关系到 SQLite 得到 relation_hash
   - 再 graph.add_edges([(subject, object)], relation_hashes=[hash], weights=...)
   - 同一端点批内重复边：覆盖语义（取最后一个权重），不要因矩阵格式不同出现求和/覆盖不一致

4. 检索用法（可选但建议预留）
   - 从 query 得到 1~2 个种子实体
   - 1-hop（必要时 2-hop）沿邻接矩阵扩边，用 edge_hash_map 取回 SPO
   - 可选 Personalized PageRank：在邻接矩阵上迭代（阻尼约 0.85），用个性化向量对节点打分做重排
   - GraphStore 只提供矩阵与邻居；PPR 可作为独立算子类

# 技术栈约束
- 必须：Python 3.12+、NumPy、SciPy(scipy.sparse)、SQLite（或等价嵌入式库）
- 不要：把完整 SPO 塞进矩阵边属性；不要默认依赖 NetworkX 作为运行时主存储（仅允许做格式转换）
- 未安装 SciPy 时应显式报错，不要静默假实现

# 交付物
1. GraphStore 模块（含模式切换、增删边、邻居查询、快照 save/load）
2. RelationStore（SPO CRUD + 按 hash 查询）
3. 一个最小写入示例：文本/JSON -> 抽或手工 relations -> 入库+建边
4. 一个最小召回示例：种子实体 -> hop 扩边 -> 返回 SPO 列表
5. 简短 README：数据模型、三种模式何时用、与向量检索如何并联（向量另做，本组件只负责图结构）

# 验收标准
- 10 万级稀疏边可正常 add/save/load（内存与时间合理即可，不要求分布式）
- 同一对实体多谓词关系不丢（靠 edge_hash_map + RelationStore）
- INCREMENTAL 与 BATCH 写入后邻居查询结果一致
- 快照写入中断后不应留下半残主索引（原子切换）

请先给出模块划分与接口草案，再实现代码；实现时注释用简体中文。
```

## Event + 命名 Hook 双模式扩展点

适用：要给第三方 / 插件插进来，核心又不想被改乱；流程长、决策点多；需要可中止 / 可改参，又要防扩展拖垮主流程。

- **Event**：粗粒度阶段广播，数量少、命名稳。例如 `app.started` / `message.received`
- **Hook**：细粒度手术切口，每个挂点必须有 HookSpec。例如 `chat.receive.before_process`

Event 表示走到哪一站了；Hook 表示这一刀能不能改 / 停。不要做成同名同义的两套 API，也不要在每个私有函数里乱打 Hook。

### 可带走的 Event + Hook 实现提示词

```text
请在本项目中设计并实现「Event + 命名 Hook」双模式扩展点系统。目标是让核心流程可扩展，但扩展可控、可文档化、可演进。不要引入 Django signals / Blinker / pluggy 作为主框架（可参考其思想，但请自研清晰的 API 与契约）。

# 一、设计目标（必须理解）
用两套挂点，而不是一套打天下：

1) Event（粗粒度阶段广播）
- 语义：系统到达某个稳定生命周期/业务阶段
- 特点：数量少、命名稳、对外承诺强
- 用途：第三方插件订阅「发生了什么」；可旁观，或在明确允许时拦截
- 示例风格：app.started / message.received / plan.started / message.sent

2) 命名 Hook（细粒度手术切口）
- 语义：核心代码在关键决策点显式调用 invoke_hook("domain.action.phase", **kwargs)
- 特点：数量可多，但每个挂点必须有 HookSpec（契约）
- 用途：精确改参数、中止流程、或只观察
- 示例风格：chat.receive.before_process / memory.write.before_upsert / reply.before_post_process

原则：
- Event = 「走到哪一站了」（稳定面）
- Hook = 「在这一刀能不能改/停」（精确面）
- 禁止把 Event 和 Hook 做成同名同义的两套重复 API
- 禁止在每个私有函数里乱打 Hook；只放在决策边界（校验后、入库前、发外部副作用前、模型调用后等）

# 二、核心组件
请实现至少这些模块：

1. EventBus / EventDispatcher
- emit(event_type, payload) -> DispatchResult
- 订阅者按 priority/weight 排序
- 支持两种处理器角色：
  - observe：并发/旁路，不影响主流程成败（失败只记日志）
  - intercept（可选）：串行，可要求停止继续传播或改 payload（需显式授权）
- 事件名建议常量或枚举，变更需版本说明

2. HookSpecRegistry
- register_spec(HookSpec)
- HookSpec 至少包含：
  - name: str（点分命名，如 domain.action.phase）
  - description: str
  - parameters_schema: JSON Schema 或等价结构
  - allow_abort: bool
  - allow_kwargs_mutation: bool
  - default_timeout_ms: int
  - owner_module: str（哪个业务模块拥有该挂点）
- 各业务模块自己注册 Spec（像目录 catalog），避免全球一个大文件堆挂点

3. HookDispatcher
- invoke_hook(name, **kwargs) -> HookDispatchResult
- HookDispatchResult 至少包含：kwargs（最终参数）、aborted、stopped_by、errors、custom_results
- 处理器排序固定且可预测，建议：
  1. blocking 先于 observe
  2. early / normal / late
  3. 内置/一等扩展 先于 第三方
  4. plugin_id / handler_name
- blocking：串行；可按 Spec 修改 kwargs；若 allow_abort 则可 abort
- observe：并发旁路；禁止影响主流程控制；超时/异常不影响返回
- 严格执行 Spec：不允许 mutation 时忽略或拒绝修改；不允许 abort 时拒绝中止
- 每个 handler 有超时与错误隔离；可加简单熔断（连续失败后跳过）

4. Extension Registry（插件/扩展注册表）
- 扩展可注册：EventHandler、HookHandler
- 先支持同进程实现
- 预留接口：未来可换成 RPC/子进程（不要把业务逻辑写死在“必须同进程”）

# 三、核心代码如何埋点
- Event：在阶段切换处 emit，payload 尽量是稳定 DTO，避免塞入不可序列化对象（若计划跨进程）
- Hook：在关键边界调用：
  result = await hooks.invoke("xxx.before_y", **ctx)
  if result.aborted: ...
  ctx = result.kwargs
- 新增 Hook 必须先 register_spec，再 invoke；禁止无 Spec 的裸字符串挂点上线

# 四、安全与稳定性（默认不信任扩展）
- 超时：每个 handler 必有 timeout
- 权限：哪些 hook 允许 abort / mutation 由 Spec 决定，不由插件自己声明说了算
- 失败策略：
  - observe 失败：记录并继续
  - blocking 失败：可配置 fail-open 或 fail-closed；默认建议 fail-open + 错误列表，关键安全点可 fail-closed
- 日志：记录 hook/event 名、handler、耗时、是否 abort、错误摘要
- 不要让扩展直接拿到任意全局单例写爆；优先通过 kwargs 与显式 Capability API

# 五、文档与治理
- 生成/维护 Hook Catalog 与 Event Catalog（名称、描述、可否 abort、所有者）
- 约定命名：
  - Event: noun.past_or_stage（稳定）
  - Hook: domain.action.phase（before_/after_/on_）
- 破坏性变更：改 Event 名或 Hook 参数需版本号或迁移说明

# 六、交付物
1. 上述模块的可运行实现（语言与本项目一致）
2. 2~3 个示例：
   - 一个 Event 订阅（observe）
   - 一个 Hook blocking（改参数）
   - 一个 Hook abort（在 allow_abort=true 的挂点）
3. 简短 README：何时用 Event、何时用 Hook、如何新增挂点、排序与超时语义
4. 最小测试：排序、mutation 权限、abort 权限、observe 失败不影响主流程、超时隔离

# 七、验收标准
- 同名 Event/Hook 职责不混淆，文档能一句话说清区别
- blocking/observe 行为符合 Spec，越权修改/中止无效
- 主流程在 observe 崩溃或超时时仍能完成
- 新增业务挂点只需：注册 Spec + invoke +（可选）示例处理器，无需改分发器内核

请先输出接口草案与事件/挂点命名规范，再实现代码。注释与文档使用简体中文。
```
