二层隧道：VxLAN、L2TP、PPTP、VPLS  
封装的是数据帧的信息  
三层隧道：GRE、MPLS VPN、IPsec  
封装的是IP包的数据  
应用层隧道：SSL VPN  
封装的是应用层的数据
 
NVO3：将IP网络作为底层网络，在之上构建隧道
 
Underlay network：底层网络（IGP相互通信的网络）  
Overlay network：虚拟网络（建立隧道网络）

VxLAN的基本概念：  
1.NVE：网络虚拟化边缘设备（实际上就是执行VxLAN报文封装与解封装的设备，类似于MPLS网络的PE设备）  
2.VTEP：VxLAN隧道端口（唯一标识一台NVE设备）（一台NVE设备必须要有一个或多个VTEP地址）  
（VTEP地址是全网唯一的）（两个VTEP地址可以唯一确定一条VxLAN隧道）  
3.VNI：虚拟网络标识（用来区分不同的广播域）  
类似于VLAN ID的用户标识，不同的VNI是隔离二层通信的  
vlan id是12 bit，VNI 是24 bit

VxLAN的虚拟化概念：  
VxLAN要经过两次虚拟化  
第一次：通过VTEP构建的VxLAN隧道将网络虚拟化为一台交换机  
第二次：通过VNI来为虚拟的交换机做VLAN的隔离  
VxLAN的网关概念：  
网关信息分为两种  
第一种：2层网关，站点PC接入，实现同子网互访  
第二种：3层网关，网络站点接入，实现跨子网互访或VxLAN和非VxLAN网络的互访  
VxLAN网关部署的类型：  
1.集中式网关（3层网关部署在同一台设备上）  
2.分布式网关（3层网关下沉到个leaf节点）  
BD域：  
作为NVE设备的本地概念，没有全局的意义  
只在本地区分不同的大二层网络  
BD域和VNI是一对一绑定的（BD的值是24bit）
 
VAP：虚拟网络接入点  
对于NVE设备连接中断的接入方式  
1.子接口接收数据（通过子接口的vid接收对应VLAN的数据，且子接口绑定BD域）  
2.VLAN接收数据（通过VLAN id接收对应的VLAN数据，且VLAN绑定BD域）
![](assets/5、静态VXLAN总结/file-20251212164642009.png)

```
CE1：  
#  
interface Nve1    设置设备NVE的接口  
 source 1.1.1.1   设置VTEP地址  
 vni 10 head-end peer-list 3.3.3.3    
 指定vxlan隧道的VNI值，并指定隧道的对端地址  
#  
bridge-domain 10  
 vxlan vni 10  
#  
interface GE1/0/0  
 undo shutdown  
#  
interface GE1/0/0.1 mode l2  
 encapsulation dot1q vid 10  
 bridge-domain 10  
#
```

```
CE3：  
#  
interface Nve1  
 source 3.3.3.3  
 vni 10 head-end peer-list 1.1.1.1  
#  
bridge-domain 10  
 vxlan vni 10  
#  
interface GE1/0/0  
 undo shutdown  
#  
interface GE1/0/0.1 mode l2  
 encapsulation dot1q vid 10  
 bridge-domain 10  
#
```