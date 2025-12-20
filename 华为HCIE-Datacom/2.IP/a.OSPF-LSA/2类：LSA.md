```
二类LSA的头部：  
  Type      : Network    LSA的类型  
  Ls id     : 10.1.12.2  伪节点的RID（DR接口的地址）  
  Adv rtr   : 10.2.2.2   DR设备的RID  
  Ls age    : 149   
  Len       : 32   
  Options   :  E    
  seq#      : 80000002   
  chksum    : 0xe52d
```
 
```
二类LSA携带的内容：  
  Net mask  : 255.255.255.0   掩码  （路由信息）  
  Priority  : Low  
     Attached Router  10.2.2.2   连接的路由器 （拓扑信息）  
     Attached Router 10.1.1.1  
***需要将LS id和netmask两者结合 形成广播型网络的网段  
（路由信息）
```

![Exported image](Exported%20image%2020251206152201-0.png)

```
2类LSA：  
network LSA：网络LSA（二类LSA） ，P2P网络中不存在二类LSA  
作用：描述广播型网络的路由信息和拓扑信息  
谁产生的：DR产生  
作用范围：区域内泛洪
```

![Exported image](Exported%20image%2020251206152202-1.png)

```
二类LSA的更新撤销：  
已经通告的LSA：（非DR设备增加或减少）  
Network   10.1.12.2   10.2.2.2   3  32    80000011  
第一次更新：seq+1，age=0，携带新增加的链路状态（拓扑信息）
```
 
```
Network   10.1.12.2   10.2.2.2   1  36    80000012  
第二次更新：seq+1，age=0，携带新增加的链路状态（拓扑信息）
```
 
```
Network   10.1.12.2   10.2.2.2   1  40    80000013  
第一次撤销：seq+1，age=0，不携带要撤销的链路状态（拓扑信息）
```
 
```
Network   10.1.12.2   10.2.2.2   1  32    80000014  
更新式的撤销（拓扑信息更新、撤销）
```

```
DR设备失效后，二类LSA的处理：  
 1.BDR会成为DR，并在广播型网络中通告一条新的2类LSA  
 2.原DR通告的2类LSA依然在LSDB中存在（要等待3600s的时间老化）  
 3.如果原DR设备恢复正常，则对于旧的2类LSA是否会进行处理？  
   Type      LinkState ID    AdvRouter    Age  Len   Sequence  
   Network   10.1.12.2       10.2.2.2     3600  36    80000018  
   会将旧的2类LSA老化时间直接设置为3600s，进行老化
```

```
虚拟设备（伪节点）  
在广播型网络中，用于连接所有的物理设备  
伪节点就是DR，伪节点的所有信息都是通过DR进行通告的  
如何区分一条LSA？  
 通过 type、LS id、adv router 三者进行区分
```
 
```
1. 原来的DR是：AR4的10.1.12.4
```

![Exported image](Exported%20image%2020251206152204-2.png)  

```
2.关闭DR，AR4的接口后，产生新的DR：
```

![Exported image](Exported%20image%2020251206152205-3.png)  

```
3.将原来的DR接口打开后：
```

![Exported image](Exported%20image%2020251206152207-4.png)  

```
4.原来的2类消失，产生新的二类：
```

![Exported image](Exported%20image%2020251206152212-5.png) ![Exported image](Exported%20image%2020251206152215-6.png)  

```
 
```