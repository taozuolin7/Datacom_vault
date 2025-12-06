**IPv4到IPv6的过渡：**  
1.共存技术（IPv4和IPv6同时存在）  
*在ipv4环境下实现ipv6地址的相互通信  
1.双栈  
2.隧道：  
1.手工隧道  
gre  
6over4  
2.自动隧道  
6to4  
ISTAP
 
2.互通技术（将ipv4和ipv6相互转换）  
NAT64

![Exported image](Exported%20image%2020251206150223-0.png)

R1

```
￼#
interface Tunnel0/0/1
 ipv6 enable 
 ipv6 address FE80::1 link-local
 tunnel-protocol gre
 source 1.1.1.1
 destination 3.3.3.3
#
ipv6 route-static 2033:: 64 Tunnel 0/0/1
```

R3

```
￼#
interface Tunnel0/0/1
 ipv6 enable 
 ipv6 address FE80::3 link-local
 tunnel-protocol gre
 source LoopBack0
 destination 1.1.1.1
#
ipv6 route-static 2011:: 64 Tunnel 0/0/1 
```

1.GRE隧道