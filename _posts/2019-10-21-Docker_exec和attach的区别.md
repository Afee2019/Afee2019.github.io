# Docker exec和Docker attach的区别

1. attach 直接进入容器 启动命令 的终端，不会启动新的进程。
2. exec 则是在容器中打开新的终端，并且可以启动新的进程。
3. 如果想直接在终端中查看启动命令的输出，用 attach；其他情况使用 exec。



不论是开发者是[运维](https://www.baidu.com/s?wd=运维&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)人员，都经常有需要进入容器的诉求。
目前看，主要的方法不外乎以下几种：

1. 使用ssh登陆进容器
2. 使用nsenter、nsinit等第三方工具
3. 使用docker本身提供的工具

方法1需要在容器中启动sshd，存在开销和攻击面增大的问题。同时也违反了Docker所倡导
的一个容器一个进程的原则。
方法2需要额外学习使用第三方工具。
所以大多数情况最好还是使用Docker原生方法，Docker目前主要提供了Docker exec和
Docker attach两个命令。

以下在fedora21，docker1.7上验证。

Docker attach

Docker attach可以attach到一个已经运行的容器的stdin，然后进行命令执行的动作。
但是需要注意的是，如果从这个stdin中exit，会导致容器的停止。

[root@localhost temp]# docker ps
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
2327e7eab0ed busybox:buildroot-2014.02 "/bin/sh" About a minute ago Up About a minute bb2
[root@localhost temp]# docker attach bb2
/ # ls
bin dev etc home lib lib64 linuxrc media mnt opt proc root run sbin sys tmp usr var
/ # pwd
/
/ #

Docker exec

关于-i、-t参数

可以看出只用-i时，由于没有分配伪终端，看起来像pipe执行一样。但是执行结果、命令
返回值都可以正确获取。

[root@localhost temp]# docker exec -i bb2 /bin/sh
date
Tue Jul 14 04:01:11 UTC 2015
echo $?
0
dir
/bin/sh: dir: not found
echo $?
127

使用-it时，则和我们平常操作console界面类似。而且也不会像attach方式因为退出，导致
整个容器退出。
这种方式可以替代ssh或者nsenter、nsinit方式，在容器内进行操作。

```
[root@localhost temp]# docker exec -it bb2 /bin/sh
/ # pwd
/
/ # echo $?
0
/ # dir
/bin/sh: dir: not found
/ # echo $?
127
```



如果只使用-t参数，则可以看到一个console窗口，但是执行命令会发现由于没有获得stdin
的输出，无法看到命令执行情况。

```
[root@localhost temp]# docker exec -t bb2 /bin/sh
/ # pwd

hanging....
[root@localhost temp]# docker exec -t bb2 pwd
/
[root@localhost temp]# echo $?
0
[root@localhost temp]# docker exec -t bb2 dir
2015/07/14 04:03:57 docker-exec: failed to exec: exec: "dir": executable file not found in $PATH
[root@localhost temp]# echo $?
0
```



docker exec执行后，会命令执行返回值。（备注Docker1.3似乎有Bug，不能正确返回命令执行结果）

```
[root@localhost temp]# docker exec -it bb cat /a.sh
echo "running a.sh"
exit 10
[root@localhost temp]# docker exec -t bb /a.sh
running a.sh
[root@localhost temp]# echo $?
10
[root@localhost temp]# docker exec -it bb /a.sh
running a.sh
[root@localhost temp]# echo $?
10
[root@localhost temp]# docker exec -i bb /a.sh
running a.sh
[root@localhost temp]# echo $?
10
```



关于-d参数

在后台执行一个进程。可以看出，如果一个命令需要长时间进程，使用-d参数会很快返回。
程序在后台运行。

[root@localhost temp]# docker exec -d bb2 /a.sh
[root@localhost temp]# echo $?
0

如果不使用-d参数，由于命令需要长时间执行，docker exec会卡住，一直等命令执行完成
才返回。

```
[root@localhost temp]# docker exec bb2 /a.sh
^C[root@localhost temp]#
[root@localhost temp]#
[root@localhost temp]# docker exec -it bb2 /a.sh
^C[root@localhost temp]#
[root@localhost temp]# docker exec -i bb2 /a.sh
^C[root@localhost temp]# docker exec -t bb2 /a.sh
^C[root@localhost temp]#
```

