ISIS是基于TLV架构的，所以对于IPv6不需要扩展新的协议，只需要在原有的基础上添加需要的TLV即可
 
ospfv2 ipv4  
ospfv3 ipv6
 
ipv4 132 128、130  
ISIS  
ipv6 232 236
   

```
[AR1]ipv6 
[AR1]isis 
[AR1-isis-1]ipv6 enable 
[AR1]interface GigabitEthernet0/0/0
[AR1-GigabitEthernet0/0/0]ipv6 enable 
[AR1-GigabitEthernet0/0/0]ipv6 address FE80::1 link-local
[AR1-GigabitEthernet0/0/0]isis ipv6 enable 1
```

ISIS为了支持IPv6利用的处理和计算，新增了两个TLV 和 一个NLPID（网络层协议标识符）
 
1.NLPID：设备两端通过hello报文的129tlv协商可以支持的ip能力/ipv6能力
 
2.新增的TLV：  
1.232 TLV：携带接口IPv6的地址，最多可以携带16个IPv6地址  
1.hello报文中的232tlv，携带发送hello报文接口的链路本地地址  
2.LSP报文中的232tlv，携带所有接口的第一个全球单播地址  
2.236 TLV：携带前缀信息 和 前缀信息对应的功能  
1.携带设备所有接口的前缀信息  
2.存在的功能bit：  
1.U bit：UP/DOWN bit位，执行L2到L1渗透时会进行bit的置位防环  
2.X bit：标识内外部路由条目的，X=0标识内部路由，X=1标识外部路由  
3.S bit：标识子TLV是否置位，用于携带route tag

```
[AR1]dis isis lsdb 0000.0000.0001.00-00 verbose

                        Database information for ISIS(1)
                        --------------------------------

                          Level-2 Link State Database

LSPID                 Seq Num      Checksum      Holdtime      Length  ATT/P/OL
-------------------------------------------------------------------------------
0000.0000.0001.00-00* 0x0000000d   0xddb3        1135          169     0/0/0   
 SOURCE       0000.0000.0001.00
 NLPID        IPV6
 AREA ADDR    49.0001 
 INTF ADDR V6 2011::1                        
 INTF ADDR V6 2012::1                        
 INTF ADDR V6 2100::1                        
 Topology     Standard
+NBR  ID      0000.0000.0001.01  COST: 10        
 IPV6         2011::1/128                      COST: 0         
 IPV6         2012::/64                        COST: 10        
 IPV6         2111::/64                        COST: 10        
 IPV6         2100::/64                        COST: 10        
```
 
对于IPv6路由的携带，ISIS都会认为是路由条目不做区分，还是当作叶子挂载到拓扑节点

如果网络中存在ipv4网络和ipv6网络，对于设备如何支持两种网络？  
1.设备支持双栈（即支持IPv4也支持IPv6）  
2.ISIS的多拓扑功能按照需求实现v4和v6的分别转发￼￼  
ISIS默认会按照SPF计算得到一条最优路径  
因为ISIS是同时支持IPv4和IPv6协议的  
所以计算的最优路径 ipv4和ipv6都会进行优选使用
 
isis默认是按照ipv4计算的最优路径

![Exported image](Exported%20image%2020251206150205-0.png)

**ISIS多拓扑：**  
设置ISIS对于ipv6支持的模式：  
[AR1-isis-1]ipv6 enable topology  
compatible 兼容模式  
ipv6 ipv6模式  
standard 标准模式（默认的）
 
标准模式：isis只会根据ipv4计算一条最短路径树，ipv4和ipv6都使用该最短路径转发  
兼容模式：isis会将ipv4的最短路径树，也会计算ipv6的最短路径树，ipv4和ipv6都使用ipv4的最短路径转发  
ipv6模式：isis会将ipv4的最短路径树，也会计算ipv6的最短路径树，ipv4和ipv6各自使用对应的最短路径转发