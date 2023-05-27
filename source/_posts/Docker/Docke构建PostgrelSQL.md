---
layout: _post
title: Docke 构建 PostgrelSQL
date: 2023-04-01
tags: 
  - Docker
categories: 
  - Docker
---
## Docke 构建 PostgrelSQL

## 1.查询 postgresql 镜像

``````bash
docker search postgres
``````

![查询镜像](查询镜像.jpg)

## 2.拉取镜像

```bash
docker pull postgres
```

## 3.启动容器

```bash
docker run --name postgres \
           --restart=always \
           -e POSTGRES_PASSWORD=123456 \
           -p 5432:5432 \
           -v /app/postgres/data:/var/lib/postgresql/data \
           -d postgres
```

> run ：创建并运行一个容器；
> –name ：指定容器名称
> -e POSTGRES_PASSWORD=password，设置环境变量，指定数据库的登录口令为 password（password 自己设置）
> -p ：指定宿主机和 Docker 容器端口映射，冒号前为宿主机端口号，另一个是容器端口号。（Docker 的容器默认情况下只能由本地主机访问，即 A 主机上的容器不能被 B 主机访问，所以要做端口映射）（端口号 自己设置）
> -d postgres：指定使用 postgres 作为镜像

## 4.查看运行容器状态

```bash
docker ps
```

![查看运行容器状态](查看运行容器状态.jpg)

## 5.通过可视化工具连接数据库

![Datagrip连接数据库](Datagrip连接数据库.jpg)