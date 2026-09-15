---
title: Claude Code 源码笔记：Prompt 缓存、三层记忆与上下文压缩
date: 2026-09-15 16:43:00
updated: 2026-09-15 16:43:00
tags:
  - Claude Code
  - Prompt Caching
  - 记忆架构
  - 上下文压缩
  - MCP
categories:
  - 教程
cover: /img/cover-claude-code.png
top_img: false
description: Claude Code 源码侧观察：Prompt Caching 与缓存分裂、热温冷三层记忆、五级上下文压缩、MCP 工具按需加载，以及 Karpathy 式 Markdown Wiki 知识库。
keywords: Claude Code,Prompt Caching,DYNAMIC_BOUNDARY,Self-Heading Memory,上下文压缩,MCP,RAG,grep
---

{% note info %}
从 Claude Code 相关实现里抽出几块：Prompt 缓存怎么拆、记忆怎么分层、上下文怎么漏斗式压缩、MCP 工具怎么按需暴露。
{% endnote %}

## 1. Prompt Caching 与缓存分裂

### 缓存分裂：把「不变的」和「变的」拆开

Anthropic 的 Prompt Cache：只要每次请求的提示词**前半部分完全一样**，API 就把这部分缓存起来，只算一次钱、只处理一次。

用边界标记 `DYNAMIC_BOUNDARY`，把提示词切成两半：

**上半部分：静态缓存区（可全球共享）**

- 内容：角色定义、行为规范、工具说明、通用规则等**永远不变**的内容
- 效果：大量用户共享同一份缓存，加载一次后几乎零成本、零耗时

**下半部分：动态独立区（每人独算）**

- 内容：当前时间、用户 Git 状态、个性化配置、本次请求专属信息等**每次都变**的内容
- 效果：只对这一小段重新计算，明显减少计费 token，也更快

## 2. 三层记忆：热常驻、温按需、冷检索

（Self-Heading Memory）

| 层级 | 数据类型 | 存储形式 | 加载方式 | 核心作用 |
| --- | --- | --- | --- | --- |
| 第一层（热数据） | MEMORY.md 常驻索引 | 固定文件 | 每次对话**强制加载** | 项目 / 用户核心规则、目录、偏好，始终在场 |
| 第二层（温数据） | 话题文件 | 独立文件 | 新对话时**按需加载**（最多约 5 个） | 编码偏好、架构约定、踩过的坑等细节 |
| 第三层（冷数据） | 历史对话 | `.jsonl` 归档 | 仅**关键词检索**时加载 | 更早的历史，按需召回 |

### Grep 和向量 RAG

- **Grep**：关键词精确搜索，快、稳、准、省、简单；适合更早历史对话的按需召回
- **向量 RAG**：语义模糊搜索，懂意思，但更慢、更复杂、更贵

## 3. 五级上下文压缩：漏斗式瘦身

AI 代码助手（如 Claude Code）的上下文窗口保护：像漏斗一样**从轻到重、层层递进**压缩上下文，尽量保住关键信息，避免提示词过长触发 API 报错，同时压低 token 成本。

实在压不住时，还有断路器：连续失败约 3 次就自动停下来。

## 4. 工具设计：MCP 按需加载

调用很多 MCP 工具时，Claude Code **不会**把所有 MCP 的完整描述都塞进系统提示词。

做法是：先给模型一份**工具名 + 一句话介绍**的精简清单，让模型自己按需选择，再加载完整定义。这样能少掉大量 token。

## 5. Karpathy 式 LLM 知识库

核心思路：让 LLM 像程序员维护代码一样，持续构建 Markdown 知识库（Wiki），而不是只做一次性检索。

三层：

- **Raw Sources**：原始资料
- **Wiki**：LLM 维护的知识层
- **Schema**：规则文件

三大操作：

- **Ingest**：录入
- **Query**：提问
- **Lint**：体检

每次交互都让知识库更完善，少掉传统 RAG「用完即弃」的局限。
