---
title: Docker 和 K8s：镜像、Compose、代码进不进镜像
date: 2026-09-15 15:27:00
updated: 2026-09-15 15:27:00
tags:
  - Docker
  - Kubernetes
  - docker-compose
  - Dockerfile
  - 容器
categories:
  - 教程
cover: /img/cover-docker-k8s.png
top_img: false
description: Docker 把程序和环境打包运行；Dockerfile 是清单，镜像是模具，Compose 一键拉起多服务。澄清代码是否打进镜像，以及 SSE、字体、Volume、多副本等部署坑。
keywords: Docker,K8s,Kubernetes,Dockerfile,docker-compose,Volume,镜像,容器
---

{% note info %}
Docker 是一款可以把程序和环境打包并运行的工具。K8s 是应用服务和服务器之间的中间层，通过 API 简化部署、扩容等运维。
{% endnote %}

![Docker 概念](/img/docker-k8s/01-docker-concept.png)

**Dockerfile**：从操作系统到应用服务启动要做哪些事，列清楚的一份清单。

![Dockerfile](/img/docker-k8s/02-dockerfile.png)

![镜像构建](/img/docker-k8s/03-image-build.png)

关系可以记成：

- **镜像**：做容器的模具
- **Dockerfile**：做模具的图纸，详细列出镜像怎么造出来

## docker-compose

- 单个 `docker run` = 开一台单独的小电脑
- `docker-compose` = 一键启动一整套（Web + 数据库 + Redis 等），并自动把它们连好网
- **一个配置文件，一条启动命令，管理一整套 Docker 服务**

容器之间可以建子网。Compose 用 yml 管理多个容器：怎么创建、怎么协同。Docker 会为每个 compose 文件自动建一个子网，还可以自定义启动顺序。

![Compose 与子网](/img/docker-k8s/04-compose-network.png)

## 常用命令

| 命令 | 作用 |
| --- | --- |
| `docker pull` | 从仓库下载镜像 |
| `docker image` | 查看已下载镜像 |
| `docker rmi` | 删除镜像 |
| `docker rm` | 删除容器 |
| `docker run 镜像名` | 用镜像创建并启动容器，每个容器有独立 ID |
| `docker run -d` | 后台运行，不阻塞当前窗口 |
| `docker run -p 80:80` | 端口映射：冒号前是宿主机端口，后是容器端口。容器网络默认与宿主机隔离，不映射就访问不到容器内服务 |
| `docker run -v 宿主机目录:容器内目录` | 挂载卷，宿主机与容器目录绑定，用于持久化 |
| `docker ps` | 查看进程状态（process status） |
| `docker start` / `docker stop` | 启动 / 停止已有容器；之前的端口映射、挂载、环境变量不用重写 |
| `docker inspect` | 查看端口映射、环境变量等配置 |
| `docker logs` | 看容器日志 |
| `docker exec -it` | 进入容器内命令行，与宿主机隔离 |

生产环境重启策略：

- `docker run -d --restart always`：容器一停就重启（崩溃、宿主机断电等）
- `docker run -d --restart unless-stopped`：手动停过的容器不再自动重启

## Kubernetes

![K8s 定位](/img/docker-k8s/05-k8s.png)

K8s 处在应用服务和服务器之间，暴露一系列 API，让部署、扩容等运维更省事。

## 澄清误区：镜像装的不止是依赖

Docker 镜像**不止是依赖环境**。镜像 = 基础操作系统 + 系统库 + 语言运行时 + 第三方依赖包 + **业务代码 / 配置**，整包打在一起。

### 模式 A：业务代码打进镜像（`COPY`）

Dockerfile 里 `COPY . .`，代码进镜像。

- 镜像内：OS + Python + pip 依赖（如 vllm、langchain）+ RAG 业务代码
- 启动：`docker run`，直接跑镜像里的代码

优点：

1. **环境 + 代码一体**。镜像到哪跑，就是哪一版代码，不会本地更新了线上还是旧的。
2. 交付简单：只交镜像，不用另拷代码。K8s 生产多数是这种。
3. 版本可控：镜像打 tag（v1.0、v1.1），本身就代表环境 + 代码版本。

缺点：

- 代码一改就要重新 build
- 镜像体积变大

### 模式 B：镜像只存依赖，代码用 Volume 挂进来

镜像只装 Python、依赖等，**不 COPY 业务代码**。启动时：

```bash
docker run -v /home/myrag-code:/app my-base-env-image
```

镜像里只有依赖；**代码在宿主机，容器读宿主机文件**。

优点：改代码不用 rebuild，重启容器就生效，适合本地调试。

生产慎用：

1. 镜像和代码版本脱钩，不知道跑的是哪份代码；宿主机误改、误删，容器直接挂。
2. 换机器要同步拷代码，容易漏文件。
3. K8s 不推荐大规模用宿主机挂业务代码，运维麻烦。

{% note tip %}
模型权重一般用挂载，不打进镜像：几十 GB 打进镜像会爆、分发也慢。业务代码通常很小，适合打进镜像。
{% endnote %}

### 什么时候用哪种

1. **本地开发调试**：Volume 挂代码，镜像只存依赖。
2. **测试、生产**：业务代码打进镜像，环境 + 代码一体交付。

前后端可以拆成不同镜像：

![前后端镜像拆分](/img/docker-k8s/06-frontend-backend-images.png)

## Docker 部署常见坑

1. **SSE 长连接**：常配 Nginx 反向代理；Nginx 默认缓冲、超时会打断 SSE。要单独改配置。本地没 Nginx，问题不易暴露。
2. **WeasyPrint PDF**：依赖系统库和中文字体；轻量镜像常缺，Dockerfile 要额外装，否则中文乱码或直接报错。
3. **持久化**：容器销毁，内部数据没了。MySQL、报告、日志必须挂 Volume，漏挂就丢业务数据。
4. **多副本扩展**：若用进程内 EventBus、内存缓存，Docker 很容易起多副本，但内存状态不跨实例共享，硬扩容会异常，要先改架构。

{% note warning %}
共性：本地一切正常，一进 Docker 才踩坑。本地自带系统库、没有 Nginx、单进程跑，容易把这些问题盖住。
{% endnote %}
