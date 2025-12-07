手工隧道需要手工指定隧道的目标地址  
（使用静态的隧道指定数据的封装，隧道的目标只有一个）
 
如果需要访问的站点存在多个，则一个隧道无法实现多个站点的访问
 
静态隧道需要配置多个隧道接口来实现多个站点之间的访问￼￼自动隧道的本质就是通过已规划的地址信息，让隧道自动学习到要封装的DIP

![Exported image](Exported%20image%2020251206150226-0.png)

```
R1￼#
interface Tunnel0/0/1
 ipv6 enable 
 ipv6 address FE80::1 link-local
 tunnel-protocol ipv6-ipv4
 source 1.1.1.1
 destination 3.3.3.3
#
ipv6 route-static 2033:: 64 Tunnel 0/0/1
```

```
R3￼#
interface Tunnel0/0/1
 ipv6 enable 
 ipv6 address FE80::3 link-local
 tunnel-protocol ipv6-ipv4
 source LoopBack0      #
```

多个源，不再是单一的源地址  

```
 destination 1.1.1.1
#
ipv6 route-static 2011:: 64 Tunnel 0/0/1 
```