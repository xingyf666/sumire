# PowerShell

## 界面设置

### 字体设置

打开注册表，找到

```
\HKEY_CURRENT_USER\Console\%SystemRoot%_System32_WindowsPowerShell_v1.0_powershell.exe
```

目录下的 codepage，设置为 `1b5`，即可在属性中设置更多字体。



## 基本命令

### 命令形式

PowerShell 中的命令以“动词+名词”的模式命名。例如

```powershell
get-help
```

获得帮助。为了兼容一些 cmd 命令，通过别名的方式达成这一目标。



PowerShell 中的命令**不区分大小写**，因此

```powershell
Get-Help
```

具有相同的效果。



### 第一条命令

在 PowerShell 中支持 Cmdlet 命令、CMD 命令、别名命令、各类脚本、VBS 脚本以及 Function 。但是要知道哪些命令可以执行，首先要了解的命令就是

```powershell
PS C:\Users\xyf> get-command

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Alias           Add-AppPackage                                     2.0.1.0    Appx
Alias           Add-AppPackageVolume                               2.0.1.0    Appx
Alias           Add-AppProvisionedPackage                          3.0        Dism
Alias           Add-ProvisionedAppPackage                          3.0        Dism
Alias           Add-ProvisionedAppxPackage                         3.0        Dism
Alias           Add-ProvisioningPackage                            3.0        Provisioning
Alias           Add-TrustedProvisioningCertificate                 3.0        Provisioning
Alias           Apply-WindowsUnattend                              3.0        Dism
Alias           Disable-PhysicalDiskIndication                     2.0.0.0    Storage
Alias           Disable-StorageDiagnosticLog                       2.0.0.0    Storage
Alias           Dismount-AppPackageVolume                          2.0.1.0    Appx
Alias           Enable-PhysicalDiskIndication                      2.0.0.0    Storage
Alias           Enable-StorageDiagnosticLog                        2.0.0.0    Storage
Alias           Flush-Volume                                       2.0.0.0    Storage
```

这条命令会列出所有别名、函数和 Cmdlet 命令。要查找需要的命令，可以根据目标对象，例如 user，使用通配符搜索

```powershell
PS C:\Users\xyf> get-command -Noun *user

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Cmdlet          Disable-LocalUser                                  1.0.0.0    Microsoft.PowerShell.LocalAccounts
Cmdlet          Enable-LocalUser                                   1.0.0.0    Microsoft.PowerShell.LocalAccounts
Cmdlet          Get-LocalUser                                      1.0.0.0    Microsoft.PowerShell.LocalAccounts
Cmdlet          New-LocalUser                                      1.0.0.0    Microsoft.PowerShell.LocalAccounts
Cmdlet          Remove-LocalUser                                   1.0.0.0    Microsoft.PowerShell.LocalAccounts
Cmdlet          Rename-LocalUser                                   1.0.0.0    Microsoft.PowerShell.LocalAccounts
Cmdlet          Set-LocalUser                                      1.0.0.0    Microsoft.PowerShell.LocalAccounts

```



### 获得命令

可以指定获得可以执行的**动词、名词**部分。例如获得所有动词

```powershell
PS C:\Users\xyf> get-verb

Verb        Group
----        -----
Add         Common
Clear       Common
Close       Common
Copy        Common
Enter       Common
Exit        Common
Find        Common
Format      Common
Get         Common
```

获得指定动词对应的命令

```powershell
PS C:\Users\xyf> get-command -verb update

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Function        Update-AutologgerConfig                            1.0.0.0    EventTracingManagement
Function        Update-Disk                                        2.0.0.0    Storage
Function        Update-DscConfiguration                            1.1        PSDesiredStateConfiguration
Function        Update-EtwTraceSession                             1.0.0.0    EventTracingManagement
Function        Update-HostStorageCache                            2.0.0.0    Storage
Function        Update-IscsiTarget                                 1.0.0.0    iSCSI
Function        Update-IscsiTargetPortal                           1.0.0.0    iSCSI
Function        Update-Module                                      1.0.0.1    PowerShellGet
Function        Update-ModuleManifest                              1.0.0.1    PowerShellGet
```

获得包含名词的命令

```powershell
PS C:\Users\xyf> get-command -noun type

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Cmdlet          Add-Type                                           3.1.0.0    Microsoft.PowerShell.Utility

```



### 帮助命令

使用 get-help 获得命令说明

```powershell
get-help get-childitem
```

如果对应的说明文档不存在，可以先更新帮助

```powershell
update-help
```

更具体地，可以指定参数

* -Detailed 给出参数使用格式的详解
* -Full 所有参数内容
* -Examples 显示案例
* -Parameter * 显示所有支持的参数
* -ShowWindow 通过窗口显示信息
* -Online 打开在线文档

例如查看所有参数

```powershell
PS C:\Users\xyf> get-help New-Item -Parameter *

-ApplicationName <System.String>
    This is a dynamic parameter made available by the WSMan provider. The WSMan provider and this parameter are only av
    ailable on Windows.

    Specifies the application name in the connection. The default value of the ApplicationName parameter is WSMAN .

    For more information, see New-WSManInstance (../Microsoft.WSMan.Management/New-WSManInstance.md).

    是否必需?                    False
    位置?                        named
    默认值                None
    是否接受管道输入?            False
    是否接受通配符?              False

```

查看指定参数

```powershell
PS C:\Users\xyf> get-help New-Item -Parameter value

-Value <System.Object>
    Specifies the value of the new item. You can also pipe a value to `New-Item`.

    是否必需?                    False
    位置?                        named
    默认值                None
    是否接受管道输入?            True (ByPropertyName, ByValue)
    是否接受通配符?              False

```



### 命令信息

配合管道操作可以获得命令的信息。例如获得命令的参数个数

```powershell
PS C:\Users\xyf> Get-Help Get-Command -Parameter * |measure


Count    : 14
Average  :
Sum      :
Maximum  :
Minimum  :
Property :

```

使用 more 获得更详细的信息，与直接使用 `-Parameter *` 的区别就在于它会**一段一段**地显示，而不是一次性输出所有内容。

```powershell
PS C:\Users\xyf> Get-Help Get-Command -Parameter * |more

-All <System.Management.Automation.SwitchParameter>
    Indicates that this cmdlet gets all commands, including commands of the same type that have the same name. By defau
    lt, `Get-Command` gets only the commands that run when you type the command name.

    For more information about the method that PowerShell uses to select the command to run when multiple commands have
     the same name, see about_Command_Precedence (About/about_Command_Precedence.md). For information about module-qual
    ified command names and running commands that do not run by default because of a name conflict, see about_Modules (
    About/about_Modules.md).

    This parameter was introduced in Windows PowerShell 3.0.

    In Windows PowerShell 2.0, `Get-Command` gets all commands by default.

    是否必需?                    False
    位置?                        named
    默认值                False
    是否接受管道输入?            True (ByPropertyName)
    是否接受通配符?              False
```



### 内置命令

| 命令                       | 作用          | 命令          | 作用               |
| ------------------------ | ----------- | ----------- | ---------------- |
| get-command              | 获得所有可使用的命令  | get-help    | 查找帮助手册           |
| get-childitem / ls / dir | 获得当前目录下的所有项 | new-item    | 创建新的项            |
| get-psprovider           | 获得 PS 接口    | get-member  | 获得参数（必须是对象）的属性方法 |
| write-output             | 输出文本        | get-content | 获得文件内容           |
| read-host                | 获得交互输入      |             |                  |



#### 新建项

使用 new-item 命令创建新的项，例如创建文件类型，指定创建路径

```powershell
PS C:\Users\xyf> new-item -ItemType File -Path E:\Desktop\test.txt


    目录: E:\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         2024/3/12     14:57              0 test.txt
```

可以看到 PowerShell 中参数通过 `-` 来传递。



#### 导入内容

使用 get-content 获得文本内容

```powershell
PS C:\Users\xyf> Get-Content .\test.txt
hello
my
name
is
Li
Hua.
```



#### 筛选内容

使用 select-string 选择要导入的内容。例如指定导入匹配 `lo` 的文本内容

```powershell
PS C:\Users\xyf> Select-String -Path .\test.txt -Pattern "lo" -AllMatches

test.txt:1:hello

```

可以保存匹配结果，然后遍历其中对象的属性

```powershell
PS C:\Users\xyf> $res=Select-String -Path .\test.txt -Pattern "o" -AllMatches
PS C:\Users\xyf> $res |foreach{$_.matches}


Groups   : {0}
Success  : True
Name     : 0
Captures : {0}
Index    : 4
Length   : 1
Value    : o


```

使用 `-context` 参数指定同时获取周围几行的内容

```powershell
PS C:\Users\xyf> $res=Select-String -Path .\test.txt -Pattern "o" -AllMatches -Context 2
PS C:\Users\xyf> $res

> test.txt:1:hello
  test.txt:2:my-name-is-Li-Hua


```

还有更多匹配参数，例如 `-NotMatch` 等。



#### 导出内容

使用 set-content 导出内容

```powershell
PS C:\Users\xyf> Set-Content -Value "hello world" .\test.txt
PS C:\Users\xyf> Get-Content .\test.txt
hello world
```

这样导出的内容会覆盖之前的所有内容。如果要追加内容，使用 add-content

```powershell
PS C:\Users\xyf> Add-Content -Value "hello world2" .\test.txt
PS C:\Users\xyf> Get-Content .\test.txt
hello world
hello world2
```



#### 结果输出

使用 set-content 只能指定具体的值，如果希望将命令输出的结果输出到文件中，就使用**支持管道符**的 out-file 命令

```powershell
PS C:\Users\xyf> "This is an example." |Out-File .\test.txt
PS C:\Users\xyf> Get-Content .\test.txt
This is an example.
```



#### 输入输出

使用 write-output 输出文本，read-host 读取输入

```powershell
PS C:\Users\xyf> Write-Output hello world
hello
world
PS C:\Users\xyf> $str=Read-Host "输入文本"
输入文本: test
```

如果不希望显示输入，可以使用 `-AsSecureString` 参数

```powershell
PS C:\Users\xyf> $str=Read-Host "输入文本" -AsSecureString
输入文本: ******
```



#### 输出管道

| 命令         | 作用           | 命令           | 作用            |
| ------------ | -------------- | -------------- | --------------- |
| write-output | 输出到管道     | out-file       | 输出到普通文件  |
| set-content  | 输出到普通文件 | export-csv     | 输出到 csv 文件 |
| out-printer  | 输出到打印机   | write-eventlog | 输出到日志      |
| add-content  | 追加到普通文件 |                |                 |



#### Provider

Provider 是 PowerShell 与数据存储路径和访问组件的一个标准接口。能够将访问的路径转换为 PowerShell 可以识别的访问接口。

```powershell
PS C:\Users\xyf> Get-PSProvider

Name                 Capabilities                                      Drives
----                 ------------                                      ------
Registry             ShouldProcess, Transactions                       {HKLM, HKCU}
Alias                ShouldProcess                                     {Alias}
Environment          ShouldProcess                                     {Env}
FileSystem           Filter, ShouldProcess, Credentials                {C, D, E}
Function             ShouldProcess                                     {Function}
Variable             ShouldProcess                                     {Variable}

```



#### PSDirve

Driver 与文件驱动器命名类似，所有显示在列表中的驱动都可以通过 cd 命令进入。

```powershell
PS Cert:\> Get-PSDrive

Name           Used (GB)     Free (GB) Provider      Root                                               CurrentLocation
----           ---------     --------- --------      ----                                               ---------------
Alias                                  Alias
C                 140.32         96.96 FileSystem    C:\                                                      Users\xyf
Cert                                   Certificate   \
D                 622.33        109.17 FileSystem    D:\
E                 158.10         41.90 FileSystem    E:\
Env                                    Environment
Function                               Function
HKCU                                   Registry      HKEY_CURRENT_USER
HKLM                                   Registry      HKEY_LOCAL_MACHINE
Variable                               Variable
WSMan                                  WSMan
```

还可以自己创建一个新的驱动盘，通过 Name 指定盘符，Root 指定路径，PSProvider 指定接口类型

```powershell
PS C:\Users\xyf> New-PSDrive -Name "t" -Root \\127.0.0.1\d$ -PSProvider "FileSystem"

Name           Used (GB)     Free (GB) Provider      Root                                               CurrentLocation
----           ---------     --------- --------      ----                                               ---------------
t                                      FileSystem    \\127.0.0.1\d$

PS C:\Users\xyf> Remove-PSDrive -Name "t"
```

最后删除这个驱动盘。



#### Alias

可以直接查看当前所有的别名

```powershell
PS E:\Desktop> Get-Alias

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Alias           % -> ForEach-Object
Alias           ? -> Where-Object
Alias           ac -> Add-Content
Alias           asnp -> Add-PSSnapin
Alias           bl -> blender.exe
Alias           cat -> Get-Content
Alias           cd -> Set-Location
Alias           CFS -> ConvertFrom-String                          3.1.0.0    Microsoft.PowerShell.Utility
Alias           chdir -> Set-Location
Alias           clc -> Clear-Content
Alias           clear -> Clear-Host
Alias           clhy -> Clear-History
Alias           cli -> Clear-Item
```

也可以通过 Provider 进入 alias 驱动查看

```powershell
PS E:\Desktop> cd alias:
PS Alias:\> ls

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Alias           % -> ForEach-Object
Alias           ? -> Where-Object
Alias           ac -> Add-Content
Alias           asnp -> Add-PSSnapin
Alias           bl -> blender.exe
Alias           cat -> Get-Content
Alias           cd -> Set-Location
Alias           CFS -> ConvertFrom-String                          3.1.0.0    Microsoft.PowerShell.Utility
Alias           chdir -> Set-Location
Alias           clc -> Clear-Content
Alias           clear -> Clear-Host
Alias           clhy -> Clear-History
Alias           cli -> Clear-Item
```



要设置别名，格式为

```powershell
PS Alias:\> Set-Alias lsall Get-ChildItem
```

最好将别名写在配置文件中，打开 code

```powershell
PS C:\Users\xyf> code $PROFILE
```

在其中设置好别名即可。注意别名只在当前环境中生效，一旦退出就会销毁。



## 运算符

### 重定向

使用 `>` 进行重定向，例如输出文本内容重定向到文件

```powershell
PS C:\Users\xyf> Write-Output hello, my name is Li Hua. > .\test.txt
PS C:\Users\xyf> Get-Content .\test.txt
hello
my
name
is
Li
Hua.
```



### 管道

使用管道将左侧命令的输出作为右侧命令的输入。例如

```powershell
PS C:\Users\xyf> Write-Output 1 | Get-Member


   TypeName:System.Int32

Name        MemberType Definition
----        ---------- ----------
CompareTo   Method     int CompareTo(System.Object value), int CompareTo(int value), int IComparable.CompareTo(Syste...
Equals      Method     bool Equals(System.Object obj), bool Equals(int obj), bool IEquatable[int].Equals(int other)
GetHashCode Method     int GetHashCode()
GetType     Method     type GetType()
GetTypeCode Method     System.TypeCode GetTypeCode(), System.TypeCode IConvertible.GetTypeCode()
ToBoolean   Method     bool IConvertible.ToBoolean(System.IFormatProvider provider)
ToByte      Method     byte IConvertible.ToByte(System.IFormatProvider provider)
ToChar      Method     char IConvertible.ToChar(System.IFormatProvider provider)
ToDateTime  Method     datetime IConvertible.ToDateTime(System.IFormatProvider provider)
ToDecimal   Method     decimal IConvertible.ToDecimal(System.IFormatProvider provider)
ToDouble    Method     double IConvertible.ToDouble(System.IFormatProvider provider)
ToInt16     Method     int16 IConvertible.ToInt16(System.IFormatProvider provider)
ToInt32     Method     int IConvertible.ToInt32(System.IFormatProvider provider)
ToInt64     Method     long IConvertible.ToInt64(System.IFormatProvider provider)
ToSByte     Method     sbyte IConvertible.ToSByte(System.IFormatProvider provider)
ToSingle    Method     float IConvertible.ToSingle(System.IFormatProvider provider)
ToString    Method     string ToString(), string ToString(string format), string ToString(System.IFormatProvider pro...
ToType      Method     System.Object IConvertible.ToType(type conversionType, System.IFormatProvider provider)
ToUInt16    Method     uint16 IConvertible.ToUInt16(System.IFormatProvider provider)
ToUInt32    Method     uint32 IConvertible.ToUInt32(System.IFormatProvider provider)
ToUInt64    Method     uint64 IConvertible.ToUInt64(System.IFormatProvider provider)

```

可以获得 1 这个参数的类型、属性以及方法。



#### 支持类型

通过 Get-Help 来查看某个参数是否支持管道符

```powershell
PS C:\Users\xyf> get-help Write-Host -Parameter object

-Object <System.Object>
    Objects to display in the host.

    是否必需?                    False
    位置?                        0
    默认值                None
    是否接受管道输入?            True (ByValue)
    是否接受通配符?              False

```

通常情况下，PowerShell 命令执行后不会返回任何结果，除非通过 `-PassThru` 参数指定。因此如果输出命令传递时报错，就可以添加这个参数。

> 只要是 Object 对象，都支持管道符。



#### 传递方式

管道符支持通过属性传递 ByProperty 和值传递 ByValue，如上面结果所示。

* ByProperty 表示将对象的所有属性通过管道符传递
* ByValue 将会把整个对象直接通过管道符传递

我们创建一个文本文件

```txt
name
property1
property2
property3
property4
```

然后将其导入，会得到 name 下的 4 个属性。

```powershell
PS C:\Users\xyf> Import-Csv .\test.txt

name
----
property1
property2
property3
property4

PS C:\Users\xyf> Get-Help New-Item -Parameter name

-Name <System.String>
    Specifies the name of the new item. You can specify the name of the new item in the Name or Path parameter value, a
    nd you can specify the path of the new item in Name or Path value. Items names passed using the Name parameter are
    created relative to the value of the Path parameter.

    是否必需?                    True
    位置?                        named
    默认值                None
    是否接受管道输入?            True (ByPropertyName)
    是否接受通配符?              False


```

由于 name 恰好是 New-Item 的参数名，因此现在可以通过管道符创建传递这些属性

```powershell
PS C:\Users\xyf> Import-Csv .\test.txt |New-Item


    目录: C:\Users\xyf


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         2024/3/13     17:41             17 property1
-a----         2024/3/13     17:41             17 property2
-a----         2024/3/13     17:41             17 property3
-a----         2024/3/13     17:41             17 property4

```

在对应文件夹下可以看到这些属性文件。

![[PowerShell.assets/image-20240313174224080.png|500]]



如果我们修改为

```txt
oldname
property1
property2
property3
property4
```

则它不能通过 New-item 直接创建，因为**读入的内容是整个对象的值而非它的属性**。这时候需要使用 `foreach` 遍历其中的属性来构造

```powershell
PS C:\Users\xyf> Import-Csv .\test.txt |foreach{New-Item -Name $_.oldname}


    目录: C:\Users\xyf


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         2024/3/13     17:44              0 property1
-a----         2024/3/13     17:44              0 property2
-a----         2024/3/13     17:44              0 property3
-a----         2024/3/13     17:44              0 property4

```



#### 异常处理

当管道过程出错时，会遵循默认的首选项设置。

```powershell
PS C:\Users\xyf> $DebugPreference
SilentlyContinue
PS C:\Users\xyf> $ErrorActionPreference
Continue
PS C:\Users\xyf> $VerbosePreference
SilentlyContinue
PS C:\Users\xyf> $WarningPreference
Continue
```

这些值都可以更改，例如 DebugPreference 可选

* continue 输出 debug 信息并继续执行
* stop 输出 debug 信息并停止，输出错误
* inquire 输出 debug 信息并询问是否继续
* silentlycontinue 没有影响，不输出任何信息，继续执行



#### 右过滤

使用 `-Filter` 结合管道符来筛选结果。例如筛选所有命令中，版本 version 为 1.0.0.0 的所有命令

```powershell
PS C:\Users\xyf> get-command -Filter *|where {$_.version -eq "1.0.0.0"}

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Function        Add-BCDataCacheExtension                           1.0.0.0    BranchCache
Function        Add-BitLockerKeyProtector                          1.0.0.0    BitLocker
Function        Add-DnsClientNrptRule                              1.0.0.0    DnsClient
Function        Add-DtcClusterTMMapping                            1.0.0.0    MsDtc
Function        Add-EtwTraceProvider                               1.0.0.0    EventTracingManagement
Function        Add-NetEventNetworkAdapter                         1.0.0.0    NetEventPacketCapture
Function        Add-NetEventPacketCaptureProvider                  1.0.0.0    NetEventPacketCapture
Function        Add-NetEventProvider                               1.0.0.0    NetEventPacketCapture
Function        Add-NetEventVFPProvider                            1.0.0.0    NetEventPacketCapture
Function        Add-NetEventVmNetworkAdapter                       1.0.0.0    NetEventPacketCapture
Function        Add-NetEventVmSwitch                               1.0.0.0    NetEventPacketCapture
Function        Add-NetEventVmSwitchProvider                       1.0.0.0    NetEventPacketCapture
Function        Add-NetEventWFPCaptureProvider                     1.0.0.0    NetEventPacketCapture
Function        Add-NetIPHttpsCertBinding                          1.0.0.0    NetworkTransition
Function        Add-NetNatExternalAddress                          1.0.0.0    NetNat
```

还可以使用通配符匹配

```powershell
PS C:\Users\xyf> Get-Process |where name -like "ut*"

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
    295      17    30756      60468       0.38   6916   1 uTools
    430      24    66820      47008       4.70  14416   1 uTools
    404      33    16568      41076      15.11  14540   1 uTools
    327      19    46932      65444       5.06  14668   1 uTools
   1872     151   123280     144868     114.02  15212   1 uTools
    281      16    31632      59536       0.41  17256   1 uTools
    312      18    43656      80896       1.14  18608   1 uTools
    273      16    28536      61512       0.48  19272   1 uTools
    272      16    31608      63048       0.11  21984   1 uTools
    279      16    28584      74624       0.19  22812   1 uTools
```



### 算术运算

#### 数字运算

基本的算术运算都有支持：`+,-,\*,/,%` 用于整型，`+,-,*,/` 用于小数。



#### 字符运算

使用 `+` 连接字符串，`*` 重复字符串

```powershell
PS C:\Users\xyf> "abc" + "cde"
abccde
PS C:\Users\xyf> "abc" * 3
abcabcabc
```



#### 组合类型运算

可以对组合类型使用 `+=` 添加元素

```powershell
PS C:\Users\xyf> $s = @(1,2,"a","b")
PS C:\Users\xyf> $s += "k"
PS C:\Users\xyf> $s
1
2
a
b
k
```



### 逻辑运算

常用逻辑运算如下

| 运算      | 作用     | 运算   | 作用     |
| --------- | -------- | ------ | -------- |
| -eq       | 等于     | -ne    | 不等于   |
| -lt       | 小于     | -gt    | 大于     |
| -le       | 小于等于 | -ge    | 大于等于 |
| -in       | 在其中   | -notin | 不在其中 |
| -like     | 模糊查询 | -match | 正则匹配 |
| -contains | 包含元素 |        |          |



例如在 if 语句中使用

```powershell
PS C:\Users\xyf> $a = "1"
PS C:\Users\xyf> $b = @("1","2",4)
PS C:\Users\xyf> if ($a -in $b)
>> {
>> Write-Output a in b
>> }
a
in
b
PS C:\Users\xyf> $s = "abc"
PS C:\Users\xyf> if ($s -like "*c")
>> {
>> Write-Output s like c
>> }
s
like
c
```



### 布尔运算

支持基本的布尔运算

* -and 与
* -or 或
* -xor 异或
* -not 非



## 对象

### 定义变量

PowerShell 中的变量都是弱类型，不需要定义变量类型，但是它仍然具有自动分配的对象类型。使用 `$` 指定变量

```powershell
PS C:\Users\xyf> $a = 123
PS C:\Users\xyf> $a
123
```

使用 `@` 指定数组变量，其中可以存放各种变量

```powershell
PS C:\Users\xyf> $a = @("1",2,"hello")
PS C:\Users\xyf> $a
1
2
hello
```



### 特殊变量

变量 `$_` 是一个特殊变量，在循环遍历时使用，依次表示每一个元素。例如

```powershell
PS C:\Users\xyf> Get-Content .\test.txt |foreach {$_}
hello world
PS C:\Users\xyf> Get-Content .\test.txt |foreach {$_ -replace "hello","Hello"}
Hello world
PS C:\Users\xyf> $a |foreach {$_}
1
2
hello
```



### 获得属性

对象都有属性，通过 get-member 获得指定对象的属性。例如获得匹配类型的属性

```powershell
PS E:\Desktop\blender> $a = "hello" | Select-String -Pattern "hello"
PS E:\Desktop\blender> Write-Output $a | Get-Member


   TypeName:Microsoft.PowerShell.Commands.MatchInfo

Name         MemberType Definition
----         ---------- ----------
Equals       Method     bool Equals(System.Object obj)
GetHashCode  Method     int GetHashCode()
GetType      Method     type GetType()
RelativePath Method     string RelativePath(string directory)
ToString     Method     string ToString(), string ToString(string directory)
Context      Property   Microsoft.PowerShell.Commands.MatchInfoContext Context {get;set;}
Filename     Property   string Filename {get;}
IgnoreCase   Property   bool IgnoreCase {get;set;}
Line         Property   string Line {get;set;}
LineNumber   Property   int LineNumber {get;set;}
Matches      Property   System.Text.RegularExpressions.Match[] Matches {get;set;}
Path         Property   string Path {get;set;}
Pattern      Property   string Pattern {get;set;}

```



对象属性可以直接使用。例如对于字符串对象，具有一些常用的操作和属性

```powershell
PS C:\Users\xyf> "hello-world".Length
11
PS C:\Users\xyf> "hello-world".Split("-")
hello
world
PS C:\Users\xyf> "hello-world".Remove(1,3)
ho-world
```

我们可以通过管道选择输出的部分属性，并且设置属性的 name 及其值。例如查看文件、路径和创建时间（当前时间减去创建时间）

```powershell
PS C:\Users\xyf> Get-ChildItem -Path E:\Desktop\ |select name,path,@{name="创建时间";expression={((Get-Date)-$_.creationtime).totaldays}}

Name                                          path         创建时间
----                                          ----         --------
ACIS                                               108.131545327472
Blender                                            14.2697519007685
cagd                                               7.20898205445833
DoxygenDemo                                        99.2484827858889
gitee                                              111.093428242012
github                                              107.13560357491
gitlab                                             91.0450178687349
GME-main                                           105.358581188825
OneDrive_1_2024-2-2                                39.0382484365729
QRT                                                59.4499491118183
spacemacs                                          93.0746814074097
周报                                               6.04622328375347
开源项目                                           2.90279214758218
海舟                                               59.5362799467257
裁剪曲面项目                                       106.293576957025
Are_Anime_Titties_Aerodynamic.pdf                  86.0460189448831
cuda_12.3.2_546.12_windows.exe                     17.4379699098275
CUDA编程基础与实践 -- 樊哲勇.pdf                   57.2948198150937
[NHD].Cyberpunk+2077+[FitGirl+Repack].torrent      41.4301737866007
[NHD].God_of_War-FLT.torrent                       41.4301066976539
番列表.txt                                         94.5103481687755
```



### 添加属性

可以通过 add-member 来为对象添加属性。首先创建对象

```powershell
PS C:\Users\xyf> $testObject=New-Object psobject
PS C:\Users\xyf> $testObject|Get-Member


   TypeName:System.Management.Automation.PSCustomObject

Name        MemberType Definition
----        ---------- ----------
Equals      Method     bool Equals(System.Object obj)
GetHashCode Method     int GetHashCode()
GetType     Method     type GetType()
ToString    Method     string ToString()
```

然后添加一个新的属性：名称为 size，类型为别名属性 AliasProperty，值为 length

```powershell
PS C:\Users\xyf> $testObject |Add-Member -MemberType AliasProperty -Name size -Value length
PS C:\Users\xyf> $testObject|Get-Member


   TypeName:System.Management.Automation.PSCustomObject

Name        MemberType    Definition
----        ----------    ----------
size        AliasProperty size = length
Equals      Method        bool Equals(System.Object obj)
GetHashCode Method        int GetHashCode()
GetType     Method        type GetType()
ToString    Method        string ToString()
```

另一种常用属性是 ScriptProperty，可以随着时间动态变化

```powershell
PS C:\Users\xyf> $testObject |Add-Member -MemberType ScriptProperty -Name "changeWithTime" -Value {(Get-Date)-$_.creationtime}
PS C:\Users\xyf> $testObject|Get-Member


   TypeName:System.Management.Automation.PSCustomObject

Name           MemberType     Definition
----           ----------     ----------
size           AliasProperty  size = length
Equals         Method         bool Equals(System.Object obj)
GetHashCode    Method         int GetHashCode()
GetType        Method         type GetType()
ToString       Method         string ToString()
changeWithTime ScriptProperty System.Object changeWithTime {get=(Get-Date)-$_.creationtime;}
```

静态属性 NoteProperty，这实际上就是一般的可赋值属性。



### 类型创建

常用的数据类型包括 `[wmi][wmiclass][ADSI][DateTime][XML]`，我们可以基于自定义的类型或系统默认类型创建我们的对象。

* 可以基于数据类型重定义。例如通过 `-as` 将字符型强制转换为 datetime 类型

```powershell
PS C:\Users\xyf> $i="2019-10-21" -as [datetime]
PS C:\Users\xyf> $i

2019年10月21日 0:00:00

PS C:\Users\xyf> 11.25 -as [int]
11
```

* 可以基于系统中的 COM 对象创建。例如将 wscript.shell 即 VBS 脚本创建为对象，并调用其方法

```powershell
PS C:\Users\xyf> $wsh=New-Object -ComObject wscript.shell
PS C:\Users\xyf> $wsh.Popup("hello")
1
PS C:\Users\xyf> $wsh |Get-Member


   TypeName:System.__ComObject#{41904400-be18-11d3-a28b-00104bd35090}

Name                     MemberType            Definition
----                     ----------            ----------
AppActivate              Method                bool AppActivate (Variant, Variant)
CreateShortcut           Method                IDispatch CreateShortcut (string)
Exec                     Method                IWshExec Exec (string)
ExpandEnvironmentStrings Method                string ExpandEnvironmentStrings (string)
LogEvent                 Method                bool LogEvent (Variant, string, string)
Popup                    Method                int Popup (string, Variant, Variant, Variant)
```



### 数组类型

使用数组对文本进行划分

```powershell
PS C:\Users\xyf> Write-Output hello,my-name-is-Li-Hua > .\test.txt
PS C:\Users\xyf> (Get-Content .\test.txt).Split(@("-",","))
hello
my
name
is
Li
Hua
```



### foreach-object

使用 foreach 遍历数组中的元素。

> foreach 具有别名 `%`，即可以使用 `%` 来替换 foreach 使用

具体语法为

```powershell
PS C:\Users\xyf> help ForEach-Object

名称
    ForEach-Object

摘要
    Performs an operation against each item in a collection of input objects.


语法
    ForEach-Object [-MemberName] <System.String> [-ArgumentList <System.Object[]>] [-InputObject <System.Management.Aut
    omation.PSObject>] [-Confirm] [-WhatIf] [<CommonParameters>]

    ForEach-Object [-Process] <System.Management.Automation.ScriptBlock[]> [-Begin <System.Management.Automation.Script
    Block>] [-End <System.Management.Automation.ScriptBlock>] [-InputObject <System.Management.Automation.PSObject>] [-
    RemainingScripts <System.Management.Automation.ScriptBlock[]>] [-Confirm] [-WhatIf] [<CommonParameters>]

```

可以指定初始块、循环块、结束块，例如

```powershell
PS C:\Users\xyf> 1..4 | foreach -Begin {"This is begin block."} -Process {"This is process block"} -End {"This is end block"}
This is begin block.
This is process block
This is process block
This is process block
This is process block
This is end block
```

可以省略参数名，按照顺序给出参数

```powershell
PS C:\Users\xyf> 1..4 | foreach {"This is begin block."} {"This is process block"} {"This is end block"}
```

其中在 begin 和 end 之间添加的块会循环执行，例如

```powershell
PS C:\Users\xyf> 1..4 | foreach {"This is begin block."} {"This is A block."} {"This is B block"} {"This is end block"}
This is begin block.
This is A block.
This is B block
This is A block.
This is B block
This is A block.
This is B block
This is A block.
This is B block
This is end block
```



借助 begin 和 end 可以实现嵌套循环，例如

```powershell
PS E:\Desktop\test> ls


    目录: E:\Desktop\test


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         2024/3/25     13:26              0 1.txt
-a----         2024/3/25     13:26              0 2.txt
-a----         2024/3/25     13:26              0 3.txt
-a----         2024/3/25     13:26              0 4.txt


PS E:\Desktop\test> Get-ChildItem | foreach -Begin {$daytime = Get-Date -Format FileDateTime | foreach {$_ -replace "T","t"} ; $char="_" ; $i = 1;} -Process {cp $_.FullName "E:\Desktop\$daytime$char$i.txt"; $i++} -End {Get-Date; "Complete!"}

2024年3月25日 13:28:44
Complete!

```

其中 `;` 用于分隔命令，我们可以将命令拆分为更明显的步骤

```powershell
Get-ChildItem | 
foreach -Begin 
{
	$daytime = Get-Date -Format FileDateTime | 
	# 循环替换文件名中的 T 为 t
	foreach 
	{
		$_ -replace "T","t"
	} ; 
	# 定义 char, i 两个变量
	$char="_" ; 
	$i = 1;
} -Process 
{
	# 复制文件，利用变量重命名，i 自增
	cp $_.FullName "E:\Desktop\$daytime$char$i.txt"; 
	$i++
} 
-End 
{
	Get-Date; 
	"Complete!"
}
```



### where-object

使用 where 筛选数组中的元素

```powershell
PS C:\Users\xyf> Get-Process |where name -like "ut*"

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
    295      17    30756      60468       0.38   6916   1 uTools
    430      24    66820      47008       4.70  14416   1 uTools
    404      33    16568      41076      15.11  14540   1 uTools
    327      19    46932      65444       5.06  14668   1 uTools
   1872     151   123280     144868     114.02  15212   1 uTools
    281      16    31632      59536       0.41  17256   1 uTools
    312      18    43656      80896       1.14  18608   1 uTools
    273      16    28536      61512       0.48  19272   1 uTools
    272      16    31608      63048       0.11  21984   1 uTools
    279      16    28584      74624       0.19  22812   1 uTools
```



## 脚本

### 环境配置

Powershell 定义的环境配置文件有 4 个不同的地方，分别对应不同的配置需求。

| 配置文件类型                       | 配置文件路径                         |
| ---------------------------- | ------------------------------ |
| Current user, PowerShell ISE | `$PROFILE`                     |
| All user, PowerShell ISE     | `$PROFILE.CurrentUserAllHosts` |
| Current user, All hosts      | `$PROFILE.CurrentUserAllHosts` |
| All usera, All hosts         | `$PROFILE.AllUsersAllHosts`    |

只需要执行 `. $PROFILE` 即可通过报错信息看到配置文件路径。输入

```powershell
notepad $PROFILE
```

在记事本中打开配置文件。设置

```powershell
function Prompt { "PS> " }
```

表示只显示 `PS>` 作为命令行开头，不显示完整路径。



### 运行脚本

直接将命令存放到 `.ps1` 文件中，双击即可执行。需要注意修改执行策略。用管理员模式运行 PowerShell，执行

```powershell
Set-ExecutionPolicy RemoteSigned
```

然后可以在 `$PROFILE` 对应路径下设置窗口属性

```powershell
# 设置窗口属性
$Shell = $Host.UI.RawUI

$size = $Shell.BufferSize
$size.width = 5000
$size.height = 5000
$size = $Shell.WindowSize
$size.width = 120
$size.height = 40

$shell.BackgroundColor = "Black"
$shell.ForegroundColor = "white"

$curComp = (Get-ChildItem Env:\COMPUTERNAME).Value
$shell.WindowTitle = "PS $((Get-Location).Path)"
```

然后在 powershell 中执行

```powershell
. $PROFILE
```

就可以完成配置。



### 脚本参数

在脚本前添加 `param()` 来设定变量，即参数默认值，如果没有输入对应的参数，就会使用给出的参数值。例如

```powershell
param(
$MyDate = "aaaa-bb-cc"
)

Write-Output $MyDate
```

然后可以通过如下方式调用

```powershell
PS E:\Desktop\test> .\test.ps1
aaaa-bb-cc
PS E:\Desktop\test> .\test.ps1 hello
hello
PS E:\Desktop\test> .\test.ps1 -MyDate hello
hello
```



### 脚本函数

可以通过 function 定义脚本中的函数，对应的函数可以直接在 PowerShell 中调用

```powershell
# 设定别名
sal p python
sal bl "D:\Blender 4.0\blender.exe"

function render($path, $sample){
    $config_py = $(cat C:\Users\xyf\Documents\WindowsPowerShell\config_template).replace('$$', $sample)
    sc C:\Users\xyf\Documents\WindowsPowerShell\config.py $config_py
    bl -b $path -P C:\Users\xyf\Documents\WindowsPowerShell\config.py -f 1 -o result -- --cycles-device CUDA
}

function batch-render($path, $arr){
    $arr | % {
        render $path $_
        mv 0001.exr "$_.exr" -force
    }
}
```



### 帮助文档

可以为自定义脚本添加文档，在脚本最前面添加

```powershell
<#

.SYNOPSIS
Some thing.

.DESCRIPTION
Some thing.

.EXAMPLE
.\test.ps1 hello

.EXAMPLE
.\test.ps1 -MyDate hello

#>

param(
$MyDate = "aaaa-bb-cc"
)

Write-Output $MyDate
```

调用方法为

```powershell
PS E:\Desktop\test> help .\test.ps1

名称
    E:\Desktop\test\test.ps1

摘要
    Some thing.


语法
    E:\Desktop\test\test.ps1 [[-MyDate] <Object>] [<CommonParameters>]


说明
    Some thing.


相关链接

备注
    若要查看示例，请键入: "get-help E:\Desktop\test\test.ps1 -examples".
    有关详细信息，请键入: "get-help E:\Desktop\test\test.ps1 -detailed".
    若要获取技术信息，请键入: "get-help E:\Desktop\test\test.ps1 -full".


```



## 数据呈现

### 格式化输出

有 4 种不同的格式化方法

* Format-list (fl) 以列表方式显示数据
* Format-table (ft) 以表格方式显示数据（默认）
* Format-wide (fw) 以指定宽度显示数据
* Format-Custom (fc) 高级格式化，一般不需要



#### fl

列表显示指定的属性

```powershell
PS C:\Users\xyf> Get-ChildItem |fl -Property name


Name : .cache

Name : .config

Name : .designer

Name : .ipython

Name : .keras
```

指定分组属性，会将属性值相同的项归为一类

```powershell
PS C:\Users\xyf> Get-ChildItem |fl -GroupBy mode


    目录: C:\Users\xyf



Name           : .cache
CreationTime   : 2023/12/8 10:14:18
LastWriteTime  : 2023/12/8 10:14:18
LastAccessTime : 2024/3/17 21:56:06
Mode           : d-----
LinkType       :
Target         : {}

Name           : .config
CreationTime   : 2023/11/22 20:42:40
LastWriteTime  : 2023/11/22 20:42:40
LastAccessTime : 2024/3/17 21:56:06
Mode           : d-----
LinkType       :
Target         : {}

Name           : .designer
```



#### ft

表格可以指定

* -AutoSize 自动改变显示大小
* -Wrap 自动换行
* -RepeatHeader 当显示内容超出屏幕，会重复显示表头



#### fw

例如显示 4 列数据

```powershell
PS C:\Users\xyf> Get-ChildItem |fw -Column 4
```

还可以指定 `-Property` 等属性来显示。



### 选择性输出

使用 select 指定输出的属性

```powershell
PS C:\Users\xyf> Get-ChildItem -Path E:\Desktop\ |select name,path,@{name="创建时间";expression={((Get-Date)-$_.creationtime).totaldays}}

Name                                          path         创建时间
----                                          ----         --------
ACIS                                               108.131545327472
Blender                                            14.2697519007685
cagd                                               7.20898205445833
DoxygenDemo                                        99.2484827858889
gitee                                              111.093428242012
github                                              107.13560357491
gitlab                                             91.0450178687349
GME-main                                           105.358581188825
OneDrive_1_2024-2-2                                39.0382484365729
QRT                                                59.4499491118183
spacemacs                                          93.0746814074097
周报                                               6.04622328375347
开源项目                                           2.90279214758218
海舟                                               59.5362799467257
裁剪曲面项目                                       106.293576957025
Are_Anime_Titties_Aerodynamic.pdf                  86.0460189448831
cuda_12.3.2_546.12_windows.exe                     17.4379699098275
CUDA编程基础与实践 -- 樊哲勇.pdf                   57.2948198150937
[NHD].Cyberpunk+2077+[FitGirl+Repack].torrent      41.4301737866007
[NHD].God_of_War-FLT.torrent                       41.4301066976539
番列表.txt                                         94.5103481687755
```



### 数据排序

使用 sort 指定排序属性来对数据进行排序（默认按名称字母排序）

```powershell
PS C:\Users\xyf> Get-ChildItem |Sort-Object -Property Length


    目录: C:\Users\xyf


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         2023/12/1     18:13             45 .git-credentials
-a----         2024/3/17     20:58            402 .gitconfig
-a----          2024/1/5     10:05           1179 .viminfo
d-r---        2023/11/22     19:39                Music
d-r---        2023/11/22     19:42                OneDrive
d-r---         2024/1/12     18:28                Pictures
d-r---        2023/11/22     19:39                Links
d-r---         2024/3/12     10:33                Documents
d-----        2023/11/28     10:39                Downloads
```



### 数据统计

使用 measure 统计数据，可以指定不同参数，统计不同内容。使用 get-content 获得文件内容，通过管道符进行统计

```powershell
PS C:\Users\xyf> Get-ChildItem |Measure-Object


Count    : 33
Average  :
Sum      :
Maximum  :
Minimum  :
Property :



PS C:\Users\xyf> Get-ChildItem |Measure-Object -Character

Lines Words Characters Property
----- ----- ---------- --------
                   270
                   
PS C:\Users\xyf> Get-Content .\test.txt
hello
my
name
is
Li
Hua.
PS C:\Users\xyf> Get-Content .\test.txt |Measure-Object


Count    : 6
Average  :
Sum      :
Maximum  :
Minimum  :
Property :


```



### 数据分组

使用 group 指定属性，对数据分组

```powershell
PS C:\Users\xyf> Get-ChildItem | Sort-Object -Property Extension | Group-Object -Property Extension

Count Name                      Group
----- ----                      -----
   19                           {ansel, Calibre Library, Links, 3D Objects...}
    1 .cache                    {.cache}
    1 .config                   {.config}
    1 .designer                 {.designer}
    1 .gitconfig                {.gitconfig}
    1 .git-credentials          {.git-credentials}
    1 .ipython                  {.ipython}
    1 .keras                    {.keras}
    1 .matplotlib               {.matplotlib}
    1 .ssh                      {.ssh}
    1 .thumbnails               {.thumbnails}
    1 .txt                      {test.txt}
    1 .viminfo                  {.viminfo}
    1 .VirtualBox               {.VirtualBox}
    1 .vs                       {.vs}
    1 .vscode                   {.vscode}

```



### 正则表达式

PowerShell 中正则表达式的基本语法，与 Python 一致。使用 `-match` 或通过管道符接 `Select-String` 进行匹配

```powershell
PS E:\Desktop\test> "powershell" -match "p[a-z]wershell"
True
PS E:\Desktop\test> Get-ChildItem | Select-String -Pattern "\d"

test.ps1:10:.\test.ps1 hello
test.ps1:13:.\test.ps1 -MyDate hello

```



