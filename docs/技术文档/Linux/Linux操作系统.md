
## Ubuntu

[Ubuntu-获取镜像](https://cn.ubuntu.com/download)

[MSDN-获取镜像](https://msdn.itellyou.cn/)

镜像使用建议：

- 1.虚拟机：使用VMware安装Ubuntu虚拟机，软件密匙网上有。使用镜像文件安装虚拟机即可。

    网络：不需要担心网络问题，虚拟机将使用虚拟网卡就行网络连接

- 2.本地安装：将iso文件刻录在U盘（UltraISO），并且进入电脑BIOS作为启动盘进行安装。

    推荐使用自动安装，而不是手动分区，手动容易产生不必要的麻烦和无法解决的问题，消耗时间。所以建议单独出一个硬盘来安装

    网络：需要网络适配器。先用以太网有线网络连接，或者USB热点。再更新需要的网路适配器，才能使用无线网络

## 常用配置

- 更换软件源：

Setting-> about-> Software updates-> Download From-Other-> Select Best Server

## Linux 命令

    chmod xxx
    修改文件权限
    xxx分别为 owner权限 group权限 others权限
    例如777，即-rwxrwxrwx，7表示二进制111，分别使rwx置为1有权限
    其他编码使用二进制类推

    apt list --installed
    显示系统中所有已安装软件包的列表，包括软件包的名称和版本等信息

    service network-manager restart
    重启网卡

## Shell脚本

shell脚本是类似Windows的batch批处理脚本

shell是纯文本文件，命令从上而下，逐行执行。shell脚本文件命名为***.sh

文本的首行为：`#!/bin/bash`,表示使用bash

## Makefile

[廖雪峰-makefile](https://liaoxuefeng.com/books/makefile/introduction/index.html)

*在Linux环境下，当我们输入make命令时，它就在当前目录查找一个名为Makefile的文件，然后，根据这个文件定义的规则，自动化地执行任意命令，包括编译命令。*

## Linux 下文件传输

1. FTP文件传输 + MobaXterm
2. SSH + MobaXterm
3. Samba

    samba是通过网络来进行windows和ubuntu互传文件的

    所以我们必须保证windows和ubuntu直接可以互相ping通。

## Linux Tools

[MobaXterm](https://mobaxterm.mobatek.net/)

- SSH连接虚拟机：

    1 请打开虚拟机所有网卡,确保windows可以ping成功

    2 使用SSH和虚拟机ip登录，如果refuse ，请安装openssh-server。

    指令：sudo apt-get install openssh-server

    3 apt安装如果报错，请尝试重启，或者后台终结进程
