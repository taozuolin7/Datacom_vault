```
集成ISIS的由来：￼IETF             ISO               IETF 
```
 
```
TCP/IP模型      OSI模型          TCP/IP模型
```
 
```
OSPF             ISIS            集成ISIS
```
 
```
IP网络          CLNP网络          IP网络
```
 
```
IPv4/IPv6       CLNS服务         IPv4/IPv6
```
 
```
IPv4地址        NSAP地址       net地址/IPv4地址  
OSPF和ISIS都是链路状态路由协议，基于SPF计算的
```

```
什么企业会用到IS-IS：￼数据中心，运营商
```

```
NSAP地址：   
1.IDP： 相当于 网络位  
   1.AFI：表示 地址的分配机构  
   2.IDI：表示地址的范围域  
2.DSP： 相当于主机位  
   1.High Order DSP：将范围域进一步划分  
   2.system-id：主机地址，表示每一个终端设备  
   3.SEL：标识上层的应用服务
```
 
```
地址的长度范围是1-20字节：  
其中AFI是1字节，必须存在 49  
system-id是6字节，必须存在 0000.0000.0000  
SEL是1字节，必须存在 .00
```
 
```
IDI和high order dsp是12字节，可以省略
```

```
NET地址就是通过NSAP地址转换的  
即将NSAP地址的SEL字段设置为全0 就是 NET地址
```

```
IDP（Initial Domian Part）相当于IP地址中的主网络号。  
AFI（Authority and Format Identifier）与  
IDI（Initial Domain Identifier）两部分组成。
```
   

```
DSP（Domian Specific Part）相当于IP地址中的子网号和主机地址。  
High Order DSP、System ID和SEL三个部分组成。
```

```
[AR1-isis-1]network-entity 47.0001.0002.0003.0004.0005.0006.0000.0000.0001.00  
net地址通过16进制（4位为一组）进行表示
```

```
net地址的规划，一般将设备的RID规划为net地址的system-id
```
 
```
[AR1]router id 10.1.1.1 
```
 
```
RID值：10.1.1.1
```
 
```
将每一组进行填充：010.001.001.001   
将每一组进行重新组合：0100.0100.1001
```
 
```
[AR1-isis-1]netw 49.0001.0101.0100.1001.00
```
 
```
ISIS的设备类型：  
1.Level-1的设备  
   *类似于OSPF非骨干区域的IR设备  
   只能和L1设备建立L1的邻居关系
```
 
```
2.Level-2的设备  
   *类似于OSPF骨干区域的IR设备  
   只能和L2设备建立L2的邻居关系
```
 
```
3.Level-1-2的设备（默认的设备级别）  
   *类似于OSPF的ABR设备  
   和L1设备建立L1的邻居关系  
   和L2设备建立L2的邻居关系  
   和L1/2设备建立L1/2的邻居关系
```
   

```
[AR1-isis-1]is-level ?  
  level-1      
  level-1-2  
```
   

```
1  level-2 
```
 
```
[AR1]dis isis brief   查看ISIS的状态信息
```

```
ISIS的接口类型：  
1.Level-1的接口  
2.Level-2的接口  
3.Level-1-2的接口（默认的接口级别）
```
 
```
[AR1-GigabitEthernet0/0/0]isis circuit-level ?  
  level-1      
  level-1-2    
  level-2
```
 
```
[AR1]display isis interface GigabitEthernet 0/0/0     
查看ISIS接口的信息
```

```
设备级别和接口级别的关系：  
L1/2设备和L1接口 则发送L1的报文  
L1/2设备和L2接口 则发送L2的报文
```
 
```
发送什么级别的报文，取决于设备级别和接口级别的交集  
如果不存在交集，则按照设备的级别发送报文
```
 
```
邻居建立交互的报文级别一定要相同，不同无法正常协商建立邻居
```

```
ISIS的区域划分：  
*在net地址中配置的域 只是一个信息内容的表示  
1.骨干区域：是网络中所有连续的L2设备组合  
   不强制要求骨干区域的设备都存在相同的区域ID
```
 
```
2.非骨干区域：是网络中所有连续的L1设备组合  
   要求非骨干区域内的设备都必须配置相同的区域ID
```
 
```
*通过L1/2设备连接骨干区域和非骨干区域  
区域的划分：  
  OSPF是以设备作为划分的边界  
  ISIS是以链路作为划分的边界
```

```
[AR1]display isis peer  查看ISIS的邻居状态
```

![Exported image](Exported%20image%2020251206164306-0.png)

```
ISIS的网络类型：  
1.广播型网络（默认网络）  
2.P2P网络[AR2-GigabitEthernet0/0/0]isis circuit-type p2p
```

![Exported image](Exported%20image%2020251206164308-1.png)

```
[AR1]isis   
[AR1-isis-1]netw 49.0001.0000.0000.0001.00  
[AR1-isis-1]is-level level-1   
[AR1]interface GigabitEthernet 0/0/0  
[AR1-GigabitEthernet0/0/0]ip address 10.1.12.1 24   
[AR1-GigabitEthernet0/0/0]isis enable 1
```

```
[AR2]isis   
[AR2-isis-1]netw 49.0001.0000.0000.0002.00  
[AR2-isis-1]is-level level-1  
[AR2]interface GigabitEthernet 0/0/0   
[AR2-GigabitEthernet0/0/0]ip address 10.1.12.2 24   
[AR2-GigabitEthernet0/0/0]isis enable 1
```

```
ISIS的链路开销：  
1.和OSPF相同的是：  
    链路的开销的值越小，则路径越优先级  
   对于路由计算的开销值，是叠加路径的开销  
  **路由接收入方向的和 或 数据发送出方向的和
```
 
```
2.和OSPF不同的是：  
默认情况ISIS的cost计算和接口带宽没有直接关系  
   ISIS默认将所有的链路开销都作为10计算
```
   

```
ISIS的开销计算：（优选规则从高到低）  
1.接口开销设置：  
   [AR1-GigabitEthernet0/0/0]isis cost 30  
2.全局开销设置：  
   [AR1-isis-1]circuit-cost 35  
3.自动开销设置：  
   [AR1-isis-1]auto-cost enable 
```

```
 
```

```
OSPF计算路径开销：参考带宽/实际带宽
```

![Exported image](Exported%20image%2020251206164310-2.png)
    

 **Intermediate System to Intermediate System Routing Protocol****￼****中间系统到中间系统路由协议**

**CLNP（ConnectionLessNetwork Protocol，无连接网络协议）**  
NET（Network Entity Title，网络实体名称）  
NSAP（Network Service Access Point，网络服务访问点）

NET（Network Entity Title，网络实体名称）