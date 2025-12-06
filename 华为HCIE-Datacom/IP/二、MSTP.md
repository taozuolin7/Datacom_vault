```
MSTP：802.1S  
1.多实例生成树  
2.多域生成树
```

```
多实例生成树：  
1.相比于单生成树（STP、RSTP），可以根据物理拓扑虚拟出多颗逻辑上的生成树  
2.多实例生成树如何区分不同生成树的信息？  
     根据生成树的映射信息来实现区分  
     映射信息就是设备所在的VLAN ID值  
3.vlan和实例映射的关系？  
     实例id就是逻辑生成树的编号  
     vlan就是逻辑生成树可以转发的数据  
     一个生成树的实例 可以 绑定多个VLAN的数据  
4.多实例生成树影响的范围？  
     类似于ospf的区域，通过管理员指定的值进行域的划分
```

![Exported image](Exported%20image%2020251206164735-0.png)

```
#一个域内的所有交换机，域配置都是相同的  
stp region-configuration  
 region-name 123  
 revision-level 21  
 instance 1 vlan 10  
 instance 2 vlan 20  
 active region-configuration  激活配置  
#  
[S1]stp instance 1 priority 0    设置设备在实例1的stp优先级值￼[S2]stp instance 1 root secondary   设置设备在实例1中作为备根桥￼[S2]stp instance 2 root primary 
```