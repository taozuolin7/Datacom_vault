前置配置：  
AS内底层配置IGP互通（ospf ，isis），￼全局使能MPLS LDP￼接口使能mpls， mpls ldp
 
方案配置：  
1.配置同一AS内的PE与ASBR之间建立IBGP邻居  
2.配置ASBR之间建立EBGP邻居￼3.配置PE建立MP-BGP邻居￼4.配置ASBR关闭VPN-Targe检查￼5.配置IBGP，EBGP邻居标签路由转发能力￼5.配置路由策略重分发MPLS 标签
 ![Exported image](Exported%20image%2020251206151654-0.png)

```
PE1  
ip vpn-instance A  
 ipv4-family  
  route-distinguisher 100:1  
  vpn-target 1:1 export-extcommunity  
  vpn-target 1:1 import-extcommunity  
#  
bgp 10  
 peer 4.4.4.4 as-number 10   
 peer 4.4.4.4 connect-interface LoopBack0  
 peer 7.7.7.7 as-number 100   
 peer 7.7.7.7 ebgp-max-hop 10   
 peer 7.7.7.7 connect-interface LoopBack0  
 #  
 ipv4-family unicast  
  undo synchronization  
  peer 4.4.4.4 enable  
  peer 4.4.4.4 label-route-capability  
  undo peer 7.7.7.7 enable  
 #   
 ipv4-family vpnv4  
  policy vpn-target  
  peer 7.7.7.7 enable  
 #  
 ipv4-family vpn-instance A   
  network 192.168.1.0 24  
#  
ospf 2 router-id 2.2.2.2 vpn-instance A  
 import-route bgp  
 area 0.0.0.0 
```

```
ASBR1：  
#  
bgp 10  
 peer 2.2.2.2 as-number 10   
 peer 2.2.2.2 connect-interface LoopBack0  
 peer 10.0.45.5 as-number 100   
 #  
 ipv4-family unicast  
  undo synchronization  
  network 2.2.2.2 255.255.255.255   
  peer 2.2.2.2 enable  
  peer 2.2.2.2 route-policy 1 export  
  peer 2.2.2.2 label-route-capability  
  peer 10.0.45.5 enable  
  peer 10.0.45.5 route-policy 2 export  
  peer 10.0.45.5 label-route-capability  
 #   
 ipv4-family vpnv4  
  undo policy vpn-target  
#  
Route-policy 1 permit node 10   
  if-match mpls-label  
  apply mpls-label   
Route-policy 2 permit node 10    
  apply mpls-label 
```

```
PE2  
#  
ip vpn-instance C  
 ipv4-family  
  route-distinguisher 200:1  
  vpn-target 1:1 export-extcommunity  
  vpn-target 1:1 import-extcommunity  
#  
bgp 100  
 peer 2.2.2.2 as-number 10   
 peer 2.2.2.2 ebgp-max-hop 10   
 peer 2.2.2.2 connect-interface LoopBack0  
 peer 5.5.5.5 as-number 100   
 peer 5.5.5.5 connect-interface LoopBack0  
 #  
 ipv4-family unicast  
  undo synchronization  
  peer 2.2.2.2 enable  
  peer 5.5.5.5 enable  
  peer 5.5.5.5 label-route-capability  
 #   
 ipv4-family vpnv4  
  policy vpn-target  
  peer 2.2.2.2 enable  
 #  
 ipv4-family vpn-instance C   
  network 192.168.2.0   
#  
ospf 2 router-id 2.7.7.7 vpn-instance C  
 import-route bgp  
 area 0.0.0.0 
```

```
ASBR2：  
#  
bgp 100  
 peer 7.7.7.7 as-number 100   
 peer 7.7.7.7 connect-interface LoopBack0  
 peer 10.0.45.4 as-number 10   
 #  
 ipv4-family unicast  
  undo synchronization  
  network 7.7.7.7 255.255.255.255   
  peer 7.7.7.7 enable  
  peer 7.7.7.7 route-policy 1 export  
  peer 7.7.7.7 label-route-capability  
  peer 10.0.45.4 enable  
  peer 10.0.45.4 route-policy 2 export  
  peer 10.0.45.4 label-route-capability  
 #   
 ipv4-family vpnv4  
  undo policy vpn-target  
#  
Route-policy 1 permit node 10   
  if-match mpls-label  
  apply mpls-label   
Route-policy 2 permit node 10    
  apply mpls-label 
```

控制平面：  
PC1访问PC2：￼1.PE2将192.168.2.0/24通告在 ipv4-family vpn-instance C 邻居当中，PE2为其分配一个私网标签1026；￼2.涉及到跨AS传输，所以会封装一个介于私网标签和公网标签之间的一个标签：1027  
（该标签的作用是，在ASBR之间模拟公网标签进行数据包的传输）  
3.PE2的环回口，将自己的入标签1025 通告给：IBGP邻居ASBR2，到达ASBR2之前的一跳会将公网标签pop掉￼4.ASBR2因为存在route-policy，会将介于公网标签和私网标签之间的标签重新分配￼5.ASBR1会将PE2 的路由通告IBGP邻居通告给PE1，并且告知PE2，该路由跨AS，需要中间件标签￼6.PE1通过RT值，将该路由传递给与该RT值相匹配的接口

数据平面：￼1.当数据包到达PE1，查看VPN-Istance A 路由表，发现是从EBGP邻居学习来的，并且需要递归到7.7.7.7￼\<PE1\>dis ip routing-table vpn-instance A 192.168.2.0

![Exported image](Exported%20image%2020251206151656-1.png)  

2.查看VPNv4路由表，接收到1026的私网标签，需要封装1026私网标签￼[PE1]display bgp vpnv4 all routing-table 192.168.2.0

![Exported image](Exported%20image%2020251206151658-2.png)  

3.查看去往7.7.7.7是否需要通过MPLS 标签封装，TunnelID不为0，需要封装标签  
[PE1] dis fib

![Exported image](Exported%20image%2020251206151700-3.png)  

4.查看封装的去往7.7.7.7封装的标签，该标签是在私网标签与公网标签之中￼[PE1]display mpls lsp

![Exported image](Exported%20image%2020251206151703-4.png)  

5.问题来了：有去往7.7.7.7的标签，但是下一跳该走哪里？￼因为7.7.7.7路由时ASBR1在BGP引入到OSPF中，所以：要去查看公共路由表￼[PE1]display ip routing-table 7.7.7.7

![Exported image](Exported%20image%2020251206151708-5.png)  

6.需要迭代到4.4.4.4，查看去往4.4.4.4是否需要MPLS封装，TunnelID非0，需要公网标签封装

![Exported image](Exported%20image%2020251206151709-6.png)

￼7.去往7.7.7.7迭代到4.4.4.4封装1024公网标签￼[PE1]display mpls lsp include 4.4.4.4 32

![Exported image](Exported%20image%2020251206151712-7.png)  
![Exported image](Exported%20image%2020251206151714-8.png)  

8.当数据包到达ASBR1后，会将公网标签弹出，查找去往7.7.7.7 的路由，通过EBGP从10.0.45.5学习到7.7.7.7

![Exported image](Exported%20image%2020251206151715-9.png) ![Exported image](Exported%20image%2020251206151717-10.png)  

9.当数据包从ASBR1发出时，因为做了route-policy，重新分发了标签为1025

![Exported image](Exported%20image%2020251206151718-11.png)

\<ASBR1\>dis mpls lsp

![Exported image](Exported%20image%2020251206151723-12.png)

10.数据包到达ASBR2上后，查看路由表：通过OSPF学习到7.7.7.7，需要通过mpls封装标签

![Exported image](Exported%20image%2020251206151731-13.png) ![Exported image](Exported%20image%2020251206151734-14.png)  
![Exported image](Exported%20image%2020251206151736-15.png)  

11.数据包到达PE2时，pop公网标签，只剩下标签，然后PE2查表，1026标签时为VPN-Instance C标识，￼数据包从VPN-Instance绑定的接口发出

![Exported image](Exported%20image%2020251206151739-16.png) ![Exported image](Exported%20image%2020251206151744-17.png)  

最后数据到达CE2，CE2查表，到网关，再到PC2

为什么需要重新分布标签呢？￼因为：在AS10内使用的中间层标签可能，在AS100内可能以及被其他标签占用，如果不重新分发标签，回导致找不到路

`