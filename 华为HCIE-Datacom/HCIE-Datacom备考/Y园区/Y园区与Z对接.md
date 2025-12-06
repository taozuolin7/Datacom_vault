1.在SD-WAN的互联配置中：  
在Y_OA_To_Sites的LAN-WAN互联的时候，除了对接CORE，可以配置对接Z的OA的接口：  
Y_RD_To_Sites 也如此  
G0/0/7-valn10和G0/0/6-vlan20都选择“三层子接口”  
10.20.2.1/30 10.20.2.9/30
 ![Exported image](Exported%20image%2020251206141523-0.png)  

创完接口后，还需创建BGP会话  
10.20.2.2 - 65003

![Exported image](Exported%20image%2020251206141525-1.png)  

10.20.2.10/30 65003

![Exported image](Exported%20image%2020251206141527-2.png)