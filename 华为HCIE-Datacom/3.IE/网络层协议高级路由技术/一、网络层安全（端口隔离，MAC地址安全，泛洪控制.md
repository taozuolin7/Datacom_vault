**端口隔离：**  
实现交换机连接终端的数据信息隔离  
通过端口绑定隔离组，隔离组内的接口不能相互通信  
[S1]interface GigabitEthernet 0/0/2  
[S1-GigabitEthernet0/0/2]port-isolate enable group 1
 
**端口隔离的模式：**  
1.二层隔离  
[S1]port-isolate mode l2  
2.二层、三层隔离  
[S1]port-isolate mode all
 
**端口隔离的分类：**  
1.双向隔离  
在同一个隔离组内的接口，不能相互转发数据信息  
2.单向隔离  
[S1]interface GigabitEthernet 0/0/1  
[S1-GigabitEthernet0/0/1]am isolate GigabitEthernet 0/0/3  
单向隔离，指定0/1 接口的数据不能向 0/3接口转发
 
**MAC地址表安全：**  
[S2]mac-address static 5489-9833-3333 GigabitEthernet 0/0/3 vlan 1  
[S2]mac-address blackhole 5489-9833-3334 vlan 1  
动态MAC地址表项是交换机收到了流量后，将流量的SMAC和接口的端口进行关联生成的表项  
[S2]mac-address aging-time \<0,10-1000000\> 设置动态MAC地址表项的老化时间  
[S2-GigabitEthernet0/0/1]mac-limit maximum 2 设置接口动态MAC地址的学习数量（不包含静态MAC地址）  
如果超出了设置的值，交换机就按照未知单播帧进行泛洪处理  
[S2-GigabitEthernet0/0/2]mac-address learning disable 设置接口不学习动态MAC地址  
[S2-GigabitEthernet0/0/2]mac-address learning disable action discard 对于端口没有保存的MAC地址，执行丢弃操作  
[S2-GigabitEthernet0/0/2]mac-address learning disable action forward 对于端口没有保存的MAC地址，执行泛洪操作（默认）  
**如果按照MAC地址表安全记录MAC地址，同时接口不学习动态MAC地址，如果动态MAC地址表老化，则无法实现数据访问
 
**端口安全：**  
将接口学习到 会 记录的MAC地址 转换为安全MAC  
1.安全动态MAC  
2.安全静态MAC  
3.stick MAC  
[S2]interface GigabitEthernet 0/0/2  
[S2-GigabitEthernet0/0/2]port-security enable 将端口转变为安全端口，同时将接口的动态MAC地址转换为安全动态MAC地址  
[S2-GigabitEthernet0/0/2]port-security mac-address sticky 设置为安全stick MAC  
[S2-GigabitEthernet0/0/5]port-security mac-address sticky 5489-9856-5656 vlan 1 设置安全静态MAC地址
 
对于安全动态MAC地址的处理：  
[S2-GigabitEthernet0/0/2]port-security aging-time \<1-1440\> 设置安全动态MAC地址表项的老化时间（默认不老化）  
[S2-GigabitEthernet0/0/2]port-security max-mac-num \<1-4096\> 设置安全MAC地址的学习数量（默认数量为1）  
[S2-GigabitEthernet0/0/2]port-security protect-action 如果收到没有保存的MAC地址，则执行一下操作  
protect 丢弃报文  
restrict 丢弃报文，提示日志（默认）  
shutdown 接口error-down￼  
**MAC地址漂移处理：**  
1.设置端口优先级  
[S3]interface GigabitEthernet 0/0/1  
[S3-GigabitEthernet0/0/1]mac-learning priority 3  
设置MAC地址学习的优先级值，默认值为0，优先级值越大越优先  
2.漂移检测+漂移处理  
[S3]mac-address flapping detection 配置全局MAC地址的漂移检测  
[S3]mac-address flapping detection exclude vlan \<1-4094\> 排除部分VLAN不做MAC地址漂移检测  
[S3]interface GigabitEthernet 0/0/2  
[S3-GigabitEthernet0/0/2]mac-address flapping trigger 接口发生MAC地址漂移后的处理  
error-down 接口shutdown  
quit-vlan 接口退出VLAN  
[S3]mac-address flapping quit-vlan recover-time \<0,1-1440\> 配置接口发生MAC地址漂移后，退出VLAN的自动恢复时间，默认10分钟  
[S3]error-down auto-recovery cause mac-address-flapping interval \<30-86400\> 设置漂移检测端口shutdown后的恢复时间￼￼  
**交换机流量控制：**  
1.交换机对于广播、组播、未知单播（BUM流量）执行的是泛洪动作  
2.如果存在大量泛洪报文时，则会降低设备的性能  
解决办法：  
通过减低报文的转发速率  
[S3-GigabitEthernet0/0/1]unicast-suppression 未知单播帧的限值  
INTEGER\<0-100\> 设置报文的速率对比接口百分比的速率  
block 超出阈值的部分，阻塞发送接口  
packets 设置报文的速率值  
[S3-GigabitEthernet0/0/1]multicast-suppression 组播帧的限值  
INTEGER\<0-100\> 设置报文的速率对比接口百分比的速率  
block 超出阈值的部分，阻塞发送接口  
packets 设置报文的速率值  
[S3-GigabitEthernet0/0/1]broadcast-suppression 广播帧的限值  
INTEGER\<0-100\> 设置报文的速率对比接口百分比的速率  
block 超出阈值的部分，阻塞发送接口  
packets 设置报文的速率值
 
[S3-GigabitEthernet0/0/1]storm-control unicast min-rate cir 1000 max-rate cir 3000  
[S3-GigabitEthernet0/0/1]storm-control multicast min-rate cir 1000 max-rate cir 3000  
[S3-GigabitEthernet0/0/1]storm-control broadcast min-rate cir 1000 max-rate cir 3000  
设置广播风暴控制的最小值为1000 最大值为3000  
*报文转发的速率大于最大值后对应的处理动作：  
[S3-GigabitEthernet0/0/1]storm-control action  
block 阻塞接口  
shutdown 关闭接口