---
title: Git 学习笔记：三个区域、常用命令与协作坑
date: 2026-09-21 21:57:00
updated: 2026-09-21 21:57:00
tags:
  - Git
  - 分支
  - 协作
categories:
  - 教程
cover: /img/cover-git-notes.png
top_img: false
description: 本地工作目录、暂存区、资源库，加上远程一共四个区域。日常记住 add、commit、push 这条链路，协作时避开 main 裸奔，冲突也不能全交给 AI。
keywords: Git,工作目录,暂存区,分支,协作
---

## 三个区域

Git 本地有三个工作区域：工作目录（Working Directory）、暂存区（Stage / Index）、资源库（Repository，也叫 Git Directory）。如果再加上远程仓库（Remote Directory），就可以分成四个工作区域。文件在这四个区域之间的转换关系如下：

![四个工作区域](/img/git-notes/01-areas.png)

## 创建工作目录与常用指令

工作目录一般就是你希望 Git 帮你管理的文件夹，可以是项目目录，也可以是一个空目录，建议路径里不要有中文。

日常使用记住下面这 6 个命令：`add`、`commit`、`push`、`fetch` / `clone`、`pull`、`checkout`。

![常用命令关系](/img/git-notes/02-commands.png)

分支相关再补这几条：

![分支命令与冲突](/img/git-notes/03-commands.png)

- `git branch 分支名`：创建分支
- `git branch -v`：查看分支
- `git checkout 分支名`：切换分支
- `git merge 分支名`：把指定分支合并到当前分支

分支冲突的原因是改了同一个文件的同一个位置。解决办法是手工合并；尽量避免多人同时改同一个文件。

## 协作里容易踩的坑

| 坑 | 规则 |
| --- | --- |
| main 分支长期不更新 | 每次开发前必须 `git pull origin main` |
| 本地写完代码不推远程，本地裸奔 | 功能分支要推远程，及时备份代码 |
| 直接往 main 合并 / push | main 受保护，只能网页 PR 合并，禁止本地合并到 main 后 push |
| 新建分支不知道绑定远程 | 首次 push 加 `-u`：`git push -u origin xxx` |
| AI 生成代码不看逻辑直接上测试 | AI 代码必须通读、本地跑通自测，再提交 |
| 冲突交给 AI 全权处理 | AI 仅辅助，人必须校验合并结果 |
