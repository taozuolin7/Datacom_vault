\> [!caution] This page contained a drawing which was not converted.   

```
OSPF特殊区域
```
 
```
什么是特殊区域？  
  管理员将非骨干区域手工改变为特殊区域
```
 
```
特殊区域的类型：  
  1.stub  
  2.totally stub  
  3.NSSA  
  4.totally NSSA
```
 
```
为什么要有特殊区域？  
  为了减少区域内LSDB的规模（LSA的数量）
```
 
```
为什么要减少LSA的数量？  
  业务需求  
  为了减轻设备的计算压力  
  优化末节网络设备（老旧设备的正常计算）
```
 
```
特殊区域如何减少LSA的数量？  
  每一种特殊区域的工作内容
```

```
STUB区域：末节区域  
[AR2-ospf-1-area-0.0.0.1]stub   
选定为特殊区域的，区域内每一台设备都需要执行特殊区域的配置
```
 
```
作用：不允许区 域内存在4类、5类LSA，可以存在1类、2类、3类LSA  
特点 ：由ABR下发缺省的3类LSA给特殊区域内的IR设备
```
 
```
区域间信息 如何访问？  
  1.骨干区域 如何访问特殊区域的区域间路由？  
     通过ABR 泛洪的3类LSA  
  2.特殊区域如 何访问骨干区域的区域间路由？  
     通过ABR泛 洪的3类LSA
```
 
```
区域外信息如何 访问？  
  1.骨干区域如 何访问特殊区域的区域外路由？  
      stub区域内不 存在ASBR设备，即无法产生5类LSA
```

![Exported image](Exported%20image%2020251206152526-0.png)  

```
  2.特殊区域如何访 问骨干区域的区域外路由？  
      通过ABR下发 的缺省3类LSA计算的缺省路由访问
```
 
```
totally stub：完全末节区域  
[AR2-ospf-1-area-0.0.0.1]stub no-summary   
作用：不允许区域内存在3类、4类、5类LSA，可以存在1类、2类LSA  
特点：由ABR下发缺省的3类LSA给特殊区域内的IR设备
```
 
```
区域间信息如何访问？  
  1.骨干区域如何访问特殊区域的区域间路由？  
     通过ABR泛洪的3类LSA访问  
  2.特殊区域如何访问骨干区域的区域间路由？  
     通过ABR下发的缺省3类LSA计算的缺省路由访问
```
 
```
区域外信息如何访问？  
  1.骨干区域如何访问特殊区域的区域外路由？  
     完全stub区域内不存在ASBR设备，即无法产生5类LSA  
  2.特殊区域如何访问骨干区域的区域外路由？  
     通过ABR下发的缺省3类LSA计算的缺省路由访问
```

```
即想拥有特殊区域的LSA过滤，也想拥有ASBR外部路由的引入  
NSSA：非纯末节区域（Not-So-Stubby Area）  
[AR2-ospf-1-area-0.0.0.1]nssa   
作用：区域内不存在4类、5类LSA，允许存在1类、2类、3类、7类LSA  
特点：ABR会下发缺省的7类LSA给特殊区域内的IR设备  
   NSSA区域内允许存在ASBR设备，即允许外部路由信息的引入  
特殊区域内不存在5类LSA，所以外部路由引入通过7类LSA来描述
```
 
```
7类LSA：  
作用：描述外部路由信息  
谁产生的：NSSA区域内的ASBR （NSSA区域的ABR会产生缺省7类LSA）  
泛洪范围：NSSA区域  
[AR4]dis ospf lsdb nssa 100.4.4.4   
LSA的头部信息：  
  Type      : NSSA      7类LSA  
  Ls id     : 100.4.4.4 外部路由信息的网络地址  
  Adv rtr   : 10.4.4.4  ASBR的RID  
  Ls age    : 233   
  Len       : 36   
  Options   :  NP    
  seq#      : 80000001   
  chksum    : 0x28f2  
LSA的内容：  
  Net mask  : 255.255.255.255 外部路由信息的掩码  
  TOS 0  Metric: 1            外部路由的开销  
  E type    : 2                 
  Forwarding Address : 10.1.24.4   
  Tag       : 1   
  Priority  : Low
```
 
```
区域间信息如何访问？  
  1.骨干区域如何访问特殊区域的区域间路由？  
     通过ABR泛洪的区域间3类LSA访问  
  2.特殊区域如何访问骨干区域的区域间路由？  
     通过ABR泛洪的区域间3类LSA访问
```
 
```
区域外信息如何访问？  
1.骨干区域如何访问特殊区域的区域外路由？  
 骨干区域不存在7类LSA，如何访问NSSA区域的外部路由？  
 骨干区域需要通过5类LSA去访问  
5类LSA和7类LSA之间的关系？  
 5类、7类LSA的内容信息都是相同的，所以两者可以相互转换  
5类LSA和7类LSA之间如何转换？  
 通过ABR设备实现转换  
   ABR执行7转5的操作：  
      7类LSA的通告者是NSSA区域内ASBR的RID  
      5类LSA的通告者是ABR的RID  
    如果存在多台ABR设备，则RID大的执行7转5操作
```
 
```
 执行7转5操作的ABR存在的特点：  
  会成为ASBR设备（产生了5类LSA）
```
 
```
骨干区域通过5类LSA访问特殊区域的外部路由？  
  5类LSA的ADV router是ABR的RID传递  
  5类LSA产生的区域，不存在4类LSA  
  骨干区域内的设备通过1类LSA计算到5类LSA的ASBR（ABR）
```
 
```
如果执行7转5的ABR在NSSA区域内的路径开销更大，则会形成次优  
如何解决7转5后形成的次优路径？  
    通过5类LSA携带的转发地址解决次优（FD）
```

![Exported image](Exported%20image%2020251206152528-1.png)  

![Exported image](Exported%20image%2020251206152529-2.png)

```
区域间信息如何访问？  
  1.骨干区域如何访问特殊区域的区域间路由？  
       
  2.特殊区域如何访问骨干区域的区域间路由？  
      
```
 
```
区域外信息如何访问？  
  1.骨干区域如何访问特殊区域的区域外路由？  
       
  2.特殊区域如何访问骨干区域的区域外路由？
```

```
特殊区域和普通区域的不同点：  
对于设备而言，需要为自身存在的特殊区域进行标识  
通过1类LSA中option字段中的Ebit和Nbit进行描述  
Ebit 表示是否支持5类LSA  
Nbit 表示是否支持7类LSA
```
 
```
Ebit、Nbit =  0 0    STUB区域  
Ebit、Nbit =  0 1    NSSA区域  
Ebit、Nbit =  1 0    普通区域  
Ebit、Nbit =  1 1    不存在的
```
 
```
查看接口ospf错误信息：  
display ospf error interface g0/0/0   
OSPF 建立邻居的还有一个需求：  
两个直连接口的所属的区域类型要相同：特殊区域对应特殊区域
```
   

```
stub区域 和 完全stub区域之间的关联：  
  字段描述的区域类型都是 E=0 N=0 所以可以正常建立邻居关系
```
 
```
如果一端为stub区域，一端为完全stub区域  
则按照ABR配置的区域类型执行LSDB数据库LSA的过滤
```

```
[AR2-ospf-1-area-0.0.0.1]nssa translator-always    
手工指定7转5设备  
指定完成后，RID大的设备不会在执行7转5操作
```
   

```
totally-nssa:完全非纯末节区域  
[AR2-ospf-1-area-0.0.0.1]nssa no-summary   
作用：区域内不允许存在3类、4类、5类LSA，允许存在1类、2类、7类LSA  
特点：由ABR下发 缺省的3类LSA 和缺省的7类LSA
```
 
```
区域间信息如何访问？  
  1.骨干区域如何访问特殊区域的区域间路由？  
     通过ABR泛洪的区域间明细3类LSA访问  
  2.特殊区域如何访问骨干区域的区域间路由？  
     通过ABR下发的缺省3类LSA计算的缺省路由访问
```
 
```
区域外信息如何访问？  
  1.骨干区域如何访问特殊区域的区域外路由？  
     通过7转5的5类LSA访问  
  2.特殊区域如何访问骨干区域的区域外路由？  
     通过ABR下发的缺省7类LSA计算的缺省路由访问  
（理论上的说法）
```
 
```
通过ABR下发的缺省3类LSA计算的缺省路由访问  
***主要是3类优于7类，所以通过3类访问
```

```
1类、2类LSA 优于 3类LSA 优于 5类LSA（type 1） 优于 5类LSA（type 2）  
5类LSA 等于 7类LSA
```

```
拓扑图如下：
```

![Exported image](Exported%20image%2020251206152531-3.png)  
![Exported image](Exported%20image%2020251206152533-4.png)   
```
[AR4]ip route-static 100.4.4.4 32 n 0  
[AR4-ospf-1]import-route static   
引入外部路由，但是没产生5类LSA信息
```

![Exported image](Exported%20image%2020251206152535-5.png)   ![Exported image](Exported%20image%2020251206152537-6.png)

```
所有特殊区域的问题：  
 1.特殊区域内的AR4设备通过缺省访问其他区域的外部路由  
 2.AR4设备的缺省路由是负载分到到多个ABR（AR2、AR3）  
 3.如果AR2到ASBR的开销为100，AR3到ASBR的开销为10  
 4.AR4如果负载分担到AR2执行数据转发，则形成次优路径  
*问题本质为 缺省LSA由ABR产生并泛洪，ABR不会叠加计算到达ASBR的开销
```
 
```
解决方式：  
 1.AR4将次优路径的开销值改大  
 2.修改ABR通告的缺省LSA开销值  
 [AR2-ospf-1-area-0.0.0.1]default-cost  \<0-16777214\>
```

![Exported image](Exported%20image%2020251206152542-7.png) ![Exported image](Exported%20image%2020251206152544-8.png) 

```
Totally stub区域：不存在明细的3类LSA信息
```

![Exported image](Exported%20image%2020251206152546-9.png)  

```
两个路由器设备的区域类型不同一个是stub，另一个是totally-stub，它们之间依然能够建立邻居关系
```

![Exported image](Exported%20image%2020251206152548-10.png)  

![Exported image](Exported%20image%2020251206152549-11.png) 

```
[AR4-ospf-1]import-route static 引入外部路由
```

![Exported image](Exported%20image%2020251206152551-12.png)  
![Exported image](Exported%20image%2020251206152553-13.png)

![Exported image](Exported%20image%2020251206152558-14.png) ![Exported image](Exported%20image%2020251206152600-15.png) 

```
次优路径的由来：（如图所示的开销，缺省cost =1）  
1.当AR4通告自己的nssa区域的7类LSA信息到ARB后，RID大的ABR会执行7转5类LSA的操作，所以AR3将5类LSA信息在Area 0泛洪；AR1收到5类LSA信息，确定adv router 是AR3.  
2.AR1如何访问AR4？因为不存在4类LSA信息，（原因是：5类LSA信息是在Area 0 里面产生的，产生对应5类LSA信息的区域，不产生对应的4类LSA信息）  
3.所以AR1访问AR4只能通过1类LSA信息访问AR4 ，根据5类的adv router，找到ABR，AR3，通过AR3中的7类LSA信息访问AR4，至此形成次优路径
```

```
5类LSA的转发地址如何存在？  
   通过7转5时，继承7类LSA的adv router 的RID  
7类LSA的转发地址如何存在？  
   由NSSA的ASBR设备引入外部路由，产生7类LSA时，主动添加的ASBR添加的转发地址的值？  
ASBR设备的环回口优先  
（不存在环回口，则优选第一个物理接口地址）  
1.当不存在环回接口：
```

![Exported image](Exported%20image%2020251206152602-16.png) ![Exported image](Exported%20image%2020251206152604-17.png)

```
2.当存在环回接口：
```

![Exported image](Exported%20image%2020251206152605-18.png)

```
转发地址的解决次优的过程：  
    1.骨干区域内的设备收到5类LSA，并检查存在转发地址  
[AR1]display ospf lsdb  
[AR1]display ospf lsdb ase 100.4.4.4  
   
```

![Exported image](Exported%20image%2020251206152607-19.png)     

```
 2.将转发地址作为5类LSA计算路由的下一跳  
[AR1]dis ip routing-table 
```

![Exported image](Exported%20image%2020251206152609-20.png)

```
 3.下一跳通过区域间路由访问（明细的3类LSA）
```

![Exported image](Exported%20image%2020251206152614-21.png)

```
 4.3类LSA会叠加计算区域间的开销值
```
   

![Exported image](Exported%20image%2020251206152615-22.png) ![Exported image](Exported%20image%2020251206152617-23.png) ![Exported image](Exported%20image%2020251206152618-24.png) ![Exported image](Exported%20image%2020251206152620-25.png) ![Exported image](Exported%20image%2020251206152622-26.png) ![Exported image](Exported%20image%2020251206152623-27.png) ![Exported image](Exported%20image%2020251206152628-28.png)

```
NSSA区域引入外部：N比特变为了P比特，表示执行7类转5类LSA
```
 
```
7类LSA：
```

![Exported image](Exported%20image%2020251206152629-29.png)  

```
1类LSA：
```

![Exported image](Exported%20image%2020251206152631-30.png)  

```
如果是NSSA区域中的7类LSA，则Nbit会变为Pbit，表示是否执行7转5的操作
```

Propagate 传播