段路由SR（Segment Routing）是基于源路由理念而设计的在网络上转发数据包的一种技术架构

LDP与IGP同步问题
RSVP-TE配置繁琐，带宽资源预留后无法被其他业务使用

网络架构：
1.现有网络都是基于SPF计算最优的路径作为网络架构
   1.所有业务数据都会走最优的路径（容易造成网络拥塞）
   2.如果存在标签访问、隧道访问，无法进行在业务之间进行精细的区分

2.SR网络是基于业务来定义网络的架构
   1.不存在唯一最优的路径，而是根据业务的特点进行合适路径的分配
   2.IGP的SPF计算最优路径的方法不存在了

SR的概述：
	段路由 将原本计算最优路径，变为了为业务分配合适路径
	段路由 是软功能，添加在IGP/BGP中，以此让IGP/BGP可以自动分配标签（不需要存在LDP）
	设备 可以通过在线升级等方式来支持SR的功能
	段路由 引入源路由机制（源设备会执行标签的封装，中间设备会执行标签的转发）

网络的组成：
 路由器+路由器连接的接口+互联的链路

SR的构想：
 将网络的每一个节点，每一个链路 都理解为一个个的片段，称为segment id
 根据目标 来组合 不同的段ID（只有源设备进行组合，组合为段列表，中间设备只是识别段ID进行转发）

SR的网络编程：
1.不是进行所谓的代码编写，根据节点、链路生成的标识ID
  在将标识ID进行有规律的排列，让数据具有业务上的转发意义
2.如果设备访问的节点ID，则根据节点ID依然会进行最短路径计算（可能存在负载分担）

**分段路由：**
1.SR-MPLS三种类别的segment id
  1.adj-sid：邻接段ID
     1.手工配置 或 自动生成
        [NE1-segment-routing]ipv4  adjacency local-ip-addr 10.1.12.1 remote-ip-addr 10.1.12.2 sid 322102
     2.标识网络中两台设备互联的网段
     3.通过IGP扩散，全局可见且全局有效
     4.邻居SID的范围是 除SRGB空间以外的值

  2.prefix-sid：前缀段ID
     1.手工配置
     2.标识网络中一个网段的描述（末节网段）
     3.通过IGP扩散，全局可见且全局有效
     4.前缀SID的范围是 SRGB
     *Node sid：节点段ID
       *就是前缀段ID的一种
       1.手工配置
       2.用于标识设备的环回口接口
         在环回口上配置前缀SID就是在配置节点SID

2.SRGB（Segment Routing Global Block）：用户指定的为Segment Routing MPLS预留的全局标签集合
```
[NE1]segment-routing                       全局开启SR功能
[NE1]ospf 1
[NE1-ospf-1]opaque-capability enable       支持不透明LSA的功能
[NE1-ospf-1]segment-routing mpls           支持转发标签的能力
[NE1-ospf-1]segment-routing global-block 16000 18000   配置SRGB的空间范围

[NE1-LoopBack0]ospf prefix-sid             环回口下设置node sid的值
    index <0-65534>     配置接口指定的值，发送给其他设备，其他设备会自行计算
                       （将该值加上自身SRGB的起始值，且得到的和在自身SRGB的范围内，否则认为该值错误）
    absolute <16000-321535, 524288-1048575>    设置固定的值，该值要求在自身SRGB的范围内
                                               通过LSA发送时，携带的值为 设置的值 减去 SRGB的起始值
```

```
OSPF对于实现SR MPLS扩展的支持，引入type 10LSA
类型10LSA存在三种对应的LSA

类型10LSA - 4.0.0.0
[NE1]dis ospf lsdb opaque-area 4.0.0.0 

          OSPF Process 1 with Router ID 10.1.1.1
                          Area: 0.0.0.0
                  Link State Database


  Type      : Opq-Area
  Ls id     : 4.0.0.0
  Adv rtr   : 10.1.1.1
  Ls age    : 258
  Len       : 44
  Options   :  E
  seq#      : 80000002
  chksum    : 0xd6a3
  Opaque Type: 4
  Opaque Id: 0
  Router-Information LSA TLV information:
    SR-Algorithm TLV:
      Algorithm: SPF          携带自身的算法SPF
    SID/Label Range TLV:
      Range Size: 2001        SRGB的取值数量
      SID/Label Sub-TLV:
        Label: 16000          SRGB的起始值


类型10LSA - 8.0.0.0
[NE1-ospf-1]dis ospf lsdb opaque-area 8.0.0.0

          OSPF Process 1 with Router ID 10.1.1.1
                          Area: 0.0.0.0
                  Link State Database


  Type      : Opq-Area
  Ls id     : 8.0.0.0
  Adv rtr   : 10.1.1.1
  Ls age    : 646
  Len       : 48
  Options   :  E
  seq#      : 80000001
  chksum    : 0x7da9
  Opaque Type: 8
  Opaque Id: 0
  OSPFv2 Extended Link Opaque LSA TLV information:
    OSPFv2 Extended Link TLV:   拓扑信息，描述广播网络的连接
      Link Type: TransNet
      Link ID: 10.1.12.2 
      Link Data: 10.1.12.1        
      Adj-SID Sub-TLV:          对于邻接的段ID描述
        Flags: 0x60 (-|V|L|-|-|-|-|-)
        MT ID: 0 
        Weight: 0
        Label: 48020  
```