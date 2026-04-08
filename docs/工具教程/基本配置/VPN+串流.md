# 本地 VPN

## 自建 VPN

快捷键 win + x 打开 PowerShell 输入下面指令执行

```shell
Add-VpnConnection -Name "ZJUVPN" -ServerAddress "lns.zju.edu.cn" -TunnelType "L2tp" -EncryptionLevel "Optional" -AuthenticationMethod ("Chap", "MSChapv2") -RememberCredential
```

现在可以在“设置-网络-VPN”中看到建立的 `ZJUVPN` 选项，输入 VPN 账户密码连接。



## 开机自启 + 自连

在 PowerShell 输入

```shell
ping lns.zju.edu.cn
```

将 `undead.ps1` 用记事本方式打开，其中

```bat
$VpnName = "ZJUVPN"
Write-Output "$(Get-Date) Start program"
$Ping = New-Object System.Net.NetworkInformation.Ping
# 这里 Address 改成得到的 ip 地址
$Address = "10.0.2.72"
while(1)
{
    $VpnStatus = (Get-VpnConnection -Name $VpnName).ConnectionStatus
    # Write-Output "$(Get-Date) VPN $VpnStatus"
    try
    {
        $LnsStatus = $Ping.Send($Address, 1000).Status
    }
    catch
    {
        $LnsStatus = "TimeOut"
    }
    # Write-Output "$(Get-Date) Lns $LnsStatus"

    if($LnsStatus -eq "Success")
    {
        if($VpnStatus -eq "Disconnected")
        {
            rasphone -d $VpnName
            Write-Output "$(Get-Date) Connceting"
        }
    }
    else
    {
        if($VpnStatus -eq "Connected")
        {
            rasphone -h $VpnName
            Write-Output "$(Get-Date) Disconnceting"
        }
    }
    sleep 1
}
```

将 `auto-start.vbs` 打开

```vb
WScript.CreateObject("wscript.shell").Run "PowerShell -ExecutionPolicy Bypass -WindowStyle Hidden -Command ""%USERPROFILE%\Documents\Scripts\zjuvpn.ps1 > %TEMP%\zjuvpn.log 2>&1""",0
```

其中 `%USERPROFILE%` 这一部分改成 `undead.ps1` 脚本的路径，放在开机自启动文件夹下

```shell
C:\Users\"用户名"\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
```



## 关闭 VPN 连接窗口

打开 PowerShell 执行如下命令

```shell
$vpnProfileFile = "$env:APPDATA\Microsoft\Network\Connections\Pbk\rasphone.pbk"
$vpnProfile = Get-Content -path $vpnProfileFile -Raw
$vpnProfile = $vpnProfile -replace "(?s)PreviewUserPw=1(?!.*?PreviewUserPw=1)", "${1}PreviewUserPw=0${2}"
$vpnProfile = $vpnProfile -replace "(?s)ShowDialingProgress=1(?!.*?ShowDialingProgress=1)", "${1}ShowDialingProgress=0${2}"
Set-Content -Value $vpnProfile -Path $vpnProfileFile
```



## 设置静态路由

打开 `静态路由设置 by shuishui.bat` 运行，运行后输入“1”，按回车。如果返回“已设置[1]的消息”，那么静态路由就设置成功了。



# Moonlight + Sunshine 串流

## 安装

在 GitHub 开源网站下载最新的 release 版本，找到 windows 安装器下载。

* 在主机端下载 [sunshine](https://github.com/LizardByte/Sunshine) 客户端
* 在控制端下载 [moonlight](https://github.com/moonlight-stream/moonlight-qt) 客户端



## 网络设置

连接非 secure 的校网或者 zjuwlan；将网络设置改为“专用”，允许访问；还需要设置静态路由，通过前面的方法设置。

![[image-20231202232912878.png]]

![[image-20231202232927709.png]]



## 配置 sunshine

打开 sunshine 后会跳转到网页界面，设置用户名和密码直接登录即可。



在主机端以管理员权限运行 power shell，执行

```sh
netsh advfirewall firewall add rule name="GameStream TCP" dir=in protocol=tcp localport=47984,47989,48010 action=allow
netsh advfirewall firewall add rule name="GameStream UDP" dir=in protocol=udp localport=5353,47998-48010 action=allow
```

让防火墙放行端口。



打开 GeForce Experience 设置，启用 GameStream，添加需要串流的游戏。我们设置为 `C:\Windows\System32\mstsc.exe`，这样就可以直接访问桌面。

![[image-20231202233112227.png]]



## 连接 moonlight

在控制端打开 moonlight，点击加号，输入前面网络界面中的 IPV4 地址即可连接。或者在命令行输入

```sh
ipconfig
```

即可查看网络信息。



## 虚拟屏

下载安装简易虚拟屏工具，首次启动需要安装驱动

```embed
title: "Release EasyVirtualDisplay 0.1 · KtzeAbyss/Easy-Virtual-Display"
image: "https://opengraph.githubassets.com/538d6c45ecbe8a66aedf21753fd6dca4499c1b2527975955c2cbb908d9360263/KtzeAbyss/Easy-Virtual-Display/releases/tag/0.1"
description: "一键创建你的专属虚拟显示器 Effortlessly Generate Your Personalized Virtual Display"
url: "https://github.com/KtzeAbyss/Easy-Virtual-Display/releases/tag/0.1"
```

进入 `D:\Sunshine\tools` 目录下，执行

```shell
$ .\dxgi-info.exe
```

可以看到显示信息

```shell
====== ADAPTER =====
Device Name      : NVIDIA GeForce RTX 2060
Device Vendor ID : 0x000010DE
Device Device ID : 0x00001F08
Device Video Mem : 5969 MiB
Device Sys Mem   : 0 MiB
Share Sys Mem    : 8108 MiB

    ====== OUTPUT ======
    Output Name       : \\.\DISPLAY1
    AttachedToDesktop : yes
    Resolution        : 1920x1080

    Output Name       : \\.\DISPLAY37
    AttachedToDesktop : yes
    Resolution        : 1920x1080

====== ADAPTER =====
Device Name      : NVIDIA GeForce RTX 2060
Device Vendor ID : 0x000010DE
Device Device ID : 0x00001F08
Device Video Mem : 5969 MiB
Device Sys Mem   : 0 MiB
Share Sys Mem    : 8108 MiB

    ====== OUTPUT ======
====== ADAPTER =====
Device Name      : NVIDIA GeForce RTX 2060
Device Vendor ID : 0x000010DE
Device Device ID : 0x00001F08
Device Video Mem : 5969 MiB
Device Sys Mem   : 0 MiB
Share Sys Mem    : 8108 MiB

    ====== OUTPUT ======
====== ADAPTER =====
Device Name      : Microsoft Basic Render Driver
Device Vendor ID : 0x00001414
Device Device ID : 0x0000008C
Device Video Mem : 0 MiB
Device Sys Mem   : 0 MiB
Share Sys Mem    : 8108 MiB

    ====== OUTPUT ======
```

打开 sunshine，将第二个输出名称填写在下面对应的位置即可

![[image-20240803144932243.png]]



# Zerotier 内网穿透

### 远程组网

首先注册 Zerotier 账号并登录

```embed
title: "ZeroTier Central"
image: "https://my.zerotier.com/img/ZT%20full%20logo%20gold%20white.png"
description: ""
url: "https://my.zerotier.com/"
```

登录后 Create A Network，然后复制 NETWORK ID

![[image-20241017102532444.png]]

在需要组网的机器上安装客户端软件

```embed
title: "Download - ZeroTier"
image: "https://www.zerotier.com/wp-content/uploads/2024/04/ZT-Primary-Logo%E2%80%93Gold-White.svg"
description: "Global Area Networking"
url: "https://www.zerotier.com/download/"
```

在任务栏找到图标，点击后选择 Join New Network，输入 NETWORK ID 加入网络

![[image-20241017102806519.png]]

回到网页，点击 NODES 查看连接设备。勾选设备，点击 Authorize 授权，然后 Refresh 信息，可以看到 Managed IPs 就是设备在网络中的地址。

![[image-20241017103120716.png]]



### WinSMB 共享

在要共享的设备上，选择文件夹右键“属性”，选择“共享-高级共享”

![[clipboard_2024-10-17_10-35.bmp]]

可以在其中设置访问和修改权限。最后在网络中输入 `//` 加上 Managed IPs 就可以建立连接，输入用户名和密码即可。



#### 服务端

在“控制面板”打开“程序-程序和功能-启动或关闭 Windows 功能”，确保 SMB 1.0 共享支持关闭。

![[image-20240219001738648.png]]

然后进入“用户账户”，点击“更改账户类型-在电脑设置中添加新用户”，进入后点击”添加账户“，依次选择“我没有这个人的登录信息-添加一个没有 Microsoft 账户的用户”

![[image-20240219001934770.png]]

创建完成后，右键选择要共享的文件夹，打开高级共享

![[image-20240219002722829.png]]

开启“共享此文件夹”，然后点击“权限”按钮，删除预设的 everyone 账户，添加之前创建的账户

![[image-20240219003403256.png]]



#### 客户端

记录之前设置好的“共享网络路径”

![[image-20240219003517778.png]]

控制面板进入“用户账户”，点击“管理 Windows 凭据-添加 Windows 凭据“，输入网络地址、用户名和密码即可

![[image-20240219003722146.png]]

进入“网络”文件夹，在上方输入之前的路径，右键需要映射的文件夹，选择“映射网络驱动器”，就可以在“此电脑”查看共享文件夹

![[image-20240219003903444.png]]

首先需要在网络和共享中心开启“公用文件夹共享”

![[image-20250404013332833.png]]

如果“映射网络驱动器”是灰色，可以通过命令行映射，例如

```shell
net use g: \\DESKTOP-QL404R6\d
```

将目标计算机的 d 盘映射到当前的 g 盘。



#### FTP 连接

在局域网下，手机端打开 MT 管理器，开启“远程管理”，在电脑端输入路径，即可在电脑上访问手机。



## Jellyfin

### 安装配置 

首先在官网下载服务端

```embed
title: "Jellyfin Repository"
image: "https://repo.jellyfin.org/repo-banner-dark.svg"
description: ""
url: "https://repo.jellyfin.org/?path=/server/windows/latest-stable/amd64"
```

开始安装，选择安装为服务，这样可以开机自启动。之后可以使用默认配置安装

![[image-20241017165608577.png]]

现在打开 `127.0.0.1:8096` 进入配置界面

![[image-20241017165834978.png]]

创建管理账号，添加资源库路径，使用默认配置即可

![[image-20241017170128864.png]]



还需要修改防火墙入站规则，这样能从外部访问到当前设备资源。打开高级安全，添加入站规则

![[image-20241017170836304.png]]

设置应用的端口号

![[image-20241017170935393.png]]

现在可以通过 `IP:PORT` 格式远程访问。



# 网络配置

## Ping

主机无法被 ping 时，可能是防火墙禁用了 ping，这时候可以在控制面板找到防火墙

![[image-20231215104101564.png]]

点击高级设置，在入站规则中点击新建规则，依次选择“自定义-所有程序-ICMPv4”，其余默认即可。最后指定名称，设置完成后即可 ping 。

![[image-20231215104127647.png]]
