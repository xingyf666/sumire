# Sigil

## 基本介绍

Sigil 用于编写电子书并导出 epub 格式文件。建议下载 `0.9.9` 版本而不是最新的版本安装。



### 书籍视图

新版本取消了书籍视图功能，将其移动到另一个 PageEdit 软件中。所以新版需要先下载 PageEdit 软件，然后在 Sigil 的“编辑-首选项”中修改“首选的外部xhtml编辑器”。

![](1.png)

点击图标栏中的 X 图标即可开启书籍视图。



### 格式介绍

EPUB 格式本质上是一个压缩包，其中包含了几个主要部分

![](image-20240131151654803.png)

在 mimetype 中指出这个文件不是 zip 压缩包而是 epub 文件

```
application/epub+zip
```

在 META-INF 文件夹中的 container.xml 中给出具体的信息

```xml
<?xml version='1.0' encoding='utf-8'?>
<container xmlns="urn:oasis:names:tc:opendocument:xmlns:container" version="1.0">
  <rootfiles>
    <rootfile media-type="application/oebps-package+xml" full-path="EPUB/content.opf"/>
  </rootfiles>
</container>
```

例如 full-path 变量给出内容的路径。对应的 content.opf 给出文件的信息

* metadata 表示元数据
* manifest 声明文件内容来源。注意某些不规范的电子书没有声明图片路径，可能会导致图片无法读取
* spine 表明数据清单，指定打开章节的顺序
* guide 表示引导信息，声明封面、目录等特殊页面

下面是一个 epub 文件的完整内容

```xml
<?xml version='1.0' encoding='utf-8'?>
<package xmlns="http://www.idpf.org/2007/opf" unique-identifier="bookid" version="3.0" prefix="rendition: http://www.idpf.org/vocab/rendition/#">
    
	<!-- 这一部分是元数据 -->
  <metadata xmlns:dc="http://purl.org/dc/elements/1.1/" xmlns:opf="http://www.idpf.org/2007/opf">
    <meta property="dcterms:modified">2022-01-06T11:42:58Z</meta>
    <meta name="cover" content="cover-image"/>
    <dc:title>宇宙的最后三分钟</dc:title>
    <dc:creator opf:role="aut" opf:file-as="［澳］保罗·戴维斯（Paul Davies）;高晓鹰译">［澳］保罗·戴维斯（Paul Davies）;高晓鹰译</dc:creator>
    <dc:identifier id="bookid">urn:uuid:273fd756-62f2-4858-8d67-99e08f24bba9</dc:identifier>
    <dc:identifier opf:scheme="ASIN">77737290-3985-4e66-9666-add3fa4e6f6c</dc:identifier>
    <dc:contributor opf:file-as="CompanyName" opf:role="own">Epubor</dc:contributor>
    <dc:contributor opf:file-as="PersonalName" opf:role="own">Ultimate</dc:contributor>
    <dc:contributor opf:file-as="eCore" opf:role="bkp">eCore v0.9.12.751 [ http://www.epubor.com/ecore.html ]</dc:contributor>
    <dc:contributor opf:file-as="SiteURL" opf:role="own">http://www.epubor.com</dc:contributor>
    <dc:date>2020-07-14T16:00:00+00:00</dc:date>
    <dc:publisher>cj5_1865</dc:publisher>
    <dc:language>zh</dc:language>
  </metadata>
    
    <!-- 这一部分是文件清单，声明 epub 是由哪些文件构成的，包括文本、图片、样式等 -->
  <manifest>
    <item href="text00000.html" id="id_1" media-type="application/xhtml+xml"/>
    <item href="text00001.html" id="id_2" media-type="application/xhtml+xml"/>
    <item href="text00002.html" id="id_3" media-type="application/xhtml+xml"/>
    <item href="text00003.html" id="id_4" media-type="application/xhtml+xml"/>
    <item href="text00004.html" id="id_5" media-type="application/xhtml+xml"/>
    <item href="text00005.html" id="id_6" media-type="application/xhtml+xml"/>
    <item href="text00006.html" id="id_7" media-type="application/xhtml+xml"/>
    <item href="text00007.html" id="id_8" media-type="application/xhtml+xml"/>
    <item href="text00008.html" id="id_9" media-type="application/xhtml+xml"/>
    <item href="text00009.html" id="id_10" media-type="application/xhtml+xml"/>
    <item href="text00010.html" id="id_11" media-type="application/xhtml+xml"/>
    <item href="text00011.html" id="id_12" media-type="application/xhtml+xml"/>
    <item href="text00012.html" id="id_13" media-type="application/xhtml+xml"/>
    <item href="text00013.html" id="id_14" media-type="application/xhtml+xml"/>
    <item href="text00014.html" id="id_15" media-type="application/xhtml+xml"/>
    <item href="text00015.html" id="id_16" media-type="application/xhtml+xml"/>
    <item href="text00016.html" id="id_17" media-type="application/xhtml+xml"/>
    <item href="text00017.html" id="id_18" media-type="application/xhtml+xml"/>
    <item href="text00018.html" id="id_19" media-type="application/xhtml+xml"/>
    <item href="text00019.html" id="id_20" media-type="application/xhtml+xml"/>
    <item href="text00020.html" id="id_21" media-type="application/xhtml+xml"/>
    <item href="text00021.html" id="id_22" media-type="application/xhtml+xml"/>
    <item href="text00022.html" id="id_23" media-type="application/xhtml+xml"/>
    <item href="text00023.html" id="id_24" media-type="application/xhtml+xml"/>
    <item href="text00024.html" id="id_25" media-type="application/xhtml+xml"/>
    <item href="text00025.html" id="id_26" media-type="application/xhtml+xml"/>
    <item href="text00026.html" id="id_27" media-type="application/xhtml+xml"/>
    <item href="text00027.html" id="id_28" media-type="application/xhtml+xml"/>
    <item href="text00028.html" id="id_29" media-type="application/xhtml+xml"/>
    <item href="text00029.html" id="id_30" media-type="application/xhtml+xml"/>
    <item href="toc.ncx" id="ncx" media-type="application/x-dtbncx+xml"/>
    <item href="flow0001.css" id="id_31" media-type="text/css"/>
    <item href="flow0002.css" id="id_32" media-type="text/css"/>
    <item href="Image00000.jpg" id="id_33" media-type="image/jpeg"/>
    <item href="Image00001.jpg" id="id_34" media-type="image/jpeg"/>
    <item href="Image00002.jpg" id="id_35" media-type="image/jpeg"/>
    <item href="Image00003.jpg" id="id_36" media-type="image/jpeg"/>
    <item href="Image00004.jpg" id="cover-image" media-type="image/jpeg" properties="cover-image"/>
    <item href="Image00005.jpg" id="id_38" media-type="image/jpeg"/>
    <item href="Image00006.jpg" id="id_39" media-type="image/jpeg"/>
    <item href="Image00007.jpg" id="id_40" media-type="image/jpeg"/>
    <item href="Image00008.jpg" id="id_41" media-type="image/jpeg"/>
    <item href="Image00009.jpg" id="id_42" media-type="image/jpeg"/>
    <item href="Image00010.jpg" id="id_43" media-type="image/jpeg"/>
    <item href="Image00011.jpg" id="id_44" media-type="image/jpeg"/>
    <item href="Image00012.jpg" id="id_45" media-type="image/jpeg"/>
    <item href="Image00013.jpg" id="id_46" media-type="image/jpeg"/>
    <item href="Image00014.jpg" id="id_47" media-type="image/jpeg"/>
    <item href="Image00015.jpg" id="id_48" media-type="image/jpeg"/>
    <item href="Image00016.jpg" id="id_49" media-type="image/jpeg"/>
    <item href="Image00017.jpg" id="id_50" media-type="image/jpeg"/>
    <item href="Image00018.jpg" id="id_51" media-type="image/jpeg"/>
    <item href="Image00019.jpg" id="id_52" media-type="image/jpeg"/>
    <item href="Image00020.jpg" id="id_53" media-type="image/jpeg"/>
    <item href="Image00021.jpg" id="id_54" media-type="image/jpeg"/>
    <item href="Image00022.jpg" id="id_55" media-type="image/jpeg"/>
    <item href="Image00023.jpg" id="id_56" media-type="image/jpeg"/>
    <item href="Image00024.jpg" id="id_57" media-type="image/jpeg"/>
    <item href="Image00025.jpg" id="id_58" media-type="image/jpeg"/>
    <item href="Image00026.jpg" id="id_59" media-type="image/jpeg"/>
    <item href="Image00027.jpg" id="id_60" media-type="image/jpeg"/>
    <item href="Image00028.jpg" id="id_61" media-type="image/jpeg"/>
    <item href="Image00029.jpg" id="id_62" media-type="image/jpeg"/>
    <item href="Image00030.jpg" id="id_63" media-type="image/jpeg"/>
    <item href="Image00031.jpg" id="id_64" media-type="image/jpeg"/>
    <item href="Image00032.jpg" id="id_65" media-type="image/jpeg"/>
    <item href="Image00033.jpg" id="id_66" media-type="image/jpeg"/>
    <item href="Image00034.jpg" id="id_67" media-type="image/jpeg"/>
    <item href="image/003872.jpg" id="static_0" media-type="image/jpeg"/>
    <item href="003872.xhtml" id="chapter_0" media-type="application/xhtml+xml"/>
    <item href="ad_chapter15.xhtml" id="chapter_1" media-type="application/xhtml+xml"/>
    <item href="ad_chapter30.xhtml" id="chapter_2" media-type="application/xhtml+xml"/>
  </manifest>
    
    <!-- 这一部分是数据清单，按照顺序打开不同的章节 -->
  <spine toc="ncx">
    <itemref idref="id_1"/>
    <itemref idref="id_2"/>
    <itemref idref="id_3"/>
    <itemref idref="id_4"/>
    <itemref idref="id_5"/>
    <itemref idref="id_6"/>
    <itemref idref="id_7"/>
    <itemref idref="id_8"/>
    <itemref idref="id_9"/>
    <itemref idref="id_10"/>
    <itemref idref="id_11"/>
    <itemref idref="id_12"/>
    <itemref idref="id_13"/>
    <itemref idref="id_14"/>
    <itemref idref="id_15"/>
    <itemref idref="chapter_1"/>
    <itemref idref="id_16"/>
    <itemref idref="id_17"/>
    <itemref idref="id_18"/>
    <itemref idref="id_19"/>
    <itemref idref="id_20"/>
    <itemref idref="id_21"/>
    <itemref idref="id_22"/>
    <itemref idref="id_23"/>
    <itemref idref="id_24"/>
    <itemref idref="id_25"/>
    <itemref idref="id_26"/>
    <itemref idref="id_27"/>
    <itemref idref="id_28"/>
    <itemref idref="id_29"/>
    <itemref idref="chapter_2"/>
    <itemref idref="chapter_0"/>
    <itemref idref="id_30"/>
  </spine>
    
    <!-- 这一部分是引导信息，表明特殊页面 -->
  <guide>
    <reference type="text" title="封面页" href="text00000.html#coverpage"/>
    <reference type="toc" title="目录" href="text00003.html"/>
  </guide>
</package>
```



### 导入文本

打开文件后，注意图书浏览器上方的两个图标，分别表示书籍视图和代码视图。

![](image-20240131172225916.png)

删除所有代码，然后在书籍视图下，粘贴文本，然后切换到文本视图，就能得到转换后的代码。

![](image-20240131172414176.png)

但是这样会得到 div 标签。通常我们希望使用段落语义 p 标签，因此应该保留原始代码

```html
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN"
  "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<title></title>
</head>
<body>
<p>&nbsp;</p>
</body>
</html>
```

**将光标放在 p 标签中间**，然后在书籍视图下粘贴

![](image-20240131210115613.png)



### 拆分章节

在代码视图下，找到需要拆分的位置，插入

```html
<hr class="sigil_split_marker" />
```

然后按 F6 就可以拆分章节。每一章节的标题移动到 title 标签之间

```html
<html xmlns="http://www.w3.org/1999/xhtml">
<head><title>1.向着伊修地区！择库洛姆的影子！异变！</title>
</head>
```

也可以在编辑菜单中选择在光标处拆分。



### 样式表

在 Styles 文件夹位置右键创建新的样式表

```css
#first {
	color: green;
}
```

然后右键选择 xhtml 文件选择链接样式表，就可以在标签中加入样式

```html
<div id="first">2.神奇宝贝系统？超控状态？（上）</div>
```



### 插入图片

首先需要声明图片，在 Images 文件夹右键选择添加现有文件，导入图片资源。然后在 xhtml 文件中指定位置处停留光标，点击插入图片的图标就可以自动插入一个 img 标签。

![image-20240131174723408](image-20240131174723408.png)

这时插入的图片大小还是原本的大小，为了与页面保持一致，应该添加图片样式，例如

```css
.image img{
	width: 100%;
}
```

指定 image 类下的所有 img 标签具有 100% 宽度。如果需要让图片占据整个页面，就需要为其单独创建页面。



### 添加封面

在 content.opf 文件中，将光标停留在

```html
<guide>
</guide>
```

标签之后，然后选择需要作为封面的 xhtml 文件（通常命名为 cover），右键选择添加语义，选择封面选项即可创建封面。



### 编辑目录

创建制作信息、简介和目录的 xhtml 文件，利用超链接实现页面跳转。现在点击“工具-目录-编辑目录”可以按照顺序设置目录条目

![](image-20240131205412806.png)

然后调整 xhtml 文件的顺序，将封面、制作信息、简介页面放在前面，然后是彩插，接着是目录，最后是正文。



### 元数据

在“工具-元数据编辑器”处启动编辑，也可以通过 F8 快捷键启动，编辑文件的各种信息。



## 生成样例

### 创建书籍

以同人文《神奇宝贝系统》为例，首先创建新的 epub 文件，然后用 Sigil 打开。创建新的空白 xhtml 文件，粘贴全文

![](image-20240131211334463.png)

虽然段落提供了空格缩进，但是我们有更好的方法实现缩进，因此先去掉所有空格。使用正则表达式替换掉所有空格即可。



### 拆分段落

由于章节数量过多，因此需要先进行拆分。使用正则表达式标记所有章节位置，在前面添加拆分标记。根据章节名格式得到表达式

```shell
<p>([0-9]+\..*)</p>
```

注意到章节存在重复，因此删除多余的章节名。现在替换目标为

```shell
<h1>\1</h1>
```

即将匹配的 `()` 中的元素不动，放到 h1 标签中。在 h1 标签前面添加拆分标记，将 `<h1>` 替换为

```shell
<hr class="sigil_split_marker" />\n<h1>
```

然后按 F6 进行拆分。



### 搜索模板

上述搜索替换过程可能较为费事，对于大量重复的替换过程，可以在“工具-搜索模板”中创建搜索替换流程，这样就可以一键完成所有替换过程。右键创建组，在组中创建条目。点击“加载搜索”，然后“替换所有”即可完成整个流程。

![](image-20240208135257681.png)



### 替换标题

在 title 标签中需要填充章节名称，首先设置替换格式

```shell
(?s)<title></title>(.*<h1>(.*)</h1>)
<title>\2</title>\1
```

其中第一个表达式在单行模式 `(?s)` 下匹配从 title 标签到 h1 标签的所有内容，第二个表达式将中间 `(.*)` 匹配的内容填充到 title 标签中，将 title 后面的所有内容填充到 title 后面不变。现在修改模式后面的选项，实现全部替换。

![image-20240131220651428](image-20240131220651428.png)



### 重命名

将选中文件重命名来区分章节与其它类型的文件。右键选择重命名，按照格式 chapter-1 进行批量重命名。



### 插入图片

在指定文本位置插入图片。创建样式表，设置宽度填充

```css
img{
	width: 100%;
}
```



### 生成目录

可以自动为 h1 标签生成目录。只需要将所有章节目录填充在 h1 标签中，就可以在“工具-目录-生成目录”中自动生成目录。



### 创建封面

创建 cover.xhtml 文件，右键添加语义，设为封面。右键选择想要作为封面的图片，设为封面图片。



## 精排制作

### 全局设定

首先我们需要对文本和图片的样式进行全局设定，具体设定方式如下：

![](Screenshot_20240201_103727.jpg)



#### 添加框架

首先将 body 中的内容添加到 div 标签中

```shell
(?s)<body>(.*)</body>
<body>\n<div class="wrap color">\n\1</div>\n</body>
```

然后将图片放在 div 标签中

```shell
<img.*>
<div class="images">\0</div>
```



#### 设定样式

然后创建新的样式表，按照下面设置样式

```css
@charset "UTF-8";

/* 全局设定 */

/* 重置 CSS */

* {
	margin: 0;
	padding: 0;
	border: none;
}

ul, ol {
	list-style: none;
}

a {
	text-decoration: none;
}

/* Warp */

.warp {
	/* 一行最多 40 字符*/
	max-width: 40em;
	margin: 0 auto;
}

.align-r {
	/* 右对齐 */
	text-align: right;
}

/* Images */

.images img {
	width: 100%;
}

.color .images {
	/* 彩图上下间距 */
	margin: 0 0 1.5em;
}

.article .images {
	margin: 1.5em 0;
	border: 1px solid #000
}

/* p */

p {
	/* 1.5 倍行高 */
	line-height: 1.5em;
	margin: 0.5em 0;
	/* 设置缩进 */
	text-indent: 2em;
}
```

由于右对齐样式非常常用，因此在全局样式中添加右对齐类。



### 彩页格式

接着设置彩页的样式，向其中添加标题文字等信息。具体样式如下：

![](Screenshot_20240201_110118.png)



#### 添加样式

创建一个新的样式表，设置图片和彩页字体样式

```css
/* Color Image Pages */

.img-chara h2 {
	border-bottom: 1px solid #000;
}

.light h2, .light div {
	float: right;
}

.light h2 {
	margin: 2.7em 0 auto 0.5 em;
	padding-right: 1em;
}
```

创建彩页页面，链接样式表

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN"
  "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

<html xmlns="http://www.w3.org/1999/xhtml">
<head>
  <title></title>
  <link href="../Styles/color-images.css" type="text/css" rel="stylesheet"/>
  <link href="../Styles/global.css" type="text/css" rel="stylesheet"/>
</head>

<body>
<div class="wrap color">
<div class="images"><img alt="light" src="../Images/light.jpg"/></div>

<div class="img-chara light">

<h2>小光</h2>

<div class="align-r">

<p>在米可利杯上打败了小瑶，</p>
<p>“没问题！”（Daijōbu!）</p>
<p>真正重拾自信、目标坚定，</p>
<p>再次踏上旅途！</p>

</div>

</div>

</div>
</body>
</html>
```



#### 提取字体

通常的字体库都非常大，但我们实际上需要用到的特殊字体只有少数几个，因此下载 FontForge 软件，从字体库中提取部分字体。安装时没有中文选项，可以修改安装目录的 fontforge.bat 文件

```bat
@ECHO OFF
set "FF=%~dp0"
set "HOME=%FF%"
set "PYTHONHOME=%FF%"
set AUTOTRACE=potrace

::Set this to your language code to change the FontForge UI language
::See share/locale/ for a list of supported language codes
set LANGUAGE=zh_CN

::Only add to path once
if not defined FF_PATH_ADDED (
set "PATH=%FF%;%FF%\bin;%PATH:"=%"
set FF_PATH_ADDED=TRUE
)

for /F "tokens=* USEBACKQ" %%f IN (`dir /b "%FF%lib\python*"`) do (
set "PYTHONPATH=%FF%lib\%%f"
)

"%FF%\bin\fontforge.exe" -nosplash %*

:: bye


```

然后打开软件就能看到中文界面。



我们用 FontForge 打开一个字体文件，在菜单栏选择“编码-紧凑”就能看到字体样式

![](image-20240201153549595.png)

新建一个字体文件，选择“编码-重新编码”下的 Unicode 编码，就可以看到所有字符的编码。

![](image-20240201160222726.png)

我们只需要一部分字符，因此在需要的字体库中查找字符，复制粘贴到新字体文件中对应编码的位置即可。



精简得到的字体库如下，在“文件-生成字体”处点击生成字体，然后就可以导入到 Sigil 中使用

![](image-20240201161335510.png)



修改样式表，读取并使用新字体

```css
/* Font */
@font-face {
	font-family: "colorimg";
	src: url("../Fonts/colorimg.ttf");
}

/* Color Image Pages */

.img-chara h2 {
	font-family: colorimg;
	border-bottom: 1px solid #000;
}

.light h2, .light div {
	float: right;
	font-family: colorimg;
}

.light h2 {
	color: rgb(224, 169, 164);
	margin: 2.7em 0 auto 0.5 em;
	padding-right: 1em;
}
```



### 目录页

点击“工具-目录-从目录生成HTML”就可以通过之前生成的目录自动产生一个目录 xhtml 文件。

![](Screenshot_20240201_161953.jpg)



要支持我们自定义的样式，首先要去除自动生成的样式。通过正则表达式替换

```shell
<div.*>
<li>
```

将所有 div 标签替换为 li 标签。然后用 div-ul 标签包含所有 li 标签

```xml
<div class="wrap color toc">
<ul>
    <li><h2>目录</h2></li>
    ...
</ul>
</div>
```

给章节套上 span 标签

```shell
<a(.*)>(.*)</a>
<a\1><span class="toc-num">\2</span></a>
```

然后分离章节数与章节名

```shell
<span(.*)>([0-9]+\.)(.*)</span>
<span\1>\2</span><span class="toc-item">\3</span>
```

修改原本的数字章节

```shell
([0-9]+)\.</span>
第\1章</span>
```

将“信息”与“后记”两章的 span 标签改为 toc-num-2 类，特殊处理。



然后链接一个新的样式表

```css
/* Font */
@font-face {
	font-family: "toc";
	src: url("../Fonts/toc.ttf");
}

/* TOC */

.toc, .toc a {
	color: #175497;
}

.toc ul {
	/* table 自适应宽度 */
	display: table;
	margin: 0 auto;
}

.toc li {
	/* 每个 li 占一行 */
	display: table-row-group;
}

.toc li span {
	/* 每个 span 是一个单元格*/
	display: table-cell;
	line-height: 1.5em;
}

.toc li h2 {
	font-size: 1.5em;
	font-weight: normal;
	margin-bottom: 1em;
}

[class*="toc-item"] {
	font-family: toc;
}

.toc-num {
	max-width: 4em;
}

.toc-num-2 {
	width: 2em;
}

[class*="toc-num"] {
	padding-right: 1em;
}

span[class*="-2"] {
	/* 设置“信息与后记”的字体 */
	font-size: 0.85;
}
```



### 章节标题

我们将章节标题分块处理，分别显示章节数+分割符+章节名。具体样式如下：

![](Screenshot_20240201_171316.png)



首先要将 h1 标签中的内容分块

```shell
<h1>(第[0-9]+章) (.*)</h1>
<h1 class="title">\n<span class="title-num">\1</span>\n<span><i class="round"></i></span>\n<span>\2</span>\n</h1>
```

然后链接新的样式表

```css
/* Chapter Title */

.chapter {
	display: table;
	font-size: 2em;
	height: 4em;
	width: 100%;
}

.chapter span {
	display: table-cell;
	vertical-align: middle;
	text-align: center;
}

[class*="chapter-num"] {
	max-width: 5em;
}

.title {
	display: table;
	font-size: 1em;
	height: 4em;
	line-height: 4em;
	width: 100%;
}

.title span {
	display: inline-block;
	vertical-align: middle;
}

.title i {
	display: block;
}

[class*="title-num"] {
	padding-left: 1.0em;
	max-width: 5em;
}

.round {
	width: 0.6em;
	height: 0.6em;
	margin: 0 0.4em;
	background: #000;
	/* CSS3 特性加前缀确保兼容性 */
	-moz-border-radius: 50%;
	-webkit-border-radius: 50%;
	border-radius: 50%;
	/* 这样设置目的是在黑色背景下显示白色外环 */
	box-shadow: inset 0 0 0 0.1em #fff, 0 0 0 0.1em #fff;
	border: 0.1em solid #000;
}
```



## 标准样式

我们将下面样式统一保存到一个路径下，这样每次制作时可以直接引用。



### global

```css
@charset "UTF-8";

/* 全局设定 */

/* 重置 CSS */

* {
	margin: 0;
	padding: 0;
	border: none;
}

ul, ol {
	list-style: none;
}

a {
	text-decoration: none;
}

/* Warp */

.warp {
	/* 一行最多 40 字符*/
	max-width: 40em;
	margin: 0 auto;
}

.align-r {
	/* 右对齐 */
	text-align: right;
}

/* Images */

.images img {
	width: 100%;
}

.color .images {
	/* 彩图上下间距 */
	margin: 0 0 1.5em;
}

.article .images {
	margin: 1.5em 0;
	border: 1px solid #000
}

/* p */

p {
	/* 1.5 倍行高 */
	line-height: 1.5em;
	margin: 0.5em 0;
	/* 设置缩进 */
	text-indent: 2em;
}
```



### toc

这里 `toc.ttf` 需要自己提供，不过没有这个字体文件也没有影响。

```css
/* Font */
@font-face {
	font-family: "toc";
	src: url("../Fonts/toc.ttf");
}

/* TOC */

.toc, .toc a {
	color: #175497;
}

.toc ul {
	/* table 自适应宽度 */
	display: block;
	margin: 0 auto;
}

.toc li {
	/* 每个 li 占一行 */
	display: block;
}

.toc li span {
	/* 每个 span 是一个单元格*/
	display: inline-block;
}

.toc li h2 {
	font-size: 1.5em;
	font-weight: normal;
	margin-bottom: 1em;
}

[class*="toc-item"] {
	font-family: toc;
}

.toc-num {
	max-width: 15em;
}

.toc-num-2 {
	width: 2em;
}

[class*="toc-num"] {
	padding-right: 1em;
}

span[class*="-2"] {
	/* 设置“信息与后记”的字体 */
	font-size: 0.85;
}

/* Chapter */

.chap {
	width: 100%;
	line-height: 2.5em;
}

[class*="chap-item"] {
	font-family: toc;
	font-size: 150%;
}

[class*="chap-num"] {
	font-size: 150%;
	padding-right: 1em;
}
```



### title

```css
/* Chapter Title */

.chapter {
	display: table;
	font-size: 2em;
	height: 4em;
	width: 100%;
}

.chapter span {
	display: table-cell;
	vertical-align: middle;
	text-align: center;
}

[class*="chapter-num"] {
	max-width: 5em;
}


.title {
	display: table;
	font-size: 1em;
	height: 4em;
	width: 100%;
}

.title span {
	display: table-cell;
	vertical-align: middle;
}

.title i {
	display: block;
}

[class*="title-num"] {
	padding-left: 1.0em;
	max-width: 5em;
}

.round {
	width: 0.6em;
	height: 0.6em;
	margin: 0 0.4em;
	background: #000;
	/* CSS3 特性加前缀确保兼容性 */
	-moz-border-radius: 50%;
	-webkit-border-radius: 50%;
	border-radius: 50%;
	/* 这样设置目的是在黑色背景下显示白色外环 */
	box-shadow: inset 0 0 0 0.1em #fff, 0 0 0 0.1em #fff;
	border: 0.1em solid #000;
}
```



### colorimg

```css
/* Color Image Pages */

.img-chara h2 {
	border-bottom: 1px solid #000;
}

.light h2, .light div {
	float: right;
}

.light h2 {
	margin: 2.7em 0 auto 0.5 em;
	padding-right: 1em;
}
```



## 自动化流程

根据目前配置好的自动化模板，可以快速生成带有封面、目录、彩插的 EPUB 文件。

![](image-20240630000024259.png)



### 文本样式

首先创建空白文件，通过外部 xhtml 编辑器导入文本内容

![](image-20240630000215026.png)

然后运行 Connect 模板，得到规范化的文本格式。此时按 F6 可以按照部分、篇、章节进行拆分。拆分后，选中所有文件，执行 Global Settings 模板，进一步规范文本，主要是添加样式。到这一步已经完成基本样式框架，可以导入并链接 global 和 title 两个样式表。如果有彩插，则链接 colorimg 样式表。


### 章节样式

对于拆分出的 Chapter 页面，执行 Edit Chapter 模板，然后链接 global 和 title 两个样式表。对于带有 Intro 部分的页面，则执行 Edit Intro 模板，然后链接 global 和 title 两个样式表。



### 目录样式

通过工具创建目录，创建过程中调整好目录的嵌套关系，然后导出 HTML 文件。此时得到 TOC 目录文件，执行 Edit Toc 模板，然后链接 global 和 toc 两个样式表。

