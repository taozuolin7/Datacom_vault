```
FTP（File Transfer Protocol，文件传输协议）￼FTP传输数据时支持两种传输模式：
```

![Exported image](Exported%20image%2020251206145727-0.png)   
```
ASCII模式和Binary模式。  
1.ASCII模式用于传输文本文件。发送端的字符在发送前被转换成ASCII码格式之后进行传输，接收端收到之后再将其转换成字符。  
2.二进制模式常用于发送图片文件和程序文件，发送端在发送这些文件时无需转换格式即可传输。
```
 
```
cc：VRP版本文件。
```
 
```
FTP存在两种工作方式：主动模式（PORT）和被动模式（PASV）。￼（被动和主动视角位于服务器）  
1.主动模式
```
 ![Exported image](Exported%20image%2020251206145729-1.png)

```
2.被动模式
```

![Exported image](Exported%20image%2020251206145734-2.png)  

```
主动模式和被动模式建立数据连接方式完全不同，在实际使用中各有  
利弊：  
1.使用主动模式传输数据时，如果FTP客户端在私有网络中并且FTP客户端和FTP服务器端之间存在NAT设备，那么FTP服务器端收到的PORT报文中携带的端口号、IP地址并不是FTP客户端经过NAT转换之后的地址、端口号，因此服务器端无法向PORT报文中携带的私网地址发起TCP连接（此时，客户端的私网地址在公有网络中路由不可达）。  
2.使用被动模式传输数据时，FTP客户端主动向服务器端的一个开放端口发起连接，如果FTP服务器端在防火墙内部区域中，并且没有放通客户端所在区域到服务器端所在区域的主动访问，那么这个连接将无法建立成功，从而导致FTP无法正常传输。￼  
# 开启FTP服务器端功能  
[Huawei]ftp [ ipv6 ] server enable  
# 配置FTP本地用户  
[Huawei]aaa  
[Huawei]loser user-namecal-u password irreversible-cipher password
```
 
```
[Huawei]local-user user-name privilege level level  
[Huawei]local-user user-name service-type ftp  
[Huawei]local-user user-name ftp-directory directory  
# 必须将用户级别配置在3级或者3级以上，否则FTP连接将无法成功。  
￼TFTP-Trivial File Transfer Protocol简单文件传输协议）  
相较于FTP，TFTP的设计就是以传输小文件为目标，协议实现  
使用UDP进行传输（端口号69）  
无需认证  
只能直接向服务器端请求某个文件或者上传某个文件，无法查看服务器端的文件目录。￼
```

![Exported image](Exported%20image%2020251206145736-3.png) ![Exported image](Exported%20image%2020251206145737-4.png)  

```
下载:  
\<Huawei\> tftp TFTP_Server-IP-address get filename  
TFTP无需登录，直接输入服务器端IP地址以及操作命令即可。  
上传:  
\<Huawei\> tftp TFTP_Server-IP-address put filename
```