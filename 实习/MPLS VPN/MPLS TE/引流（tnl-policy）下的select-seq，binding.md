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
# 二、如何引流
1.通过 tunnel-policy 进行引流
```R
tunnel-policy passTh-AR4 
 tunnel select-seq cr-lsp load-balance-number 1
```
2.tunnel-policy 中存在两种引流方式
- 隧道类型优先级（select-seq）：改变隧道类型的优先选择顺序或者进行隧道的负载分担。
- 隧道绑定（binding）：将隧道与某个目的地址绑定，使到该目的地址的VPN流量承载在绑定的隧道上，保证VPN的QoS。


