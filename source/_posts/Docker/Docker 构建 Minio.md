---
layout: _post
title: Docker 构建 Minio
date: 2023-04-01
tags: 
  - Docker
categories: 
  - Docker
---

## Docker 构建 Minio

### 简介

MinIO是世界上最快的对象存储服务器，在标准硬件上，读写速度分贝为183GB/s 和 171GB/s，对象存储可以作为主要存储层，用于Spark，Presto，TensorFlow，H20.ai 以及替代产品等各种工作负载用于Hadoop HDFS

MinIO是一种高性能的分布式对象存储系统，它是软件定义的，可在行业标准硬件上运行，并且在Apache 2.0许可下，百分百开放源代码。

文档地址：https://docs.min.io/cn/

### 安装

#### 下载镜像

```bash
docker search minio/minio
docker pull minio/minio
```

#### 挂载数据及配置

```bash
mkdir -p /data/minio/{config,data}
```

#### 启动容器

```bash
docker run -p 9000:9000 -p 9090:9090 \
     --net=host \
     --name minio \
     -d --restart=always \
     -e "MINIO_ACCESS_KEY=minioadmin" \
     -e "MINIO_SECRET_KEY=minioadmin" \
     -v /data/minio/data:/data \
     -v /data/minio/config:/root/.minio \
     minio/minio server \
     /data --console-address ":9090" -address ":9000"
```

