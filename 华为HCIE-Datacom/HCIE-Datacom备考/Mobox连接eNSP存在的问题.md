通过Telnet连接上后，会出现按下回车显示\<M\<M\<M\<M,按下Tab会显示\<H\<H\<H
 
1.首先关闭所有mobx  
2.通过修改mobox根路径的ini文件，  
在文件顶上添加：  
[MottyOptions]  
LocalEcho=1  
LocalEdit=1
 
保存后，重新打开Mobx就行了