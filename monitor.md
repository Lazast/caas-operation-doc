# Monitor

## prometheus架构及基本概念

### 架构图

![](/Users/rainf/Pictures/监控文档截图/架构图.png)



#### 基本架构

Prometheus生态包括了很多组件，它们中的一些是可选的：

主服务Prometheus Server负责抓取和存储时间序列数据
客户库负责检测应用程序代码
支持短生命周期的PUSH网关
基于Rails/SQL仪表盘构建器的GUI
多种导出工具，可以支持Prometheus存储数据转化为HAProxy、StatsD、Graphite等工具所需要的数据存储格式
警告管理器
命令行查询工具
其他各种支撑工具
多数Prometheus组件是Go语言写的，这使得这些组件很容易编译和部署。

#### 特征

Prometheus的主要特征有：

多维度数据模型
灵活的查询语言
不依赖分布式存储，单个服务器节点是自主的
以HTTP方式，通过pull模型拉去时间序列数据
也通过中间网关支持push模型
通过服务发现或者静态配置，来发现目标服务对象
支持多种多样的图表和界面展示，grafana也支持它

