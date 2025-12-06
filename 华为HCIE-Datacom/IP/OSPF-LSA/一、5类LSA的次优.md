\> [!caution] This page contained a drawing which was not converted.   

```
5类LSA次优的场景下
```
 
```
为了解决次优 通过将外部5类LSA的转发地址置位
```
 
```
转发地址的作用？  
  OSPF设备收到5类LSA后，如果转发地址置位  
  则将转发地址作为5类LSA计算路由的下一跳
```
   

```
转发地址如何置位（置位条件）？  
  1.ASBR连接外部路由的接口要使能OSPF  
  2.ASBR连接外部路由的接口是广播型网络  
  3.ASBR连接外部路由的接口是非静默接口
```
 
```
什么是静默接口？  
  为了减少OSPF无用协议报文的发送，  
  可以将OSPF的接口设置为静默接口  
  静默接口只能通告网段，不发送协议报文  
  一般用于连接的终端业务网段  
  [AR2-ospf-1]silent-interface GigabitEthernet 0/0/1 
```
 
```
转发地址置位的值？  
  ASBR会将转发地址的值设置为去往外部路由的下一跳
```
 
```
如果ASBR去往外部路由存在多个下一跳（负载分担路由），  
如何选择下一跳？  
  选择下一跳地址大的作为转发地址
```

```
OSPF-5类LSA的转发地址  
默认情况下转发地址为0.0.0.0（不存在的）         
```
 
```
当5类LSA计算的路由存在次优路径，可以通过转发地址解决次优路径
```

```
如图：图中AR1，AR2属于Area0，而AR3是外部的一个路由器，  
现在AR2路由引入AR3静态路由。
```

![Exported image](Exported%20image%2020251206152513-0.png)

```
1.缺省情况下：如果AR1要去访问AR3那么会通过AR2再到AR3，这样便形成了次优路径，因为AR1直接到AR3更近。
```
 
```
因此：OSPF为解决这个问题，产生了FD（转发地址）；AR1直接向转发地址转发，而不用经过AR2中转，从而解决了次优路径
```

![Exported image](Exported%20image%2020251206152515-1.png)  
![Exported image](Exported%20image%2020251206152516-2.png)  
![Exported image](Exported%20image%2020251206152518-3.png)