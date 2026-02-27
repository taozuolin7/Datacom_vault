
# 1、why？
1.经过之前的学习，只是mpls te流量工程只是在 IP承载下的一些简单实验，比如“静态CR-LSP配置”，“静态双向共路实验”，“基于RSVP-TE的动态CR-LSP选路实验”都是基础的理论实验，那么MPLS TE 流量工程需要用在哪些位置，在什么地方部署。通过上面方式？
2.流量工程主要是部署在（已知的案例）：中国移动的IP承载网中，在核心区域：由9个城市的CR或者CR兼做BR组成的核心层，部署LDP Over RSVP（LDP Over TE），实质就是MPLS TE 流量工程。
![](assets/MPLS%20TE%20L3%20VPN(tnl-policy)实验/file-20260227170407261.png)

# 2、如何配置：
1.as内底层IGP可达，环回口可达
2.配置MPLS 进程，在IGP使能MPLS TE功能
3.配置隧道接口Tunnel
4.在隧道两端建立MP-IBGP邻居
5.在对应的vpn实例下进行引流

# 3、配置参考
```R
#
ip vpn-instance a
 ipv4-family
  route-distinguisher 100:1
  tnl-policy tunnel1
  vpn-target 1:1 export-extcommunity
  vpn-target 1:1 import-extcommunity
#
tunnel-policy tunnel1
 tunnel select-seq cr-lsp load-balance-number 1
#
interface Tunnel1
 description At least 10Mbit/s for tunnel
 ip address unnumbered interface LoopBack0
 tunnel-protocol mpls te
 destination 3.3.3.3
 mpls te record-route label
 mpls te bandwidth ct0 10000
 mpls te tunnel-id 100
#
bgp 100
 router-id 1.1.1.1
 peer 3.3.3.3 as-number 100
 peer 3.3.3.3 connect-interface LoopBack0
 peer 192.168.10.2 as-number 65535
 #
 ipv4-family vpnv4
  policy vpn-target
  peer 3.3.3.3 enable
 #
 ipv4-family vpn-instance a
  peer 192.168.10.2 as-number 65535
  peer 192.168.10.2 substitute-as
```