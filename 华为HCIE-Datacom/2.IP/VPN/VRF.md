# VRF
**VRF : virtual routing forwarding 虚拟路由转发**

广域链接设备角色：  
	1.PE设备：运营商边界设备
	2.CE设备：用户端边界设备
	3.P设备：运营商骨干设备

**一台PE设备可以链接多台CE设备**

CE设备是站点的出口设备==（站点：园区网络的概称）==

**对于不同站点内的网络，可以使用相同的私网地址**

**对于私网地址，公网设备如何处理？**
	1.P设备处理：不需要处理私网信息  
	2.PE设备处理：根据隧道进行数据的再封装，或接收数据解封装得到私网数据

**对于不同站点相同私网地址，PE设备如何识别？  **
- 通过VRF技术来实现（VRF：虚拟路由转发）  
- VRF将一台物理设备，逻辑上变为多台设备

**VRF的基本原理**：  
	1.通过VPN-instance来实现虚拟路由器的建立  
		[AR1]ip vpn-instance A  
		[AR1-vpn-instance-A]ipv4-family   
	2.对于物理设备而言路由表的名称默认为public  
		创建的逻辑虚拟路由器路由表的名称为vpn-instance的名称  
		对于不同VPN实例的路由表之间，路由是相互隔离的 
	3.将链接站点的接口绑定VPN实例（即将接口划分给站点）  
		interface GigabitEthernet0/0/0  
		ip binding vpn-instance A  
	4.对于VPN实例是一个本地有效的概念（即CE设备没有任何感知）
	5.默认设备查询ping时是通过public路由表找寻的路由  
		**如果需要在VPN实例下查找路由，则需要指定vpn实例的ping**
	6.VPN实例隔离的路由，可以是相同网段的，也可以是不同网段的  
		如果需要不同VPN实例的路由相互通信，则需要不同VPN实例的路由是不同网段的  
	==7.在模拟器上只能通过物理接口来访问，而不能通过环回口+来访问


![](assets/VRF/file-20251207204731175.png)```
```Java
[AR2]ip vpn-instance  A   
[AR2-vpn-instance-A]ipv4-family  
[AR2]ip vpn-instance  B   
[AR2-vpn-instance-B]ipv4-family   
[AR2]interface GigabitEthernet 0/0/0  
[AR2-GigabitEthernet0/0/0]ip binding vpn-instance A   
[AR2-GigabitEthernet0/0/0]ip ad 10.1.12.2 24   
[AR2]interface GigabitEthernet 0/0/1   
[AR2-GigabitEthernet0/0/1]ip binding vpn-instance B   
[AR2-GigabitEthernet0/0/1]ip ad 10.1.23.2 24
[AR2]ip route-static vpn-instance A 3.3.3.3 32 vpn-instance B 10.1.23.3
[AR2]ip route-static vpn-instance B 1.1.1.1 32 vpn-instance A 10.1.12.1 
[AR2]ip route-static vpn-instance A 10.1.23.0 255.255.255.0 vpn-instance B 10.1.23.3  
[AR2]ip route-static vpn-instance B 10.1.12.0 255.255.255.0 vpn-instance A 10.1.12.1
[AR1] ip route-static 0.0.0.0 0 10.0.12.2
[AR2] ip route-static 0.0.0.0 0 10.0.23.2
```
通过物理接口：进行连通性测试
![](assets/VRF/file-20251207204731166.png)
**与普通数据封装没区边**



