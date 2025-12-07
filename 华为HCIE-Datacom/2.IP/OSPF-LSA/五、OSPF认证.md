```
OSPF认证：  
*接口认证和区域认证在最终的结果上没有任何区别，唯一的区别是认证的影响范围  
1.接口认证：认证的功能只作用于认证的接口  
    [AR4-GigabitEthernet0/0/0]ospf authentication-mode simple plain huawei 
```

![Exported image](Exported%20image%2020251206164257-0.png)  

```
2.区域认证：认证的功能会影响所有在该区域宣告的接口  
    [AR3]ospf  
    [AR3-ospf-1]area 1   
    [AR3-ospf-1-area-0.0.0.1]authentication-mode simple plain huawei
```
 
```
    [AR3-GigabitEthernet0/0/0]ospf authentication-mode null   
*接口认证优于区域认证
```
 
```
*如果存在虚链路，则需要考虑是否配置了区域0的区域认证  
 如果区域0 存在区域认证，则虚链路也需要配置认证  
 [AR2-ospf-1-area-0.0.0.1]vlink-peer 10.4.4.4 md5 1 plain Huawei@123
```
 
```
特殊区域参与认证：
```

![Exported image](Exported%20image%2020251206164300-1.png)