```
为了在发生网络故障后，减小故障带来的影响，需要快速探测到故障并切换到备用路径，通过BFD实现  
BFD：（Bidirectional Forwarding Detection，双向转发检测）  
提供了一个通用的、标准化的、介质无关和协议无关的快速故障检测机制
```

```
BFD的会话建立：  
1.静态BFD会话  
   1.手工指定设备两端的标识ID值  
2.动态BFD会话  
   1.设备自动生成设备两端的标识ID值
```

```
静态BFD：  
[AR1]bfd # 使能BFD  
[AR1]bfd AR1_AR2 bind peer-ip 10.1.12.2 source-ip 10.1.12.1   
[AR1-bfd-session-ar1_ar2]discriminator local 111  
[AR1-bfd-session-ar1_ar2]discriminator remote 222  
[AR1-bfd-session-ar1_ar2]commit  #做完配置要激活
```
 
```
[AR1]ip route-static 2.2.2.2 32 10.1.12.2 track bfd-session AR1_AR2   
静态路由绑定BFD：  
  1.BFD状态为UP，则静态路由生效  
  2.BFD状态为down，静态路由失效
```

```
[AR2]bfd  
[AR2]bfd AR2_AR1 bind peer-ip 10.1.12.1 source-ip 10.1.12.2   
[AR2-bfd-session-ar2_ar1]discriminator local 222  
[AR2-bfd-session-ar2_ar1]discriminator remote 111  
[AR2-bfd-session-ar2_ar1]commit
```

```
动态BFD：  
[AR1]bfd #使能BFD  
[AR1]bfd AR1_AR2 bind peer-ip 10.1.12.2 source-ip 10.1.12.1 auto￼  
[AR2]bfd AR2_AR1 bind peer-ip 10.1.12.1 source-ip 10.1.12.2 auto 
```

```
BFD的检测机制：  
1.异步检测  
   两台支持BFD的设备，相互检测  
   对于检测的时间，本端的接收时间 乘以 对端的检测次数  
2.查询检测  
   单台支持BFD的设备，自检测  
   对于检测的时间，本端的接收时间 乘以 本端的检测次数
```

```
BFD的收敛时间设置：  
[AR1-bfd-session-ar1_ar2]min-tx-interval 500   发送BFD报文的间隔时间  
[AR1-bfd-session-ar1_ar2]min-rx-interval 300   接收BFD报文的间隔时间  
[AR1-bfd-session-ar1_ar2]detect-multiplier 3   BFD报文检测的次数
```

```
BFD的收敛时间设置：  
[AR2-bfd-session-ar1_ar2]min-tx-interval 600   发送BFD报文的间隔时间  
[AR2-bfd-session-ar1_ar2]min-rx-interval 400   接收BFD报文的间隔时间  
[AR2-bfd-session-ar1_ar2]detect-multiplier 4   BFD报文检测的次数
```

```
出口路由器的单臂回声BFD：￼当设备进行BFD配置，且一端无法支持BFD配置，可以使用单臂回声的功能来实现  
检测BFD的一端，发送BFD报文的SIP 和 DIP 是相同的，所以对端设备收到报文后 会根据DIP进行报文回复￼  
[AR1]bfd #使能BFD  
[AR1]bfd AR1_AR2 bind peer-ip 10.1.12.2 interface GigabitEthernet 0/0/0 one-arm-echo  
[AR1-bfd-session-ar1_ar2]discriminator local 1111     只需要配置本端标识  
[AR1-bfd-session-ar1_ar2]min-echo-rx-interval 300     只存在接收报文的时间间隔  
[AR1-bfd-session-ar1_ar2]detect-multiplier 4  
[AR1-bfd-session-ar1_ar2]commit 
```

```
BFD会话检测时长由：TX（Desired Min TX Interval 期望最小发送间隔）  
                                 RX（Required Min RX Interval 要求最小接收间隔）  
                                 DM（Detect Multi）  
BFD报文的实际发送时间间隔，实际接受时间间隔由BFD会话协商决定。  
本地BFD报文实际发送时间间隔＝MAX { 本地配置的发送时间间隔，对端配置的接收时间间隔 }  
本地BFD报文实际接收时间间隔＝MAX { 对端配置的发送时间间隔，本地配置的接收时间间隔 }  
本地BFD报文实际检测时间：  
异步模式：本地BFD报文实际检测时间＝本地BFD报文实际接收时间间隔×对端配置的BFD检测倍数  
查询模式：本地BFD报文实际检测时间 = 本地BFD报文实际接收时间间隔×本端配置的BFD检测倍数
```

![Exported image](Exported%20image%2020251206164808-0.png)