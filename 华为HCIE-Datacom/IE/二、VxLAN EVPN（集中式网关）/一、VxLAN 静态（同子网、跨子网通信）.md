![Exported image](Exported%20image%2020251206152111-0.png)

```
spine：  
#  
bridge-domain 10  
 vxlan vni 10  
#  
bridge-domain 20  
 vxlan vni 20  
#  
interface Nve1  
 source 2.2.2.2  
 vni 10 head-end peer-list 1.1.1.1  
 vni 20 head-end peer-list 3.3.3.3  
#  
interface Vbdif10  
 ip address 192.168.1.254 255.255.255.0  
#  
interface Vbdif20  
 ip address 192.168.2.254 255.255.255.0  
#
```

```
CE1#  
interface Nve1      创建nve接口  
 source 1.1.1.1     设置vtep地址  
 vni 10 head-end peer-list 3.3.3.3    
 构建vxlan隧道，并指定隧道的目标地址  
#  
bridge-domain 100   创建BD域  
 vxlan vni 10       绑定VNI  
#  
interface GE1/0/0.1 mode l2  
 encapsulation dot1q vid 10  
 bridge-domain 100  接口绑定BD域  
#
```

```
CE2￼#  
interface Nve1  
 source 3.3.3.3  
 vni 10 head-end peer-list 1.1.1.1  
#  
bridge-domain 10  
 vxlan vni 10  
#  
interface GE1/0/0.1 mode l2  
 encapsulation dot1q vid 10  
 bridge-domain 10  
#
```

跨子网通信要通过三层网关实现  
将三层网关部署在spine设备上

模拟器中无法实现 同子网和跨子网同时通信

NVE设备对于站点内数据的处理：  
*NVE设备通过子接口接收站点数据的三种接收模式  
[CE3-GE1/0/0.1]encapsulation ?
 
dot1q 只接收站点内带有vlan标签数据 ，根据接口的vid解封装vlan tag值（剥离vlan tag）  
对于收到VxLAN报文的处理：  
1.如果内层不携带vlan tag标签，将接口的vid添加为vlan tag后转发  
2.如果内层携带vlan tag标签，在原有的vlan tag上添加接口配置的vid值，在进行转发（模拟器实验）
 
untag 只接收 站点内不携带vlan tag的数据，不对原始报文进行处理（不添加vlan tag）  
对于收到VxLAN报文的处理：  
1.如果内层不携带vlan tag值，则直接转发  
2.如果内层携带vlan tag值，则接口不做报文信息的处理，直接丢弃（模拟器实验）
 
default 接收 站点内的数据（不管是否携带vlan tag），不对原始的报文做处理（不添加、不替换、不剥离vlan tag）  
对于收到VxLAN报文的处理：  
不对原始的报文做处理（不添加、不替换、不剥离vlan tag）

交换机对于BUM报文的处理：  
B 广播帧  
U 未知单播帧  
M 组播帧  
都是泛洪处理的
 
VxLAN网络中 NVE设备对于BUM报文的处理  
根据头端复制列表 进行封装转发  
头端复制列表：  
即interface nve 接口下配置的 vni xx head-end peer-list x.x.x.x  
NVE设备将收到的BUM报文 根据头端复制列表中的地址 转换为单播报文进行转发