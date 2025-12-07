ipv6地址前缀的获取方式：  
1.无状态自动配置（SLAAC）
 
2.有状态自动配置（DHCPv6）  
1.DHCPv6有状态  
ipv6地址 和 其他参数（DNS）  
2.DHCPv6无状态  
其他参数（DNS）

DHCPv6有状态自动配置

`AR1`：  

```
#
ipv6
#
interface GigabitEthernet0/0/0
 ipv6 enable 
 ipv6 address 2012::1/64 
 ipv6 address FE80::1 link-local
#
dhcp enable 
#
dhcpv6 pool test
 address prefix 2012::/64
 excluded-address 2012::1
 dns-server 2088::8
#
interface GigabitEthernet0/0/0
 dhcpv6 server test
```

![Exported image](Exported%20image%2020251206150105-0.png)

`AR2`：  

```
#
ipv6
#
dhcp enable
#
interface GigabitEthernet0/0/0
 ipv6 enable 
 ipv6 address FE80::2 link-local
 ipv6 address auto dhcp
#
```

```
[AR2]display dhcpv6 client interface GigabitEthernet 0/0/0 
GigabitEthernet0/0/0 is in stateful DHCPv6 client mode.
State is BOUND.
Preferred server DUID   : 0003000100E0FC9E4A1D
  Reachable via address : FE80::1
IA NA IA ID 0x00000031 T1 43200 T2 69120
  Obtained     : 2025-06-14 15:16:35
  Renews       : 2025-06-15 03:16:35
  Rebinds      : 2025-06-15 10:28:35
  Address      : 2012::3
    Lifetime valid 172800 seconds, preferred 86400 seconds
    Expires at 2025-06-16 15:16:35(172752 seconds left)
DNS server     : 2088::8
```

SLAAC+DHCPv6无状态自动配置

![Exported image](Exported%20image%2020251206150107-1.png)  

```
AR1：  
#  
ipv6  
#  
dhcp enable   
#  
dhcpv6 pool test  
 address prefix 2012::/64  
 excluded-address 2012::1  
 dns-server 2088::8  
#  
interface GigabitEthernet0/0/0  
 ipv6 enable   
 ipv6 address 2012::1/64   
 ipv6 address FE80::1 link-local  
 ipv6 nd ra prefix 2012::/64 10000 5000   
 ipv6 nd ra preference high   
 undo ipv6 nd ra halt  
 dhcpv6 server test  
#  
为了使终端通过slaac得到前缀  
通过dhcpv6得到其他参数  
所以将dhcpv6分配的地址前缀 和 接口的网络前缀配置为相同网段
```

```
AR2：  
#  
ipv6  
#  
dhcp enable  
#  
interface GigabitEthernet0/0/0  
 ipv6 enable   
 ipv6 address FE80::2 link-local  
 ipv6 address auto global  
 ipv6 address auto dhcp  
#
```

```
[AR2]display ipv6 interface GigabitEthernet 0/0/0   
GigabitEthernet0/0/0 current state : UP   
IPv6 protocol current state : UP  
IPv6 is enabled, link-local address is FE80::2  
  Global unicast address(es):  
    2012::2E0:FCFF:FEC8:418B,  
    subnet is 2012::/64 [SLAAC 1970-01-01 00:17:35 10000S]￼￼  
[AR2]display dhcpv6 client interface GigabitEthernet 0/0/0   
GigabitEthernet0/0/0 is in stateful DHCPv6 client mode.  
State is BOUND.  
Preferred server DUID   : 0003000100E0FC9E4A1D  
  Reachable via address : FE80::1  
IA NA IA ID 0x00000031 T1 43200 T2 69120  
  Obtained     : 2025-06-14 15:16:35  
  Renews       : 2025-06-15 03:16:35  
  Rebinds      : 2025-06-15 10:28:35  
  Address      : 2012::3  
    Lifetime valid 172800 seconds, preferred 86400 seconds  
    Expires at 2025-06-16 15:16:35(172752 seconds left)  
DNS server     : 2088::8
```

**DHCPv6****——****pd****（前缀委托）**

![Exported image](Exported%20image%2020251206150112-2.png)

```
AR3：  
#  
ipv6  
#  
dhcp enable  
#  
dhcpv6 pool test  
 prefix-delegation 2075::/64 64  
#  
interface GigabitEthernet0/0/0  
 ipv6 enable   
 ipv6 address 2034::3/64   
 dhcpv6 server test
```

```
AR4：  
#  
ipv6   
#  
dhcp enable  
#  
interface GigabitEthernet0/0/0  
 ipv6 enable   
 ipv6 address 2034::4/64   
 ipv6 address FE80::4 link-local  
 dhcpv6 client pd test  
#  
interface GigabitEthernet0/0/1  
 ipv6 enable   
 ipv6 address test ::4/64  
 ipv6 address FE80::4 link-local  
 undo ipv6 nd ra halt  
#￼  
GigabitEthernet0/0/1    up   up           
[IPv6 Address] 2075::4
```

```
AR5：  
#  
ipv6  
#  
interface GigabitEthernet0/0/0  
 ipv6 enable   
 ipv6 address auto global  
#￼  
GigabitEthernet0/0/0         up          up           
[IPv6 Address] 2075::2E0:FCFF:FED2:263B
```

**DHCPv6****——****relay**

![Exported image](Exported%20image%2020251206150113-3.png)

```
AR6：  
#  
ipv6  
#  
dhcp enable  
#  
dhcpv6 pool test  
 address prefix 2078::/64  
 excluded-address 2078::7  
#  
interface GigabitEthernet0/0/0  
 ipv6 enable   
 ipv6 address 2067::6/64   
 dhcpv6 server test  
#  
ipv6 route-static 2078:: 64 2067::7 
```

```
AR7：  
#  
ipv6  
#  
dhcp enable  
#  
interface GigabitEthernet0/0/0  
 ipv6 enable   
 ipv6 address 2067::7/64   
#  
interface GigabitEthernet0/0/1  
 ipv6 enable   
 ipv6 address 2078::7/64   
 dhcpv6 relay destination 2067::6  
#
```

```
AR8：  
#  
ipv6  
#  
dhcp enable  
#  
interface GigabitEthernet0/0/0  
 ipv6 enable   
 ipv6 address FE80::8 link-local  
 ipv6 address auto dhcp  
#
```

**DHCPv6中继：终端已经确定所在的网络前缀，服务器只是进行地址和网络参数的分配**  
**DHCPv6-PD：终端不存在网络前缀，服务器需要规划网络前缀为终端分配**

**四、典型使用场景**

1. **DHCPv6 Relay 的核心场景**
    - **跨网段通信**：当 DHCPv6 服务器位于核心层，客户端分布在接入层不同子网时，通过 Relay 转发报文。
    - **多服务器负载分担**：配置多个 DHCPv6 服务器组，Relay 根据负载均衡策略转发请求。
    - **高可用性部署**：在 VRRP/VRRP6 主备场景中配置 Relay 双机热备，确保中继功能不中断。
2. **Prefix Delegation 的核心场景**
    - **ISP 到用户分配**：ISP 通过 PD 为家庭或企业路由器分配 / 48 前缀，用户自行划分子网。
    - **企业分支动态地址管理**：总部通过 PD 为分支机构路由器分配前缀，分支机构无需手动配置地址空间。
    - **云网络弹性扩展**：云平台通过 PD 为租户动态分配前缀，支持多租户地址空间隔离。

**五、当前网络组网的适配性分析**

1. **Relay 的优势场景**
    - **集中管理需求**：适用于 SDN（软件定义网络）或云管理平台，通过 Relay 实现 DHCPv6 服务器的集中部署和策略统一。
    - **复杂网络架构**：在多层交换网络、混合 VLAN/IPoE 环境中，Relay 能简化跨网段通信配置。
    - **高可靠性要求**：结合 VRRP/VRRP6 或 E-Trunk 主备技术，Relay 可实现双机热备，提升网络健壮性。
2. **PD 的优势场景**
    - **地址空间动态扩展**：适合 IPv6 地址资源紧张的场景，通过 PD 按需分配前缀，避免地址浪费。
    - **层次化网络设计**：支持 ISP - 运营商 - 企业多级地址分配，符合 RFC8415 等标准的层次化架构要求。
    - **自动配置需求**：与 SLAAC（无状态地址自动配置）结合，实现终端设备的零配置接入。
3. **协同部署的最佳实践**
    - **ISP 场景**：ISP 通过 PD 为用户分配前缀，用户侧路由器通过 Relay 转发终端设备的 DHCPv6 请求，实现端到端自动化配置。
    - **大型企业园区**：核心层部署 DHCPv6 服务器，通过 Relay 为各楼层接入层提供服务，同时利用 PD 为分支机构动态分配地址空间。
    - **云数据中心**：租户网络通过 Relay 访问集中式 DHCPv6 服务器，同时通过 PD 获取独立的 IPv6 前缀，满足多租户隔离需求。

![Exported image](Exported%20image%2020251206150115-4.png)