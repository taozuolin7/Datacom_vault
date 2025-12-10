# Option-B
前置配置：  
AS内底层配置IGP互通（ospf ，isis），￼全局使能MPLS LDP接口使能mpls， mpls ldp
 
方案配置：  
1.配置PE和域内ASBR间使用MP-IBGP  
2.配置不同AS的ASBR间使用MP-EBGP  
3.配置ASBR不对VPNv4路由进行VPN-target过滤  
4.配置ASBR之间互联接口使能mpls
![](assets/13、Option-B/file-20251211000824757.png)

## 具体关键配置：AS10内： 
```
PE1：  
ip vpn-instance A  
 ipv4-family  
  route-distinguisher 100:1  
  vpn-target 1:1 export-extcommunity  
  vpn-target 1:1 import-extcommunity  
#  
bgp 10  
 peer 4.4.4.4 as-number 10   
 peer 4.4.4.4 connect-interface LoopBack0  
 #  
 ipv4-family unicast  
  undo peer 4.4.4.4 enable  
 #   
 ipv4-family vpnv4  
  policy vpn-target  
  peer 4.4.4.4 enable  
 #  
 ipv4-family vpn-instance A   
  network 192.168.1.0   
#  
ospf 2 router-id 2.2.2.2 vpn-instance A  
 import-route bgp  
 area 0.0.0.0 
```
**ASBR1:  **
```
interface GigabitEthernet0/0/1  
 ip address 10.0.45.4 255.255.255.0   
 mpls  
#  
bgp 10  
 peer 2.2.2.2 as-number 10   
 peer 2.2.2.2 connect-interface LoopBack0  
 peer 10.0.45.5 as-number 100   
 #  
 ipv4-family unicast  
  undo synchronization  
  undo peer 2.2.2.2 enable  
  undo peer 10.0.45.5 enable  
 #   
 ipv4-family vpnv4  
  undo policy vpn-target  
  peer 2.2.2.2 enable  
  peer 10.0.45.5 enable

```
**AS 100内 ：PE2： **
```
#  
ip vpn-instance C  
 ipv4-family  
  route-distinguisher 200:1  
  vpn-target 1:1 export-extcommunity  
  vpn-target 1:1 import-extcommunity  
#  
bgp 100  
 peer 5.5.5.5 as-number 100   
 peer 5.5.5.5 connect-interface LoopBack0  
 #  
 ipv4-family unicast  
  undo peer 5.5.5.5 enable  
 #   
 ipv4-family vpnv4  
  policy vpn-target  
  peer 5.5.5.5 enable  
 #  
 ipv4-family vpn-instance C   
  network 192.168.2.0   
#  
ospf 2 router-id 2.7.7.7 vpn-instance C  
 import-route bgp  
 area 0.0.0.0   
#
```
ASBR2:
```
#  
interface GigabitEthernet0/0/1  
 ip address 10.0.45.5 255.255.255.0   
 mpls  
#  
bgp 100  
 peer 7.7.7.7 as-number 100   
 peer 7.7.7.7 connect-interface LoopBack0  
 peer 10.0.45.4 as-number 10   
 #  
 ipv4-family unicast  
  undo synchronization  
  undo peer 7.7.7.7 enable  
  peer 10.0.45.4 enable  
 #   
 ipv4-family vpnv4  
  undo policy vpn-target  
  peer 7.7.7.7 enable  
  peer 10.0.45.4 enable  
#
```

## **与Option-A的区别：
1.ASBR之间建立VPNv4邻居  
2.ASBR之间的接口不再绑定实例，  
通过在接口上使能mpls，  
以及在BGP下的ipv4-family vpnv4 undo policy vpn-target关闭掉 RT值的检测
 
==优点：减轻了ASBR的压力，不需要配置VPN-Instance来标记路由==
 
控制平面：
区别：在ASBR2将192.168.2.0/24的路由通告给ASBR2时  
继续沿用PE2给192.168.2.0/24的私网标签  
即：单向路径上：私网标签一直存在，且不会改变
 
转发（数据）平面：
区别：只是在ASBR之间是通过MPLS标签交换路由