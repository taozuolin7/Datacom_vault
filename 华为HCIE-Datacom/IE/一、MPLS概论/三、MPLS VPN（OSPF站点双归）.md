对于双归场景下，不同协议的处理
 
OSPF站点为了加强冗余的功能，部署多台PE，该场景下会存在问题（即OSPF和MPLS VPN（BGP），执行了双点双向）

在MPLS VPN网络中站点OSPF双规部署  
单向引入后，没有出现路由回馈和路由次优的问题  
AR2将BGP的路由引入到OSPF后￼￼前提概要：双规部署，因为缺省情况下，如果站点都是用OSPF，那么他们的do-main id 值都是0.0.0.0，￼ 导致：在区域中通告的路由条目都是，3类LSA，如果想在本区域中通告的是5类LSA，可以通告修改do-main id 值。  
其次：缺省引入外部路由也会还是5类LSA  
￼第一种情况： do-main id 相同：引入的是3类LSA  
AR3收到OSPF的3类LSA不会进行路由的计算  
因为3类LSA携带的DN bit置位，AR3收到DN bit置位的LSA不会计算路由信息

![Exported image](Exported%20image%2020251206151418-0.png)

DN bit的作用：  
绑定了VPN实例的OSPF设备，不会计算DN bit置位的LSA  
DN bit置位的条件：  
实例路由在OSPF中引入import
 
对于DN bit如果是跨越区域传递的LSA，则DN bit自动清除  
如果DN bit不置位，那么AR3是可以正常计算3类LSA的路由信息  
[AR2-ospf-1]dn-bit-set disable summary 在引入实例路由的PE设备上，配置命令后 不会将3LSA设置DN bit  
[AR3-ospf-1]dn-bit-check disable summary 在建立实例OSPF的PE设备上，配置命令后 忽略3类LSA的DN bit
 
第二种情况：两端的do-main id值不相同，引入的是5类LSA  
如何让它引入的路由是5类LSA，修改do-main ID

![Exported image](Exported%20image%2020251206151422-1.png) ![Exported image](Exported%20image%2020251206151423-2.png) ![Exported image](Exported%20image%2020251206151425-3.png)  

如果存在5类LSA，5类LSA也具备DN bit和3类LSA一样存在DN bit防环的功能  
但是5类LSA还具备route tag防环  
route tag是PE设备上通过bgp的as编号自动计算所得，如果as编号相同，则计算的值就相等  
如果收到5类LSA携带的route tag和自身计算的route tag 相同，则不会计算该路由条目

![Exported image](Exported%20image%2020251206151427-4.png)

[AR3-ospf-1]route-tag 123 该命令修改ospf的route tag值  
[AR3-ospf-1]vpn-instance-capability simple 在建立实例OSPF的PE设备上，配置命令后 忽略DN bit的检查 和 route tag的检查  
# 一定不要做￼

![Exported image](Exported%20image%2020251206151429-5.png)

MCE：多CE场景  
在一个站点内存在多种业务，每一种业务都需要业务隔离  
PE设备不能很好隔离内部业务网络，所以需要CE设备来执行隔离的操作
 
对于MCE设备而言，从PE学习到对端的路由信息会存在dn bit置位  
而MCE默认不会计算dn bit置位的LSA  
[AR3-ospf-1]vpn-instance-capability simple 在建立实例OSPF的PE设备上，配置命令后 忽略DN bit的检查 和 route tag的检查