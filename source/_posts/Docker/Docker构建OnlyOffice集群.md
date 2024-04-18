---
layout: _post
title: Docker构建OnlyOffice集群
date: 2023-05-27
tags: 
  - Docker
categories: 
  - Docker
---

# Docker构建OnlyOffice集群

# 1.选型

因为项目需要，需要在内网的环境下部署一套office的在线编辑软件。综合对比了一下网上的几种主流的解决方案，感觉onlyoffice比较合适。
对比如下，摘自http://www.dzzoffice.com/

| 解决方案         | 在线预览 | 在线编辑 | 本地依赖   | 内网使用 | 私有化部署 | 免费使用           |
| ---------------- | -------- | -------- | ---------- | -------- | ---------- | ------------------ |
| onlyoffice       | √        | √        | 无         | √        | √          | √ 最大20连接数     |
| collabora        | √        | √        | 无         | √        | √          | √ 最大20连接数     |
| MS Office Online | √        | √        | 无         | √        | √          | 预览免费编辑需购买 |
| google Doc       | √        | √        | 无         | ×        | ×          | √                  |
| 永中Office       | √        | ×        | 无         | ×        | ×          | √                  |
| pageOffice       | √        | √        | Office软件 | √        | √          | ×                  |

## 1.1 onlyOffice优点：

1. 文档还原度非常高，复杂文档还原度最好。
2. 安装部署方便，不需要独立服务器，支持docker容器安装，支持windows下部署。
3. 有社区版可免费使用（限制20连接数），同时提供多种版本可供企业购买使用，价格较便宜，按每服务器收取。
4. 有详尽开发文档，二次开发方便。
5. 提供贴牌服务。

## 1.2 OnlyOffice项目信息

onlyofficeAPI文档：https://api.onlyoffice.com/editors/basic
onlyoffice项目地址：https://github.com/ONLYOFFICE/Docker-DocumentServer
官方示例：https://api.onlyoffice.com/zh/editors/basic


# 2 依赖环境安装

## 2.1 安装Docker

### 2.1.1 安装Docker

```bash
curl -fsSL https://get.docker.com |bash -s docker --mirror Aliyun
```

### 2.1.2 检测Docker是否安装成功

```bash
docker ps
```

### 2.1.3 设置docker开机启动

```bash
systemctl enable docker.service
```

### 2.1.4 设置docker镜像源

docker镜像从官方dockerHub下载会很慢，可以设置国内镜像源来加速下载

**编辑如下文件:**

```bash
vi /etc/docker/daemon.json
```

**内容为**

```plain
{
  "registry-mirrors": ["https://awkamezj.mirror.aliyuncs.com"]
}
```

修改完成后重启docker

```plain
sudo systemctl daemon-reload

sudo systemctl restart docker
```

## 2.2 安装nginx

nginx在构建集群时需要用来做负载，也可以使用`HAProxy`来做负载均衡

### 2.2.1 docker安装nginx

```plain
#1.查询镜像
docker search nginx

#2.下载镜像
docker pull nginx
```

### 2.2.2 nginx服务配置及启动

容器中的文件内容是可以被修改的，但是**一旦容器重启，所有写入到容器中的，针对数据文件、配置文件的修改都将丢失**。所以为了保存容器的运行状态，执行结果，我们需要将容器内的一些重要的数据文件、日志文件、配置文件映射到宿主机上。

### 2.2.3 启动容器

```plain
docker run --name nginx -p 80:80 -d nginx
```

### 2.2.4 目录映射

|                    | **容器中路径**        | **宿主机中自定义映射路径** |
| ------------------ | --------------------- | -------------------------- |
| 存储网站网页的目录 | /usr/share/nginx/html | /data/nginx/html            |
| 日志目录           | /etc/nginx/nginx.conf | /data/nginx/conf/nginx.conf |
| nginx配置文件目录  | /var/log/nginx        | /data/nginx/logs            |
| 证书存放目录       | /etc/nginx/cert/      | /data/nginx/cert            |
| 子配置项存放处     | /etc/nginx/conf.d     | /data/nginx/conf.d          |

### 2.2.5 创建挂载目录

```plain
mkdir -p /data/nginx/{conf,conf.d,html,logs,cert}
```

### 2.2.6 复制配置到宿主机

将nginx配置文件copy到宿主机中

```plain
docker cp nginx:/etc/nginx/nginx.conf /data/nginx/conf
docker cp nginx:/etc/nginx/conf.d /data/nginx/
docker cp nginx:/usr/share/nginx/html/ /data/nginx/html/
docker cp nginx:/var/log/nginx/ /data/nginx/logs/
docker cp nginx:/etc/nginx/cert/ /data/nginx/cert/
```

### 2.2.7 停止并移除容器

```plain
docker stop nginx
docker rm nginx
```

### 2.2.8 启动容器

```plain
docker run -d \
           --name nginx \
           --restart=always \
           -p 80:80 \
           -p 443:443 \
           -v /data/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
           -v /data/nginx/html/:/usr/share/nginx/html/ \
           -v /data/nginx/logs/:/var/log/nginx/ \
           -v /data/nginx/conf.d/:/etc/nginx/conf.d \
           -v /data/nginx/cert/:/etc/nginx/cert \
           --privileged=true \
           nginx
```



# 3.重制OnlyOffice镜像

## 3.1 特殊需求

目前OnlyOffice官方镜像并不能满足业务需求，考虑对官方镜像进行改造并重新提供适合需求的镜像，的需求目前收集到的如下:

1.需要适配`WOPI`协议(官方镜像默认不支持，需要修改镜像配置)

2.OnlyOffice在线编辑器的logo字样为`ONLYOFFICE`，需要进行修改


## 3.2 docker镜像安装

```plain
#1.查找镜像
docker search onlyoffice/documentserver

#2.下载镜像
docker pull onlyoffice/documentserver
```

## 3.3 重构Docker镜像
以下为文件目录结构

├── Dockerfile
├── build-image.sh
├── header
│   ├── dark-logo_s.svg
│   ├── header-logo_s.svg
│   └── icons.svg
└── local.json

下面是相关文件及配置的一些说明

### 3.3.1 Dockerfile

```dockerfile
#指定基础镜像
FROM onlyoffice/documentserver:latest

LABEL maintainer = "xxx@126.com"
LABEL build_date = "2023-05-25"
LABEL comments = "1.开启WOPI协议;2.更换编辑器logo"

#镜像操作指令安装vim
RUN apt-get update
RUN apt-get install -y vim

#暴露程序端口80
EXPOSE 80 443

#更换编辑器logo
COPY ./header /var/www/onlyoffice/documentserver/web-apps/apps/common/main/resources/img/header/

COPY ./local.json /etc/onlyoffice/documentserver/

ENTRYPOINT ["/app/ds/run-document-server.sh"]
```



### 3.3.2 local.json

local.json 文件为OnlyOffice/documentserver的容器配置，来源于容器内的 `/etc/onlyoffice/documentserver/local.json`

```json
{
  "services": {
    "CoAuthoring": {
      "sql": {
        "type": "postgres",
        "dbHost": "localhost",
        "dbPort": "5432",
        "dbName": "onlyoffice",
        "dbUser": "onlyoffice",
        "dbPass": "onlyoffice"
      },
      "token": {
        "enable": {
          "request": {
            "inbox": false,
            "outbox": false
          },
          "browser": false
        },
        "inbox": {
          "header": "Authorization",
          "inBody": false
        },
        "outbox": {
          "header": "Authorization",
          "inBody": false
        }
      },
      "secret": {
        "inbox": {
          "string": "m0SnCOYxU9SoLOV2zGqKgiq6ARBBxc4i"
        },
        "outbox": {
          "string": "m0SnCOYxU9SoLOV2zGqKgiq6ARBBxc4i"
        },
        "session": {
          "string": "m0SnCOYxU9SoLOV2zGqKgiq6ARBBxc4i"
        }
      }
    }
  },
  "rabbitmq": {
    "url": "amqp://guest:guest@localhost"
  },
  "wopi": {
    "enable": true
  },
  "ipfilter": {
    "rules": [
        {
            "address": "*",
            "allowed": true
        }
    ],
    "useforrequest": true,
    "errorcode": 403
  }
}

```

#### 3.3.2.1 开启WOPI协议

开启WOPI协议，需要修改local.json，新增如下配置

```
"wopi": {
    "enable": true
  },
  "ipfilter": {
    "rules": [
        {
            "address": "*",
            "allowed": true
        }
    ],
    "useforrequest": true,
    "errorcode": 403
  }
```

### 3.3.3 header

header目录为onlyoffice编辑器样式中logo部分引用的图片，下边有2个文件`header-logo_s.svg`和`header-logo_s.svg`，为了实现对logo的自定义，需要修改这2个文件

### 3.3.4 build-image.sh

按照Dockerfile重新构建镜像，镜像名称为`exfoide/onlyoffice:7.3.3`，其中`7.3.3`是`onlyoffce/documentserver`官方镜像的版本，表示`exfoide/onlyoffice:7.3.3`是基于`onlyoffce/documentserver:7.3.3`版本构建而成

```bash
#/bin/bash
docker build --no-cache -f ./Dockerfile -t exfoide/onlyoffice:7.3.3 .
```

执行`build-image.sh`脚本后，docker中新创建了容器:

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22370594/1685000007506-2cbfc63a-39a2-4210-89b7-a63b2f4d9d1d.png)

## 3.4 替换logo方法(参考)

### 3.4.1 将容器内的logo图片拷贝到本地

```bash
sudo docker cp $(docker ps|grep onlyoffice/documentserver |awk '{print $1}'):/var/www/onlyoffice/documentserver/web-apps/apps/common/main/resources/img/header ~/Desktop/header
```

### 3.4.2 修改logo图片

### 3.4.3 修改完成后将图片信息拷贝到容器

```bash
sudo docker cp ~/Desktop/header $(docker ps|grep onlyoffice/documentserver |awk '{print $1}'):/var/www/onlyoffice/documentserver/web-apps/apps/common/main/resources/img/
```

# 4.OnlyOffice伪集群部署

本例中准备3台服务器，机器环境如下

| **服务器**   | **ip**       | **部署服务** | 端口 |
|-----------|--------------| ------------ | ---- |
| localhost | 127.0.0.1    | OnlyOffice   | 8091 |
| localhost | 127.0.0.1    | OnlyOffice   | 8092 |
| localhost | 127.0.0.1    | OnlyOffice   | 8093 |
| localhost | 127.0.0.1    | nginx        | 80   |

## 4.1 安装PostgreSQL

### 4.1.1 获取镜像

```bash
docker pull postgres
```

### 4.1.2 启动容器

```bash
docker run --name postgres -p 5432:5432 -e POSTGRES_PASSWORD=123456 --restart=always -v /data/postgres/data:/var/lib/postgresql/data -d postgres
           
```

## 4.2 准备PostgreSQL数据库环境

通过 <<**4.1安装PostgreSQL**>> 已经安装了PostgreSQL数据库，接下来需要初始化OnlyOffice数据库及数据表

### 4.2.1 创建数据库

```bash
docker exec -it $(docker ps|grep postgres |awk '{print $1}') bash
```

```bash
# 连接数据库
psql -h 127.0.0.1 -U postgres

#Password for user postgres:
输入: 123456

#创建用户
CREATE USER onlyoffice WITH PASSWORD 'onlyoffice';

#创建数据库
CREATE DATABASE onlyoffice OWNER onlyoffice;
```

### 4.2.2 创建数据表

```sql
CREATE TABLE IF NOT EXISTS "public"."doc_changes" (
"tenant" varchar(255) COLLATE "default" NOT NULL,
"id" varchar(255) COLLATE "default" NOT NULL,
"change_id" int4 NOT NULL,
"user_id" varchar(255) COLLATE "default" NOT NULL,
"user_id_original" varchar(255) COLLATE "default" NOT NULL,
"user_name" varchar(255) COLLATE "default" NOT NULL,
"change_data" text COLLATE "default" NOT NULL,
"change_date" timestamp without time zone NOT NULL,
PRIMARY KEY ("tenant", "id", "change_id")
)
WITH (OIDS=FALSE);

-- ----------------------------
-- Table structure for task_result
-- ----------------------------
CREATE TABLE IF NOT EXISTS "public"."task_result" (
"tenant" varchar(255) COLLATE "default" NOT NULL,
"id" varchar(255) COLLATE "default" NOT NULL,
"status" int2 NOT NULL,
"status_info" int4 NOT NULL,
"created_at" timestamp without time zone DEFAULT NOW(),
"last_open_date" timestamp without time zone NOT NULL,
"user_index" int4 NOT NULL DEFAULT 1,
"change_id" int4 NOT NULL DEFAULT 0,
"callback" text COLLATE "default" NOT NULL,
"baseurl" text COLLATE "default" NOT NULL,
"password" text COLLATE "default" NULL,
"additional" text COLLATE "default" NULL,
PRIMARY KEY ("tenant", "id")
)
WITH (OIDS=FALSE);

CREATE OR REPLACE FUNCTION merge_db(_tenant varchar(255), _id varchar(255), _status int2, _status_info int4, _last_open_date timestamp without time zone, _user_index int4, _change_id int4, _callback text, _baseurl text, OUT isupdate char(5), OUT userindex int4) AS
$$
DECLARE
	t_var "public"."task_result"."user_index"%TYPE;
BEGIN
	LOOP
		-- first try to update the key
		-- note that "a" must be unique
		IF ((_callback <> '') IS TRUE) AND ((_baseurl <> '') IS TRUE) THEN
			UPDATE "public"."task_result" SET last_open_date=_last_open_date, user_index=user_index+1,callback=_callback,baseurl=_baseurl WHERE tenant = _tenant AND id = _id RETURNING user_index into userindex;
		ELSE
			UPDATE "public"."task_result" SET last_open_date=_last_open_date, user_index=user_index+1 WHERE tenant = _tenant AND id = _id RETURNING user_index into userindex;
		END IF;
		IF found THEN
			isupdate := 'true';
			RETURN;
		END IF;
		-- not there, so try to insert the key
		-- if someone else inserts the same key concurrently,
		-- we could get a unique-key failure
		BEGIN
			INSERT INTO "public"."task_result"(tenant, id, status, status_info, last_open_date, user_index, change_id, callback, baseurl) VALUES(_tenant, _id, _status, _status_info, _last_open_date, _user_index, _change_id, _callback, _baseurl) RETURNING user_index into userindex;
			isupdate := 'false';
			RETURN;
		EXCEPTION WHEN unique_violation THEN
			-- do nothing, and loop to try the UPDATE again
		END;
	END LOOP;
END;
$$
LANGUAGE plpgsql;
```

## 4.3 Docker-compose启动集群

### 4.3.1 安装Docker-compose

```bash
#从Docker官方网站下载Docker Compose最新版本的二进制文件
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/bin/docker-compose

#授予Docker Compose二进制文件执行权限
sudo chmod +x /usr/bin/docker-compose 
```

### 4.3.2 Docker-Compose配置

以下为文件目录结构
├── .env
├── delete-container.sh
├── docker-compose.yml
└── start.sh
下面是相关文件及配置的一些说明

#### 4.3.2.1 .env

指定docker-compose文件中的环境变量

```.env
#指定数据库
DB_TYPE=postgres

#数据库ip
DB_HOST=127.0.0.1

JWT_ENABLED=false

#onlyoffice容器挂载到本地的目录
MAPPING_LOCAL_DIR=/data/onlyoffice
```

#### 4.3.2.2 docker-compose.yml

指定启动3个onlyoffice实例:onlyoffice-8091、onlyoffice-8092、onlyoffice-8093
```dockerfile
version: '3'
services:
  onlyoffice-8091:
    build:
      context: .
      dockerfile: ../docker-build/Dockerfile
    image: exfoide/onlyoffice:7.3.3
    container_name: onlyoffice-8091
    environment:
      - DB_TYPE
      - DB_HOST
      - DB_PORT=5432
      - DB_NAME=onlyoffice
      - DB_USER=onlyoffice
      # Uncomment strings below to enable the JSON Web Token validation.
      - JWT_ENABLED
      #- JWT_SECRET=secret
      #- JWT_HEADER=Authorization
      #- JWT_IN_BODY=true
    ports:
      - '8091:80'
    stdin_open: true
    restart: always
    stop_grace_period: 60s
    volumes:
      - ${MAPPING_LOCAL_DIR}/DocumentServer/logs:/var/log/onlyoffice
      - ${MAPPING_LOCAL_DIR}/DocumentServer/data:/var/www/onlyoffice/Data
      - ${MAPPING_LOCAL_DIR}/DocumentServer/lib:/var/lib/onlyoffice
      - ${MAPPING_LOCAL_DIR}/DocumentServer/db:/var/lib/postgresql
      - ${MAPPING_LOCAL_DIR}/DocumentServer/cache/files:/var/lib/onlyoffice/documentserver/App_Data/cache/files
      - /usr/share/fonts
    privileged: true

  onlyoffice-8092:
    build:
      context: .
      dockerfile: ../docker-build/Dockerfile
    image: exfoide/onlyoffice:7.3.3
    container_name: onlyoffice-8092
    environment:
      - DB_TYPE
      - DB_HOST
      - DB_PORT=5432
      - DB_NAME=onlyoffice
      - DB_USER=onlyoffice
      # Uncomment strings below to enable the JSON Web Token validation.
      - JWT_ENABLED
      #- JWT_SECRET=secret
      #- JWT_HEADER=Authorization
      #- JWT_IN_BODY=true
    ports:
      - '8092:80'
    stdin_open: true
    restart: always
    stop_grace_period: 60s
    volumes:
      - ${MAPPING_LOCAL_DIR}/DocumentServer/logs:/var/log/onlyoffice
      - ${MAPPING_LOCAL_DIR}/DocumentServer/data:/var/www/onlyoffice/Data
      - ${MAPPING_LOCAL_DIR}/DocumentServer/lib:/var/lib/onlyoffice
      - ${MAPPING_LOCAL_DIR}/DocumentServer/db:/var/lib/postgresql
      - ${MAPPING_LOCAL_DIR}/DocumentServer/cache/files:/var/lib/onlyoffice/documentserver/App_Data/cache/files
      - /usr/share/fonts
    privileged: true

  onlyoffice-8093:
    build:
      context: .
      dockerfile: ../docker-build/Dockerfile
    image: exfoide/onlyoffice:7.3.3
    container_name: onlyoffice-8093
    environment:
      - DB_TYPE
      - DB_HOST
      - DB_PORT=5432
      - DB_NAME=onlyoffice
      - DB_USER=onlyoffice
      # Uncomment strings below to enable the JSON Web Token validation.
      - JWT_ENABLED
      #- JWT_SECRET=secret
      #- JWT_HEADER=Authorization
      #- JWT_IN_BODY=true
    ports:
      - '8093:80'
    stdin_open: true
    restart: always
    stop_grace_period: 60s
    volumes:
      - ${MAPPING_LOCAL_DIR}/DocumentServer/logs:/var/log/onlyoffice
      - ${MAPPING_LOCAL_DIR}/DocumentServer/data:/var/www/onlyoffice/Data
      - ${MAPPING_LOCAL_DIR}/DocumentServer/lib:/var/lib/onlyoffice
      - ${MAPPING_LOCAL_DIR}/DocumentServer/db:/var/lib/postgresql
      - ${MAPPING_LOCAL_DIR}/DocumentServer/cache/files:/var/lib/onlyoffice/documentserver/App_Data/cache/files
      - /usr/share/fonts
    privileged: true

```

#### 4.3.2.3 start.sh
通过docker-compose启动集群容器

```bash
#/bin/bash
docker-compose down  --remove-orphans
docker-compose -p onlyoffice -f ./docker-compose.yml up -d
```

#### 4.3.2.4 delete-container.sh

通过docker-compose停止并删除集群容器

```bash
#/bin/bash
docker-compose -p onlyoffice down --remove-orphans
```


### 4.3.3 启动集群

执行start.sh 启动3个OnlyOffice实例

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22370594/1685002964009-e822b725-6ff3-48fe-b5be-7ad0d179d1c5.png)


## 4.4 部署ngnix

参照本文目录 <<**1.2 安装nginx**>>

### 4.4.1 配置负载配置

配置nginx负载

```bash
cd /data/nginx/conf.d
touch office.conf
vi office.conf
```

文件内容

```plain
upstream office {
  hash $remote_addr consistent;
  server 127.0.0.1:8091;
  server 127.0.0.1:8092;
  server 127.0.0.1:8093;
}

server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        proxy_pass http://office;

        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header X-Nginx-Proxy true;

        # WebSocker配置
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # 显示具体负载的机器的ip,X-Route-Ip随便命名
        add_header X-Route-Ip $upstream_addr;
        add_header X-Route-Status $upstream_status;
    }
}
```

# 5.OnlyOffice集成WOPI协议

官方文档: https://api.onlyoffice.com/zh/editors/wopi/

## 5.1 WOPI协议集成

通过<<**4.OnlyOffice伪集群部署**>>我们已经部署了3个OnlyOffice实例的集群，并且已经开启了WOPI协议

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22370594/1685154630255-ef36d531-a51f-4720-aa76-c333643f048d.png?x-oss-process=image%2Fresize%2Cw_1500%2Climit_0)

## 5.2 WOPI协议交互流程

![img](https://cdn.nlark.com/yuque/0/2023/jpeg/22370594/1684546162673-72f1dd88-b451-4b60-821e-5fb1e6a5f0f1.jpeg)

1. 浏览器请求编辑或查看Office文件
2. WOPI Host服务返回调用wopi所需的信息，如http://127.0.0.1/hosting/wopi/cell/edit?WOPISrc=http://127.0.0.1:8099/api/wopi/files/%E4%BA%A7%E5%93%81%E4%B8%AD%E5%BF%83%E5%91%A8%E6%8A%A520230503.xlsx, 其中http://127.0.0.1/hosting/wopi/cell/edit 表示需要通过wopi协议对excel类文件做编辑操作,

WOPISrc=http://127.0.0.1:8099/api/wopi/files/%E4%BA%A7%E5%93%81%E4%B8%AD%E5%BF%83%E5%91%A8%E6%8A%A520230503.xlsx 表示编辑器中需要展示的文件链接

1. 浏览器请求OnlyOffice文档服务对文件进行操作
2. OnlyOffice文档服务请求WOPI Host 服务查询文件信息，调用的WOPI协议接口为:[CheckFileInfo](https://api.onlyoffice.com/zh/editors/wopi/restapi/checkfileinfo)
3. WOPI Host 返回文件名称、大小、是否能修改、是否支持锁等信息
4. OnlyOffice文档服务请求WOPI Host 服务查询文件信息，调用的WOPI协议接口为:[GetFile](https://api.onlyoffice.com/zh/editors/wopi/restapi/getfile)
5. WOPI Host 返回二进制格式的完整文件内容
6. OnlyOffice文档服务将文件内容转换为 Office Open XML 格式, 返回给 js编辑器, js编辑器渲染Office文档内容
7. 用户在js编辑器对文档进行编辑等操作，在编辑过程中会调用一些WOPI协议接口，如[Lock](https://api.onlyoffice.com/zh/editors/wopi/restapi/lock), [UnLock](https://api.onlyoffice.com/zh/editors/wopi/restapi/unlock), [RefreshLock](https://api.onlyoffice.com/zh/editors/wopi/restapi/refreshlock)等
8. 用户编辑完成，保存文档
9. OnlyOffice文档服务请求更新文件内容，调用的WOPI协议接口为:[PutFile](https://api.onlyoffice.com/zh/editors/wopi/restapi/putfile)

### 5.2.1 协议核心理解

#### 5.2.1.1 认证阶段

该阶段上面的交互流程图中的step1、2步骤，需要Server事先提供Host页面，用户通过Browser请求Host页面(用户感知到的只有Host页面，而不是Office页面)，Server内部生成两个重要信息：

- 文档唯一ID：fileId
- 认证信息：access_token、access_token_ttl 然后内部通过调用/hosting/discovery接口，返回一个可以访问Office服务的urlsrc。具体交互时序图如下：

![img](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/baf9a4bd5745440ab20833f3681d7769~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

为何Browser不能跳过第1、2两步，直接进入Step3通过URL访问Office页面，这样Office客户端根据WOPISrc地址去Server执行file操作？理论上这样交互是可行，但是这么做相当于Server文档对任何外部请求都是可访问的，这对Server方是不能接受的，除非Server方信任并知道Client的地址情况下通过IP白名单机制来限制，但这样导致Server和Client存在耦合关系。

步骤1、2就是从Server获取文档权限信息，这样保护文件安全。Server在页面初始化时生成access_token，access_token看成是一个认证凭证，代表用户是否有权访问文档：

- 生成好的access_token返回给Browser，也就是上图step2返回内容之一
- 然后接下来Browser向Client发出的请求带上access_token，即step3发出的请求
- Client收到后在后续和Server交互时，会原样在请求URL参数里带上access_token，这样Server会先校验请求携带的access_token是否合法，如果是说明Office有权限打开文档

这个过程可以总结为Client和Server通过Browser这个中介来传递access_token。当然access_token值并不是一直有效，WOPI协议规定Server设置access_token_ttl，用来通知Client关于access_token值的有效期，默认是10h，超过这个时间后Client会取消本次会话。

注意：step3中access_token是怎么传递给Server？如果开始时直接在URL参数里携带，URL在公网传输，就有提前泄漏access_token的风险，也就是说Browser不能主动在URL里写入access_token，但是Client是可以的，因为access_token有效性是由Server决定，Client只管在请求里原样返回并由Server校验。所以WOPI协议的demo建议step3通过POST方法访问Office平台，access_token通过表单提交，这样在https传输协议下access_token传输至少是安全的。

#### 5.2.1.2 文档打开和编辑保存

从文档生命周期来看，文档操作包括打开、查看/编辑和保存流程。

查看和编辑是Office平台能力，打开文档请求Server获取文档内容，编辑文档后关闭页面，通知Server保存文档最新内容。WOPI协议定义了文件操作接口，其中CheckFileInfo、GetFile、PutFile接口实现上述打开和保存文件功能， GetFile、PutFile比较好理解，CheckFileInfo接口功能比较复杂，但是它决定Client端对文档的UI展示行为和后续允许的操作，这是由Server端提供属性信息决定，该接口返回包括：

- 文件基本信息：比如大小、文件展示名字等，
- 用户权限属性，常见属性包括：

- - UserCanWrite：指示当前请求用户是否有权限更改文件，这决定Client后续是否允许调用PutFile接口
- ReadOnly：文档是否只读
- UserCanRename：文档是否允许重命名，为false时Client在UI展示时不会提供重命名按钮

- 指示Client，Server支持哪些功能属性（capabilities properties），列举几个常见属性

- - SupportsGetLock：为true说明Server支持GetLock操作
- SupportsLocks：为true说明Server端支持Lock、Unlock等操作，Lock相关语义方面会提到
- SupportsRename：为true说明Server端支持对文档重命名操作
- SupportsUpdate：为true，说明Server支持更新文档操作 Server通知Client后，Client就可以在Server提供的属性决定后续操作是否允许调用，比如假设Server端通过checkFileInfo接口设置SupportsUpdate为false，那Client端知道Server不支持提供PutFile接口来更新文档，当文档关闭后Client不会把文档更新内容通知Server保存。

这些权限属性和access_token有什么关系？事实上，access_token充当对任何请求来源的认证机制，只有匹配Server端生成的值才是合法的，如果Server校验access_token不合法时，Client发出的任何操作请求被拒绝；而上述权限属性是在请求被认证通过的基础上，用来限制用户在Client端的操作能力。

#### 5.2.1.3 锁机制

如果用户有权限编辑文档内容，Server就要关心是否支持对文档的加解锁操作。锁的作用就像一把进入文档编辑的钥匙，只有拿到钥匙的合法用户才能编辑文档。锁机制判断文档是否允许多个用户同时编辑。

锁信息用锁id（lockId）标识，lockId的生命周期由Client负责控制，Server存储指定文档的lockId，在Client获取文档内容之前开始加锁，调用文档保存接口后解锁：

- 加锁操作时Client向Server发出lock请求，lockId放在请求头的X-WOPI-Lock字段中，Server获取该字段值后，判断文档是否被加锁或者校验请求lockId和已有是否匹配，匹配才能允许Client获取文档内容。
- 解锁操作时Client向Server发出unlock请求，同样Server检查lockId是否匹配，匹配成功才能释放锁

为何lockId还需要Server来存储，Server没必要感知到锁的存在？上面提到Server通过SupportsLocks属性通知Client是否支持对文档加解锁，Server虽然不负责lockId的生成，但是具体加解锁判断成功与否的策略由Server控制，举个例子，如果两个用户A、B操作同一个文档，A先加锁拿到lockID，B后续通过unlock请求解锁表示要提前保存文档，并且解锁请求头携带的lockId和A相同，这种情况下Client无法判断是否允许这样操作，Client生成的lockId和具体用户没有任何关联，只有Server有权决定不同用户拿到相同lockId时操作是否允许。

### 5.2.2 组成部分

按照WOPI协议交互流程图所示，整个onlyoffice集成过程中有3个角色，分别是:

- Web Office编辑器: 通过js实现office编辑器，需要在浏览器中展现
- WOPI服务器(Host): 实现[WOIPI REST API](https://api.onlyoffice.com/zh/editors/wopi/restapi)的文件管理系统
- WOPI客户端(Client): 理解为提供Office查看和编辑能力的平台，比如前面安装的Onlyoffice

#### 5.2.2.1 Web Office编辑器集成

Web Office编辑器需要一个前端页面来承载，传送门[主机页面](https://api.onlyoffice.com/zh/editors/wopi/hostpage)

其中actionUrl、token、tokenTtl需要传入，actionUrl指定需要Office编辑器执行的操作

举例如下:

http:{documentserverIP}/hosting/wopi/:documentType/:mode?WOPISrc=http://serverHost/wopi/files/:fileId：根据指定文档fileId，返回用户可以查看或者编辑文档的HTML内容，渲染器最终渲染该页面，域名和路径就是/hosting/discovery接口返回的urlsrc，其中

- documentserverIP表示部署only office/documentserver服务的地址
- documentType表示文档类型，包括xlsx、docx等
- mode表示查看或者编辑模式，包括show、edit。上图中step3通过该接口访问Office Client服务
- WOPISrc：协议约定的一个URL，需要Server提供，Browser根据该信息通知Client可以对Office文档执行WOPI规定的文档操作

#### 5.2.2.2 WOPI服务器(Host):

实现了[WOPI Rest Api](https://api.onlyoffice.com/zh/editors/wopi/restapi)的后端应用，需要实现对文件的查看、下载、锁定等api接口。

#### 5.2.2.3 WOPI客户端(Client)

理解为提供Office查看和编辑能力的平台，比如前面安装的onlyoffice/documentserver等，之前的步骤中已经安装好了onlyoffice/documentserver，并且开启了wopi协议，Client的所有配置及开发工作已经完成，后续操作中需要确保onlyoffice/documentserver保持运行
