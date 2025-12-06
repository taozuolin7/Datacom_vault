**isatap隧道：**  
终端获取路由器前缀（SLAAC），自动生成接口标识  
6to4隧道是将DIPv4地址封装在网络前缀中  
istap隧道是将DIPv4地址封装在接口标识中  
要求：地址格式 前缀自动获取，接口标识根据规则生成  
接口标识 64bit，前32bit固定，第7bit根据ipv4地址来进行判断  
如果IPv4地址是全局唯一的，第7bit置1， 0200:5efe  
如果IPv4地址是非全局唯一的，第7bit置0， 0000:5efe

**用管理员打开****cmd****：**

```
￼C:\Windows\system32\>netsh       
netsh\>interface isatap
netsh interface isatap\>set router 2.2.2.2    
```

设置地址为`isatap`服务端的地址  
确定。  

```
netsh interface isatap\>show state
ISATAP 
```

状态

```
     : default
netsh interface isatap\>set state enable      
```

开启`isatap`的接口  
确定。  

```
netsh interface isatap\>show state
ISATAP 
```

状态

```
     : enabled
netsh interface isatap\>exit
```

R2

```
￼#
interface Tunnel0/0/1
 ipv6 enable 
 ipv6 address 2022::5EFE:0202:0202/64 
 tunnel-protocol ipv6-ipv4 isatap
 source 2.2.2.2
 undo ipv6 nd ra halt
```

**查看建立好后的****isatap****隧道的****ip****地址：**

```
￼C:\Windows\system32\>ipconfig
Windows IP 
```

配置  
以太网适配器

```
 Ethernet0:
   
```

连接特定的 `DNS` 后缀

```
 . . . . . . . :
   
```

本地链接 `IPv6` 地址

```
. . . . . . . . : fe80::6f:12aa:99f3:a9c6%7
   IPv4 
```

地址

```
 . . . . . . . . . . . . : 192.168.120.200
   
```

子网掩码

```
  . . . . . . . . . . . . : 255.255.255.0
   
```

默认网关`. . . . . . . . . . . . . : 192.168.120.254`
 
隧道适配器 `isatap.{422C5744-F8D8-4C0F-B888-9E22AC5A063C}:`
 
连接特定的 `DNS` 后缀

```
 . . . . . . . :
   IPv6 
```

地址

```
 . . . . . . . . . . . . : 2022::5efe:192.168.120.200
   
```

本地链接 `IPv6` 地址

```
. . . . . . . . : fe80::5efe:192.168.120.200%4
   
```

默认网关`. . . . . . . . . . . . . : fe80::5efe:2.2.2.2%4`

![Exported image](Exported%20image%2020251206150236-0.png)