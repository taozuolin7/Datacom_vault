# MPLS概论
**路由器转发的三个阶段：**  
1.基于CPU的转发：所有的数据都要通过CPU进行处理（查找路由表），转发性能低下，限制网络发展   1.5 为了提高设备的转发性能，设备在处理转发时，如果不需要查路由表 就可以提升  
MPLS：将数据封装为带标签的数据，设备直接根据标签执行转发，不会在查找上层的IP信息  
因为MPLS封装是在IP报头之前 和 以太网数据帧之后，所以称MPLS为2.5层协议
 
2.基于缓存的转发：一次路由，多次交换（同类数据的第一个报文进行CPU查表，后续的报文根据缓存表快速转发）
 
3.基于拓扑的转发：设备根据转发表，直接转发数据，不需要查路由表￼￼  
由于硬件设备的不断升级，软件性能的不断提高  
如今的设备就算是按照CPU的转发，也不会消耗大量的性能资源  
如今的设备是分布式系统架构，所以对于报文的转发处理只会更快  
如此MPLS在设备转发方面已经成为了鸡肋
 
**对于MPLS设备转发不在适用，但是对应数据信息的访问是的封装有很好的适应性**  
（VPN隧道的本质是？ 数据的再封装）  
现有网络中存在私网和公网地址  
私网地址需要通过NAT实现网络通信  
如果私网地址之间需要通信，就需要在公网网络中搭建私有网络的隧道  
私有网络的隧道就可以通过MPLS进行搭建

**MPLS：多协议标签交换**  
MPLS的术语：  
1.MPLS domain：运行了MPLS协议的设备组成的区域  
2.设备类型：  
1.LER（Label Edge Router）：边界设备  
1.ingress：入节点，设备对应的标签处理动作为push（压入标签）  
2.egress：出节点，设备对应的标签处理动作为pop（弹出标签）  
2.LSR（Label Switching Router）：内部设备  
transit：中间节点，设备对应的标签处理动作为swap（替换标签）￼  
3.LSP：（Label Switched Path，标签交换路径）  
由一个入节点 到 一个出节点 组成一条转发路径  
该路径上可以存在 0个 或 多个 中间节点  
*判断LSP的方向 可以根据上下游设备来判断  
ingress 是 egress 的 上游节点  
egress 是 ingress 的 下游节点  
ingress 是 transit的 上游节点  
transit 是 egress 的 上游节点  
4.MPLS体系结构：

![Exported image](Exported%20image%2020251206150256-0.png)  

5.FEC：（Forwarding Equivalence Class，转发等价类）  
MPLS对于某些特征相同的报文 并 执行相同的处理动作，称为FEC  
FEC可以通过SIP、DIP、Sport、Dport、VPN等要素来  
判断是否为相同特征的报文  
***对于认为相同的流量，有统一的处理动作

**MPLS的标签：**  
1.报文格式（封装在IP和数据链路层之间）  
*MPLS报文头部一共是32bit长度  
报文格式：

![Exported image](Exported%20image%2020251206150257-1.png)

报文格式的内容：  
1.标签值域（0-20bit）：标签的取值范围是多少  
1.特殊标签值：取值范围是0-15  
0号标签：显示空标签，在倒数第二跳设备发送数据时，会保留标签给到最后一跳设备  
该功能主要用于保证qos数据优先转发的操作  
3号标签：隐式空标签，在倒数第二跳设备发送数据时，会直接执行数据的标签弹出（默认的功能）
 
1号标签：在最外层，表示收到1号标签按照下一层标签的处理进行转发，转发时还要封装1号标签
 
2号标签：显示空标签，和0号标签的作用类型，用于IPv6网络
 
14号标签：通过发送OAM报文检测和通告LSP故障，告知LSP路径上的所有设备
 
2.静态标签值：取值范围是16-1023  
3.动态标签值：取值范围是1024-2^20
 
2.exp（21-23bit）（Experimental Use）：优先级的取值  
3bit 值是0-7 给QoS调度服务时使用 000-111  
该值的使用，只能在MPLS域中
 
3.S （Bottom of Stack）（24bit）：栈低标签位  
MPLS是可以支持多层嵌套的  
S bit 置 0：表示MPLS头部不是靠近IP报文头  
S bit 置 1：表示MPLS头部紧挨着IP报文头
 
4.TTL（25-32bit）：和IP的TTL值作用相同
 
标签栈：多个MPLS标签头部的集合，称之为标签栈  
标签栈的第一个头部（靠近数据链路层头部的MPLS头部），称为栈顶标签  
标签栈的最后一个头部（靠近IP层的MPLS头部），称为栈低标签

MPLS 1 S bit = 0 exp=7  
MPLS 1088 S bit = 0  
MPLS 1024 S bit = 1  
SIP 1.1.1.1 DIP 2.2.2.2

配置思路：  
1.设备接口地址配置完成（AR1、AR5存在环回口地址）  
2.构建AR1-AR5的MPLS静态隧道完成环回口通信

MPLS是直接通过标签执行数据转发的  
设备如果存在标签值，但是不存在路由  
设备可以直接通过标签值转发数据

ingress只存在出标签，没有入标签  
egress只存在入标签，没有出标签

```
[AR1]mpls lsr 1.1.1.1    在LDP环境下使用  
[AR1]mpls                全局开启MPLS  
[AR1-GigabitEthernet0/0/0]mpls   接口开启MPLS功能（即让接口具备转发标签报文的能力）
```
 
```
以上命令所有mpls domain内的设备都需执行
```
 
```
AR1：  
[AR1]static-lsp ingress AR1_AR5 destination 5.5.5.5 32 outgoing-interface GigabitEthernet 0/0/0 nexthop 10.1.12.2 out-label 102  
[AR1]static-lsp egress AR5_AR1 incoming-interface GigabitEthernet 0/0/0 in-label 201  
[AR1]ip route-static 5.5.5.5 32 10.1.12.2     对于ingress要存在到达目标网络的路由信息（激活FEC）  
AR2：  
[AR2]static-lsp transit AR1_AR5 incoming-interface Gi 0/0/0 in-label 102 outgoing-interface Gi 0/0/1 nexthop 10.1.23.3 out-label 203  
[AR2]static-lsp transit AR5_AR1 incoming-interface Gi 0/0/1 in-label 302 outgoing-interface gi 0/0/0 nexthop 10.1.12.1 out-label 201  
AR3：  
[AR3]static-lsp transit AR1_AR5 incoming-interface Gi 0/0/0 in-label 203 outgoing-interface Gi 0/0/1 nexthop 10.1.34.4 out-label 304  
[AR3]static-lsp transit AR5_AR1 incoming-interface Gi 0/0/1 in-label 403 outgoing-interface gi 0/0/0 nexthop 10.1.23.2 out-label 302  
AR4：  
[AR4]static-lsp transit AR1_AR5 incoming-interface Gi 0/0/0 in-label 304 outgoing-interface Gi 0/0/1 nexthop 10.1.45.5 out-label 405  
[AR4]static-lsp transit AR5_AR1 incoming-interface Gi 0/0/1 in-label 504 outgoing-interface gi 0/0/0 nexthop 10.1.34.3 out-label 403  
AR5：  
[AR5]static-lsp egress AR1_AR5 incoming-interface GigabitEthernet 0/0/0 in-label 405  
[AR5]static-lsp ingress AR5_AR1 destination 1.1.1.1 32 outgoing-interface GigabitEthernet 0/0/0 nexthop 10.1.45.4 out-label 504  
[AR5]ip route-static 1.1.1.1 32 10.1.45.4    对于ingress要存在到达目标网络的路由信息（激活FEC）
```

![Exported image](Exported%20image%2020251206150259-2.png)
 
1.值得注意的是：ping 的时候需要指定ip地址，不然它缺省使用接口地址去ping￼

![Exported image](Exported%20image%2020251206150304-3.png)  

2.查看fib表：tunnel 值不为0￼

![Exported image](Exported%20image%2020251206150306-4.png)  

3.查看静态mpls-lsp￼

![Exported image](Exported%20image%2020251206150308-5.png)  
![Exported image](Exported%20image%2020251206150309-6.png)