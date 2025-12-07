影响QoS的因素又有哪些呢？  
带宽（Bandwidth）: 网络中IP包的传输速率，可用平均速率或峰值速率来表征。  
——带宽竞争，可以通过增大带宽来解决，但不能无限增大。  
延迟 （Latency）: 在两个参考点间，某一IP包从发送到接收间的时间间隔。  
——时延敏感流量，如视频、语音等。  
时延抖动 (Jitter）: 沿同一路径传输的一个数据流（Stream）中，不同分组传输延迟的变化。  
——与时延相关，时延小则抖动的范围小，对语音、视频等实时业务影响大。  
丢包率（Packet Loss）: 某一业务在网络中传输时，可允许的最大丢包率。  
——衡量网络可靠性，少量丢包对业务的影响不大，但大量丢包会严重影响传输效率。  
可用性（Availability）: 用户与IP业务连接的可靠性，包括建立时间、保持时间等。
 
通常QoS提供以下三种服务模型：  
Best-Effort service（尽力而为服务模型）  
Integrated service（综合服务模型，简称IntServ）  
Differentiated service（区分服务模型，简称DiffServ）

![Exported image](Exported%20image%2020251206151854-0.png)