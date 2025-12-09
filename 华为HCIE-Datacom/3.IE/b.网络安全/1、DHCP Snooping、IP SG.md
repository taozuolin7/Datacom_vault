# DHCP Snooping、IP SG

**情景：当网络中存在攻击者，冒冲DHCP服务器下发地址（DHCP仿冒者攻击）**
==要求：如果终端使用攻击者提供的地址，则不能正常上网==
该问题可以通过dhcp snooping检测，来解决
```D
[S1]dhcp enable  
[S1]dhcp snooping enable ipv4
[S1]interface GigabitEthernet 0/0/3
[S1-GigabitEthernet0/0/3]dhcp snooping enable 接口开启监听功能
```  
连接上行的所有接口都需要开启该功能  
将接口检测为信任或非信任接口  
默认使能了dhcp snooping的接口都是==非信任接口==
	1.信任接口：收到终端的DHCP报文可以向信任接口转发  
	2.非信任接口：丢弃非信任接口收到的DHCP报文  
==[S1-GigabitEthernet0/0/3]dhcp snooping trusted 将接口设置为信任接口  ==
**[S1]dis dhcp snooping user-bind all**
```D
DHCP Dynamic Bind-table:
Flags:O - outer vlan ,I - inner vlan ,P - map vlan
IP Address       MAC Address     VSI/VLAN(O/I/P) Interface      Lease          
-------------------------------------------------------------------------------
192.168.1.253    5489-9809-17c6  1   /--  /--    GE0/0/1        2025.06.08-14:00
-------------------------------------------------------------------------------
print count:           1          total count:           1
```

### **DHCP服务是一种静默服务，容易遭受攻击**  
**1.DHCP server饿死攻击**
- 1.网络中存在大量的攻击者，伪装成客户端，发送报文请求IP地址  
```D
[S1-GigabitEthernet0/0/1]dhcp snooping max-user-number \<1-1024\>  
设置接口的最大地址分配数量  
```
- 2.网络中存在一个攻击者，不断改变dhcp报文中的chaddr字段，模拟大量终端申请IP地址  
```d
[S1-GigabitEthernet0/0/1]dhcp snooping check dhcp-chaddr enable  
检查DHCP报文中的chaddr字段和报文中的SMAC地址是否相同（不相同则丢弃报文）  
```
**2.DHCP 报文攻击**
- 1.攻击者不断发送大量的DHCP报文，消耗服务器的资源  
```D
[S1-GigabitEthernet0/0/1]dhcp snooping check dhcp-rate enable 设置DHCP报文发送速率的限值  
[S1-GigabitEthernet0/0/1]dhcp snooping check dhcp-request enable alarm dhcp-reply threshold 20  
```
- 2.使能并设置全局DHCP报文的阈值  
```D
[S1]dhcp snooping alarm dhcp-rate enable threshold 20
```
 
### **IPSG：IP源防护** **IP source guard**  
攻击者使用正常PC的IP地址进行上网  
IPSG功能会检查报文发送的SIP 和 SMAC对应的关系  
必须符合记录的dhcp snooping表项 才能实现转发
```D
interface GigabitEthernet0/0/5  
ip source check user-bind enable 接口使能IPSG  
ip source check user-bind alarm enable 使能检查告警  
ip source check user-bind alarm threshold 20 告警的阈值  
ip source check user-bind check-item ip-address mac-address vlan 设置检查的项
[S1]dis dhcp snooping user-bind all 检查基于dhcp snooping表项的MAC、IP对应关系（动态的表项）  
[S1]user-bind static ip-address 192.168.1.100 mac-address 5489-9800-0100 interface GigabitEthernet 0/0/5 vlan 1
```

**[S1]dis dhcp snooping user-bind all**
```D
DHCP Dynamic Bind-table:  
Flags:O - outer vlan ,I - inner vlan ,P - map vlan  
IP Address MAC Address VSI/VLAN(O/I/P) Interface Lease
--------------------------------------------------------------------------------  
192.168.1.253 5489-9809-17c6 1 /-- /-- GE0/0/1 2025.06.08-14:00  
--------------------------------------------------------------------------------  
print count: 1 total count: 1
```

**[S1]dis dhcp static user-bind all 查看静态IPSG的表项**
```D
DHCP static Bind-table:  
Flags:O - outer vlan ,I - inner vlan ,P - map vlan  
IP Address MAC Address VSI/VLAN(O/I/P) Interface
--------------------------------------------------------------------------------  
192.168.1.100 5489-9800-0100 1 /-- /-- GE0/0/5  
--------------------------------------------------------------------------------  
print count: 1 total count: 1
```
