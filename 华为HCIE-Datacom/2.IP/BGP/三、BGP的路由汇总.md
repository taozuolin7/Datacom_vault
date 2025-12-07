```
BGP的路由聚合：  
1.自动聚合：[AR1-bgp]summary automatic   
  *只能聚合引入的路由  
  *只能聚合为有类网段  
  *只有通告的设备才能执行路由聚合  
  聚合后，明细路由和汇总的关系：  
   1.明细路由变为抑制的路由（不能转发的路由条目）  
   2.如果明细路由消失，则汇总也随之消失  
   3.as-path：汇总路由不会继承明细的AS-path（汇总路由的AS-path列表为空）  
   4.origin：汇总路由不会继承明细的origin（汇总路由的origin为?）  
   5.nexthop：汇总路由会将下一跳设置为127.0.0.1（代指新产生的路由条目）  
2.手工聚合：[AR2-bgp]aggregate 172.16.1.0 30   
  *可以聚合所有的BGP路由（network、import、从邻居学习）  
  *可以聚合为无类网段  
  *任意设备对于学习到的BGP路由都可以执行聚合  
  聚合后，明细路由和汇总路由的关系：  
   1.默认聚合完成后，明细路由没有任何变化  
   2.如果明细路由消失，则汇总也随之消失  
   3.as-path：汇总路由默认不会继承明细的AS-path，可以通过配置参数继承明细路由的AS-path  
   4.origin：汇总路由会继承明细的origin（继承明细路由最差的origin值）  
   5.nexthop：汇总路由会将下一跳设置为127.0.0.1（代指新产生的路由条目）
```
 
```
  手工聚合路由后 可以通过添加参数实现明细路由 和 汇总路由的控制：  
  as-set                为聚合路由添加，明细路由经过的AS-path  
  attribute-policy      为聚合路由修改属性的策略（类似于宣告路由时添加的route-policy）  
  detail-suppressed     聚合路由后，明细路由被抑制不转发  
  origin-policy         通过（route-policy）匹配明细路由，指定聚合路由的生成  
  suppress-policy       通过指定设置，选择明细路由是否被抑制
```

![Exported image](Exported%20image%2020251206164624-0.png)

```
聚合后抑制明细路由：￼[AR3-bgp]aggregate 100.1.1.0 255.255.255.0 detail-suppressed
```

```
BGP聚合的环路：  
AS 100内存在路由信息100.1.1.1/32  
AR3执行路由聚合，汇总为 100.1.1.0/24，传递给AR4  
AR4传递给AR6  
AR6进一步将路由聚合，汇总为100.1.0.0/16，传递给AR3  
以此形成了路由环路（理论上的路由环路）
```
 
```
实际上，BGP为了防止数据访问环路 做了优化  
BGP执行了路由聚合后，会为聚合的路由生成一条指向null0的空路由  
如果行成了环路，则会通过路由黑洞丢掉
```

![Exported image](Exported%20image%2020251206164629-1.png)  

```
#as-set 的使用方法：  
默认情况下，聚合路由不会继承明细路由的as-path  
所以失去了AS-path的AS间防环功能  
如果聚合路由继承明细的as-path则路由环路自然解决
```

![Exported image](Exported%20image%2020251206164630-2.png)  

```
#origin-policy的使用方法：  
ip ip-p 1 p 100.1.1.1 32   
route-policy ori permit node 10   
 if-match ip-prefix 1   
[AR3-bgp]aggregate 100.1.1.0 255.255.255.0 origin-policy ori  detail-suppressed   
现象：  
[AR3-bgp]dis bgp routing-table 
```
 
```
      Network            NextHop        MED        LocPrf    PrefVal Path/Ogn  
 *\>   100.1.1.0/24       127.0.0.1                             0      i  
 s\>i  100.1.1.1/32       1.1.1.1         0          100        0      i  
 *\>   100.1.1.2/32       0.0.0.0         0                     0      ?￼￼**就匹配的100.1.1.1做了抑制，而100.1.1.2没有被抑制
```

```
suppress-policy调用方式：
```
 
```
[AR3]ip ip-prefix 1 index 10 permit 100.1.1.1 32   
[AR3]ip ip-prefix 2 index 10 permit 100.1.1.2 32 
```
 
```
[AR3]route-policy test permit node 10  
[AR3-route-policy]if-match ip-prefix 1
```
 
```
[AR3]route-policy test deny node 20   
[AR3-route-policy]if-match ip-prefix 2
```
 
```
[AR3-bgp]aggregate 100.1.1.0 255.255.255.0 detail-suppressed test
```
 
```
现象：  
[AR3]dis bgp routing-table 
```
 
```
      Network            NextHop        MED        LocPrf    PrefVal Path/Ogn  
 *\>   100.1.1.0/24       127.0.0.1                             0      ?  
 s\>i  100.1.1.1/32       1.1.1.1         0          100        0      i  
 *\>   100.1.1.2/32       0.0.0.0         0                     0      ?
```