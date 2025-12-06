```
IPSec（IP Security） VPN一般部署在企业出口设备之间￼通过加密与验证等方式，实现了数据来源验证、数据加密、数据完整性保证和抗重放等功能。
```

```
IPSec协议体系￼IPSec不是一个单独的协议，它给出了IP网络上数据安全的一整套体系结构，包括￼AH（Authentication Header）、ESP（Encapsulating Security Payload）、IKE（Internet Key Exchange）等协议。
```

```
IPSec两个站点之间的数据的加密传输，及验证￼
```

![Exported image](Exported%20image%2020251206164836-0.png)

```
IPSec-VPN，暂时只能做在路由器出接口上：
```
 
```
IPsec配置思路：  
1.配置ipsec感兴趣流  
#  
acl number 3000    
 rule 5 permit ip source 192.168.1.1 0 destination 192.168.2.2 0   
#  
2.配置ike的提议集  
#  
ike proposal 1  
 encryption-algorithm aes-cbc-256  
 authentication-algorithm md5  
#  
3.配置ike的peer  
#  
ike peer AR3 v2  
 pre-shared-key simple huawei@123  
 ike-proposal 1  
 remote-address 10.1.23.3  
#  
4.配置ipsec的提议集  
#  
ipsec proposal 1  
 esp authentication-algorithm sha2-256   
 esp encryption-algorithm aes-256  
#  
5.配置ipsec的SA  
#  
ipsec policy test 1 isakmp  
 security acl 3000  
 ike-peer AR3  
 proposal 1  
#  
6.接口绑定ipsec的策略  
#  
interface GigabitEthernet0/0/1  
 ipsec policy test  
#￼￼[AR1]ip route-static 0.0.0.0 0 10.0.12.2 
```

```
IPSec基本原理：￼IPSec隧道建立过程中需要协商IPSec SA（Security  Association）安全联盟，IPSec SA 一般通过IKE协商。                  
```

![Exported image](Exported%20image%2020251206164841-1.png)

![Exported image](Exported%20image%2020251206164843-2.png) ![Exported image](Exported%20image%2020251206164845-3.png) ![Exported image](Exported%20image%2020251206164846-4.png) ![Exported image](Exported%20image%2020251206164848-5.png)