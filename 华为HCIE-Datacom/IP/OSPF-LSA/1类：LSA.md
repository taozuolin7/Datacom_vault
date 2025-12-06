```
LSA的信息内容：  
 1.LSA的头部:  
  Type      : LSA的类型  
  Ls id     : 链路状态ID  
  Adv rtr   : 通告设备  
  Ls age    : 链路状态的有效时间  
  Len       : 长度  
  Options   : 功能字段  
  seq#      : 序列号  
  chksum    : 校验和
```
 
```
2.LSA的内容:  
Router LSA：路由器LSA（一类LSA）  
作用：描述设备自身的链路状态信息（拓扑信息和路由信息）  
谁产生的：设备自身  
作用范围：在区域内泛洪  
一类LSA的头部：  
  Type      : Router     LSA的类型  
  Ls id     : 10.3.3.3   被描述设备的RID  
  Adv rtr   : 10.3.3.3   设备的RID  
  Ls age    : 853        LSA的老化时间  
  Len       : 48   
  Options   :  E    
  seq#      : 80000005   
  chksum    : 0xb909
```
 
```
一类LSA会携带哪些链路状态信息？  
1.transet：广播型网络的描述  
2.P2P：点到点网络的描述  
3.virtual：虚拟网络的描述 （拓扑信息）  
4.stubnet：末节网络的描述  
每一个链路状态信息都会有 link id、data、link type 三个信息来描述
```
 
```
一类LSA携带的内容：  
  Link count: 2   链路的数量  
   * Link ID: 10.1.12.2    描述伪节点的RID  
     Data   : 10.1.12.1    去往伪节点的接口地址  
     Link Type: TransNet     （拓扑信息的描述）  
     Metric : 1            去往DR的接口开销
```
 
```
   * Link ID: 3.3.3.3       描述末节网络的网络地址  
     Data   : 255.255.255.255   末节网络的掩码  
  Link Type: StubNet  （路由信息的描述）  
     Metric : 0             末节网络的开销值  
     Priority : Medium      （路由计算的优先顺序）
```
 
```
[AR1]interface g0/0/0  
[AR1-GigabitEthernet0/0/0]ospf network-type p2p  
# 将接口的类型改为p2p  
     
* Link ID: 10.2.2.2     描述P2P邻居的RID  
     Data   : 10.1.12.1    去往P2P邻居的接口地址  
     Link Type: P-2-P        （ 拓扑信息的描述）  
     Metric : 1            去往P2P邻居的接口开销
```
 
```
   * Link ID: 10.1.12.0    描述P2P网络的网络地址  
     Data   : 255.255.255.0  P2P网络的掩码  
     Link Type: StubNet        
     Metric : 1              P2P网络的开销值  
     Priority : Low
```
 
```
如何区分一条LSA？  
 通过 type、LS id、adv router 三者进行区分
```
 
```
如何优选一条LSA？  
 1.先比较seq，seq越大越优先  
 2.seq相等，比较chksum，chksum越大越优先  
 3.chksum相等，比较age：  
     1.age≤900 ，则优选本地的（即：相同序列号的更新信息如果旧的序列号信息的age≤900，旧的不会被新的替换）  
     2.age\>900 ，则优选age值小的
```

![Exported image](Exported%20image%2020251206152146-0.png)   
```
LSA的age：  
  从0开始递增的时间  
  3600s为最大值  
  1800s为刷新更新
```
 
```
***如果接收到的LSA age为3600s，该LSA为最新的LSA  
   3600s代指撤销LSA的动作
```
 
```
广播型网络：  
 拓扑信息 transnet  
 路由信息 不是1类LSA来描述的，而是2类LSA
```
 
```
点到点网络：  
 拓扑信息 p2p  
 路由信息 stubnet
```
 
```
链路状态类型、链路状态ID、通告路由器三元组唯一地标识了一个LSA  
链路状态老化时间 、链路状态序列号 、校验和用于判断LSA的新旧
```
 
```
广播型网络和点对点网络（P2P）最大的OSPF区别：  
1.广播型网络：会选举DR，BDR，DBOther，而P2P不会。
```
 
![Exported image](Exported%20image%2020251206152148-1.png)  

```
AR1如何学习到AR3的loopback0的3.3.3.3/32的:  
1.AR1从其他设备发来的ospf包中的LSA信息中包含了10.3.3.3的信息，  
其中存在一条sutbnet的信息其中包含了3.3.3.3和mask32，然后AR1进行ospf路由计算出了路由条目。  
[AR1]dis ospf lsdb router 10.3.3.3
```

![Exported image](Exported%20image%2020251206152150-2.png)  
![Exported image](Exported%20image%2020251206152156-3.png) ![Exported image](Exported%20image%2020251206152157-4.png)   
```
1类LSA的撤销、更新：（给自己增加一个环回口，再把环回口撤销）  
已经通告的：  
Router    10.1.1.1        10.1.1.1          1174  36    8000000C  
第一次更新：seq+1 ，age=0，携带更新的链路状态（路由信息）  
Router    10.1.1.1        10.1.1.1            1   48    8000000D       
第二次更新：seq+1 ，age=0，携带更新的链路状态（路由信息）  
Router    10.1.1.1        10.1.1.1            1   60    8000000E       
```
 
```
第一次撤销：seq+1 ，age=0，不携带撤销的链路状态（路由信息）  
Router    10.1.1.1        10.1.1.1            1   48    8000000F  
第二次撤销：seq+1 ，age=0，不携带撤销的链路状态（路由信息）  
Router    10.1.1.1        10.1.1.1            1   36    80000010
```
 
```
更新式的撤销（路由信息更新、撤销）￼age重新从0开始，序列号增加1位，表示LSA更新
```