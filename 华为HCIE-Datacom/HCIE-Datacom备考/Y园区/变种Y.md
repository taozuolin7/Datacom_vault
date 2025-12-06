1.零配置开局-设置WAN链路的时候需要添加接口：Y_Export1和Store_Export1都要添加（G0/0/9和G0/0/10）

![Exported image](Exported%20image%2020251206141722-0.png)  
![Exported image](Exported%20image%2020251206141723-1.png)   
2.设备的角色类型不同  
VXlan Fabric网络-添加设备
 
变种y 接入支持vxlan/汇聚不支持vxlan  
核心=边界网关+路由反射器=所有设备网关所在  
汇聚=透传节点=不处理vxlan报文的设备  
接入=边缘节点=vxlan边缘设备

![Exported image](Exported%20image%2020251206141726-2.png)   
3.接口绑定认证模板的时候有不同  
变种Y没有策略联动：所以不需要填VLAN以及IP，  
并且所有接口的 对端连接类型 全是非VXLAN设备  
认证方式全是 8021.x_MAC

![Exported image](Exported%20image%2020251206141730-3.png)  
![Exported image](Exported%20image%2020251206141735-4.png)  

4.VXlan虚拟网络-策略控制  
创建策略矩阵的时候，选择设备： 两台ACC1和CORE

![Exported image](Exported%20image%2020251206141739-5.png)