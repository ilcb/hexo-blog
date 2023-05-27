---
layout: _post
title: Docker 构建 MongoDB
date: 2023-04-01
tags: 
  - Docker
  - Redis
categories: 
  - Docker
---
## Docker 构建 MongoDB

## 1.查询 MongoDB 镜像

``````bash
docker search mongo
``````

![查询镜像](查询镜像.jpg)

## 2.拉取镜像

```bash
docker pull mongo
```

## 3.挂载配置目录

将 redis 的配置文件进行挂载，以配置文件方式启动 redis 容器。(挂载：即将宿主的文件和容器内部目录相关联，相互绑定，在宿主机内修改文件的话也随之修改容器内部文件）

- 数据挂载目录

  ```bash
  sudo mkdir -p /app/mongodb/data
  ```
  
## 4.启动容器

```bash
docker run -itd \
           --name mongo \
           --restart=always \
           -v /app/mongodb/data:/data/db \
           -p 27017:27017 \
           --privileged=true \
           mongo \
           --auth 
```
### 参数解析

> -d：表示后台启动 mongo；
>
> –-name 给容器命名
>
> -p 27017:27017: 将宿主机 27017 端口与容器内 27017 端口进行映射，冒号之前为物理机端口
>
> -v: 将宿主机目录或文件与容器内目录或文件进行挂载映射，将宿主机的/app/mongodb/data 映射到容器的/data/db 目录，将数据持久化到宿主机，以防止删除容器后，容器内的数据丢失
>
> --auth: 需要密码才能访问容器服务

## 4.创建用户并设置密码

```bash
docker exec -it  mongo mongosh
Current Mongosh Log ID:	6427e62c783ab3e5c62806f3
Connecting to:		mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+1.8.0
Using MongoDB:		6.0.5
Using Mongosh:		1.8.0

test> db.createUser(
...     {
...         user:"root",
...         pwd:"123456",
...         roles:[{role:"root",db:"admin"}]
...     }
... );
{ ok: 1 }
test> db.auth('root', '123456')
{ ok: 1 }

```
