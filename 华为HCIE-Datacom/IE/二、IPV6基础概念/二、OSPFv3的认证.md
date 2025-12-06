OSPFv3的认证：  
1.OSPFv3的区域认证、接口认证，报文内直接携带认证的信息  
2.利用IPsec的esp协议通过IPv6扩展报头对整个OSPF报文进行加密和认证  
3.利用IPsec的ah协议通过ipv6扩展报头添加认证协议，ospfv3的明文传递的

`NE1`： `ah`认证  

```
#
ipsec proposal test
 encapsulation-mode transport
 transform ah
#
ipsec sa test
 proposal test
 sa spi inbound ah 12345
 sa string-key inbound ah cipher Huawei@123
 sa spi outbound ah 54321
 sa string-key outbound ah cipher Huawei@123
#
ospfv3 1
 router-id 10.1.1.1
 ipsec sa test
```

![Exported image](Exported%20image%2020251206150202-0.png)

`NE2`：  

```
#
ipsec proposal test
 encapsulation-mode transport
 transform ah
#
ipsec sa test
 proposal test
 sa spi inbound ah 54321
 sa string-key inbound ah cipher Huawei@123
 sa spi outbound ah 12345
 sa string-key outbound ah cipher Huawei@123
#
ospfv3 1
 router-id 10.2.2.2
 ipsec sa test
```

`NE1`： `esp`认证  

```
#
ipsec proposal test
 encapsulation-mode transport
 transform esp
#
ipsec sa test
 proposal test
 sa spi inbound esp 12345
 sa string-key inbound esp cipher Huawei@123
 sa spi outbound esp 54321
 sa string-key outbound esp cipher Huawei@123
#
ospfv3 1
 router-id 10.1.1.1
 ipsec sa test
```

`NE2`：  

```
#
ipsec proposal test
 encapsulation-mode transport
 transform esp
#
ipsec sa test
 proposal test
 sa spi inbound esp 54321
 sa string-key inbound esp cipher Huawei@123
 sa spi outbound esp 12345
 sa string-key outbound esp cipher Huawei@123
#
ospfv3 1
 router-id 10.2.2.2
 ipsec sa test
```