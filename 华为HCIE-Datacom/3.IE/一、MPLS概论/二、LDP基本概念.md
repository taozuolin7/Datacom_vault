**MPLS：多协议标签交换**  
本质就是标签转发的能力（转发(数据)平面的功能  
对标签的生成，需要其他协议的帮助

**LDP：****label D****istribute** **protocol** **标签分发协议**￼LDP的作用：就是在LSR之间建立LDP会话，然后向邻居通告标签和FEC（转发等价类）信息，  
以此来达到为路由动态分配标签并建立LSP的目的  
￼LDP的邻接体：  
收到对方发送的hello报文就认为对方是自己的邻接体  
1.本地邻接体：就是直连的设备之间通过组播发送hello消息  
2.远端邻接体：就是非直连的设备之间通过单播发送hello消息
 
LDP的会话：发现、建立、维护  
1.本地LDP会话：就是直连的设备建立会话  
2.远端LDP会话：就是非直连的设备建立会话  
**主要进行会话消息、标签消息、地址消息等协商
 
LDP的对等体：  
收到对方的会话消息、标签消息、地址消息等认为自己和对方建立了对等体

**LDP的4大消息类型：**  
1.discover  
2.session  
3.advertise  
4.notification

![Exported image](Exported%20image%2020251206150313-0.png)

[AR1]mpls lsr-id 1.1.1.1 即充当LDP设备的标识，默认也会作为传输地址使用  
[AR1]mpls  
[AR1]mpls ldp  
[AR1]interface GigabitEthernet 0/0/0  
[AR1-GigabitEthernet0/0/0]mpls  
[AR1-GigabitEthernet0/0/0]mpls ldp
 
LDP的发现机制：（LDP邻接体的建立）  
**就是通过LDP的discover消息来完成LSR之间的发现，使用的报文为hello报文  
1.基本的发现机制  
LSR周期（5s）发送LDP链路的hello消息用于建立本地LDP会话  
hello报文使用UDP报文（646），目标地址为224.0.0.2  
如果接口收到hello消息，则认为链路上存在LDP邻居  
2.扩展的发现机制  
[AR1]mpls ldp remote-peer AR4  
[AR1-mpls-ldp-remote-ar4]remote-ip 4.4.4.4 指定的远端邻居地址，该地址要求路由可达  
LSR周期（15s）发送LDP链路的hello消息用于建立本地LDP会话  
hello报文使用UDP报文（646），目标地址为4.4.4.4  
如果收到hello消息，则认为存在LDP邻居  
*discover消息：主要通过hello消息来发现和维护邻接关系  
hello消息5s发送15s超时，15s发送45超时  
hello消息内会携带传输地址（LSR-id地址）  
传输地址之间是相互通信的（建立TCP连接的）
 
对于传输地址的修改：  
1.如果使能了MPLS，则无法直接修改MPLS LSR-ID（关闭MPLS的使能，然后进行修改）  
2.[AR1-mpls-ldp]lsr-id 10.1.1.1 将hello报文通告的LSR-ID 和 传输地址都修改为配置的地址值  
3.[AR1-GigabitEthernet0/0/0]mpls ldp transport-address LoopBack 2  
将hello报文通告的传输地址替换为 loopback 2接口的地址   *TCP连接建立：  
当收到邻居的hello消息后，根据携带的传输地址，判断其是否可达  
如果地址可达，则由传输地址大的一方发起TCP连接建立（TCP报文都是单播的，源目IP地址为双方的传输地址）  
如果地址不可达，则不会建立TCP连接，也不会发送任何报错消息，继续发送hello报文  
***如果地址小的一方存在大的一方路由条目，但是地址大的一方不存在小的一方路由条目  
该场景下TCP连接不会建立，因为设备可以通过hello报文中的传输地址判断谁的地址更大  
所以当判断自身的传输地址为小的一方，就只能被动等待对方发起TCP连接

![Exported image](Exported%20image%2020251206150315-1.png)

2.session：会话消息，用来建立、维护、终止邻居之间的会话  
1.initialization：TCP建立完成后，由传输地址大的一方发送初始化消息，协商LDP的参数  
报文信息

![Exported image](Exported%20image%2020251206150316-2.png)

2.keepalive：维护邻居之间的会话  
收到邻居的init消息后，如果参数协商没有问题，则回复keepalive报文并发送init报文与对方协商  
报文信息  
对于keepalive消息周期15s发送，45s超时

![Exported image](Exported%20image%2020251206150321-3.png)

3.advertis message：通告消息  
1.address 消息  
由传输地址大的一方 回复 地址小的一方一个keepalive报文 同时发送一个address消息  
address消息内容：  
1.通告地址

![Exported image](Exported%20image%2020251206150322-4.png)  

2.撤销地址

![Exported image](Exported%20image%2020251206150324-5.png)  

2.Label 消息  
LSR之间将自身所有的映射消息同步给邻居  
1.lable mapping：宣告FEC/标签映射信息

![Exported image](Exported%20image%2020251206150326-6.png)

2.lable withdrawal：撤销FEC/标签映射信息

![Exported image](Exported%20image%2020251206150327-7.png)

3.lable release：释放FEC/标签映射信息

![Exported image](Exported%20image%2020251206150329-8.png)

4.lable request：请求FEC/标签映射信息

![Exported image](Exported%20image%2020251206150331-9.png)

5.lable abrot request：终止未完成的标签请求

![Exported image](Exported%20image%2020251206150336-10.png)

4.notification message：通知消息  
用于提供差错消息的

![Exported image](Exported%20image%2020251206150338-11.png)

LDP的状态机：

![Exported image](Exported%20image%2020251206150742-12.png)

在任何状态下收到通知消息或该状态的定时器超时，都会进入non-existent（重新建立TCP的连接）