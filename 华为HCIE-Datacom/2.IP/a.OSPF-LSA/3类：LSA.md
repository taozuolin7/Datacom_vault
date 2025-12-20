```
为什么OSPF要划分多区域？  
1.区域内减少LSA的数量   
2.设备的路由计算速度更快（收敛速度快）  
3.拓扑计算复杂、路由计算复杂  
4.拓扑故障影响区域内的所有设备
```

```
划分多区域的优势：  
1.定量控制区域内的LSA  
2.防止设备故障后，收敛时间过长  
3.可以更加精细控制业务路由信息
```

```
区域如何划分？  
 OSPF区域的边界是设备  
 通过area id 值的编号做区分
```

```
区域的分类：  
1.骨干区域： 就是area id = 0 的区域  
    在一个OSPF网络中，有且只有一个  
2.非骨干区域：就是area id ≠ 0 的区域  
    在一个OSPF网络中，可以存在多个  
3.特殊区域：就是非骨干区域转换的区域类型
```
 
```
***非骨干的区域 必须 和 骨干区域连接
```

```
区域划分的设备类型：  
1.IR：区域内路由器，所有接口都在一个区域内的设备  
（Internal Router）  
2.ABR：区域边界路由器，设备的接口在不同的区域内  
(Area Border Router)
```

```
ABR的作用：在区域之间产生并进行LSA的泛洪  
就是将本区域内的1类、2类LSA携带的路由信息，转换为连接区域内的3类LSA
```

```
区域间的信息如何相互传递？  
   通过ABR做信息传递
```
 
```
区域间的信息是什么？  
   是3类LSA
```
 
```
什么是3类LSA？
```

```
summary network lsa：汇总网络LSA（3类LSA）  
作用：描述了区域间的路由信息  
谁产生的：ABR的产生的  
泛洪范围：在区域间泛洪
```
 
```
3类LSA的头部信息：  
  Type      : Sum-Net   3类LSA  
  Ls id     : 3.3.3.3   区域间的网络地址（只能描述一个网络）  
  Adv rtr   : 10.2.2.2  ABR的RID  
  Ls age    : 360   
  Len       : 28   
  Options   :  E    
  seq#      : 80000001   
  chksum    : 0x66d9
```
 
```
3类LSA的内容信息：  
  Net mask  : 255.255.255.255   掩码  
  Tos 0  metric: 1              路由的开销  
  Priority  : Medium            路由计算的优先顺序
```
 
```
***按照3类LSA描述路由信息的方式，  
一条3类LSA只能描述一条路由信息￼
```

![Exported image](Exported%20image%2020251206152243-0.png) ![Exported image](Exported%20image%2020251206152244-1.png)

```
区域内设备 收到3类LSA 如何判断 3类LSA是可用的？  
   收到的3类LSA 通告者 如果是自身识别的ABR设备，则3类LSA可用  
区域内设备 如何识别ABR设备？  
   成为了ABR的设备会告知区域内设备 ABR是谁  
   ABR设备通告的1类LSA中flag字段会有B bit置位  
如何成为ABR？  
   连接两个或两个以上的区域，并且有一个区域是区域0 的设备 就是ABR设备
```
 
```
ABR的分类：  
   1.假ABR：连接两个或两个以上的区域，并且有一个区域是区域0，但区域0没有任何使能的接口  
   不具备真实ABR任何的能力（产生LSA、泛洪LSA、防环）
```
 
```
   2.真假ABR：连接两个或两个以上的区域，并且有一个区域是区域0，区域0存在使能的接口（区域0内不存在邻接关系）  
(环回接口作为Area 0)  
具备真实ABR的LSA产生和泛洪LSA的能力，但是不具备防环的功能
```
 
```
   3.真ABR：连接两个或两个以上的区域，并且有一个区域是区域0，且区域0存在使能的接口并形成了邻接关系      
    具备的能力（产生LSA、泛洪LSA、防环）
```

![Exported image](Exported%20image%2020251206152249-2.png)

```
ABR的防环：  
 ABR收到非骨干区域的3类LSA时，会接收到LSDB中，但不会计算路由信息  
如图所示：R2，和R3会接受R4，R5带来的LSA信息，但不会计算路由信息。
```

![Exported image](Exported%20image%2020251206152250-3.png)  

```
如图2所示，R2,R3,R4,R5都是ABR，防环机制怎么实现呢？  
1.当ABR接收到1，2类LSA后，会将1，2类LSA信息转换为3类LSA信息，然后区域0的设备又会将3类LSA信息泛洪到Area2，但是R5虽然会接收来自非骨干区域的3类LSA信息，但是不会计算路由信息，故防环。
```

![Exported image](Exported%20image%2020251206152252-4.png)  

```
真假ABR举例  
当存在两个Area 0，并且只有环回接口在Area 0中，这种ABR称作真假ABR，不具备防环功能，但能泛红LSA，和产生LSA信息。  
1.路由条目3.3.3.3是在Area1中计算出来的。  
2.在Area1中3.3.3.3是一条3类LSA。  
3.因为是真假ABR，所以再AR1的LSDB中会存在那么一条3类LSA并且会存在一条对应的路由条目，原因是因为真假ABR不存在防环功能。  
（当前这个ABR是不可靠的）
```

![Exported image](Exported%20image%2020251206152254-5.png) ![Exported image](Exported%20image%2020251206152256-6.png) ![Exported image](Exported%20image%2020251206152257-7.png)   
```
如图：在AR1所在的Area0中增加一个路由器，使得AR1存在邻接关系，变为真ABR，  
1.在OSPF路由中不存在3.3.3.3的路由条目了  
2.R1成为真的ABR，虽然会接收来自非骨干区域的3类LSA，但是不会计算路由条目。导致没有3.3.3.3的路由  
3.但是在Area1的LSDB中存在3.3.3.3的3类LSA信息。  
4.如果在AR11的loopback0增加一条信息1.1.1.1/32，AR3会学习到，应为AR3是真假ABR，它会计算路由条目，但是不存在防环功能。
```

![Exported image](Exported%20image%2020251206152259-8.png) ![Exported image](Exported%20image%2020251206152303-9.png) ![Exported image](Exported%20image%2020251206152305-10.png)  

```
1.在区域中：有几个路由器就有几个1类，有几个DR就有几个2类（或者说网段也行）  
area 0内 存在几条1类 和 几条2类LSA？  
             2          1  
area 1内 存在几条1类 和 几条2类LSA？  
             3          2  
2.ABR会将1类，2类LSA转换为3类LSA。（这里没有环回口，不考虑1类）  
area 0内存在几条3类LSA？  
            2  
area 1内存在几条3类LSA？  
            1
```

![Exported image](Exported%20image%2020251206152307-11.png)

```
区域间的路由计算开销：  
  ABR产生的3类LSA会 携带ABR 到路由产生设备的路径开销  
  区域内的设备收到3类LSA后，会先计算到达ABR的开销 在 叠加LSA携带的开销
```
 
```
如图：图中AR3存在环回接口，那么AR11到AR3的开销是多少呢？
```

![Exported image](Exported%20image%2020251206152310-12.png)  

```
1.[AR11]display ip routing-table 3.3.3.3，可以看到开销是3
```

![Exported image](Exported%20image%2020251206152312-13.png)  

```
2.[AR11]display ospf lsdb，看到开销是2，为什么呢？  
因为：ABR接收Area 1 的1，2类LSA信息，并转换为3类LSA信息，但是开销是按照ABR到目的地的开销来计算的所以是2  
3.为什么路由器算出来的开销是3呢？  
因为：AR11得到路由后，先计算自己到ABR的开销，是1，然后再将ABR到目的的开销2，相加得到3
```

![Exported image](Exported%20image%2020251206152313-14.png)  

```
3类LSA的撤销、更新：（就只有时间值变化）  
更新：seq=1，age=0，携带需要泛洪的路由信息  
Sum-Net   3.3.3.3         10.1.1.1             1  28    80000001
```
 
```
撤销：seq=1，age=3600，携带需要撤销的路由信息  
Sum-Net   3.3.3.3         10.1.1.1          3600  28    80000001
```

![Exported image](Exported%20image%2020251206152316-15.png) ![Exported image](Exported%20image%2020251206152321-16.png) ![Exported image](Exported%20image%2020251206152324-17.png)

```
如何修改三类LSA中的Priority字段：  
1.配置网络前缀：  
[AR11]ip ip-prefix 1 permit 3.3.3.3 32  
2.配置优先级  
[AR11]ospf 1  
[AR11-ospf-1]prefix-priority ?  
  critical  Config the priority of routes as critical  
  high      Config the priority of routes as high  
  medium    Config the priority of routes as medium  
[AR11-ospf-1]prefix-priority high ip-prefix 1 
```