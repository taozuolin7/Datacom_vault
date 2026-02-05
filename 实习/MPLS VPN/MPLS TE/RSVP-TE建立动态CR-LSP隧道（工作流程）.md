基于OSPF建立的 基于RSVP-TE的动态CR-LSP的建立，（没设置带宽）
![](assets/RSVP-TE建立动态CR-LSP隧道（工作流程）/file-20260205182053812.png)
**关键配置如下：**
```R
mpls lsr-id 1.1.1.1
mpls
 mpls te
 mpls rsvp-te
 mpls te cspf
#
interface GigabitEthernet0/0/0
 ip address 10.1.12.1 255.255.255.0
 ospf enable 1 area 0.0.0.0
 mpls
 mpls te
 mpls rsvp-te
#
interface LoopBack0
 ip address 1.1.1.1 255.255.255.255
 ospf enable 1 area 0.0.0.0
#
interface Tunnel0/0/0
 ip address unnumbered interface LoopBack0
 tunnel-protocol mpls te
 destination 4.4.4.4
 mpls te tunnel-id 104
 mpls te commit
#
ospf 1 router-id 1.1.1.1
 opaque-capability enable
 area 0.0.0.0
  mpls-te enable
#
```
## 从AR1-AR4的 PSB内容显示
```R
[AR1]display mpls rsvp-te psb-content
==============================================================
                       The PSB Content
==============================================================
 Tunnel Addr: 4.4.4.4                      Exist time: 0h 0m 0s
 Tunnel ExtID: 1.1.1.1                     Session ID: 104
 Ingress LSR ID: 1.1.1.1                   Local LSP ID: 1
 Previous Hop :  ----- /0x0     Next Hop : 10.1.12.2
 Incoming / Outgoing Interface: -----  / GigabitEthernet0/0/0
 InLabel : NULL                            OutLabel : 1024
 Send Message ID : 1                       Recv Message ID : 0
 Refresh Timer : 3028595476                Cleanup Timer : ---
 Session Attribute-
  SetupPrio: 7         HoldPrio: 7
  SessionAttrib: SE Style desired.
 LSP Type: -
 FRR Flag : No protection                  Local RRO Flag : 0x0
 ERO Information-
     L-Type         ERO-IPAddr       ERO-PrefixLen
   ERHOP_STRICT    10.1.12.2             32
   ERHOP_STRICT    10.1.23.2             32
   ERHOP_STRICT    10.1.23.3             32
   ERHOP_STRICT    10.1.34.3             32
   ERHOP_STRICT    10.1.34.4             32
 RRO Information-
    -----
 SenderTspec Information-
  Token bucket rate: 0.00
  Token bucket size: 1000.00
  Peak data rate: 0.00
  Minimum policed unit: 0
  Maximum packet size: 4294967295
 Path Message arrive on ----- from PHOP  -----
 Path Message sent to NHOP 10.1.12.2  on GigabitEthernet0/0/0
 Resource Reservation OK

 LSP Statistics Information:
   SendPacketCounter: 109         RecvPacketCounter: 104
   SendPathCounter: 109           RecvPathCounter: 0
   SendResvCounter: 0             RecvResvCounter: 104


```

```R
[AR2]display mpls rsvp-te psb-content
==============================================================
                       The PSB Content
==============================================================
 Tunnel Addr: 4.4.4.4                      Exist time: 0h 0m 0s
 Tunnel ExtID: 1.1.1.1                     Session ID: 104
 Ingress LSR ID: 1.1.1.1                   Local LSP ID: 1
 Previous Hop : 10.1.12.1/0x3     Next Hop : 10.1.23.3
 Incoming / Outgoing Interface: GigabitEthernet0/0/0  / GigabitEthernet0/0/1
 InLabel : 1024                            OutLabel : 1024
 Send Message ID : 1                       Recv Message ID : 0
 Refresh Timer : 3028198692                Cleanup Timer : 3028198324
 Session Attribute-
  SetupPrio: 7         HoldPrio: 7
  SessionAttrib: SE Style desired.
 LSP Type: -
 FRR Flag : No protection                  Local RRO Flag : 0x0
 ERO Information-
     L-Type         ERO-IPAddr       ERO-PrefixLen
   ERHOP_STRICT    10.1.23.3             32
   ERHOP_STRICT    10.1.34.3             32
   ERHOP_STRICT    10.1.34.4             32
 RRO Information-
    -----
 SenderTspec Information-
  Token bucket rate: 0.00
  Token bucket size: 1000.00
  Peak data rate: 0.00
  Minimum policed unit: 0
  Maximum packet size: 1500
 Path Message arrive on GigabitEthernet0/0/0 from PHOP 10.1.12.1
 Path Message sent to NHOP 10.1.23.3  on GigabitEthernet0/0/1
 Resource Reservation OK

 LSP Statistics Information:
   SendPacketCounter: 219         RecvPacketCounter: 213
   SendPathCounter: 112           RecvPathCounter: 112
   SendResvCounter: 107           RecvResvCounter: 101

```

```R
[AR3]display mpls rsvp-te psb-content
==============================================================
                       The PSB Content
==============================================================
 Tunnel Addr: 4.4.4.4                      Exist time: 0h 0m 0s
 Tunnel ExtID: 1.1.1.1                     Session ID: 104
 Ingress LSR ID: 1.1.1.1                   Local LSP ID: 1
 Previous Hop : 10.1.23.2/0x4     Next Hop : 10.1.34.4
 Incoming / Outgoing Interface: GigabitEthernet0/0/0  / GigabitEthernet0/0/1
 InLabel : 1024                            OutLabel : 3
 Send Message ID : 1                       Recv Message ID : 0
 Refresh Timer : 3028222832                Cleanup Timer : 3028223292
 Session Attribute-
  SetupPrio: 7         HoldPrio: 7
  SessionAttrib: SE Style desired.
 LSP Type: -
 FRR Flag : No protection                  Local RRO Flag : 0x0
 ERO Information-
     L-Type         ERO-IPAddr       ERO-PrefixLen
   ERHOP_STRICT    10.1.34.4             32
 RRO Information-
    -----
 SenderTspec Information-
  Token bucket rate: 0.00
  Token bucket size: 1000.00
  Peak data rate: 0.00
  Minimum policed unit: 0
  Maximum packet size: 1500
 Path Message arrive on GigabitEthernet0/0/0 from PHOP 10.1.23.2
 Path Message sent to NHOP 10.1.34.4  on GigabitEthernet0/0/1
 Resource Reservation OK

 LSP Statistics Information:
   SendPacketCounter: 214         RecvPacketCounter: 209
   SendPathCounter: 112           RecvPathCounter: 113
   SendResvCounter: 102           RecvResvCounter: 96

```

```R
[AR4]display mpls rsvp-te psb-content
==============================================================
                       The PSB Content
==============================================================
 Tunnel Addr: 4.4.4.4                      Exist time: 0h 0m 0s
 Tunnel ExtID: 1.1.1.1                     Session ID: 104
 Ingress LSR ID: 1.1.1.1                   Local LSP ID: 1
 Previous Hop : 10.1.34.3/0x4     Next Hop :  -----
 Incoming / Outgoing Interface: GigabitEthernet0/0/0  / -----
 InLabel : 3                               OutLabel : NULL
 Send Message ID : 0                       Recv Message ID : 0
 Refresh Timer : ---                       Cleanup Timer : 3028333516
 Session Attribute-
  SetupPrio: 7         HoldPrio: 7
  SessionAttrib: SE Style desired.
 LSP Type: -
 FRR Flag : No protection                  Local RRO Flag : 0x0
 ERO Information-
     -----
 RRO Information-
    -----
 SenderTspec Information-
  Token bucket rate: 0.00
  Token bucket size: 1000.00
  Peak data rate: 0.00
  Minimum policed unit: 0
  Maximum packet size: 1500
 Path Message arrive on GigabitEthernet0/0/0 from PHOP 10.1.34.3
 Path Message sent to NHOP  -----   on -----
 Resource Reservation OK

 LSP Statistics Information:
   SendPacketCounter: 97          RecvPacketCounter: 114
   SendPathCounter: 0             RecvPathCounter: 114
   SendResvCounter: 97            RecvResvCounter: 0

```

## 从AR4-AR1的 RSB内容显示
1.此时需要在 隧道接口上配置：**mpls te record-route label**
```R
[AR4]display mpls rsvp-te rsb-content
==============================================================
                       The RSB Content
==============================================================
 Tunnel Addr: 4.4.4.4                 Session Tunnel ID: 104
 Tunnel ExtID: 1.1.1.1
 Next Hop:  -----                     Reservation Style: SE Style
 Reservation Incoming Interface: -----
 Reservation Interface: -----
 Message ID : 0
 Filter Spec Information-
   The filter number: 1
   Ingress LSR ID: 1.1.1.1         Local LSP ID: 1       OutLabel: NULL
   Cleanup Timer : ---
   RRO Information-
    -----
 FlowSpec Information-
   Token bucket rate: 0.00
   Token bucket size: 1000.00
   Peak data rate: 0.00
   Minimum policed unit: 0
   Maximum packet size: 1500
   Bandwidth guarantees: 0.00
   Delay guarantees: 0
   Qos Service is Controlled
 Resv Message arrive on Unknown(0x0) from NHOP 0.0.0.0

```

```R
[AR3]display mpls rsvp-te rsb-content
==============================================================
                       The RSB Content
==============================================================
 Tunnel Addr: 4.4.4.4                 Session Tunnel ID: 104
 Tunnel ExtID: 1.1.1.1
 Next Hop: 10.1.34.4                  Reservation Style: SE Style
 Reservation Incoming Interface: GigabitEthernet0/0/1
 Reservation Interface: GigabitEthernet0/0/1
 Message ID : 0
 Filter Spec Information-
   The filter number: 1
   Ingress LSR ID: 1.1.1.1         Local LSP ID: 3       OutLabel: 3
   Cleanup Timer : 3028223476
   RRO Information-
     RRO-CType: IPV4   RRO-IPAddress: 10.1.34.4        RRO-IPPrefixLen: 32
     RRO-CType: Label    RRO-Label: 3
     RRO-CType: IPV4   RRO-IPAddress: 4.4.4.4          RRO-IPPrefixLen: 32
     RRO-CType: Label    RRO-Label: 3
 FlowSpec Information-
   Token bucket rate: 0.00
   Token bucket size: 1000.00
   Peak data rate: 0.00
   Minimum policed unit: 0
   Maximum packet size: 1500
   Bandwidth guarantees: 0.00
   Delay guarantees: 0
   Qos Service is Controlled
 Resv Message arrive on GigabitEthernet0/0/1 from NHOP 10.1.34.4

```

```R
<AR2>display mpls rsvp-te rsb-content
==============================================================
                       The RSB Content
==============================================================
 Tunnel Addr: 4.4.4.4                 Session Tunnel ID: 104
 Tunnel ExtID: 1.1.1.1
 Next Hop: 10.1.23.3                  Reservation Style: SE Style
 Reservation Incoming Interface: GigabitEthernet0/0/1
 Reservation Interface: GigabitEthernet0/0/1
 Message ID : 0
 Filter Spec Information-
   The filter number: 1
   Ingress LSR ID: 1.1.1.1         Local LSP ID: 3       OutLabel: 1026
   Cleanup Timer : 3028198968
   RRO Information-
     RRO-CType: IPV4   RRO-IPAddress: 10.1.23.3        RRO-IPPrefixLen: 32
     RRO-CType: Label    RRO-Label: 1026
     RRO-CType: IPV4   RRO-IPAddress: 3.3.3.3          RRO-IPPrefixLen: 32
     RRO-CType: Label    RRO-Label: 1026
     RRO-CType: IPV4   RRO-IPAddress: 10.1.34.3        RRO-IPPrefixLen: 32
     RRO-CType: IPV4   RRO-IPAddress: 10.1.34.4        RRO-IPPrefixLen: 32
     RRO-CType: Label    RRO-Label: 3
     RRO-CType: IPV4   RRO-IPAddress: 4.4.4.4          RRO-IPPrefixLen: 32
     RRO-CType: Label    RRO-Label: 3
 FlowSpec Information-
   Token bucket rate: 0.00
   Token bucket size: 1000.00
   Peak data rate: 0.00
   Minimum policed unit: 0
   Maximum packet size: 1500
   Bandwidth guarantees: 0.00
   Delay guarantees: 0
   Qos Service is Controlled
 Resv Message arrive on GigabitEthernet0/0/1 from NHOP 10.1.23.3

```

```R
<AR1>display mpls rsvp-te rsb-content
==============================================================
                       The RSB Content
==============================================================
 Tunnel Addr: 4.4.4.4                 Session Tunnel ID: 104
 Tunnel ExtID: 1.1.1.1
 Next Hop: 10.1.12.2                  Reservation Style: SE Style
 Reservation Incoming Interface: GigabitEthernet0/0/0
 Reservation Interface: GigabitEthernet0/0/0
 Message ID : 0
 Filter Spec Information-
   The filter number: 1
   Ingress LSR ID: 1.1.1.1         Local LSP ID: 3       OutLabel: 1026
   Cleanup Timer : 3028595936
   RRO Information-
     RRO-CType: IPV4   RRO-IPAddress: 10.1.12.2        RRO-IPPrefixLen: 32
     RRO-CType: Label    RRO-Label: 1026
     RRO-CType: IPV4   RRO-IPAddress: 2.2.2.2          RRO-IPPrefixLen: 32
     RRO-CType: Label    RRO-Label: 1026
     RRO-CType: IPV4   RRO-IPAddress: 10.1.23.2        RRO-IPPrefixLen: 32
     RRO-CType: IPV4   RRO-IPAddress: 10.1.23.3        RRO-IPPrefixLen: 32
     RRO-CType: Label    RRO-Label: 1026
     RRO-CType: IPV4   RRO-IPAddress: 3.3.3.3          RRO-IPPrefixLen: 32
     RRO-CType: Label    RRO-Label: 1026
     RRO-CType: IPV4   RRO-IPAddress: 10.1.34.3        RRO-IPPrefixLen: 32
     RRO-CType: IPV4   RRO-IPAddress: 10.1.34.4        RRO-IPPrefixLen: 32
     RRO-CType: Label    RRO-Label: 3
     RRO-CType: IPV4   RRO-IPAddress: 4.4.4.4          RRO-IPPrefixLen: 32
     RRO-CType: Label    RRO-Label: 3
 FlowSpec Information-
   Token bucket rate: 0.00
   Token bucket size: 1000.00
   Peak data rate: 0.00
   Minimum policed unit: 0
   Maximum packet size: 1500
   Bandwidth guarantees: 0.00
   Delay guarantees: 0
   Qos Service is Controlled
 Resv Message arrive on GigabitEthernet0/0/0 from NHOP 10.1.12.2

```

