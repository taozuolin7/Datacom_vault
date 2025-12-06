MPLS-VPN 与 SR 的区别：
 
MPLS-VPN  
1.MPLS-VPN工作机制：  
依靠LDP分发标签，指导数据包转发  
2.由工作机制引发的问题：  
1.依赖LDP协议。于是乎会发送很多LDP报文来维护LDP邻居，导致链路压力庞大  
2.LDP与底层IGP（isis,ospf）同步问题，IGP的收敛比LDP快。  
当主用路出现问题，流量切回备用路径，当主路径恢复，流量切回主用路径，但是由于LDP收敛慢于IGP，所有会导致丢包的情况  
3.MPLS-VPN的不足：  
1.路径的选择只能通告底层开销以及route-policy来做出选择，面对复杂的路径规划消耗大
   

SR  
1.SR工作机制：  
通过底层IGP扩散标签，指导数据报转发  
a.标签（SID）（需要在对应的接口开启MPLS 功能）  
b.IGP（ospf: 10类LSA）（isis: 与SID相关的TLV）  
c.BGP(BGP-LS) link state 地址族分配标签  
解决了MPLS-LDP的所有缺陷，并且推出新功能： 路径规划  
路径规划 也是SR被称为‘源路由’即：在路径源头就做好了数据包的转发路径  
2.路径规划  
1.抛开开销值，让特定的业务走最优的路径  
3.SR‘ 生不逢时’  
由于SR只至此ipv4的网络，但是现如今我们在逐步的从ipv4过渡到ipv6的网络，所有几乎只是用MPLS-VPN和SRv6
 
SRv6  
1.工作机制：  
通过提前编排好的路径转发数据包  
2.功能：  
1.路径编排  
2.SID（segment identifier）  
SRv6中的SID与SR中的SID存在区别：

|   |   |   |
|---|---|---|
|SRv6|end sid 描述设备|end-x sid 描述接口|
|SR|prefix sid（node sid） 描述设备|adj sid 描述接口|
 
3.MPLS数据报包与SRv6数据包的区别：  
MPLS

|   |   |   |
|---|---|---|
|以太网报头|SMAC|DMAC|
|MPLS LABEL|38091||
|MPLS LABEL|10081||
|IPv4 报头|SIP|DIP|
 
SRv6： 路径编排在IPv6报头中的 next header 43 中 type4（所以说抓包报文就如下，不会像MPLS-VPN那样）

|       |       |
| ----- | ----- |
| 以太网头部 |       |
| SIPv6 | DIPv6 |
| SIPv4 | DIPv4 |

4.SRv6的特点：  
SID = locator + function = 指令  
= ipv6地址  
路径 = Segment List

|   |   |   |
|---|---|---|
|Segment List|XPE1-ZPE1|SL=1|
|index 10|fc02:1::30|SL=1|
|index 20|fc02:5::1|SL=0|

在数据包转发的时：SL默认等于SL最大的SL=1  
此时SRv6的数据包的报头的DIPv6=fc02:1::30，转发到下一路由器，  
如果该IPv6地址为路由器的地址，那么SL-1=0，Segment List 整体不变，（目的是：溯源）  
如果不是，那么按照底层IGP转发，SL=1  
为什么不弹出该指令呢？  
因为中间的节点无法对该Segment List做修改，只有首尾节点才能对Segment List做修改。  
1.在SRv6中，报文头部的DIPv6地址会不断的变换，这可人让支持IPv6但是不支持SRv6的设备也能正常收发数据包