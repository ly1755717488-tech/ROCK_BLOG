---
title: Docker 和 K8s：镜像、Compose、代码进不进镜像
date: 2026-09-15 15:27:00
updated: 2026-10-09 11:22:22
tags:
  - Docker
  - Kubernetes
  - docker-compose
  - Dockerfile
  - 容器
  - Service
categories:
  - 教程
cover: /img/cover-docker-k8s.png
top_img: false
description: Docker 把程序和环境打包运行；Dockerfile 是清单，镜像是模具，Compose 一键拉起多服务。澄清代码是否打进镜像，以及 K8s 里 Container / Pod / Service 的层级与集群内外通信。
keywords: Docker,K8s,Kubernetes,Pod,Service,ClusterIP,Dockerfile,docker-compose,Volume,镜像,容器
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

## Docker 容器和镜像的关系 + 挂载

- **镜像**：只读模板（代码、运行时、配置），不能运行；
- **容器**：镜像启动后的运行实例，镜像之上加一层可写层；多个容器共用同一个镜像。
- **挂载（volume / bind mount）**：把宿主机目录 / 存储挂载到容器内，实现数据持久化。容器删除，挂载卷数据还在。三种：bind 挂载（宿主机指定目录）、volume（Docker 托管）、tmpfs（内存挂载）。

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

## Docker 和 K8s 的关系

- Docker：**容器引擎**，负责打包镜像、本地创建 / 运行容器。
- K8s（Kubernetes）：**容器编排平台**。

> 一句话：Docker 是造容器、跑容器的工具；K8s 用来管理一大堆容器，做调度、扩缩容、自愈、服务发现。
>
> 类比：Docker = 汽车；K8s = 交通调度中心

Docker 造出来的「车」，在 K8s 里不会裸跑：它们会被装进档口（Pod），再挂到稳定窗口（Service）后面。下面把这套层级和通信方式摊开。

## Kubernetes

![K8s 定位](/img/docker-k8s/05-k8s.png)

K8s 处在应用服务和服务器之间，暴露一系列 API，让部署、扩容等运维更省事。

### 先理清层级：Container → Pod → Service

从最小运行单元往上：

- **容器（Container）**：最底层的进程隔离单元，真正跑业务代码（比如一个 Java / Node 进程），共享宿主机内核。
- **Pod**：K8s 的最小调度单元。一个 Pod 里可以有 1 个或多个容器（业务上多数是 1 个）。每个 Pod 有自己的 Pod IP，但 Pod 是临时的——销毁、重启、扩缩容时，IP 都会变。
- **Service**：服务抽象。它**不跑业务程序，也不是容器**，角色是「稳定访问入口 + 集群内负载均衡」。

### Service 和 Pod 怎么对上

Service 用**标签选择器（selector）**匹配一组 Pod，匹配到的都是它的后端实例：

- **1 个 Service 通常对应多个 Pod**：生产最常见。例如用户服务 3 个副本，就是 3 个 Pod，一起挂在 `user-service` 后面；请求打到 Service，再负载均衡到这 3 个 Pod。
- 后端 Pod 扩缩容、重启、故障替换时，Service 自动更新端点列表，对调用方透明。
- 特殊情况下，1 个 Pod 也可以同时被多个 Service 匹配。

通俗一点：

- 容器 = 后厨厨师，真正做菜（执行业务）
- Pod = 一个后厨档口（里面有一个或多个厨师）
- Service = 前台收银台 + 派单：有固定窗口地址（稳定 IP / 域名），顾客只到前台下单，前台再分给空闲档口；后厨换人，不影响前台怎么接待

### 服务怎么通信

集群内通信和集群外访问，都主要靠 **Service + DNS**。

**集群内（主流）：**

1. **ClusterIP + 内部 DNS**  
   每个 Service 有一个稳定的集群内虚拟 IP（ClusterIP）；CoreDNS 还会生成域名，形如 `service-name.namespace.svc.cluster.local`。服务之间用域名或 ClusterIP 调用，是最常用的方式。
2. **按场景选 Service 类型**  
   - Headless Service：无 ClusterIP，DNS 直接返回后端 Pod IP 列表，适合有状态服务（如数据库集群）直连  
   - NodePort / LoadBalancer：主要对外暴露，也可用于节点间访问
3. **Pod 间直连**  
   Pod 之间可以用 Pod IP 互通，但 IP 会随重建变化，业务里不推荐直接写死。

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
