```
网络终端设备网络通信需要的参数：  
IP地址、网络掩码、网关、DNS
```
 
```
网络参数如何获得？  
1.管理员手工配置  
    1.配置繁琐，容易出错  
    2.故障点不易排查  
    3.地址资源浪费（无法重复利用）  
    4.设备替换网段需要大规模改变
```
 
```
2.动态获取（DHCP：动态主机配置协议）  
    1.指定服务器为终端自动分配地址  
    2.地址资源可以回收利用
```

```
DHCP是C/S的架构： client/server: 是一种静默的服务  
（服务器不会主动发送信息，只等待客户端发送信息 进行回复）
```
 
```
DHCP是为了给终端分配地址，所以终端是没有地址需要进行地址获取的
```

```
DHCP的原理：  
1.客户端 向 服务器 发送广播的 discover报文  
  SIP 0.0.0.0  DIP 255.255.255.255  
  向服务器申请网络参数  
  发送广播discover的目的是为了找寻网段内所有的DHCP服务器
```
 
```
2.服务器 向 客户端 回复单播的 offer 报文  
  SIP 服务器的地址  DIP 就是为客户端分配的地址  
  给客户端回复网络参数  
  只要是收到了discover报文的服务器，都会向客户端回复offer报文  
  客户端对于第一个接收的offer报文进行使用
```
 
```
3.客户端 向 服务器 发送广播的 request报文  
  SIP 0.0.0.0  DIP 255.255.255.255  
  向服务器确认收到的网络参数  
  1.向 使用offer的服务器 进行确认使用网络参数  
  2.向 为使用offer的服务器 进行通告，让其回收分配的地址
```
 
```
4.服务器 向 客户端 回复单播的 ACK报文  
  SIP 服务器的地址  DIP 就是为客户端分配的地址  
  给客户端确认网络参数可以使用
```

```
服务器回复的报文 是 单播还是广播 是由客户端决定的  
客户端发送的报文中，存在一个 Bit（置1：广播  置0：单播）
```

```
DHCP的配置方式：  
1.接口配置  
  就是将设备的三层接口作为DHCP server  
  [AR1]dhcp enable  开启DHCP服务功能  
  [AR1]interface GigabitEthernet 0/0/0   
  [AR1-GigabitEt0/0/0]ip address 192.168.1.254 24  
  [AR1-GigabitEe0/0/0]dhcp select interface   
特点：  
 1.终端的网段是和接口服务器所在的网段相同  
 2.终端的IP地址就是接口服务器网段内地址从大到小依次分配的  
 3.终端的网关就是接口服务器的IP地址
```
 
```
相关配置：  
   [AR1-GigabitEt0/0/0]dhcp server excluded-ip-address 192.168.1.101 192.168.1.251  
在地址池中排除一组地址（排除的地址不会进行地址分配）
```
 
```
   [AR1-GigabitEt0/0/0]dhcp server static-bind ip-address 192.168.1.4 mac-address 5489-9844-4444  
为绑定的MAC地址进行单独的地址分配（被绑定的地址已经被排除在地址池外，不会进行动态分配了）
```
 
```
  [AR1-GigabitEt0/0/0]dhcp server dns-list 8.8.8.8 114.114.114.114 配置DNS列表
```
 
```
2.全局配置（中继模式）  
在全网互通的前提下：  
[DHCP server]dhcp enable  
[DHCP server]ip pool A   
[DHCP server-ip-pool-A]gateway-list 192.168.1.254   
[DHCP server-ip-pool-A]network 192.168.1.0 mask 24   
[DHCP server-ip-pool-A]dns-list 10.1.1.200   
[DHCP server-GigabitEthernet0/0/0]dhcp select global    # 在接口进行DHCP报文的处理
```
 
```
中继：  
S2：  
[S2]dhcp enable  
[S2]interface vlanif 10   
[S2-Vlanif10]dhcp select relay  
# 启用中继  
[S2-Vlanif10]dhcp relay server-ip 10.1.1.100  
# 指明服务器的ip
```
 
```
DHCP服务器和 终端不在一个网段内，如何实现地址分配？  
  终端发送的广播报文 无法到达服务器  
  需要通过中继设备帮助客户端转发  
  一般终端的网关会作为中继设备
```
 
```
中继如何转发客户端的广播报文？  
  客户端到中继之间的广播报文，  
  在中继和服务器之间变为单播报文
```

![Exported image](Exported%20image%2020251206145747-0.png)  
![Exported image](Exported%20image%2020251206145749-1.png)

```
PC\>ipconfig    查看终端设备的网络参数  
PC\>ipconfig /release 撤销网络参数  
PC\>ipconfig /renew   重新获取网络参数
```

![Exported image](Exported%20image%2020251206145754-2.png)