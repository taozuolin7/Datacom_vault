OSPFv2 IPv4 ospfv2无法支持ipv6的扩展  
OSPFv3 IPv6￼  
OSPFv3基于链路运行以及拓扑计算，而不再是网段  
因为存在链路本地地址，所以不存在其他网络前缀，ospfv3可以正常进行拓扑计算
 
OSPFv3支持一个链路上多个实例  
可以通过实例值将一条链路，虚拟为多条逻辑链路
 
OSPFv3对于报文、LSA条目内容做了改变
 
**ospfv3的配置：**  
[AR1]ospfv3 1  
[AR1-ospfv3-1]router-id 10.1.1.1
 
ospfv3的RID必须手工配置  
RID值依然是32位无符号的整数  
#  
interface GigabitEthernet0/0/0  
ospfv3 1 area 0.0.0.0  
#ospfv3的地址通告只能在接口下使能ospfv3

**ospfv3的特点：**  
1.链路本地地址的使用  
1.ospfv3发送的报文SIP是每台设备的链路本地地址  
ospfv3发送的报文DIP是固定的ff02::5/6  
2.作为路由计算后的下一跳  
3.设备的直连链路可以配置不同的网段前缀
 
2.通过添加实例参数，来影响同网段内邻居建立的状态  
1.实例ID可以取值（0-255）  
2.实例ID和ospfv3的进程ID 是一一对应的  
3.默认情况下，ospfv3的进程ID绑定实例0  
4.实例ID和进程ID绑定完成后，只有删除重新绑定，无法直接修改  
5.对于实例ID相同的设备可以正常交互报文建立邻居关系  
对于实例ID不同的设备，会直接丢弃对方的报文信息

OSPFv3的报文信息：  
1.报文头部

![Exported image](Exported%20image%2020251206150125-0.png)
 2.报文内容  
1.hello报文

![Exported image](Exported%20image%2020251206150127-1.png)
 对于interface id字段，就是链路接口的描述值  
同一台设备的每一个接口值都是不同的  
2.DD报文：没有变化  
3.LSR报文：没有变化  
4.LSU报文：没有变化  
5.LSack报文：没有变化

OSPFv3的LSA的信息：  
1.LSA的头部

![Exported image](Exported%20image%2020251206150132-2.png)
 link state id：  
1、3、5、7、9类LSA都是使用本地唯一的32位整数表示  
2类LSA 使用DR的接口id进行标识  
4类LSA 使用ASBR的RID进行标识  
8类LSA 使用接口ID做标识  
link type：  
扩展了字段的位数，现由 Ubit + S2/S1 + function code组成  
U bit：取值为0，将类型未知的LSA当作链路本地范围进行处理  
取值为1，按照S2/S1bit表示的泛洪范围进行处理  
S2/S1：表示LSA的泛洪范围  
取值为00，表示本地链路的泛洪范围  
取值为01，表示整个区域的泛洪范围  
取值为10，表示整个自治系统的泛洪范围  
取值为11，保留  
function code：表示LSA类型  
**如果是function code表示的已知LSA，则Ubit没有作用，S2/S1按照已知LSA的泛洪范围进行置位

![Exported image](Exported%20image%2020251206150134-3.png)
 2.LSA的内容  
1.1类LSA：作用类似，但是只描述拓扑信息，不描述路由信息

![Exported image](Exported%20image%2020251206150138-4.png)
 2.2类LSA：作用类似，但是只描述拓扑信息，不描述路由信息

![Exported image](Exported%20image%2020251206150141-5.png)
 ***1类、2类LSA不包含路由信息，所以完成了拓扑路由分离的操作，即路由的变化不会影响拓扑的重新计算
 
3.3类LSA：作用类似，名称改变为inter-area-prefix-lsa（区域间前缀LSA）

![Exported image](Exported%20image%2020251206150143-6.png)  

4.4类LSA：作用类似，名称改变为inter-area-router-lsa（区域间路由器LSA）

![Exported image](Exported%20image%2020251206150148-7.png)
   
5.5类LSA：作用类似，功能相同

![Exported image](Exported%20image%2020251206150150-8.png)
 
7.7类LSA：作用类似，功能相同  
和5类LSA内容完全相同

**OSPFv3新增的LSA：**  
1.8类LSA：名称Link-LSA（链路LSA）  
泛洪范围是链路泛洪  
ospfv3设备在所有使能的接口上都会产生8类LSA  
8类LSA用于描述接口的链路本地地址（该地址作为路由计算的下一跳使用）  
8类LSA会携带自身接口的所有网络前缀  
8类LSA携带接口网络前缀的目的？  
为了本端接口配置的网络前缀可以被网段的DR得知
 
2.9类LSA：名称Intra-Area-Prefix-LSA（区域内前缀LSA）  
用于描述区域内的路由信息  
1.末节路由（环回口路由） （v2通过1类LSA的stubnet描述）  
对于末节路由，9类LSA每一台设备都会产生  
会使用借鉴字段，对应到拓扑节点（设备的1类LSA）
 
2.连接路由（接口网段路由）（v2通过2类LSA的netmask描述）  
对于连接路由，9类LSA由DR设备产生  
会使用借鉴字段，对应到拓扑节点（设备的2类LSA）
 
如果直连网段下的多台设备，网络前缀配置不同  
DR设备要先获取到非DR设备的8类LSA，根据8类LSA携带的路由信息  
才能生成对应的9类LSA
 
对于9类LSA，如果要在不同区域转发  
ABR设备收到本区域内的9类LSA，会转换为3类LSA在区域泛洪

![Exported image](Exported%20image%2020251206150158-9.png)

```
\<AR5\>dis ospfv3 lsdb 

* indicates STALE LSA

           OSPFv3 Router with ID (10.5.5.5) (Process 1)
               Link-LSA (Interface GigabitEthernet0/0/0)

Link State ID   Origin Router    Age   Seq#       CkSum  Prefix
0.0.0.3         10.5.5.5         1550  0x80000001 0x6233      0
0.0.0.3         10.6.6.6         1519  0x80000001 0x632e      0

               Router-LSA (Area 0.0.0.0)

Link State ID   Origin Router    Age   Seq#       CkSum    Link
0.0.0.0         10.5.5.5         1469  0x80000005 0x35aa      1
0.0.0.0         10.6.6.6         1478  0x80000005 0x23b8      1

               Network-LSA (Area 0.0.0.0)

Link State ID   Origin Router    Age   Seq#       CkSum
0.0.0.3         10.6.6.6         1479  0x80000001 0x5480

               Inter-Area-Prefix-LSA (Area 0.0.0.0)

Link State ID   Origin Router    Age   Seq#       CkSum
0.0.0.1         10.6.6.6         1406  0x80000001 0x20d1

               Inter-Area-Router-LSA (Area 0.0.0.0)

Link State ID   Origin Router    Age   Seq#       CkSum
10.7.7.7        10.6.6.6         0623  0x80000001 0x5b70
```
   

```
               AS-External-LSA

Link State ID   Origin Router    Age   Seq#       CkSum  Type
0.0.0.1         10.7.7.7         0624  0x80000001 0x8f2c E2
```