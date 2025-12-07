```
VRP5 适用于中低端设备，可以直接执行命令  
VRP8 适用于高端设备，对于命令存在候选列表  
display this 显示当前视图下的所有配置信息  
             如果视图下默认执行的命令，是不显示的
```

```
设备管理：  
1.CTL管理：命令行管理  
   1.带外管理，不需要网络存在  
       console线管理设备  
       要求管理员和设备面对面  
   2.带内管理  
       通过远程登录的方式管理设备  
       1.telnet  
       2.SSH  
2.web管理：页面管理  
   1.通过图形化的页面，更加直接的管理设备  
   2.该方式管理设备的能力没有命令行全面
```

```
登录防火墙：  
Username:admin  
Password:Admin@123  
The password needs to be changed. Change now? [Y/N]: y  
Please enter old password: Admin@123  
Please enter new password: Huawei@123  
Please confirm new password: Huawei@123
```
 
```
针对防火墙配置：  
\<USG6000V1\>system-view    进入系统试图  
[USG6000V1]interface GigabitEthernet 0/0/0  进入接口试图  
[USG6000V1-GigabitEthernet0/0/0]ip address 192.168.123.2 24  配置接口IP地址及掩码  
[USG6000V1-GigabitEthernet0/0/0]service-manage ping permit   允许ping功能的使用  
[USG6000V1-GigabitEthernet0/0/0]service-manage https permit  允许https功能的使用
```

```
[AR1]telnet client-source -a x.x.x.x   
     指定自身发起telnet连接时的SIP
```

```
[AR1]telnet client-source -i gi 0/0/0  
      指定哪个什么接口的IP作为telnet发起连接的SIP
```

```
使用telnet远程登录：  
\<AR1\>telnet 192.168.1.2  
  Press CTRL_] to quit telnet mode  
  Trying 192.168.1.2 ...  
  Connected to 192.168.1.2 ...  
Login authentication
```

```
配置Telnet：  
\<Huawei\>system-view   
[Huawei]sysname AR2  
[AR2]interface GigabitEthernet 0/0/0  
[AR2-GigabitEthe0/0/0]ip address 192.168.1.2 24   
[AR2]telnet server enable    # 开始telnet服务  
[AR2]user-interface vty 0 4    
#开启远程连接的虚拟接口5个  
[AR2-ui-vty0-4]authentication-mode password       
#设置登录模式为密码登录  
Please configure the login password   
(maximum length 16):Huawei@123  # 设置登录密码  
[AR2-ui-vty0-4]user privilege level 3    
#设置用户的权限级别为3  
[AR2-ui-vty0-4]protocol inbound telnet   
#设置用户远程登录的协议  
[AR2]telnet server port 12345     
#设置telnet服务的端口号为12345  
[AR2-ui-console0]idle-timeout 0 0    
登录设备将超时时间设置为空
```
 
```
[AR2]user-interface vty 0 4   
[AR2-ui-vty0-4]authentication-mode aaa  
[AR2-ui-vty0-4]protocol inbound all
```
 
```
[AR2]aaa   进入3a试图  
[AR2-aaa]local-user a1 password cipher Huawei@123  
#创建用户及密码  
[AR2-aaa]local-user a1 privilege level 3       
#设置用户权限级别  
[AR2-aaa]local-user a1 service-type telnet     
#设置用户的服务协议类型
```
 
```
[AR2-aaa]local-user b2 password cipher Huawei@123  
[AR2-aaa]local-user b2 privilege level 0  
[AR2-aaa]local-user b2 service-type telnet 
```
 
```
[AR2]display users  查看登录的用户
```
 
```
telnet：  
  是一种C/S架构  
  1.密码登录  
     所有登录用户权限都是相同的  
  2.用户密码登录  
     通过创建不同的用户分配不同的权限
```

```
\<AR1\>telnet 192.168.1.2 12345  #登的接口id  
  Press CTRL_] to quit telnet mode  
  Trying 192.168.1.2 ...  
  Connected to 192.168.1.2 ...
```
 
```
Login authentication  
Username:b2  
Password:
```

```
SSH：  
  是一种C/S的架构  
  1.用户密码登录
```

```
[AR1]stelnet 192.168.1.2   
Please input the username:a1  
Trying 192.168.1.2 ...  
Press CTRL+K to abort  
Connected to 192.168.1.2 ...  
The server is not authenticated. Continue to access it? (y/n)[n]:y  
[AR1]  
Save the server's public key? (y/n)[n]:y  
The server's public key will be saved with the name 192.168.1.2. Please wait...  
[AR1]  
Enter password:
```

```
[AR1]ssh client first-time enable   
     第一次登录需要先获取公钥
```
 
```
[AR2]stelnet server enable   #开启SSH服务  
[AR2]ssh user a1 authentication-type all     
#指定ssh用户  
[AR2]user-interface vty 0 4   
[AR2-ui-vty0-4]authentication-mode aaa  
[AR2-ui-vty0-4]protocol inbound all  
[AR2]aaa     
#进入3a试图  
[AR2-aaa]local-user a1 password cipher Huawei@123   
#创建用户及密码  
[AR2-aaa]local-user a1 privilege level 3       
#设置用户权限级别  
[AR2-aaa]local-user a1 service-type ssh     
#设置用户的服务协议类型  
[AR2-aaa]local-user b2 password cipher Huawei@123  
[AR2-aaa]local-user b2 privilege level 0  
[AR2-aaa]local-user b2 service-type ssh 
```
 
```
[USG6000V1]ssh user a1 service-type stelnet   
#模拟器中AR路由器设备不支持该命令
```
 
```
[AR2]rsa local-key-pair create       
#服务器生成公私密钥对  
The key name will be: Host  
% RSA keys defined for Host already exist.  
Confirm to replace them? (y/n)[n]:y  
The range of public key size is (512 ~ 2048).  
NOTES: If the key modulus is greater than 512,  
       It will take a few minutes.  
Input the bits in the modulus[default = 512]:  
Generating keys...  
..............++++++++++++  
......++++++++++++  
..................++++++++  
...................................++++++++
```

![Exported image](Exported%20image%2020251206145806-0.png)