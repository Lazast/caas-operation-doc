# CaaSPortal

## CaaSPortal

## 简介

## 功能模块

```text
caas-agamaha: 最外层负载，负载前端hcenter和后端haproxy-haproxy
caas-hcenter：前端程序，处理用户请求
caas-haproxy：后端haproyx，负载后端caas-muddle，NFS，OpenShift和caas-omp
caas-muddle：后端java程序，主要提供用户容器相关操作接口
caas-omp：后盾java程序，主要提供用户相关管理的接口
caas-redis： 数据缓存
MySQL：数据持久化存储
```

除MySQL外，其他组件均为容器部署，容器环境变量信息如下

```text
组件                    变量名                            变量值                        备注
caas-agamaha         HCENTER_SERVER                    caas-hcenter    
                    HCENTER_PORT                    8080    
                    WEB_SERVER                        caas-haproxy    
                    WEB_PORT                        8888    
caas-hcenter        LANG                            C.UTF-8    
                    EGG_WORKERS                        1    
                    CAAS_BACKEND                    caas-haproxy    
                    CAAS_BACKEND_PORT                8888    
                    LOGO                            sextant    
caas-haproxy        WEB_SERVER                        caas-muddle    
                    WEB_PORT                        8080    
                    NFS_URL                            10.74.248.252    
                    NFS_PORT                        8080    
                    OS_BJ_URL                        os-console.caas.example.com    
                    LANG                            C.UTF-8    
                    OMP_SERVER                        caas-omp    
                    OMP_PORT                        8081    
caas-muddle            PATH                            /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin    
                    APPS_HOME                        /AppServer    
                    TZ                                Asia/Shanghai    
                    JAVA_HOME                        /usr/lib/jvm/java-8-openjdk-amd64    
                    CA_CERTIFICATES_JAVA_VERSION    20140324    
                    JAVA_OPTS                        -Xmx2048m -Xms2048m    
                    JAVA_VERSION                    8u111    
                    LANG                            C.UTF-8    
                    DB_URL                            jdbc:mysql://10.74.248.254:3306/hcpaas?characterEncoding=utf8&autoReconnect=true&allowMultiQueries=true    
                    REDIS_URL                        caas-redis.caasportal.svc.cluster.local    
                    REDIS_PORT                        6379    
                    vol_create_sleep                5000    
                    openshift_terminal_url            ws://portalapi.caas.example.com    
                    hb_info                            YWRtaW46Q2FhczEyMzQ1                harbor的admin:passwd的base64加密信息    
                    JAVA_DEBIAN_VERSION                8u111-b14-2~bpo8+1    
caas-omp            PATH                            /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin    
                    APPS_HOME                        /AppServer    
                    TZ                                Asia/Shanghai    
                    JAVA_HOME                        /usr/lib/jvm/java-8-openjdk-amd64    
                    CA_CERTIFICATES_JAVA_VERSION    20140324    
                    JAVA_OPTS                        -Xmx2048m -Xms2048m    
                    JAVA_VERSION                    8u111    
                    LANG                            C.UTF-8    
                    spring_datasource_druid_bmp_url    jdbc:mysql://10.74.248.254:3306/hcpaas_bmp?characterEncoding=utf8&autoReconnect=true&allowMultiQueries=true&serverTimezone=Asia/Shanghai    
                    spring_datasource_druid_portal_url    jdbc:mysql://10.74.248.254:3306/hcpaas?characterEncoding=utf8&autoReconnect=true&allowMultiQueries=true&serverTimezone=Asia/Shanghai    
                    spring_redis_host                caas-redis    
                    spring_redis_port                6379    
                    spirit_caasUrl                    http://caas-haproxy:8888    
                    JAVA_DEBIAN_VERSION                8u111-b14-2~bpo8+1
```

各个组件之间关系如下,右侧虚线框中为其他组件 ![](.gitbook/assets/caas-structure.png)

## 备注

服务无法正常访问

排查步骤

1. 检查客户端网络连接是否正常
2. 域名解析是否正确
3. 检查caas-agamaha：环境变量是否正确，caas-agamaha是否能跟caas-hcenter和caas-haproxy正常通信
4. 检查caas-hcenter是否启动正常，环境变量是否正确，能否跟caas-haproxy正常通信
5. 检查caas-haproxy是否启动正常，环境变量是否正确，能否跟环境变量中的caas-muddle，NFS，OpenShift和caas-omp服务器正常通信
6. 检查caas-muddle是否启动正常，环境变量是否正确，查看日志中是否有报错
7. 检查caas-omp是否启动正常，环境变量是否正确，查看日志中是否正常
8. 检查caas-redis是否启动正常
9. 检查mysql是否正常提供服务

用已创建命名空间的用户登录仍然提示输入命名空间 排查步骤

1. 检查caas-haproxy中关于caas-muddle的环境变量是否设置正确
2. 检查caas-muddle是否能正常提供服务

