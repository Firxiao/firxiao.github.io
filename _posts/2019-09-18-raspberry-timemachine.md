---
layout: post
title:  "树莓派4 时间机器(Time Machine)"
date:   2019-09-18 20:00:00 +0800
categories: ['raspberry']
---

之前一直用树莓派3, 迫于`usb2.0`和`百兆网卡`, `NAS`性能捉衣见肘.  
树莓派4终于迎来了`usb3.0`和`千兆网卡`. 测试了下实际性能, 几乎可以跑满千兆网.  
用来做Mac的时间胶囊感觉足矣, 可以省去插硬盘备份的烦恼.  
在此记录下配置`Time Machine`的步骤, 系统使用的是官方的`Raspbian`.

## 系统信息

```bash
pi@raspberrypi:~ $ screenfetch
    .',;:cc;,'.    .,;::c:,,.    pi@raspberrypi
   ,ooolcloooo:  'oooooccloo:    OS: Raspbian 10 buster
   .looooc;;:ol  :oc;;:ooooo'    Kernel: armv7l Linux 4.19.66-v7l+
     ;oooooo:      ,ooooooc.     Uptime: 14h 55m
       .,:;'.       .;:;'.       Packages: 2141
       .... ..'''''. ....        Shell: 14429
     .''.   ..'''''.  ..''.      CPU: ARMv7 rev 3 (v7l) @ 4x 1.5GHz
     ..  .....    .....  ..      GPU: DRM
    .  .'''''''  .''''''.  .     RAM: 371MiB / 1507MiB
  .'' .''''''''  .'''''''. ''.
  '''  '''''''    .''''''  '''
  .'    ........... ...    .'.
    ....    ''''''''.   .''.
    '''''.  ''''''''. .'''''
     '''''.  .'''''. .'''''.
      ..''.     .    .''..
            .'''''''
             ......
```

## 配置步骤

### 准备硬盘

插上硬盘后,通过`lsblk`查看新硬盘盘符.  
如下所示我的硬盘是`sdb`

```bash
pi@raspberrypi:~ $ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sdb           8:16   0  1.8T  0 disk
mmcblk0     179:0    0 59.5G  0 disk
├─mmcblk0p1 179:1    0  256M  0 part /boot
└─mmcblk0p2 179:2    0 59.2G  0 part /
```

分区的话,推荐使用 `gparted`, `parted`的图形化版本.
安装`gparted`命令如下.

```bash
sudo apt update
sudo apt install gparted
```

安装完成后可以在系统工具中找到, 或者直接在终端中敲`gparted`打开.  
如图所示, 我创建了一个1T的分区`sdb1`.
![gparted](/static/img/posts/gparted.png "gparted")

创建目录`/shared/TimeMachine`

```bash
sudo mkdir -p /shared/TimeMachine
```

添加下面一行到`/etc/fstab`

```bash
 /dev/sdb1    /shared/TimeMachine ext4    defaults,noatime 0 1
```

挂载并检查分区

```bash
sudo mount -a
df -hP|grep shared
/dev/sdb1            984G   72M  934G    1% /shared/TimeMachine
```

### 安装配置`netatalk`

安装软件包

```bash
sudo apt-get update
sudo apt-get install netatalk avahi-daemon
```

因为这里要使用用户`pi`来进行登录, 所以直接把刚才挂载好的目录赋予给它.  
如果你用的是其他用户 你需要赋予用户写入`/share/TimeMachine`的权限.

```bash
sudo chown pi.pi /shared/TimeMachine/
```

把下面几行添加到`/etc/netatalk/afp.conf`

```conf
 [My Time Machine Volume]
 path = /shared/TimeMachine
 time machine = yes
```

重启服务并设置为开机自动启动

```bash
sudo systemctl restart netatalk
sudo systemctl enable netatalk
```

至此, 配置就完成了.

接下来在Mac的`Time Machine`中`选择磁盘`就会发现刚创建好的卷.  
选择使用该卷, 然后使用`pi`登录即可.

#### 参考:

[1] <http://netatalk.sourceforge.net/3.1/htmldocs/>
