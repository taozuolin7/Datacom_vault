**为什么要有qos？**
为了提供网络质量的服务（提高一部分的数据转发，降低一部分的数据转发）  
为了带宽资源的合理利用（带宽的有偿性服务）
 
前提:在网络中，带宽资源是远远不够使用的（带宽资源会被滥用）  
qos的三种服务模型：1.先进先出 2.资源预留 3.区分服务
 
qos区分流量的方式：简单流分类、复杂流分类  
分类、打标、映射、流分类
 
对于区分的流量，qos需要执行拥塞管理

### **拥塞管理：**  
网络中出现间歇性的拥塞或关键性报文需要被优先转发时，需要进行的==队列调度==
上下游链路速率不匹配，多个接口收到的流量从一个出口发送

**当网络带宽足够时，qos的拥塞管理机制是没有作用的*****
 
**拥塞的影响：**  
1.增加了报文的时延和抖动  
2.时延过高会导致重传  
3.吞吐量不稳定
 
**什么是队列？**  
硬件队列：设备默认的队列，链路不存在拥塞时使用的队列  
分类：没有分类的  
插入: 没满就存（满了就出现软件队列）  
调度：先进先出
 
软件队列：当网络出现拥塞后，硬件队列满存，则软件队列出现  
分类：分类器（将不同的流量打上不同的标记）  
插入：插入机制（没满就存，满了就尾丢弃）  
调度：调度机制（通过qos的策略将软件队列的报文放入到硬件队列中）  
*软件队列默认情况下是0 1 2 3 4 5 6 7 一共8个队列

**拥塞管理的处理步骤：**  
1.链路已经拥塞，软件队列已经出现  
2.使用流分类为报文打上不同的标记  
3.根据报文携带的标记值，对应qos的LP值  
LP值就对应队列数值  
4.根据队列的调度机制，执行不同队列内报文的转发顺序

**LP：****Locak preference** **本地优先级值 （0-7**）  
设备qos自身具备的优先级值，和其他优先级值进行对照的  
还可以和自身的软件队列进行对照（一对一）

**软件队列的类型：**  
1.PQ队列：优先级队列  
提供了快速转发业务，适合实时性的业务转发（语音、视频、直播等）  
优点:高优先级的业务可以得到最大程度的保障  
缺点:底优先级的业务饿死，没有带宽保障  
2.RR队列：轮询队列  
优点：每一个队列都能得到转发，不会出现饿死的情况  
缺点：没有体现区分的能力，可能会为高优先级的业务带来时延  
3.WRR队列：加权的轮询队列  
在轮询队列的基础上增加权重值（手工分配）（*将权重值理解为转发报文的个数）  
权重值在一轮转发后，恢复为初始值 （按照最高队列的权重值作为一轮转发的值判断）  
优点：每一个队列都能得到带宽，不会饿死，存在区分服务的特点  
缺点：低优先级报文过大会导致高优先级出现时延问题  
4.DRR队列：赤字的轮询队列  
在轮询队列的基础上增加权重值（手工分配）（*将权重值理解为转发报文的大小）  
权重值在每一次转发后，都增加（如果队列现有的值为赤字，则不转发业务数据，只有赤字大于0，才能转发业务数据）  
优点：每一个队列都能得到带宽，不会饿死，存在区分服务的特点  
缺点：低优先级报文过大会导致高优先级出现时延问题  
**5.WFQ队列：加权的公平队列**  
在队列调度前，为每一个队列分配权重值（手工分配）（*将权重值理解为分配的带宽占比）  
1.按照优先级值分配权重  
优点：按照权重值实现公平调度，自动分类（低优先级不会饿死）  
缺点：不支持自定义类的区分  
2.基于流的分类（就是CBQ的BE队列）  
将SIP+DIP+协议号+Sport+Dport+tos 六元组，将该值进行哈希计算得到的值就是队列值  
所有hash计算的队列平分高级优先级队列剩下的带宽值  
**6.CBQ队列：基于类的加权公平队列**  
基于类的，为每一个队列分配固定的带宽值  
先根据报文的信息（优先级、5元组）对报文进行匹配分类  
让不同类别的报文进入不同的队列中，没有匹配的报文进行默认的队列  
***CBQ自定义了三种队列：  
队列的名称和DSCP关键字表示的名称，没有任何关系  
1.EF队列：满足低延迟的业务，分配的带宽是最大值的带宽（最多可以使用的带宽值）  
提供给实时性业务的，提供低时延的服务  
*LLQ队列：特殊的EF队列  
提供给更低延迟的业务，建议给语音业务使用  
2.AF队列：满足带宽保障的业务，分配的带宽是最小值的带宽（最少可以使用的带宽值）  
提供给数据性的业务，提供带宽保障的服务  
3.BE队列：给不需要qos保障的业务使用，执行尽力而为的服务  
就是基于流的WFQ队列，队列满存丢弃后续的报文
 
优点：可以支持自定义的分类，按照带宽进行合理的分配  
缺点：CBQ没有动态的机制，所以只能手工配置
 
对于设备队列满存后，会将后续的报文直接丢弃掉（该动作称为尾丢弃）  
尾丢弃：  
1.无差别的丢弃  
2.会造成TCP全局同步  
3.会造成TCP饿死
 
尾丢弃的问题是无法直接解决的，只能通过方式进行优化
 
拥塞避免：  
RED：早期随机检测（提前丢弃报文延缓尾丢弃的到来）  
提前丢弃的报文是无差别的  
WRED：加权的早期随机检测（通过为不同优先级值的报文设置不同的丢包阈值和概率 延缓尾丢弃的到来）  
丢弃的报文是按照优先级值进行分类的
 
拥塞避免机制是使用在拥塞管理机制中的  
即在软件队列调度时可以使用拥塞避免机制  
PQ队列不适用拥塞避免机制
 
拥塞管理的队列一定是设备的出接口

AR4和AR5通过复杂流分类  
将访问PC1的流量  
AR4标记为802.1p优先级值4  
AR5标记为802.1p优先级值6

![Exported image](Exported%20image%2020251206151900-0.png)

```
AR4：  
#  
acl number 3000    
 rule 5 permit ip destination 192.168.3.1 0   
#  
traffic classifier test operator or  
 if-match acl 3000  
#  
traffic behavior test  
 remark 8021p 4  
#  
traffic policy test  
 classifier test behavior test  
#  
interface GigabitEthernet0/0/0  
 traffic-policy test outbound
```

```
AR5：  
#  
acl number 3000    
 rule 5 permit ip destination 192.168.3.1 0   
#  
traffic classifier test operator or  
 if-match acl 3000  
#  
traffic behavior test  
 remark 8021p 6  
#  
traffic policy test  
 classifier test behavior test  
#  
interface GigabitEthernet0/0/0  
 traffic-policy test outbound
```

AR1信任802.1p优先级值并可以进行修改  
AR1通过qos的映射表将  
802.1p优先级值4 送入队列2做转发  
802.1p优先级值6 送入队列3做转发  
AR1通过qos的映射表将  
802.1p优先级值4 发出时替换为dscp优先级值af11  
802.1p优先级值6 发出时替换为dscp优先级值af33

![Exported image](Exported%20image%2020251206151901-1.png)

```
#  
interface GigabitEthernet0/0/0  
 trust 8021p override  
#  
qos map-table dot1p-lp  
  input 4 output 2  
  input 6 output 3  
#  
qos map-table dot1p-dscp  
  input 4 output 10  
  input 6 output 30  
#
```

AR2信任DSCP优先级值并可以进行修改  
通过MQC将  
DSCP值af11送入pq队列7  
DSCP值af33送入wfq队列5  
为WFQ队列设置拥塞避免  
通过MQC将  
DSCP值af11 发出时替换为dscp优先级值af21  
DSCP值af33 发出时替换为dscp优先级值ef

![Exported image](Exported%20image%2020251206151903-2.png)

```
#  
interface GigabitEthernet0/0/0  
 trust dscp override  
#  
traffic classifier test operator or  
 if-match dscp af11   
traffic classifier abc operator or  
 if-match dscp af33   
#  
traffic behavior test  
 remark local-precedence cs7  
 remark dscp af21  
traffic behavior abc  
 remark local-precedence ef  
 remark dscp ef  
#  
traffic policy test  
 classifier test behavior test  
 classifier abc behavior abc  
#  
interface GigabitEthernet0/0/1  
 traffic-policy test outbound  
#  
qos queue-profile test  
  schedule wfq 5 to 6 pq 7  
  queue 5 weight 50  
  queue 6 weight 100  
  queue 5 drop-profile A  
#  
interface GigabitEthernet0/0/1  
 qos queue-profile test   
#  
drop-profile A  
wred dscp  
  dscp af33 low-limit 30 high-limit 80 discard-percentage 20  
#
```

虽然基于DifServ模型的QoS为用户提供了服务质量保证，但是该模型也存在以下问题：  
① 队列少，无法针对特定用户做SLA保障，无法提供确定性的时延保障。  
② 无法为用户提供独立的物理资源。  
③ 只是单跳行为，没有改变网络拓扑架构。  
④ 不改变业务行为，单条流如果突发流量过大，依然会存在拥塞。