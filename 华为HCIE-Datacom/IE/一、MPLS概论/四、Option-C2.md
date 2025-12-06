前置配置：  
AS内底层配置IGP互通（ospf ，isis），￼全局使能MPLS LDP￼接口使能mpls， mpls ldp  
￼关键配置：  
ASBR间建立EBGP对等体关系  
将域内PE的路由发布给远端PE  
使能标签IPv4路由交换能力  
为带标签的公网BGP路由建立LDP LSP  
PE间建立MP-EBGP对等体关系
 ![Exported image](Exported%20image%2020251206151751-0.png)

```
PE1  
ip vpn-instance A  
 ipv4-family  
  route-distinguisher 100:1  
  vpn-target 1:1 export-extcommunity  
  vpn-target 1:1 import-extcommunity
```
 
```
ospf 2 router-id 2.2.2.2 vpn-instance A  
 import-route bgp  
 area 0.0.0.0   
#  
bgp 10  
 peer 7.7.7.7 as-number 100   
 peer 7.7.7.7 ebgp-max-hop 10   
 peer 7.7.7.7 connect-interface LoopBack0  
 #  
 ipv4-family unicast  
  undo peer 7.7.7.7 enable  
 #   
 ipv4-family vpnv4  
  policy vpn-target  
  peer 7.7.7.7 enable  
 #  
 ipv4-family vpn-instance A   
  network 192.168.1.0 
```

```
ASBR1：  
#  
bgp 10  
 peer 10.0.45.5 as-number 100   
 #  
 ipv4-family unicast  
  network 2.2.2.2 255.255.255.255    
  peer 10.0.45.5 enable  
  peer 10.0.45.5 route-policy 2 export  
  peer 10.0.45.5 label-route-capability  
 #   
 ipv4-family vpnv4  
  undo policy vpn-target  
#  
ospf 1 r 1.4.4.4  
   import-route bgp  
#  
interface GigabitEthernet0/0/1  
 ip address 10.0.45.4 255.255.255.0   
 mpls  
#  
mpls  
 lsp-trigger bgp-label-route  
#  
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
ospf 2 router-id 2.7.7.7 vpn-instance C  
 import-route bgp  
 area 0.0.0.0   
#  
bgp 100  
 peer 2.2.2.2 as-number 10   
 peer 2.2.2.2 ebgp-max-hop 10   
 peer 2.2.2.2 connect-interface LoopBack0  
 #  
 ipv4-family unicast  
  undo peer 2.2.2.2 enable  
 #   
 ipv4-family vpnv4  
  policy vpn-target  
  peer 2.2.2.2 enable  
 #  
 ipv4-family vpn-instance C   
  network 192.168.2.0 
```

```
ASBR2：  
#  
bgp 100  
 peer 10.0.45.4 as-number 10   
 #  
 ipv4-family unicast  
  network 7.7.7.7 255.255.255.255   
  peer 10.0.45.4 enable  
  peer 10.0.45.4 route-policy 2 export  
  peer 10.0.45.4 label-route-capability  
 #   
 ipv4-family vpnv4  
  undo policy vpn-target  
#  
ospf 1 r 1.5.5.5  
   import-route bgp  
#  
interface GigabitEthernet0/0/1  
 ip address 10.0.45.5 255.255.255.0   
 mpls  
#  
mpls  
 lsp-trigger bgp-label-route  
#  
Route-policy 2 permit node 10   
  apply mpls-label 
```

与Option-C1方案的区别：￼1.不再需要PE与ASBR之间建立IBGP邻居￼2.不再需要中间件标签，通过 lsp-trigger bgp-label-route ，代替该功能￼3.PE1之间路由的学习不再需要通路的BGP学习到，通过在ASBR中的IGP中引入BGP实习

控制平面：￼1.PE之间，之间建立LDP LSP，而不是BGP LSP￼2.P1到ASBR之间的标签不会只剩下私网标签，而是存在公网标签，但是公网标签与私网标签一样，￼因为，PE1与PE2之间是通过LDP LSP分发的标签，所以PE1到ASBR1不算做倒数第二天，￼而是，到PE2才算做是倒数第二跳，才会把公网标签pop出。

![Exported image](Exported%20image%2020251206151756-1.png)  
![Exported image](Exported%20image%2020251206151759-2.png)

数据平面：￼1.PE1查表：￼[PE1]display bgp vpnv4 all routing-table 192.168.2.0，发现去往192.168.2.0/24需要进行私网标签1027封装，下一跳是7.7.7.7需要递归到IGP

![Exported image](Exported%20image%2020251206151802-3.png)  
![Exported image](Exported%20image%2020251206151805-4.png)  

2.去往7.7.7.7的TunnelID非0，需要mpls‘封装 1025标签￼[PE1]display fib 7.7.7.7

![Exported image](Exported%20image%2020251206151807-5.png)  
![Exported image](Exported%20image%2020251206151809-6.png)  
![Exported image](Exported%20image%2020251206151811-7.png)  

3.通过中间结点：公网标签替换为1027，与私网标签一致￼[P1]dis mpls lsp

![Exported image](Exported%20image%2020251206151818-8.png)

￼公网标签

![Exported image](Exported%20image%2020251206151825-9.png)  

4.数据到达ASBR1后，因为存在route-policy，会更新mpls标签，该标签会一直沿用到倒数第二跳弹出

![Exported image](Exported%20image%2020251206151826-10.png)  

6.数据包到达PE2，后公网标签弹出，PE2更具私网标签，将数包送到对应的接口￼

![Exported image](Exported%20image%2020251206151828-11.png)