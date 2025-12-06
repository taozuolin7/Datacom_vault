```
匹配工具：  
  ACL （路由、流量）  
  ip-prefix （路由）
```
 
```
执行工具：  
  路由： route-policy 、 filter-policy  
  流量： PBR、traffic-policy、traffic-filter
```

```
PBR：策略路由  
 和route-policy的书写方式，规则方式都一致  
 多个节点之间是或的关系  
 节点内可以存在多个条件语句 和 执行语句  
 节点内多个条件语句是与的关系  
 匹配规则按照节点从小到大依次匹配
```

```
PBR的配置方式：  
1.接口PBR：根据接口收到的流量执行策略的转发  
2.全局PBR：设备自身发起的流量执行策略的转发  
*可以同时配置，不会相互影响
```

![Exported image](Exported%20image%2020251206164652-0.png)

```
要求：  
1.PC1访问PC2的流量访问路径为AR1-AR2  
2.PC1访问PC3的流量访问路径为AR1-AR3-AR2
```

```
[AR1]acl 3000   
[AR1-acl-adv-3000]rule 5 permit ip source 192.168.1.0 0.0.0.255 destination 172.16.2.3 0  
[AR1]policy-based-route test permit node 10   
[AR1-policy-based-route-test-10]if-match acl 3000  
[AR1-policy-based-route-test-10]apply ip-address next-hop 10.1.13.3  
[AR1]interface GigabitEthernet 0/0/0  
[AR1-GigabitEthernet0/0/0]ip policy-based-route test    接口PBR的调用
```
 
```
[AR1]ip local policy-based-route test     全局PBR的调用
```

```
流策略（MQC：模块化的qos命令行）  
通过三个组件 组成（三板斧）  
1.traffic-classifier：流分类   匹配出需要的流量  
2.traffic-behavior：流行为     为流量执行对应的操作  
3.traffic-policy：流策略       关联流分类和流行为
```
 
```
该方式 具备 易读、易写、易移植 的能力
```

![Exported image](Exported%20image%2020251206164653-1.png)

```
[AR1]acl 3000   
[AR1-acl-adv-3000]rule 5 permit ip source 192.168.1.4 0 destination 172.16.2.6 0
```
 
```
[AR1]acl 3001   
[AR1-acl-adv-3001]rule 5 permit ip source 192.168.1.4 0 destination 172.16.2.5 0
```
 
```
[AR1]traffic classifier test   
[AR1-classifier-test]if-match acl 3000
```
 
```
[AR1]traffic classifier abc   
[AR1-classifier-abc]if-match acl 3001
```
 
```
[AR1]traffic behavior test  
[AR1-behavior-test]redirect ip-nexthop 10.1.13.3
```
 
```
[AR1]traffic behavior abc  
[AR1-behavior-test]deny
```
 
```
[AR1]traffic policy test 	  
[AR1-trafficpolicy-test]classifier test behavior test   
[AR1-trafficpolicy-test]classifier abc behavior abc 
```
 
```
[AR1]interface GigabitEthernet 0/0/0  
[AR1-GigabitEthernet0/0/0]traffic-policy test inbound 
```

```
要求：  
1.PC1访问PC2的流量拒绝  
2.PC1访问PC3的流量访问路径为AR1-AR3-AR2
```

```
在华为的eNSP模拟器中： PBR在模拟器中通过router设备来实现  
AR2220设备不支持
```