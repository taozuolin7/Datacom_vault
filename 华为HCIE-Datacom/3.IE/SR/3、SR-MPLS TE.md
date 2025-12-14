SR-MPLS TE
 1.手工建立隧道路径
   对于隧道封装的方式：
     1.松散模式
        以每台设备的node sid进行封装
        node sid是可以通过IGP进行最短路径计算的
     2.严格模式
        以每台设备的adj sid进行封装
        adj sid对于每台设备而言是固定的，所以是设备唯一转发的路径
     *松散模式和严格模可以结合使用
   SR是一种源路由的访问机制
   TE是由源设备封装路径上所有设备的段ID，中间设备只需要根据段ID进行转发即可

 2.动态建立隧道路径
   需要具备控制器软件来定义

AR1到AR2的隧道访问路径：
NE1-NE2-NE3-NE4

AR2到AR1的隧道访问路径：
NE4-NE2-NE3-NE1
![](assets/3、SR-MPLS%20TE/file-20251214165128773.png)

在每台设备上规划adj sid（手工配置静态的adj sid值）
防止设备故障重启后，动态的adj sid变换为其他值
```
#
isis
 segment-routing auto-adj-sid disable    关闭动态adj sid的标签分配
```

```
NE1：
#
segment-routing
 ipv4 adjacency local-ip-addr 10.1.12.1 remote-ip-addr 10.1.12.2 sid 322102
 ipv4 adjacency local-ip-addr 10.1.13.1 remote-ip-addr 10.1.13.3 sid 322103
```

```
NE2：
#
segment-routing
 ipv4 adjacency local-ip-addr 10.1.12.2 remote-ip-addr 10.1.12.1 sid 322201
 ipv4 adjacency local-ip-addr 10.1.23.2 remote-ip-addr 10.1.23.3 sid 322203
 ipv4 adjacency local-ip-addr 10.1.24.2 remote-ip-addr 10.1.24.4 sid 322204
```

```
NE3：
#
segment-routing
 ipv4 adjacency local-ip-addr 10.1.13.3 remote-ip-addr 10.1.13.1 sid 322301
 ipv4 adjacency local-ip-addr 10.1.23.3 remote-ip-addr 10.1.23.2 sid 322302
 ipv4 adjacency local-ip-addr 10.1.34.3 remote-ip-addr 10.1.34.4 sid 322304
```
```
NE4：
#
segment-routing
 ipv4 adjacency local-ip-addr 10.1.24.4 remote-ip-addr 10.1.24.2 sid 322402
 ipv4 adjacency local-ip-addr 10.1.34.4 remote-ip-addr 10.1.34.3 sid 322403
```

```
构建显示路径：
NE1：
#
explicit-path NE1_NE4
 next sid label 322102 type adjacency index 1     NE1到NE2的adj sid
 next sid label 322203 type adjacency index 2     NE2到NE3的adj sid
 next sid label 322304 type adjacency index 3     NE3到NE4的adj sid


NE4：
#
explicit-path NE4_NE1
 next sid label 322402 type adjacency index 1     NE4到NE2的adj sid
 next sid label 322203 type adjacency index 2     NE2到NE3的adj sid
 next sid label 322301 type adjacency index 3     NE3到NE1的adj sid
```

```
构建NE1和NE4之间的SR MPLS TE隧道：
NE1:
#
mpls lsr-id 1.1.1.1     SR隧道接口的相对应的SIP、DIP，不配置则隧道无法生效
mpls
 mpls te                开启MPLS TE功能
#
interface Tunnel1
 ip address unnumbered interface LoopBack0   借用环回口地址作为隧道接口地址
 tunnel-protocol mpls te                     隧道的协议为MPLS TE
 destination 4.4.4.4                         隧道的目标地址
 mpls te signal-protocol segment-routing     隧道使用的信令协议SR
 mpls te tunnel-id 1                         隧道的ID值（本设备唯一）
 mpls te path explicit-path NE1_NE4          隧道绑定的显示路径

NE4:
#
mpls lsr-id 4.4.4.4
mpls
 mpls te
#
interface Tunnel1
 ip address unnumbered interface LoopBack0
 tunnel-protocol mpls te
 destination 1.1.1.1
 mpls te signal-protocol segment-routing
 mpls te tunnel-id 1
 mpls te path explicit-path NE4_NE1
```

```
[NE1]display tunnel-info all 

Tunnel ID            Type                Destination                             Status

----------------------------------------------------------------------------------------

0x000000000300002001 sr-te               4.4.4.4                                 UP  
```


```
站点信息隔离绑定互通
NE1：
#
ip vpn-instance A
 ipv4-family
  route-distinguisher 100:1
  apply-label per-instance
  vpn-target 1:1 export-extcommunity
  vpn-target 1:1 import-extcommunity
#
ospf 1 router-id 10.1.1.1 vpn-instance A
 import-route bgp
 opaque-capability enable
 area 0.0.0.0
#
interface GigabitEthernet3/0/0
 undo shutdown
 ip binding vpn-instance A
 ip address 192.168.1.254 255.255.255.0
 ospf enable 1 area 0.0.0.0
#

bgp 100
 private-4-byte-as enable
 peer 4.4.4.4 as-number 100
 peer 4.4.4.4 connect-interface LoopBack0
 #
 ipv4-family unicast
  undo synchronization
  undo peer 4.4.4.4 enable
 #
 ipv4-family vpnv4
  policy vpn-target
  peer 4.4.4.4 enable
 #
 ipv4-family vpn-instance A
  network 100.1.1.1 255.255.255.255
```


```
站点信息隔离绑定互通
NE4：
#
ip vpn-instance A
 ipv4-family
  route-distinguisher 100:2
  apply-label per-instance
  vpn-target 1:1 export-extcommunity
  vpn-target 1:1 import-extcommunity
#
ospf 1 router-id 10.4.4.4 vpn-instance A
 import-route bgp
 opaque-capability enable
 area 0.0.0.0
#
interface GigabitEthernet3/0/0
 undo shutdown
 ip binding vpn-instance A
 ip address 192.168.2.254 255.255.255.0
 ospf enable 1 area 0.0.0.0
#
bgp 100
 private-4-byte-as enable
 peer 1.1.1.1 as-number 100
 peer 1.1.1.1 connect-interface LoopBack0
 #
 ipv4-family unicast
  undo synchronization
  undo peer 1.1.1.1 enable
 #
 ipv4-family vpnv4
  policy vpn-target
  peer 1.1.1.1 enable
 #
 ipv4-family vpn-instance A
  network 100.2.2.2 255.255.255.255
```

以此TE的隧道建立完毕
且BE的隧道也建立完毕

数据访问优选BE隧道转发
需要执行隧道的优选

```
NE1：
#
tunnel-policy test
 tunnel select-seq sr-te load-balance-number 1
#
ip vpn-instance A
  tnl-policy test
```

```
NE4：
#
tunnel-policy test
 tunnel select-seq sr-te load-balance-number 1
#
ip vpn-instance A
  tnl-policy test
```

```
NE1的接口 1/0/1 抓包
封装的标签为
322203
322304
48060


NE4的接口 1/0/1 抓包
322203
322301
48060
```
	 