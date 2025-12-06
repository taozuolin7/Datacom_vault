```
OSPF：Open Shortest Path First，开放式最短路径优先(89)
```
 
```
OSPF支持VLSM（Variable Length Subnet Mask，可变长子网掩码），支持手工路由汇总
```
 
```
链路状态路由协议  
1.先建立一个邻居关系  
2.LSA通告：链路状态通告 （链路状态 连接的接口 设备的状态 设备的类型）  
3.LSDB存储：链路状态数据库（收到其他设备的LSA，存储到自身的LSDB中）  
4.SPF计算：最短路径优先算法（将LSDB中的LSA进行计算 得到最短路径树 和 路由信息）  
5.将计算得到的路由条目加入到路由表中
```
 
```
OSPF的基本概念：  
1.进程ID：  默认进程ID为1  
   [AR1]ospf \<1-65535\>  
   每一个进程都存在设备的一种工作运行状态
```
 
```
2.router id：设备ID  
   如果不配置，则设备不存在RID，使用0.0.0.0 表示RID不存在  
   RID有两种配置方式：  
      1.手工配置：[AR1]ospf 1 router-id 10.1.1.1  
      2.自动配置：OSPF自动将全局的RID作为OSPF的RID  
          *全局的RID如何配置：  
              1.手工配置：[Huawei]router id 123.1.1.1  
              2.自动配置：将第一个配置的接口IP地址作为全局的RID  
   RID的显示内容：  
      是类似于IP地址的点分十进制表示  
      RID不是一个IP地址，而是一段32位无符号的整数  
   RID的特点：  
      1.设备的RID是唯一的，不能冲突  
      2.RID配置完成后，不会自动更改  
      3.如果需要改变OSPF的RID，则需要进行OSPF的进程重启  
3.area id：区域ID  
   限制了一定范围内设备的数量  
   设备数量过多，会导致LSA信息多 LSDB变大，SPF计算困难  
   区域ID的配置内容：  
     手工配置：[Huawei-ospf-1]area   
     1.数字表示：\<0-4294967295\>  
     2.点分十进制表示：0.0.0.0 - 255.255.255.255  
     如果是数字表示，则会转换为点分十进制  
4.度量值：cost开销  
   运行了OSPF的链路会存在链路的开销  
   链路的开销 通过 计算得到（参考带宽（100） ?  接口的带宽）  
   如果计算的值 小于1 ，则按照1计算  
   cost配置方式：  
    [AR1-ospf-1]bandwidth-reference \<1-2147483648\> 修改参考带宽  
    [AR1-GigabitEthernet0/0/0]ospf cost \<1-65535\>  直接修改接口的开销值
```
 
```
   路由cost计算：  
    1.路由接收的入方向，开销叠加  
    2.数据发送的出方向，开销叠加  
    *路由学习 和 数据转发 的方向是相反的
```

```
[Huawei]display router id   查看全局的RID   
RouterID:0.0.0.0
```
 
```
\<Huawei\>reset ospf 1 process  
```
 
```
[Huawei]display ospf peer brief  查看OSPF的邻居关系
```
 
```
[AR1]display ospf interface GigabitEthernet 0/0/0   
     查看运行OSPF的接口状态信息
```
 
```
[AR1]ospf 1 router-id 10.1.1.1    配置OSPF进程和设备ID  
[AR1-ospf-1]area 0                指定设备所在的区域ID  
[AR1-ospf-1-area-0.0.0.0]network 10.1.12.1 0.0.0.0  网段通告  
[AR1-ospf-1-area-0.0.0.0]network 192.168.1.0 0.0.0.255
```
 
```
[AR3]ospf 1 router-id 10.3.3.3   
[AR3-ospf-1]area 0   
[AR3]interface GigabitEthernet 0/0/0   
[AR3-GigabitEthernet0/0/0]ospf enable 1 area 0  指定接口开始运行OSPF 
```
 
```
network 后面添加需要通告给邻居的网段  
        网络地址 +  反掩码
```
 
```
IP地址配置时 配置的是正掩码
```
 
```
正掩码 +  反掩码 = 255.255.255.255
```
 
```
正掩码 1 表示网络位  0 表示主机位  
反掩码 1 表示主机位  0 表示网络位
```
   

```
OSPF计算路由加入路由表后  
路由的protocol 是 OSPF  
路由的preference 是 10
```
 
```
OSPF邻居建立的过程：
```
 
```
OSPF邻居需要通过报文进行协商  
   hello报文（会携带一系列的参数 用于设备之间进行协商）a  
            （1.建立邻居关系的作用 2.维护邻居关系的作用）  
   周期（10s）发送hello报文
```
 
```
OSPF需要通过报文进行数据交互  
   DD报文、LSR报文、LSU报文、LSAck报文
```
   

```
[AR1]display ospf peer brief   查看OSPF建立的邻居关系
```
 
```
 OSPF Process 1 with Router ID 10.1.1.1  
  Peer Statistic Information  
 ----------------------------------------------------------------------------  
 Area Id          Interface                        Neighbor id      State      
 0.0.0.0          GigabitEthernet0/0/0             10.2.2.2         Full          
 ----------------------------------------------------------------------------  
[AR1]display ospf peer  查看已经建立邻居的详细信息
```
 
```
[AR1]dis ospf lsdb   查看OSPF的LSDB
```
 
```
 OSPF Process 1 with Router ID 10.1.1.1  
 Link State Database 
```
 
```
         Area: 0.0.0.0  
 Type      LinkState ID    AdvRouter          Age  Len   Sequence   Metric  
 Router    10.3.3.3        10.3.3.3             3  48    8000000B      10  
 Router    10.2.2.2        10.2.2.2            40  48    8000000C      10  
 Router    10.1.1.1        10.1.1.1           315  36    80000020       2  
 Network   10.1.23.3       10.3.3.3           268  32    80000005       0  
 Network   10.1.12.1       10.1.1.1           308  32    80000005       0
```
 
```
[AR1]dis ospf routing  查看OSPF计算的路由表
```
 
```
 OSPF Process 1 with Router ID 10.1.1.1  
  Routing Tables 
```
 
```
 Routing for Network   
 Destination        Cost  Type       NextHop         AdvRouter       Area  
 10.1.12.0/24       2     Transit    10.1.12.1       10.1.1.1        0.0.0.0  
 10.1.23.0/24       7     Transit    10.1.12.2       10.3.3.3        0.0.0.0  
 172.16.2.0/24      14    Stub       10.1.12.2       10.3.3.3        0.0.0.0
```
 
```
 Total Nets: 3    
 Intra Area: 3  Inter Area: 0  ASE: 0  NSSA: 0 
```
 
![Exported image](Exported%20image%2020251206145606-0.png)