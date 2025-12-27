# 1.IGP的防环DN 和 route-tag 防环
## 当分支PE与CE直接建立IGP邻居时：
经过超级骨干区域（VPNv4）的路由通告PE设备引入到Site中时：
**3类的LSA的路由**会将DNbit置位1，当其他PE接收到该DNbit置位1的LSA后，会自动过滤掉该LSA。
**5类的LSA**会将DNbit置位1，当其他PE接收到该DNbit置位1的LSA后，会自动过滤掉该LSA。并且会有route-tag防环。

## 当部署多PE的情况下如何解决防环问题：

针对3类LSA在另一个==设备==上或者说==另一个IGP进程==上忽略DNbit置位1的检查
针对5类LSA除了忽略DNbit置位还要对route-tag进行配置：两个PE设备配置为不同的route-tag==直接在进程下配置==
```
ospf 1
 route-tag 1
 
ospf 2 
 route-tag 2
```

# 2.BGP的解决防环 allow-as-loop 和 修改as号
由于BGP本身存在AS防环，所以说当分支PE与CE之间建立EBGP邻居时
## 当分支PE与CE之间建立EBGP邻居时：
当部署多PE的情况下如何解决防环问题：