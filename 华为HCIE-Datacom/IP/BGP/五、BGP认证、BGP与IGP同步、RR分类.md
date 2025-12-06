```
BGP的认证：  
1.BGP报文不存在认证字段，通过tcp报文的option字段携带认证信息  
  [AR5-bgp]peer 10.1.45.4 password simple Huawei@123 
```
 
```
2.GTSM：通用TTL安全保护机制  
  通过设置TTL的值来限制BGP邻居的建立  
  [AR1-bgp]peer 3.3.3.3 valid-ttl-hops \<1-255\>  IBGP邻居的TTL值设置  
  [AR4-bgp]peer 10.1.45.5 ebgp-max-hop \<1-255\>  EBGP邻居的TTL值设置（超出了TTL值正常计算）  
  [AR4-bgp]peer 10.1.45.5 valid-ttl-hops \<1-255\> EBGP邻居的TTL值设置
```
 ![Exported image](Exported%20image%2020251206164638-0.png)  

```
BGP与IGP的同步：  
问题由来：设备的IGP邻居 一定是 优于 BGP邻居先建立的；设备一定是先学习了IGP的路由 才会 学习BGP的路由  
当主用路径的设备重启后，因为IGP建立优于BGP，所以可能会导致BGP的路由黑洞（AR2重开没建立IBGP邻居，会导致找不到路）
```
 
```
OSFP：  
通过设置[AR2-ospf-1]stub-router on-startup 100       OSPF协议将设备配置为末节路由器（即将IGP的路径开销设置为最大值65535）
```
 
```
ISIS：  
通过设置[AR2-isis-1]set-overload on-startup wait-for-bgp 100   ISIS协议将设备作为末节设备设置最大值的开销
```