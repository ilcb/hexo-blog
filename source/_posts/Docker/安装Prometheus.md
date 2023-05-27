# 安装Prometheus

## 1.简介

普罗米修斯是一个开源系统，Prometheus 收集其指标并将其存储为时间序列数据，即指标信息与记录它的时间戳一起存储，以及称为标签的可选键值对

## 2.什么是指标
通俗地说，指标是数字度量，时间序列意味着随时间记录更改，用户想要测量的内容因应用程序而异。对于Web服务器，它可能是请求时间，对于数据库，它可能是活动连接数或活动查询数等。

指标在理解应用程序以某种方式工作的原因方面起着重要作用。假设您正在运行一个 Web 应用程序，发现该应用程序很慢，您将需要一些信息来了解您的应用程序发生了什么。例如，当请求数很高时，应用程序可能会变慢。如果您有请求计数指标，则可以找出原因并增加处理负载的服务器数量。

## 3.组件概述
### 3.1 Prometheus Server

收集和存储时间序列数据，Prometheus Server 是 Prometheus 组件中的核心部分，负责实现对监控数据的获取，存储以及查询。 Prometheus Server 可以通过静态配置管理监控目标，也可以配合使用 Service Discovery 的方式动态管理监控目标，并从这些监控目标中获取数据。其次 Prometheus Server 需要对采集到的监控数据进行存储，Prometheus Server 本身就是一个时序数据库，将采集到的监控数据按照时间序列的方式存储在本地磁盘当中。最后Prometheus Server 对外提供了自定义的 PromQL 语言，实现对数据的查询以及

### 3.2 Node-Exporter

用于暴露已有的第三方服务的 metrics 给 Prometheus。Exporter 将监控数据采集的端点通过 HTTP 服务的形式暴露给 Prometheus Server，Prometheus Server 通过访问该 Exporter 提供的 Endpoint 端点，即可获取到需要采集的监控数据。

### 3.3 Grafana

第三方展示工具，可以编写 PromQL 查询语句，通过 http 协议与 prometheus 集成

### 3.4 AlertManager

从 Prometheus Server 端接收到 alerts 后，会进行去除重复数据，分组，并路由到对方的接受方式，发出报警。常见的接收方式有：电子邮件，钉钉、企业微信，pagerduty等

### 3.5 Push Gateway

主要用于短期的 jobs。由于这类 jobs 存在时间较短，可能在 Prometheus 来 pull 之前就消失了。为此，这些 jobs 可以直接向 Prometheus server 端推送它们的 metrics。

## 4. Prometheus工作流程

### 4.1 指标采集

prometheus server 通过 pull 形式采集监控指标，可以直接拉取监控指标，也可以通过 pushgateway 做中间环节，监控目标先 push 形式上报数据到 pushgateway；

### 4.2 指标处理

prometheus server 将采集的数据存储在自身 db 或者第三方 db；

### 4.3 指标展示

prometheus server 通过提供 http 接口，提供自带或者第三方展示系统；

### 4.4 指标告警

prometheus server 通过 push 告警信息到 alert-manager，alert-manager 通过"静默-抑制-整合-下发"4个阶段处理通知到观察者

## 5. Prometheus指标分类
### 5.1 Counter

计数器类型，只增不减，如机器的启动时间，HTTP 访问量等。机器重启不会置零，在使用这种指标类型时，通常会结合rate()方法获取该指标在某个时间段的变化率。

### 5.2 Gauge

仪表盘，可增可减，如CPU使用率，大部分监控数据都是这种类型的。

### 5.3 Summary

客户端定义，Summary，Histogram 都属于高级指标，用于凸显数据的分布情况。

## 6. 安装prometheus

### 6.1 获取镜像

```bash
docker search prometheus
docker pull bitnami/prometheus
```

### 6.2 创建持久化路径

```bash
mkdir -p /app/prometheus/{config,data,rules}
```

### 6.3 写主配置文件

```bash
cd /app/prometheus/config
touch prometheus.yml
vi prometheus.yml
```



```bash
# my global config
global:
  # 设置抓取数据的时间间隔，间隔设置为每15秒一次。默认为每1分钟。
  scrape_interval: 15s
  # 设定抓取数据的超时时间，默认为10s
  scrape_timeout: 15s
  # 设置规则刷新，每15秒刷新一次规则。默认值为每1分钟。
  evaluation_interval: 60s

# 监控报警配置（需要额外安装 alertmanager组件）
alerting:
  alertmanagers:
  - static_configs:
    # 设定alertmanager和prometheus交互的接口，即alertmanager监听的ip地址和端口
    #- targets: ["localhost:9093"]

# 报警规则文件
rule_files:
  #- '/home/deploy/alertmanager/rules/*.yml'

# 普罗米修斯与抓取模块交互的接口配置
scrape_configs:
  # 一定要全局唯一, 采集 Prometheus 自身的 metrics
  - job_name: prometheus
    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
    static_configs:
      - targets: ['localhost:9090']
        labels:
          instance: prometheus
 
  - job_name: linux
    static_configs:
      - targets: ['192.168.1.109:9100']
        labels:
          instance: localhost
```

### 6.4 创建rule告警阈值

```bash
cd /app/prometheus/rules
touch rule.yml
vi rule.yml
```



```bash
groups:
- name: Hosts.rules
  rules:
  - alert: HostDown           
    expr: up{job=~"node-exporter|prometheus|grafana|alertmanager"} == 0
    for: 0m
    labels:
      severity: ERROR
    annotations:
      title: 'Instance down'
      summary: "{{$labels.instance}}"
      description: "主机: 【{{ $labels.instance }}】has been down for more than 1 minute"
## 注意==0为报警条件，当有服务宕机测会告警，如果测试需改为1
注：- alert: HostDown这个数据本机基本数据类，如果要监控其他的可以设置例外的名字
```

### 6.5 运行容器

将对应的rules 文件上传到 /app//prometheus/rules下

```bash
docker run -d \
           -u root \
           --restart=always \
           --name prometheus \
           -p 9090:9090 \
           -v /etc/localtime:/etc/localtime \
           -v /app/prometheus/config/prometheus.yml:/etc/prometheus/prometheus.yml \
           -v /app/prometheus/data:/prometheus \
           -v /app/prometheus/rules:/usr/local/prometheus/rules \
           bitnami/prometheus \
           --storage.tsdb.retention.time=100d \
           --config.file=/etc/prometheus/prometheus.yml
```

### 6.6 查看主服务部署状况

http://localhost:9090



## 7. 安装暴露节点node-exporter

负责收集 host 硬件和操作系统数据，将以容器方式运行在所有 host 上

### 7.1 获取镜像

```bash
docker search node-exporter
docker pull prom/node-exporter
```

### 7.2 启动容器

```bash
docker run -d \
           -p 9100:9100 \
           --name node-exporter \
           --restart=always \
           -v "/proc:/host/proc:ro" \
           -v "/sys:/host/sys:ro" \
           -v "/:/rootfs:ro" \
           prom/node-exporter
```

选项说明:

>-d, --detach=false， 指定容器运行于前台还是后台，默认为false
>
>--net="host"， 容器网络设置:
>       bridge        使用docker daemon指定的网桥
>       host            容器使用主机的网络
>       container:NAME_or_ID >   使用其他容器的网路，共享IP和PORT等网络资源
>       none          容器使用自己的网络（类似--net=bridge），但是不进行配置
>       
>--cap-add=[]， 添加权限，权限清单如下,SYS_TIME:设置实时时间：
>--pid="host"     将 PID 模式设置为主机 PID 模式。这将打开之间的共享 容器和主机操作系统的 PID 地址空间。 使用此标志启动的容器可以访问和操作其他 裸机计算机命名空间中的容器，反之亦然。
>-v                       给容器挂载存储卷，挂载到容器的某个目录(/宿主机目录:/docker目录)
>-e, --env=[]，   指定环境变量，容器中可以使用该环境变量,设置容器 ，HOST=ip，PORTO=端口
>/start 启动之后要执行的命令(可以是shell也可以是脚本)，如带/bin/bash则是解释器(注：脚本启动失败则容器启动失败)
>
>--restart=always 还可以设置自动重启容器

### 7.3 查看

http://localhost:9100/

## 8. 安装alertmanager

### 8.1 获取镜像

```bash
docker search alertmanager
docker pull prom/alertmanager
```

### 8.2 创建持久化目录

```bash
mkdir -p /app/prometheus/alertmanager/{config,template}
chmod -R 777 /app/prometheus/alertmanager
```

### 8.2 告警邮件配置

```bash
cd /app/prometheus/alertmanager/config
touch vi alertmanager.yml
vi alertmanager.yml
```



```bash
global:
  resolve_timeout: 5m 
  smtp_smarthost: 'smtp.qq.com:465'
  smtp_from: '976765156@qq.com'
  smtp_auth_username: '976765156@qq.com'  
  # qq邮箱授权码
  smtp_auth_password: 'gqpdxuougsqcbcbd'  
  smtp_require_tls: false
  smtp_hello: 'qq.com'

templates:
  # 指定预警内容模板
  - '/etc/alertmanager/template/email.tmpl'
  
route:
  group_by: ['alertname']
  group_wait: 5s
  group_interval: 5s
  repeat_interval: 5m
  receiver: 'email'
  
receivers:
  - name: 'email'
    email_configs:
    - to: 'exfoide@126.com'
      send_resolved: true
```

>
>
>全局配置（global）：用于定义一些全局的公共参数，如全局的SMTP配置，Slack配置等内容；
>
>模板（templates）：用于定义告警通知时的模板，如HTML模板，邮件模板等；
>
>告警路由（route）：根据标签匹配，确定当前告警应该如何处理；
>
>接收人（receivers）：接收人是一个抽象的概念，它可以是一个邮箱也可以是微信，Slack或者Webhook等，接收人一般配合告警路由使用；
>
>抑制规则（inhibit_rules）：合理设置抑制规则可以减少垃圾告警的产生
>
>

### 8.3 准备预警内容模板文件

```bash
cd /app/prometheus/alertmanager/template
touch email.tmpl
vi email.tmpl
```



```bash
{{ define "email.html" }}
<table border="1">
        <tr>
                <td>报警项</td>
                <td>实例</td>
                <td>报警阀值</td>
                <td>开始时间</td>
                <td>告警信息</td>
        </tr>
        {{ range $i, $alert := .Alerts }}
                <tr>
                        <td>{{ index $alert.Labels "alertname" }}</td>
                        <td>{{ index $alert.Labels "instance" }}</td>
                        <td>{{ index $alert.Annotations "value" }}</td>
                        <td>{{ $alert.StartsAt }}</td>
                        <td>{{ index $alert.Annotations "description" }}</td>
                </tr>
        {{ end }}
</table>
{{ end }}
```

### 启动容器

```bash
docker run -d --name=alertmanager \
    -p 9093:9093 \
    -v /etc/localtime:/etc/localtime:ro \
    -v /app/prometheus/alertmanager/config/alertmanager.yml:/etc/alertmanager/alertmanager.yml \
    -v /app/prometheus/alertmanager/template:/etc/alertmanager/template \
    prom/alertmanager
```

### 8.4 查看并等待告警

http://localhost:9093

## 9. 安装grafana

### 9.1 获取镜像

```bash
docker search grafana
docker pull grafana/grafana
```



### 9.1 创建持久化目录

```bash
mkdir -p /app/prometheus/grafana/{data,plugins,config}
chmod 777 -R /app/prometheus/grafana
```

### 9.2 启动容器

```bash
# 先临时启动一个容器
docker run --name grafana-tmp -d -p 3000:3000 grafana/grafana
# 将容器中默认的配置文件拷贝到宿主机上
docker cp grafana-tmp:/etc/grafana/grafana.ini /app/prometheus/grafana/config/grafana.ini
# 移除临时容器
docker stop grafana-tmp
docker rm grafana-tmp


docker run -d \
           -p 3000:3000 \
           --name=grafana \
           -v /etc/localtime:/etc/localtime:ro \
           -v /app/prometheus/grafana/data:/var/lib/grafana \
           -v /app/prometheus/grafana/plugins/:/var/lib/grafana/plugins \
           -v /app/prometheus/grafana/config/grafana.ini:/etc/grafana/grafana.ini \
           -e "GF_SECURITY_ADMIN_PASSWORD=admin" \
           -e "GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-simple-json-datasource,grafana-piechart-panel" \
           grafana/grafana
```