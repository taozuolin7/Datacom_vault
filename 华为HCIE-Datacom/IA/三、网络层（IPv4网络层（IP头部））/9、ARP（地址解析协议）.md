```
应用层将数据称为Data
```
 
```
传输层将数据封装为Segment（段）
```
 
```
网络层将数据封装为Packet（包）
```
 
```
数据链路层将数据封装为Frame（帧）
```
 
```
物理层将数据变为Bitstream（比特流）
```
 
```
网络层会存在逻辑地址  
（可以跟随着设备不同位置而变化的）  
数据链路层会存在物理地址  
（与设备一同存在，默认不会改变）
```

```
逻辑地址  -\>  IP地址 -\>   管理员分配
```
 
```
物理地址  -\>  MAC地址 -\>  固定存在的 
```

![Exported image](Exported%20image%2020251206145642-0.png)

```
PC1访问PC2   （PC\>ping 192.168.1.2）
```
 
```
数据链路层（SMAC 11-11  DMAC ？？？）  
网络层（SIP 1.1  DIP 1.2）  
传输层（TCP/UDP）  
data
```

```
ARP：地址解析协议(D:destination目的)  
  根据DIP 进行 DMAC的获取  
Address Resolution Protocol
```

```
解析步骤：  
1.PC1检查自身的ARP表项是否存在去往目标的条目（查表）  
   1.检查存在，则将条目对应的MAC信息进行封装，转发数据  
   2.检查不存在，则需要进行ARP 解析  
       1.PC1发送arp request报文携带需要解析的内容  
    
2.PC2收到arp request报文  
   1.需要将报文对应的信息记录，将IP和MAC对应加入自身的arp表项  
   2.判断request报文内携带的DIP是否是自身的IP地址  
       1.如果不是自身的IP，则报文不继续处理  
       2.如果是自身的IP，需要回复arp reply报文
```
 
```
3.PC1收到arp reply报文  
   1.需要将报文对应的信息记录，将IP和MAC对应加入自身的arp表项
```

```
arp request：  
SMAC 11-11   DMAC FF-FF
```
 
```
arp request报文携带的内容：  
SMAC:11-11  
SIP :1.1  
DMAC:00-00  
DIP :1.2
```

```
arp reply：  
SMAC 22-22  DMAC 11-11
```
 
```
arp reply报文携带的内容：  
SMAC:22-22  
SIP :1.2  
DMAC:11-11  
DIP :1.1
```

|
|
```
默认ARP表项老化时间为1200s
```
```
20分钟
```

```
免费ARP：  
  检测地址冲突的  
  当设备配置IP地址后，为了防止地址冲突，需要发送免费ARP  
  免费ARP报文：  
    SMAC 设备1的mac  DMAC FFFF  
    携带的内容  
    SMAC 设备1的mac  
    SIP  设备1的IP  
    DMAC 00-00  
    DIP  设备1的IP
```
 
```
（冲突）  
如果网络中存在IP地址相同的设备2  
设备2回复reply报文，告知对方要进行地址替换  
设备1也回复reply报文，告知对方要进行地址替换  
只有管理员手工替换某一台设备的IP才可以解决问题
```

![Exported image](Exported%20image%2020251206145644-1.png)

```
1.什么是VRP系统？  
  VRP是华为公司数据通信产品的通用操作系统平台，作为华为公司从低端到核心的全系列路由器、以太网交换机、业务网关等产品的软件核心引擎。  
VRP提供以下功能：  
    实现统一的用户界面和管理界面，实现控制平面功能，并定义转发平面接口规范实现各产品转发平面与VRP控制平面之间的交互  
屏蔽各产品链路层对于网络层的差异
```
 
![Exported image](Exported%20image%2020251206145647-2.png) ![Exported image](Exported%20image%2020251206145648-3.png)