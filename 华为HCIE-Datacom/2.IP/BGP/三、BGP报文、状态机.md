```
BGP的状态机：  
1.idle：初始化状态，等待邻居建立的使能事件（管理员手工指定建立的操作）  
2.connet：连接状态，发起TCP连接  
   1.连接失败，进入actice状态  
      1.active状态，再次发起TCP连接  
           1.TCP连接失败，直到TCP定时器超时退回connect状态  
           2.TCP连接成功，进入到opensent状态，并发送open报文  
   2.连接成功，进入到opensent状态，并发送open报文  
3.opensent：等待对端的open报文  
   1.open报文协商失败 （Router id要求邻居不能一致），发送notification报文退回至IDLE状态  
（[AR1-bgp]router-id 10.1.1.1 ）  
   2.open报文协商成功，进入到openconfirm状态，并发送keepalive报文  
4.openconfirm：等待对端的keepalive报文  
   1.没有收到keepalive报文，发送notification报文退回至IDLE状态  
   2.收到keepalive报文，进入established状态  
5.established：邻居建立完成  
   可以通过route-refresh报文，去请求路由信息  
   可以通过update报文，去更新、撤销路由信息  
   可以发送notification报文断开邻居关系  
   或者接口notification报文断开邻居关系
```
 ![Exported image](../../../Exported%20image%2020251206164538-0.octet-stream)

```
BGP的报文类型：  
*BGP的所有报文都是单播的  
*报文内容分类：  
 报文头部：所有BGP报文都存在的相同信息  
    Marker:   
    Length:   
    Type: 
```
 
```
 报文内容  
   1.open报文：发现邻居、建立邻居的           
```

![Exported image](Exported%20image%2020251206164540-1.png)

```
   2.keepalive报文：维护邻居的  
      周期60s发送一次，180s邻居失效￼                 [AR1] display bgp peer 2.2.2.2 verbose 查看bgp 邻居详细信息  
[AR1-bgp]timer keepalive 30 hold 90   
修改BGP通告的keepalive时间  
两端时间不同，不影响邻居的正常建立  
时间不同，会进行协商，协商为小的值
```

![Exported image](Exported%20image%2020251206164542-2.png)

```
   3.notification报文：用于通知错误，断开邻居的连接  
               
```

![Exported image](Exported%20image%2020251206164547-3.png)   
```
----------------------------以上为建立BGP对等体期间发送的报文--------------------------
```
 
```
   3.update报文：用于路由信息的通告、撤销
```

![Exported image](Exported%20image%2020251206164549-4.png)

```
         
   4.route-refresh报文：用于路由信息的请求       
```

![Exported image](Exported%20image%2020251206164550-5.png)

```
         \<AR1\>refresh bgp all import  发送route-refresh报文请求对端的路由信息  
         \<AR2\>refresh bgp all export  直接发送updata报文给对端路由信息
```

```
无论是ibgp还是ebgp，谁先使能bgp，谁就先发起TCP请求，  
建立完成后，发起方会发送Open报文，对端收到之后，会请求断开TCP，对端再发起ＴＣＰ，然后一直用下去
```