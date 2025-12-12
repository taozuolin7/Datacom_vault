**1.****如果不使用BGP-EVPN**  
问题1：N个节点需要创建N*(N-1)/2条隧道，配置工作量大。

![Exported image](Exported%20image%2020251206152103-0.png)  

问题2：通过flood and learn机制学习MAC地址，泛洪流量大。

![Exported image](Exported%20image%2020251206152105-1.png)  

**2.****使用BGP EVPN作为控制面协议**  
设备使能BGP EVPN，建立BGP EVPN对等体关系。  
设备之间通过BGP EVPN路由通告，完成VXLAN控制面的相关工作。  
VXLAN隧道通过BGP EVPN自动建立，转发表项通过BGP EVPN动态刷新。

![Exported image](Exported%20image%2020251206152107-2.png)

**1.****静态****VXLAN****的致命缺点**￼对于静态VXLAN，该方案没有控制平面，是通过数据平面的流量泛洪进行VTEP发现和主机信息  
（包括IP地址、MAC地址、VNI、网关VTEP IP地址）学习的，这种方式导致VXLAN网络存在很多泛洪流量。  
为了解决这一问题，VXLAN引入了BGP EVPN作为控制平面，通过在VTEP之间交换BGP EVPN路由实现VTEP的自动发现、主机信息相互通告等，从而避免了不必要的数据流量泛洪。  
**2.****静态方式配置****VXLAN****的问题**  
N台设备建立VXLAN隧道，手工配置方式最高达到 N*(N-1)/2 次头端列表配置。  
静态VXLAN隧道只有数据转发平面。  
只能通过ARP广播的方式学习远端MAC地址。