```
NAT（Network Address Translation，网络地址转换）
```

```
背景：  
网络用户多，IP地址不够用  
将IP地址进行分类（公网地址、私网地址）  
公网地址全球唯一，私网地址可以重复使用  
私网地址不能在公用网络中出现
```
 
```
私网地址如何访问公网地址？  
 通过NAT技术来实现  
 就是在网络通信时，将私网地址转换为公网地址
```

```
NAT分类：  
按照转换的内容区分：  
1.SNAT  
  1.静态NAT  
  2.动态NAT  
  3.NAPT  
  4.easy ip  
2.DNAT  
  1.nat server
```

```
NAT分类：  
按照转换的项区分：  
1.NAT  
  1.静态NAT  
  2.动态NAT  
2.PAT：端口地址转换  
  1.NAPT  
  2.easy ip  
  3.nat server
```

```
静态NAT：  
在出口设备的外网出接口配置：  
第一种配置：（之间在出接口上配置）  
[AR1]interface GigabitEthernet 0/0/1  
[AR1-GigabitEthernet0/0/1]nat static global 100.1.12.100 inside 192.168.1.253  
第二种配置：（在系统视图下配置后绑定出接口）  
[AR1]nat static global 100.1.12.100 inside 192.168.1.253  
[AR1]interface GigabitEthernet 0/0/1  
[AR1-GigabitEthernet0/0/1]nat static enable 
```
 
```
实现的是一对一的地址映射  
（每一个通信的私网地址，要存在对应一个公网地址上网）NAT的本质是实现内部网络的安全保护（隐藏真实网络的IP地址）  
NAT是不安全的（在映射过程中，公网设备可以直接访问私网映射后的地址）  
***映射后的公网地址，在公网中是要被其他设备路由可达的，否则无法正常通信
```
 
```
出口NAT设备，执行映射后，如何判断公网回复的报文 是自身要进行处理的？  
  出口设备会存在一条路由信息  
200.1.1.1/32  Unr(用户路由)  64  0   D  127.0.0.1  InLoopBack0  
  在执行了NAT配置后，会为转换的公网地址自动生成路由条目  
\<AR1\>dis nat static   
  Static Nat Information:  
  Interface  : GigabitEthernet0/0/1  
    Global IP/Port     : 200.1.1.1/----   
    Inside IP/Port     : 192.168.1.253/----  
    Protocol : ----       
    VPN instance-name  : ----                              
    Acl number         : ----  
    Netmask  : 255.255.255.255   
    Description : ----
```
 
```
  Total :    1  
\<AR1\>
```
 
```
动态NAT：  
可以实现地址多对一的映射  
多个私网地址 可以 用一个公网地址上网通信  
在同一时刻内只能实现 一对一的映射
```
 
```
需要存在 访问控制列表  和  公网地址池  
在出口将私网地址池和公网地址池 进行绑定
```
 
```
[AR1]acl 2000   
[AR1-acl-basic-2000]rule 5 permit source 192.168.1.0 0.0.0.255 
```
 
```
[AR1]nat address-group 1 200.1.1.1 200.1.1.10
```
 
```
[AR1]interface GigabitEthernet 0/0/1  
[AR1-GigabitEthernet0/0/1]nat outbound 2000 address-group 1 no-pat
```
 
```
NAPT：网络地址端口转换  
实现多对一映射（同一时刻内实现多个私网地址 转换为 一个公网地址）
```
 
```
是从动态NAT进行改进的，所以也存在 私网地址池 和 公网地址池
```
 
```
[AR1]acl 2000   
[AR1-acl-basic-2000]rule 5 permit source 192.168.1.0 0.0.0.255   
[AR1-acl-basic-2000]rule 10 permit source 192.168.2.0 0.0.0.255
```
 
```
[AR1]nat address-group 1 200.1.1.1 200.1.1.10
```
 
```
[AR1]interface GigabitEthernet 0/0/1  
[AR1-GigabitEthernet0/0/1]nat outbound 2000 address-group 1   
NAPT通过公网地址池的地址 实现 私网的转换  
一个公网地址可以实现6W+的私网地址转换  
2^16 = 65,536(减去一些特殊和著名的端口)=6w+
```
 
```
\<AR1\>dis nat session all   查看转换的地址信息  
  NAT Session Table Information:
```
 
```
     Protocol          : ICMP(1)  
     SrcAddr   Vpn     : 192.168.1.253                                    
     DestAddr  Vpn     : 3.3.3.3                                          
     Type Code IcmpId  : 0   8   7883   
     NAT-Info  
       New SrcAddr     : 200.1.1.1        
       New DestAddr    : ----  
       New IcmpId      : 10647
```
 
```
     Protocol          : ICMP(1)  
     SrcAddr   Vpn     : 192.168.2.253                                    
     DestAddr  Vpn     : 3.3.3.3                                          
     Type Code IcmpId  : 0   8   7888   
     NAT-Info  
       New SrcAddr     : 200.1.1.1        
       New DestAddr    : ----  
       New IcmpId      : 10658
```
 
```
easy ip：  
节省了公网地址池的公网地址  
将私网地址直接转换为 外网出接口的 公网IP
```
 
```
[AR1]acl 2000   
[AR1-acl-basic-2000]rule 5 permit source 192.168.1.0 0.0.0.255   
[AR1-acl-basic-2000]rule 10 permit source 192.168.2.0 0.0.0.255  
# 直接绑定送出接口  
[AR1]interface GigabitEthernet 0/0/1  
[AR1-GigabitEthernet0/0/1]nat outbound 2000
```
 
```
***一般出口路由器都是直接 缺省路由指向外网  
私网内的设备也需要存在缺省路由，用于访问外部路由
```
 
```
如果配置静态路由，则需要每台内部设备都指定缺省  
如果配置动态路由（OSPF），通过在出口设备下发缺省路由即可  
[AR2-ospf-1-area-0.0.0.0]default-route-advertise
```
 
```
NAT server：  
私网内部的服务器为公网用户提供服务  
（公网用户可以访问私网的服务应用）  
[AR1-GigabitEthernet0/0/1]nat server protocol tcp global 200.1.1.100 12345 inside 192.168.1.100 www  
# 将私网服务192.168.1.100 www 映射到 公网地址200.1.1.100 端口 12345 
```
 
```
[AR1-GigabitEthernet0/0/1]nat server protocol tcp global current-interface 12345 inside 192.168.1.100 www  
# 将私网服务192.168.1.100 www 映射到 接口的公网地址 端口 12345
```
 
```
[AR1-GigabitEthernet0/0/1]nat server protocol tcp global 200.1.1.100 ftp inside 192.168.1.100 ftp
```
 
![Exported image](Exported%20image%2020251206145655-0.png)