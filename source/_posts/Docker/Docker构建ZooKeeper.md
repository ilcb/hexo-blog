---
layout: _post
title: Docker 构建 ZooKeeper
date: 2023-04-01
tags: 
  - Docker
categories: 
  - Docker
---
## Docker 构建 ZooKeeper

## 1.查询 ZooKeeper 镜像

``````bash
docker search zookeeper
``````

![查询镜像](查询镜像.jpg)

## 2.拉取镜像

```bash
docker pull zookeeper
```

## 3.挂载配置目录

将 redis 的配置文件进行挂载，以配置文件方式启动 redis 容器。(挂载：即将宿主的文件和容器内部目录相关联，相互绑定，在宿主机内修改文件的话也随之修改容器内部文件）

- 数据挂载目录

  ```bash
  mkdir -p /app/zookeeper/data
  ```
- 配置挂载目录

  ```bash
  mkdir -p /app/zookeeper/conf
  ```
- 日志挂载目录

  ```bash
  mkdir -p /app/zookeeper/logs  
  ```

## 4.启动容器

```bash
docker run -d \
           --name zookeeper \
           --privileged=true \
           --restart=always \
           -p 2181:2181 \
           -v /app/zookeeper/data:/data \
           -v /app/zookeeper/conf:/conf \
           -v /app/zookeeper/logs:/datalog \
           zookeeper
```
### 参数解析

> -d：表示后台启动 zookeeper；
>
> –-name 给容器命名
>
> -p 2181:2181: 将宿主机 2181 端口与容器内 2181 端口进行映射，冒号之前为物理机端口
>
> -v 将宿主机目录或文件与容器内目录或文件进行挂载映射；

## 4.验证容器状态

```bash
docker exec -it zookeeper /bin/bash ./bin/zkServer.sh status
```

![容器状态](容器状态.jpg)
