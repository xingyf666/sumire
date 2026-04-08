# Inno Setup

Inno Setup 是一个开源免费的程序打包软件，具有友好的窗口交互方式用于创建安装程序。



## 脚本向导

创建新文件，使用 Inno Setup 的脚本向导创建简单的安装程序。如果点击“创建新的空脚本文件”，就会创建空的脚本；这里我们直接选择“下一步”：

![[image-20230611230402077.png]]

然后指定应用程序的基本信息。注意**黑色标识**的信息必填，其余选填。这一部分的信息不是关键信息，可以随意填写

![[image-20230611230510064.png]]

然后设置安装文件夹，默认安装为 C:/Program Files 目录（32 位则是 C:/Program Files (x86) ）。这里允许用户修改安装路径

![[image-20230611230704604.png]]

然后设置主执行文件，勾选“允许用户...”会在安装完成后，询问用户是否启动程序；可以添加文件，以及递归地添加文件夹

![[image-20230611230950143.png]]

注意在这里，选中文件夹，然后点击编辑，设置**目标子文件夹**，将这个文件夹下的文件安装到同名的文件夹中

![[image-20230611233020926.png]]

然后指定在开始菜单添加程序的快捷方式。如果勾选“允许用户创建桌面快捷方式”，就会询问用户是否要创建桌面快捷方式

![[image-20230611231152597.png]]

然后指定安装过程中显示的文档内容，需要注意文档的编码问题，如果内容有中文，则应当**使用 ACSII 编码**的文本文件

![[image-20230611231532677.png]]

然后指定安装模式，使用管理员安装模式会比较方便

![[image-20230611231608915.png]]

然后指定可以安装的语言

![[image-20230611231704225.png]]

这里指定编译脚本后得到的安装包要输出到哪个文件夹，这里我们将安装包放在桌面上，然后设置安装包的名称和图标。还可以指定安装时需要输入的密码

![[image-20230611231908812.png]]

之后点击下一步完成脚本生成，点击**构建-编译脚本**即可得到安装包。



## 参数详解

我们分析通过脚本向导生成的内容。第一部分

```pascal
#define MyAppName "main"
#define MyAppVersion "1.5"
#define MyAppPublisher "Me"
#define MyAppURL "http://www.example.com/"
#define MyAppExeName "main.exe"
```

定义软件基本信息的变量。



使用 `[]` 包围的内容称为**段**，用 `;` 开头的行是注释。不同的段具有相应的含义，例如 `[Setup]` 是设置段

```pascal
[Setup]
; 注: AppId的值为单独标识该应用程序。
; 不要为其他安装程序使用相同的AppId值。
; (若要生成新的 GUID，可在菜单中点击 "工具|生成 GUID"。)
; 随机生成的软件标记 GUID，具有相同 id 的软件被认为相同
AppId={{178B18AF-568E-4D57-967C-0CCA91E7A035}

; 应用基本信息，使用前面定义的宏赋值
AppName={#MyAppName}									
AppVersion={#MyAppVersion}
; AppVerName={#MyAppName} {#MyAppVersion}

; 开发者信息
AppPublisher={#MyAppPublisher}
AppPublisherURL={#MyAppURL}

; 支持网站和更新网站（这里设为和开发者网站相同）
AppSupportURL={#MyAppURL}
AppUpdatesURL={#MyAppURL}

; 默认路径安装，这里 autopf 是 Program Files 目录
DefaultDirName={autopf}\{#MyAppName}

; 不在开始菜单创建文件夹
DisableProgramGroupPage=yes

; 许可证文件位置
LicenseFile=F:\Desktop\test\LICENSE.txt
; 以下行取消注释，以在非管理安装模式下运行（仅为当前用户安装）。
; PrivilegesRequired=lowest

OutputDir=F:\Desktop			; 脚本编译后的输出路径
OutputBaseFilename=main-setup	; 输出安装包的名称
Compression=lzma				; 压缩文件格式
SolidCompression=yes			; 是否一次性压缩所有文件
WizardStyle=modern				; 安装包风格
```



语言段 `[Languages]` 设置安装语言，这里只有 chinesesimp 即简体中文

```pascal
[Languages]
Name: "chinesesimp"; MessagesFile: "compiler:Default.isl"
```



任务段 `[Tasks]`，包含创建快捷方式的设置

```pascal
[Tasks]
Name: "desktopicon"; Description: "{cm:CreateDesktopIcon}"; GroupDescription: "{cm:AdditionalIcons}"; Flags: unchecked
```



文件段 `[Files]` 设置资源文件目录和安装目录，Flags 设置特性，这里 `ignoreversion` 即表示忽略版本进行安装

```pascal
[Files]
Source: "F:\Desktop\test\main.exe"; DestDir: "{app}"; Flags: ignoreversion
Source: "F:\Desktop\test\README.md"; DestDir: "{app}"; Flags: ignoreversion
Source: "F:\Desktop\test\pictures\*"; DestDir: "{app}\pictures"; Flags: ignoreversion recursesubdirs createallsubdirs
; 注意: 不要在任何共享系统文件上使用“Flags: ignoreversion”
```



图标段 `[Icons]` 设置程序图标，Name 是快捷方式的名称，Filename 是程序所在的目录

```pascal
[Icons]
Name: "{autoprograms}\{#MyAppName}"; Filename: "{app}\{#MyAppExeName}"
Name: "{autodesktop}\{#MyAppName}"; Filename: "{app}\{#MyAppExeName}"; Tasks: desktopicon
```



运行段 `[Run]` 设置运行安装程序的配置

```pascal
[Run]
Filename: "{app}\{#MyAppExeName}"; Description: "{cm:LaunchProgram,{#StringChange(MyAppName, '&', '&&')}}"; Flags: nowait postinstall skipifsilent
```



## 更换安装图片

![[image-20230612001227802.png]]

在安装界面中左侧显示的图片可以修改。首先要准备两张 .bmp 格式的图片，然后在脚本的设置段中添加

```pascal
WizardImageFile=test.bmp
WizardSmallImageFile=small.bmp
```

两张图片分别对应左侧的显示图片以及右上角的小图标。



## Code 段

Inno Setup 支持通过 Pascal 语言编写的脚本，放在 Code 段中

```pascal
[Code]
; ...
```

通过 Code 段进行路径判断等操作。