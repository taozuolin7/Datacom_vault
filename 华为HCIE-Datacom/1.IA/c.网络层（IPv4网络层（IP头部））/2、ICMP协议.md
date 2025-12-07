```
Internet控制消息协议ICMP (Internet Control Message Protocol)是IP协议的辅助协议。￼协议号：1￼IGMP：2
```

```
ICMP协议用来在网络设备间传递各种差错和控制信息，  
对于收集各种网络信息、诊断和  
排除各种网络故障等方面起着至关重要的作用。
```

```
ICMP的报文：  
1.type：表明ICMP报文的作用  
2.code：表明ICMP报文的具体值  
3.Chksum：校验报文的完整
```

```
ICMP重定向功能：  
    
  PC访问服务器需要先去网关（AR2）  
  AR2收到数据后，判断收到数据的接口 还要将数据进行转发，以此触发重定向  
  重定向报文（ICMP报文 type 5 code 0 携带的内容   
              target address 服务器的地址  destination address AR1的地址）  
  PC收到重定向报文后，会将访问服务器的下一跳地址 更改为报文中携带的AR1的地址
```

```
[AR2]dis ip interface brief  查看接口配置的地址信息
```

![Exported image](Exported%20image%2020251206145550-0.png)

```
ICMP错误报告
```
 
```
tracert 检测网络路径的功能  
        主要通过IP报文中的TTL值来实现的  
   设备1发送TTL值为1的报文给下一台设备2  
   设备2收到报文后，因为TTL值为1丢弃该报文，并且向设备1回复一个ICMP的TTL超时报文  
                 ICMP的TTL超时报文 type=11 code=0  
                 报文的SIP为 设备2收到报文的接口地址  DIP为设备1的地址  
   设备1收到 TTL超时报文后，根据报文的SIP判断出下一台设备的地址  
   设备1会在创建一个 TTL=2的报文进行设备3的探索  
   重复上面的步骤
```

```
ICMP差错检测  
ICMP Echo消息常用于诊断源和目的地之间的网络连通性，同时还可以提供其他信息，如报文往返时间等。
```

![Exported image](Exported%20image%2020251206145552-1.png)