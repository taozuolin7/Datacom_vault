![Exported image](Exported%20image%2020251206164633-0.png)  

```
AS 100    联盟AS  
AS1、AS2  联盟子AS
```

![Exported image](Exported%20image%2020251206164635-1.png)  

```
联盟的邻居建立：  
[AR1]bgp 1     
[AR1-bgp]confederation id 100    指定联盟AS的ID  
[AR1-bgp]peer 2.2.2.2 as-number 1    建立联盟子AS内的IBGP邻居  
[AR1-bgp]peer 2.2.2.2 con LoopBack 0   
[AR2-bgp]peer 3.3.3.3 as-number 2    建立联盟子AS间的EBGP邻居  
[AR2-bgp]peer 3.3.3.3 connect-interface LoopBack 0  
[AR2-bgp]peer 3.3.3.3 ebgp-max-hop 10 
```
 
```
对于建立的EBGP邻居，默认情况下设备会使用联盟AS号访问对方  
对于建立联盟子AS的EBGP邻居，需要事先指定联盟子AS的邻居AS号  
[AR2-bgp]confederation peer-as 2    指定AS 2 为联盟子AS号
```

```
联盟内的路由传递：  
从联盟AS-EBGP邻居学习到的路由，传递给联盟AS内的IBGP邻居 或 联盟AS间的EBGP邻居 都需要修改下一跳使得路由可达
```

```
***对于所有发出联盟AS的路由 都会清空携带的联盟子AS编号
```

==联邦（Confederation）==