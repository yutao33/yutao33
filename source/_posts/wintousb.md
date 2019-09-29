---
title: 克隆磁盘或者使用硬盘盒后重置Windows系统
date: 2019-09-29 16:17:00
mathjax: false
tags: [Windows, WinToUSB]
---


错误1：蓝屏代码为0x0000000..

这种是BCD文件配置问题
可按教程修复BCD文件。

使用bootice修复BCD启动配置文件
https://bbs.luobotou.org/forum.php?mod=viewthread&tid=12540&fromuid=1

UEFI启动模式可以使用bcdboot命令：

diskpart->选择u盘efi分区 -> assign letter=x -> exit
重启explorer.exe -> X分区出现
删除X分区的efi文件夹
bcdboot E:\Windows /s X: /f UEFI



错误2：蓝屏代码为inaccessible boot device


此类是注册表问题

解决办法：加载WTG系统的System注册表，将HardwareConfig中子项清空。

过程：

打开注册表编辑器，加载配置单元（Load Hive），选择E:\WINDOWS\SYSTEM32\CONFIG\SYSTEM(E是WTG系统盘符)将HardwareConfig中两项右键删除，卸载配置单元（UnLoad Hive）



参考
* https://bbs.luobotou.org/forum.php?mod=viewthread&tid=14042&fromuid=1
