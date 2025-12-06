\> [!caution] This page contained a drawing which was not converted.   

```
通过BGP学习到对端的路由信息，且路由信息是最优的可用的  
该路由信息就会加入到IP路由表中，且flag字段的Rbit置位表示递归路由
```

```
BGP路由的生成：  
***BGP无法通过链路状态等信息计算路由，只能发布或接收完整的路由信息  
1.通过network进行路由宣告[AR4-bgp]network 100.4.4.4 32  
   1.该方式通告的路由信息，是需要在IP路由表中存在的网段  
   2.所有协议学习的路由信息都可以被BGP进行network  
2.通过import进行路由通告[AR4-bgp]import-route direct | static | ospf 1 | isis 1 |  
   1.该方式通告的路由信息，是对应协议的所有路由信息（路由表中最优，但是直连除外）  
   2.所有协议的路由信息都可以被引入到BGP中  
3.通过路由聚合的方式进行路由通告[AR4-bgp]summary automatic   
4.从邻居学习到的（路由传递的方式）（以ASBR为视角）  
   1.从EBGP邻居可以正常学习路由信息，可以传递给IBGP邻居  
        路由是不可用、非优先的，因为传递的路由的下一跳没有改变，导致下一跳是不可达的  
        因为BGP的路由都是需要进行递归、迭代的，所以BGP的路由需要根据下一跳进行路由递归  
        对于BGP路由下一跳不可达：  
              1.通过路由信息的学习，使得下一跳可达：配置静态路由（不切实际）  
              2.下一跳转变为可达的地址 ：[AR3-bgp]peer 1.1.1.1 next-hop-local   
                  向IBGP邻居传递路由信息时，将路由的下一跳变为自身的更新源地址
```
 
```
   2.从EBGP邻居可以正常学习路由信息，可以传递给EBGP邻居  
        路由是最优的、可用的，下一跳为邻居的更新源地址（建立邻居的地址）
```
 
```
   3.从IBGP邻居可以正常学习路由信息，可以传递给EBGP邻居  
        **优化后的结果是：路由是最优的、可用的，下一跳为邻居的更新源地址（建立邻居的地址）
```
 
```
        没有优化之前，需要满足BGP与IGP同步，才可以实现路由信息传递给EBGP邻居  
        为了防止路由黑洞的出现  
        要求在将BGP路由信息传递给EBGP邻居时，需要满足在IGP也学习到对应的路由信息  
        保障AS内所有设备都是存在路由信息的（给到对端AS的路由条目，本端AS内所有的设备都存在路由信息）  
        优化的操作是将BGP与IGP同步的功能 默认关闭（无法手动开启）[undo  synchronization 撤销同步]
```
 
```
   4.从IBGP邻居可以正常学习路由信息，不可以传递给IBGP邻居  
        为了路由信息在AS内传递 防止环路的（AS内水平分割）
```

```
[AR4]display bgp routing-table 
```
 
```
 BGP Local router ID is 10.4.4.4   
 Status codes: * - valid, \> - best, d - damped,  
               h - history,  i - internal, s - suppressed, S - Stale  
               Origin : i - IGP, e - EGP, ? - incomplete
```
   

```
 Total Number of Routes: 1  
      Network            NextHop        MED        LocPrf    PrefVal Path/Ogn
```
 
```
 *\>   100.4.4.4/32       0.0.0.0         0                     0      i  
*代表路由可用   \>代表路由最优
```

![Exported image](Exported%20image%2020251206164554-0.png)

```
对于BGP路由的访问：需要通过指定源去访问
```
 
```
BGP的数据访问还是要通过物理链路访问的，所以要求经过的每一台设备都要存在BGP的路由信息  
如果存在设备没有路由信息，则会形成BGP的路由黑洞  
如何解决BGP的路由黑洞：  
  1.从EBGP邻居学习到的路由，引入到IGP内  
  2.通过VPN隧道来实现  
  3.建立全互联的IBGP邻居 （IBGP邻居关系建立的数量 和 设备的数量 成正比）  
  4.通过建立RR反射器 （减少AS内IBGP邻居关系数量）
```

```
通过BGP学习到对端的路由信息，且路由信息是最优的可用的  
该路由信息就会加入到IP路由表中，且flag字段的Rbit置位表示递归路由
```

![Exported image](Exported%20image%2020251206164555-1.png)