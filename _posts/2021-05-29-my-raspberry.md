---
layout: post
title:  "我的树莓派"
date:   2021-05-29
categories: ['raspberry']
---

最近折腾了下**4b**从[usb启动](https://www.raspberrypi.org/documentation/hardware/raspberrypi/bootmodes/msd.md), 顺便重装了下系统, 重新配置了各种服务, 在此做个记录.


安装系统

这里选择了最小化安装的[Manjora ARM](https://manjaro.org/download/#raspberry-pi-4-minimal)
使用[Raspberry Pi Imager](https://www.raspberrypi.org/downloads)制作好启动盘.
在启动分区创建**ssh**文件, 以确保ssh开启, 等启动后 **ssh root@IP** 跟随指引进行配置即可.

一切就绪后, 登录配置国内源

```
sudo pacman-mirrors -c China
```

升级系统到最新
```
sudo pacman -Syu
sudo reboot
```

安装一些常用软件

```
sudo pacman -S vim bash-completion fzf htop zsh screen wget iotop yay
```

安装一些开发工具
```
sudo pacman -S github-cli rbenv docker docker-compose base-devel golang
```

安装监控软件

之前安装的**zabbix**这次试试看**prometheus-node-exporter**.

**grafana** 提供了一个非常好用的**node exporter**的dashboard, 整体配置也比较简单.

```
yay pacman -S grafana-bin
sudo pacman -S prometheus prometheus-node-exporter
```
效果如下:
![grafana](/static/img/posts/node-exporter-grafana.png "grafana")

https://grafana.com/grafana/dashboards/1860

配合**vscode** **remote-ssh** 做一些轻量的开发还是挺不错的.