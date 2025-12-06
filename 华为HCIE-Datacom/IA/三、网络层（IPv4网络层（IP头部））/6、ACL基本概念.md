```
ACL：访问控制列表（Access Control List）  
可以进行报文流的精确匹配，和其他技术结合达到控制网络访问的目的。
```
 
```
ACL是一个匹配工具，能够对报文进行匹配和区分
```
 
```
需要执行工具调用，来实现控制访问操作的  
  traffic-filter 、traffic-policy、router-policy、filter-policy、NAT等。
```
 
```
ACL的分类：  
1.基本ACL：取值范围\<2000-2999\>，只能匹配SIP 和基本参数值  
           一般用于路由信息的匹配。
```
 
```
2.高级ACL：取值范围\<3000-3999\>，可以匹配SIP、DIP、协议号、Sport、Dport和基本参数值  
          一般用于流量信息的匹配。
```
 
```
3.二层ACL：取值范围\<4000-4999\>，可以匹配SMAC、DMAC和基本参数
```
 
```
*[AR1]acl name PC2_PC3 2001  配置ACL的名称
```
 
```
ACL是由一系列permit或deny语句组成的、有序规则的列表：  
[AR1]acl 2000   
[AR1-acl-basic-2000]rule 5 permit source 192.168.1.1 0.0.0.0 
```
 
```
1.rule 5：规则+编号     
[AR1-acl-basic-2000]rule \<0-4294967294\>  每条规则，按照编号从小到大依次排列
```
 
```
2.permit/deny：允许或拒绝  通过ACL为匹配的信息 做的标记
```
 
```
3.source：匹配项，代指ACL要匹配的内容
```
 
```
4.192.168.1.1：参数，匹配项的具体内容
```
 
```
5. 0.0.0.0：通配符掩码，限制匹配的参数
```
 
```
正掩码：连续的0 和 连续1组合，1表示网络位 0表示主机位  
反掩码：连续的0 和 连续1组合，0表示网络位 1表示主机位  
通配符掩码：可以是不连续的1和0组合，0表示精确匹配，1表示任意匹配
```
 
```
参数：      192.168.1.1  
通配符掩码：0.0.0.255  
匹配的范围是 192.168.1.0-255
```

```
参数：      192.168.1.1  
通配符掩码：0.255.0.255  
匹配的范围是：192.0-255.1.0-255
```

```
在192.168.1.0/24的范围内，使用通配符掩码匹配出所有的奇数地址：  
参数        192.168.1.1  
通配符掩码    0.0.0.254   
0.0.0.11111110
```
 
```
在192.168.1.0/24的范围内，使用通配符掩码匹配出所有的偶数地址：  
参数        192.168.1.0  
通配符掩码    0.0.0.254   
0.0.0.11111110
```
   

```
ACL的匹配方式：  
 1.数据信息到达ACL后，从小到达依次匹配规则编号  
 2.如果数据信息匹配的某一个规则，则不在进行后续的规则匹配
```
 
```
 *在定义ACL规则时，需要将精细的规则，在范围规则之前进行书写  
  可以通过修改规则步长的方式，在规则之间添加新的规则  
  [AR1-acl-basic-2000]step \<1-20\>  修改规则的步长，默认步长为5。  
 *建议配置规则时，使用默认步长5
```
 
```
ACL存在默认的规则：（最大编号的规则节点）  
  根据ACL匹配的信息，才能判断默认规则的动作  
  如果判断ACL匹配的信息？需要通过执行工具来判断  
  执行工具：  
    1.匹配流量：ACL默认的规则是，允许所有     （traffic-filter）  
    2.匹配路由：ACL默认的规则是，拒绝所有     （filter-policy）
```
 
```
  [AR1-acl-basic-2000]rule 1000 permit source any  配置默认节点
```
 
![Exported image](Exported%20image%2020251206145658-0.png)

```
流量过滤：使用基本ACL来实现  
接口出方向过滤：  
[S1]acl 2000  
[S1-acl-basic-2000]rule 5 deny source 192.168.1.1 0.0.0.0   
[S1]interface GigabitEthernet 0/0/3  
[S1-GigabitEthernet0/0/3]traffic-filter outbound acl 2000 
```
 
```
acl 可以为流量 标记为permit 或 deny
```
 
```
traffic-filter收到流量标记为permit的放行  
              收到流量标记为deny的拒绝
```
 
```
流量过滤：使用基本ACL来实现  
接口入方向过滤：  
[S1]acl 2000  
[S1-acl-basic-2000]rule 5 deny source 192.168.1.1 0.0.0.0   
[S1]interface GigabitEthernet 0/0/1  
[S1-GigabitEthernet0/0/1]traffic-filter inbound acl 2000 
```
 
```
流量过滤：使用高级ACL来实现  
接口入方向过滤：  
[S1]acl 3000   
[S1-acl-adv-3000]rule 5 deny ip source 192.168.1.1 0 destination 192.168.1.3 0  
[S1]interface GigabitEthernet 0/0/1  
[S1-GigabitEthernet0/0/1]traffic-filter inbound acl 3000 
```
 
```
流量过滤：使用高级ACL来实现  
接口出方向过滤：  
[S1]acl 3000   
[S1-acl-adv-3000]rule 5 deny ip source 192.168.1.1 0 destination 192.168.1.3 0  
[S1]interface GigabitEthernet 0/0/3  
[S1-GigabitEthernet0/0/3]traffic-filter outbound acl 3000 
```

```
流量过滤入方向 一般应用在靠近源的位置  
流量过滤出方向 一般应用在靠近目标的位置
```

![Exported image](Exported%20image%2020251206145700-1.png)

```
要求AR1学习AR2的奇数路由，不学习偶数路由
```

```
[AR1]acl 2000   
[AR1-acl-basic-2000]rule 5 deny source 2.2.2.0 0.0.0.254   
[AR1-acl-basic-2000]rule 10 permit source 2.2.2.1 0.0.0.254  
[AR1]ospf  
[AR1-ospf-1]filter-policy 2000 import 
```

```
ACL注意点：  
配置acl一定是地址和通配符掩码的组合  
使用acl一定是要明确是出接口还是入接口  
默认规则：允许流量，拒绝路由
```
 
```
acl 匹配路由拒绝所有，匹配流量允许所有
```