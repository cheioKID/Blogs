---
date: 2018-07-02
title: "Install more than one Kernel for Ubuntu Linux"
tags:
    - Linux
    - Ubuntu
categories:
    - CS
comment: true
---

Ubuntu 内核 

修改源
```bash
sudo cp /etc/apt/sources.list /etc/apt/sources.list_bak
sudo vim /etc/apt/sources.list
deb http://security.ubuntu.com/ubuntu trusty-security main
sudo apt-get update
```

安装内核3.16
```bash
sudo apt-get install linux-image-extra-3.16.0-43-generic
```

查看内核是否安装
```bash
dpkg -l | grep 3.16.0-43-generic
```

修改grub
```bash
sudo gedit /etc/default/grub
```


将`GRUB_DEFAULT=0`替换为
```bash
GRUB_DEFAULT="1>3"
#一级菜单选项1下的第4个选项
#默认选择
```

注释这一行
```bash
#GRUB_HIDDEN_TIMEOUT=0
#注释这一行
```

更新grub
```bash
sudo update-grub
sudo reboot
```