
# why？
1.经过之前的学习，只是mpls te流量工程只是在 IP承载下的一些简单实验，比如“静态CR-LSP配置”，“静态双向共路实验”，“基于RSVP-TE的动态CR-LSP选路实验”都是基础的理论实验，那么MPLS TE 流量工程需要用在哪些位置，在什么地方部署。通过上面方式？
2.流量工程主要是部署在（已知的案例）：中国移动的IP承载网中，在核心区域：由9个城市的CR或者CR兼做BR组成的核心层，部署LDP Over RSVP（LDP Over TE），实质就是MPLS TE 流量工程。
![](assets/MPLS%20TE%20L3%20VPN(tnl-policy)实验/file-20260227170407261.png)

# 如何配置：
1.as内底层IGP可达，环回口可达
2.配置MPLS 进程，