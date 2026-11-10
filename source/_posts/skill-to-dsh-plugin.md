---
title: 从 Skill 到 dsh 插件
date: 2026-11-10 09:52:01
updated: 2026-11-10 09:52:01
tags:
  - Skill
  - dsh
  - DeepSeek
  - 插件
  - Agent
categories:
  - 教程
cover: /img/cover-skill-to-dsh-plugin.jpg
top_img: false
description: 写好的 Skill 能不能直接变成 DeepSeek dsh 插件？先拿这个开发模板试一遍。
keywords: Skill,dsh,DeepSeek Harness,插件开发,Agent
---

现在写 skill 不算难事，基本人人都会。那能不能把 skill 直接转成 DeepSeek dsh 插件？我想试试这个：

{% note info %}
仓库地址：[liangdabiao/dsh-plugin-developer-skill](https://github.com/liangdabiao/dsh-plugin-developer-skill)
{% endnote %}

![dsh-plugin-developer 仓库介绍](https://cdn3.ldstatic.com/optimized/4X/e/1/c/e1c12251db62b9bb13ea13b35fe0c9779d8b6cb5_2_690x322.png)

![插件开发流程](https://cdn3.ldstatic.com/optimized/4X/2/0/9/209f982aa01b4f5bcfff2c2653450a5c37d1d586_2_690x344.jpeg)

![天气插件示例](https://cdn3.ldstatic.com/optimized/4X/1/d/a/1da8d65c6c6f286abec40ce78de1ab5e302e67e2_2_540x500.jpeg)

## 这个仓库是做什么的

dsh-plugin-developer 是一个写给 AI Agent 看的开发 Skill，带它从 0 到 1 走完 dsh 插件的开发、构建、安装和测试。内容基于 dsh 0.1.1-rc.2 和 dsh-openmaic 项目的实战经验，里面还放了一个已经在 Web 界面实测过的天气插件，作为整包参考案例。

## dsh 的插件化思路

dsh 的设计是「无特权内核、万物皆插件」。模型适配器、工具注册表、Agent 循环、网页界面，全都做成插件，内核只保留最少的调度能力。

对写 skill 的人来说，这个结构并不陌生：把一套流程和约束写成文件，交给 Agent 去执行。区别在于插件还要多走构建、安装、运行这几步。

## 怎么试

1. 先把仓库 clone 到本地。
2. 让 Agent 读仓库里的 SKILL.md，按里面的步骤往下做。
3. 从天气插件这个现成案例改起，先跑通，再换成自己的 skill。

Skill 和插件之间隔着的，主要是工程那一层：怎么打包、怎么注册、怎么在界面里验证。这个仓库把这几步都写清楚了，剩下的就是动手。
