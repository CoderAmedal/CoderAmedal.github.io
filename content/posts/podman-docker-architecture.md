+++
date = '2026-04-14'
title = 'Podman与Docker的架构分析'
categories = ['技术分析']
tags = ['Podman', 'Docker', '容器']
+++

# Podman与Docker的架构分析

## 为什么要关注这个对比

Docker Desktop 从 2021 年起对大型企业收费，加上 Docker 本身的一些架构问题（守护进程单点故障、root 权限依赖），促使很多人开始考虑替代方案。Podman 是目前最成熟的替代者。

我从一个日常使用容器的开发者的角度，聊聊这两个工具的核心差异。

## 架构差异

### Docker：守护进程模型

```
用户 → docker CLI → dockerd（守护进程）→ containerd → runc → 容器
```

Docker 的核心问题是**所有操作都经过一个 root 权限的常驻守护进程（dockerd）**。这意味着：
- dockerd 挂了，所有容器的管理能力就没了
- dockerd 本身是 root 权限运行，是一个安全风险面
- 日志、网络等资源都受 dockerd 控制，难以用 systemd 等标准工具管理

### Podman：无守护进程模型

```
用户 → podman CLI → 直接 fork → conmon → runc → 容器
```

Podman 没有后台守护进程。每次运行容器时，Podman 直接 fork 出一个子进程，通过 conmon 作为容器的监控进程。这意味着：
- **没有单点故障**
- **可以用 systemd 管理容器**（配合 `podman generate systemd`）
- **天然支持 rootless**

## rootless 容器的实际意义

Docker 也有 rootless 模式（Docker 19.03+ 引入），但是通过用户态模拟实现的，存在一些限制。Podman 的 rootless 模式从一开始就是核心设计目标，实现得更加成熟。

rootless 容器意味着：
- 容器内的 root 映射到宿主机的普通用户，**即使容器被攻破，攻击者也只能获得普通用户权限**
- 多租户环境：每个用户可以在自己的命名空间中运行容器，互不干扰
- CI/CD 场景：不需要给 CI runner root 权限

## 命令兼容性

Podman 的设计目标之一就是**兼容 Docker CLI**。日常使用中，`alias docker=podman` 基本够用：

```bash
# 基本操作几乎一致
podman build -t myapp .
podman run -d -p 8080:80 myapp
podman ps
podman logs <container>
```

### Podman 独有的好功能

- **`podman generate systemd`**：为容器生成 systemd 服务文件，让容器像普通服务一样由 systemd 管理，这是我觉得最实用的功能
- **`podman play kube`**：直接用 Kubernetes YAML 定义运行容器，一站式从本地开发到 K8s 部署
- **`podman machine`**：在 macOS/Windows 上提供虚拟机支持（不需要 Docker Desktop）

## 如何选择

| 场景 | 推荐 |
|------|------|
| 个人开发、学习容器 | Podman（免费、rootless、systemd 集成） |
| 团队已有 Docker 成熟工作流 | 继续用 Docker，迁移有成本 |
| CI/CD 流水线 | Podman（不需要持久守护进程） |
| 需要 docker-compose 的项目 | `podman-compose` 替代或直接用 `docker-compose` |
| 生产环境容器编排 | Kubernetes/containerd，Docker/Podman 都是本地开发工具 |

## 一点经验

我从 Docker 切到 Podman 的体验总体是顺畅的，99% 的命令可以直接 alias。唯一遇到问题的是个别镜像的 `ENTRYPOINT` 写法依赖 Docker 特定的行为，偶尔需要微调。另外 Podman 的 `--network` 行为和 Docker 略有差异（Podman 默认不创建 bridge 网络），了解这一点就不会被坑。

总体来说，对于大多数开发场景，Podman 已经是一个可以直接替换 Docker 的选择。
