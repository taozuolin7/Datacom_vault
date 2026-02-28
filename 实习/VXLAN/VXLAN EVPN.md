# 为什么需要EVPN
EVPN（Ethernet Virtual Private Network）是一种用于二层网络互联的VPN技术。EVPN技术采用类似于BGP/MPLS IP VPN的机制，在BGP协议的基础上定义了一种新的NLRI（Network Layer Reachability Information，网络层可达信息）即EVPN NLRI，EVPN NLRI定义了几种新的BGP EVPN路由类型，用于处在二层网络的不同站点之间的MAC地址学习和发布。

==原有的VXLAN实现方案没有控制平面==，是通过数据平面的流量泛洪进行VTEP发现和主机信息（包括IP地址、MAC地址、VNI、网关VTEP IP地址）学习的，这种方式导致数据中心网络存在很多泛洪流量。为了解决这一问题，VXLAN引入了EVPN作为控制平面，通过在VTEP之间交换BGP EVPN路由实现VTEP的自动发现、主机信息相互通告等特性，从而避免了不必要的数据流量泛洪。

```R
ip vpn-instance a
 ipv4-family
  route-distinguisher 1:3
  vpn-target 1:3 export-extcommunity
  vpn-target 1:3 import-extcommunity
 vxlan vni 100 /三层VNI
#
bridge-domain 10
 vxlan vni 10 /二层VNI
 evpn
  route-distinguisher 1:1
  vpn-target 1:1 export-extcommunity
  vpn-target 1:1 import-extcommunity
#
bridge-domain 20
 vxlan vni 20 /二层VNI
 evpn
  route-distinguisher 1:2
  vpn-target 1:2 export-extcommunity
  vpn-target 1:2 import-extcommunity
```
#  BGP EVPN路由
## **Type2路由——MAC/IP路由**

![](assets/VXLAN%20EVPN/file-20260228092548511.png)

| 字段                          | 说明                                      |
| :-------------------------- | :-------------------------------------- |
| Route Distinguisher         | 该字段为EVPN实例下设置的RD（Route Distinguisher）值。 |
| Ethernet Segment Identifier | 该字段为当前设备与对端连接定义的唯一标识。                   |
| Ethernet Tag ID             | 该字段为当前设备上实际配置的VLAN ID。                  |
| MAC Address Length          | 该字段为此路由携带的主机MAC地址的长度。                   |
| MAC Address                 | 该字段为此路由携带的主机MAC地址。                      |
| IP Address Length           | 该字段为此路由携带的主机IP地址的掩码长度。                  |
| IP Address                  | 该字段为此路由携带的主机IP地址。                       |
| MPLS Label1                 | 该字段为此路由携带的二层VNI。                        |
| MPLS Label2                 | 该字段为此路由携带的三层VNI。                        |
|                             |                                         |

该类型路由在VXLAN控制平面中的作用包括：

- 主机MAC地址通告
    
    要实现同子网主机的二层互访，两端VTEP需要相互学习主机MAC。作为BGP EVPN对等体的VTEP之间通过交换MAC/IP路由，可以相互通告已经获取到的主机MAC。其中，MAC Address字段为主机MAC地址。
    
- 主机ARP通告
    
    MAC/IP路由可以同时携带主机MAC地址+主机IP地址，因此该路由可以用来在VTEP之间传递主机ARP表项，实现主机ARP通告。其中，MAC Address字段为主机MAC地址，IP Address字段为主机IP地址。此时的MAC/IP路由也称为ARP类型路由。主机ARP通告主要用于以下两种场景：
    
    1. ARP广播抑制。当三层网关学习到其子网下的主机ARP时，生成主机信息（包含主机IP地址、主机MAC地址、二层VNI、网关VTEP IP地址），然后通过传递ARP类型路由将主机信息同步到二层网关上。这样当二层网关再收到ARP请求时，先查找是否存在目的IP地址对应的主机信息，如果存在，则直接将ARP请求报文中的广播MAC地址替换为目的单播MAC地址，实现广播变单播，达到ARP广播抑制的目的。
        
    2. 分布式网关场景下的虚拟机迁移。当一台虚拟机从当前网关迁移到另一个网关下之后，新网关学习到该虚拟机的ARP（一般通过虚拟机发送免费ARP实现），并生成主机信息（包含主机IP地址、主机MAC地址、二层VNI、网关VTEP IP地址），然后通过传递ARP类型路由将主机信息发送给虚拟机的原网关。原网关收到后，感知到虚拟机的位置发生变化，触发ARP探测，当探测不到原位置的虚拟机时，撤销原位置虚拟机的ARP和主机路由。
        
- 主机IP路由通告
    
    在分布式网关场景中，要实现跨子网主机的三层互访，两端VTEP（作为三层网关）需要互相学习主机IP路由。作为BGP EVPN对等体的VTEP之间通过交换MAC/IP路由，可以相互通告已经获取到的主机IP路由。其中，IP Address字段为主机IP路由的目的地址，同时MPLS Label2字段必须携带三层VNI。此时的MAC/IP路由也称为IRB（Integrated Routing and Bridge）类型路由。