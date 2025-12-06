\> [!caution] This page contained a drawing which was not converted.   
 
```
OSPF的报文分为：  
1.报文头  
   固定的参数，所有报文都会携带
```

```
OSPF的报文：  
1.Hello报文：建立邻居，维护邻居  
             每一台设备都会发送hello报文（10s）  
             目标地址为 224.0.0.5、224.0.0.6（组播）  
2.DD报文：数据库描述报文，携带LSA的摘要来完成LSDB的同步  
（链路状态通告Link State Advertisement，LSA）
```
 
```
3.LSR报文：链路状态请求报文，向对方请求获取自身不存在的LSA。  
4.LSU报文：链路状态更新报文，向对方回复完整的LSA信息。  
5.LSAck报文：链路状态确认报文，向对方回复确认收到了LSU报  
文。
```

```
DD报文：  
I bit：置1 表示为第一个DD报文  
M bit：置1 表示为还有携带数据的DD报文  
MS bit：置1 表示为master设备  
seq：随机产生的数值
```

![Exported image](Exported%20image%2020251206145613-0.png)

```
OSPF的状态机：  
1.down：未使能ospf的状态  
2.init：发送或接收hello中不存在邻居的RID  
3.2-way：发送或接收hello中存在邻居的RID
```
 
```
4.exstart：通过DD报文进行主从设备的选举  
5.exchange：通过DD报文进行LSA摘要信息的同步  
6.loadding：通过LSR和LSU报文进行LSDB的同步  
7.FULL：通过LSAck报文确认LSDB同步完成
```

```
想要进一步进行数据同步，则需要建立邻接关系
```

![Exported image](Exported%20image%2020251206145615-1.png) ![Exported image](Exported%20image%2020251206145617-2.png)

```
exstart状态选举的主从设备
```
 
```
从设备使用主设备的seq  
主设备会以此将seq+1
```
 
```
该机制称为隐式确认
```

```
OSPF建立邻居的影响因素：（在报文中会协商的参数）  
1.version：要求两端的版本相同  
2.rotuer id：要求设备之间的RID一定不相同  
   1.同区域下 直连设备之间，要求RID一定不相同  
   2.同区域下 非直连设备之间，RID可以相同（但是会存在问题）  
3.area id：要求设备之间的区域ID一定相同  
4.auth type/data：要求设备之间的认证一定相同  
5.netmask：要求设备之间的网段是相同的  
6.hello time：要求设备之间时间相同  
7.dead time：要求设备之间时间相同  
8.option：要求设备之间字段相同
```

![Exported image](Exported%20image%2020251206145620-3.png)

```
ospf相关的进阶配置
```
 
```
[AR2-GigabitEthernet0/0/0]ospf dr-priority 100   
修改OSPF的接口优先级值
```
 
```
\<AR1\>dis ospf error interface GigabitEthernet 0/0/0  检查ospf在0/0/0接口存在的错误计数
```
 
```
    如果发现错误计数都为0：  
        1.邻居关系正常建立  
        2.报文没有发送
```
 
```
\<AR1\>reset ospf counters 清除OSPF的错误计数
```
 
```
在ospf中这么复杂的邻接的建立 和 报文交换机制 是为了什么？
```
 
```
 为了安全的建立连接和安全的报文交互  
 OSPF是基于IP的
```

```
DR、BDR的基本概念：  
1.DR：指定路由、BDR：备份指定路由  
2.没有成为DR和BDR的设备，称为DROther设备
```
 
```
选举方式：  
1.通过hello报文中携带的优先级值  
  优先级值越大越优先，取值范围0-255，默认值为1  
  如果取值为0，则代指不参与DR、BDR的选举  
2.如果优先级值相等，则优选RID大的
```
 
```
邻接的建立：  
1.DROther和DR建立邻接  
2.DROther和BDR建立邻接  
3.DR和BDR建立邻接  
4.DROther之间建立邻居
```
 
```
DR、BDR的特点：  
1.DR、BDR是不可抢占的  
2.DR失效 BDR一定会成为DR  
3.先选举BDR，在选举DR  
4.选举时间为40s（邻居的超时时间）
```

```
OSPF的网络类型：  
1.广播（BMA）  
2.NBMA  
3.P2P：串口的，不选举DR、BDR，所以建立的过程更加快速  
4.P2MP
```
 
```
[AR2-GigabitEthernet0/0/0]ospf network-type  修改OSPF的网络类型
```

```
OSPF中邻居关系的建立：
```

![Exported image](Exported%20image%2020251206145621-4.png)
 
![Exported image](Exported%20image%2020251206145623-5.png)

**NBMA 网络的全称是 "Non-Broadcast Multiple Access"（非广播多路访问网络）**。  
这是一种特殊类型的计算机网络，在该网络中：

- ==允许多个设备连接到同一网络==
- ==支持点到点通信，但==**不提供广播功能**
- ==数据只能通过虚拟电路或交换结构直接从一台计算机传输到另一台，而不能通过广播方式发送给所有设备==

典型的 NBMA 网络包括帧中继 (Frame Relay)、X.25 和异步传输模式 (ATM) 等广域网技术。在 OSPF 路由协议中，NBMA 是定义的四种主要网络类型之一。  
NBMA 网络与广播多路访问 (BMA) 网络相对，后者支持广播功能，如常见的以太网。

==一、核心报文协商参数（Hello/DBD 报文中明确携带，必须完全一致）==

|   |   |   |   |
|---|---|---|---|
|**校验参数**|**强制匹配要求**|**异常现象**|**华为设备排查方法**|
|Version（版本）|两端必须为同一版本（主流为 OSPFv2，IPv6 用 OSPFv3）|不发送 Hello 报文，或接收后直接丢弃，无邻居关系|执行display ospf查看协议版本；确认接口未误配置 OSPFv3|
|Router ID（RID）|1. 直连邻居必须不同；  <br>2. 非直连邻居建议不同（相同会导致 LSDB 冲突）|直连时无法建立邻居；非直连时可能出现路由振荡|执行display ospf router-id核查；冲突时通过ospf router-id手动指定唯一 RID|
|Area ID（区域 ID）|两端接口所属 OSPF 区域必须相同（骨干区域 0 需特殊注意）|\接收 Hello 报文后丢弃，邻居停留在 Down 状态|执行display ospf interface [接口名]查看区域配置；检查是否误将接口划入不同区域|
|Auth Type/Data（认证类型 / 数据）|认证类型（不认证 / 明文认证 / MD5/HMAC-MD5 等）及密钥必须一致|收到 Hello 报文因认证失败丢弃，邻居无法初始化|执行display ospf interface [接口名]查看认证配置；核对ospf authentication-mode命令参数|
|Netmask（子网掩码）|直连接口必须在同一网段，子网掩码完全一致|邻居停留在 Down 状态，Hello 报文交互失败|执行display ip interface [接口名]查看 IP 和掩码；确认两端接口属于同一广播域 / 子网|
|Hello Time（Hello 间隔）|同一网段邻居 Hello 时间必须一致（广播 / 点到点默认 10s，NBMA 默认 30s）|Hello 报文发送周期不匹配，邻居无l,法完成初始化|执行display ospf interface [接口名]查看；通过ospf hello-interval命令统一配置|
|Dead Time（死亡时间）|通常为 Hello 时间的 4 倍，两端必须一致|提前判定邻居失效，或邻居超时时间不一致导致频繁震荡|执行display ospf interface [接口名]查看；通过ospf dead-interval命令调整|
|Option（选项字段）|关键标志位必须一致（如 E 位、N 位、DC 位等，决定区域类型和路由计算规则）|收到 Hello 报文后因字段不匹配丢弃，邻居无法建立|执行display ospf packet hello抓取报文分析；确认两端区域类型（骨干 / 非骨干）配置一致|
|MTU（最大传输单元）|DBD 报文中携带，默认要求两端接口 MTU 一致（华为设备默认校验）|邻居卡在 EXSTART 状态，无法交换 DBD 报文|执行display ip interface [接口名]查看 MTU；临时用os|

==二、接口隐含匹配要求（非报文协商，需兼容才能建立邻居）==

|   |   |   |   |
|---|---|---|---|
|**校验参数**|**兼容要求**|**异常现象**|**华为设备排查方法**|
|接口 OSPF 网络类型|两端网络类型需兼容（如广播↔广播、点到点↔点到点、NBMA 需手动指定邻居）|Hello 报文发送方式 / 目标地址不同，邻居停留在 INIT 状态|执行display ospf interface [接口名]查看网络类型；通过ospf network-type调整为兼容类型|
|接口物理状态|接口必须 UP/UP（物理层 + 数据链路层正常）|接口无法发送 Hello 报文，邻居始终为 Down 状态|执行display interface [接口名]查看物理状态；排查线缆、链路协议（如以太网协商、PPP 状态）|
|链路可达性|直连链路需双向互通，无 ACL / 防火墙阻断 OSPF 报文|单向能收到 Hello 报文，邻居停留在 INIT 状态|执行display acl查看是否有规则阻断 OSPF 协议号 89；用ping测试直连链路连通性|

==三、邻居建立异常状态快速排查流程（按优先级执行）==

2. **邻居 Down 状态**==：优先查接口物理状态→IP 网段 / 掩码→Area ID→认证配置==
3. **邻居 INIT 状态**==：查链路双向可达→ACL 阻断→OSPF 网络类型不兼容→Hello 报文目标地址==
4. **邻居 EXSTART 状态**==：优先查 MTU 不匹配→Router ID 冲突→接口 OSPF 网络类型不兼容==
5. **邻居 LOADING 状态**==：查 LSR/LSU 报文交互→链路带宽瓶颈→设备 CPU / 内存过载==
6. **邻居 FULL 后震荡**==：查 Dead Time 不匹配→接口抖动→路由协议冲突→RID 重复==