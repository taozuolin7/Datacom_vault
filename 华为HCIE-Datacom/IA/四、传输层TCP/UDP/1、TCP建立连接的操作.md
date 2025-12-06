\> [!caution] This page contained a drawing which was not converted.   

```
TCP的处理机制：  
1.三次握手  
2.数据传递  
3.四次挥手  
4.滑动窗口机制
```

```
Byte：字节  
数据大小的计量单位
```
 
```
Bit：比特
```
 
```
1 字节（B） = 8 比特（b）
```
 
```
1个bit可以使用 0 或 1 表示  
2个bit可以用 00 、01 、10 、11 表示
```

```
三次握手：主要是为了建立一个面向连接的操作
```

```
TCP  
seq：100（随机值）     
ack：0  
SYN bit （我想和你建立连接）
```

```
seq是上一个报文的ack  
ack是上一个报文的seq+1
```

```
TCP  
seq：200（随机值）  
ack：100+1  
ACK bit（我同意和你建立连接）  
SYN bit（我想和你建立连接）
```

```
TCP  
seq：101  
ack：200+1  
ACK bit（我同意和你建立连接）
```

![Exported image](Exported%20image%2020251206145713-0.png)

![Exported image](Exported%20image%2020251206145718-1.png)   

```
数据传递：客户端和服务器之间交互数据信息
```

![Exported image](Exported%20image%2020251206145720-2.png)

```
seq是上一个报文的ack  
ack是上一个报文的seq+数据载荷的长度
```

```
传100字节
```

```
返还200字节
```

```
TCP  
seq：101  
ack：201  
数据载荷 100字节
```

```
TCP  
seq：201  
ack：101+100  
数据载荷 100字节
```

```
TCP  
seq：201  
ack：201+100  
数据载荷 0字节
```

```
TCP  
seq：301  
ack：201+0  
数据载荷 100字节
```

```
TCP  
seq：201  
ack：301+100  
数据载荷 0字节
```

```
bit 置位 表示该bit为1  
bit不置位表示该bit为0
```

```
ｗin=100Ｂｉｔｅ
```

```
四次挥手：数据传输完毕后，客户端和服务器断开TCP连接
```

```
seq是上一个报文的ack  
ack是上一个报文的seq+1
```

```
TCP  
seq：201  
ack：301+100  
FIN bit（我想和你断开连接）  
ACK bit
```

```
TCP  
seq：401  
ack：201+1  
ACK bit 我同意和你断开连接
```

```
TCP  
seq：401  
ack：201+1  
FIN bit（我想和你断开连接）  
ACK bit
```

```
TCP  
seq：202  
ack：401+1  
ACK bit 我同意和你断开连接
```

![Exported image](Exported%20image%2020251206145721-3.png) ![Exported image](Exported%20image%2020251206145722-4.png)    

```
TCP滑动窗口机制：
```

![Exported image](Exported%20image%2020251206145724-5.png)  

```
 解释发送方 Win 字段未变化的原因:
```

```
发送方（PC1）在接收到接收方（PC2）的确认报文（Ack = 104，win = 1）之前，已经按照初始协商的窗口大小（win = 3）组装好了下一个要发送的报文（seq = 104，win = 3）。
```

```
TCP 发送方在发送报文时，窗口大小是根据其本地的发送窗口状态来设置的，而不是实时根据接收方最新的窗口通告来动态调整正在发送的报文的窗口字段。
```

```
只有当发送方收到接收方的窗口通告（即包含新的 win 值的 ACK 报文）后，在后续组装新的报文时，才会根据新的窗口大小来设置 Win 字段。而图中 seq = 104 的报文在发送时，还没有来得及根据 PC2 最新的窗口通告（win = 1） 来调整，所以其 Win 字段仍然是 3。
```