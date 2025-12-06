**NAT64：IPv4和IPv6的互通技术**  
通过提前定义需要转换的ipv6网络前缀  
设备收到定义的前缀值，执行转换操作  
设备收到非定义的前缀值，执行正常ipv6的转发操作
 
对于定义的IPv6网络前缀：  
1.知名定义：64:FF9B::/96  
2.自定义：

![Exported image](Exported%20image%2020251206150241-0.png)

AR1在访问时是通过IPv6访问的
 
AR2是IPv4的地址10.1.12.2
 
AR1在访问AR2时，会认为AR2是IPv6的地址 2002::0a01:0c02/96
 
该IPv6地址是将 NAT64的前缀 + IPv4地址组合而成
 
FW收到DIPv6地址为2002::a01:c02的数据，就会执行NAT64转换的操作  
转换为DIPv4地址10.1.12.2  
转换后的SIPv4地址是 1.1.1.1（静态NAT64规划的地址）

**动态****NAT64**

```
￼FW
```

指定`NAT64`转换的网络前缀  
`[FW]nat64 prefix 2002:: 96`
 
接口下开启`NAT64`的转换功能  
`[FW-GigabitEthernet1/0/0]nat64 enable`
 
为`IPv6`地址动态规划转换为的`IPv4`地址  
`#ipv4`地址池  

```
nat address-group 1 0
 mode pat
 section 0 1.1.1.1 1.1.1.100
#
nat-policy
 rule name NAT64
  nat-type nat64 #
```

如果不加，它默认是nat  
`action source-nat address-group 1`
 
`FW`需要进行安全策略的放行  

```
#
security-policy
 rule name 1
  source-zone trust
  source-zone untrust
  destination-zone trust
  destination-zone untrust
  action permit
#
```

**静态****NAT64****：**

```
￼FW
```

指定`NAT64`转换的网络前缀  
`[FW]nat64 prefix 2002:: 96`
 
接口下开启`NAT64`的转换功能  
`[FW-GigabitEthernet1/0/0]nat64 enable`
 
为`IPv6`地址静态规划转换为的`IPv4`地址  
`[FW]nat64 static 2001::1 1.1.1.1 unr-route`
 
`FW`需要进行安全策略的放行  

```
#
security-policy
 rule name 1
  source-zone trust
  source-zone untrust
  destination-zone trust
  destination-zone untrust
  action permit
#
```

![Exported image](Exported%20image%2020251206150242-1.png) ![Exported image](Exported%20image%2020251206150247-2.png)

Web方式配置NAT64￼￼1.先开启IPv6以及NAT64功能。

![Exported image](Exported%20image%2020251206150249-3.png)

2.配置：配置前缀，开启NAT64功能￼

![Exported image](Exported%20image%2020251206150250-4.png)

3.配置NAT策略￼

![Exported image](Exported%20image%2020251206150252-5.png)