\> [!caution] This page contained a drawing which was not converted.   

```
路径属 性分类：  
1.公认必遵：所有设备必须识别，且必须携带的属性  
   1.as-path   2.origin   3.nexthop  
2.公认任意：所有设备必须识别，根据自身的需求选择是否携带该属性  
   1.Local_Preference  2.Atomic_aggregate   
3.可选过渡：所有设备可以不识别该属性，但是收到该属性后可以传递给其他邻居  
   1.Aggregator   2.Community  
4.可选非过渡：所有设备可以不识别该属性，收到该属性后不可以传递给其他邻居  
   1.MED     2.Cluster-List   3.Originator-ID 
```

```
as-path：AS路径  
  作用：1.选路   2.防环  
  EBGP邻居在传递路由信息时，会将自身的AS编号，写入as-path属性中，给路由携带  
  as-path属性值按照从右往左依次添加（让路由记录了传递过的路径）  
  选路的方式：  
    接收EBGP邻居传递的路由信息，会比较as-path的数量，路由经过的AS-path数量越少越优先  
  防环的方式：  
    EBGP邻居不接收 携带了自身AS号的路由信息
```
 
```
  通过策略修改AS-path值：  
    可以对收到路由的as-path列表进行 添加、清空、复写 操作  
    [AR7]ip ip-prefix test index 10 permit 100.5.8.5 32   
    [AR7]route-policy AS permit node 10  
    [AR7-route-policy]if-match ip-prefix test  
   [AR7-route-policy]apply as-path 800 200 additive   添加                                                                                              
    [AR7]bgp 700  
    [AR7-bgp]peer 10.1.67.6 route-policy AS import        策略是入方向，则会立即生效（因为设备会主动发送route-refresh报文请求）
```
 
```
    清空  
    [AR7]route-policy none-as permit node 10   
    [AR7-route-policy]apply as-path none overwrite   清空as列表  
    [AR7]bgp 700  
    [AR7-bgp]peer 10.1.78.8 route-policy none-as export   策略是出方向，则需要等待策略的生效时间
```
 
```
  AS-path的分类：  
    1.有序AS列表：AS直接正常交互的路由信息 记录的AS-path  
       *\>   100.5.8.5/32       10.1.67.6  0      800 200 600 500i  
    2.无序AS列表：聚合后的汇总路由记录的AS-path  
    3.联盟内有序AS列表  
    4.联盟内无序AS列表
```

```
[AR5]dis bgp routing-table peer 10.1.56.6 advertised-routes 100.5.8.5  
    查看发送给邻居路由时，添加的详细信息
```

```
***BGP只传递最优的路由条目***
```

![Exported image](Exported%20image%2020251206164601-0.png)

```
origin：起源属性  
  作用：1.选路  
  记录了路由信息生成的方式  
  i：network的方式生成的路由  
  e：通过egp协议生成的路由  
  ?：import的方式生成的路由 
```
 
```
  按照i \> e \> ? 的方式优选  
  该属性默认情况不会更改，只有管理员手工进行策略修改
```
 
```
  修改起源属性的策略：  
  [AR7]ip ip-prefix A index 10 permit 172.16.5.5 32
```
 
```
  [AR7]route-policy origin permit node 10  
  [AR7-route-policy]if-match ip-prefix A  
  [AR7-route-policy]apply origin egp 123   #123 标识来源EGP 的AS号，       
               #为什么需要，因为本网络都是BGP，所有需要标识egp的来源。  
   
  [AR7]route-policy origin permit node 1000  
  [AR7]bgp 700  
  [AR7-bgp]peer 10.1.67.6 route-policy origin import  
  [AR7-bgp]peer 10.1.78.8 route-policy origin export   
  对于起源属性的策略可以在入方向或出方向更改
```

```
next hop：下一跳  
  作用：确保路由是可达的  
  设备通过network生成的路由下一跳为0.0.0.0  
  设备通过import生成的路由下一跳为0.0.0.0  
  设备通过聚合生成的路由下一跳为127.0.0.1  
  设备通过邻居学习的路由下一跳为更新源地址
```
 
```
  从EBGP邻居收到的路由传递给IBGP邻居时，下一跳不会改变  
  导致IBGP邻居收到路由是不可达的  
  需要在ASBR设备上对IBGP邻居配置下一跳变为自身的更新源地址
```

```
local preference：本地优先级  
   作用：1.选路  
   BGP路由存在本地 优先级值，该值越大越优先，默认值为100  
   该属性只能在IBGP邻居之间传递，在EBGP邻居之间该属性无效  
  *该属性的含义，是为了本AS多出口，优选某一个出口  
  *影响AS内设备的路径选择  
     
   本地优先级值的策略修改：  
   [AR4]route-policy LP permit node 10  
   [AR4-route-policy]apply local-preference 1234  
   [AR4]bgp 100   
   [AR4-bgp]peer 2.2.2.2 route-policy LP export     IBGP邻居的出方向修改  
   [AR4-bgp]peer 10.1.45.5 route-policy LP import   EBGP邻居的入方向修改
```
 
```
   [AR2-bgp]peer 3.3.3.3 route-policy LP import     IBGP邻居的入方向修改
```

```
community：团体属性  
  类似于route tag，用于路由信息的标记  
  route tag 每条路由只能携带一个标记，不能指示路由的多重含义  
  而团体属性每条路由可以携带多个，
```
 
```
  团体属性的分类：  
   1.公认团体属性：赋予路由可以传递的范围  
     internet   ：默认使用的值，路由信息的传递不受限制  
     no-advertise ：不通告，不会将路由发送给IBGP和EBGP邻居  
     no-export   ：不发出，不将路由发送给EBGP邻居，发送给IBGP邻居  
     no-export-subconfed：联盟内不发出  
   2.自定义团体属性：为路由手工设置团体值，在通过匹配团体值执行策略  
     团体属性值的设置两种方式：  
       1.数字表示范围\<0-4294967295\>  
       2.AA:NN表示取值范围aa\<0-65535\>:nn\<0-65535\>   一般使用的方式  
       
   *团体属性的传递方式需要邻居之间相互支持团体属性传递的能力  
    [AR3-bgp]peer 10.1.35.5 advertise-community
```

```
[AR3]route-policy com permit node 10   
[AR3-route-policy]apply community internet   
[AR3]bgp 100   
[AR3-bgp]network 100.3.3.3 32 route-policy com 
```

```
100.2.2.2  3:4 3:3 34:34   
           2:4 4:4 34:34 
```
 
```
100.3.3.3  3:4 3:3 34:34   
100.4.4.4  2:4 4:4 34:34 
```

```
对于团体属性，BGP具备独特的匹配工具ip-community-filter  
可以使用执行工具route-policy调用
```

```
[AR3]route-policy com permit node 10  
[AR3-route-policy]apply community 3:4 3:3 34:34   
[AR3]bgp 100   
[AR3-bgp]network 100.3.3.3 32 route-policy com 
```

```
[AR4]route-policy com permit node 10  
[AR4-route-policy]apply community 2:4 4:4 34:34   
[AR4]bgp 100   
[AR4-bgp]network 100.4.4.4 32 route-policy com 
```

```
[AR5]ip community-filter 1 permit 2:4 
```
 
```
[AR5]route-policy test permit node 10  
[AR5-route-policy]if-match community-filter 1   
[AR5-route-policy]apply local-preference 1000
```
 
```
[AR5]bgp 500   
[AR5-bgp]peer 10.1.45.4 route-policy test import 
```

```
100.2.2.2  2:4 4:4 34:34   
100.3.3.3  3:4 3:3 34:34   
100.4.4.4  2:4 4:4 34:34 
```

```
MED：多出口鉴别器  
   作用：1.选路  
  *本质是本AS内发布的路由可以直接影响对端AS的出口选路  
   实际上就是BGP路由的开销  
   该属性值只能传递到邻居的AS，不能跨越AS传递  
   比较规则，MED值越小越优先
```
 
```
   本AS接收邻居的路由比较MED的要求：  
     1.路由要求是来自同一个AS  
     2.可以通过配置命令让BGP比较不同AS的路由MED   
       BGP视图下配置：compare-different-as-med
```

![Exported image](Exported%20image%2020251206164602-1.png) 
 
a