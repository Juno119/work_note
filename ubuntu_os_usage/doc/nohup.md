# linux的nohup命令的用法 
在应用 `Unix/Linux` 时，我们一般想让某个程序在后台运行，于是我们将常会用 `&` 在程序结尾来让程序自动运行。这样我们就需要 `nohup` 命令，怎样使用nohup命令呢？这里讲解 `nohup` 命令的一些用法。   
```bash
nohup /root/start.sh &
```
在 shell 中回车后提示：   
[~]$ appending output to nohup.out    
原程序的的标准输出被自动改向到当前目录下的 `nohup.out` 文件，起到了 `log` 的作用。但是有时候在这一步会有问题，当把终端关闭后，进程会自动被关闭，察看 nohup.out 可以看到在关闭终端瞬间服务自动关闭。  
咨询 Linux 工程师后，他也不得其解，在我的终端上执行后，他启动的进程竟然在关闭终端后依然运行。在第二遍给我演示时，我才发现我和他操作终端时的一个细节不同：他是在当 shell 中提示了 nohup 成功后还需要按终端上键盘任意键退回到 shell 输入命令窗口，然后通过在 shell 中输入 exit 来退出终端；而我是每次在 nohup 执行成功后直接点关闭程序按钮关闭终端。所以这时候会断掉该命令所对应的 session，导致 nohup 对应的进程被通知需要一起 shutdown 。   
这个细节有人和我一样没注意到，所以在这儿记录一下了。  
## 1. nohup命令
nohup 命令
用途：不挂断地运行命令。   
语法：nohup Command [ Arg … ] [　& ]   
描述：nohup 命令运行由 Command 参数和任何相关的 Arg 参数指定的命令，忽略所有挂断（SIGHUP）信号。在注销后使用 nohup 命令运行后台中的程序。要运行后台中的 nohup 命令，添加 & （ 表示”and”的符号）到命令的尾部。    
无论是否将 nohup 命令的输出重定向到终端，输出都将附加到当前目录的 nohup.out 文件中。如果当前目录的 nohup.out 文件不可写，输出重定向到 $HOME/nohup.out 文件中。如果没有文件能创建或打开以用于追加，那么 Command 参数指定的命令不可调用。如果标准错误是一个终端，那么把指定的命令写给标准错误的所有输出作为标准输出重定向到相同的文件描述符。   
退出状态：该命令返回下列出口值：   
126 可以查找但不能调用 Command 参数指定的命令。   
127 nohup 命令发生错误或不能查找由 Command 参数指定的命令。   
否则，nohup 命令的退出状态是 Command 参数指定命令的退出状态。   
1.2 nohup命令及其输出文件   
nohup命令：如果你正在运行一个进程，而且你觉得在退出帐户时该进程还不会结束，那么可以使用nohup命令。该命令可以在你退出帐户/关闭终端之后继续运行相应的进程。nohup就是不挂起的意思(n ohang up)。   
该命令的一般形式为：nohup command &   
使用nohup命令提交作业   
如果使用nohup命令提交作业，那么在缺省情况下该作业的所有输出都被重定向到一个名为nohup.out的文件中，除非另外指定了输出文件：   
nohup command > myout.file 2>&1 &   
在上面的例子中，输出被重定向到myout.file文件中。   
使用 jobs 查看任务。   
使用 fg %n　关闭。   
