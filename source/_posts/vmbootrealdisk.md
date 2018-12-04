---
title: VMware虚拟机启动实际硬盘分区的Ubuntu, EFI+GPT
date: 2018-10-14 16:13:40
mathjax: true
tags: [VMware, 虚拟机]
---


新建虚拟机，建立一个虚拟SATA硬盘，添加实际硬盘，选择Ubuntu所在分区，设置虚拟硬盘为SATA0:2，实际硬盘为SATA0:0

使用Ubuntu安装光盘启动系统， 格式化硬盘

```sh
fdisk /dev/sdb

# 建立ESP分区(类型1) 和Linux分区
# AAAABBBB是原来系统里fstab里挂载EFI分区使用的UUID，形如AAAA-BBBB
mkfs.msdos -F 32 -i AAAABBBB /dev/sda1

msfs.ext4 /dev/sda2

mount /dev/sda2 /mnt

```

安装grub for EFI系统

```sh
sudo apt install grub-efi-amd64

grub-install --boot-directory=/mnt --target=x86_64-efi /dev/sdb

```


重启虚拟机系统，GRUB2手动引导

```sh
# (hd0,gpt5)是根分区或者boot分区（如果有）, /dev/sda5 是根分区
root=(hd0,gpt5)
linux /boot/vmlinux-xxxx root=/dev/sda5
initrd /boot/initrd-xxxx
```

进入系统后，设置GRUB2引导配置

```sh
mount /dev/sdb2 /mnt
cd /mnt/grub
sudo grub-mkconfig -o grub.cfg
```

重启虚拟机系统可以正常进入系统。



