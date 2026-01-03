# 网络现状：
![](assets/SRv6/file-20260103230435626.png)
# 从MPLS到SRv6
MPLS带来了网络孤岛问题。SRv6统一转发平面，同时拥有简化协议、高扩展性和可编程等优势。
![](assets/SRv6/file-20260103230741125.png)
## SRv6原理简介
lSRv6在头节点上对数据压入段路由扩展报文头（SRH，Segment Routing Header）来指导数据转发。
lSRv6报文没有改变原有IPv6报文的封装结构，SRv6报文仍旧是IPv6报文，普通的IPv6设备也可以识别，所以说SRv6是Native IPv6技术。
lSRv6 的Native IPv6特质使得SRv6设备能够和普通IPv6设备共同组网，对现有网络具有更好的兼容性。

## IPv6 SRH介绍
![](assets/SRv6/file-20260103230932986.png)

## SRv6 Segment介绍
SRv6 Segment是IPv6地址形式，通常也可以称为SRv6 SID（Segment Identifier）。
如图所示，SRv6 SID由Locator、Function和Arguments三部分组成，格式为Locator:Function:Arguments。注意Length(L+F+A) <= 128。当长度和小于128时，保留位用0补齐。
如果没有Arguments字段，格式则是Locator:Function。Locator占据IPv6地址的高比特位，Function部分占据IPv6地址的剩余部分。
![](assets/SRv6/file-20260103231002804.png)



根据目标地址，匹配的网络号进行转发。