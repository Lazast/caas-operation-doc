# NetworkRelated

## Networking Related

### 基础铺垫

OpenShift的网络实现依赖于:

> * haproxy
> * iptables
> * route
> * OVS
> * dnsmasq

与网络相关的OpenShift 资源主要有:

> * Pod
> * Service:
>
>   Service是內间的Pod负载均衡，在实际业务场景中，常常不会直接访问Pod，而是访问Service;
>
> * Router/Route:
>
>   Router实际上是个OpenShift內建的运行着haproxy的Pod，对于每一个以http/https方式暴露的Service，都需要配置一条Route，其本质是ACL规则，将请求的URL映射到Service后面的Pods;

#### 基础网络拓扑

OpenShift网络基本拓扑:

```text
.    
                        +----------------+
                        |  Loadbalancer  |
                        +-------+--------+
                                |
                                |
                          +-----+-------------+
                         +------+------------+|
                        +-------+-----------+||
                        |      ethX         |||
                        |       |    /-URL  ||| 
                        |     haproxy       ||| 
                        |       |    \-POD  |||
                        |      tun0         |||
                        |       |           ||+
                        |      ethY         |+
                        +-------+-----------+
                                |
                           +----+-----+
                           |  VXLAN   |
                           +-+------+-+        +------ Remote TCP
            Remote TCP       |      |          |
                |            |      +----------+------------+
                |            |                 |            |
            +---+------------+---+         +---+------------+---+
            |  ethY        ethX  |         |  ethY        ethX  |
            |   |            |   |         |   |            |   |
            |   +..iptables  |   |         |   +..iptables  |   |
            |   |            |   |         |   |            |   |
            |  tun0---------OVS  |         |  tun0---------OVS  |
            |                |   |         |                |   |
            |              veth  |         |              veth  |
            |                |   |         |                |   |
            |               Pod  |         |               Pod  |
            +--------------------+         +--------------------+
```

#### 流量简述

南向HTTP/HTTPS流量简述:

> * 外部请求经过外部负载均衡到达Master节点后，由Master的网卡接收；
> * 由于Router Pod的网络模式是hostNetwork，并且监听在80、
>
>   443端口，因此外部的HTTP、 HTTPS请求会被Router
>
>   Pod接收，并有haproxy处理，haproxy通过ACL匹配到与请求的URL所对应的Service后端的Pods，然后利用负载均衡算法，将请求转发到具体的某个Pod；
>
> * 请求在haproxy处理后，在由Master的路由，由tun0虚拟网卡发出，从而进入到OVS网桥，经OVS流表处理后，通过VXLAN隧道，发向目标Pod所在的计算节点；
> * 在目标计算节点上，请求由网卡接收后，进入OVS网桥，在经过流表处理后，从Pod对应的veth虚拟网卡流出OVS网桥，进入Pod。

南向TCP流量简述:

> * 这种情况下，请求的目标IP是集群的任意节点IP，而以外部负载均衡节点IP为目标的请求，随后也会被转发到集群的某一个节点，因此本质是一样的；
> * 请求到达集群的某一节点后，由网卡接收，之后由iptables进行处理，变为以Service后端的某个具体Pod
>
>   IP为目标的请求；
>
> * 在经过节点路由后，请求从tun0进入到OVS网桥，进而由流表进行处理；
> * 如果目标Pod在节点本地，则会从veth虚拟网卡流入Pod，而如果Pod在其他计算节点，则请求通过VXLAN隧道发向目标节点，在目标节点接收后，同理通过OVS，veth进入Pod。

东西向访问Pod流量简述:

> * 以Pod为目标IP的请求，在从源Pod的veth网卡进入到OVS网桥，经过流表处理后，发向目标Pod（如果目标Pod不在当前节点，则需要进过VXLAN隧道），在请求准备进入目标Pod的veth网卡前，OVS流表会做VNID的检查，即判断目标Pod与源Pod在否同一namespace，是则放行流量，否则就会丢弃。

东西向访问Service流量简述:

> * 以Service为目标的请求，在进入OVS网桥后，会先从tun0虚拟网卡流出，进入到计算节点进行iptables的处理，iptables会Service到Pod的负载代理，将请求变为Pod到Pod的，之后在节点路由的作用下请求会再次从tun0进入到OVS网桥，之后再发向目标Pod，同理，也会早VNIC的检查。

北向流量简述:

> * Pod以集群外的域名、IP为目标进行访问时，属于这一类情况；
> * 首先当对域名进行请求时，DNS请求会发往所在节点，由节点本地的dnamasq进行代理，解析出对应的目标IP，之后Pod再针对所获取的IP进行请求；
> * 请求在进入OVS网桥后，会从tun0虚拟网卡流出，借助节点路由以及iptalbes的SNAT的处理后，从节点网卡发出。

### 网络排查常用命令工具

#### tcpdump

tcpdump命令在日常运维以及故障排查中，常用来检查数据包请求或应答是否到达某处的问题。常见用法如下:

```text
# 检查ping包或者ARP包是否到达某网口
tcpdump -tni ethX icmp or arp

# 检查某网口上能否收到HTTP请求
tcpdump -tnni ethX tcp and dst port 80

# 检查某网口上能否收到来自某一IP（如192.168.1.100) 的HTTPS请求
tcpdump -tnni ethX tcp and src host 192.168.1.100 and dst port 80

# 检查某网口上能否收到来自某一地址段（如192.168.2.0/24) 的访问3306请求
tcpdump -tnni ethX tcp and src net 192.168.2.0/24 and dst port 3306

# 检查某节点（192.168.3.101）上是否有数据包通过VXLAN隧道发向某一目标节点（192.168.3.102）
tcpdump -tnni ethX src host 192.168.3.101 and dst host 192.168.3.102 -P out | grep \
    VXLAN

# 检查某节点（192.168.3.101）上Pod（10.128.4.100）是否有数据包通过VXLAN隧道发向
# 某一目标节点（192.168.3.102）
tcpdump -tnni ethX src host 192.168.3.101 and dst host 192.168.3.102 -P out | grep \
    10.128.4.100

# 检查Pod发出的DNS请求
tcpdump -tnni vethXXX port 53

# 检查某网卡上收到的HTTP请求的headers
tcpdump -tnni ethX tcp and port 80 -A
```

以上的例子中，关于tcpdump的参数可以用 _tcpdump -h_ 来查询，如果环境上没有tcpdump命令，则需要通过 _yum install -y tcpdump_ 来安装。

ethX在实际环境中可能包含三类端口:

> * 节点的物理网卡，如eth0, bond0等
> * tun0 OVS虚拟网口
> * veth虚拟网口 （注意：当Pod的网络模式是hostNetwork时，则没有对应的veth虚拟网卡。） 可以用如下命令来获取某个Pod（IP如10.128.0.211\)的veth:
>
>   ```text
>   # 根据实际情况，将10.128.0.211替换为实际IP，注意：需要在Pod所在节点执行该命令
>   echo "10.128.0.211" | xargs -I{} ovs-ofctl -O OpenFlow13 dump-flows br0 \
>       "table=70,ip,nw_dst={}" | cut -d ':' -f 3 | cut -d '-' -f 1 | \
>       xargs -I{} printf "%d" {} | xargs -I{} ovs-vsctl find interface ofport={} | \
>       awk -F'"' '/name /{print $2}'
>   ```

关于VXLAN的抓包，通过计算UDP包的数据偏移位与协议等信息是可以指定只抓取VXLAN包的，但是实际中通过grep方法来的更容易且基本不会出错，因此建议通过grep来检查VXLAN包。

#### nslookup

nslookup命令在日常运维中常用来检查某一域名能否被解析，或者能获得正确的解析。根据个人偏好用dig命令也可以。nslookup命令的常见用法如下:

```text
# 检查 example.abc.com的域名能否被解析
nslookup example.abc.com

# 检查指定DNS服务器（10.72.8.10）能否解析example.abc.com域名
nslookup example.abc.com 10.72.8.10

# 检查指定IP所对应的域名
nslookup 192.168.100.200
```

#### curl

curl命令在日常运维中常用来检查某个URL是否可达，以及检查返回结果。这里只列举一些curl的一些参数:

> * -H: Pass custom header\(s\) to server，如-H "Accept:
>
>   application/json, _/_"
>
> * -i: Include protocol response headers in the output
> * -k: Allow insecure server connections when using SSL
> * -S: Show error even when -s is used
> * -s: Silent mode
> * -X: Specify request command to use，如-X POST
> * --resolve &lt;host:port:address&gt; Resolve the host+port to this
>
>   address，如--resolve www.baidu.com:80:0.0.0.0

#### 其他

其他一些基础命令如ping, telnet，route，netstat, ps，iptables等，作为linux系统的基础运维命令，这里略过不讲。

### 故障排查思路

对于故障的排查，需要基于网络拓扑和命令工具的知识。当发生容器服务访问故障时，一般会分场景去排查以缩小故障范围，一般的思路是:

> 1. 检查目标Pod的服务是否正常
> 2. 检查目标Pod的IP是否从Master节点可达
> 3. 检查目标Service（IP + Port）是否可达
> 4. 检查容器的集群内置域名是否可以解析到相应的Service IP
> 5. 针对HTTP/HTTPS的访问，检查外部请求能否到达Master节点的80/443端口
> 6. 针对TCP的访问，检查外部请求能否到达Master节点或者集群内指定节点的相应端口，检查防火墙规则是否拦截了对应端口

注意：如果是集群内，东西方向通过Service访问，则没有上述的第5,6步。

以外部通过HTTP访问容器服务为例，基本步骤如下:

> 1. 在所访问Pod的宿主节点，通过curl命令，以Pod的IP和端口进行访问，如果成功则说明Pod可以提供服务；
> 2. 如果是访问Service，并且Service下有多个Pod，那么针对Service下的多个Pod需要多次执行步骤1；
> 3. 从某个Master去ping任意目标Pod所在的节点的tun0 IP（以可以直接ping
>
>    Pod IP），如果能通，则说明VXLAN隧道是通的；
>
> 4. 如果步骤3失败，则从Master去ping目标所在节点的SDN IP（可以通过命令
>
>    _oc get hostsubnet_
>
>    查看），如果能通，则说明底层网络是通的，但是VXLAN隧道出现故障，通常而言，这需要检查Master节点与目标节点的路由及IP，以确保节点间通信时使用但是SDN
>
>    IP；
>
> 5. 从Master去telnet Service的IP加端口，如telnet 172.30.100.200
>
>    8080，如果能通，则说明Service到Pod的代理是通的，否则需要检查Service中注册的端口（可以通过命令
>
>    _oc get svc YOUR\_SERVICE -o yaml_
>
>    查看）是否与Pod提供服务的端口一致；
>
> 6. 在Pod所在节点通过命令 \*nslookup
>
>    YOUR\_SERVICE.YOUR\_NAMESPACE.svc.cluster.local\*
>
>    来做域名解析检查，以检验集群内置的域名解析正常；
>
> 7. 如果步骤6失败，则在Master节点重复步骤6的检查，如果通过，则说明目标节点到Master节点的53/8053端口是不通的，需要进一步检查是因为底层网络问题，还是两个节点上的防火墙规则问题导致的；
> 8. 在Master节点的主网卡上通过tcpdump命令监听，确保外部请求能够到达Master节点，如果失败，则需要检查集群外的负载均衡、DNS解析等是否能够将外部请求正常的引流到Master节点。

当然，也可以按照从外到内的排查进行，以外部通过TCP直接访问集群内某节点（非Pod所在节点）为例，基本步骤如下:

> 1. 检查外部节点是否能ping通集群节点；
> 2. 检查外部节点是否能够telnet集群节点的IP +
>
>    Port，如果失败，则检查集群节点的防火墙规则是否拦截了相应端口（例如通过命令
>
>    _iptables -S INPUT_ 可以进行基本查看）；
>
> 3. 在集群节点上通过Service IP +
>
>    Port的telnet容器服务，如果失败，则需要检查Service中注册的端口（可以通过命令
>
>    _oc get svc YOUR\_SERVICE -o yaml_
>
>    查看）是否与Pod提供服务的端口一致；
>
> 4. 在集群节点上通过ping Pod所在节点的tun0
>
>    IP，如果能通，做说明VXLAN隧道没有问题，否则需要检查两个节点间的路由及IP是否与注册到OpenShift的SDN
>
>    IP一致；
>
> 5. 在目标Pod所在节点上以Pod IP +
>
>    Port的方式访问容器服务，以检查Pod能否提供服务。

通常而言，我们建议从内到外的进行检查，即先检查Pod是否可以提供服务，最后检查外部请求是否可以到达集群。当然，根据您的经验以及熟练度可以酌情选择进行哪些检查步骤。

### 其他补充

#### 关于集群节点的SDN IP

默认时，如果不进行任何配置，那么OpenShift会通过 _ip route get 8.8.8.8_ 来探测节点的主网卡，然后以主网卡的IP作为节点的SDN IP。当然，根据实际情况，也可以在/etc/origin/node/node-config.yaml中配置nodeIP来指定所想使用的IP，配置项nodeIP与nodeName等是平级的，无需考虑将它嵌套加入哪个已有的配置项下。

#### 关于错误No route to host

当访问集群节点IP + Port时，如果遇到错误"No route to host"，一般而言，在确保路由没有问题的情况下，很可能是因为目标节点的iptables规则进行了reject。登录到目标节点后，可以通过命令 _iptables -S INPUT_ 进行检查，可以看到包含类似 "-j REJECT --reject-with icmp-host-prohibited" 的这样一条记录。解决方法有两种:

> * 直接删除该规则，或者，
> * 在这条规则之前，加入其他规则来放行您的流量，例如 -A INPUT -p tcp
>
>   -m tcp --dport 8080 -j ACCEPT 来放行到本地8080端口的TCP访问流量。

#### SDN实现细节

如果想要深入了解OpenShift SDN网络的细节，可以参看系列文档 [http://medoc.readthedocs.io/en/latest/docs/openshift/ovs-multitenant/index.html](http://medoc.readthedocs.io/en/latest/docs/openshift/ovs-multitenant/index.html) 。

