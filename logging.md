# Logging



## 日志采集

openshift日志管理采用：EFK方案：

1. Elasticsearch
2. Fluentd
3. Kibana

**Fluentd:** 
读取宿主机上的/var/log/message和/var/lib/docker目录下的系统及日志信息，并格式化为json信息 
*部署在每个node上面*

**Elasticsearch:** 
开源分布式搜索和分析引擎，负责收到日志信息后，存储信息并建立索引

**Kibana：** 
提供图形界面供用户检索和分析

**Curator：** 
数据清理器，清理Elasticsearch中过期的数据。

**AuthProxy：** 
身份验证，实现web console单点登录