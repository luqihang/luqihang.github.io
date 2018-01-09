---
layout:     post
title:      "Redis性能——持久化策略"
subtitle:   ""
date:       2017-12-27
author:     "luqh"
header-img: "img/post-bg-js-version.jpg"
tags:
    - Redis
---



# 持久化选项

Redis 提供两种持久化方法。分别为**快照（snapshotting）**和**只追加文件(append-only file,AOF)**。

快照：它可以将存在于某一时刻所有的数据都写入硬盘。
AOF：它会在执行写命令是，将被执行的写命令复制到硬盘中。

这两种持久化方式既可以同时使用，又可以单独使用。


## AOF持久化
AOF持久化会将被执行的写命令写入到AOF文件的末尾，因此，只要从头到尾执行一次AOF文件中的所有命令，就可以恢复AOF文件所记录的数据集。


|选项| 同步帧率|
|---|---|
|always|每条写命令都会同步写入磁盘，严重降低Redis速度|
|everysec|每秒执行一次同步，显式地将多个写命令同步到磁盘|
|no|让操作系统决定何时进行同步|

一般推荐使用erversec方式，这种方式和不使用任何持久化的性能相差无几，而且可以保证在系统崩溃后，也最多丢失一秒的数据。

## 整理AOF文件
相比于快照，AOF无论从性能还是容错性上，都高于快照，AOF就十全十美吗？当然也不是，随着Reids不断的将被执行的写命令写入AOF文件中，随着Redis不断运行，AOF的体积也会不断增长，在极端情况下，可能会耗完整个磁盘空间。而且AOF体积越大，Redis重启之后重新执行AOF的命令所需的时间就越长。

为了解决这个问题，可以执行``BGREWRITEAOF``命令，这个命令会移除AOF文件中的冗余，从而减小AOF文件的体积。``BGREWRITEAOF``命令和``BGSAVE``工作原理类似，Redis会创建一个子进程，然后由子进程负责对AOF文件进行重写，随之而来的问题是，如果不加以控制，AOF文件体积会比快照文件的体积大几倍，在AOF重写并删除旧AOF文件的时候，删除一个体积达到数十个GB的旧AOF文件可能会导致操作系统挂机数秒。

AOF文件可以自动重写，设置``auto-aof-rewrite-percentage``和``auto-aof-rewrite-min-size``选项来自动执行``BGREWRITEAOF``。
例如，``auto-aof-rewrite-percentage 100``指AOF文件比上一次重写大一倍的时候触发重写，``auto-aof-rewrite-min-size 64M``指AOF文件比上一次增加了64M的时候触发重写。


## 主从复制