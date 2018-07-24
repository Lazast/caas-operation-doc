# OpenShift Web Console

## 登录

OpenShift Web Console域名在CaaS平台部署时可以商定，因此如果不确定域名是什么，可以询问平台的部署实施工程师。

在浏览器中，需要以 https://域名:8443的方式登录。

## 创建项目

点击屏幕右侧的【Create Project】就可以创建项目，创建时需要填入Name, display name（建议和name一致\) 以及 description。

![](.gitbook/assets/image%20%283%29.png)

## 管理项目

### 查看所有项目列表

创建好的项目，可以在右侧的项目列表里显示。但默认只会显示最新创建或更新的五个项目，当需要查看所有项目列表时，点击上图中的【View All】。点击后会进入项目列表展示页面，如下图

![](.gitbook/assets/image%20%284%29.png)

在该页面中，在上方的输入框中可以输入项目名称进行搜索。

### 项目基本维护

在上图中，点击项目右侧的竖排三个点，可以对项目进行基本的维护操作，如编辑项目中的用户成员角色信息，编辑项目的描述，以及删除项目。

注意：由于CaaS OMP已经提供了对项目的管理和维护，因此，除非有特殊原因，不建议管理员在这里进行操作。

## 租户项目管理

在上面的项目列表中，点击某个项目，即可进入该项目的管理页面，如下图

![](.gitbook/assets/image%20%286%29.png)

左边栏分别为概览，应用，构建，资源，存储，监控和编排商店。

在预览上方的下拉菜单中，可以选择其他项目进行跳转，便于管理员在不通项目间切换。

### Overview/概览

在橄榄页面中，点击某个应用，可以展开查看更多细节，如图

![](.gitbook/assets/image%20%287%29.png)

在单个应用的细节展示中，我们可以看到当前应用部署的版本号（如图中的caas-agamaha已经部署到了第5版），可以通过点击右侧圆圈旁的上下箭头来灵活的增减容器的副本数，对于没有配置Route的应用还将会在路由显示的地方展示创建Route的链接。点击右上方的竖排三个点，可以看到有deploy, edit, view logs三个选项，分别用来（基于当前的配置）重新部署一个新的版本，编辑应用的部分配置，以及查看应用的日志。

### Applications/应用

应用栏一共包括Deployments，Stateful Sets, Pods, Services, Routes等子栏。由于本平台不使用Stateful Sets功能，因此这里忽略。

#### Deployments

Deployments页面中主要包含应用的部署信息，基本展示页如下

![](.gitbook/assets/image%20%285%29.png)

可以查看到当前所有部署的应用的部署信息列表。可以看到每个应用当前部署的最新版本，健康状态，创建时间，以及自动部署触发类型。

点击单个应用部署信息后，会跳转到对应的详情页面，如下图

![](.gitbook/assets/image%20%282%29.png)

该页面中可以查看到部署版本历史，部署的基本配置信息，应用部署所使用的环境变量，以及部署事件。

点击左上角的Deploy可以手动触发新一轮的部署。

点击Actions下拉列表，可以看到Pause Rollouts/Resume Rollouts，Add Storage，Add AutoScaler，Edit Resource Limits，Edit Health Checks， Edit Yaml， Delete等操作。这些操作分别用于冻结/恢复部署，添加存储，添加自动扩缩容配置，编辑资源限制，编辑健康检查，编辑应用部署的YAML文件，以及删除。

根据实际经验，可以酌情选择进行操作，毕竟CaaS平台已经在用户界面封装了这些功能，一般不需要管理员在这里进行操作。

#### Pods

Pod界面中可以查看当前项目下的所有Pod，能够观察到Pod是否在运行（或者故障），Pod的运行时间。

![](.gitbook/assets/image%20%281%29.png)

在Pod实例的详情页面中

![](.gitbook/assets/image.png)

我们通常只关心Details和Logs，还有Terminal。Details中可以看到Pod所在节点的节点，便于底层在运维，而Logs和Terminals分别可以查看到Pod的日志，以及通过远程终端的方式登录到Pod。

#### Services

#### Routes

### Builds/构建

#### D

### Resources/资源

#### D

### Storage/存储

#### D

### Monitoring/监控

#### D

### Catalog/编排商店

#### D



