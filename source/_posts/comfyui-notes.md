---
title: ComfyUI 教学笔记：提示词权重、图生图与 ControlNet
date: 2026-09-14 16:45:00
updated: 2026-09-14 16:45:00
tags:
  - ComfyUI
  - Stable Diffusion
  - ControlNet
  - 提示词
  - 图生图
categories:
  - 教程
cover: /img/cover-comfyui.png
top_img: false
description: ComfyUI 实用笔记：反向提示词是权重调节而非彻底根除；提示词按质量、主体、氛围排序；图生图降噪不要拉满；ControlNet 的线条、软边缘、OpenPose、Depth。
keywords: ComfyUI,Stable Diffusion,反向提示词,K采样器,降噪,ControlNet,OpenPose,Depth
---

反向提示词并不能把不想要的东西从画面里彻底抹掉，它调的是画面中的权重。要完全根除，还需要别的操作。

## 扩散模型

在 Stable Diffusion 这类 AI 生成工具里，用来生成图像的核心模型就是**扩散模型（Diffusion Model）**。

- 核心原理：逐步向图像中添加噪声，再通过反向过程预测并去除噪声，最终生成高质量图像。
- Qwen、SeedVR2 等模型，本质上也都是基于扩散 / 流匹配机制的变体，只是架构和优化方向不同。

![扩散模型示意](/img/comfyui/01-diffusion.png)

## 提示词书写顺序

写提示词时：先写质量词汇（杰作、高质量、极致的细节），再写主体，最后写氛围词汇。

{% note tip %}
越靠前的词汇权重越大，整体画面里的占比也更大。
{% endnote %}

![提示词顺序与权重](/img/comfyui/02-prompt-order.png)

## 图生图：K 采样器的降噪

图生图时，K 采样器里的降噪不能设成 `1.00`。那样等于给原图铺满噪点再去噪，原图结构会丢光。应设成 `0.6` 一类数值，按一定比例降噪，才能保住和原图的相似度。

![图生图降噪](/img/comfyui/03-img2img-denoise.png)

## ControlNet 效果

常用：线条、软边缘、OpenPose、Depth。

![ControlNet 效果](/img/comfyui/04-controlnet.png)

### OpenPose

可以先做骨骼图分析，用来稳定人物的姿势、姿态。

![OpenPose 骨骼控制](/img/comfyui/05-openpose.png)

### Depth

让图片有纵深感和空间感。

![Depth 纵深控制](/img/comfyui/06-depth.png)

## 调用别人的模型

借鉴别人的工作流时，会给出相关参数参考，再按自己的目标做微调。

## 精准视频或项目反推提示词（Gemini）

### `VideoAnalysisService`（视频解析）

- **视频处理核心**：FFmpeg（用 FFmpeg.AutoGen / FFmpeg.NET 等 .NET 绑定库调用）
  - 解封装、解码、拆帧
  - 提取关键帧、按间隔抽帧
- **AI 分析**：
  - 调用大模型 API（通义千问、火山引擎）做镜头识别、场景理解
  - 封装在 `Infrastructure/AI` 层，用 `HttpClient` 或 SDK 发起请求
- **图像数据处理**：SixLabors.ImageSharp 或 SkiaSharp 做帧格式转换、缩放、裁剪
