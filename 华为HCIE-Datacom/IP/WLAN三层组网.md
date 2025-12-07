# WLAN三层组网
**二层无线组网 AP和AC在同一个网段内 AP和AC的capwap隧道构建？** 
   AP发送广播报文找AC  
   AC发送单播报文找AP

**三层无线组网 AP和AC不在同一个网段 AP和AC的capwap隧道构建？**
   AC发送单播报文找AP  
**AP如何找AC？**
   AP也通过单播找AC

LSW1作为AP的网关，通过vlan 50通信LSW1连接AC通过VLAN 100三层通信
![400](assets/WLAN三层组网/file-20251207195401479.png)

```R
[AC]vlan 100  
[AC]vlan 10   
[AC]int gi 0/0/1  
[AC-GigabitEthernet0/0/1]p l t  
[AC-GigabitEthernet0/0/1]p t a v a   
[AC-GigabitEthernet0/0/1]q  
[AC]int vlanif 100   
[AC-Vlanif100]ip ad 10.1.1.1 24 
```

```R
[S1]vlan 50   
[S1]int vlanif 50    
[S1-Vlanif50]ip ad 10.1.50.254 24   
[S1]int gi 0/0/2   
[S1-GigabitEthernet0/0/2]p l t  
[S1-GigabitEthernet0/0/2]p t a v 50   
[S1]int gi 0/0/3   
[S1-GigabitEthernet0/0/3]p l t  
[S1-GigabitEthernet0/0/3]p t a v 50 ￼  
[S3]vlan 50   
[S3-vlan50]q  
[S3]int gi 0/0/1   
[S3-GigabitEthernet0/0/1]p l t  
[S3-GigabitEthernet0/0/1]p t a v 50   
[S3-GigabitEthernet0/0/1]q  
[S3]int gi 0/0/2   
[S3-GigabitEthernet0/0/2]p l t  
[S3-GigabitEthernet0/0/2]p t p v 50  
[S3-GigabitEthernet0/0/2]p t a v 50 ￼￼
```

**AC和AP之间构建capwap隧道**
```
[AC]ip route-static 10.1.50.0 24 10.1.1.254 
[AC]capwap source interface Vlanif 100   
[AC]wlan  
[AC-wlan-view]ap-group name 1  
[AC-wlan-view]ap-id 1 ap-mac 00e0-fcc5-36e0   
[AC-wlan-ap-1]ap-name AP1  
[AC-wlan-ap-1]ap-group 1￼￼


```
LSW1作为终端的网关，并为终端分配IP地址  
[S1]vlan 10   
[S1-vlan10]q  
[S1]int vlanif 10   
[S1-Vlanif10]ip ad 192.168.10.254 24   
[S1-Vlanif10]dhcp select interface 
```

```
[S1]vlan 100  
[S1]int gi 0/0/1   
[AC-GigabitEthernet0/0/1]p l t  
[AC-GigabitEthernet0/0/1]p t a v a   
[S1-GigabitEthernet0/0/1]q  
[S1]int vlanif 100   
[S1-Vlanif100]ip ad 10.1.1.254 24 
```

```
[S2]vlan 50   
[S2-vlan50]q  
[S2]int gi 0/0/1   
[S2-GigabitEthernet0/0/1]p l t  
[S2-GigabitEthernet0/0/1]p t a v 50   
[S2]int gi 0/0/2   
[S2-GigabitEthernet0/0/2]p l t  
[S2-GigabitEthernet0/0/2]p t p v 50   
[S2-GigabitEthernet0/0/2]p t a v 50 
```

```
LSW1作为DHCP的服务器为AP分配地址  
[S1]dhcp enable   
[S1]ip pool AP  
[S1-ip-pool-ap]gateway-list 10.1.50.254   
[S1-ip-pool-ap]network 10.1.50.0 mask 24   
[S1-ip-pool-ap]option 43 sub-option 3 ascii 10.1.1.1  
[S1]int vlanif 50   
[S1-Vlanif50]dhcp select global 
```
 
```
AP自动获取地址后，会主动发送单播的capwap报文找AC建立capwap隧道
```

```
AC配置下发的给AP的业务参数  
[AC]wlan  
[AC-wlan-view]ssid-profile name 1  
[AC-wlan-ssid-prof-1]ssid A
```
 
```
[AC-wlan-view]security-profile name 1	  
[AC-wlan-sec-prof-1]security open 
```
 
```
[AC-wlan-view]vap-profile name test  
[AC-wlan-vap-prof-test]ssid-profile 1  
[AC-wlan-vap-prof-test]security-profile 1   
[AC-wlan-vap-prof-test]service-vlan vlan-id 10   
[AC-wlan-vap-prof-test]forward-mode tunnel 
```
 
```
[AC-wlan-view]ap-group name 1  
[AC-wlan-ap-group-1]vap-profile test wlan 1 radio 0 
```

|
|
```
维度
```
```
二层组网
```
```
三层组网
```
```
业务 VLAN 规划
```
```
所有 AP 归属同一 VLAN
```
```
AP 可归属不同 VLAN
```
```
支持的漫游类型
```
```
仅二层漫游
```
```
二层漫游和三层漫游
```
```
IP 地址变化
```
```
不变
```
```
可能变化（三层漫游时）
```
```
典型场景
```
```
小型园区、单一办公区域
```
```
大型园区、跨楼栋 / 区域部署
```