SR-MPLS TE隧道：
1.单隧道建立
   本端PE到对端PE指定一条显示路径
2.主备隧道建立
   本端PE到对端PE指定多条显示路径
   在一个隧道口下绑定了多条显示路径
   显示路径绑定是选择主备
   对于主路径，可以通过BFD进行检测
interface Tunnel1
 tunnel-protocol mpls te
 mpls te path explicit-path 1
 mpls te path explicit-path 2 secondary

3.负载分担隧道建立
   1.如果只存在一条隧道，是将底层多条显示路径的adj sid 设置为相同的值

   2.建立多条隧道，隧道的目标指定相同的值DIP，并指定对等的显示路径

```
#
interface Tunnel1
 tunnel-protocol mpls te
 destination 3.3.3.3
 mpls te tunnel-id 1
 mpls te path explicit-path 1
#
interface Tunnel2
 tunnel-protocol mpls te
 destination 3.3.3.3
 mpls te tunnel-id 2
 mpls te path explicit-path 2
```

SR MPLS TE可以执行显示路径的转发
但是无法对于业务进行区分

SR MPLS TE policy
可以在SR MPLS TE的基础上进行业务的区分
*优化了隧道接口的使用

## 配置思路：
1.底层IGP SR 使能
2.配置静态的adj sid
![](assets/SR-MPLS%20TE隧道/file-20251214165437392.png)

业务1  100.2.2.2/32
业务2  200.2.2.2/32

```
NE1：
#
segment-routing
 ipv4 adjacency local-ip-addr 10.1.12.1 remote-ip-addr 10.1.12.2 sid 322102
 ipv4 adjacency local-ip-addr 10.1.14.1 remote-ip-addr 10.1.14.4 sid 322104
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
NE4：
#
segment-routing
 ipv4 adjacency local-ip-addr 10.1.14.4 remote-ip-addr 10.1.14.1 sid 322401
 ipv4 adjacency local-ip-addr 10.1.24.4 remote-ip-addr 10.1.24.2 sid 322402
 ipv4 adjacency local-ip-addr 10.1.34.4 remote-ip-addr 10.1.34.3 sid 322403
```

```
NE3：
#
segment-routing
 ipv4 adjacency local-ip-addr 10.1.23.3 remote-ip-addr 10.1.23.2 sid 322302
 ipv4 adjacency local-ip-addr 10.1.34.3 remote-ip-addr 10.1.34.4 sid 322304
```

### 在NE1和NE3上执行显示路径的配置
```
NE1：
#
segment-routing
 segment-list NE1_NE3_backup
  index 1 sid label 32210
  index 2 sid label 322403
 segment-list NE1_NE3_master
  index 1 sid label 322102
  index 2 sid label 322204
  index 3 sid label 322403
```

```
NE3：
#
segment-routing
 segment-list NE3_NE1_backup
  index 1 sid label 322304
  index 2 sid label 322401
 segment-list NE3_NE1_master
  index 1 sid label 322302
  index 2 sid label 322204
  index 3 sid label 322401
```
在NE1和NE3上配置SR te policy
```
NE1：
#
segment-routing
 sr-te policy test endpoint 3.3.3.3 color 100
  candidate-path preference 200
   segment-list NE1_NE3_master
  candidate-path preference 100
   segment-list NE1_NE3_backup
```

```
NE3：
#
segment-routing
 sr-te policy test endpoint 1.1.1.1 color 300
  candidate-path preference 200
   segment-list NE3_NE1_master
  candidate-path preference 100
   segment-list NE3_NE1_backup
```

建立站点信息的配置
VPN实例 RD、RT
VPNv4邻居建立
私网路由传递

执行隧道优选的配置，在NE1和NE3优选隧道te policy

```
NE1：
#
tunnel-policy test
 tunnel select-seq sr-te-policy load-balance-number 1 unmix
#
ip vpn-instance A
  tnl-policy test
```

```
NE3：
#
tunnel-policy test
 tunnel select-seq sr-te-policy load-balance-number 1 unmix
#
ip vpn-instance A
  tnl-policy test
```
### 对于业务2  NE1的配置：
```

#
segment-routing
 segment-list NE1_NE3_YW2_backup
  index 1 sid label 322102
  index 2 sid label 322203
 segment-list NE1_NE3_YE2_master
  index 1 sid label 322104
  index 2 sid label 322402
  index 3 sid label 322203
 sr-te policy admin endpoint 3.3.3.3 color 120
  candidate-path preference 200
   segment-list NE1_NE3_YE2_master
  candidate-path preference 100
   segment-list NE1_NE3_YW2_backup
#
ip ip-prefix 2 index 10 permit 200.2.2.2 32
#
route-policy color permit node 20
 if-match ip-prefix 2
 apply extcommunity color 0:120
#
```

```
[NE1]dis ip rou v A 
Destination/Mask    Proto   Pre  Cost        Flags NextHop           Interface
      100.1.1.1/32  OSPF    10   1             D   192.168.1.1       GigabitEthernet3/0/0
      100.2.2.2/32  IBGP    255  2             RD  3.3.3.3            test
      200.2.2.2/32  IBGP    255  2             RD  3.3.3.3            admin
```

NE1和NE3要相互执行路由策略来为路由标记颜色
根据颜色匹配进行te policy隧道执行数据转发
```
NE1：
#
ip ip-prefix 1 index 10 permit 100.2.2.2 32
#
route-policy color permit node 10
 if-match ip-prefix 1
 apply extcommunity color 0:100
#
bgp 100
 #
 ipv4-family vpnv4
  peer 3.3.3.3 route-policy color import
```

```
NE3：
#
ip ip-prefix 1 index 10 permit 100.1.1.1 32
#
route-policy color permit node 10
 if-match ip-prefix 1
 apply extcommunity color 0:300
#
bgp 100
 #
 ipv4-family vpnv4
  peer 1.1.1.1 route-policy color import
```

为了实现主备链路的切换
需要通过备份机制（SBFD、hot stanby）来实现
```
NE1：
#
mpls lsr-id 1.1.1.1
#
bfd
#
sbfd
 reflector discriminator 1.1.1.1
#
segment-routing
 sr-te-policy backup hot-standby enable
 sr-te-policy seamless-bfd enable
```

```
NE3：
#
mpls lsr-id 3.3.3.3
#
bfd
#
sbfd
 reflector discriminator 3.3.3.3
#
segment-routing
 sr-te-policy backup hot-standby enable
 sr-te-policy seamless-bfd enable
```

对于跨域的方案：
可以通过部署bing sid 来实现一个AS内所有明细sid的概括

SR-MPLS te 跨域 
 在隧道接口下执行
 [NE1-Tunnel1]mpls te binding-sid label 

SR-MPLS te policy 跨域
 在sr-te policy 下执行
 [NE1-segment-routing-te-policy-test]binding-sid 