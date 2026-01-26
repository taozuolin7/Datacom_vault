EVN 跨数据中心大二层技术，华为私有协议。现在跨数据中心2层互访，没有3层互访的能力。

华为基于EVPN协议开发出的 私有 二层VPN技术。用于和友商对标协议。目前华为不在对EVN做支持。

DCI

数据中心互联网络分类

在数据大集中的背景下，企业产生的数据量越来越大，数据的重要性也越来越高。出于灾备、用户就近接入、提升资源利用率等方面的考虑，企业可能在不同物理地点建立多个数据中心站点。最常见是企业两地三中心的数据中心站点设置：一个主数据中心、一个同城数据中心、一个异地数据中心。一般情况下，多个数据中心之间的互联

网络包括图所示的三种。

![0](https://share.note.youdao.com/yws/public/resource/fefec1c1fb8643c9ae9a59fbd4922ea9/xmlnote/5376A06EAF66401697148C8EF74174E2/46294)

@三层 MPLS VPN/IP 互联，也称为数据中心前端网络互联，主要实现数据共享、文件复制、邮件访问等应用层级别的互访。一般情况下，企业都会实现数据中心间的三层互联。

@存储互联，也称为 SAN 互联，主要实现数据中心存储网络中磁盘阵列的数据同异步复制,存储的双活。

@二层互联，也称为数据中心服务器网络互联，主要实现跨站点服务器集群或虚拟机动态迁移等业务的需要。

互联网络实现方案

根据选择的互联网络不同，可以选择如图 2-52 所示的裸光纤、DWDM、MSTP、VPN等不同的技术实现方案。

![0](https://share.note.youdao.com/yws/public/resource/fefec1c1fb8643c9ae9a59fbd4922ea9/xmlnote/0D9A37E4A0794FE38EE276E7EA74DA8A/46295)

从图 2-52 我们可以看到这几种方案都可以用于数据中心二层网络互联，但是在实际应用中，这些方案在应用场景、成本投入等方面都存在的不足。这些不足限制了其实际

应用的范围。接下来我们就来看看这些实现方式存在哪些不足？

裸光纤/DWDM/MSTP 二层互联之所以将这几个方案放在一起讲，因为本质上来说，它们都是通过物理连线延伸的方式实现二层网络的扩展，安全性比较高，而且带宽也比较高。

图 2-53 是一个通过DWDM 实现二层互联的示例，在二三层分界的核心交换机上通过 LAG 链路接入DWDM 网络。

![0](https://share.note.youdao.com/yws/public/resource/fefec1c1fb8643c9ae9a59fbd4922ea9/xmlnote/9F9A56C2DFF34288A3530D0A18040C73/46296)

这类实现方式的传输距离相对较短，因此一般适用于同城数据中心的互联；而且从部署成本来看，带宽越宽越贵，距离越远越贵。

以裸光纤为例，点到点的施工成本大概在 1.2 万元/公里。如果是运营商的裸光纤线路，最终的租售价大约是成本价的 1.3 倍，这还不包括后续的维护费用。成本成了制约该类

方案应用的关键因素，当前主要由运营商，大型互联网企业或者国有企业等不差钱的企业使用。除此之外，这类方案实现二层网络扩展的同时，实际上也大大扩展了二层网络中环路、广播风暴的影响范围。而其解决方案跟一般二层网络一样，通过引入破环协议、虚拟化技术、链路聚合等实现。二层网络变大后，这些技术将涉及到跨数据中心的部署，对网络维护人员的能力要求将变得更高。

MPLS VPN 二层互联

这种实现方式中数据中心之间通过 MPLS 网络实现三层互联，在此基础上部署 VPLS（点到多点）/VLL（点到点）完成数据中心的二层互联。

如果说裸光纤等互联方式是土豪玩的方式， 那么这种方式从成本上来说接则接地气了一些（以租用运营商提供的 100M VPN 专线为例，费用大概是 2 万/月），最主要的是

还能实现长距离的数据中心互联。图 2-54 所示的是一个典型的 VPLS 实现数据中心二层互联的组网方案。

![0](https://share.note.youdao.com/yws/public/resource/fefec1c1fb8643c9ae9a59fbd4922ea9/xmlnote/EE2FFD81B94A4F8C8848A911ACB3BE2A/46297)

由于 VPLS 技术的标准化，该方案一度是使用最广泛的数据中心二层互联方案。然而，随着数据中心内服务器数量的大量增加以及虚拟化技术的应用（VM 的产生使得二层

域内 MAC 的数量显著增加）。VPLS 互联所带来的问题也逐渐变得越来越突出：

@ 互联网络中的设备必须全部运行 MPLS，而且需要实现所有站点间互联设备的全连接配置。初始部署复杂是一方面，后续的维护也相对复杂，甚至不具备可行性：

一旦需要扩展新的数据中心，或者由于数据中心内二层网络规模的增加，需要新增或者变更 PE 设备，就必须得在所有站点的 PE 设备上挨个刷新一遍配置，以建立新的全连接关系。

@ 跟普通二层网络一样，互联后的不同数据中心站点间，依靠数据面的流量泛洪（ARP 广播报文）扩散 MAC。数据中心互联后，大量广播报文在数据中心间扩散，浪费了互联链路带宽的同时，也造成了站点 PE 的性能瓶颈（每一个 PE 都必须学习所有站点的 MAC）。

@ VPLS 网络为了避免骨干网侧的环路引入了水平分割，但是对于 CE 多归属接入时，可能引入的端到端的环路问题，VPLS 本身没有解决方案。因此 VPLS 实现二层网

络互联时，不能直接支持 CE 的多归属接入。为了弥补 VPLS 的不足，业界在 VPLS 基础上推出了各种增强方案，例如集群+VPLS、PBB-VPLS、MC-LAG+VPLS 等一系列方案。这些方案在解决了 VPLS 部分问题的同时，也带来了新的问题，那就是配置和维护变得更加复杂。上面我们都是从技术的维度去分析 VPLS 方案的不足。实际上在企业选择方案时，除了技术满足度以外，还有一点不能不考虑，那就是成本。

我们知道企业出于成本考虑一般采用向运营商租赁带宽的方式实现互联网络。而 VPLS方案由于采用站点间广播的方式学习 MAC，这种方式将导致宝贵的互联带宽被不必要

的浪费。另一方面，VPLS 本身不支持多归属接入，意味着即使出于可靠性需要，存在CE-PE 间的备份链路，也无法直接通过 VPLS 实现流量的负载分担，当然可以选择使

用更复杂的技术解决，例如 STP Over VPLS、Smart-link + VPLS 等，而这又会引入下

面的问题。

VPLS 组网方案本身技术比较复杂，部署及运维管理难度较大，对人员的能力要求比较高。而在 VPLS 基础上的增强方案，更是进一步提高了对人员的能力要求。这就意味

着企业要付出更多的成本在员工的能力培养或者服务外包上。如果说运营商不 care 成本，出于技术成熟度的考虑，没有更大的动力对 VPLS 方案做大的变革。那么对企业来说，对新技术的诞生则有了更多的期待。在推出新技术方面，Cisco、Juniper、Alcatel-Lucent 等一往无前的冲在了前面。2012 年Cisco、Juniper、AT&T、Alcatel-Lucent 等一帮小伙伴们一起提出了 EVPN 草案（draft-ietf-l2vpn-evpn-00）。

基于 IP 网络实现二层互联

该类方案只要求数据中心之间 IP 网络可达即可，在此基础上通过建立隧道来实现数据中心的二层互联。如果说随着 2015 年 2 月 EVPN 成为正式的 RFC7432，EVN 已经有了

相对统一的执行标准，那么基于 IP 网络的二层互联方案，目前为止则还是群雄逐鹿的状态。

目前已经发布的基于 IP 网络的二层互联方案主要包括 Cisco 的 OTV、H3C 的 EVI、HUAWEI 的 EVN（Ethernet Virtual Network）。其中 OTV 和 EVI 方案均使用扩展的

ISIS 来实现 MAC 地址的发布（有没有想到 TRILL？），公网的隧道采用 GRE；而HUAWEI 的 EVN 方案使用扩展的 BGP 来发布 MAC 地址（又想到了 EVPN？），公网

的隧道采用 VXLAN 的封装方式。到这里我们终于看到了主角 EVN 的粉墨登场，提前预告一下，EVN 不依赖 MPLS、配置简单，且在环路避免、多归属支持、广播风暴抑制等方面均有很好的解决方案，在接下来的篇幅中我们将围绕 EVN 的具体实现来展开。

抽丝剥茧看实现

上一章我们介绍了 EVN 出现的背景，这一篇我们重点介绍 EVN 的具体实现。所谓知其然，更要知其所以然，我们不仅要告诉你 EVN 行，还要告诉你，为什么它行。

这一篇的内容会有点多，还请各位看官耐心。为了防止一下子上太多导致大家消化不良，我们先大概了解一下 EVN 的技术要点。

EVN 是一种基于 BGP 和 VXLAN 的数据中心二层网络互联解决方案，目前只有CloudEngine 系列交换机支持。该方案通过扩展 BGP 协议在 PE 之间传递二层网络的

MAC 地址信息，完成控制面表项的建立；通过 VXLAN 的“MAC in UDP”方式对数据报文进行封装，完成转发面报文的传输。

提到使用 BGP 来传递 MAC 信息，仔细看前一篇的小伙伴们一定有疑惑，EVPN 不也是这么做的么？事实上，EVN 借鉴了 EVPN 中关于使用 BGP 来传递 MAC 信息的实现

思路，并且在 EVPN 基础上实现了突破，主要表现在以下几点：

@ 引入 BGP RR，避免了 PE 之间全连接的复杂配置。而且 PE 间 BGP 邻居关系建立后，VXLAN 隧道自动建立，不需要额外的隧道配置。这两点使得 EVN 的配置复

杂度大大减少（基本功能只需要 6 条命令）。

@ 使用 VXLAN 隧道代替了 MPLS 隧道，降低了企业对运营商 MPLS 网络的依赖。

@ 支持端到端的基于流的负载分担。接入侧支持 CE 的多归属接入，且支持基于VLAN+接口进行逐流负载分担；互联网络侧数据报文采用“MAC in UDP”封装，

UDP 端口号通过 HASH 随机生成，因此封装后的报文在互联网络上传输时，可以基于 UDP 端口的逐流负载分担。

@ 用户可以自主选择是否将未知单播报文、ARP 广播报文泛洪到远端 PE。关闭这两类报文的跨站点泛洪可以显著的减少对互联带宽的占用，以及站点内广播报文对

其他站点可能的影响。知易行难，真要想抽丝剥茧可不是这么三两句就完事了的。接下来的内容我们将分EVN 组网模型、控制平面工作原理、转发平面工作原理、多归接入场景、EVN 其他增强功能五个章节来深入阐述 EVN 的工作原理。首先我们先借助 EVN 的组网模型来了解一些基本概念。

EVN 组网模型

EVN 的基本组网模型如图 2-55 所示，与 VPLS 组网类似，其角色组成包括数据中心站点网络 Site、PE 和 CE。

![0](https://share.note.youdao.com/yws/public/resource/fefec1c1fb8643c9ae9a59fbd4922ea9/xmlnote/37426C35D09F417CA03E8214A94F942D/46298)

@ Site ：Site 是指数据中心站内由服务器和交换机组成的二层网络域。多个 site 之间通过 EVN 网络实现互联。Site 的边界位于数据中心二三层网络分界的交换机上。

@ PE ：PE 之间是实现二层网络互联的逻辑 EVN 网络，其实际上的物理承载网络可以为运营商提供的传输专线、MPLS 网络等，总之只要 IP 路由可达即可。

PE 是 EVN 网络的核心设备，从形态来说既可以是独立的物理设备，也可以是从现有的站点边界设备中虚拟出来的逻辑设备（例如，汇聚层交换机作为二三层的边界，可以采用多台交换机先 CSS 后再 VS 的方式，使用两个 VS 分别作为 EVN PE 和三层网关）。

@ CE ：连接站点内二层网络域与 PE 的设备。既可以是独立的物理设备，也可以是虚拟的逻辑设备。使用独立的物理设备时，通常为汇聚层交换设备。数据中心内部的二层网络域可以是部署了 CSS、STP 或者 TRILL 的二层网络。

与 VPLS 中使用不同的 VSI 来区分不同的二层互联域类似，EVN 中使用 EVN 实例来区分同一个 PE 上相互独立的二层互联域。

之所以说 EVN 实例是一个独立的二层 互联域，而不是一个独立二层域，是因为 EVN方案中多个 VLAN 可以通过同一个 EVN 实例实现跨站点扩展。这样既保留了不同

VLAN 之间的转发隔离，也不需要为每一个 VLAN 独立进行 EVN 相关的配置，达到了简化配置的目的。

EVN 实例的标记包括 EVN Name 和 EVN ID。 EVN ID 一样才能互访

@ EVN Name 只具有本地意义，字符串形式，一般用于标识该 EVN 实例的作用。

@ EVN ID 具有全局意义，需要全局唯一。不同 PE 上属于同一个二层互联域的 EVN实例，其 EVN ID 必须一致。

EVN ID 的主要用于 PE 在收到对端发送过来的 MAC 地址可达信息时决定向哪一个 EVN 实例引入该可达信息。这一点同 VPN 实例中的路由交叉引入是类似的，具体的过程在后面我们会仔细介绍。

图 2-56 所示的是一个典型的通过 EVN 实现数据中心不同二层域互联的示例，DC1 中site 和 DC2、DC3 中 site 分别有二层互联的要求。

有二层互通需求的 Site 的 EVN 实例 ID 相同，例如图中 PE 上用于实现 site1 互联的EVN ID 都定义为 100，用于实现 Site2 互联的 EVN ID 都定义为 200。在 PE2 上 EVN

实例相关的配置如下：

#PE2 上第一个 EVN 实例，用于实现 VLAN100 到 199 的跨数据中心二层互联。

[PE2] evn evn-name site1

[*PE2-evn-site1] evn-id 100

[*PE2-evn-site1] vlan-list 100 to 199

[*PE2-evn-site1] quit

#PE2 上第二个 EVN 实例，用于实现 VLAN200 到 299 的跨数据中心二层互联。

[PE2] evn evn-name site2

[*PE2-evn-site1] evn-id 200

[*PE2-evn-site1] vlan-list 200 to 299

[*PE2-evn-site1] quit

![0](https://share.note.youdao.com/yws/public/resource/fefec1c1fb8643c9ae9a59fbd4922ea9/xmlnote/616367FA2CB347B5A01CB9C99B23065F/46299)

在图 2-56 中，有大部分 CE 是单链路连接到 PE，但 CE11 是双链路分别连接到了 PE11和 PE12，在前面我们预告过 EVN 能够支持这种双归属组网的场景。从实现原理的角度来说，双归属的场景下的原理比单归属场景下要复杂一些。接下来的内容，我们先介绍单归属接入场景下的基本实现原理，然后介绍双归属场景下的一些特殊实现。

在介绍实现原理时，我们将从控制平面和转发平面两个维度来描述。控制平面的建立过程大家可以理解为一个修路通车的过程，目的在于打通在始发地和目的地之间的通

道。而转发平面则可以理解为一次沿着既有道路的乘车过程。

控制平面工作原理

EVN 的控制层面主要包括两个阶段：第一阶段是 PE 之间 BGP 邻居关系建立，即修路过程；第二阶段是 PE 上转发表项（MAC 路由表、BUM 转发表）生成以及 PE 之间VXLAN 隧道建立，即通车过程。

MAC 路由表用于指导单播流量的转发；BUM 转发表用于指导广播、未知单播和组播报文的转发；VXLAN 隧道用于后续数据报文的封装。

BGP 邻居关系建立

在 VPLS 方式中通过在 PE 之间建立全连接来实现 PE 之间邻居关系的建立。为简化全连接配置，EVN 引入 BGP RR（Route Reflector，路由反射器）功能，所有 PE 设备均

指向 RR 并与之建立 BGP 对等体关系，RR 发现并接受 PE 发起的 BGP 连接后形成Client 列表，即将从某个 PE 收到的路由反射给所有其它 PE。

在 EVN 网络中， RR可以是一台独立的设备，也可以由 PE 设备兼任。在下图所示的单归属组网中，假设PE2 同时作为路由反射器，其 BGP 相关配置如下：

[~PE2] evn bgp

[*PE2-evnbgp] source-address 2.2.2.2

[*PE2-evnbgp] peer 1.1.1.1 reflect-client

[*PE2-evnbgp] peer 3.3.3.3 reflect-client

PE1、PE3 上只需要进行如下配置即可，以 PE1 为例：

[~PE1] evn bgp

[*PE1-evnbgp] source-address 1.1.1.1

[*PE1-evnbgp] peer 2.2.2.2

![0](https://share.note.youdao.com/yws/public/resource/fefec1c1fb8643c9ae9a59fbd4922ea9/xmlnote/95F893FE32094802BDF1ABC6824E0F02/46300)

BGP 邻居关系建立以后，可以通过 display evn bgp peer 查看对等体的建立情况。

“State”字段为“Established”表示 BGP 邻居建立成功。

[~PE2] display evn bgp peer

BGP local router ID : 2.2.2.2

Local AS number : 65534

Total number of peers : 2

Peers in established state : 2

Peer V AS MsgRcvd MsgSent OutQ Up/Down State PrefRcv

1.1.1.1 4 65534 4 6 0 00:00:23 Established 1

3.3.3.3 4 65534 4 6 0 00:00:14 Established 1

针对 EVN BGP 邻接关系的安全性和可靠性，CE 交换机还分别支持了 Keychain 认证和BFD 检测，实现与针对普通 BGP 的配置一致。示例如下：

system-view

[~HUAWEI] evn bgp

[*HUAWEI-evnbgp] authentication keychain abc

[*HUAWEI-evnbgp] bfd min-tx-interval 100 min-rx-interval 100 detect-multiplier 5

BGP 邻居管理建立完成后，两个站点之间的道路已经打通了，但是真正的实现载客还必须有一个通车的过程。这个过程在 EVN 中就是转发表项和 VXLAN 隧道的生成过程。

转发表项和 VXLAN 隧道生成

建立 BGP 邻居关系后，PE 之间开始了路由可达信息的通告过程。在 EVN 中 PE 之间会交互 4 种路由，熟悉 EVPN 的同学对这四种路由应该不陌生，分别是：集成多播路

由、MAC 地址通告路由、以太自动发现路由和以太网段路由。EVPN 中这四种路由的功能定义被保留了下来，但是其路由中包含的信息则在 EVN 中做了重新定义，这一点

我们在后面讲路由信息的交互过程中会详细描述。

这四种路由信息的功能简要描述如下：

@ 集成多播路由（Inclusive Multicast Route）：当 PE 之间的 BGP 邻居关系建立成功后，PE 之间会传递集成多播路由。用于生成 BUM 转发表，同时自动建立传送数

据报文的 VXLAN 隧道。

@ MAC 地址通告路由（MAC Advertisement Route）：BGP 邻居关系建立后，当 PE上的 MAC 表信息发生变化时，PE 即向对等体 PE 发送 MAC 地址通告路由，用于

更新对端 PE 上的 MAC 表，指导单播报文的转发。vtep

@ 以太自动发现路由（Ethernet Auto-Discovery Route）： PE 之间的 BGP 邻居关系建立成功后， PE 之间会传递以太自动发现路由。以太自动发现路由可以向其他 PE

通告本端 PE 对接入站点的 MAC 地址的可达性，主要用于 CE 多归属的场景中的水平分割和快速收敛。

@ 以太网段路由（Ethernet Segment Route）：当 PE 之间的 BGP 邻居关系建立成功后，PE 之间会传递以太网段路由，用来实现连接到相同 CE 的 PE 设备之间互相自动

发现。以太网段路由主要用于 CE 多归属场景中的 DF 的选举。在单归属场景中，主要涉及前两种路由的交互。接下来我们就看一下，这两种路由信息是如何促成控制面表项和隧道生成的。

MAC 路由表生成

MAC 路由表用于指导单播报文的转发，PE 上 MAC 表中的 MAC 地址信息来源于两部分，第一部分是 PE 下挂的 CE 侧本地网络的 MAC 地址信息，另一部分为由对端 PE

同步过来的 MAC 地址信息。PE 上本地网络的 MAC 地址学习过程与普通二层网络的 MAC 地址学习过程没有差别，通过数据平面（site 内经过 CE 发送的 ARP 请求报文、免费 ARP 报文等）从与 CE 设备相连的接口上学习到新的 MAC 地址。对于从网络侧同步对端 PE 的 MAC 地址信息，这个过程通过在 PE 间传递 MAC 地址通告路由来实现，是 EVN 控制面表项生成的一个重要环节。

PE 上生成新的本地 MAC 表项（MAC，VLAN，Interface）后，即开始了向对端 PE 发送 BGP Update 信息。Update 信息的 NLRI字段携带的 MAC 地址通告路由的关键信息。

如图 2-58 所示。

![0](https://share.note.youdao.com/yws/public/resource/fefec1c1fb8643c9ae9a59fbd4922ea9/xmlnote/2990A616055E4F5E9E00B8AD49E78691/46301)

各主要字段在 EVN 中的定义如下：

@ Route Target：用于控制 EVN 实例的路由引入，由 AS number 和 EVN ID 组成。只有 Target 的值一致才会引入该消息中携带的 MAC 路由信息。

@ RD：表示该地址族的路由标志符。由发送方 PE 的 Source IP + EVN ID 组成。

@ ESI：PE 上连接 CE 的链路标识。由用户手工配置或者通过 LLDP 生成，主要用于

双归属场景，在后面介绍。（注意单归属场景下也必须要配置。）

@ MAC address：需要通告给对端 PE 的 MAC 地址。

@ Ethernet Tag：学习到该 MAC 的 VLAN ID。

@ IP Addr 和 MPLS Label：暂未使用，全 0。

从网络侧同步 MAC 地址信息的具体过程，我们以图 2-59 所示的组网为例进行说明。当站点内主机（MAC-1）上线后， PE1 上生成相应的 MAC 表项（MAC-1, VLAN100,

Interface1），各 PE 之间开始了 MAC 路由信息的同步。

![0](https://share.note.youdao.com/yws/public/resource/fefec1c1fb8643c9ae9a59fbd4922ea9/xmlnote/915D804B2284428EBD390003A35B338D/46302)

@ PE1 向作为 RR 的 PE2 发送携带了 MAC 地址通告路由的 Update 消息。

@ PE2 从 PE1 接收到 Update 消息后，根据 NLRI 字段携带的 EVN ID、Source IP、VLAN ID、MAC 地址等信息向指定 EVN 实例中引入 MAC 路由信息。

@ 与传统 MAC 转发表项不同的是，通过这种方式生成的 MAC 转发表项不再是MAC 地址、VLAN 与端口的关联关系，而是 MAC 地址、VLAN 与 IP 地址

（Update 报文中的 SourceIP 地址）的关联关系。

@可以通过 display macm table evn evn-name 可以查看生成的 MAC 路由信息。PE2 根据接收到的 MAC 地址通告路由，将相关 MAC 地址可达信息引入到 EVN ID=100（EVN name=site1）的 EVN 实例中，display macm table evn site1 的显示

信息如下：

display macm table evn site1

Evn-name:site1 LocalNum:2,RemoteNum:3

---------------------------------

MAC Address VLAN Learned-From

---------------------------------

MAC-1 100 1.1.1.1

MAC-2 100 1.1.1.1

MAC-3 100 1.1.1.1

……

@ PE2 将从 PE1 接收到 MAC 路由，反射给 Client 列表中的 PE3，PE3 上的处理步骤与 PE2 上一致。如果想要更精确的控制 MAC 路由的发布，可以通过引入 acl 过滤实现。相关命令为 filter-policy{ acl-number | acl-name acl-name } export | import（EVN 实例视图）MAC 路由表只能指导已知单播报文的转发，针对广播报文、未知单播报文和组播报文，设备则是根据 BUM 转发表来转发。EVN 中，无论是哪一种报文在 PE 之间转发时，都经过了 VXLAN 封装。而 BUM 转发表和 VXLAN 隧道的建立都是通过在 PE 间交互集成多播路由信息来实现的。BUM 转发表生成和 VXLAN 隧道的自动建立EVN 中对 BUM 流量转发采用头端复制的方式，即 PE 将从 CE 侧接收并识别出的BUM 流量逐一复制并转发到所有 Peer PE 设备。设备内部记录了一个本端 PE 需要复制的远端 PE 设备地址列表，这个内部存在的表我们暂且称为 BUM 转发表。那么这个转发表是如何建立的呢？

BUM 转发表建立主要依靠 BGP Update 信息中 NLRI 字段携带的集成多播路由信息。BGP 邻居关系建立后就开始集成多播路由的发送，该路由信息的具体字段如图 2-60 所

示。

![0](https://share.note.youdao.com/yws/public/resource/fefec1c1fb8643c9ae9a59fbd4922ea9/xmlnote/A22CF6596FB445BEB1D86E0BD899CBC2/46303)

各主要字段在 EVN 中的定义如下：

 RD：表示地址族的路由标志符。由发送方 PE 的 Source IP + EVN ID 组成。

 Ethernet Tag：全 0

 Originating Router IP Addr：发送方 PE 的 Source IP 地址。

需要说明的是，当 PE 发送的 Update 信息中携带了集成多播路由信息时，其 PMSI 隧道属性信息的“Tunnel Type”字段被指定为“Ingress Replication”即头端复制，明确了

EVN 中对 BUM 报文的处理方式（其他的处理方式还包括 P2MP LSPs 等）。PE 上生成的 BUM 转发表中 PE 相关基本信息如图 2-61 所示。

![0](https://share.note.youdao.com/yws/public/resource/fefec1c1fb8643c9ae9a59fbd4922ea9/xmlnote/878C01169E86496BB4C9CC94A26ADF3C/46304)

在控制面生成 BUM 转发表的同时，VXLAN 隧道的建立也同步进行。Update 消息中携带的发送方 PE 的 Source IP 地址将作为隧道的目的地址，本地接收方 PE 的 BGP Source IP 同时作为 VXLAN 隧道的源地址。隧道建立完成后可以通过 display vxlan tunnel 查看隧道是否建立成功。

display vxlan tunnel

Number of vxlan tunnel : 2

Tunnel ID Source Destination State Type

--------------------------------------------------------------

33686018 2.2.2.2 3.3.3.3 up dynamic

33686019 2.2.2.2 1.1.1.1 up dynamic

到这里为止，控制面表项和隧道的建立过程就描述完了。正所谓路修好了，车也通了，接下来应该就是来一场说走就走的旅行了。接下来我们就来看一下报文是如何根据控制面的表项进行转发的，也即转发平面的工作原理。

2.5.2.3 转发平面工作原理

2.5.2.3.1 单播流量转发

单播流量转发过程在站点内和站点间有所不同，对于站点内单播流量，当报文到达 PE设备时，PE 在 MAC 表中查找报文目的 MAC，得到出接口为本地接口，报文按传统二

层报文转发方式转发到对应出接口，这里不再详述。站点间的单播流量转发过程我们以图 2-62 中 CE1 和 CE2 两个主机之间的单播流量转发为例来阐述。

![0](https://share.note.youdao.com/yws/public/resource/fefec1c1fb8643c9ae9a59fbd4922ea9/xmlnote/894447644B4E4FEBBC75FBC1E5E53A81/46305)

当 CE1 将从 MAC-A 接收到的单播报文透传给 PE1 时，PE1 在 MAC 表中查找报文目的 MAC 地址 MAC-B，得到的出接口不是本地接口，而是远端 PE2 的 IP 地址 IP2。报文进入 EVN 网络进行转发。

 PE1 设备对接收到的原始以太数据报文进行“MAC in UDP”封装（VXLAN 封装），

然后按照标准三层转发流程进行下一跳查找转发。

PE1 根据报文的 VLAN 值先进行 VXLAN 头部封装（VNI 值=VLAN 值），之后依次进行 UDP 头、IP 头和最外层的 ETH 封装。封装后的报文格式如图 2-63 所示。

− UDP 头中 Sport 为 HASH 获得随机值，Dport 固定为 4789 表示 VXLAN 封装；由于报文的 Sport 是随机生成的，承载网络中的设备接收报文后，如果到达目

的 IP 存在多条等价路径，则可利用外层 IP 封装的五元组信息将报文 Hash 到多条等价路径之间进行负载分担。

− IP 头部中的源地址为 PE1 的 IP 地址 IP1，目的地址设置为远端 PE2 的 IP 地址IP2。

− 最外层 ETH 封装中路由表中下一跳对应的 MAC 地址。源 MAC 为发送方的MAC 地址。

![0](https://share.note.youdao.com/yws/public/resource/fefec1c1fb8643c9ae9a59fbd4922ea9/xmlnote/FCB143893D0441EB902959255101C03B/46306)

PE2 对接收到的报文进行解封装，剥掉 IP 头后，根据 VNI 的值映射到对应 VLAN，在 VLAN 内根据报文的目的 MAC（MAC-B）查找 MAC 表，得到出接口为本地接口 Interface1。报文进入本地二层网络，按照普通二层报文转发方式转发到MAC-B。

BUM 流量转发

BUM 流量由于采用的是头端复制的广播方式，转发的过程相对简单。报文在发送端PE 上进行 VXLAN 封装后遵循普通 IP 报文的转发流程，到达 BUM 转发表中的对端PE。具体的转发过程如图 2-64 所示。

![0](https://share.note.youdao.com/yws/public/resource/fefec1c1fb8643c9ae9a59fbd4922ea9/xmlnote/36FC7A8826874126A2B19B9648A52FDA/46307)

@ 当目的 MAC 为 F-F-F 的报文到达 PE1 时，PE1 发现其目的 MAC 地址为广播地址，则报文将进入跨站点的广播过程。

@ PE1 设备查询 BUM 转发表发现报文需要发送到对端的 PE2 和 PE3。PE1 将报文复制 2 份，并分别进行 VXLAN 封装，封装格式与单播流量转发封装格式相同。

@ 封装后得到的 IP 报文经过 IP 报文的转发流程到达目的 PE。PE2 和 PE3 对接收到的报文进行解封装，获取原始的以太数据报文，再按传统二层报文转发方式转发到站点内主机。

在介绍完单归场景下 EVN 的实现原理后，我们可以开始复杂的多归属场景下的原理学习了，首先我们先了解一下 EVN 中有哪些实现多归属接入的方案。

多归接入场景

数据中心二层网络互联中的 CE 多归接入主要是指在一个站点内部署多个 PE（最典型的是图 2-65 所示的双归属接入），PE 分别与站点内的同一个 CE 互联，以提高网络的可靠性。

![0](https://share.note.youdao.com/yws/public/resource/fefec1c1fb8643c9ae9a59fbd4922ea9/xmlnote/898AB115DEDC4ECA8F804A32569DE955/46308)

在前面介绍 VPLS 方案的问题时，我们有提到过，这种跨站点的多归属组网很容易出现环路。从环路的解决方案来看，不外乎两种：1、阻塞链路，即多条链路中只有一条用于转发，其余链路被阻塞；2、负载分担，多链路之间分担流量转发。EVN 中，这两种解决方案均得到了实现。

第一种方案被称为 Single-Active 模式，也即单活模式.

第二种方案被称为 ALL-Active 模式，也即多活模式。需要强调的是，在 EVN 中,与同一 CE相连的所有 PE 均为 all-active 模式时，则这些PE 在传输单播流量时才可以进行负载分担。只要有一个的 PE 设备是 single- active 模式，则这些 PE、CE 间链路将只能进行流量备份。

Single-Active 模式

Single-Active 模式的核心是只保留一条链路用于转发，其余链路均处于阻塞状态，实现多链路备份的功能。那么问题来了，选择哪一条链路用来转发呢？

在 EVN 中，通过 DF选举来决定哪一条链路用于转发流量。在实际流量过程中，我们发现当只保留一条物理链路来转发流量，所有备份链路空闲时，链路利用率太低。为使流量转发能在多个链路上形成负载分担，而且又不会形成环路，EVN 将每个 PE 设备都作为主 DF 负责一部分 VLAN 的流量转发。即对于每一个 VLAN 来说都有唯一的PE 负责转发流量，对每一个 PE 来说可以同时承担多个 VLAN 的流量转发。接下来我们就看一下如何确定某个 VLAN 的主 DF，也即 DF 的选举过程。

DF 选举

连接同一个 CE 的多个 PE 组成一个冗余组，为了能够根据设备及链路状态自动调整流量的分担情况，需要实现 PE 设备的自动发现（PE 的退出或者加入）和 DF 选举。EVN 利用以太网段路由信息的交互来实现该功能。PE 之间 BGP 邻居关系建立后，如果在 PE 的接口上配置了 ESI，且接口的状态为 Up，就开始了通过 BGP Update 信息发送以太网段路由信息的过程。以太网段路由信息与之前的集成多播路由信息一样包含在 Update 报文的 NLRI 字段中，不同的是 BGP 的 Update 报文中还携带了 ES-Import 扩展团体属性，该路由信息的主要字段如图 2-66 所示。

![0](https://share.note.youdao.com/yws/public/resource/fefec1c1fb8643c9ae9a59fbd4922ea9/xmlnote/6406B1D719AC4F779BF8D2E2EF8D83CF/46309)

![0](https://share.note.youdao.com/yws/public/resource/fefec1c1fb8643c9ae9a59fbd4922ea9/xmlnote/0DB4A1162010410CAAC24826D2085B66/46310)

各主要字段在 EVN 中的定义如下：

 RD：该地址族的路由标志符，由发送方 PE 的 Source IP 构成。

 ESI：标识接入 CE 的链路。当不同 PE 上的链路定义了相同的 ESI 值时，表示这些 PE 上的链路连接到了同一个 CE。可以通过手工配置或动态生成。

− 静态方式：基于接口配置。示例：

[~PE1] interface 10ge 1/0/0

[*PE1-10GE2/0/0] esi 0000.1111.1111.1111.2222

− 动态方式：通过 LLDP 生成。

即使是单归属模式，EVN 也要求配置 ESI 值，以防止在后续链路扩充形成多归属时还需要修改已有链路的配置。

 Ethernet Tag：全 0

 Originating Router IP Addr：发送方 PE 的 Source IP 地址。

 ES-Import RT：位于 ES-Import 扩展团体属性中，该字段的值系统从 ESI 中自动获取。只有连接到同一 ESI 的 PE 之间才能互相引入 Ethernet Segment Route。接收到携带了以太网段路由的 Update 消息后，存在相同 ESI 值的 PE 将引入以太网路由信息。

− 根据以太网段路由信息中携带的 ESI 值， PE 上会生成多归 PE 列表，包含连接到同一个 CE 的所有 PE 的信息。

− 根据以太网段路由获取 Source IP 地址（包含在 RD 字段中）的大小顺序，对多归 PE 列表内的 PE 进行从 0 开始，由小到大排序 。

− 每一个 PE 计算自身作为 DF 的序号，DF 序号 I = VLAN ID mod N。其中VLAN ID 为 PE 上 EVN 实例绑定的 VLAN 的 ID，N 为多归 PE 列表内包含的PE 数量。

PE 上如果算出的 i 值和之前根据 Source IP 地址大小的顺序排序时候的值一致，则该 VLAN 在本地 DF 选举为主，否则 DF 选举为备。备 DF 上连接 CE 的接口退出相应的 VLAN。

以图 2-67 所示的双归属组网为例，图中存在 2 个冗余组 PE1 和 PE2、PE3 和 PE4，用于实现 VLAN100、VLAN101 的跨站点二层互联。

假设 IP1PE1-->PE3-->CE34 转发。

思考题：本端PE对如何得知端site的VLAN的DF是那台设备?

1、备DF不学习对应VLAN的MAC地址，无法通过MAC地址路由。

2、PE3,PE4用于相同ESI，以太网段路由通告给PE1之后，PE1能自己计算出PE3是VLAN 100 DF。

![0](https://share.note.youdao.com/yws/public/resource/fefec1c1fb8643c9ae9a59fbd4922ea9/xmlnote/2E765D6140164EB4B75EC7E9FAEED003/46311)

针对 BUM 报文，当 MAC-1 发送的目的 MAC 为广播地址的报文后：由于 PE2 上连接 CE12 的接口上阻塞了 VLAN100 的流量，因此只有 PE1 能接收到该广播报文。前面我们提到过 EVN 中转发 BUM 流量时，会向所有头端复制列表中 PE 复制报文。

在图 2-67 所示的组网中，PE1 会将广播报文进行 VXLAN 封装后，复制给 PE2、PE3、PE4。PE2 接收到报文后，对报文进行解封装，由于连接 CE12 的接口上阻塞了 VLAN100 的流量，因此报文不会发送给 CE12，即接入侧不会形成 CE12 --> PE1 --> PE2 --> CE12的本端环路。对于网络侧，PE3 接收到 PE1 发送的报文后进行解封装，并将解封装后的报文发送给CE34。PE4 由于连接 CE34 的接口阻塞了 VLAN100 的流量，不会将解封装后的报文发送给 CE34。为了避免 PE3 再次收到 PE2 或者 PE3 发送过来的广播报文， EVN 中所有的 PE 设备不支持流量解封装之后再封装，即从对端 PE 接收到的流量不会在本端 PE 解封装后再向其他 PE 发送。在上图中，CE12->PE1->PE2 过来的流量不支持在 PE2 上解封装之后，再向 PE3 复制。

在 Single-Active 模式下，多条链路之间是一种备份的关系，那么一旦原来的转发链路发生故障，将如何进行备份链路的切换呢，整个网络又将如何进行收敛呢？

EVN 实现了快速收敛机制来保证原有链路故障时整个网络的快速收敛。

快速收敛

在图 2-68 所示的网络中，CE1 双归连接到 PE1 和 PE2，PE1 和 PE2 的 EVN 冗余模式配置为 Single-Active。PE1 被选举为 VLAN 100 的 DF 后，只有 PE1 上会生成 MAC-1表项，且 PE1 通过 MAC 地址通告路由向 PE3 通告 MAC-1 的可达信息。而 BGP 邻居关系建立后 PE1 和 PE2 会同时向 PE3 发送 Ethernet AD 路由。

正是 Ethernet AD 路由中携带的信息让 PE3 感知到 PE1 和 PE2 均连接到 CE1（ESI A），且 PE1、 PE2 与 CE1 之间的链路处于 Single-Active 模式。那么 Ethernet A-D 路由中携带了哪些具体信息呢？

![0](https://share.note.youdao.com/yws/public/resource/fefec1c1fb8643c9ae9a59fbd4922ea9/xmlnote/0AFB25C7A33F418585E1767F35E7DA28/46312)

Ethernet A-D 路由与其他路由一样包含在 Update 报文的 NLRI 字段中，不同的是 BGP的 Update 报文中还携带了 Route Target 和 ESI Label 扩展团体属性，具体的路由信息样式如图 2-69 所示。

![0](https://share.note.youdao.com/yws/public/resource/fefec1c1fb8643c9ae9a59fbd4922ea9/xmlnote/97F789ABDAFB4F9DB09726EF012F5DEA/46313)

![0](https://share.note.youdao.com/yws/public/resource/fefec1c1fb8643c9ae9a59fbd4922ea9/xmlnote/62C07510D36845FC8A44FB70880873F9/46314)

各主要字段在 EVN 中的定义如下：

 RD：该地址族的路由标志符，取值为发送方 PE 的 Source IP。

 Ethernet Tag ID：全 0

 MPLS Label：全 0

 RT：通过 AS Number + EVN ID 自动生成；该接口接入多个 EVN 实例的话，就携

带多个 RT。

 ESI Label 扩展团体属性：其中的 ESI Label 和 Flags 字段分别携带了 ESI 值和冗余

模式（Single-Active 或 ALL-Active）。用户通过命令行指定冗余模式为哪一种。例

如：

# 指定冗余模式为多活模式（缺省情况下为 Single-Active 模式）。

[~PE1] evn bgp

[*PE1-evnbgp] redundancy-mode all-active

PE3 收到 PE1 发送 MAC 地址通告路由后，发现 MAC-1 来自 ESI A。而通过接收到的Ethernet A-D 路由，PE3 发现 PE1 和 PE2 连接到了相同的 ESI 且处于 Single-active 状态，因此 PE3 认为 MAC-1 通过 PE2 也可达，即 PE3 将 PE2 作为到达 MAC -1 的备份。

当 PE1 与 CE1 之间的链路故障时（ESI 关联的接口变为 Down），网络的收敛情况如图2-70 所示，PE1 向 PE3 撤销之前发送的 Ethernet A-D 路由（发送 withdraw 报文），PE3接收到 PE1 发送的撤销报文后，可批量撤销 ESI A 所关联的 MAC 路由，将所关联的MAC 路由的下一跳从 PE1 切换为备份下一跳 PE2，而不需要通过逐一接收 MAC 撤销路由来触发路由更新，从而加快 PE3 上的路由更新和收敛。

![0](https://share.note.youdao.com/yws/public/resource/fefec1c1fb8643c9ae9a59fbd4922ea9/xmlnote/BDB7D60914AA4D8CBC0C3AE2260B5087/46315)

在图 2-70 所示的组网中， PE1 与 CE1 之间的链路恢复后，如果 PE1 先与 PE2 建立起邻居关系，后与 PE3 建立邻居关系，此时会出现如下问题。

PE1 与 PE2 相互发送以太网段路由并恢复成为主 DF，PE2 则降为备份 DF，此时 PE3由于尚未与 PE1 建立邻居关系，依然向 PE2 转发流量，而这些流量将会被丢弃。EVN中为了解决这个问题，支持对等体状态跟踪功能。接下来我们就看一下对等体状态跟踪功能是如何实现的？

![0](https://share.note.youdao.com/yws/public/resource/fefec1c1fb8643c9ae9a59fbd4922ea9/xmlnote/79B09977EE92433C9492B16B85E70934/46316)

对等体状态跟踪

在图 2-70 所示的组网中， PE1 连接 CE1 的接口上配置 EVN BGP 对等体状态跟踪功能且 PE1 连接 CE1 的接口 Up 后，PE1 会先后启动以太网段路由的发送延时计时器和以太网段路由的接收计时器，在这两个计时器结束前，PE1 将不参与进行 DF 选举。具体过程可以分为两部分：

 PE1 首先启动以太网段路由的发送延时计时器，然后 PE1 会跟踪其与 PE2 和 PE3的邻居状态。3

− 在计时器时间间隔内，如果 PE1 和 PE2、PE3 的邻居状态都 Up 了，则 PE1 会向 PE2 和 PE3 发送以太网段路由。

− 计时器超时后，PE1 则仅向邻居关系状态为 Up 的 PE 设备发送以太网段路由；

 当以太网段路由的发送延时计时器结束后，PE1 会启动以太网段路由接收计时器。PE1 将会在该计时器规定时间间隔内等待其他 PE 发送来的以太网段路由，当计时器结束后，PE1 将根据收到的全部以太网段路由进行 DF 选举。

EVN BGP 对等体状态跟踪功能涉及的相关配置如下：

system-view

[~HUAWEI] interface 10ge 1/0/0 //进入 PE 上连接 CE 的接口视图

[~HUAWEI-10GE1/0/0] es track evn-peer 1.1.1.1 //使能针对指定对等体的状态跟踪功能

配置该功能后，缺省的以太网段路由的发送延时计时器时间间隔为 600 秒，以太网段路由接收计时器的时间间隔为 3 秒。如果需要修改可以在 EVN BGP 视图下执行以下命

令修改：

system-view

[~HUAWEI] evn bgp

[*HUAWEI-evnbgp] timer es-advertisement 10 //配置发送以太网段路由的延时计时器的时间间隔为10 秒

[*HUAWEI-evnbgp] timer es-reception 1 //配置以太网段路由接收计时器的时间间隔为 1 秒

尽管 Single-Active 模式这种基于 VLAN 进行流量分担的方式已经提高了链路利用率，但是在某一 VLAN 的流量特别大的情况下，这种实现方式可能出现链路负载不均衡的问题，最好的方案是能够实现流量在所有的多归链路间负载分担，也即多活 All-Active方案。

2.5.2.4.2 ALL-Active

当 CE 多归连接到多个 PE 时，这些 PE 形成一个冗余组。对于某个 VLAN，如果冗余组中所有 PE 都能从 CE 接收流量或转发流量到 CE，则这种多归冗余模式称为“All-Active”模式。注意，这种场景下要求在 CE 设备上将多条连接 PE 的链路进行捆绑。All-Active 模式与 Single-Active 模式一样同样存在 DF 选举且选举算法相同，只是在对选举结果的应用上有所不同。

在 All-Active 模式下，冗余组中的特定 VLAN 的 non-DF 不再是简单地阻塞其与 CE 之间链路上对应的 VLAN，而是针对单播流量和 BUM 流量采用不同的方案。

单播流量

如图 2-71 所示，CE1 双归连接到 PE1 和 PE2，PE1 和 PE2 的 EVN 冗余模式配置为All-Active。对于单播流量，CE1 发出的流量经过 LAG HASH 发给 PE1 或 PE2，PE1 和 PE2 上查MAC 表单播到 CE2。由于 PE1、PE2 都会学习到 MAC-1，都会给 PE3 发布 MAC-1 的路由，则 PE3 上就会形成到 MAC-1 的等价路由。这样在回程的时候，MAC-2 发给MAC-1 的流量在 PE3 上进行 ECMP HASH 选路，发给 PE1 和 PE2，从而实现负载分担。流量到达 PE1 和 PE2 后，再根据 MAC 表单播给 CE1。

![0](https://share.note.youdao.com/yws/public/resource/fefec1c1fb8643c9ae9a59fbd4922ea9/xmlnote/6C7EC3676B014E74A5CD346744612534/46317)

BUM 流量

对于 BUM 流量，在 CE1 往 CE2 方向，CE1 通过 LAG HASH 将流量发给 PE1 和 PE2。PE1 和 PE2 将流量复制给 BUM 转发表中的其他 PE。在下图所示的网络中，为了避免流量经过多归的 PE 再返回给 CE1（CE1 --> PE1 --> PE2 -->CE1）形成本端环路，EVN中还实现了水平分割功能，即主备 DF 之间不能相互复制 BUM 流量。

以图 2-72 所示的 VLAN100 的流量为例。PE1 收到的 CE1 过来的 VLAN100 的 BUM流量，虽然 PE2 也在 PE1 的 BUM 转发表中，但是 PE1 不再向 PE2 复制该流量（PE1将 PE2 从自己的 BUM 转发表中剔除），只向 PE3 复制。这即样做避免了前面提到本端环路的问题，也避免了 PE3 收到 2 份相同报文的问题。

在 CE2 往 CE1 方向，CE2 发出的 BUM 流量到达 PE3 后，PE3 发现 PE1 和 PE2 均存在于 VLAN100 的 BUM 转发表中，于是将流量复制两份分别发送给 PE1 和 PE2。流量到达 PE1 和 PE2 后，如果 PE1 和 PE2均将报文进行解封装后发送给 CE1，CE1 将重复收到 2 份流量。为了避免这一问题，在 EVN 中，只有作为 VLAN100 主 DF 的 PE1 能够将解封装后的流量发送给 CE1，而作为备 DF 的 PE2 将接入链路下行退出 VLAN100，上行不退，从而实现了流量可以从 CE1 发送给 PE2，而不能从 PE2 发送给 CE1。

PE1 通告 EVN ID ESI PE2 通过 EVN ID ESI PE1,PE2 连着同一个CE，模式多活

![0](https://share.note.youdao.com/yws/public/resource/fefec1c1fb8643c9ae9a59fbd4922ea9/xmlnote/65BD6AA43D0C47168DE9FACB95C59187/46318)

到这里位置 EVN 的基本功能我们就都已经描述完了，接下来我们就来看一看 EVN 还提供了哪些增强功能？

在前面描述物理方式扩展二层网络时，我们说过，这种方式实际上也大大扩展大二层网络中环路、广播风暴的影响范围。而在 EVN 中，我们实现了 STP 隔离、未知单播洪

泛抑制和 ARP 代理功能用来实现站点故障隔离，将环路以及广播风暴的影响范围控制在本地二层网络内。接下来我们就看一下，这些功能的详细描述

EVN 其他增强功能

未知单播报文丢弃

缺省情况下，CE 设备对 BUM 报文中的未知单播报文也采取泛洪方式处理，站点内的未知单播报文可以跨数据中心广播。用户可以选择关闭该功能，以避免站点内的广播

影响其他站点，这样做也同时可以降低对互联带宽的占用，避免对端站点内学习到太多不必要的 MAC 地址。

以图 2-73 所示的组网为例，当目的 MAC 为 MAC2 的以太数据报文到达 PE1 设备时，PE1 的 MAC 地址表中查找不到 MAC2 对应的表项，即该报文将被视为未知单播报文。

如果用户在 PE1 上使能了 EVN 网络中未知单播报文丢弃功能，PE1 则不会将该未知单播报文通过 EVN 网络跨站点的转发到 PE2，该类报文只在本地进行泛洪。

![0](https://share.note.youdao.com/yws/public/resource/fefec1c1fb8643c9ae9a59fbd4922ea9/xmlnote/C1CA6A034F2E4F8F8048D2A21EA4825A/46319)

需要特别注意的是，当网络中存在只接收数据包而不发送数据包，或很少发送数据包的静默服务器时，关闭未知单播泛洪功能后，这类服务器可能因为没有 MAC 表项而导

致流量一直不通，此时可以使能针对指定 MAC 的泛洪功能解决问题。

关闭未知单播泛洪功能的相关配置如下：

system-view

[~HUAWEI] evn

[*HUAWEI-evn] unknown-unicast drop //使能 EVN 中丢弃未知单播报文功能

[*HUAWEI-evn] quit

[*HUAWEI] mac-address static 1234-2222-3333 vlan 10 flooding //只有 VLAN 10 中目的 MAC地址为 1234-2222-3333 的未知单播报文可以泛洪

ARP 缓存代答

网络中的广播报文主要为 ARP 广播报文。实现跨数据中心的大二层互联后，如果大量的 ARP 广播报文均进行跨站点泛洪，浪费互联带宽的同时，也使得站点故障机率增加。

为了解决这个问题，EVN 解决方案引入了 ARP 缓存代答功能。PE 设备通过捕获 ARP报文生成 ARP 缓存表，后续的 ARP 请求会先在本地 PE 的 ARP 缓存表中检索，如果

存在则在本地直接代答，无需广播到所有远端 PE。

![0](https://share.note.youdao.com/yws/public/resource/fefec1c1fb8643c9ae9a59fbd4922ea9/xmlnote/577BC12432A947F2BB2BBCCC166DBE49/46320)

PE1 接收到本地站点发出的首次 ARP 请求报文（请求 MAC- B 对应的 IP 地址），在本地 ARP 缓存表中没有查找到对应表项，将报文洪泛到远端站点。

 CE3 下 Site 中的主机（MAC-B）响应 ARP 请求，并由 PE3 将 ARP 响应返回给PE1。

 PE1 接收到 PE3 发送的 ARP 应答报文，在本地建立对应的 ARP 缓存表项，并将ARP 应答报文返回给本地站点。

 PE1 接收到本地站点中 CE1 发出的后续 ARP 请求报文，在本地 ARP 缓存表中查找到对应表项。

 PE1 代表远端站点直接响应 ARP 请求，返回 MAC-B 对应的 IP 地址 IP B，不再向远端站点广播 ARP 请求。

EVN 中 ARP 缓存代答功能涉及的相关配置如下：

system-view

[~HUAWEI] evn

[*HUAWEI-evn] arp cache enable

[*HUAWEI-evn] arp cache timeout 600

EVN双活网关迁移场景

@部署EVN多活网关后，若数据中心对应的网关设备故障，则其与外部网络通信的流量无法再通过EVN网络进

行迂回转发。因此，部署EVN多活网关时，请确保每个数据中心对应的VRRP网关组中都存在主备设备

@采用VRRP部署多活网关时，不同网关设备的IP地址和MAC地址都相同。为了避免PE设备上产生MAC地址漂移进而影响到报文三层转发，需要在EVN网络中不发布网关设备的虚MAC。

@采用VRRP部署多活网关时，不同网关设备的IP地址和MAC地址都相同。