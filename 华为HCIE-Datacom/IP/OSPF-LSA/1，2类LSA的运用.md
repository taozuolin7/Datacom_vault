\> [!caution] This page contained a drawing which was not converted.   

```
在一个OSPF网络中如何判断有多少个1类LSA，2类LSA？  
1.每台允许OSPF的设备产生1条1类LSA。  
2.有几个DR就有几条2类LSA。  
(也可以描述为：存在几个广播型网络，就有几条LSA)  
3.P2P网络中不存在2类LSA
```

```
五条1类LSA，3条2类LSA：
```

![Exported image](Exported%20image%2020251206152219-0.png)  

```
OSPF采用SPF算法会计算一棵无环的、最短的路径树  
SPF就是通过1类、2类LSA携带的拓扑信息计算路径计算的  
     在通过1类、2类LSA携带的路由信息计算路由条目
```

```
实节点 到 伪节点的开销是接口开销  
伪节点 到 实节点的开销是0（不存在）
```

```
如何根据已知的OSPF的LSDB手绘出拓扑图：
```

![Exported image](Exported%20image%2020251206152221-1.png)  

```
1.利用本台路由器的RID：查看相关信息：  
找出伪节点的RID，以及去往伪节点的开销  
[AR1]display ospf lsdb router 10.1.1.1
```
 
```
 OSPF Process 1 with Router ID 10.1.1.1  
         Area: 0.0.0.0  
 Link State Database   
  Type      : Router  
  Ls id     : 10.1.1.1  
  Adv rtr   : 10.1.1.1    
  Ls age    : 839   
  Len       : 36   
  Options   :  E    
  seq#      : 80000005   
  chksum    : 0x44df  
  Link count: 1  
   * Link ID: 10.1.123.3     
     Data   : 10.1.123.1     
     Link Type: TransNet (广播型网络)       
     Metric : 1
```
 
```
2.使用伪节点的RID查看新的LSA：  
查看伪节点连接的其他路由器  
[AR1]display ospf lsdb network 10.1.123.3（2类LSA）
```
 
```
 OSPF Process 1 with Router ID 10.1.1.1  
         Area: 0.0.0.0  
 Link State Database   
  Type      : Network  
  Ls id     : 10.1.123.3  
  Adv rtr   : 10.3.3.3    
  Ls age    : 336   
  Len       : 36   
  Options   :  E    
  seq#      : 80000004   
  chksum    : 0x196d  
  Net mask  : 255.255.255.0  
  Priority  : Low  
     Attached Router    10.3.3.3  
     Attached Router    10.1.1.1  
     Attached Router    10.2.2.2
```

```
3.根据新的RID，继续查看LSA数据库  
[AR1]display ospf lsdb router 10.2.2.2
```
 
```
 OSPF Process 1 with Router ID 10.1.1.1  
         Area: 0.0.0.0  
 Link State Database   
  Type      : Router  
  Ls id     : 10.2.2.2  
  Adv rtr   : 10.2.2.2    
  Ls age    : 639   
  Len       : 48   
  Options   :  E    
  seq#      : 80000009   
  chksum    : 0x3983  
  Link count: 2  
   * Link ID: 10.1.123.3     
     Data   : 10.1.123.2     
     Link Type: TransNet       
     Metric : 1  
   * Link ID: 10.1.24.4      
     Data   : 10.1.24.2      
     Link Type: TransNet       
     Metric : 1
```
 
![Exported image](Exported%20image%2020251206152223-2.png) ![Exported image](Exported%20image%2020251206152225-3.png) ![Exported image](Exported%20image%2020251206152227-4.png)

```
3.1根据伪节点的RID继续查看LSA数据库：  
[AR1]display ospf lsdb network 10.1.24.4
```
 
```
 OSPF Process 1 with Router ID 10.1.1.1  
         Area: 0.0.0.0  
 Link State Database   
  Type      : Network  
  Ls id     : 10.1.24.4  
  Adv rtr   : 10.4.4.4    
  Ls age    : 1385   
  Len       : 32   
  Options   :  E    
  seq#      : 80000002   
  chksum    : 0x8075  
  Net mask  : 255.255.255.0  
  Priority  : Low  
     Attached Router    10.4.4.4  
     Attached Router    10.2.2.2（拓扑信息x）
```

![Exported image](Exported%20image%2020251206152232-5.png)

```
4.根据上一次查看的2类LSA信息查看RID=10.4.4.4的lsdb￼[AR1]display ospf lsdb router 10.4.4.4
```
 
```
 OSPF Process 1 with Router ID 10.1.1.1  
         Area: 0.0.0.0  
 Link State Database   
  Type      : Router  
  Ls id     : 10.4.4.4  
  Adv rtr   : 10.4.4.4    
  Ls age    : 1631   
  Len       : 48   
  Options   :  E    
  seq#      : 80000008   
  chksum    : 0x7ec9  
  Link count: 2  
   * Link ID: 10.1.24.4      
     Data   : 10.1.24.4      
     Link Type: TransNet       
     Metric : 1  
   * Link ID: 10.1.45.5      
     Data   : 10.1.45.4      
     Link Type: TransNet     
```
 
```
     Metric : 1
```

![Exported image](Exported%20image%2020251206152233-6.png)

```
4.1根据伪节点连接的网络，查看二类LSA信息：10.1.45.5  
[AR1]display ospf lsdb network 10.1.45.5
```
 
```
 OSPF Process 1 with Router ID 10.1.1.1  
         Area: 0.0.0.0  
 Link State Database 
```
   

```
  Type      : Network  
  Ls id     : 10.1.45.5  
  Adv rtr   : 10.5.5.5    
  Ls age    : 1016   
  Len       : 32   
  Options   :  E    
  seq#      : 80000003   
  chksum    : 0xe0f1  
  Net mask  : 255.255.255.0  
  Priority  : Low  
     Attached Router    10.5.5.5  
     Attached Router    10.4.4.4
```

![Exported image](Exported%20image%2020251206152235-7.png)

```
5.根据上一次2类LSA的信息查看1类LSA的信息：10.5.5.5  
[AR1]display ospf lsdb router 10.5.5.5
```
 
```
 OSPF Process 1 with Router ID 10.1.1.1  
         Area: 0.0.0.0  
 Link State Database   
  Type      : Router  
  Ls id     : 10.5.5.5  
  Adv rtr   : 10.5.5.5    
  Ls age    : 1299   
  Len       : 72   
  Options   :  E    
  seq#      : 8000000b   
  chksum    : 0xcbb9  
  Link count: 4  
   * Link ID: 10.1.45.5      
     Data   : 10.1.45.5      
     Link Type: TransNet       
     Metric : 1  
   * Link ID: 5.5.5.5        
     Data   : 255.255.255.255   
     Link Type: StubNet        
     Metric : 0   
     Priority : Medium  
   * Link ID: 10.3.3.3       
     Data   : 10.1.35.5      
     Link Type: P-2-P          
     Metric : 48  
   * Link ID: 10.1.35.0      
     Data   : 255.255.255.0   
     Link Type: StubNet        
     Metric : 48   
     Priority : Low
```
 
![Exported image](Exported%20image%2020251206152236-8.png)

```
6.根据10.5.5.5，判断AR3-AR5是P2P链路  
[AR1]display ospf lsdb router 10.3.3.3
```
 
```
 OSPF Process 1 with Router ID 10.1.1.1  
         Area: 0.0.0.0  
 Link State Database 
```
   

```
  Type      : Router  
  Ls id     : 10.3.3.3  
  Adv rtr   : 10.3.3.3    
  Ls age    : 1454   
  Len       : 60   
  Options   :  E    
  seq#      : 8000000c   
  chksum    : 0xd542  
  Link count: 3  
   * Link ID: 10.1.123.3     
     Data   : 10.1.123.3     
     Link Type: TransNet       
     Metric : 1  
   * Link ID: 10.5.5.5       
     Data   : 10.1.35.3      
     Link Type: P-2-P          
     Metric : 48  
   * Link ID: 10.1.35.0      
     Data   : 255.255.255.0   
     Link Type: StubNet        
     Metric : 48   
     Priority : Low
```

![Exported image](Exported%20image%2020251206152238-9.png)