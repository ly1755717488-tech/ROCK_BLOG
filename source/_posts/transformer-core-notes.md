---
title: Transformer 核心知识点梳理：Encoder、Decoder 与 BERT / GPT
date: 2026-09-03 16:13:55
updated: 2026-09-03 16:13:55
tags:
  - Transformer
  - BERT
  - GPT
  - 深度学习
  - 自然语言处理
categories:
  - 教程
cover: /img/transformer/01-overview.png
top_img: false
description: 从 Encoder、Decoder 出发，梳理自注意力、FFN、残差连接、层归一化，并对比 BERT 与 GPT 的核心差异。
keywords: Transformer,Encoder,Decoder,BERT,GPT,自注意力,LayerNorm
---

{% note info %}
整理自学习笔记，区分 Encoder、Decoder，顺带理清 BERT 与 GPT 核心差异。文中图片已本地保存，方便长期访问。
{% endnote %}

![Transformer 结构示意](/img/transformer/01-overview.png)

## 一、编码器 Encoder

### 1. 自注意力机制

核心三向量：**Q 查询向量、K 索引向量、V 值向量**，代表 Token 携带的信息。

逻辑：将当前 Token 的 Q 与所有 Token 的 K 做点积，计算相互关联程度；依据关联程度，抽取对应 V 的信息。

> 一句话总结：**用 Q 找 K，提取 Value**

- Q 完成查询作用后就不再复用；真正留存使用的是 K、V。
- 位置编码会嵌入 K 中，参与 QK 点积运算，给模型引入序列位置信息。

### 2. 前馈神经网络层 FFN

对序列中**每一个位置单独做非线性变换（ReLU 激活）**，挖掘更复杂的语义特征。

### 3. 残差连接

解决两大问题：

1. **网络退化**：网络层数加深，训练集效果反而下降。深层网络很难学习恒等映射 \(H(x)=x\)，理想情况下新增层至少不降低原有效果。
2. **缓解梯度消失**：存在恒等直连路径，梯度可以直接回传浅层，深层网络参数能够正常更新。

{% note warning %}
残差连接**不能解决过拟合**。过拟合需要依靠 Dropout、正则化、数据增强等方法处理。
{% endnote %}

### 4. 层归一化 Layer Norm

统一每个 Token 内部多维度特征的数值分布，稳定模型训练。

#### BatchNorm vs LayerNorm

- **Batch Norm**：面向**同一个特征维度**做归一化，处理一批样本。

![BatchNorm 示意](/img/transformer/02-batchnorm.png)

- **Layer Norm**：面向单条样本的词向量做归一化，多用于 Transformer、RNN。

## 二、解码器 Decoder

1. **Masked 自注意力（掩码自注意力）**  
   增加掩码 Mask 约束。训练过程中，每个 token 只能看见它**前面的文本**，屏蔽未来位置 token 信息。从结构上实现逐词自回归生成，杜绝信息泄露。

2. **前馈神经网络层**  
   和编码器结构一致。

## 三、BERT & GPT 核心对比

### BERT｜Encoder-Only 双向理解模型

底层直接复用 Transformer 编码器。下游任务两种输出方式：

1. **Token-Level 词级别任务（命名实体识别 NER）**：取每个 token 位置输出向量。
2. **Sequence-Level 序列级别任务（文本分类、句子匹配）**：取开头特殊标记 `[CLS]` 的输出，用来整合整句语义。

预训练两大任务：

1. **MLM 掩码语言模型**：学习词语上下文语义
2. **NSP 下一句预测**：学习句子之间逻辑关系

> BERT 特点：**双向上下文，主打文本理解**

### GPT｜Decoder-Only 单向生成模型

依靠掩码自注意力做自回归，不断预测下一个 token。

> GPT 特点：**单向从左到右，主打文本生成**

## 四、简短小结

- **Encoder**：完整双向注意力，适合理解类任务，代表：BERT
- **Decoder**：带 Mask 的单向注意力，适合生成类任务，代表：GPT
- **Transformer 四大组件**：自注意力、FFN 前馈网络、残差连接、层归一化

{% note success %}
记忆口诀：BERT 看两边（理解），GPT 看左边（生成）；Encoder 全看见，Decoder 戴面罩。
{% endnote %}
