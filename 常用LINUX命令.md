## 常用LINUX命令

nohup   不挂断的执行某个命令，即在终端退出后，命令执行不会被关闭

2>&1  将错误输出重定向到标准输出

2 > &1 & 最后一个&的意思是，在后台执行



netstat -tunlp   查看端口占用情况

kill -9 4118    杀死PID为4118的进程



## LINUX命令状态码

LINUX SHELL下的每个命令都使用退出状态码(exit status)，来告诉SHELL它完成了处理，退出状态码是一个0-255之间的整数值，在命令结束时由命令传递给SHELL，用户可以捕捉这个值在脚本中使用

LINUX使用了`$?` 专属变量来保存上个执行的命令的退出状态码，在需要查看的命令执行完后，马上查看`$?`变量，它的值就是SHELL中执行的最后一条命令的退出状态码，如

`[root@yogurtbee]# pwd`

`[root@yogurtbee]# /usr/home`

`[root@yogurtbee]# echo $?`

`[root@yogurtbee]# 0`



状态码的含义：

```xml
0     命令成功结束
1     通用未知错误
2     误用shell命令
126   命令不可执行
127   没找到命令
128   无效退出参数
128+x Linux信号x的严重错误
130   Linux信号2的严重错误，即命令通过SIGINT（Ctrl+C）终止
255   退出状态码月结
```

