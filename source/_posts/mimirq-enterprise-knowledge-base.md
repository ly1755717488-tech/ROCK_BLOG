---
title: MimirQ 企业知识库搭建要点：把 RAG 做成可审计的数据流水线
date: 2026-09-04 17:15:03
updated: 2026-09-04 17:15:03
tags:
  - RAG
  - 企业知识库
  - MimirQ
  - 可观测性
  - 架构设计
categories:
  - 教程
cover: /img/cover-mimirq.png
top_img: false
description: MimirQ 偏向 RAG 基础设施层，解决企业知识库“排错难、改崩效果、无法验收”的工程痛点。本文梳理可观察性、插件合约、组件工厂、Golden 门禁、证据优先与权限裁剪等核心要点。
keywords: RAG,企业知识库,MimirQ,Citation,Golden,可观测性
---

{% note info %}
MimirQ 偏向 RAG 基础设施层，解决企业知识库「排错难、改崩效果、无法验收」的工程痛点，适合对知识库质量、可审计性有高要求的项目。
{% endnote %}

**浓缩成一句话：**  
学的是把 AI 知识库当成可审计的数据流水线来建——组件可插拔、过程可看见、版本可回归；模型只是流水线里可替换的一环。

## 1. 先治链路，再谈模型

企业 RAG 真正难的是：**错了能不能定位、策略能不能换、质量能不能证明没滑坡。**

把链路拆成：

> 解析 → 治理 → 切块 → 索引 → 召回 → 重排 → 引用 → 回归

每一步都要有明确输入输出，而不是一个黑盒按钮。

## 2. 可观察性是产品能力，不是运维附加

解析结果、chunk 边界、召回片段、重排分数、最终引用——全部可视化。

做 AI 系统时，把 **Trace / Preview / 证据引用** 作为首要能力；没有中间结果，就没有根因分析，只有玄学调参。

{% note tip %}
LangSmith 是 LangChain 官方推出的 SaaS 形态 LLM/Agent 可观测、调试、评估平台，适合看提示词与模型输入输出；但上游解析、切块、治理问题，仍要靠业务 Trace。
{% endnote %}

现有 RAG（解析、切块、向量检索、问答）建议分阶段补齐：

1. **Citation 证据引用（优先）**：带字符位置信息
2. **Preview 切块预览**：支持人工审核
3. **Trace-Bundle 日志复盘**：整链回放

每个文档块至少记录这些关键字段：

- `document_id`：文档唯一 ID
- `chunk_id`：块唯一 ID
- `content`：片段文本
- `score`：相似度分数
- `rerank_score`：重排分数（如启用）
- `evidence_start_char`、`evidence_end_char`：该块在原始全文的字符起止下标（最重要）

## 3. 平台中立 + 业务进插件

平台不写「某市某事项怎么排」；业务规则进插件合约：

- governance
- chunk
- metadata
- retrieval_policy
- golden

通用内核吃标准合约，行业差异用插件扩展。换客户、换领域时，不用改核心架构。

### RAG 项目流程

PDF 通用解析完成 → governance 插件做业务治理 → metadata_schema 校验 → chunk 插件按业务规则切块 → 再校验并做视图投影

#### 1）调用 governance 插件：全局业务治理

在完整文档上执行业务逻辑，例如：

- 识别全文办事事项清单，梳理整体结构
- 补全文档级业务字段：`所属区县=常州市`、`办理部门=市行政审批局`
- 清洗页眉页脚、水印、备注说明
- 统一业务术语格式

#### 2）第一次校验：metadata_schema 契约校验

平台拿着插件自带的 `metadata_schema.json` 检查：

- 有没有写契约外的非法字段
- 字段类型对不对
- 有没有试图修改平台保留字段（如 `document_id`）

校验通过才往下走；有问题直接拦截，避免错误数据进入切块，导致索引全作废。

#### 3）调用 chunk 插件：按业务粒度切块

#### 4）第二次校验 + 投影视图

切块后每个 chunk 再跑一遍 schema 校验，生成：

- `_indexed_metadata`：索引用
- `_display_metadata`：前端展示用

校验通过后，才送去建向量索引、BM25 索引。

#### 5）通用检索器执行 filter / boost / expand

业务插件只负责写规则，平台只负责通用执行。

**filter（结果过滤）示例：**

```json
"filter": [
  { "field": "区县", "value": "常州市", "operator": "==" }
]
```

平台只做字段匹配，不理解“区县”的行政含义。

**boost（加权提权）示例：**

```json
"boost": [
  { "field": "section_type", "value": "申报流程", "weight": 2.0 }
]
```

**query_expand（查询扩展）示例：**

```json
"query_expand": {
  "append_terms": ["办理流程", "申报材料", "办理条件"]
}
```

{% note success %}
本质是：判断「过滤什么、给谁加权、扩什么词」的业务逻辑全在插件配置里；平台检索器是通用执行器。换金融、医疗、教育，核心代码都不用改，只换插件 JSON。
{% endnote %}

## 4. 「可替换」靠边界，不靠堆功能

30 个解析器、86 种切块、13 类重排——价值不在数量，而在：

> 统一接口 + 注册表 + 工厂 + 配置

### 配置：告诉系统「用哪个」

```text
chunk_strategy = "政务事项型"
RERANKER_PROVIDER = "bge"
VECTOR_BACKEND = "milvus"
```

### 注册表：组件名单 + 地址簿

| 组件名称 | 对应代码路径 | 备注 |
| --- | --- | --- |
| `按字数切` | `chunks.char_size.Chunker` | 基础切块 |
| `政务事项型` | `chunks.gov_affair.GovChunk` | 政务业务切块 |
| `语义切块` | `chunks.semantic.SemanticChunker` | 语义切块 |

建议懒加载：用到哪个才导入哪个。

### 工厂：按名字取组件

流水线里不要写一堆 `if/else`，只写：

```text
chunker = ChunkerFactory.get_chunker(配置名)
```

### 易变部分接口化

解析器、embedding、向量库、rerank 都定义统一基类接口，例如：

- `BaseChunker.split_documents(docs) → docs`
- `BaseReranker.rerank(query, docs) → ranked_docs`
- `BaseVectorStore.add_docs / search`

换 BGE 重排为 LLM 重排，通常只需：写新实现 → 注册表加一行 → 改配置名。检索编排、Citation、Trace 不用动。

### 稳定部分强约束

1. **Dataset 范围**：禁止无边界全库搜索
2. **Provenance**：每个 chunk / citation 可追溯到原始文档
3. **Citation 合约**：输出字段固定，前端与 Trace 只认这一套格式

一句话：

**可替换 = 统一接口 + 注册表挂实现 + 工厂按配置取件**  
**靠边界 = dataset 范围、来源追溯、引用格式是硬合约**

## 5. 检索质量 ≠ 回答好听

门禁看的是：证据有没有进 TopK、metadata 是否命中、噪声是否过高——而不是 LLM 答得顺不顺。

很多项目会踩坑：**检索很烂，但大模型很会“圆话”**。表面通顺，实际证据全错，线上悄悄崩掉。

### 门禁看什么

| 门禁维度 | 核心指标 | 通俗解释 |
| --- | --- | --- |
| 证据进 TopK | hit@1/3/5/10、MRR、NDCG | 正确答案有没有排进前几名 |
| metadata 命中 | expected_metadata_hit_rate | 业务字段过滤是否生效 |
| 有效依据 / 噪声 | effective_context_rate、noise_rate | TopK 里多少真正有用 |
| must_recall | must_recall_pass_rate | 必须召回的文件/锚点是否出现 |

业务上“什么算对、什么必须有”，写在插件的 `golden_rules.json` 里。

### 两道门分开验收

1. **retrieval-only gate（发版必过）**：只跑检索，不调用大模型
2. **answer / RAGAS gate（可选补充）**：看忠实度、相关性

检索不合格，直接拦在发版前，连生成资格都没有。

## 6. Golden 题集 = 质量安全带

固定题集 + 阈值 + CI fail-closed。升级模型、改切分、换解析器后自动回归。

任何影响召回的改动，都要回答四个问题：

1. 改了哪一层？
2. 有没有业务逻辑漏进平台？
3. 有没有回归证据？
4. 有没有回滚路径？

### 三要素

1. **固定题集**：平台通用题 + 插件 `golden_rules.json`
2. **阈值**：如 `hit@3 ≥ 90%`、`噪声率 ≤ 20%`、`must_recall = 100%`
3. **CI fail-closed**：不达标直接失败，不准发版

对应落地手段：

- `retrieval_config_hash` / `pipeline_hash`
- `test_pipeline_plugin_boundary.py`
- gate report / `run.detail.json`
- 插件开关、旧配置切换、文档多版本激活

## 7. 先评估数据，再选技术

扫描件用 MinerU、复杂版式用 Docling、普通 Office 用 MarkItDown——没有全局最优解析器。

完整闭环：

> 客户语料 → 数据画像 → 多解析器对比预览 / 基准测试 → 自动生成入库策略 → 锁定配置 → 正式入库（保留版本与质量证据）

关键原则：

- 解析器建议可以自动给，但最终以配置为准
- 不要在平台核心硬编码“扫描件必须用某解析器”
- **数据形态决定技术选型，而不是反过来**

## 8. 证据优先于答案

四件套：

1. **Citation**：带字符偏移的证据片段
2. **Provenance**：完整来源链，防篡改可审计
3. **must_recall**：核心证据缺失可触发拒答
4. **Evidence Capsule**：一次检索证据的封存快照

系统默认：

> 没有依据 = 不合格 = 不能过

可配合：

- `RAG_VISIBLE_EVIDENCE_ONLY`
- `retrieval_contract_mode = evidence_strict`

## 9. 机器观测 vs 业务 Trace

| | OTel / LangSmith / Prometheus | 业务 Trace |
| --- | --- | --- |
| 记录内容 | 函数调用、耗时、模型输入输出 | 每步业务数据（分数、片段、最终证据） |
| 定位范围 | 性能、服务故障、提示词问题 | 检索质量、证据链路、业务逻辑问题 |
| 层级 | 技术运维增强层（可关） | 产品核心能力层（自研） |

用户投诉“回答错了”时：

- Prometheus：可能只看到接口正常
- LangSmith：可能只看到最终喂给模型的上下文
- 业务 Trace：能回放改写、多路召回、融合、重排、最终 Citation 全链路

## 10. 全链路版本指纹：方便回滚

一次发布绑定多段指纹：

| 指纹 | 覆盖什么 |
| --- | --- |
| pipeline_hash | 入库管线：解析器、切块、治理等 |
| retrieval_config_hash | 检索策略、融合、重排等 |
| plugin package_hash | 业务插件包内容哈希 |
| 阈值 / 开关快照 | 门禁阈值、KG 开关、特性 flag |

标准回滚路径：

> 关插件 → 关 KG → 切回 retrieval profile → 恢复旧 thresholds

目标是：**切配置，不改代码、不重跑全库**。

## 11. 补充工程基建

### Pre-POC：Python 规则扫描（非大模型）

| 模块 | 作用 |
| --- | --- |
| format_distribution | 格式分布 |
| length_distribution | 篇幅分布 |
| pdf_page_classifier | 扫描件 / 混合件 / 原生文本判断 |
| md5_dedup | 精确去重 |
| simhash_similarity | 近重复检测 |
| sensitive_info | 敏感信息抽检 |

### 存储与任务队列

- **MinIO / S3**：原始文件
- **PostgreSQL**：元数据、权限、citations、Trace（JSONB）
- **SQLite**：单文档表格旁路存储
- **Arq + Redis**：异步任务队列，支持幂等

### 知识库权限：三层 ACL + Security Trimming

1. 租户 RBAC
2. Dataset 权限
3. Document ACL（只能收紧，不能放宽）

检索前后双重校验：先按允许的 `document_id` 搜索，返回后再二次过滤。没权限的内容不进 Trace、不进 API、不喂给 LLM。

### Query 增强手段

| 技术 | 优点 | 风险 |
| --- | --- | --- |
| sparse 稀疏检索 | 擅长专有名词、编号、术语 | 同义语义弱 |
| query 改写 | 口语转书面检索问句 | 改写跑偏 |
| multi-query 扩展 | 扩大召回 | 增加噪声与成本 |
| HyDE | 短问句也能拿高质量向量 | 假想文本编造错误 |
| 业务词典别名替换 | 快，处理简称行话 | 词典维护成本 |

### 数据库怎么选

- 核心主存、多租户、半结构化元数据 → **PostgreSQL**
- 单文档附属表格 → **SQLite**
- 技术栈强绑定且场景简单 → 可考虑 MySQL

{% note success %}
落地口诀：先把链路拆开、证据留住、门禁立住、组件可换、业务进插件。模型可以换，质量闭环不能丢。
{% endnote %}
