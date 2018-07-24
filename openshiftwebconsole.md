# OpenShift Web Console

## 摘要

由于CaaS Portal已经封装了大量的OpenShift相关操作，因此通常来讲，管理员并不需要在OpenShift Web Console中进行操作。因此，本页内容将不会逐个讲解OpenShift Web Console的页面内容及操作，但会针对CaaS Portal进行一部分补充讲解。

虽然任何用户都可以登录OpenShift Web Console，但是我们只建议管理员登录。因此以下所有的内容都默认是以管理员身份登录的。

## 登录

OpenShift Web Console域名在CaaS平台部署时可以商定，因此如果不确定域名是什么，可以询问平台的部署实施工程师。

在浏览器中，需要以 https://域名:8443的方式登录。

## 选择项目

每次登录后都会进入到OpenShift Web Console的主页面，在这里，我们可以通过右上角的【View All】来进入到项目检索页面

![](.gitbook/assets/image%20%283%29.png)

由于是管理员，因此点击【View All】后跳转到【My Project】页面后能看到所有的项目。在该页面中，在上方的输入框中可以输入项目名称进行搜索。

![](.gitbook/assets/image%20%284%29.png)

在上面的项目列表页面中，点击任意的项目，即可进入到该项目的管理页面

![](.gitbook/assets/image%20%289%29.png)

我们可以看到左边中的应用，构建，资源，存储，监控等功能页面都是和CaaS Portal中的用户可操作资源及内容是相关的。

在预览上方的下拉菜单中，可以选择其他项目进行跳转，便于管理员在不同项目间切换。

## 单个项目的管理

以下内容，除非必要，我们都将只讲解如何查看。

虽然OpenShift Web Console提供了Overview，即概览页面，但是通常我们不会检查这个页面的信息，而是有针对的检查其他gong栏及其子页面。

### Applications/Deployments

Deployments页面中主要包含应用的deployments信息，基本展示页如下

![](.gitbook/assets/image%20%287%29.png)

可以看到当前所有部署的应用的deployments列表。可以看到每个应用当前部署的最新版本，健康状态，创建时间，以及自动部署触发类型。

点击单个deployment后，会跳转到对应的详情页面，如下图

![](.gitbook/assets/image%20%282%29.png)

该页面中，我们通常只需要关注【Environments】栏和【Deploy】按键。

Environments中包含了deployments的相关环境变量配置和configMap，以及secrets。

而对于某些集群服务相关的应用，在修改了相关配置后，我们可能需要通过【Deploy】来手动出发新一轮的部署。

### Applications/Pods

Pod界面中可以查看当前项目下的所有Pod，能够观察到Pod是否在运行（或者故障），Pod的运行时间。

![](.gitbook/assets/image%20%281%29.png)

在Pod实例的详情页面中

![](.gitbook/assets/image.png)

我们通常只关心Details和Logs，还有Terminal。Details中可以看到Pod所在节点的节点，便于底层在运维，而Logs和Terminals分别可以查看到Pod的日志，以及通过远程终端的方式登录到Pod。

### Applications/Services

在Service的详情页面

![](.gitbook/assets/image%20%285%29.png)

通常我们会关注【Details】栏下的【Type】属性和【Traffic】下的【Route/Node Port】，当Type是NodePort时，则说明这个Service是以TCP方式暴露的，相应的Router/Node Port就会显示出该服务在集群上暴露出来的端口。

### Applications/Routes

对于以HTTP/HTTPS方式暴露的服务，通常我们只会关心在Route详情页面点击【Actions】-&gt; 【Edit】后进入的编辑页面

![](.gitbook/assets/image%20%2813%29.png)

这个页面包含了域名，路径，Service端口等基本信息。而对于HTTPS，在勾选【Secure route】后，将展开

![](.gitbook/assets/image%20%2811%29.png)

我们向看到可以选择的TLS终点模式，非安全链接的控制策略，以及最重要的上传相关证书。

### Resources/Config Maps

某些平台服务会用到Config Map，对此可能会需要在Config Maps页面中进行调试。在Config Maps页面中

![](.gitbook/assets/image%20%288%29.png)

一般我们需要点击【Actions】-&gt; 【Edit】按钮，然后在展开的内容页面中，在文本框中编辑value，或者从本地上传新的配置文件

![](.gitbook/assets/image%20%286%29.png)

