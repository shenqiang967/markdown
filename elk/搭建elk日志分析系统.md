



# 开篇介绍

ELK 是三个开源项目的首字母缩写，这三个项目分别是：Elasticsearch、Logstash 和 Kibana。

- Elasticsearch 是一个**搜索和分析引擎**。
- Logstash 是**服务器端数据处理管道**，能够同时从多个来源采集数据，转换数据，然后将数据发送到诸如 Elasticsearch 等存储库中。
- Kibana 则可以让用户在 Elasticsearch 中**使用图形和图表对数据进行可视化**。

Elasticsearch 的核心是搜索引擎，所以用户开始将其用于日志用例，并希望能够轻松地对日志进行采集和可视化。有鉴于此，Elastic 引入了强大的采集管道 Logstash 和灵活的可视化工具 Kibana。

ELK日志系统数据流图如下：

![image-20210813092957476](C:\Users\18055\Desktop\笔记\elk\搭建elk日志分析系统.assets\image-20210813092957476-16288181998761.png)

# elasticsearch

> 参考文档：https://www.elastic.co/guide/en/elasticsearch/reference/index.html

## 安装

+ 下载：https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.14.0-linux-x86_64.tar.gz

+ 解压：tar -xvf elasticsearch-7.14.0-linux-x86_64.tar.gz

+  ```she
   cd elasticsearch-7.14.0/bin
   # 设置后台启动  -p 参数 记录进程ID为一个文件 -d 后台运行
   ./bin/elasticsearch -p /tmp/elasticsearch-pid -d
   ```

### <font color="red">错误</font>

1.can not run elasticsearch as root

![image-20210813095303374](C:\Users\18055\Desktop\笔记\elk\搭建elk日志分析系统.assets\image-20210813095303374-16288195847252.png)

> 原因是不能在root用户下启动es

```shell
#新增组和用户
groupadd es
useradd es -g es
passwd es
#授权目录 sq是目录
chown -R es:es sq
#更改目录权限命令，sq是目录
chmod -R 755 sq
#切换用户
su es
#修改配置文件
vim sq/elasticsearch-7.14.0/config/elasticsearch.yml
```

```yaml
# The primary way of configuring a node is via this file. This template lists
# the most important settings you may want to configure for a production cluster.
#
# Please consult the documentation for further information on configuration options:
# https://www.elastic.co/guide/en/elasticsearch/reference/index.html
#
# ---------------------------------- Cluster -----------------------------------
#
# 集群名称，唯一不可重复
#
cluster.name: tzhy-es-cluster
#
# ------------------------------------ Node ------------------------------------
#
# 节点名称
#
node.name: tzhy-es-31-9206
#
# Add custom attributes to the node:
#
#node.attr.rack: r1
#
# ----------------------------------- Paths ------------------------------------
#
# 数据存放目录
#
path.data: /usr/sq/data
#
# log日志输出目录
#
path.logs: /usr/sq/logs
#
# ----------------------------------- Memory -----------------------------------
#
# Lock the memory on startup:
#
#bootstrap.memory_lock: true
#
#
# ---------------------------------- Network -----------------------------------
#
# 
# 
#节点的白名单，0.0.0.0默认全开放
#network.host: 192.168.208.31
#
# 
# 给节点设置端口号
#
http.port: 9206
#节点对外暴露的对象
transport.tcp.port: 9400
#
# For more information, consult the network module documentation.
#
# --------------------------------- Discovery ----------------------------------
#
# 节点发现
# The default list of hosts is ["127.0.0.1", "[::1]"]
#
discovery.seed_hosts: ["192.168.208.31:9400", "192.168.208.32:9400"]
#
# 主节点的候选节点组，与node-name同名:
#
cluster.initial_master_nodes: ["tzhy-es-31-9206"]
#
# For more information, consult the discovery and cluster formation module documentation.
#
# ---------------------------------- Various -----------------------------------
#
# Require explicit names when deleting indices:
#
#action.destructive_requires_name: true

```

