```
区域内：完成单区域内拓扑计算和路由信息计算
```
 
```
区域间：完成两个或两个以上区域的路由计算
```
 
```
区域外：完成不同路由协议之间的路由计算
```

```
为什么要规划区域外？  
  为了和不同的路由协议进行对接
```
 
```
如何判断是区域外的信息？  
  需要将非OSPF协议的路由，放入到OSPF协议中，这样的路由都是区域外路由
```
 
```
对于存在区域外路由的设备，称为ASBR设备（自治系统边界设备）  
ASBR设备可以是OSPF系统中的任意一台设备
```
 
```
区域外路由信息如何放入OSPF协议中？  
  通过引入的方式来实现  
  [AR3-ospf-1]import-route static  在OSPF试图下引入静态路由  
执行引入路由后，会有什么现象产生？  
  1.OSPF协议内的设备会自动计算出区域外的路由信息  
     192.168.3.0/24   OSPF外部路由的协议表示O_ASE   
OSPF外部路由的优先级值：150  
  2.执行外部路由引入的设备会称自身为ASBR设备  
     在1类LSA的flag字段中将Ebit置位  
  3.引入的路由会传递给OSPF自治系统内的所有设备
```

![Exported image](Exported%20image%2020251206152328-0.png) ![Exported image](Exported%20image%2020251206152330-1.png) ![Exported image](Exported%20image%2020251206152333-2.png) ![Exported image](Exported%20image%2020251206152335-3.png) ![Exported image](Exported%20image%2020251206152337-4.png)

```
引入的路由，OSPF是如何传递给自治系统内的所有设备？  
  在引入路由后，ASBR设备会生成一个新的LSDB（AS External   
Database），和区域内的LSDB分开  
  产生的新的LSDB放入到新的LSDB中（该LSA为 AS external   
LSA，即5类LSA）
```
 
```
5类LSA的具体信息：  
作用：描述外部路由信息的  
谁产生的：ASBR设备产生的  
泛洪范围：整个OSPF自治系统泛洪  
[AR3]display ospf lsdb ase 192.168.3.0   
5类LSA的头部  
  Type      : External     5类LSA  
  Ls id     : 192.168.3.0  外部路由的网络号  
  Adv rtr   : 10.3.3.3     ASBR的RID  
  Ls age    : 831   
  Len       : 36   
  Options   :  E    
  seq#      : 80000001   
  chksum    : 0x92be  
5类LSA的内容：  
  Net mask  : 255.255.255.0   外部路由的掩码  
  TOS 0  Metric: 1            外部路由的开销  
  E type    : 2               外部路由的开销类型  
  Forwarding Address : 0.0.0.0 外部路由的转发地址  
  Tag       : 1               外部路由的路由标记  
  Priority  : Low
```

![Exported image](Exported%20image%2020251206152342-5.png)

```
4类LSA：  
[AR1]dis ospf lsdb asbr 10.3.3.3   
作用：描述去往ASBR的路由信息  
谁产生的：ABR产生  
泛洪范围：产生的每一个区域内泛洪  
4类LSA的头部：  
  Type      : Sum-Asbr    4类LSA  
  Ls id     : 10.3.3.3    ASBR的RID  
  Adv rtr   : 10.2.2.2    ABR的RID  
  Ls age    : 1028   
  Len       : 28   
  Options   :  E    
  seq#      : 80000002   
  chksum    : 0xfa3c  
  Tos 0  metric: 1        ABR去往ASBR的开销
```
 
```
4类LSA和5类LSA的关系：  
一般情况下，4 类 LSA 与 5 类 LSA 是伴随出现的，因为：
```

- ==当 ASBR 将外部路由引入 OSPF 时，会生成 5 类 LSA==
- ==连接 ASBR 所在区域的 ABR 会收到 5 类 LSA，并生成 4 类 LSA 通告给其他区域==

**但存在例外情况**，区域内可能有 4 类 LSA 而无 5 类 LSA：

1. ==ASBR 上配置了====import-route====命令但没有成功引入任何外部路由 (如被过滤)==
2. ==ASBR 生成了 5 类 LSA，但由于访问控制列表等原因，5 类 LSA 未能在区域内泛洪，而 4 类 LSA 已生成==
 
```
  ABR收到5类LSA后，泛洪到其他连接的区域，并在区域中产生4类LSA泛洪  
  区域内的IR设备收到4类LSA和5类LSA时：  
    判断5类LSA的adv router无法识别，在根据4类LSA判断去往5类LSA的adv router需要先去往ABR，再由ABR在去往ASBR。
```
 
```
*区域内存在5类LSA，一定会存在4类LSA？  
   不一定，4类LSA的作用是为了找到5类的adv router  
   （ASBR所在的区域不会存在对应自身的4类LSA）
```

![Exported image](Exported%20image%2020251206152344-6.png)  

```
*区域内存在4类LSA，一定会存在5类LSA？  
（ABR产生4类LSA的判断条件）  
   ABR产生4类LSA的条件是收到设备的1类LSA中flags字段的Ebit置位（而不是收到5类LSA）
```

- ==虽然 4 类 LSA 通常与 5 类 LSA 一起出现，但==**4 类 LSA 的生成直接依赖于****ABR****对 ASBR 的识别 (通过 1 类 LSA 的 Ebit 位)，而非直接依赖于收到 5 类 LSA**
- ==从逻辑时序看：先有 ASBR 生成带 Ebit 位的 1 类 LSA → ABR 识别后生成 4 类 LSA → ASBR 生成 5 类 LSA 并泛洪==
 ![Exported image](Exported%20image%2020251206152345-7.png)  

```
 *设备只需要执行引入的动作（空引入），就会成为ASBR设备
```
 
```
为什么不能直接将5类LSA在ABR泛洪时修改adv router为ABR的RID？  
  因为5类LSA描述的是外部路由，如果在泛洪时修改adv router为ABR的RID，则和内部路由没有区分了
```
 
```
**只有1，2类是拓扑信息，3，4，5，7都是路由信息
```

```
5类LSA如何计算路由？  
  5类LSA的开销类型：  
   1.type 1  
      外部路由引入形成的5类LSA，会默认携带1的开销值  
      外部路由计算，会计算5类LSA携带的开销值，同时叠加计算路径开销  
      外部路由的开销 = 5类LSA携带的开销 + IR设备到ABR的开销 + ABR到ASBR的开销  
      可以直接在引入时，通过配置修改5类LSA携带的开销值  
      [AR3-ospf-1]import-route static type 1 cost 100 
```
 
```
   2.type 2，默认的开销类型  
      外部路由引入形成的5类LSA，会默认携带1的开销值  
      外部路由计算，只会计算5类LSA携带的开销值，不会叠加计算路径开销  
      可以直接在引入时，通过配置修改5类LSA携带的开销值  
      [AR3-ospf-1]import-route static type 2 cost 100
```
 
```
   type1的意义：设备优选去往外部路由的最优路径  
   type2的意义：可以通过管理员意志优选外部路由
```

```
缺省路由是不能够被路由协议之间相互引入的
```
 
```
OSPF可以通过  
[AR3-ospf-1]default-route-advertise always    
命令实现下发缺省路由的操作  
  该命令执行后，就是让设备成为ASBR设备，并产生一条缺省的5类LSA  
[AR3-ospf-1]default-route-advertise always     
OSPF直接下发缺省的5类LSA   
[AR3-ospf-1]default-route-advertise            
设备要存在非OSPF协议的缺省路由，OSPF才会产生缺省5类LSA
```

![Exported image](Exported%20image%2020251206152347-8.png)  

```
1.在AR3上部署引入外部静态路由；在图中AR1会存在多少条5类LSA？  
（易错点：判断为两条）  
答案：1条，因为在OSPF中5类LSA信息在泛洪后不会做出任何改变  
2.AR1访问AR3会走哪一条路径？  
答案：缺省情况下-负载分担
```

![Exported image](Exported%20image%2020251206152349-9.png)

```
￼2.1如果说AR1的G0/0/0接口的ospf开销改为10，那么AR1去访问AR3会走G0/0/1
```

```
总结：  
1类、2类LSA 优于 3类LSA   优于 5类LSA
```
 
```
ASBR在非骨干区域：  
ABR可以通过非骨干区域的1类访问到ASBR，也可以通过骨干区域的4类访问ASBR  
1类LSA的开销 和 4类LSA的开销进行对比，优选开销值小的
```

![Exported image](Exported%20image%2020251206152351-10.png)   
```
ASBR在骨干区域：  
ABR可以通过骨干区域的1类访问到ASBR，也可以通过非骨干区域的4类访问ASBR  
优选1类LSA
```

![Exported image](Exported%20image%2020251206152353-11.png)   
```
ABR成为ASBR：  
另一台ABR可以通过骨干区域的1类访问到ASBR，也可以通过非骨干区域的1类访问ASBR  
如果开销值相等，则优选area id 大的1类LSA  
如果开销值不等，则优选开销值小的
```

![Exported image](Exported%20image%2020251206152358-12.png)  

```
5类LSA在泛洪时，ABR如何处理？  
  ABR不会执行任何操作（即5类LSA在泛洪传递时，不会有任何信息的改变）  
1.ABR在执行3类LSA泛洪时，会修改3类LSA的Adv router，  
2.区域内IR设备收到3类LSA后，根据Adv router可以判断要先去往ABR，  
3.在由ABR执行数据转发。
```
 
```
ABR执行5类LSA泛洪时，不会修改adv router，区域内IR设备如何得知要怎么去往ASBR？  
  需要通过ABR的帮助，ABR会产生4类LSA，帮助区域内IR设备访问ASBR
```