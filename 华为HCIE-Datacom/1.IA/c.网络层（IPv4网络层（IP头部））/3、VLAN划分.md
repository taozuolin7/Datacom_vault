```
1.VLAN存在的原因：￼传统以太网共享网络，所有设备在一个广播域内。  
交换机泛洪转发未知单播帧、组播帧、广播帧，会造成大量带宽的浪费。  
设备资源利用率下降，整体网络出现卡顿、延迟、拥塞。  
为了解决广播域过大的问题：  
通过VLAN进行广播域的分隔，将大的广播域划分为一个个小的广播域。
```

```
VLAN帧：  
802.1Q帧 =  以太网Ⅱ格式帧  +  802.1Q tag  
802.1Q tag:  
1.TPID（标签协议标识符）：16bit，标识数据帧的类型，值为0x8100时表示802.1Q帧。  
2.PRI（优先级）：3bit，标识帧的优先级，主要用于QoS。  
3.CFI（标准格式指示符）：1bit，在以太网环境中，该字段的值为0。  
4.VLAN ID（VLAN标识符）：12bit，标识该帧所属的VLAN。  
    取值范围是0-4095
```

```
VLAN：（Virtual Local Area Network）虚拟局域网  
[S1]undo info-center enable   关闭日志提示  
[S1]display vlan   查看VLAN和接口的对应关系
```

```
VLAN基本概念：  
1.设备默认存在VLAN 1，且VLAN 1被所有接口绑定  
2.VLAN 1也称为系统VLAN  
3.VLAN 1不能被修改，不能被删除  
4.[S1]vlan 10  创建一个VLAN  
5.[S1]vlan batch 20 30  同时创建多个VLAN  
6.[S1]vlan batch 50 to 100  创建一组连续的VLAN  
*创建VLAN，默认不存在接口绑定  
*每一个VLAN都是一个广播域，需要进行接口的划分
```

```
VLAN的划分方式：  
1.基于接口的VLAN划分（重要：）  
2.基于MAC的VLAN划分：一般用于哑终端的划分  
   通过MAC地址分配对应的VLAN  
   [S1]vlan 10   
   [S1-vlan10]mac-vlan mac-address 5489-9811-1111  
   [S1]interface GigabitEthernet 0/0/1  
   [S1-GigabitEthernet0/0/1]mac-vlan enable   
   [S1-GigabitEthernet0/0/1]port hybrid untagged vlan 10
```
 
```
4.基于协议的VLAN划分  
   [S1]vlan 10   
   [S1-vlan10]protocol-vlan ipv4    
   [S1]interface GigabitEthernet 0/0/2	   
   [S1-GigabitEthernet0/0/2]protocol-vlan vlan 10 all   
   [S1-GigabitEthernet0/0/2]port hybrid untagged vlan 10
```

```
基于接口的VLAN划分：  
1.access接口，连接接口，一般用于连接终端设备（PC、路由器、防火墙）  
2.trunk接口，干道接口，一般用于连接交换机设备  
3.hybrid接口，默认的类型
```

```
一个接口的能力：  
可以接收数据（接口入方向对于数据的处理  
可以发送数据（接口出方向对于数据的处理  
[S1]dis port vlan  查看接口类型和VLAN的对应关系
```
 
```
一个数据的分类：  
TG: Tagged，带有VLAN标签的数据    802.1Q帧  
UT: Untagged，不带VLAN标签的数据  以太网Ⅱ帧
```

```
下午：***交换机内的数据一定存在VLAN标签
```

```
access接口：  
[S1]interface GigabitEthernet 0/0/1  
[S1-GigabitEthernet0/0/1]port link-type access   修改接口的类型为access  
[S1-GigabitEthernet0/0/1]port default vlan 10    将接口划分进VLAN 10
```
 
```
*一个VLAN可以绑定多个接口 ； 一个接口只能绑定一个VLAN  
*PVID：就是接口的VLAN ID值
```
 
```
1.入方向的处理  
  1.接收untag数据帧的处理  
     会为untag数据帧，打上接口的PVID值，变为tag数据帧  
     将tag数据帧接收到交换机内部  
  2.接收tag数据帧的处理  
     1.数据携带的VLAN标签和接口的PVID值相同，将tag数据帧接收到交换机内部    
     2.数据携带的VLAN标签和接口的PVID值不同，不接收将数据帧直接丢弃  
2.出方向的处理  
  1.判断数据携带的VLAN标签是否和接口的PVID值相同  
    1.相同，剥离数据的VLAN标签，变为untag数据帧转发  
    2.不同，则不在该接口处理数据
```
 
```
测试：￼同子网同VLAN通信  
  使用ACCESS完成同子网同VLAN的数据通信  
同子网不同VLAN通信  
  使用ACCESS完成同子网不同VLAN的数据通信
```

```
trunk接口：  
[S2]interface GigabitEthernet 0/0/3  
[S2-GigabitEthernet0/0/3]port link-type trunk     修改接口类型为trunk  
[S2-GigabitEthernet0/0/3]port trunk pvid vlan 10  配置trunk接口的PVID值  
[S2-GigabitEthernet0/0/3]port trunk allow-pass vlan 10 20   配置trunk的允许列表
```
 
```
*一个VLAN可以绑定多个接口，一个接口可以绑定多个VLAN
```
 
```
1.入方向处理  
   1.收到untag数据帧的处理，打上接口的PVID值，并判断PVID是否在接口的允许列表中  
          1.在列表中，接收tag数据进交换机内部  
          2.不在列表中，丢弃该数据  
   2.收到tag数据帧的处理，判断数据携带的VLAN标签是否在接口的允许列表中  
          1.在列表中，接收tag数据进交换机内部  
          2.不在列表中，丢弃该数据
```
 
```
2.出方向处理  
  1.tag数据帧如何处理，判断数据携带的VLAN标签是否在接口的允许列表中  
    1.不在列表中，不在该接口处理数据  
    2.在列表中，则继续判断数据携带的VLAN标签是否和接口的PVID值相同  
      1.相同，则剥离VLAN标签发送untag数据帧  
      2.不同，则保留VLAN标签发送tag数据帧
```
 
```
测试：￼同子网同VLAN通信  
  使用TRUNK完成同子网同VLAN的数据通信  
同子网不同VLAN通信  
  使用TRUNK完成同子网不同VLAN的数据通信
```

```
#  
interface GigabitEthernet0/0/1  
 port link-type trunk  
 port trunk pvid vlan 10  
 port trunk allow-pass vlan 10  
#
```

```
==该配置的==  
功能是相同的
```

```
#  
interface GigabitEthernet0/0/1  
 port link-type access  
 port default vlan 10  
#
```

```
PVID：  
在acess接口下只能配置一个  
  入方向为untag数据帧打上标签  
  出方向为tag数据帧剥离标签
```
 
```
在trunk接口下只能配置一个  
  入方向为untag数据帧打上标签  
  出方向为tag数据帧剥离标签
```

```
hybrid接口： 混合接口，就是将access和trunk的功能融合了
```
 
```
具备两个列表：  
  1.untag列表：类似于access的pvid，可以发送untag数据帧  
  2.tag列表： 类似于trunk的允许列表，可以发送tag数据帧
```
 
```
也具备接口的PVID值：  
  只拥有入方向为untag数据帧打标签的能力
```
 
```
[S1]interface GigabitEthernet 0/0/1  
[S1-GigabitEthernet0/0/1]port link-type hybrid  默认接口类型为hybrid  
[S1-GigabitEthernet0/0/1]port hybrid pvid vlan 10     配置hybrid接口的PVID  
[S1-GigabitEthernet0/0/1]port hybrid tagged vlan 10 20  配置hybrid的tag列表  
[S1-GigabitEthernet0/0/1]port hybrid untagged vlan 30 40  配置hybrid的untag列表
```
 
```
1.入方向处理  
    1.收到untag数据帧，打上接口的PVID值，判断PVID值是否在tag列表或untag列表中  
         1.在列表中，则保留数据标签，接收到交换机内部  
         2.不在列表中，则丢弃  
    2.收到tag数据帧，判断数据携带的VLAN标签是否在tag列表或untag列表中  
         1.在列表中，则保留数据标签，接收到交换机内部  
         2.不在列表中，则丢弃
```
 
```
2.出方向处理  
    1.判断数据携带的VLAN标签，是否在tag列表或untag列表中  
         1.不在tag列表或untag列表中，则数据不在该接口转发  
         2.在tag列表中，发送数据保留VLAN标签  
         3.在untag列表中，发送数据剥离VLAN标签
```
 
![Exported image](../../../Exported%20image%2020251206145705-0.octet-stream)