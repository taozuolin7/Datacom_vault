# OSPF高级特性
**OSPF的1类、2类LSA既包含了拓扑信息 也 包含了路由信息**
***如果路由发生变化，也会导致拓扑的重新计算
 
**对于拓扑计算使用SPF算法**
==第一次计算拓扑是，使用SPF进行全量的计算,后续的部分拓扑变化，使用i-SPF计算==

## **OSPF相关特性：**  
**1.PRC：部分路由计算**
	对于添加的路由信息做单独的计算（1类、2类、3类、5类、7类）  
**2.智能定时器：**
- 1.控制LSA的发送、接收  
```D
[AR1-ospf-1]lsa-originate-interval intelligent-timer 6000 600 1200  
                                                   最大时间 初始时间 间隔时间  
[AR1-ospf-1]lsa-arrival-interval intelligent-timer 1200 400 600  
```
- 2.控制LSA的路由计算  
```D
[AR1-ospf-1]spf-schedule-interval intelligent-timer 11000 600 1200  
```
设备因为链路或接口等原因发生了震荡（会导致LSA计算的路由时有时无）  
每一次计算路由都会计算时间的叠加  
第一次计算的路由时间为 初始时间 0.6s  
第二次计算的路由时间为 （ 间隔时间*2（n-2） n≥2 ） 1.2s  
第三次计算的路由时间为 （ 间隔时间*2（n-2） n≥2 ） 2.4s  
第四次计算的路由时间为 （ 间隔时间*2（n-2） n≥2 ） 4.8s  
第五次计算的路由时间为 （ 间隔时间*2（n-2） n≥2 ） 9.6s   
.。。。。。  
==如果连续三次计算的路由时间都大于 设置的最大时间，则时间清空 从第一次路由计算的时间重新叠加==

**快速重路由:提高网络故障时的恢复速度**
```D
[AR1]ospf  
[AR1-ospf-1]frr  
[AR1-ospf-1-frr]loop-free-alternate 启用无环路备用路径
```
**[AR1]dis ospf routing 4.4.4.4**
```R
OSPF Process 1 with Router ID 10.1.1.1
Destination : 4.4.4.4/32  
AdverRouter : 10.4.4.4 Area : 0.0.0.0  
Cost : 15 Type : Stub  
NextHop : 10.1.13.3 Interface : GigabitEthernet0/0/1  
Priority : Medium Age : 00h01m18s  
Backup Nexthop : 10.1.12.2 Backup Interface: GigabitEthernet0/0/0  
Backup Type : LFA LINK
```
==通过预先计算并维护一条无环路的备用路径，当主路径失效时立即切换，避免路由环路和流量中断==
 
## 以下三点常考：

#### OSPF下发缺省路由：
```D
[AR1-ospf-1]default-route-advertise 路由表要存在不同类型的缺省路由，ospf才会下发缺省的LSA  
[AR1-ospf-1]default-route-advertise always 路由表不管是否存在不同类型的缺省路由，ospf都会下发缺省的LSA
```
 
对于下发了缺省LSA的ospf设备，不会计算从其他设备学习到的缺省LSA  
```D
[AR1-ospf-1]default-route-advertise permit-calculate-other
```

#### LSDB规模
```D
[AR4-ospf-1]lsdb-overflow-limit \<1-1000000\> 设置超限的LSDB数据库规模  
接收5LSA的数量  
```
**如果超限，则删除自身产生的所有外部LSA（保留缺省LSA和学习到其他设备的外部LSA）**

#### **OSPF与IGP同步：**
```D
[AR4-ospf-1]stub-router on-startup \<5-65535\>
当设备重启后，建立OSPF邻居通告的LSA开销值为65535  
```
**根据设置的时间，倒计时为0后恢复为正常的开销**
