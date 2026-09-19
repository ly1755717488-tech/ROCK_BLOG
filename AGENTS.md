# ROCK_BLOG — AI 快速上手指南

给新对话窗口的 Agent 用：先读本文，再按「指令 → 改哪些文件 → 提交推送」执行。

## 项目是什么

| 项 | 值 |
|----|-----|
| 站点名 | Rock的博客（作者 Rock罗） |
| 栈 | Hexo 8 + Butterfly 5.7 |
| 仓库 | `https://github.com/ly1755717488-tech/ROCK_BLOG.git` |
| 分支 | `main` |
| 部署 | 推送 `origin/main` → Cloudflare Pages 自动构建（**不要**依赖 `hexo deploy`） |
| 工作区 | `e:\Hexo`（Windows / PowerShell） |

本地预览（可选）：`npm run server`。生产发布：**git commit + git push**。

---

## 目录地图（只动这些）

```
e:\Hexo\
├── AGENTS.md                 ← 本文（给 AI）
├── _config.yml               ← Hexo 核心（permalink、主题名）
├── _config.butterfly.yml     ← 菜单、样式注入、KaTeX、搜索
├── package.json              ← build/server/clean
├── scaffolds/                ← hexo new 模板（实际 front-matter 以现有文章为准）
├── source/
│   ├── _posts/               ← 博客正文 *.md
│   ├── notes/index.md        ← 「随笔」页（手写 HTML 卡片）
│   ├── toolbox/index.md      ← 「工具箱」页（手写 HTML 条目）
│   ├── about/ projects/ link/ categories/
│   ├── img/                  ← 封面与配图
│   │   ├── cover-*.png       ← 文章封面（首页卡片）
│   │   └── <topic>/NN-*.png  ← 文内配图
│   └── css/custom.css        ← 随笔/工具箱等自定义样式
└── themes/butterfly/         ← 主题源码，非必要勿改
```

**不要提交：** `除AI味的提示词.md`、`node_modules/`、`public/`、`db.json`、`.cursor/`、`.env*`。

---

## 用户口头指令对照表

| 用户大概会说 | 你要做的事 |
|--------------|------------|
| 「写成新博文 / 新博客，封面用这张图」 | 新建 `_posts/<slug>.md` + `img/cover-<slug>.png`，正文轻量除 AI 味 |
| 「补充进这篇博客 / 把内容加到某某文章」 | 改对应 `_posts/*.md`，更新 `updated`，配图进 `img/<topic>/` |
| 「写入随笔 / 随笔页」 | 在 `notes/index.md` **置顶**插入卡片（非 `_posts`） |
| 「放进工具箱」 | 在 `toolbox/index.md` **置顶**插入 `toolbox-item` |
| 「改标题 / 删掉某张随笔卡片」 | 只改 `notes/index.md` |
| 「公式显示不对」 | 优先改成可读明文/引用块；本站 KaTeX 对 `$$` 常不渲染 |
| 改完内容后 | **立即** commit + `git push origin main`（见文末） |

「随笔」有两种含义，按语境区分：

1. **随笔页** = `source/notes/index.md`（菜单：生活手记 → 随笔）
2. **博文分类** = front-matter `categories: [随笔]`（和 `教程` 并列）

---

## 工作流 A：新博文 + 封面

1. 封面图复制到：`source/img/cover-<slug>.png`（用户附带的 Cursor assets 图先 `Copy-Item` 过来）。
2. 新建：`source/_posts/<slug>.md`。
3. Front-matter 模板（照现有文章填）：

```yaml
---
title: 文章标题
date: YYYY-MM-DD HH:mm:ss
updated: YYYY-MM-DD HH:mm:ss
tags:
  - 标签1
  - 标签2
categories:
  - 教程   # 或 随笔
cover: /img/cover-<slug>.png
top_img: false
description: 一句话摘要，出现在首页卡片
keywords: 关键词1,关键词2
---
```

4. 文内图：`source/img/<topic>/01-xxx.png`，正文写 `![说明](/img/<topic>/01-xxx.png)`。
5. 正文可用 Butterfly：`{% note info %}` / `warning` / `tip` / `danger`。
6. 时间用 PowerShell：`Get-Date -Format "yyyy-MM-dd HH:mm:ss"`。
7. 文件名用英文 kebab-case（如 `code-aesthetics-ai-era.md`）。

---

## 工作流 B：给已有博文追加章节

1. `Glob` / `Grep` 定位 `source/_posts/*.md`（或根据用户截图标题搜 `title:`）。
2. 在合适标题下追加 Markdown；有图则放入对应 `img/<topic>/`。
3. 只改 `updated`，一般不动 `date`。
4. commit + push。

---

## 工作流 C：随笔页卡片（notes）

文件：`source/notes/index.md`  
结构：`<div class="notes-board">` 内一串 `<article class="note-card ...">`，**新卡片插在最上面**。  
同时更新 front-matter 的 `updated`。

### 长文随笔

```html
<article class="note-card note-card--essay">
  <header class="note-card__head">
    <span class="note-card__badge">随笔</span>
    <time class="note-card__time">YYYY-MM-DD</time>
  </header>
  <h2 class="note-card__title">标题</h2>
  <div class="note-card__body">
    <p>段落</p>
    <h3>小节</h3>
    <ul><li>...</li></ul>
  </div>
</article>
```

### 短小记

```html
<article class="note-card note-card--memo">
  <header class="note-card__head">
    <span class="note-card__badge">小记</span>
    <time class="note-card__time">YYYY-MM-DD</time>
  </header>
  <h2 class="note-card__title">标题</h2>
  <div class="note-card__body">
    <p class="note-card__lead">一句话要点</p>
  </div>
</article>
```

样式在 `source/css/custom.css`（`.notes-board` / `.note-card*`）。非必要不改 CSS。

---

## 工作流 D：工具箱条目

文件：`source/toolbox/index.md`  
在 `<div class="toolbox-list">` **顶部**插入：

```html
<article class="toolbox-item">
  <div class="toolbox-meta">
    <span class="toolbox-badge">学习</span>
    <span class="toolbox-tags">
      <span>标签1</span>
      <span>标签2</span>
    </span>
  </div>
  <h3 class="toolbox-title">
    <a href="https://..." target="_blank" rel="noopener noreferrer">名称</a>
  </h3>
  <p class="toolbox-desc">一句话说明。</p>
  <a class="toolbox-cta" href="https://..." target="_blank" rel="noopener noreferrer">
    <span>打开链接</span>
    <i class="fas fa-arrow-right" aria-hidden="true"></i>
  </a>
</article>
```

Badge 常用：`学习`、`推荐`。

---

## 写作风格（除 AI 味）

本地文件 `除AI味的提示词.md`（**勿提交**）要点：

- 不用中文弯引号 `“”`（U+201C/U+201D），用「」或直接不加引号
- 少用「不是 A，而是 B」
- 避免翻译腔、生造词、过度缩写
- 保留用户原意与口吻；技术文可略润色，随笔尽量贴原文

---

## 主题与配置（少动）

| 配置 | 文件 | 说明 |
|------|------|------|
| 菜单 | `_config.butterfly.yml` → `menu` | 首页/归档/分类/工具箱/项目/随笔/关于/友链 |
| 自定义 CSS | `inject.head` → `/css/custom.css` | 已接好 |
| 公式 | `math.use: katex`, `per_page: true` | `$$` 经常原样显示；公式改明文更稳 |
| 搜索 | `local_search` + `search.xml` | |
| 站点 url | `_config.yml` 仍可能是占位 | 线上以 Cloudflare 域名为准 |

---

## 常用命令

```powershell
# 本地预览
npm run server

# 生成静态站（一般不用；CF 会 build）
npm run build
npm run clean

# 发布时间
Get-Date -Format "yyyy-MM-dd HH:mm:ss"

# 复制封面（示例）
Copy-Item "<用户提供的图路径>" "e:\Hexo\source\img\cover-<slug>.png" -Force
```

Git（PowerShell，**不要用 bash heredoc / `&&`**）：

```powershell
git add source/_posts/xxx.md source/img/cover-xxx.png
git commit -m "Publish xxx with cover."
git push origin main
# 失败则隔几秒重试数次
```

---

## 发布规范（每次内容改完必做）

1. `git add` 相关文件（跳过 `除AI味的提示词.md`）
2. `git commit`（简洁英文或中文说明 why）
3. `git push origin main`；网络失败重试
4. 回复用户：提交 hash + Cloudflare Pages 会从 GitHub 自动部署

仓库名在 GitHub 上是 **ROCK_BLOG**。推送 `main` 即部署，无需再跑 `hexo deploy`。

---

## 快速自检

- [ ] 改的是 `source/` 下内容，而不是误改 `themes/butterfly` 大段逻辑
- [ ] 新博文有 `cover` + 对应图片文件已入库
- [ ] 随笔/工具箱是「置顶插入」，并更新了页面 `updated`
- [ ] 未把本地提示词、密钥、`.cursor` 提交上去
- [ ] 已 push `origin/main`
