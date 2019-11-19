## 项目概述

目前整个项目基于 docker 容器进行部署，由于容器数量较多，暂时使用 docker-compose 方便开发和测试。容器相关的配置信息全部放在 container-conf 中。项目分为三个部分：demo 系统——piggymetrics；Skywalking 相关的 OAP 即 UI；EFK 的数据收集及可视化方案。

启动了一个方便管理 docker 容器的服务 —— portainer

**通过 172.19.240.199:9000 即可进入界面**，使用以下信息进行登录

username: admin

password: WjwvBv2YrrChGFW

### piggymetrics

基于 fork 过来的 piggymetrics 项目改造，抛弃了 monitoring 模块和 sleuth 模块（即 Spring 提供的链路追踪方案），该项目的具体架构请参见 [piggymetrics](https://github.com/NJUPiggy/piggymetrics)

每个 demo 的 Java 服务中都启动了一个 SkyWalking 的 Java agent，用于采集和发送链路追踪数据

##### 关键服务端口映射

| container name | 端口映射  |
| -------------- | --------- |
| config         | 8888:8888 |
| registry       | 8761:8761 |
| gateway        | 80:4000   |

**通过 172.19.240.199:80 即可进入 demo 系统**

##### 注意：由于基于容器的原因，且还没将数据卷挂载到相关数据库下，该 demo 系统的数据并没有做持久化，当服务重启后，所有数据都会清空

### Skywalking

基于 Skywalking 对 demo 系统进行链路追踪以及给日志打上 TraceID 的标签。

Skywalking 模块分为两个部分，一个是 OAP 服务，负责采集和分析由 Java agent 发来的 trace 数据；另一个是 UI，提供链路追踪数据的可视化（后面会需要对该部分进行二次开发）

##### 关键服务端口映射

| container name | 端口映射  |
| -------------- | --------- |
| oap            | 8888:8888 |
| registry       | 8761:8761 |
| gateway        | 80:4000   |

**通过 172.19.240.199:8080 即可进入 Skywalking 的 UI 界面**

### EFK

Elasticsearch + Fluentd + Kibana

选用 es 作为 Skywalking 的数据存储以及日志的存储

Fluentd 作为容器内部日志采集的工具，可以将容器中的日志集中采集并过滤处理后转发到 es 进行存储

Kibana 主要是方便对 es 中的数据进行查询

##### 版本

| 服务名        | 版本          |
| ------------- | ------------- |
| Elasticsearch | 6.8.4         |
| Fluentd       | v1.6-debian-1 |
| Kibana        | 6.8.4         |

##### 关键服务端口映射

| container name | 端口映射             |
| -------------- | -------------------- |
| elasticsearch  | 9200:9200、9300:9300 |
| registry       | 24224:24224          |
| gateway        | 5601:5601            |

**通过 172.19.240.199:5601 即可进入 Kibana 控制台，查询日志和 Skywalking 的数据**

## QuickStart

### 服务器信息

IP: 172.19.240.199

username: root

password: word

root 身份进入服务器后

```shell
cd ~/container-conf
docker-compose down
docker-compose up -d
```

## 目前的问题

1.  demo 系统的新增的数据还没做持久化
2.  fluentd 和需要采集日志的服务的容器的启动顺序有依赖，fluentd 必须先成功启动（需要成功连接上 es，所以 fluentd 的启动是依赖 es 的），服务的容器之后启动，才会成功采集到日志，在搜集了相关资料后依然没能解决，当前的唯一办法就是在第一次成功启动后，再 restart 所有需要采集日志的容器
3.  还没有采集 demo 系统中 Java 服务的 metrics 数据，虽然数据的端点都暴露出来了，但是还没调研用哪项技术进行采集和存储