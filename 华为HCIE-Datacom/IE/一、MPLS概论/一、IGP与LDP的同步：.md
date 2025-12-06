**IGP与LDP的同步：**  
LDP的邻居建立，需要通过IGP将传输地址进行相互学习后才能建立（该建立的方式式同步的建立）  
问题：  
1.主路径失效后，又恢复，流量从备路径要切换回主路径。主路径IGP邻居，但是LDP邻居还未建立  
会导致主路径IP访问正常，但是MPLS标签不存在（即就是IP报文的转发）
 
2.主路径底层IGP正常，但是LDP邻居故障  
会导致主路径IP访问正常，但是MPLS标签不存在（即就是IP报文的转发）
 
解决办法：  
1.开启IGP与LDP的同步功能  
当LDP邻居还未建立，IGP邻居正常建立但是会建立成为备份的IGP邻居（通过开销进行设置）  
IGP邻居在建立后会将自身的IGP开销设置为最大值，成为备份路径  
根据设置的时间定时器，在定时器老化后，设备会将IGP的开销值恢复  
[AR1-GigabitEthernet0/0/0]ospf ldp-sync 设置IGP与LDP同步功能开启  
[AR1-GigabitEthernet0/0/0]ospf timer ldp-sync hold-max-cost 1000 手工设置定时器的时间  
[AR1-GigabitEthernet0/0/0]ospf timer ldp-sync hold-max-cost infinite OSPF的开销，在LDP邻居建立之前都是最大值

![Exported image](Exported%20image%2020251206151217-0.png) ![Exported image](Exported%20image%2020251206151220-1.png)