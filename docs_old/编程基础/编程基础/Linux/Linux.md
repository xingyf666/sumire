# Linux

## 虚拟机安装

### 基础配置

使用 Virtual Box 安装 Ubuntu 系统，由于目前已经有之前用过的虚拟机 `.vdi`  文件，因此可以直接加载。点击新建，然后选择专家模式

![](image-20231210200934782.png)

指定 Name 和文件夹后，选择使用已存在的硬盘文件，点击 Finish 即可。

![](image-20231210201003971.png)



现在配置内存

![](image-20231210201104411.png)

以及共享文件夹位置及其挂载点

![](image-20231210201154458.png)



### 开启虚拟设置

点击启动后，如果出现 `Not in a hypervisor partition (HVP=0) (VERR_NEM_NOT_AVAILABLE).` 这样的错误，窗口闪退，说明没有开启虚拟技术。需要在 Windows 设置-更新和安全-恢复，这一栏选择立即重新启动

![](image-20231210201441477.png)

然后按顺序点击下面的选项

![](20200312155000200.png)

现在会重启进入 BIOS 界面，上方可以选择切换语言为中文。然后在高级-CPU 设置菜单下，找到并启用虚拟化即可。



### 修改硬盘大小

在管理-工具-虚拟介质管理中可以修改硬盘分配空间。将所有虚拟分配空间都调整为最大空间

![](image-20240413205910622.png)



### 磁盘管理

可以先下载一个磁盘管理工具

```shell
sudo apt-get install gparted
```

通过 sudo 运行

```shell
sudo gparted
```

然后可以调整磁盘空间大小。中间的 swap 分区可以先禁用，然后删除。这样就可以连续扩容。

![](image-20240413210839078.png)



### 修改显卡

启动虚拟机时，可能会出现报错

```shell
*ERROR* Failed to send host log message
```

这是 Virtual Box 的 bug，通常没有影响。不过可以修改显卡设置为

![](image-20231215102018149.png)

下面会显示无效设置，不需要管直接点击 OK 设置。再次启动就不会有报错。



### 全屏显示

此时窗口只显示一小部分，进入界面后，在 Virtual Box 菜单的设备栏选择安装增强功能，现在会在终端中进行下载配置，完成后重启即可。

![](image-20231210201802234.png)



### 配置网络

如果使用虚拟机与主机建立连接，应该在网络中添加一个新的网卡，设为桥接模式，否则 ip 地址 `10.0.2.15` 无法访问。

![](image-20231214175705378.png)

打开虚拟机，输入 ifconfig 查看

```shell
enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.2.15  netmask 255.255.255.0  broadcast 10.0.2.255
        inet6 fe80::7e0:7718:ce5f:4a33  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:9a:45:51  txqueuelen 1000  (以太网)
        RX packets 2595  bytes 1966121 (1.9 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2358  bytes 391373 (391.3 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

enp0s8: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.72.188.185  netmask 255.255.254.0  broadcast 10.72.189.255
        inet6 2001:da8:e000:7803:413:48a5:dc12:5b8d  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::6ab:c366:5c97:c9f0  prefixlen 64  scopeid 0x20<link>
        inet6 2001:da8:e000:7803:9a10:7f38:34e4:761b  prefixlen 64  scopeid 0x0<global>
        ether 08:00:27:3b:7c:90  txqueuelen 1000  (以太网)
        RX packets 8655  bytes 5696577 (5.6 MB)
        RX errors 0  dropped 109  overruns 0  frame 0
        TX packets 3142  bytes 2325210 (2.3 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (本地环回)
        RX packets 8839  bytes 40269752 (40.2 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 8839  bytes 40269752 (40.2 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```

使用 enp0s8 网卡的 ip 地址。



为了防止 ip 冲突，可以在主机命令行输入

```shell
arp -a
```

查看当前局域网可访问的 ip 地址。



现在还需要修改网络配置，在主机命令行输入

```shell
ipconfig /all
```

可以查看子网掩码 (netmask)、网关 (gateway) 和 DNS 服务器地址。在虚拟机输入

```shell
sudo gedit /etc/network/interfaces
```

进行静态配置

```yaml
# interfaces(5) file used by ifup(8) and ifdown(8)
# auto lo
# iface lo inet loopback

auto enp0s3
iface enp0s3 inet loopback

auto enp0s8
iface enp0s8 inet static
	address 10.72.188.185
	netmask 255.255.254.0
	gateway 10.72.188.1
	dns-nameservers 10.10.0.21 10.10.2.53
```

这里 `enp0s8` 是前面看到的网卡名称；static 表示静态配置，dhcp 表示动态配置，loopback 表示本地环回。可以重启网络

```shell
sudo service network-manager restart
```

我们这里同时配置了两个网卡，是因为只修改一个可能导致无法联网的情况。将 NAT 模式的网卡设为默认的 loopback，将桥接网卡设为 static，这时候右上角网络连接会显示两个网卡均未托管。



可以在主机测试连接

```cmd
ping 10.72.188.185
```



## WSL

### 安装配置

可以直接通过 powershell 安装

```shell
wsl --install --no-distribution
```

安装完成后需要重启，然后可以查看可安装的发行版本

```shell
wsl --list --online
```

选择要安装的系统

```shell
wsl --install Ubuntu
```

安装时会要求设置用户名和密码，设置后即可进入系统。

```shell
wsl --list -v
```

查看已安装的系统和默认系统，以及系统状态。



### 启动系统

命令行启动系统

```shell
wsl -d Ubuntu
ubuntu
```

退出系统可以直接关闭终端窗口或者输入

```shell
exit
```


### 卸载系统

输入命令

```shell
wsl --unregister Ubuntu
```

即可卸载对应系统。



### 备份系统

输入命令

```shell
wsl --export Ubuntu ubuntu.tar
```

即可将系统备份至一个约 `1G` 大小的压缩包中。如果要导入系统，使用

```shell
wsl --import Ubuntu2 target_dir tar_file
```

会将压缩包导入到指定文件夹下的镜像文件中。


### 文件共享

进入子系统，输入

```shell
df -h
```

即可查看所有挂载卷。可以在“此电脑”中找到 Linux 卷，打开后可以看到系统中的所有文件。并且可以在 Ubuntu 中启动 Windows 程序，修改 Ubuntu 中的文件。



在子系统中使用 

```shell
explore.exe .
```

在当前目录打开资源管理器。



在 powershell 中可以使用 wsl 命令

```shell
Get-ChildItem | wsl grep Video
```



### 显卡直通

在子系统中可以直接使用主机的显卡

```shell
nvidia-smi
```

查看显卡信息，因此可以直接使用子系统运行 AI 大模型。



### 网络配置

默认情况下，子系统使用与主机不同的网段。可以使用配置文件 `.wslconfig` 作为全局配置。在 HOME 目录下创建 `.wslconfig` 并设置

```
[wsl2]
networkingMode=mirrored
```

然后根据 8 秒配置原则，关闭虚拟机，输入

```shell
# Settings apply across all Linux distros running on WSL 2
[wsl2]

# Limits VM memory to use no more than 4 GB, this can be set as whole numbers using GB or MB
memory=8GB

# Sets the VM to use two virtual processors
processors=8

[experimental]
autoMemoryReclaim=gradual 	# 开启自动回收内存，可在 gradual, dropcache, disabled 之间选择
networkingMode=mirrored 		# 开启镜像网络
dnsTunneling=true 		# 开启 DNS Tunneling
firewall=true 			# 开启 Windows 防火墙
autoProxy=true 			# 开启自动同步代理
sparseVhd=true 			# 开启自动释放 WSL2 虚拟硬盘空间
```

等待 8 秒后重新启动。此时子系统将与主机共用 IP 。



## 双系统

### BIOS 设置

Surface pro 4 需要按住上音量键再按电源键进入 BIOS

- 将 Secure Boot 选项修改为 None
- 将 USB Storage 拖动到第一位

在 Exit 选项点击 Restart 重启，即可进入 Ubuntu 安装界面

- 在安装过程中语言选择“中文”，键盘布局选择“英文”
- 联网时选择一个无线网连接，然后可以取消掉“安装 Ubuntu 时下载更新”
- 安装类型选择“其它选项”，这样我们可以自定义分区。选择空闲的分区
	- 分配 `500 MB` 用于 EFI 系统分区
	- 分配 `10000 MB` 用于交换分区
	- 分配根挂载点分区，挂载点选择 `\`，然后分配足够的空间
	- 剩余空间分配给 `\home` 挂载点（可以全部分配给根挂载点，缺点是重装系统时个人数据会被抹除）
- 最后，将下面的“安装启动引导器的设备”切换为 EFI 分区对应的设备名，这样才能在重启时进入双系统选择

重启后进入 BIOS，将 ubuntu 调整到第一位，然后重启。现在可以进入双系统选择界面。



### 时间同步

由于 Ubuntu 和 Windows 的时间机制不同，在网络同步时间后，会修改 BIOS 中的时间，导致两个系统的时间总是出现偏差。现在安装

```shell
sudo apt install ntpdate
```

然后使用此工具同步正确的时间

```shell
sudo ntpdate time.windows.com
```

然后修改时间同步机制

```shell
sudo hwclock --localtime --systohc
```



### 修改默认启动

我们将 Windows 设为默认启动项。终端输入

```shell
sudo gedit /etc/default/grub
```

修改 `GRUB_DEFAULT` 为 Windows 启动对应的位置，然后更新设置

```shell
sudo update-grub
```



### 安装内核

```embed
title: "Installation and Setup"
image: "https://opengraph.githubassets.com/4d671f4f017349d1a400e01a270cddbef888f3555e6cbb267fcf7f1ce3a62c1d/linux-surface/linux-surface"
description: "Linux Kernel for Surface Devices. Contribute to linux-surface/linux-surface development by creating an account on GitHub."
url: "https://github.com/linux-surface/linux-surface/wiki/Installation-and-Setup"
```

安装 linux-surface 内核来获得更好的体验。依次执行

```shell
wget -qO - https://raw.githubusercontent.com/linux-surface/linux-surface/master/pkg/keys/surface.asc \
| gpg --dearmor | sudo dd of=/etc/apt/trusted.gpg.d/linux-surface.gpg

echo "deb [arch=amd64] https://pkg.surfacelinux.com/debian release main" \
| sudo tee /etc/apt/sources.list.d/linux-surface.list

sudo apt update

sudo apt install linux-image-surface linux-headers-surface libwacom-surface
```

由于网络问题，许多包不能安装。可以在网页中[查找](https://pkg.surfacelinux.com/debian/)上述包及其依赖得到所需的包

```shell
https://pkg.surfacelinux.com/debian/pool/main/l/linux-surface/linux-image-surface_6.9.3-surface-2_amd64.deb
pool/main/l/linux-upstream/linux-image-6.9.3-surface-2_6.9.3-surface-2_amd64.deb

https://pkg.surfacelinux.com/debian/pool/main/libw/libwacom-surface/libwacom-surface_2.12.2-1_amd64.deb
pool/main/libw/libwacom2-surface/libwacom2-surface_2.12.2-1_amd64.deb
pool/main/libw/libwacom9-surface/libwacom9-surface_2.12.2-1_amd64.deb
pool/main/libw/libwacom-bin-surface/libwacom-bin-surface_2.12.2-1_amd64.deb
pool/main/libw/libwacom-common-surface/libwacom-common-surface_2.12.2-1_amd64.deb
```

每个路径添加 `https://pkg.surfacelinux.com/debian/` 前缀下载即可。然后在终端输入

```shell
dpkg -i xxx.deb
```

安装完成后，更新内核

```shell
sudo update-grub
```



### 卸载系统

通过 `win + r` 输入 `msinfo32` 进入系统信息窗口，确认 BIOS 模式后开始移除。

![](image-20240930202732333.png)

进入 BIOS，然后将 Secure Boot 选项修改为 None，将 Windows 的引导程序拖动到第一位，在 Exit 选项点击 Restart 重启。



打开 Disk Genius 软件，右键将 Linux 对应的分区 4,5,6 均删除，然后点击左上角“保存更改”

![](image-20240930203035989.png)

然后重启电脑，再次打开 Disk Genius 找到 ubuntu 子目录，选择“彻底删除”所有文件

![](image-20240930203719136.png)

最后对磁盘扩容，右键“此电脑-管理”，选择磁盘管理，右键磁盘选择扩展卷，然后一直确定即可。



## 单系统

### 准备启动盘

首先，在清华镜像站下载 iso 文件

```embed
title: "Index of /archlinux/iso/2026.03.01/ | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror"
image: "https://mirrors.tuna.tsinghua.edu.cn/static/img/logo-share.png"
description: "Index of /archlinux/iso/2026.03.01/ | 清华大学开源软件镜像站，致力于为国内和校内用户提供高质量的开源软件镜像、Linux 镜像源服务，帮助用户更方便地获取开源软件。本镜像站由清华大学 TUNA 协会负责运行维护。"
url: "https://mirrors.tuna.tsinghua.edu.cn/archlinux/iso/2026.03.01/"
```

```embed
title: "Download Linux Mint 22.3 - Linux Mint"
image: "https://www.linuxmint.com/web/img/favicon.ico"
description: "Linux Mint is an elegant, easy to use, up to date and comfortable desktop operating system."
url: "https://www.linuxmint.com/download.php"
```

然后，下载 Rufus 用于安装启动盘

```embed
title: "Rufus - 轻松创建 USB 启动盘"
image: "https://rufus.ie/pics/rufus-128.png"
description: "Rufus: Create bootable USB drives the easy way"
url: "https://rufus.ie/zh/"
```

按如下方式配置，点击开始后自动安装

![](image-20260318140706631.png)



### BIOS 设置

> >在设置之前，一定要先关闭 BitLocker，不然取消 Secure Boot 以后就必须输入密钥。如果电脑已经被锁定，可以在微软官网登录账户，找到对应设备的密钥。

按电源键启动后，按住音量加键就可以进入 BIOS，然后

1. 关闭 Security 中的 Secure Boot，点击 Change config 切换 Boot 选项，这样才能从非 Windows 的 Boot 启动
2. 在 Boot config 中将 USB 拖动到第一位



### 系统配置

#### 外观设置

在 Mint 中找到“外观”，设置 DPI 和窗口缩放，在“面板”中设置图标自动调整大小。由于大部分主题不支持缩放，需要在“窗口管理器”中修改样式为 `Default-xhdpi0` 才能正常缩放标题栏。



#### 触控板

执行命令

```shell
synclient VertScrollDelta=-30
```

能够将触控板滑动效果翻转。

> >默认触控板需要按下才能点击，可以在设置中更改触摸板行为。



#### 交换内存

Linux 默认在内存使用率达到 `60%` 后使用交换空间，但其速度很慢，因此可以将阈值改为 `90%` 来避免过早启用交换空间

```shell
sudo vi /etc/sysctl.conf
```

在文件末尾添加一行

```
vm.swappiness=10
```



#### 电源管理

使用 TLP 自动优化电源管理

```shell
sudo apt install tlp
sudo tlp start
sudo tlp-stat -s
```



#### 预加载

使用 Preload 智能学习常用应用，提前加载到内存，从而加速启动

```shell
sudo apt install prelaod
```



#### 截图

安装 flameshot 截图

```shell
sudo apt install flameshot
```

然后添加快捷键，命令为 `flameshot gui`，设置相应的按键。



#### 输入法

在“输入法”中可以直接安装中文输入法，然后将标题栏中的“输入法框架”切换为"Fcitx"，重启后配置输入法为 `Sunpinyin` 就可以使用。



#### 自启动

在菜单中找到“会话和启动”，定义启动程序

- 应用名称
- 命令：可运行的程序

如果不知道应用的启动命令，使用 `which` 查找路径；如果需要延迟启动（避免与桌面环境竞争资源），可以在命令前添加 `sleep N && /usr/bin/wechat` 延迟 N 秒启动。 



#### 蓝牙

安装音频支持插件

```shell
sudo apt install pulseaudio-module-bluetooth
```



#### 安装内核

如果已经成功启用代理，则可以直接参照此 wiki 的配置方法

```embed
title: "Installation and Setup"
image: "https://opengraph.githubassets.com/4d671f4f017349d1a400e01a270cddbef888f3555e6cbb267fcf7f1ce3a62c1d/linux-surface/linux-surface"
description: "Linux Kernel for Surface Devices. Contribute to linux-surface/linux-surface development by creating an account on GitHub."
url: "https://github.com/linux-surface/linux-surface/wiki/Installation-and-Setup"
```

修改为 surface 内核以后，可以避免休眠后网络无法重新连接的问题。



#### 外观主题

在 gnome 的主题商店中搜索 `Colloid` 主题和图标下载。将主题和图标分别移动到

```
.local/share/themes
.local/share/icons
```

然后可以在“外观”中切换主题和图标。

```embed
title: "Browse Latest | https://www.gnome-look.org/browse/"
image: "https://www.gnome-look.org/stores/media/store_gnome/gnome-look-logo-2.png"
description: "Browse Latest | https://www.gnome-look.org/browse/ | A community for free and open source software and libre content"
url: "https://www.gnome-look.org/browse/"
```



#### Plank

使用 Plank 作为 dock 栏

```embed
title: "642892468"
image: "https://static.zhihu.com/heifetz/favicon.ico"
description: ""
url: "https://zhuanlan.zhihu.com/p/642892468"
```



#### Shell

使用 zsh 作为默认的 shell

```shell
sudo apt install zsh
chsh -s $(which zsh1)
```

需要下载 `oh-my-zsh` 配置

```shell
sudo apt install zsh
sh -c "$(curl -fsSL https://gitee.com/shmhlsy/oh-my-zsh-install.sh/raw/master/install.sh)"
```

然后在 `.zshrc` 中配置样式和插件。例如常用插件有

```
plugins=(git web-search vi-mode z)
```

还有第三方提示插件

```embed
title: "GitHub - zsh-users/zsh-autosuggestions: Fish-like autosuggestions for zsh"
image: "https://opengraph.githubassets.com/bb59c2e13cc336282e7dd1a1b6db5d10f05008231f5f18e5494fe329512783bd/zsh-users/zsh-autosuggestions"
description: "Fish-like autosuggestions for zsh. Contribute to zsh-users/zsh-autosuggestions development by creating an account on GitHub."
url: "https://github.com/zsh-users/zsh-autosuggestions"
```

配置好插件后，需要更新配置文件

```
source .zshrc
```



#### 快照

使用 Timeshift 保存系统快照，用于在系统崩溃时快速恢复

```embed
title: "Linux Mint 系统快照完全指南：保护你的系统安全与稳定"
image: "https://geek-blogs.com/_astro/apple-touch-icon.DSAQ9T51.png"
description: "在使用 Linux Mint 的过程中，你是否遇到过以下情况：系统突然崩溃无法启动、安装某个软件后出现兼容性问题导致桌面卡死、更新内核后硬件驱动失效……这些问题往往让新手甚至有经验的用户头疼不已。此时，**系统快照**（System Snapshot）就像一张“系统时光机”，能帮你将系统恢复到之前正常运行的状态，避免重装系统的麻烦。 本文将带你全面了解 Linux Mint 系统快照的方方面面，从基础概念到实战操作，再到高级技巧，让你轻松掌握这一“系统救星”技能。无论你是刚接触 Linux 的新手，还是希望提升系统管理能力的老用户，这篇指南都能为你提供详细的指导。"
url: "https://geek-blogs.com/blog/linux-mint-system-snapshots/"
```



#### 静态 IP（似乎无效）

设置 `/etc/netplan/` 下的一个 `01-network-manager-all.yaml` 文件

```yaml
network: 
	ethernets: 
		eth0: 
			dhcp4: false 
			addresses: [192.168.1.11/24] 
			optional: true 
			routes 
				- to: default 
				  v1a: 192.168.1.1 
			nameservers:
				addresses: [192.168.1.1] 
	version:2
```

其中 `eth0` 要改成自己的网卡名。通过 `ifconfig -a` 查看网卡。然后应用配置

```shell
sudo chmod 600 /etc/netplan/*.yaml	# 提高访问权限，只有 root 可访问
sudo netplan apply
```



#### 远程连接

使用 xrdp 实现

```embed
title: "从 Windows 远程桌面连接 Linux Ubuntu 全指南：方法、配置与最佳实践"
image: "https://geek-blogs.com/_astro/apple-touch-icon.DSAQ9T51.png"
description: "在跨平台协作日益普遍的今天，从 Windows 设备远程访问 Linux Ubuntu 系统的需求愈发常见。无论是开发者需要在 Windows 环境中操作 Ubuntu 开发服务器，系统管理员远程管理 Ubuntu 工作站，还是普通用户希望在 Windows 上运行 Ubuntu GUI 应用，远程桌面技术都能提供高效便捷的解决方案。 本文将详细介绍三种主流的远程桌面连接方法（RDP、VNC、SSH+X11 转发），涵盖从环境准备、分步配置到故障排除的全过程，并结合最佳实践和实际场景示例，帮助读者快速掌握 Windows 到 Ubuntu 的远程桌面连接技巧。"
url: "https://geek-blogs.com/blog/remote-desktop-windows-to-linux-ubuntu/"
```

安装并配置

```shell
sudo apt install xrdp -y
sudo systemctl enable --now xrdp
sudo systemctl status xrdp
```

对于 GNOME 界面还需要额外配置

```shell
echo "export GNOME_SHELL_SESSION_MODE=ubuntu" >> ~/.xsessionrc
echo "export XDG_CURRENT_DESKTOP=ubuntu:GNOME" >> ~/.xsessionrc
echo "export XDG_CONFIG_DIRS=/etc/xdg/xdg-ubuntu:/etc/xdg" >> ~/.xsessionrc
```

由于出现 `could not open X display` 错误，需要让其它 host 也可以显示

```shell
xhost +
```

> >其余的部分可以交给 AI 配置。



### 软件配置

#### Firefox

默认 Firefox 会设置阻止网页音频，可以按 `Ctrl+I` 进入“权限”界面，将最下方的“自动播放”修改为“允许音频和视频”。



#### Clash for Linux

使用 Clash for linux 便捷地在 Linux 下使用代理

```embed
title: "GitHub - nelvko/clash-for-linux-install: 😼 优雅地使用基于 clash/mihomo 的代理环境"
image: "https://opengraph.githubassets.com/a01206183a2b9501d3a27b42affdccb0ee9ecc16fc04252513b10cf9c87cea33/nelvko/clash-for-linux-install"
description: "😼 优雅地使用基于 clash/mihomo 的代理环境. Contribute to nelvko/clash-for-linux-install development by creating an account on GitHub."
url: "https://github.com/nelvko/clash-for-linux-install"
```

如果浏览器无法访问外网，可以命令行启用 TUN 模式

```shell
clashtun on
```

还可以使用 ui 界面配置，输入

```shell
clashsecret	# 查看密钥
clashui		# 打开面板，需要输入密钥
```



#### Chrome

在官网下载 Chrome 后有可能出现打开后不显示页面的问题。需要修改配置文件

```shell
sudo vim /opt/google/chrome/google-chrome
```

在最后一行追加 `--user-data-dir --no-sandbox`

```shell
exec -a "$0" "$HERE/chrome" "$@" --user-data-dir --no-sandbox
```

保存退出即可。



## 通用设置
### 输入法

在左下角进入搜索，找到语言项，进入后切换为 fctix 输入法，然后在新立得软件包中搜索 fctix，安装谷歌拼音，然后重启。在右上角点击企鹅可以配置输入法。



### 内存占用

安装 htop 工具实时查看 CPU 占用情况

```shell
sudo snap install htop
htop
```



### 端口占用

输入命令检查占用端口的程序，然后将其终止

```shell
sudo ss -tulnp | grep :8080
sudo kill -9 PID
```



### 包管理器

安装新立得包管理器

```shell
sudo apt-get install synaptic
```

然后启动

```shell
sudo synaptic
```



### 节点模式

首先安装 CPU 频率管理工具

```shell
sudo apt-get install cpufrequtils
```

然后可以查看当前 CPU 频率

```shell
cpufreq-info
```

只需要向指定文件中写入 `powersave` 选项即可

```shell
sudo -s echo powersave > /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
```



### 终端错误

在修改 Python 链接时，可能由于链接错误，导致终端无法打开。这时候通过 `Ctrl + Alt + F3` 进入命令模式。首先要登录，输入

```shell
xingyifan
123456
```

然后安装 xterm 作为临时终端

```shell
su root
sudo apt-get install xterm
```

输入 `Ctrl + Alt + F1` 回到图形界面。打开 xterm，卸载重装并重新配置终端

```shell
# Reinstalling terminall
sudo apt-get remove gnome-terminal
sudo apt-get install gnome-terminal
 
# Reconfiguring locale
sudo locale-gen --purge
sudo dpkg-reconfigure locales
 
reboot
```



### 应用自启动

按下 windows 键，搜索“启动应用程序”，然后添加启动项即可。



### 更换源

首先备份原始源

```shell
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
```

然后编译源文件

```shell
sudo gedit /etc/apt/sources.list
```

添加新的源

```shell
deb https://mirrors.zju.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb https://mirrors.zju.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb https://mirrors.zju.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb https://mirrors.zju.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
```

更新源

```shell
sudo apt-get update
```



### CMake

需要下载最新的压缩包

```embed
title: "Index of /files"
image: "https://cmake.org/icons/blank.gif"
description: ""
url: "https://cmake.org/files/"
```

解压移动后，通过软链接替换掉系统自带的 cmake

```shell
tar -xzvf cmake-3.28.3-linux-x86_64.tar.gz
sudo mv cmake-3.28.3-linux-x86_64 /opt/cmake-3.28.3
sudo ln -sf /opt/cmake-3.28.3/bin/* /usr/bin/
```



### GitHub

要能稳定访问 Github，可以查询 `github.com` 的 `DNS` 地址

```embed
title: "“${host}”A记录/cname检测结果--Dns查询|dns查询--站长工具"
image: "https://github.com/favicon.ico"
description: "通过DNS检测可以快速查出不同的地区不同的网络对你的域名解析速度，及域名DNS信息"
url: "https://tool.chinaz.com/dns/?type=1&host=github.com&ip="
```

输入后查询，选择一个合适的 IP 地址。将其写在 hosts 文件中

```shell
sudo gedit /etc/hosts
```

例如添加到最后

```text
127.0.0.1	localhost
127.0.1.1	xingyifan-VirtualBox
20.205.243.166  github.com

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

保存后重启网络配置即可

```shell
sudo service network-manager restart
```

也可以在下面两个网站中查找对应的 IP 地址

```embed
title: "GitHub.com - GitHub: Let’s build from here · GitHub"
image: "https://s.ipaddress.com/flags/circular/us.svg"
description: "GitHub is the best place to share code with friends, co-workers, classmates, and complete strangers. Over four million people use GitHub to build amazing…"
url: "https://sites.ipaddress.com/github.com/#google_vignette"
```

```embed
title: "Fastly.net - Powering the best of the internet | Fastly"
image: "https://s.ipaddress.com/flags/circular/us.svg"
description: "Fastly’s edge cloud platform delivers faster, safer, and more scalable sites and apps to customers. Elevate your edge CDN, video delivery, security, and more."
url: "https://sites.ipaddress.com/fastly.net/#ipinfo"
```

```shell
127.0.0.1	localhost
127.0.1.1	xingyifan-VirtualBox

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters


140.82.114.4 github.com
151.101.1.6 github.global.ssl.fastly.net
151.101.65.6 github.global.ssl.fastly.net
151.101.129.6 github.global.ssl.fastly.net
151.101.193.6 github.global.ssl.fastly.net
```

最好在主机上挂梯子，可能有用。



## 系统介绍

### 目录结构

| 目录 | 储存                                          |
| ---- | --------------------------------------------- |
| Bin  | 存放二进制文件                                |
| Dev  | 存放外接设备                                  |
| Etc  | 存放配置文件                                  |
| Home | 表示除了 root 用户以外其它用户的家目录        |
| Proc | 存放运行时的进程                              |
| Root | 它是 root 用户自己的家目录                    |
| Sbin | 具有 super 权限的用户可执行的文件             |
| Tmp  | 临时文件                                      |
| Usr  | 存放用户安装的软件                            |
| Var  | 存放系统的日志文件的目录                      |
| Mnt  | 当外接设备需要挂载时，就需要挂载到 mnt 目录下 |


### 权限管理

Linux 内的一切皆文件，所以对于 Linux 下文件的管理就十分的重要了。 Linux 下的文件权限分为三种：

* r（读），w（写），x（执行）

使用 chmod 修改用户权限

```shell
chmod [-可选参数][<权限范围>+/-/=<权限设置>] 文件/目录
```



指定某类用户的权限

```
chmod [ u / g / o / a ] [ + / - / = ] [ r / w / x ] file
```

其中 `[ u / g / o / a ]` 为权限范围，其中

* u：User，即文件或目录的拥有者
* g：Group，即文件或目录的所属群组
* o：Other，除了文件或目录拥有者和所属群组外，其他用户都属于这个范围
* a：All，即全部用户



权限操作

* `+` 表示增加权限
* `-` 表示取消权限
* `=` 表示取消之前的权限，并给予唯一的权限



权限代号

* r：读取权限，数字代号为 “4”
* w：写入权限，数字代号为 “2”
* x：执行权限，数字代号为 “1”
* -：不具备任何权限，数字代号为 “0”



例如给 User 用户增加对 `/code/readme. Txt` 文件 “w” 和 “x” 的权限

```shell
sudo chmod u+rw /code/readme.txt
```



同时指定三类用户的权限

```shell
chmod [xyz] file
```

其中 `x，y，z` 分别指定 `User, Group, Other` 的权限；用三位二进制数表示 `r, w , x`（注意顺序）三种权限，其中 0 代表没有该权限，1 代表有该权限，如 100 则表示，有 `r` 权限，无 `w, x` 权限；再将这个三位的二进制数转为十进制，则是 x (或 y z ) 的值

```shell
sudo chmod 774 /code/readme.txt
```

User : 7 = 111 表示具有 `r, w , x` 权限
Group : 7 = 111 表示具有 `r, w , x` 权限
Other : 4 = 100 表示只具有 `r` 权限，而没有 `w, x` 权限



```shell
sudo chmod 774 *
```

其中 `*` 为通配符，表示对当前所在目录下的所有文件做权限修改操作



```shell
sudo chmod -R 774 /code/
```

修改这个目录，以及子目录下文件的权限



| 参数 | 参数说明                         |
| ---- | -------------------------------- |
| -c   | 当发生改变时报告处理信息         |
| -f   | 错误信息不输出                   |
| -R   | 处理指定目录及子目录下的所有文件 |
| -v   | 运行时显示详细处理信息           |



### 重定向及管道

Linux 用 `<` 和 `>` 符号来进行输入/输出的重定向，通常是表示将右侧/左侧的内容作为程序的输入/输出  

```shell
./test < file 				# 将 file 内容作为标准输入
./test << 标识符 			  # 从标准输入中读取内容直到遇到标识符
./test > file 				# 以覆盖方式输出到正确信息到 file
./test >> file 				# 以追加方式输出到正确信息到 file
./test < file1 > file2 		# 将 file1 作为 test 程序的输入，结果输出到 file2
```



管道操作符 `|` 用于临时存储数据，它仅能处理经由前面一个指令传出的正确输出信息，错误信息会被忽略；与输入/输出重定向不同的是它能将多个指令连接起来执行，而不是单纯地操作一个指令的输入/输出。例如：

```shell
# 将 test 程序的输出作为 main 程序的输入
./test | ./main
```

可以通过自定义管道来建立不同命令之间的连接

```shell
mkfifo name 	# 创建管道
rm name 		# 删除管道
```

我们把管道当做一个临时储存即可，例如

```shell
mkfifo pipe 	# 创建管道 pipe
ls > pipe 		# 命令输入到 pipe 中
car < pipe 		# 命令从 pipe 中输出， pipe 被清空
```



## 命令操作

### 通用命令

| 命令         | 作用                                | 命令         | 作用                |
| ---------- | --------------------------------- | ---------- | ----------------- |
| `ifconfig` | 获得本机 `ip` 。在 `inet` 后面跟着的就是 ip 地址 | `sudo`     | 进入 root 模式        |
| `reboot`   | 重启                                | `shutdown` | 立即关机              |
| `time`     | 计算程序运行时间                          | `touch`    | 创建文件              |
| `tree`     | 查看当前目录结构                          | `pwd`      | 显示当前目录            |
| `rmdir`    | 移除目录                              | `ll`       | 列出目录下所有文件和目录的详细信息 |
| `df -h`    | 查看磁盘使用情况                          | `free -h`  | 查看内存使用情况          |
| `top`      | 动态查看内存占用                          | `lscpu`    | 查看处理器信息           |



#### ps

查询进程信息

```shell
ps -aux 					# 查询进程
ps -aux | grep name			# 查看某个进程的详细信息
sudo kill PID				# 根据 PID 杀死进程
```



#### snap

用于查看安装和更新软件的情况

```shell
snap changes				# 用于查看安装软件的情况
snap abort ID				# 放弃某个软件的安装，用于解除安装了一半的软件
```



#### man

查看手册，这里 man 似乎调用的是软件自身的手册，因此在查看外部软件的使用帮助时非常有用

```shell
man command 				 # 查看命令的手册（q退出）
```



#### cp

```shell
cp file1 file2 				 # 复制 file1 为 file2
cp -r dir1 dir2 			 # 递归地复制，用于复制文件夹
```



#### mv

```shell
mv file dir 				# 移动文件到目录
mv folder dir				# 移动文件夹到目录
```



#### rm

```shell
rm file 					# 删除文件
rm -rf dir 					# 删除文件夹
```



#### mkdir

创建目录

```shell
mkdir name 						# 创建目录
mkdir -p a/b/c					# 创建多层不存在的目录
```



#### cd

进入目录

```shell
cd / 						# 进入根目录
cd name 					# 进入 name 目录
cd ~ 						# 进入上一次列出文件的目录
cd .. 						# 返回上一级目录
cd ../ 						# 进入相对路径
```



#### ls

列出目录下文件

```shell
ls name 					# 列出目录下所有文件
ls -a 						# 列出所有文件包括隐藏文件
ls -lh 						# 列出所有文件详细信息
```



#### ldd

查看程序链接的动态库

```shell
ldd example
```



#### apt

用于安装软件

```shell
sudo apt update						# 检查是否有新的软件版本
sudo apt-get install name		 	# 安装指定的软件
sudo apt-get remove name			# 删除指定的软件
sudo apt-get --fix-broken install	# 修复依赖关系
```

如果 apt 异常中止，可能会被锁定，这时候需要解锁。可以指定任何 apt 操作，然后查看锁定文件位置，然后移除。例如

```shell
sudo rm /var/lib/dpkg/lock-frontend
sudo rm /var/lib/dpkg/lock
sudo rm /var/cache/apt/archives/lock
```



#### tar

用于解压/压缩文件

```shell
.tar 						# 用 tar -xvf 解压
.gz 						# 用 gzip -d 解压
.xz 						# 用 tar -xf 解压
.tar.gz/.tgz 				# 用 tar -zxvf 解压
.rar 						# 用 unrar e 解压
.zip 						# 用 unzip 解压
```

压缩命令

```shell
tar -zcvf file.tar.gz /file 	# 解压 file 为 .tar.gz
zip -q -r file.zip file/ 		# 压缩 file 为 .zip
```



#### dpkg

用于安装软件包 .deb

```shell
sudo dpkg -i xxx.deb
```

卸载软件包

```shell
sudo dpkg -r xxx
```



#### find

用于查找文件

```shell
find dir -name "xx.txt"	# 在 dir 下查找名为 xx.txt 的文件，可以使用 *.txt 通配符
find dir -type d		# 在 dir 下查找目录类型
find dir -type f		# 在 dir 下查找文件类型
find dir -size +100M	# 在 dir 下查找大于 100M 的文件
find dir -size -10k		# 在 dir 下查找小于 10k 的文件
find dir -mtime -7		# 在 dir 下查找 7 天内修改的文件
find dir -atime -1		# 在 dir 下查找 1 小时内修改的文件
```

上述参数可以组合使用。还可以对找到的文件进行操作

```shell
find dir -name "*.log" -type f -delete			# 删除找到的文件
find dir -name "*.sh" -exec chmod +x {} \;		# 执行找到文件，在 -exec 后接 shell 命令
```



#### objdump

用于查看程序信息，例如查看汇编代码

```shell
objdump --help 		# 查看使用帮助
objdump -S example 	# 查看源码和汇编的混合
objdump -D example	# 查看反汇编
```



### [自定义命令](https://blog.csdn.net/liuhaiquan123521/article/details/92792962)

添加自定义命令打开程序

```shell
# step 1：创建软连接放到 /usr/bin 或者 /usr/sbin 目录下
cd /usr/sbin
ln -s 程序路径
# step 2 ：添加到环境变量中
sudo gedit /etc/profile
# 在最后添加
PATH=程序所在目录:$PATH
export PATH

# step 3：使环境变量生效
source /etc/profile
```




## 编译调试

### g++

我们简单地展示编译命令的写法，以编译 file.cpp 生成 file.exe 为例

```shell
g++ [options] (-std=c++11) file file.cpp
```

**options:** 

| 选项 |           作用           |      选项       |                        作用                        |
| :--: | :----------------------: | :-------------: | :------------------------------------------------: |
| `-w` |       关闭警告信息       | `-I ../include` |          添加头文件搜索目录（ include ）           |
| `-c` |     编译生成 .o 文件     |      `-l`       |              指定库文件（ libaray ）               |
| `-o` | 编译 .o 文件为 .exe 文件 |      `-L`       |                 指定库文件搜索目录                 |
| `-g` |      开启 gdb 调试       |     `-Wall`     |                 输出 warning 信息                  |
| `-E` |      输出预处理结果      |      `-D`       | 在编译时定义宏，该宏可以在代码中使用，宏值默认为 1 |
| `-S` |       生成汇编代码       |    `-O -O2`     |                      优化编译                      |

示例代码

```cpp
// -Dname 定义宏 name ，默认定义内容为字符串 1

#include <stdio.h>

int main()
{
    #ifdef DEBUG
    	printf("DEBUG LOG\n");
    #endif
    	prinf("in\n");
}

// 在编译时使用 g++ -DDEBUG main.cpp 则会输出 DEBUG LOG
```



我们可以查看一个 `.cpp`  文件的所有依赖文件

```shell
g++ -MM main.cpp
```

即可显示编译该文件所需要的头文件和依赖文件。



### GDB

编译程序时添加 `-g` 参数，然后才能进行调试；回车键重复上一命令

```shell
g++ -g main.cpp -o main
```

调试的命令参数如下

|          命令           |                     作用                     |
| :-------------------: | :----------------------------------------: |
|        `file`         |                   加载调试文件                   |
|       `help(h)`       |                   查看命令帮助                   |
|       `run(r)`        |                    运行文件                    |
|        `start`        |            单步执行，运行程序，停在第一行执行语句             |
|       `list(l)`       | 查看源代码（`l n` 以第 n 行为中心查看代码，`l func` 查看具体函数） |
|         `set`         |                   设置变量的值                   |
|       `next(n)`       |                单步执行（函数直接执行）                |
|       `quit(q)`       |                     退出                     |
|       `display`       |                  追踪查看变量值                   |
|      `undisplay`      |                    取消追踪                    |
|       `step(s)`       |               单步执行（跳入函数内部执行）               |
|    `backtrace(bt)`    |               查看函数调用的栈帧和层级关系               |
|       `info(i)`       |                查看函数内部局部变量的值                |
|       `finish`        |              结束当前函数，返回到函数调用点               |
|     `continue(c)`     |                    继续执行                    |
|      `print(p)`       |                   打印值及地址                   |
|        `b num`        |                在 num 行设置断点                 |
|         `i b`         |                   查看所有断点                   |
|        `d num`        |                删除第 num 个断点                 |
| `enable breakpoints`  |                    启用断点                    |
| `disable breakpoints` |                    禁用断点                    |
|        `watch`        |                当被观察的变量修改时显示                |
|       `i watch`       |                   显示观察点                    |
| `run argv[1] argv[2]` |                   调用时传参                    |



### 静态库 a

a 文件是 Ubuntu 下的静态库，其在主函数编译时就将库导入

```shell
# 生成 .o 文件
g++ swap.cpp -Iinclude -c

# 生成静态库 libSwap.a
ar rs libswap.a swap.o

# 链接静态库生成 main
g++ -o main.cpp -Lsrc -lswap main

# 运行main
./main
```

**注意生成的静态库要有 lib 前缀，使用时去掉 lib 前缀**。



我们也可以生成 Windows 下的静态库文件

```shell
ar rs swap.lib swap.o
```



### 动态库 so

so 文件是 Ubuntu 下的动态库，它不会直接编译进主函数，需要在调用时添加。其中使用了 -fPIC 参数，这在 LAPACK 编译过程中提到，它表示以相对地址来实现代码，从而使其可以加载到任意位置

```shell
# 生成动态库 swap.so
g++ swap.cpp -Iinclude -fPIC -shared -o libswap.so
# 以上指令等价于
# gcc swap.cpp -Iinclude -c -fPIC
# gcc -shared -o libswap.so swap.o

# 链接动态库生成main
g++ -o main.cpp -Lsrc -lswap main

# 运行 main，要将动态库所在路径添加到参数
LD_LIBRARY_PATH=src ./main
```

同样可以生成 Windows 下的动态库文件

```makefile
gcc -shared -o swap.dll swap.c
```



## Docker

### 安装配置

安装 docker

```shell
sudo apt install docker.io
```

启用 docker，然后安装工具并重启

```shell
systemctl start docker
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
service docker restart
```

验证是否安装成功

```shell
docker -v
```



### 命令操作

#### 参数解析

使用 `docker run` 创建并运行一个容器

* `-d` 参数让容器在后台运行
* `--name` 参数指定容器名
* `-p` 参数指定端口映射
* 在镜像名后添加版本 `mysql:5.7`

例如

```shell
docker run -d --name mysql -p 3036:3036 mysql:5.7
```



对于不熟悉的命令参数，可以使用 `--help` 查看帮助

```shell
docker save --help
```



#### 常见命令

| 命令       | 作用   | 命令      | 作用        |
| :------- | :--- | :------ | :-------- |
| `images` | 查看镜像 | `rmi`   | 移除镜像      |
| `pull`   | 拉取镜像 | `run`   | 创建容器并运行镜像 |
| `build`  | 构建镜像 | `save`  | 保存镜像      |
| `load`   | 加载镜像 | `push`  | 推送镜像      |
| `stop`   | 停止镜像 | `start` | 启动镜像      |
| `ps`     | 查看容器 | `rm`    | 移除容器      |



### 镜像和容器

当利用 docker 安装应用时，docker 会自动搜索并下载应用镜像（image）。镜像不仅包含应用本身，还包含应用运行所需要的环境、配置、系统函数库。Docker 会在运行镜像时创建一个隔离环境，称为容器（container）。储存和管理镜像的平台就是镜像仓库，官方维护一个公共仓库：

```embed
title: "Docker - Hub"
image: "https://www.tutorialspoint.com/images/tp_logo_436.png"
description: "Docker - Hub - Docker Hub is a cloud-based repository service that allows users to store, share, and manage Docker container images. It is offered by Docker. Developers can package their apps and dependencies into lightweight, portable containers using the widely used Docker platform. Applications can then be depl"
url: "https://www.tutorialspoint.com/docker/docker_hub.htm"
```



#### 镜像加速

创建 Docker 的配置文件

```shell
sudo vim /etc/docker/daemon.json
```

在打开的文件中，加入以下配置项

```json
{
    "registry-mirrors": [
        "https://dockerproxy.com",
        "https://hub-mirror.c.163.com",
        "https://mirror.baidubce.com",
        "https://ccr.ccs.tencentyun.com"
    ]
}
```

然后重启 docker 服务

```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```

查看 docker 信息

```shell
sudo docker info
```

如果配置成功，可以看到添加的镜像

```text
Registry Mirrors:
  https://dockerproxy.com/
  https://hub-mirror.c.163.com/
  https://mirror.baidubce.com/
  https://ccr.ccs.tencentyun.com/
```

尝试运行 hello-world

```shell
sudo docker run hello-world
```

由于我们没有拉取 hello-world，因此会显示本地没有镜像，并自动拉取。现在运行

```shell
sudo docker images
```

查看当前所有 docker 镜像，可以找到 `hello-world` 镜像。再次运行 `hello-world` 将会看到

```shell
Hello from Docker!
```

使用 `docker ps` 查看当前正在运行的容器。



#### 部署镜像

以 Nginx 服务器为例。首先拉取镜像

```shell
sudo docker pull nginx
```

然后检查镜像

```shell
sudo docker images
```

如果希望将镜像保存，传递给其他人，可以使用

```shell
sudo docker save -o nginx.tar nginx:latest
```

打包为一个压缩包，指定最新版本。要删除镜像，使用

```shell
sudo docker rmi nginx:latest
```

现在从本地保存的镜像压缩包重新加载镜像

```shell
sudo docker load -i nginx.tar
```

创建容器并运行

```shell
sudo docker run -d --name nginx -p 80:80 nginx
```

停止运行

```shell
sudo docker stop nginx
```



#### 检查容器

现在通过 `ps` 无法看到 `nginx` 容器，使用

```shell
sudo docker ps -a
```

查看所有容器。重新启动容器

```shell
sudo docker start nginx
```

要查看日志，使用

```shell
sudo docker logs nginx
```

添加 `-f` 参数持续查看日志

```shell
sudo docker logs -f nginx
```

要删除容器，使用

```shell
sudo docker rm nginx
```



#### 访问服务

查看当前 ip 地址，可以看到 docker 自动生成的网卡。配合 nginx 设置的端口号可以访问 nginx 服务

```shell
172.17.0.1:80
```

或者使用

```shell
localhost:80
```



#### 进入容器

可以进入容器环境，使用

```shell
sudo docker exec -it nginx bash
```

其中 `exec` 执行容器，`-it` 参数添加一个可输入终端，最后配置 bash 终端。退出终端

```shell
exit
```



#### 数据卷挂载

直接进入容器进行修改并不方便，因为容器内部仅保有容器运行所需的最小运行环境，需要命令都不存在。解决方案是使用数据卷（volume）挂载，它是一个虚拟目录，是容器内目录与宿主机目录之间映射的桥梁。容器中的文件挂载到 `/var/lib/docker` 目录下的对应文件。

![](image-20240803182937760.jpeg)

数据卷使用的命令主要包括

| 命令                      | 说明        |
| :---------------------- | :-------- |
| `docker volume create`  | 创建数据卷     |
| `docker volume ls`      | 查看所有数据卷   |
| `docker volume rm`      | 删除指定数据卷   |
| `docker volume inspect` | 查看某个数据卷详情 |
| `docker volume prune`   | 清除数据卷     |

可以查看所有数据卷命令

```shell
docker volume --help
```



如果要挂载数据卷，必须在创建容器时指定。使用

```shell
sudo docker run -d --name nginx -p 80:80 -v html:/usr/share/nginx/html nginx
```

其中 `-v` 参数依次指定数据卷名称和要挂载的容器中的文件。现在可以查看数据卷

```shell
sudo docker volume ls
```

查看数据卷详情

```shell
sudo docker volume inspect html
```

其中 `Mountpoint` 就是对应的挂载点。



可以查看容器的具体信息

```shell
sudo docker inspect nginx
```

可以看到 `Mounts` 对应的挂载信息。即使在创建容器时不指定数据卷，依然会默认挂载一个匿名卷，通常是为了存放产生的数据。然而，如果需要对容器进行升级，就要删除容器后重新创建，这会导致容器数据丢失。



#### 本地目录挂载

更好的方法是每次创建容器时，都将其挂载到一个本地目录

```shell
sudo docker run -d --name nginx -p 80:80 -v ~/html:/usr/share/nginx/html nginx
```



### 自定义镜像

镜像就是包含了应用程序、程序运行的系统函数库、运行配置等文件的文件包。构建镜像的过程就是把上述文件打包的过程。



#### 镜像结构

镜像是分层的，一个镜像容器中的环境可能依赖其它镜像，这些依赖项构成层（Layer），应用依赖的系统函数库、环境、配置、文件等就是基础镜像（BaseImage）。



#### Dockerfile

Dockerfile 是一个文本文件，包含指令，说明构建镜像需要的操作。常见指令包括

| 指令           | 说明                  | 示例                                |
| :----------- | :------------------ | :-------------------------------- |
| `FROM`       | 指定基础镜像              | `FROM centos:6`                   |
| `ENV`        | 设置环境变量              | `ENV key value`                   |
| `COPY`       | 拷贝本地文件到镜像的指定目录      | `COPY ./jrel.tar.gz /tmp`         |
| `RUN`        | 执行的 `shell` 命令      | `RUN tar -zxvf /tmp/jrell.tar.gz` |
| `EXPOSE`     | 指定容器运行时监听的端口        | `EXPOSE 8080`                     |
| `ENTRYPOINT` | 镜像中应用的启动命令，在容器运行时调用 | `ENTRYPOINT java -jar xx.jar`     |
例如使用下面的项目

```embed
title: "GitHub - gzyunke/test-docker: docker test project. nodejs + koa2"
image: "https://opengraph.githubassets.com/018b2754e5be968fb47d22a5b13eeff7a38566a9ff885d36393936144f992bb2/gzyunke/test-docker"
description: "docker test project. nodejs + koa2. Contribute to gzyunke/test-docker development by creating an account on GitHub."
url: "https://github.com/gzyunke/test-docker"
```

其中 dockerfile 为

```dockerfile
FROM node:11
MAINTAINER easydoc.net

# 复制代码
ADD . /app

# 设置容器启动后的默认运行目录
WORKDIR /app

# 运行命令，安装依赖
# RUN 命令可以有多个，但是可以用 && 连接多个命令来减少层级。
# 例如 RUN npm install && cd /app && mkdir logs
RUN npm install --registry=https://registry.npm.taobao.org

# CMD 指令只能一个，是容器启动后执行的命令，算是程序的入口。
# 如果还需要运行其他命令可以用 && 连接，也可以写成一个shell脚本去执行。
# 例如 CMD cd /app && ./start.sh
CMD node app.js

```



#### 构建镜像

编写完 Dockerfile 后，可以构建镜像

```shell
sudo docker build -t test:v1 .
```

其中 `-t` 指定镜像名称，`.` 指定 Dockerfile 所在目录为当前目录。



#### Docker-compose

Docker-compose 作用在于简化多容器服务，可以在 `docker-compose.yml` 文件中配置服务信息

```yaml
version: "3.7"

services:
  app:
    build: ./
    ports:
      - 80:8080
    volumes:
      - ./:/app
    environment:
      - TZ=Asia/Shanghai
  # 获得 redis 镜像，挂载数据卷，配置时区
  redis:
    image: redis:5.0.13
    volumes:
      - redis:/data
    environment:
      - TZ=Asia/Shanghai

volumes:
  redis:

```

然后执行

```shell
sudo docker-compose up -d
```

命令使用方法与之前的命令类似

| 命令                       | 说明     |
| :----------------------- | :----- |
| `docker-compose ps`      | 查看运行容器 |
| `docker-compose stop`    | 停止运行   |
| `docker-compose restart` | 重启服务   |
| `docker-compose exec`    | 进入容器   |
| `docker-compose logs`    | 查看日志   |



### 网络配置

默认情况下，所有容器以 bridge 方式连接到 Docker 的一个虚拟网桥上，容器之间可以通过网桥连接

![](image-20240803210158250.png|725)

但是由于每次启动容器时 ip 地址分配不确定，因此容器之间访问可能需要频繁更换对应的 ip 地址。为了更方便地访问其它容器，就需要自定义网桥。



#### 自定义网桥

首先查看网络

```shell
sudo docker network ls
```

可以看到默认的 bridge 网桥。现在创建新的网桥

```shell
sudo docker network create mb
```

将容器连接到网桥

```shell
sudo docker network connect mb nginx
```

查看容器信息

```shell
sudo docker inspect nginx
```

在 `Networks` 中可以看到现在容器连接到两个网络。我们也可以在创建容器时就指定网桥

```shell
sudo docker run -d --name nginx -p 80:80 --network mb nginx
```

这样容器不会连接到默认的网桥，只会连接到指定的网桥。同一网桥下，两个容器可以通过容器名互相访问

```shell
sudo docker exec -it nginx bash		# 进入容器
ping nginx2							# 连接到 nginx2 容器
```



#### 常用命令

| 命令                          | 说明       |
| :-------------------------- | :------- |
| `docker network create`     | 创建网络     |
| `docker network ls`         | 查看所有网络   |
| `docker network rm`         | 删除指定网络   |
| `docker network prune`      | 清除未使用的网络 |
| `docker network connect`    | 指定容器连接网络 |
| `docker network disconnect` | 指定容器离开网络 |
| `docker network inspect`    | 查看网络详细信息 |



## SQLite

SQLite 是一个轻量级的开源数据库，源代码完全公开不受版权限制，实现了自给自足的、无服务器、零配置的 SQL 数据库引擎。 

```embed
title: "SQLite Home Page"
image: "https://www.sqlite.org/images/sqlite370_banner.gif"
description: "SQLite is a C-language library that implements a small, fast, self-contained, high-reliability, full-featured, SQL database engine. SQLite is the most used database engine in the world. SQLite is built into all mobile phones and most computers and comes bundled inside countless other applications that people use every day. More Information…"
url: "https://www.sqlite.org/index.html"
```



要在命令行使用 sqlite ，则下载对应的 sqlite 文件

![image-20220125174933889](image-20220125174933889.png)

下载第 1 个动态库文件和第 3 个带有命令行工具的版本。



### 打开数据库

运行 `.exe` 程序进入 sqlite 界面

```shell
sqlite3 filename
sqlite3
```

前者可以直接对文件进行操作，后者需要通过 .open 命令打开文件

```shell
.open test.db
```



进入命令行界面后，可以输入两种指令

* 一种是自身配置和格式控制相关指令，这些指令都以 `.` 开头；
* 另一种指令是 SQL 语句，实现对数据库的增删查改等操作；

两种指令都要以 `;` 结束。



### SQL 操作

#### 创建数据表

使用 CREATE TABLE 创建指定结构的表格

```sqlite
CREATE TABLE tablename(col1 type[require], col2 type[require], ...);
```

* 常用类型
    * INT（整型） TEXT（字符串） REAL（浮点数）
* 常用约束
    * PRIMARY KEY（主键）：表示该列数据唯一，可以加快数据访问
    * NOT NULL（非空）：该列数据不能为空

例如创建一个表，分别有 id, name, score 三列，类型为 int, text, real，对应的约束都为非空

```sqlite
CREATE TABLE student(id INT PRIMARY KEY NOT NULL, name TEXT NOT NULL, score REAL NOT NULL);
```



#### 删除数据表

使用 DROP TABLE 删除指定的表格

```sqlite
DROP TABLE student;
```

应当慎用，数据表一旦删除，其中数据也将消失。



#### 查询数据

使用 SELECT 查看表格内容

```shell
SELECT * FROM student;
```

这里 `*` 表示查询**所有满足条件的内容**。也可以指定列和条件，以及排序方式

```sqlite
SELECT col1, col2, ... FROM tablename;
SELECT * FROM tablename WHERE condition;
SELECT col1, col2, ... FROM tablename WHERE condition;
SELECT col1, col2, ... FROM tablename WHERE condition ORODER BY col3 ASC;
```

其中 ASC 表示升序， DESC 表示降序。



使用 LIKE 指定模糊条件

```sqlite
SELECT col1, col2 ... FROM tablename WHERE col3 LIKE condition;
```

模糊匹配通配符

*  `%` ：代表零个、一个或多个数字或字符
*  `_` ：代表一个单一的数字或字符

它们可以被组合使用，例如匹配**以 8 开头**的数据

```sqlite
SELECT * FROM student WHERE score LIKE "8%";
```

匹配**以 8 开头长度为 4 **的数据

```sqlite
SELECT * FROM student WHERE score LIKE "8___";
```



#### 插入数据

```sqlite
INSERT INTO tablename(col1, col2, ...) VALUES(val1, val2, ...);
```

如果要为表中的所有列添加值，并且插入列的顺序和创建表的顺序相同，就可以不指定列名

```sqlite
INSERT INTO tablename VALUES(col1, col2, ...);
```

其中如果有空数值，用 NULL 替代即可。例如

```sqlite
INSERT INTO student VALUES(10004,'甄姬',81.5);
INSERT INTO student VALUES(10005,'孙尚香',80);
INSERT INTO student VALUES(10006,'大乔',90);
INSERT INTO student VALUES(10007,'小乔',91);
```



#### 删除数据

使用 DELETE FROM 删除表格数据

```sqlite
DELETE FROM tablename
```

但是这样数据表的所有数据都将被删除。可以指定删除条件

```sqlite
DELETE FROM tablename WHERE condition;
```

如果有多个条件可以使用 and 和 or 连接

```sqlite
DELETE FROM student WHERE id = 10004;
DELETE FROM student WHERE name = '孙尚香' and score < 90;
```



#### 修改数据

可以修改指定列的数据

```sqlite
UPDATE tablename SET col1 = val1, col2 = val2, ...
```

但是这样数据表对应列的所有数据都将被修改。可以指定修改条件

```sqlite
UPDATE tablename SET col1 = val1, col2 = val2, ... WHERE 条件表达式;
```

新数值可以是一个常数，也可以是一个表达式。例如

```sqlite
UPDATE student SET score = score + 10 WHERE id = 10006;
```

注意这里直接使用 `=` 判断相等条件。



### 指令操作

常用的命令如下

| 命令               | 作用                         |
| ------------------ | ---------------------------- |
| .help              | 获取指令的帮助信息           |
| .open filename     | 打开指定的数据库文件         |
| .database          | 查看数据库名称和对应的文件名 |
| .table             | 查看数据表的名字             |
| .schema  tablename | 查看数据表创建时的信息       |
| .mode              | 设置显示模式                 |
| .nullvalue         | 设置空白字段显示的字符串     |
| .header on/off     | 显示/不显示数据表的表头      |
| .exit 或 .quit     | 退出程序                     |



显示模式可以设为

|  模式  |        作用         |
| :----: | :-----------------: |
| ascii  |                     |
|  csv   |        分割         |
| column |      左列对齐       |
|  html  | 显示 HTML 表格代码  |
| insert | 在表中插入 SQL 说明 |
|  line  |    每个数据一行     |
|  list  |  列表形式（默认）   |
|  tab   |     用 TAB 间隔     |
|  tcl   |  使用 TCL 排列元素  |

例如切换为 tab 格式

```shell
.mod tab
```



设置空数据显示的字符串

```shell
.nullvalue "NULL"
```

指定空白位置显示 NULL 字符串。



## GitLab

### 基本配置

要安装配置 GitLab，首先需要进行基础配置

```shell
sudo apt-get update
sudo apt-get install -y curl openssh-server ca-certificates tzdata perl
```

然后安装 postfix 来发送电子邮件

```shell
sudo apt-get install -y postfix
```

安装过程中会出现设置窗口，点击 Tab 键回车选择 `Internet Site`，然后继续 Tab 键回车完成配置，进行安装。



接着下载 GitLab 源镜像

```shell
curl -fsSL https://packages.gitlab.cn/repository/raw/scripts/setup.sh | /bin/bash
```

然后指定安装时链接到的 URL

```shell
sudo EXTERNAL_URL="http://gitlab.xingyf.com" apt-get install gitlab-jh
```

默认情况下，Linux 软件包安装将 Git 仓库数据存储在 `/var/opt/gitlab/git-data` 下。仓库存储在名为 `repositories` 的子文件夹中。



设置 `/etc/gitlab/gitlab.rb` 文件中的 `external_url`，后面填 ip 地址

```shell
external_url = "http://ip"
```

执行下面的 `reconfigure` 命令，在 `/etc/gitlab/initial_root_password` 文件中生成初始密码。然后执行 `start` 命令即可启用

```shell
sudo gitlab-ctl start							# 启动所有 gitlab 组件
sudo gitlab-ctl stop							# 停止所有 gitlab 组件
sudo gitlab-ctl restart							# 重启所有 gitlab 组件
sudo gitlab-ctl status							# 查看服务器状态
sudo gitlab-ctl reconfigure						# 重新配置
sudo gedit /etc/gitlab/gitlab.rb				# 修改配置文件
gitlab-rake gitlab:check SANITIZE=true --trace	# 检查 gitlab
sudo gitlab-ctl tail							# 查看日志
```

现在可以用 root 用户名和初始密码登录。设置 url 地址

```shell
external_url "http://10.72.188.185:9898"
```

这里我们先开启防火墙，然后允许对应的端口

```shell
sudo ufw enable
sudo ufw allow 9898/tcp
sudo ufw reload
sudo ufw status
```

如果没有开启防火墙，就不需要这一操作。（相反，如果启用防火墙可能还会导致端口被禁用，无法传送文件）



### 服务设置

在停止服务时，仅仅将 runner 服务暂停还不够，此时 gitlab 仍然有部分进程运行，导致高昂的内存开销。需要通过关闭整个 gitlab 服务来释放内存。

```shell
systemctl start gitlab-runsvdir		# 启动所有 gitlab 服务

# 先停止 gitlab 服务
sudo gitlab-ctl start							# 启动 gitlab 组件
sudo gitlab-ctl stop							# 停止 gitlab 组件

# 再停止 gitlab-runner
gitlab-runner -h 		# 查看帮助文档
# --user指定将用于执行构建的用户，--working-directory 指定将运行构建时数据存储的根目录
gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
gitlab-runner uninstall # 停止运行并从服务中卸载 GitLab Runner
gitlab-runner start     # 启动 GitLab Runner 服务
gitlab-runner stop      # 停止 GitLab Runner 服务
gitlab-runner restart   # 重启 GitLab Runner 服务
gitlab-runner status  	# 显示 GitLab Runner 服务的状态

systemctl status gitlab-runner		# 获得 runner 服务状态
systemctl start gitlab-runner		# 启动 runner 服务
systemctl stop gitlab-runner		# 停止 runner 服务
systemctl restart gitlab-runner		# 重启 runner 服务

# 再停止 gitlab-runsvdir
systemctl stop gitlab-runsvdir		# 停止 gitlab 服务并释放内存

gitlab-runner --debug <command>   	# 调试模式排查错误特别有用
gitlab-runner <command> --help    	# 获取帮助信息
gitlab-runner run       			# 普通用户模式,配置文件: ~/.gitlab-runner/config.toml
sudo gitlab-runner run  			# 超级用户模式,配置文件:/etc/gitlab-runner/config.toml
 
gitlab-runner register     			# 默认交互模式下使用，非交互模式添加 --non-interactive
gitlab-runner list      			# 此命令列出了保存在配置文件中的所有运行程序
gitlab-runner verify    			# 此命令检查注册的 runner 是否可以连接，但不验证 GitLab 服务是否正在使用 runner
gitlab-runner unregister   			# 该命令使用 GitLab 取消已注册的 runner
gitlab-runner unregister --url http://gitlab.dev.com/ --token djiih23   # 使用令牌注销
gitlab-runner unregister --name test-runner 							# 使用名称注销（同名删除第一个）
gitlab-runner unregister --all-runners 									# 注销所有 runner
```



默认情况下，服务会开机自启动，需要设置

```shell
sudo systemctl disable gitlab-runsvdir.service		# 禁止开机自启
sudo systemctl is-enabled gitlab-runsvdir.service	# 查看是否启用
```



### CI/CD 配置

要实现自动化部署，需要安装 Gitlab Runner，注意不要使用官网提供的包，版本太落后。应该在[镜像网站](https://gitlab-runner-downloads.s3.amazonaws.com/v16.6.1/index.html)中下载安装。



#### rpm

需要安装 rpm 库

```shell
sudo apt install rpm
```

用它来安装下载好的安装包

```shell
sudo rpm -ivh gitlab-runner_amd64.rpm --nodeps --force
sudo gitlab-runner --version
```

后面两个参数用来取消依赖，强制安装。



#### 创建 runner

赋予权限

```shell
sudo chmod +x /usr/lib/gitlab-runner
```

创建 runner 用户

```shell
sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash
```

安装 runner 并启用服务

```shell
sudo gitlab-runner install --user=gitlab-runner --working-directory=/home/xingyifan/gitlab-runner
sudo gitlab-runner start
```

注意需要创建 gitlab-runner 文件夹。**由于权限问题，没有选择在 home 目录下创建**。



为了提供更高的执行权限，首先卸载默认的用户，设置 root 用户权限，重启 runner

```shell
gitlab-runner uninstall
gitlab-runner install --working-directory /home/xingyifan/gitlab-runner --user root
systemctl restart gitlab-runner
```

查看当前用户

```shell
ps aux | grep gitlab-runner
```



#### 注册 runner

现在注册服务

```shell
sudo gitlab-runner register --url http://10.72.188.185:9898/ --registration-token GR1348941CFsTtV4uN-nyVipkJmsz
```

这里直接给定了 url 和 token 两个参数。这一段命令来自 GitLab 项目的“设置-CI/CD-Runner”中“新建项目 Runner”右边的详情中的注册说明。注意在注册过程中，需要指定 tags 和执行器，我们随意指定一个 tag 然后使用 shell 执行器。这时刷新 GitLab 界面已经可以看到添加的 Runner 。



#### 测试 runner

将项目克隆到本地，修改 `.gitlab-ci.yml` 配置文件

```yaml
# You can override the included template(s) by including variable overrides
# SAST customization: https://docs.gitlab.com/ee/user/application_security/sast/#customizing-the-sast-settings
# Secret Detection customization: https://docs.gitlab.com/ee/user/application_security/secret_detection/#customizing-settings
# Dependency Scanning customization: https://docs.gitlab.com/ee/user/application_security/dependency_scanning/#customizing-the-dependency-scanning-settings
# Container Scanning customization: https://docs.gitlab.com/ee/user/application_security/container_scanning/#customizing-the-container-scanning-settings
# Note that environment variables can be set in several places
# See https://docs.gitlab.com/ee/ci/variables/#cicd-variable-precedence
stages:
  - deploy

deploy-job:
  tags:
    - Test
  stage: deploy
  script: echo "hello world!"

```

然后执行一轮提。我们这里没有指定 `tag`，因此默认不会运行。需要修改设置，允许运行未打标签的作业。

![](image-20231217191325783.png)

在项目界面看到执行的任务

![](image-20231217134719735.png)

![](image-20231217134750183.png)



### Runner 配置

以下执行器都可用。

| 执行器           | 所需配置                                                     | 作业运行位置                                                 |
| :--------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `shell`          |                                                              | 本地 Shell，默认执行器。                                     |
| `docker`         | `[runners.docker]` 和 [Docker 引擎](https://docs.docker.com/engine/) | Docker 容器。                                                |
| `docker-windows` | `[runners.docker]` 和 [Docker 引擎](https://docs.docker.com/engine/) | Windows Docker 容器。                                        |
| `ssh`            | `[runners.ssh]`                                              | SSH，远程。                                                  |
| `parallels`      | `[runners.parallels]` 和 `[runners.ssh]`                     | 并行虚拟机，但与 SSH 相连。                                  |
| `virtualbox`     | `[runners.virtualbox]` 和 `[runners.ssh]`                    | VirtualBox 虚拟机，但与 SSH 相连。                           |
| `docker+machine` | `[runners.docker]` 和 `[runners.machine]`                    | 类似 `docker`，但使用[弹性伸缩 Docker Machine](https://docs.gitlab.cn/runner/configuration/autoscale.html)。 |
| `kubernetes`     | `[runners.kubernetes]`                                       | Kubernetes pods。                                            |



可用的 Shell 可以在不同平台运行。

| Shell        | 描述                                                         |
| :----------- | :----------------------------------------------------------- |
| `bash`       | 生成 Bash（Bourne-shell）脚本。所有在 Bash 上下文中执行的命令。是所有 Unix 系统的默认值。 |
| `sh`         | 生成 Sh (Bourne-shell) 脚本。所有在 Sh 上下文中执行的命令。是所有 Unix 系统的应急计划。 |
| `powershell` | 生成 PowerShell 脚本。所有在 PowerShell Desktop 上下文中执行的命令。在极狐GitLab Runner 12.0-13.12 中，是 Windows 的默认值。 |
| `pwsh`       | 生成 PowerShell 脚本。所有在 PowerShell Core 上下文中执行的命令。在极狐GitLab Runner 14.0 及更高版本中，是 Windows 的默认值。 |

当 `shell` 选项被设置为 `bash` 或 `sh` 时，Bash 的 [ANSI-C quoting](https://www.gnu.org/software/bash/manual/html_node/ANSI_002dC-Quoting.html) 用于 Shell 转义作业脚本。



### CI/CD 流程

在 `.gitlab-ci.yml` 中配置 GitLab CI/CD 的特定指令。



#### 基本介绍

现在我们给出一个稍微复杂的例子

```yaml
build-job:
  stage: build
  script:
    - echo "Hello, $GITLAB_USER_LOGIN!"

test-job1:
  stage: test
  script:
    - echo "This job tests something"

test-job2:
  stage: test
  script:
    - echo "This job tests something, but takes more time than test-job1."
    - echo "After the echo commands complete, it runs the sleep command for 20 seconds"
    - echo "which simulates a test that runs 20 seconds longer than test-job1"
    - sleep 20

deploy-prod:
  stage: deploy
  script:
    - echo "This job deploys something from the $CI_COMMIT_BRANCH branch."
  environment: production
```

其中 `GITLAB_USER_LOGIN` 和 `CI_COMMIT_BRANCH` 是在作业运行时填充的预定义变量。



现在等待作业完成，单击流水线 ID 可以看到可视化表示

![](image-20231217191659764.png)

整个执行过程按照我们的配置，分为 build, test, deploy 三个阶段，每个阶段对应完成作业。



我们可以预先声明阶段 stage，然后分别指定每个阶段的工作

```yaml
stages:
  - build
  - test

build-code-job:
  stage: build
  script:
    - echo "Check the ruby version, then build some Ruby project files:"
    - ruby -v
    - rake

test-code-job1:
  stage: test
  script:
    - echo "If the files are built successfully, test some files with one command:"
    - rake test1

test-code-job2:
  stage: test
  script:
    - echo "If the files are built successfully, test other files with a different command:"
    - rake test2
```



#### 关键字

##### default

使用它定义全局默认值，例如下面的关键字可以作为输入

- [`after_script`](https://docs.gitlab.cn/jh/ci/yaml/#after_script)
- [`artifacts`](https://docs.gitlab.cn/jh/ci/yaml/#artifacts)
- [`before_script`](https://docs.gitlab.cn/jh/ci/yaml/#before_script)
- [`cache`](https://docs.gitlab.cn/jh/ci/yaml/#cache)
- [`hooks`](https://docs.gitlab.cn/jh/ci/yaml/#hooks)
- [`image`](https://docs.gitlab.cn/jh/ci/yaml/#image)
- [`interruptible`](https://docs.gitlab.cn/jh/ci/yaml/#interruptible)
- [`retry`](https://docs.gitlab.cn/jh/ci/yaml/#retry)
- [`services`](https://docs.gitlab.cn/jh/ci/yaml/#services)
- [`tags`](https://docs.gitlab.cn/jh/ci/yaml/#tags)
- [`timeout`](https://docs.gitlab.cn/jh/ci/yaml/#timeout)

例如为所有作业指定 `ruby:3.0` 作为默认镜像

```yaml
default:
  image: ruby:3.0

rspec:
  script: bundle exec rspec

rspec 2.7:
  image: ruby:2.7
  script: bundle exec rspec
```

默认值会复制到所有未定义该关键字的作业中；如果作业定义了关键字，则会覆盖默认值。



##### stages

使用它定义作业的阶段，同一阶段的作业并行执行；不同阶段的作业按顺序执行。如果没有定义 `stages`，默认阶段为

- [`.pre`](https://docs.gitlab.cn/jh/ci/yaml/#stage-pre)
- `build`
- `test`
- `deploy`
- [`.post`](https://docs.gitlab.cn/jh/ci/yaml/#stage-post)

例如按顺序指定 build, test, deploy 阶段

```yaml
stages:
  - build
  - test
  - deploy
```

如果其中一个阶段中的任何一个作业失败，则后面的作业不会启动；如果作业未指定 [`stage`](https://docs.gitlab.cn/jh/ci/yaml/#stage)，则作业被分配到 `test` 阶段；如果定义了一个阶段，但没有作业使用它，则该阶段在流水线中不可见。 



##### image

使用 `image` 指定运行作业的 Docker 镜像。使用方法如前；还可以进一步指定名称 name 和拉取策略 pull_policy

```yaml
job1:
  script: echo "A single pull policy."
  image:
    name: ruby:3.0
    pull_policy: if-not-present

job2:
  script: echo "Multiple pull policies."
  image:
    name: ruby:3.0
    pull_policy: [always, if-not-present]
```

还可以指定容器入口点执行的命令或脚本

```yaml
image:
  name: super/sql:experimental
  entrypoint: [""]
```

创建 Docker 容器时，`entrypoint` 被转换为 Docker 的 `--entrypoint` 选项。 



##### script

使用 `script` 指定 runner 要执行的命令。几乎所有作业都需要指定此关键字

```yaml
job1:
  script: "bundle exec rspec"

job2:
  script:
    - uname -a
    - bundle exec rspec
```

还可以指定在 `script` 之前 `before_script` 和之后 `after_script` 的命令

```yaml
job:
  before_script:
    - echo "Execute this command before any 'script:' commands."
  script:
    - echo "This command executes after the job's 'before_script' commands."
  after_script:
    - echo "Execute this command after the `script` section completes."
```

需要注意的是

* 在 `before_script` 中指定的脚本与您在主 [`script`](https://docs.gitlab.cn/jh/ci/yaml/#script) 中指定的任何脚本连接在一起。组合脚本在单个 shell 中一起执行。
* 在 `after_script` 中指定的脚本在一个新的 shell 中执行，与任何 `before_script` 或 `script` 命令分开。

因此后者无权访问由 `before_script` 或 `script` 中定义的命令完成的更改。



##### stage

使用 `stage` 定义作业在哪个 [stage](https://docs.gitlab.cn/jh/ci/yaml/#stages) 中运行。同一个 `stage` 中的作业可以并行执行。如果没有定义 `stage`，则作业默认使用 `test` 阶段。

```yaml
stages:
  - build
  - test
  - deploy

job1:
  stage: build
  script:
    - echo "This job compiles code."

job2:
  stage: test
  script:
    - echo "This job tests the compiled code. It runs when the build stage completes."

job3:
  script:
    - echo "This job also runs in the test stage".

job4:
  stage: deploy
  script:
    - echo "This job deploys the code. It runs when the test stage completes."
  environment: production
```

还可以指定在流水线之前 `.pre` 和之后 `.post` 执行的阶段

```yaml
stages:
  - build
  - test

job1:
  stage: build
  script:
    - echo "This job runs in the build stage."

first-job:
  stage: .pre
  script:
    - echo "This job runs in the .pre stage, before all other stages."

last-job:
  stage: .post
  script:
    - echo "This job runs in the .post stage, after all other stages."

job2:
  stage: test
  script:
    - echo "This job runs in the test stage."
```

如果流水线仅包含 `.pre` 或 `.post` 阶段的作业，则它不会运行。 



##### workflow

使用它确定是否创建流水线，例如

```yaml
variables:
  PROJECT1_PIPELINE_NAME: 'Default pipeline name'  # A default is not required.

workflow:
  name: '$PROJECT1_PIPELINE_NAME'
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
      variables:
        PROJECT1_PIPELINE_NAME: 'MR pipeline: $CI_MERGE_REQUEST_SOURCE_BRANCH_NAME'
    - if: '$CI_MERGE_REQUEST_LABELS =~ /pipeline:run-in-ruby3/'
      variables:
        PROJECT1_PIPELINE_NAME: 'Ruby 3 pipeline'
```



##### rules

使用 `rules` 来包含或排除流水线中的作业。注意它不能与 `only/except` 出现在同一个作业中。`rules` 接受以下规则：

- `if`
- `changes`
- `exists`
- `allow_failure`
- `variables`
- `when`

使用 `rules:if` 子句指定何时向流水线添加作业：

- 如果 `if` 语句为 true，则将作业添加到流水线中
- 如果 `if` 语句为 true，但它与 `when: never` 结合使用，则不添加作业
- 如果没有 `if` 语句为 true，则不添加作业

例如

```yaml
job:
  script: echo "Hello, Rules!"
  rules:
    - if: $CI_MERGE_REQUEST_SOURCE_BRANCH_NAME =~ /^feature/ && $CI_MERGE_REQUEST_TARGET_BRANCH_NAME != $CI_DEFAULT_BRANCH
      when: never
    - if: $CI_MERGE_REQUEST_SOURCE_BRANCH_NAME =~ /^feature/
      when: manual
      allow_failure: true
    - if: $CI_MERGE_REQUEST_SOURCE_BRANCH_NAME
```


##### when

使用 `when` 配置作业运行的条件，可选

- `on_success`（默认）：仅当早期阶段没有作业失败或具有 `allow_failure: true` 时才运行作业
- `on_failure` 仅当早期阶段至少有一个作业失败时才运行作业
- `always` 无论早期阶段的作业状态如何，都运行作业
- `never` 无论早期阶段的作业状态如何，都不要运行作业
- `delayed` 延迟触发作业
- `manual` 手动触发作业

如果未在作业中定义，则默认值为 `when: on_success`。例如

```yaml
stages:
  - build
  - cleanup_build
  - test
  - deploy
  - cleanup

build_job:
  stage: build
  script:
    - make build

cleanup_build_job:
  stage: cleanup_build
  script:
    - cleanup build when failed
  when: on_failure

test_job:
  stage: test
  script:
    - make test

deploy_job:
  stage: deploy
  script:
    - make deploy
  when: manual
  environment: production

cleanup_job:
  stage: cleanup
  script:
    - cleanup after jobs
  when: always
```

在这个例子中

1. 只有当 `build_job` 失败时才执行 `cleanup_build_job`
2. 无论成功或失败，始终将 `cleanup_job` 作为流水线的最后一步执行
3. 在 GitLab UI 中手动运行时执行 `deploy_job`


##### retry

使用 `retry` 配置作业失败时重试的次数，至多重试 2 次。例如

```yaml
test:
  script: rspec
  retry: 2
```

如果未定义，则默认为 `0` 并且作业不会重试。



使用 `retry:when` 和 `retry:max` 仅针对特定的失败情况重试作业。

- `retry:max` 是最大重试次数
- `retry:when` 是重试条件，可选
	- `always`：任何失败重试（默认）
	- `unknown_failure`：当失败原因未知时重试
- 还有更多可选项，例如

```yaml
test:
  script: rspec
  retry:
    max: 2
    when: runner_system_failure
```

如果果存在除 runner 系统故障以外的故障，则不会重试作业。



##### timeout

使用 `timeout` 为指定执行时间。如果作业运行的时间超过超时时间，作业将失败。例如

```yaml
build:
  script: build.sh
  timeout: 3 hours 30 minutes

test:
  script: rspec
  timeout: 3h 30m
```


##### release

使用 `release` 创建一个发布版本。例如

```yaml
release_job:
  stage: release
  image: registry.gitlab.com/gitlab-org/release-cli:latest
  rules:
    - if: $CI_COMMIT_TAG                  # Run this job when a tag is created manually
  script:
    - echo "Running the release job."
  release:
    tag_name: $CI_COMMIT_TAG
    name: 'Release $CI_COMMIT_TAG'
    description: 'Release created using the release-cli.'
```



##### pages

使用 `pages` 定义一个 GitLab Pages 作业，将静态内容上传发布为网站。例如

```yaml
pages:
  stage: deploy
  script:
    - mkdir .public
    - cp -r * .public
    - mv .public public
  artifacts:
    paths:
      - public
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
  environment: production
```

此示例将所有文件从项目的根目录移动到 `public/` 目录。创建 `.public` 是为了使 `cp` 不会在无限循环中将 `public/` 复制到自身。



#### 代码智能

通过将极狐 GitLab CI/CD 作业添加到生成 LSIF 产物的项目的 `.gitlab-ci.yml` 中，为项目启用代码智能：

```yaml
code_navigation:
  image: sourcegraph/lsif-go:v1
  allow_failure: true # recommended
  script:
    - lsif-go
  artifacts:
    reports:
      lsif: dump.lsif
```

启用后可以在浏览代码的同时查看函数定义、跳转函数声明。



### Pages

使用极狐 GitLab Pages，您可以直接从极狐 GitLab 的仓库中发布静态网站。

- 用于任何个人或商业网站。
- 使用任何静态站点生成器 (SSG) 或纯 HTML 。
- 为您的项目、群组或用户账户创建网站。
- 在您自己的 GitLab 私有化部署实例上免费托管您的网站。
- 连接您的自定义域名和 TLS 证书。
- 将任何许可证归于您的内容。

要使用 Pages 发布网站，您可以使用任何静态网站生成器，例如 Gatsby、Jekyll、Hugo、Middleman、Harp、Hexo 或 Brunch 。您还可以发布任何直接用纯 HTML、CSS 和 JavaScript 编写的网站。



#### 开启 Pages

默认情况下 Pages 功能关闭，需要修改 `/etc/gitlab/gitlab.rc` 中的配置

```yaml
pages_external_url "http://pages.example.com/"
gitlab_pages['enable'] = true

##! Configure to expose GitLab Pages on external IP address, serving the HTTP
gitlab_pages['external_http'] = ['10.194.84.248:80']

##! Configure to expose GitLab Pages on external IP address, serving the HTTPS
gitlab_pages['external_https'] = ['10.194.84.248:443']
```

然后重新配置

```shell
sudo gitlab-ctl reconfigure
```

在左侧最下方找到 `Admin` 按钮点击，然后选择左侧“仪表盘”，可以在右侧看到功能 GitLab Pages 已经启用。

![](image-20240907064434055.png)



#### 部署仓库

GitLab 始终从仓库中名为 `public` 的特定文件夹部署网站。配置文件中名为 `pages` 的特定 `job` 声明正在部署网站。假设仓库包含以下文件：

```shell
├── index.html
├── css
│   └── main.css
└── js
    └── main.js
```

可以创建一个 `.yml` 文件，将所有文件移动到 `.public` 下，然后再移动到 `public` 下，这里创建 `.public` 是防止 `public` 本身被无限复制。

```yaml
pages:
  script:
    - mkdir .public
    - cp -r * .public
    - mv .public public
  artifacts:
    paths:
      - public
  only:
    - main
```

Pages 默认情况下与分支/标签无关，它们的部署仅依赖于在 `.gitlab-ci.yml` 中指定的内容。因此设置 `only` 参数，只在 main 分支下进行部署。



#### 管理网站

我们创建一个 pages 分支，用来管理网站。

```shell
git checkout --orphan pages
```

在这个新分支上进行的第一次提交没有上级分支，并且是与所有其它分支和提交完全断开的新历史的根分支。将静态生成器的源文件推送到 `pages` 分支。

```yaml
image: ruby:2.6

pages:
  script:
    - gem install jekyll
    - jekyll build -d public/
  artifacts:
    paths:
      - public
  only:
    - pages
```



### Wiki

如果不想将文档保存在仓库中，但希望将其与代码保存在同一个项目中，您可以使用 Wiki 。每个 Wiki 都是一个单独的 Git 仓库，因此您可以在 Web 界面中创建 Wiki 页面，或者将其克隆到本地，修改后再通过 Git 推送。



#### 创建目录

要从 Wiki 页面的副标题生成目录，使用 `[_TOC_](_TOC_)` 标签。



#### 自定义侧边栏

至少需要 root 来自定义 Wiki 导航侧边栏。在页面右上角选择编辑侧边栏，这个过程会创建一个名为 `_sidebar` 的 Wiki 页面，它完全取代了默认的侧边栏导航。

```markdown
### [Home](home)

- [Hello World](hello)
- [Foo](foo)
- [Bar](bar)

---

- [Sidebar](_sidebar)
```


