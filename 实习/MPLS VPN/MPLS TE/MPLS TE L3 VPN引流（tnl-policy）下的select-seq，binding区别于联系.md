# 一、为什么需要引流
因为从开始到学习引流之前，我们一直通过tracert lsp te tunnel0/0/1 进行隧道的测试，该测试只是保证该流量工程的隧道没问题，**但是没引流之前 业务流量是无法通过 流量工程隧道进行转发的**，所以我们需要通过 “引流”将 业务流量 引入到 隧道当中。
```R
[Huawei]tracert lsp te Tunnel 0/0/1
  LSP Trace Route FEC: TE TUNNEL IPV4 SESSION QUERY Tunnel0/0/1 , press CTRL_C t
o break.
  TTL   Replier            Time    Type      Downstream 
  0                                Ingress   10.1.12.2/[1024 ]
  1     10.1.12.2          20 ms   Transit   10.1.24.4/[1024 ]
  2     10.1.24.4          30 ms   Transit   10.1.34.3/[1024 ]
  3     10.1.34.3          30 ms   Transit   10.1.35.5/[3 ]
  4     5.5.5.5            40 ms   Egress 
```
# 二、何为引流
## 1.通过 tunnel-policy 进行引流
```R
tunnel-policy passTh-AR4 
 tunnel select-seq cr-lsp load-balance-number 1
```

## 2.tunnel-policy 中隧道策略有以下两种方式
- 隧道类型优先级（select-seq）：改变隧道类型的优先选择顺序或者进行隧道的负载分担。
- 隧道绑定（binding）：将隧道与某个目的地址绑定，使到该目的地址的VPN流量承载在绑定的隧道上，保证VPN的QoS。
- 未配置以上两种方式之一时，隧道策略默认的行为是按照LDP LSP > BGP LSP > 静态LSP > SR LSP（SR-MPLS BE）的顺序选择隧道；配置了**tunnel policy binding-default down-switch enable**后，若没有UP的以上几类隧道，可以继续按照RSVP-TE > SR-MPLS TE > GRE > BGP local ifnet的顺序选择隧道。

## 3.解析两种隧道策略的方式：
### **1.select-seq**
配置并应用按优先级顺序选择方式的隧道策略后，VPN选择隧道时将优先选择**tunnel select-seq**命令中排列在前的隧道。例如，隧道策略下配置了**tunnel select-seq cr-lsp lsp load-balance-number 2**后，VPN应用隧道策略后，将优先选择CR-LSP类型的隧道。
- 如果当前系统中有2条或者2条以上可用的CR-LSP隧道时，则系统随机选取其中的2条。
- 如果当前系统中CR-LSP类型的隧道少于2条，则不足的隧道从LSP类型隧道中选取。
- 系统中正在使用的隧道条数由2条以上降到2条以下，则触发隧道策略重新选择隧道，不足的隧道从LSP类型隧道中选取。
当在命令中使用**lsp**参数时，则有多种LSP类型的隧道可以作为候选隧道：BGP LSP、静态LSP、LDP LSP、SR LSP （SR-MPLS BE）。这几种LSP类型隧道的优先级顺序为LDP LSP > BGP LSP > 静态LSP > SR LSP（SR-MPLS BE）。例如：隧道策略下配置了tunnel select-seq lsp load-balance-number 2后：
```R
[HUAWEI-tunnel-policy-1]tunnel select-seq ?
  bgp     BGP tunnel
  cr-lsp  CR-LSP tunnel
  gre     GRE tunnel
  ldp     LDP tunnel
  lsp     LSP tunnel
  sr-lsp  Segment routing LSP
  sr-te   Segment routing TE
  te      RSVP-TE or Static TE tunnel
```
这三种LSP类型隧道的优先级顺序为LDP LSP>BGP LSP>静态LSP。例如，隧道策略下配置了**tunnel select-seq lsp cr-lsp load-balance-number 3**后：
- 如果当前系统中有3条或者3条以上可用的LDP LSP隧道时，则系统随机选取其中的3条。
- 如果当前系统中LDP LSP隧道少于3条，则不足的隧道从BGP LSP类型隧道中选取。
- 如果当前系统中LDP LSP和BGP LSP隧道的总数少于3条，则不足的隧道从静态LSP类型隧道中选取。
### 2.binding
**前置条件**
在配置所要绑定的MPLS TE隧道之前，首先应在该隧道接口视图下配置**mpls te reserved-for-binding**命令。
```R
interface Tunnel0/0/2
 ip address unnumbered interface LoopBack0
 tunnel-protocol mpls te
 destination 1.1.1.1
 mpls te tunnel-id 200
 mpls te reserved-for-binding  ## 必要的配置
 mpls te commit
```
其中隧道绑定方式只对RSVP-TE隧道、SR-MPLS TE隧道、自动隧道或SR-MPLS TE Policy隧道组有效。例如对于TE隧道，能精确的指定走哪条TE隧道，因此便于部署QoS。有些VPN业务对QoS要求较高，一般需要为其迭代专用的TE隧道，这时可以使用该命令对VPN应用隧道绑定策略。绑定多条隧道时可以形成负载分担。
- 若目的地址无法匹配到binding的地址，默认配置下流量会中断。若配置了tunnel policy binding-default down-switch enable，则会遍历各协议是否有可用的隧道。
- 若目的地址匹配到binding的地址，且所有绑定的隧道都不可用，若**tunnel binding**命令行中未使能down-switch参数，则流量会中断；若使能了down-switch参数，则会遍历各协议是否有可用的隧道。
- 遍历各协议隧道的优先级是LDP LSP > BGP LSP > 静态LSP > SR LSP（SR-MPLS BE） > RSVP-TE > SR-MPLS TE > GRE > BGP local ifnet。
### 注意事项：
命令**tunnel binding**与命令**tunnel select-seq**互斥，不支持同时配置。
```R
tunnel-policy passTh-AR4 
 tunnel select-seq cr-lsp load-balance-number 1
```
```R
tunnel-policy no-passTh-AR4 
 tunnel binding destination 1.1.1.1 te Tunnel0/0/2
```
# 三、如何引流
虽然配置了tunnel-policy 如果没有在对应的vpn实例下进行配置，那么流量是不会走 mple te流量工程隧道的。
```R
ip vpn-instance a
 ipv4-family
  route-distinguisher 100:1
  tnl-policy passTh-AR4
```


# 四、配置参考
```R
tunnel-policy passTh-AR4 
 tunnel select-seq cr-lsp load-balance-number 1
#
ip vpn-instance a
 ipv4-family
  tnl-policy passTh-AR4
#
interface Tunnel0/0/1
 ip address unnumbered interface LoopBack0
 tunnel-protocol mpls te
 destination 5.5.5.5
 mpls te tunnel-id 100
 mpls te record-route label
 mpls te path explicit-path passTh-AR4
 mpls te commit
```

```R
tunnel-policy no-passTh-AR4 
 tunnel binding destination 1.1.1.1 te Tunnel0/0/2 
#
ip vpn-instance a
 ipv4-family
  route-distinguisher 100:1
  tnl-policy no-passTh-AR4
#
interface Tunnel0/0/2
 ip address unnumbered interface LoopBack0
 tunnel-protocol mpls te
 destination 1.1.1.1
 mpls te tunnel-id 200
 mpls te reserved-for-binding
 mpls te commit
```