一、设备上线节点：  
1.在配置完成缺省后，且真机上部署完：  
agile controller host 172.21.16.198 port 10020  
此时两台出口上线（Y_Export1,Store_Export1）  
2.创建managementVN后，并且配置缺省回去 （目的网段：10.2.0.0/24 ，下一跳：10.2.1.2）  
然后配置本地上网，打开nat（顺别配置NAT，把portal服务器映射出去）  
此时除了ap以外都上线  
3.下游设备切换3996vlan上线，创建子网（10.2.0.0/24）记得在ACC1上将22端口设为trunk，放行3996  
然后在核心上，配置capwap source interface vlanif 3996  
y  
此时所有设备上线----------------------
   

二、在配置VN互通的时，源节点选择OA1，目的节点选择RD2  
源ipv4前缀：全选， 目的ipv4前缀：10.1.11-15.0/24
 
三、在准入认证模块：记得在“配置-站点配置-Site_Y-交换机-认证-无线认证”  
配置Employee与Guest的认证模板，SSID需要与题目给的一致
 
四、在部署“试点门店规划”时  
1.在配置Guest_To_Internet的Site_Y站点时：在LAN-WAN互联路由配置时：Site_Y站点需要写10条静态路由回包：￼目的网段：10.2.101.0/24 下一跳：10.2.200.9  
……  
目的网段：10.2.110.0/24 下一跳：10.2.200.9

![Exported image](Exported%20image%2020251206141715-0.png)  

五、流量的优化  
在SD-WAN部分  
1.在核心交换机上针对GuestVN，部署一条静态路由，避免Guest通过缺省访问OA业务路由：  
ip route-static vpn-instance vpn4 10.100.2.0 24 null 0
 
2.在“Y_OA_To_Site" LAN业务规划时：  
WAN路由部分：针对Site_Y,Site_Store分别对发布的路由做限制

|   |   |
|---|---|
|Site_Y：|只发布一条缺省|
|Site_Store:|10.100.2.0 24|
 
3.在“Y_RD_To_Site”  
WAN路由部分：

|   |   |
|---|---|
|Site_Y:|发布所有RD路由：10.2.11-15.0/24，10.2.21-25.0/24，10.3.99.0/24 10.3.100.0/24|
|Site_Store|10.100.3.0/24|
   

六、最后的上Internet部分  
1.“Y_OA_To_Site"，集中上网和本地上网（两个接口）  
2.“Y_RD_To_Site”，不上网  
3.“Guest_To_Internet" ，本地上网，Internet优先（四个接口）
       
测试发现：  
虚拟Y  
1.继承策略模板不能弄  
2.VN互通不能用
         

1.考场不需要创建租户，已经创建好了的，直接在登录见面登录就行
 
2.设置的密码为：在被纳管的真机上登录的密码

![Exported image](Exported%20image%2020251206141717-1.png)   
3.南北向地址：  
1.北向地址：172.21.16.199用户访问控制器web界面的地址（对用户而言）  
2.南向地址：172.21.21.198设备被控制器纳管的地址（对设备而言）
 
交换机上不了线  
1.配置残留  
undo配置  
2.数据库残留  
[全局配置]reset netconf db-configuration
   
![Exported image](Exported%20image%2020251206141718-2.png)