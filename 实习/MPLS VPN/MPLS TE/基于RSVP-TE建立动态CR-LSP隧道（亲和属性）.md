
##基于RSVP-TE的CR-LSP实验：路径选择
![](assets/基于RSVP-TE建立动态CR-LSP隧道（亲和属性）/file-20260205092550337.png)
**实验概述：**
1.底层为 ISIS level-2
2.blue 路径为非链路管理组中
3.转发路径最大带宽为：200MB，链路保护带宽为100MB
4.affinity 10001，mask 10101 得出 链路管理组为“10001，10101”

**场景复现：**
1.R7的GE0/0/1链路为100MB时的CR-LSP路径为：
```R
<Huawei>tracert lsp te tun 0/0/0
  LSP Trace Route FEC: TE TUNNEL IPV4 SESSION QUERY Tunnel0/0/0 , press CTRL_C to break.
  TTL   Replier            Time    Type      Downstream
  0                                Ingress   10.1.12.2/[1030 ]
  1     10.1.12.2          10 ms   Transit   10.1.26.6/[1028 ]
  2     10.1.26.6          30 ms   Transit   10.1.67.7/[1030 ]
  3     10.1.67.7          30 ms   Transit   10.1.78.8/[1029 ]
  4     10.1.78.8          30 ms   Transit   10.1.58.5/[3 ]
  5     5.5.5.5            40 ms   Egress
```

2.R7的GE0/0/1链路为50MB时的CR-LSP路径为：
```R
<Huawei>tracert lsp te tun 0/0/0
  LSP Trace Route FEC: TE TUNNEL IPV4 SESSION QUERY Tunnel0/0/0 , press CTRL_C to break.
  TTL   Replier            Time    Type      Downstream
  0                                Ingress   10.1.12.2/[1031 ]
  1     10.1.12.2          10 ms   Transit   10.1.26.6/[1029 ]
  2     10.1.26.6          30 ms   Transit   10.1.67.7/[1031 ]
  3     10.1.67.7          30 ms   Transit   10.1.47.4/[1029 ]
  4     10.1.47.4          30 ms   Transit   10.1.48.8/[1030 ]
  5     10.1.48.8          30 ms   Transit   10.1.58.5/[3 ]
  6     5.5.5.5            40 ms   Egress
```

**配置严格显示路径**
![](assets/基于RSVP-TE建立动态CR-LSP隧道（亲和属性）/file-20260205092550336.png)
1.路径必须经过AR4
```R
<Huawei>tracert lsp te Tunnel 0/0/0
  LSP Trace Route FEC: TE TUNNEL IPV4 SESSION QUERY Tunnel0/0/0 , press CTRL_C to break.
  TTL   Replier            Time    Type      Downstream
  0                                Ingress   10.1.12.2/[1033 ]
  1     10.1.12.2          20 ms   Transit   10.1.26.6/[1031 ]
  2     10.1.26.6          20 ms   Transit   10.1.67.7/[1033 ]
  3     10.1.67.7          30 ms   Transit   10.1.47.4/[1031 ]
  4     10.1.47.4          20 ms   Transit   10.1.48.8/[1032 ]
  5     10.1.48.8          30 ms   Transit   10.1.58.5/[3 ]
  6     5.5.5.5            50 ms   Egress
```

2.按照配置的严格显示路径转发
```R
explicit-path AR3
 next hop 10.1.12.2
 next hop 10.1.26.6
 next hop 10.1.67.7
 next hop 10.1.47.4
 next hop 10.1.34.3
 next hop 10.1.38.8
 next hop 10.1.58.8
 next hop 5.5.5.5
 
[Huawei]tracert lsp te Tunnel 0/0/0
  LSP Trace Route FEC: TE TUNNEL IPV4 SESSION QUERY Tunnel0/0/0 , press CTRL_C to break.
  TTL   Replier            Time    Type      Downstream
  0                                Ingress   10.1.12.2/[1041 ]
  1     10.1.12.2          20 ms   Transit   10.1.26.6/[1039 ]
  2     10.1.26.6          20 ms   Transit   10.1.67.7/[1041 ]
  3     10.1.67.7          30 ms   Transit   10.1.47.4/[1032 ]
  4     10.1.47.4          40 ms   Transit   10.1.34.3/[1024 ]
  5     10.1.34.3          40 ms   Transit   10.1.38.8/[1040 ]
  6     10.1.38.8          40 ms   Transit   10.1.58.5/[3 ]
  7     5.5.5.5            40 ms   Egress
```

3.按配置松散路径转发

```R
explicit-path ar4
 next hop 4.4.4.4 include loose
 
 
<Huawei>tracert lsp te Tunnel 0/0/0
  LSP Trace Route FEC: TE TUNNEL IPV4 SESSION QUERY Tunnel0/0/0 , press CTRL_C to break.
  TTL   Replier            Time    Type      Downstream
  0                                Ingress   10.1.12.2/[1044 ]
  1     10.1.12.2          20 ms   Transit   10.1.26.6/[1042 ]
  2     10.1.26.6          30 ms   Transit   10.1.67.7/[1042 ]
  3     10.1.67.7          30 ms   Transit   10.1.47.4/[1034 ]
  4     10.1.47.4          30 ms   Transit   10.1.48.8/[1043 ]
  5     10.1.48.8          40 ms   Transit   10.1.58.5/[3 ]
  6     5.5.5.5            40 ms   Egress
<Huawei>
<Huawei>reset mpls te tunnel-interface Tunnel 0/0/0
<Huawei>
Feb  4 2026 11:07:47-08:00 Huawei %%01IFNET/4/LINK_STATE(l)[14]:The line protocol IP on the interface Tunnel0/0/0 has entered the DOWN state.
<Huawei>
Feb  4 2026 11:07:47-08:00 Huawei %%01IFNET/4/LINK_STATE(l)[15]:The line protocol IP on the interface Tunnel0/0/0 has entered the UP state.
<Huawei>
<Huawei>tracert lsp te Tunnel 0/0/0
  LSP Trace Route FEC: TE TUNNEL IPV4 SESSION QUERY Tunnel0/0/0 , press CTRL_C to break.
  TTL   Replier            Time    Type      Downstream
  0                                Ingress   10.1.12.2/[1045 ]
  1     10.1.12.2          20 ms   Transit   10.1.26.6/[1043 ]
  2     10.1.26.6          20 ms   Transit   10.1.36.3/[1027 ]
  3     10.1.36.3          30 ms   Transit   10.1.34.4/[1035 ]
  4     10.1.34.4          30 ms   Transit   10.1.48.8/[1044 ]
  5     10.1.48.8          20 ms   Transit   10.1.58.5/[3 ]
  6     5.5.5.5            40 ms   Egress

```

**关键配置**
1.出接口的所有链路都需要配置链路带宽，不然一些路径走不通
```R
isis 1
 cost-style wide
 traffic-eng level-2
 
 
interface GigabitEthernet0/0/1
 ip address 10.1.16.1 255.255.255.0
 isis enable 1
 mpls
 mpls te
 mpls te link administrative group 11101
 mpls te bandwidth max-reservable-bandwidth 200000
 mpls te bandwidth bc0 100000
 mpls rsvp-te

explicit-path ar4
 next hop 4.4.4.4 include loose

interface Tunnel0/0/0
 ip address unnumbered interface LoopBack0
 tunnel-protocol mpls te
 destination 5.5.5.5
 mpls te tunnel-id 1
 mpls te record-route label
 mpls te bandwidth ct0 100000
 mpls te path explicit-path ar4
 mpls te affinity property 10001 mask 11011
 mpls te commit

```