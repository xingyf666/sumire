# HTML+CSS

## HTML

 HTML 是超文本标记语言，组成网页的基本结构。一个基本网页框架如下

```html
<!--使用 H5 标准-->
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <!--
        设置字符集
        防止乱码
    -->
    <meta charset="UTF-8">
    <title>标题</title>
</head>

<body>
    主体部分
</body>

</html>
```

网页的主体部分是 head 标签对应的头部（用户不可见）以及 body 标签对应的网页内容。在 body 标签中可以设置不同的块，其中存放各种元素。



打开任何一个网页

* F12 进入开发者模式，点击左上角的图标，移动到网页元素上方时相应部分会被选中
* Ctrl + u 打开源码页面
* Ctrl + f 调出搜索框
* F12 进入开发者模式
* Ctrl + r 重新加载信息

通过这种方式查看网页结构。



### 标签格式

网页中的任何一对标签形式为

```html
<标签名 属性1=”值” 属性2=”值” ...>...</标签名>
```

也可以使用单标签

```html
<标签名/>
```

标签之间的文本会被处理，其中的**所有空格和回车都会被替换为一个空格**。



### 注释格式

网页注释格式为

```html
<!--注释-->
```

默认情况下，不同的标签有其对应的效果。但是这些效果并不重要，因为通过 css 可以随时调整显示效果。**更重要的是标签本身的含义**。



考虑到对 IE 的兼容性，HTML 提供了条件注释

```html
<!--[if IE]>
  <p>You are using Internet Explorer</p>
<![endif]-->
```

当渲染框架为 IE 浏览器时，注释中的标签才有效。



### 基本信息

使用 meta 标签标注的是网页的基本信息。例如

```html
<!--提高 IE 浏览器兼容性-->
<meta http-equiv="X-UA-Compatible" content="IE=edge">
<!--移动端兼容-->
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<!--设置网站关键字，用于搜索-->
<meta name="keywords" content="购物，书籍，手机">
<!--网站描述-->
<meta name="description" content="网页的内容描述">
```



### 全局属性

每个标签都拥有的属性就称为全局属性。例如

| 全局属性 | 作用                                                         | 全局属性 | 作用                        |
| -------- | ------------------------------------------------------------ | -------- | --------------------------- |
| id       | 唯一标识符（head, html, meta, script, style, title 标签不能使用） | class    | 指定类名，可用 css 指定样式 |
| style    | 给标签增加 css 样式                                          | dir      | 指定排列方向，如 rtl, ltr   |
| title    | 设置文字提示，鼠标悬停显示，一般用于图片和超链接             | lang     | 指定标签语言                |



### 排版标签

使用 `<h1>` 到 `<h6>` 标签进行排版。例如

```html
<body>
    <h1>一级标签</h1>
    <h2>二级标签</h2>
    <h3>三级标签</h3>
    <h4>四级标签</h4>
    <h5>五级标签</h5>
    <h6>六级标签</h6>
</body>
```

其中 `<h1>` 标签最为重要，最好只有一个。这些标签通常用于显示标题。



使用 `<p>` 标签显示段落，显示文本

```html
<p>你好啊</p>
```



使用 `<div>` 标签作为区块容器标记，之间包含段落、表格图片

```html
<div>
	<p>你好啊</p>
	<p>早上好</p>
	<p>晚上好</p>
</div>
```

被 `<div>` 包含的内容可以通过 css 进行整体设置。



### 块级与行内元素

标签可以分为两种

* 块级元素：占据一行，换行
    * 块级元素可包含行内元素和某些块级元素
    * h1-h6 之间不能相互嵌套
    * 块级元素（包括 `<p>`）不能放在 `<p>` 标签内

* 行内元素：在一行，不换行
    * 行内元素只能包含其他行内元素，不能包含块级元素

排版标签都是块级元素。



### 划分标签

还有一些常用的标签，用于分割等作用。例如

| 标签      | 作用                  | 标签     | 作用   |
| ------- | ------------------- | ------ | ---- |
| `<br>`  | 换行符                 | `<hr>` | 分割内容 |
| `<pre>` | 包含的内容按原文显示（常用于嵌入代码） |        |      |



### 字符实体

HTML 有一些字符实体，能够表示字符。例如

<div>这里有四个&nbsp;&nbsp;&nbsp;&nbsp;空格</div>
<div>最重要的是&lt;h1&gt;标签。</div>
<div>转义显示&amp;nbsp;字符实体</div>
<div>&copy;版权所有</div>
<div>2&times;2=4</div>

还有更多的字符实体，可以直接查阅官方文档。



### 文本标签

常用的文本标签包括

| 标签                | 含义                       |
| ------------------- | -------------------------- |
| `<em></em>`         | 斜体文字（重要）           |
| `<strong></strong>` | 粗体文字（更重要）         |
| `<span></span>`     | 通用的文字容器（没有语义） |

使用 span 目的是为了方便地设置文字样式，而不用考虑语义。



### 图片标签

使用 img 标签显示图片

```html
![](香香.png)
```

其中 src 指定图片的位置，alt 指定图片描述，当图像无法显示时显示。如果只指定图片显示的宽或高，则会等比例缩放。



### 超链接标签

#### 跳转页面

用于页面跳转的标签

```html
<a href="https://zjuers.com/" target="_blank">内容</a>
```

其中 href 指定链接地址；target 指定目标窗口，可选

* `_self` 在当前页面打开（默认）
* `_blank` 在新窗口中打开
* `_top` 在顶部窗口打开
* `_parent` 在父窗口打开

还可设置 title 链接提示文字，name 指定链接名，用于设置锚点。



#### 跳转文件

可以设置打开文件的链接

```html
<a href="香香.png" target="_blank">内容</a>
```

点击链接将会在新窗口打开图片文件。



对于浏览器不能打开的文件，会自动触发下载

```html
<a href="test.zip" target="_blank">内容</a>
```

如果想下载可以打开的文件，添加 download 属性，设置下载文件名称即可

```html
<a href="香香.png" target="_blank" download="香香.png">内容</a>
```



#### 跳转锚点

锚点的作用是进行快速跳转。例如

```html
<!--指定跳转的锚点名称-->
<a href="#xx">香香</a>

<!--空锚点标记-->
<a name="xx"></a>
![](香香.png)
```



还可以指定 id 进行跳转

```html
<!--指定跳转的锚点名称-->
<a href="#xx">香香</a>

<!--指定目标 id-->
<img id="xx" src="香香.png" alt="香香" width="1000"/>
```

注意 id 最好不要是数字开头。



空的标记默认跳转回到顶部

```html
<a href="#">回到顶部</a>
<a href="">刷新页面</a>
```



也可以跳转到其它页面的锚点

```html
<a href="./test2.html#xx" target="_blank">香香</a>
```



#### 打开应用

例如可以拨打电话、打开邮箱

```html
<a href="tel:10010">拨打电话</a>
<a href="mailto:814083398@qq.com">打开邮箱</a>
<a href="sms:10010">发送短信</a>
```



### 列表

在 HTML 中有三种列表

* 有序列表

    ```html
    <ol>
        <li>第一步</li>
        <li>第二步</li>
        <li>第三步</li>
    </ol>
    ```

可使用 type 属性值设置 1 数字，a 小写字母，A 大写字母，i 小写罗马数字，I 大写罗马数字。

* 无序列表

    ```html
    <ul>
        <li>一部分</li>
        <li>另一部分</li>
        <li>还有一部分</li>
    </ul>
    ```

可使用 type 属性值设置 disc 圆点，square 正方形，circle 圆。

* 自定义列表

    ```html
    <dl>
    	<dt>标题1</dt>
    		<dd>描述1</dd>
    		<dd>描述2</dd>
    	<dt>标题2</dt>
            <dd>描述3</dd>
    </dl>
    ```



如果想要在列表中添加其它元素，应当把它们嵌套在列表项标签中。例如可以在列表中嵌套列表

```html
<ul>
	<li>一部分</li>
	<li>
		<span>另一部分</span>
		<ul>
			<li>细节1</li>
			<li>细节2</li>
		</ul>
	</li>
	<li>还有一部分</li>
</ul>
```



### 表格

基本表格的结构如下：

* thead：表格的头 （放标题之类内容）
* tbody：表格的主体 （放数据本体）
* tfoot：表格的脚 （放表格的脚注）

<table border="1">
    <!--标题，居中显示-->
    <caption>学生信息</caption>
    <!--头部-->
    <thead>
        <!--行标签-->
        <tr>
            <!--表头，内容居中，加粗显示-->
            <th>姓名</th>
            <th>性别</th>
            <th>年龄</th>
            <th>民族</th>
            <th>政治面貌</th>
        </tr>
    </thead>
    <!--主体-->
    <tbody>
        <!--行标签-->
        <tr>
            <!--单元格标签-->
            <td>张三</td>
            <td>男</td>
            <td>18</td>
            <td>汉族</td>
            <td>团员</td>
        </tr>
        <tr>
            <!--单元格标签-->
            <td>李四</td>
            <td>女</td>
            <td>16</td>
            <td>满族</td>
            <td>群众</td>
        </tr>
        <tr>
            <!--单元格标签-->
            <td>王五</td>
            <td>男</td>
            <td>19</td>
            <td>回族</td>
            <td>党员</td>
        </tr>
    </tbody>
    <!--脚注-->
    <tfoot>
        <tr>
            <!--单元格标签-->
            <td></td>
            <td></td>
            <td></td>
            <td></td>
            <td>共计：3人</td>
        </tr>
    </tfoot>
</table>



表格的常用属性包括

| 属性        | 含义                   | 属性        | 含义                       |
| ----------- | ---------------------- | ----------- | -------------------------- |
| border      | 外边框宽度             | cellpadding | 单元格内边距               |
| cellspacing | 单元格外边距           | bgcolor     | 背景色                     |
| width       | 表格宽度，各列平分宽度 | height      | 表格高度，不包括表头和脚注 |



除了 caption 标签以外，表格内标签的属性有

* align（水平对齐）可选 left, right, center, justify, char
* valign（垂直对齐）可选 top, bottom, middle, baseline
* bgcolor（背景色）格式为 rgb(x, x, x), #xxxxxx, colorname



跨行/跨列表格通过指定 colspan 和 rowspan 属性设置，控制一个单元格占几列/行。例如

<table border="1">
    <!--标题，居中显示-->
    <caption>课程表</caption>
    <!--头部-->
    <thead>
        <!--行标签-->
        <tr>
            <!--表头，内容居中，加粗显示-->
            <th align="center">项目</th>
            <th colspan="5" align="center">上课</th>
            <th colspan="2" align="center">活动与休息</th>
        </tr>
    </thead>
    <!--主体-->
    <tbody>
        <!--行标签-->
        <tr>
            <!--单元格标签-->
            <td>星期</td>
            <td>星期一</td>
            <td>星期二</td>
            <td>星期三</td>
            <td>星期四</td>
            <td>星期五</td>
            <td>星期六</td>
            <td>星期日</td>
        </tr>
        <tr>
            <!--单元格标签-->
            <td rowspan="4">上午</td>
            <td>语文</td>
            <td>数学</td>
            <td>英语</td>
            <td>英语</td>
            <td>物理</td>
            <td>数学竞赛</td>
            <td rowspan="4">休息</td>
        </tr>
        <tr>
            <!--单元格标签-->
            <td>数学</td>
            <td>语文</td>
            <td>化学</td>
            <td>物理</td>
            <td>英语</td>
            <td>篮球比赛</td>
        </tr>
        <tr>
            <!--单元格标签-->
            <td>化学</td>
            <td>语文</td>
            <td>体育</td>
            <td>历史</td>
            <td>地理</td>
            <td>每周一考</td>
        </tr>
        <tr>
            <!--单元格标签-->
            <td>体育</td>
            <td>化学</td>
            <td>语文</td>
            <td>数学</td>
            <td>英语</td>
            <td>社会实践</td>
        </tr>
        <tr>
            <!--单元格标签-->
            <td rowspan="2">下午</td>
            <td>语文</td>
            <td>英语</td>
            <td>数学</td>
            <td>物理</td>
            <td>数学</td>
            <td>英语角</td>
            <td rowspan="2">休息</td>
        </tr>
        <tr>
            <!--单元格标签-->
            <td>化学</td>
            <td>物理</td>
            <td>地理</td>
            <td>生物</td>
            <td>体育</td>
            <td>自由活动</td>
        </tr>
    </tbody>
</table>

注意到跨行的单元格会直接占用下一行的单元格，可以跳过下一行的单元格标签。



### 表单

表单是网页中的交互区域，用于收集用户信息。例如提供一个有输入框和按钮的表单

<form action="https://www.baidu.com/s" target="_blank">
    <input type="text" name="wd">
    <button>百度搜索</button>
</form>

其中 wd 是百度指定的关键字名称，跳转网页地址包含 `?wd=` 后面接输入输入内容。

> 跳转网页地址将包含表单中所有 name 变量及其获得的值。



form 标签的属性包括

* action 设置提交表单后跳转的链接
* method 可选 get, post（表单发送到页面的方式）
* name 表单名称
* target 目标窗口 _blank, _top, _bottom, _parent
* enctype 编码方式

 

#### input 标签

输入标签 input 是表单中最重要的部分，用来获得信息并提交。其属性包括

| 属性                  | 作用           | 属性      | 作用         |
| --------------------- | -------------- | --------- | ------------ |
| name                  | 变量名称       | maxlength | 最大输入长度 |
| size                  | 宽度           | value     | 变量值       |
| placeholder (H5 特性) | 输入字段的提示 | type      | 输入类型     |



type 属性可选

| type     | 类型     | type   | 类型     |
| -------- | -------- | ------ | -------- |
| password | 密码     | file   | 文件     |
| checkbox | 复选     | button | 按钮     |
| radio    | 单选     | submit | 提交按钮 |
| reset    | 重置按钮 | hidden | 隐藏     |
| image    | 图像     | text   | 文本     |

我们分别介绍这些属性的效果和用法。



#### 文本输入

一般的文本输入，使用 text 或 password 类型。例如

<form action="https://www.baidu.com/s" target="_blank">
    账户：<input type="text" name="account" value="张三" maxlength="10"><br>
    密码：<input type="password" name="pwd" value="123" maxlength="6"><br>
    <button>确认</button>
</form>

注意 name 是提交的变量名，value 是提交的变量值。



#### 选择按钮

有单选 radio 和 checkbox 复选，两种类型。例如

<form action="https://www.baidu.com/s" target="_blank">
    性别：
    <input type="radio" name="gender" value="男">男
    <input type="radio" name="gender" value="女" checked>女
    <br>
    爱好：
    <input type="checkbox" name="hobby" value="排球" checked>排球
    <input type="checkbox" name="hobby" value="篮球">篮球
    <input type="checkbox" name="hobby" value="足球">足球
    <br>
    <button>确认</button>
</form>

使用 checked 属性，表示默认选择该选项。

> 同一组选择按钮需要有相同的变量名，并且每个按钮要提供不同的值。



#### 隐藏域

使用 hidden 类型来隐藏一些输入框，可以用它进行标记，提交固定的内容

```html
<form action="https://www.baidu.com/s" target="_blank">
    <input type="hidden" name="tag" value="yes">
</form>
```



#### 提交与重置

使用 submit 和 reset 类型创建提交按钮和重置按钮

<form action="https://www.baidu.com/s" target="_blank">
    账户：<input type="text" name="account" value="张三" maxlength="10"><br>
	密码：<input type="password" name="pwd" value="123" maxlength="6"><br>
    <input type="submit" value="确认">
    <input type="reset" value="重置">
</form>

其中用 value 来设置按钮文本。也可以用 button 来写

<form action="https://www.baidu.com/s" target="_blank">
    账户：<input type="text" name="account" value="张三" maxlength="10"><br>
	密码：<input type="password" name="pwd" value="123" maxlength="6"><br>
    <button type="submit">确认</button>
    <button type="reset">重置</button>
</form>

> 所有按钮都不要添加 name 属性。



#### 普通按钮

使用 button 时，如果不指定 type，则默认为 submit，点击立即提交表单。一个普通的按钮类型是 button，例如

<button type="button">普通的按钮</button>



#### 文本域

使用 textarea 提供多行输入

<textarea name="other" cols="30" rows="3"></textarea>

可用属性有 name（名称），rows（行数），cols（列数），placeholder（提示）。



#### 下拉菜单

使用 select 提供下拉菜单。通过 option 标签指定数据列表

<form>
    籍贯：
    <select name="place">
        <option value="冀">河北</option>
        <option value="鲁">山东</option>
        <option value="晋" selected>山西</option>
        <option value="粤">广东</option>
    </select>
</form>  

在 option 中设置 value 属性，最后会提交该属性；selected 表示默认选中的选项。



#### 禁用组件

添加 disable 标记禁用组件。例如禁止修改文本输入

<form action="https://www.baidu.com/s" target="_blank">
    账户：<input disabled type="text" name="account" value="张三" maxlength="10"><br>
    密码：<input type="password" name="pwd" value="123" maxlength="6"><br>
    <button>确认</button>
</form>

也可以禁用其它组件。



#### label 标签

使用此标签将文本与组件相互关联，例如

<form action="https://www.baidu.com/s" target="_blank">
    <label for="zh">账户：</label>
    <input id="zh" type="text" name="account" value="张三" maxlength="10"><br>
    <label for="mm">密码：</label>
    <input id="mm" type="password" name="pwd" value="123" maxlength="6"><br>
    <button>确认</button>
</form>

通过 for 属性和 id 属性进行关联，点击文本就可以获得输入框的焦点。



也可以直接用 label 标签包含实现关联

<form action="https://www.baidu.com/s" target="_blank">
    爱好：
    <label>
        <input type="checkbox" name="hobby" value="排球" checked>排球
    </label>
    <label>
        <input type="checkbox" name="hobby" value="篮球">篮球
    </label>
    <label>
        <input type="checkbox" name="hobby" value="足球">足球
    </label>
    <br>
    <button>确认</button>
</form>

此时点击文本就可以选中选项。



#### 区域标签

使用 fieldset 和 legend 标签进行表单区域的划分操作。例如

<form action="https://www.baidu.com/s" target="_blank">
    <fieldset>
        <legend>主要信息</legend>
        账户：<input type="text" name="account" value="张三" maxlength="10"><br>
        密码：<input type="password" name="pwd" value="123" maxlength="6"><br>
        性别：
        <input type="radio" name="gender" value="男">男
        <input type="radio" name="gender" value="女" checked>女
    </fieldset>
    <br>
    <fieldset>
        <legend>附加信息</legend>
        爱好：
        <input type="checkbox" name="hobby" value="排球" checked>排球
        <input type="checkbox" name="hobby" value="篮球">篮球
        <input type="checkbox" name="hobby" value="足球">足球
    </fieldset>
    <br>
    <button>确认</button>
</form>

其中 fieldset 划分区域，legend 指定区域名称。



### HTML5

#### 语义化标签

新增语义化标签

| 标签    | 语义                                                 | 标签    | 语义                               |
| ------- | ---------------------------------------------------- | ------- | ---------------------------------- |
| header  | 整个页面，或部分区域的头部                           | footer  | 整个页面，或部分区域的底部         |
| nav     | 导航                                                 | article | 文章、帖子、杂志、新闻、博客、评论 |
| section | 页面中的某段文字，或文章中的某段文字（通常包含标题） | aside   | 侧边栏                             |

它们本质上就是**具有语义的 div 标签**。



#### 状态标签

增加 progress 标签，用于显示进度

<progress max="1000" value="600"></progress>

可以通过 css 调整样式。



#### 列表标签

增加 datalist 标签，可以实现搜索下拉提示功能。需要通过 id 绑定输入框

<form action="#">
    <input type="text" list="mydata">
    <button>搜索</button>
</form>
<datalist id="mydata">
    <option value="A">A</option>
    <option value="B">B</option>
    <option value="C">C</option>
</datalist>



增加 details 标签配合 summary 标签实现隐藏/显示内容

<details>
    <summary>点击显示更多内容</summary>
    <p>更多内容</p>
</details>



#### 文本标签

增加 ruby 标签配合 rt 标签实现上标注音

```html
<ruby>
    <span>魑魅魍魉</span>
    <rt>chi mei wang liang</rt>
</ruby>
```



增加 mark 标签作文字标记

<mark>一段重点标记</mark>



#### 表单控件

表单增加属性 novalidate，如果添加该属性，则不会验证格式。



增加表单控件的属性

| 属性        | 作用           | 属性     | 作用                                                 |
| ----------- | -------------- | -------- | ---------------------------------------------------- |
| placeholder | 输入字段的提示 | required | 要求填写/选择，否则不能提交                          |
| autofocus   | 自动聚焦       | pattern  | 提供正则表达式，约束提交内容的格式（多行输入不可用） |

例如增加所有这些属性的搜索框

<form action="https://www.baidu.com/s" target="_blank">
    <input type="text" placeholder="输入搜索内容" pattern="\w{6}" autofocus required>
	<button>搜索</button>
</form>

其中正则表达式要求输入 6 个数字。



input 标签新增 type 属性值

| 属性   | 作用                     | 属性           | 作用                       |
| ------ | ------------------------ | -------------- | -------------------------- |
| email  | 要求输入邮箱             | url            | 要求输入网址               |
| number | 数值格式，提供步长和范围 | search         | 可以快速清除，用于标记语义 |
| tel    | 要求输入电话             | range          | 范围选择器                 |
| color  | 颜色选择器               | date           | 日期选择器                 |
| month  | 月份选择器               | week           | 周选择器                   |
| time   | 时间选择器               | datetime-local | 日期+时间                  |

几种不同 type 属性的样式如下

<form action="https://www.baidu.com/s" target="_blank">
    <input type="number" style="width: 200px; height: 30px;" value="22" step="2" max="80" min="20"><br>
    <input type="search" style="width: 200px; height: 30px;" value="可以清除"><br>
    <input type="range" style="width: 200px; height: 30px;" step="2" max="80" min="20"><br>
    <input type="color" style="width: 200px; height: 30px;"><br>
    <input type="date" style="width: 200px; height: 30px;"><br>
</form>



#### 视频标签

增加 video 标签播放视频

<video src="" poster="" preload="auto" style="width: 300px;" controls></video>

其中 preload 表示预加载信息

* auto 下载整个视频文件
* metadata 仅获取视频元信息（例如长度）
* none 不预加载视频



可增加属性

| 属性     | 作用         | 属性  | 作用         |
| -------- | ------------ | ----- | ------------ |
| controls | 增加控制面板 | muted | 默认静音状态 |
| autoplay | 自动播放     | loop  | 循环播放     |
| poster   | 增加封面     |       |              |



#### 音频标签

增加 audio 标签播放音频

<audio src="" controls></audio>

其属性与视频标签差不多。



#### 全局/自定义属性

新增标签的全局属性

| 属性              | 作用                                        |
| --------------- | ----------------------------------------- |
| contenteditable | 允许将任何 HTML 元素设置为可编辑。可选 true、false、inherit |
| contextmenu     | 提供元素的上下文菜单，用户右键点击元素时显示                    |
| draggable       | 表示是否可以拖动。可选 true、false、auto（默认值）          |
| hidden          | 当一个元素应用了 hidden 属性，它将向所有用户隐藏              |
| spellcheck      | 拼写检查。可选 true、false、inherit                |
| data-*          | 自定义数据属性，以 data- 开头，可通过 js 获得              |



### Emmet 语法

Emmet 语法用于快速生成 HTML 标签结构。例如

```html
!	<!-- 可自动补全为 HTML 结构文件 -->
```

标签生成语法

```html
div*3			<!-- 3 个 div 标签 -->
div>p			<!-- <div><p></p></div> 嵌套结构 -->
div+p			<!-- <div></div><p></p> 相邻结构 -->
div.demo		<!-- <div class="demo"></div> -->
div#a			<!-- <div id="a"></div> -->
div[prop=val]	<!-- <div prop="val"></div> -->
div{text}		<!-- <div>text</text> -->
h$*3			<!-- <h1></h1><h2></h2><h3></h3> -->
```

利用上述基础语法，可以复合得到复杂的构建方式

```html
h$[prop=value$].name#age${text$}*3
```

生成如下标签结构

```html
<h1 prop="value1" class="name" id="age1">text1</h1>
<h2 prop="value2" class="name" id="age2">text2</h2>
<h3 prop="value3" class="name" id="age3">text3</h3>
```

由于 `$` 代表自增符号，因此标签名、属性名和文本内容都得到递增。



## CSS

CSS 即层叠样式表 （Cascading Style Sheets），用于为 HTML 元素添加样式。它具有三大特性

* 层叠性：如果出现样式冲突，则会根据优先级进行层叠覆盖；
* 继承性：元素自动继承祖先元素的部分样式。优先继承离得近的父元素；
* 优先级：需要计算权重。注意**并集选择器的每个部分需要分开计算**；



### 添加样式

有三种添加样式的方法

* 行内样式：直接通过标签的 style 属性设置，只用于简单的样式设置，不推荐大规模使用

    <h1 style="color: red;font-size: 20px;">行内样式</h1>

* 内部样式：将 css 代码提取出来，放到 style 标签中

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
	<meta charset="UTF-8">
	<title>标题</title>
	<style>
		h1 {
			color: green;
			font-size: 10px;
		}

		h2 {
			color: red;
			font-size: 20px;
		}

		h3 {
			color: blue;
			font-size: 30px;
		}
	</style>
</head>

<body>
	<h1>h1标题</h1>
	<h2>h2标题</h1>
		<h3>h3标题</h1>
</body>

</html>
```

虽然 style 可以放到任何位置，但是**建议放到 head 标签中**。

* 外部样式：将 css 代码单独提取出去

```css
h1 {
	color: green;
	font-size: 10px;
}

h2 {
	color: red;
	font-size: 20px;
}

h3 {
	color: blue;
	font-size: 30px;
}
```

然后**通过 link 标签引入 css 文件**

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>标题</title>
    <link rel="stylesheet" href="position.css">
</head>

<body>
    <h1>h1标题</h1>
    <h2>h2标题</h1>
    <h3>h3标题</h1>
</body>

</html>
```

这种方法样式可以复用，结构情形，可触发浏览器的缓存机制，提高访问速度，**最推荐使用**。



### 样式表优先级

优先级规则：行内样式 > 内部样式 = 外部样式。

* 内部样式和外部样式，后面的会覆盖前面的
* 同一个样式表内，也是后面的样式覆盖前面的




### 语法规范

CSS 通过选择器和声明块设置样式

```css
h1 {
    color: green;
    font-size: 10px;
}
```

其中 h1 就是选择器，后面 `{}` 包含的部分称为声明块。



CSS 中的注释必须通过 `/**/` 包含

```css
/* 这部分是注释 */
h1 {
    color: green;
    font-size: 10px;
}
```



### 基本选择器

#### 通配选择器

通配选择器将选中整个 HTML 中的元素

```css
* {
    color: red;
}
```

它常用于**清除样式**而非设置样式。



#### 元素选择器

指定标签元素，设置所有这种标签的样式

```css
h1 {
    color: green;
    font-size: 10px;
}
```

也可以指定 body 标签的属性。



#### 类选择器

使用 class 属性设置分类，多个分类用空格分隔。然后可以用 `.classname` 格式设置类的样式

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>诗词鉴赏</title>
    <style>
        .A {
            color: green;
        }

        .B {
            color: red;
        }

        .C {
            font-size: 20px;
        }

        .D {
            font-size: 40px;
        }
    </style>
</head>

<body>
    <p class="A C">千山鸟飞绝</p>
    <p class="A D">万径人踪灭</p>
    <p class="B C">孤舟蓑笠翁</p>
    <p class="B D">独钓寒江雪</p>
</body>

</html>
```

如果需要多个单词描述类名，建议使用 `-` 连接。



#### id 选择器

通过添加 id 属性设置标签样式。由于 id 值是唯一值，因此该样式的影响范围是最小的。使用 `#id` 格式配置样式

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>诗词鉴赏</title>
    <style>
        #first {
            color: green;
        }

        #second {
            color: red;
        }

        #third {
            font-size: 20px;
        }

        #forth {
            font-size: 40px;
        }
    </style>
</head>

<body>
    <p id="first">千山鸟飞绝</p>
    <p id="second">万径人踪灭</p>
    <p id="third">孤舟蓑笠翁</p>
    <p id="forth">独钓寒江雪</p>
</body>

</html>
```



### 复合选择器

#### 交集选择器

将标签和类名连接在一起，表示同时满足标签和类名，指定的选择器就是交集选择器。例如

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>诗词鉴赏</title>
    <style>
        p {
            color: green;
        }

        p.poetry.first {
            font-size: 10px;
        }

        p.poetry.second {
            font-size: 20px;
        }

        .title {
            font-size: 30px;
        }
    </style>
</head>

<body>
    <p class="title">江雪</p>
    <p class="poetry first">千山鸟飞绝</p>
    <p class="poetry first">万径人踪灭</p>
    <p class="poetry second">孤舟蓑笠翁</p>
    <p class="poetry second">独钓寒江雪</p>
</body>

</html>
```

可以连接多个类名进行选择。**如果具有两个类名的标签对应的类选择器有冲突的样式，则按照 css 代码中的定义顺序，使用第一个样式**。



#### 并集选择器

多个 id 和类名可以通过逗号连接，具有其中任何一个特征的标签都会获得该样式。例如

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>诗词鉴赏</title>
    <style>
        p {
            color: green;
        }

        .title {
            font-size: 30px;
        }

        #first,
        .poetry,
        .second {
            font-size: 20px;
        }
    </style>
</head>

<body>
    <p class="title">江雪</p>
    <p id="first">千山鸟飞绝</p>
    <p class="second">万径人踪灭</p>
    <p class="poetry">孤舟蓑笠翁</p>
    <p class="poetry">独钓寒江雪</p>
</body>

</html>
```



#### 后代选择器

根据标签的嵌套关系，可以确定标签的祖先和子代。后代选择器可以指定**嵌套关系**中的标签样式，例如

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>诗词鉴赏</title>
    <style>
        ul li {
            color: red;
        }

        ol li {
            color: green;
        }

        ol a {
            color: orange;
        }

        ul a {
            color: yellow;
        }

        /* 类名 subject 的标签下的 li 标签 */
        .subject li {
            color: blue;
        }

        /* 类名 subject 的标签下的 li 标签且类名为 Yu */
        .subject li.Yu {
            color: chocolate;
        }
    </style>
</head>

<body>
    <ul>
        <li>北京</li>
        <li>上海</li>
        <li>
            <a href="#">天津</a>
        </li>
    </ul>
    <ol>
        <li>吃饭</li>
        <li>睡觉</li>
        <li>
            <a href="#">学习</a>
        </li>
    </ol>
    <ol class="subject">
        <li class="Yu">语文</li>
        <li>数学</li>
        <li>英语</li>
    </ol>
</body>

</html>
```

多种选择器也可以相互嵌套使用，根据指定特征的顺序，选中满足条件的标签。



#### 子代选择器

后代选择器会选中所有后代，而子代选择器只会选中满足的子标签。使用 > 连接表示子代，例如

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>诗词鉴赏</title>
    <style>
        div>a {
            color: red;
        }

        div>p>a {
            color: skyblue;
        }

        .foot>a {
            color: green;
        }
    </style>
</head>

<body>
    <div>
        <!--这个标签被 div>a 选择-->
        <a href="#">张三</a>
        <a href="#">李四</a>
        <div>
            <!--这个标签也被 div>a 选择-->
            <a href="#">王五</a>
            <p class="foot">
                <a href="#">赵六</a>
            </p>
            <p>
                <a href="#">孙七</a>
            </p>
        </div>
    </div>
</body>

</html>
```

注意**所有具有对应子代关系的标签都会被选择**。



#### 兄弟选择器

用于选择指定标签之后的元素，例如

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>诗词鉴赏</title>
    <style>
        /* 选择 div 相邻的 a 标签 */
        div+a {
            color: red;
        }

        /* 选择 div 之后的所有 p 标签 */
        div~p {
            color: blue;
        }

        /* 选择 li 之后的所有 li 标签 */
        li~li {
            color: orange;
        }
    </style>
</head>

<body>
    <div>课程</div>
    <a href="#">内容</a>
    <div>时间</div>
    <p>周一</p>
    <p>周三</p>
    <p>周五</p>
    <ul>
        <li>数学</li>
        <li>语文</li>
        <li>英语</li>
    </ul>
</body>

</html>
```

注意由于选择的是之后的标签，因此 `li~li` 没有选中第一个 `li` 标签。



#### 属性选择器

顾名思义，属性选择器选择满足属性条件的标签。例如

```css
/* 选择具有 title 属性的标签 */
[title] {
    color: red;
}

/* 选择 title 属性为 Math 的标签 */
[title="Math"] {
    color: yellow;
}

/* 选择 title 属性以 a 开头的标签 */
[title^="a"] {
    color: blue;
}

/* 选择 title 属性以 u 结尾的标签 */
[title$="u"] {
    color: green;
}

/* 选择 title 属性包含 c 的标签 */
[title*="c"] {
    color: brown;
}
```



#### 伪类选择器

伪类选择器通过状态进行选择，使用 `:` 连接标签。



##### 动态伪类

* link 未访问状态（只有 a 标签使用）
* visited 已访问状态（只有 a 标签使用）
* hover 鼠标悬停状态
* active 激活状态

这些**状态样式要按照顺序来写**，否则可能导致它们的样式互相覆盖。例如

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>链接</title>
    <style>
        /* 未访问的 a 元素 */
        a:link {
            color: blue;
        }

        /* 访问过的 a 元素 */
        a:visited {
            color: orange;
        }

        /* 鼠标悬停的 a 元素 */
        a:hover {
            color: red;
        }

        /* 鼠标点击的 a 元素 */
        a:active {
            color: green;
        }
    </style>
</head>

<body>
    <a href="https://zjuers.com/">跳转</a>
</body>

</html>
```



input, select 等标签具有 focus 伪类，设置输入框被选中时的样式。例如

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>选择</title>
    <style>
        input:focus, select:focus {
            color: orange;
            background-color: red;
        }
    </style>
</head>

<body>
    <input type="text">
    <form>
        <select name="place">
            <option value="冀">河北</option>
            <option value="鲁">山东</option>
            <option value="晋" selected>山西</option>
            <option value="粤">广东</option>
        </select>
    </form>  
</body>

</html>
```



##### 结构伪类

通过与后代、子代选择器配合进行选择。例如

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>链接</title>
    <style>
        /* 选中 ul 的子标签 li，并要求它是自己父标签的第一个子标签 */
        ul>li:first-child {
            color: red;
        }

        /* 选中 div 的后代标签 li，并要求它是自己父标签的第一个子标签 */
        div li:first-child {
            color: green;
        }

        /* 选中 p 标签，并要求它是自己父标签的第一个子标签 */
        p:first-child {
            color: blue;
        }
    </style>
</head>

<body>
    <ul>
        <!-- 选中 -->
        <li>第一项</li>
        <li>第二项</li>
        <li>第三项</li>
    </ul>

    <div>
        <ol>
            <!-- 选中 -->
            <li>第一项</li>
            <li>第二项</li>
            <li>第三项</li>
        </ol>
    </div>

    <div>
        <!-- 选中 -->
        <p>你好</p>
        <p>世界</p>
    </div>
</body>

</html>
```



常用的结构伪类有

* first-child 自己**父标签**的第一个子标签
* last-child 自己**父标签**的最后一个标签
* nth-child(n) 自己**父标签**的第 n 个标签，从 1 开始计数：
    * 填 n 则选中所有项
    * 可以使用 2n 或 2n+1 选择偶数或奇数项
    * 或者用 even 或 odd 选择偶数或奇数项
    * 使用 an+b 的形式选中对应项
* first-of-type **同类型**的第一个标签
* last-of-type **同类型**的最后一个标签
* nth-of-type(n) **同类型**的第 n 个标签



还有一些不常见的结构伪类

* nth-last-child(n) 自己**父标签**的倒数第 n 个标签
* nth-last-of-type(n) **同类型**的倒数第 n 个标签
* only-child 选中没有兄弟的元素
* only-of-type 选中没有**同类型**兄弟的元素
* root 根元素（相当于选中 html 标签）
* empty 内容为空的元素



##### 否定伪类

否定伪类通过否定某种属性进行选择。例如

```css
/* 选中没有 title 类属性的 p 标签 */
p:not(.title) {
    color: red;
}

/* 选中没有 title, toc 类属性的 p 标签 */
p:not(.title, .toc) {
    color: red;
}

/* 选中没有 title, toc 类属性的 p 标签 */
p:not(.title) :not(.toc) {
    color: red;
}

/* 选中 title 属性不为 hello 的 div 标签 */
div:not([title="hello"]) {
    color: yellow;
}

/* 选中没有 id 为 test 的 a 标签 */
a:not(#test) {
    color: red;
}
```

可以利用此伪类提高样式优先级，例如

```css
#foo {
	color: yellow;
}

#foo:not[#bar] {
	color: yellow;
}
```

虽然上述两个样式都是针对 ID 为 `foo` 的标签，但是后者由于存在两个 ID 选择器，因此优先级是两倍。



##### 选择伪类

选择伪类 (is & where) 用于简化选择代码。例如

```css
/* 选择 header, main, footer 下的 div */
.header div:hover,
.main div:hover,
.footer div:hover {
  color: green;
  cursor: pointer;
}
```

可以简化为

```css
:is(.header, .main, .footer) div:hover {
  color: green;
  cursor: pointer;
}
```



选择伪类可以层叠使用。例如

```css
div span:hover,
div i:hover,
p span:hover,
p i:hover,
div span:focus,
div i:focus,
p span:focus,
p i:focus {
  color: green;
}


/* 层叠简化代码 */
:is(div, p) :is(span, i) :is(:hover, :focus) {
  color: green;
}
```

伪类 where 和 is 的使用方式相同，不同之处是只要使用了 where 伪类，则对应样式的优先级为 0 。



##### 包含伪类

包含伪类解决了不能选择父元素和前面兄弟元素的问题。例如

```css
/* 如果 p 标签具有 demo 类的子标签或者 i 标签，则选中该 p 标签 */
p:has(.demo, i) {
  color: red;
}
```

将其与 is 相结合可以得到奇妙的选择结果。例如

```css
:is(h1, h2, h3) :has(+ :is(h2, h3, h4)) {
  color: red;
}
```

首先 `is(h1, h2, h3)` 筛选出 `h1,h2,h3` 标签，如果兄弟标签有 `h2,h3,h4` 则选中。例如

```html
<section>
  <article>
    <h1>H1 标签</h1>
    <!-- h2 标签，并且兄弟标签有 h1 -->
    <h2>H2 标签</h2>
    <p>段落</p>
  </article>
  <article>
    <h1>H1 标签</h1>
    <!-- h2 标签，并且兄弟标签有 h1,h3 -->
    <h2>H2 标签</h2>
    <h3>H3 标签</h3>
    <p>段落</p>
  </article>
</section>
```

上述选择器将会选中第一个 `h1` 标签和第二个 `h2` 标签。



##### UI 伪类

对于一些组件，可以通过 UI 伪类选择具有相应状态的组件。例如

```css
/* 选中勾选的单选或复选框 */
input:checked {
    color: red;
}

/* 选中禁用的 input 元素 */
input:disabled {
    color: gray;
}

/* 选中可用的 input 元素 */
input:enabled {
    color: blue;
}
```



##### 目标伪类

target 伪类，选中锚点指向的元素。例如

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>链接</title>
    <style>
        div {
            width: 100px;
            height: 100px;
            background-color: white;
        }

        div:target {
            background-color: green;
        }
    </style>
</head>

<body>
    <a href="#one">目标1</a>
    <a href="#two">目标2</a>

    <div id="one"></div>
    <div id="two"></div>
</body>

</html>
```



#### 伪元素选择器

伪元素标签选中元素中的**特殊位置**，使用 `::` 连接标签。例如

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>文本</title>
    <style>
        /* 选中 div 标签中的第一个单词 */
        div::first-letter {
            color: red;
            font-size: 40px;
        }
    </style>
</head>

<body>
    <div>There are many letters.</div>
</body>

</html>
```

类似的伪元素有

* first-letter 第一个单词
* first-line 第一行文字
* selection 被鼠标选择的文字



还可以选中 input 标签中的文字，以及标签前后的位置

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>文本</title>
    <style>
        /* 选中 input 中的提示文字 */
        input::placeholder {
            color: skyblue;
        }

        /* 选中 p 标签文字之前的位置，通过 content 指定内容 */
        p::before {
            content: "￥";
        }

        /* 选中 p 标签文字之后的位置，通过 content 指定内容 */
        p::after {
            content: ".00";
        }
    </style>
</head>

<body>
    <input type="text" placeholder="请输入文字：">
    <p>100</p>
    <p>99</p>
</body>

</html>
```



### 选择器优先级

#### 简单选择器

同类型的选择器按照顺序，排在后面的选择器具有最高优先级；而不同类型的基本选择器的优先级顺序为

1. 行内选择器（行内样式）
2. ID 选择器
3. 类选择器
4. 元素选择器
5. 通配选择器
6. 继承样式



#### 复杂选择器

对于多种选择器复合的情况，则需要计算权重。权重格式为 (a, b, c)，其中

* a 表示 ID 选择器的个数
* b 表示类、伪类、属性选择器的个数
* c 表示元素、伪元素选择器的个数
* 通配选择器权重为 (0, 0, 0)

计算出权重后（vscode 提供鼠标悬停自动计算权重），按照字典序比较大小。行内样式优先级高于任意复合选择器。



有些选择器有其特性：

* 并集选择器的每个部分需要分开计算
* is，not 选择器的权重是其括号选择的元素中的最高优先级
* where 选择器对应的样式优先级始终为 0

通过 !important 可以强制提高某个属性的重要性

```css
.title {
    color: red !important;
    font-size: 40px;
}
```

虽然该选择器的权重只有 (0, 1, 0)，但是其 color 属性的优先级最高。



### 颜色表示

可以使用变量表示颜色，但是可选颜色很少。另外可以通过

* RGB/RGBA 格式 `rgb(255,255,255)` 或 `rgba(255,255,255,1)` 表示（vscode 提供鼠标悬停调整颜色）
* HEX/HEXA 格式 `#ff00ff` 或 `#ff00ffff` 表示



### 常用属性

#### 字体属性

| 属性       | 作用                          | 属性        | 作用                                                     |
| ---------- | ----------------------------- | ----------- | -------------------------------------------------------- |
| font-size  | 字体大小                      | font-family | 字体族。指定多种字体，按照顺序查找，直到找到可用的字体   |
| font-style | 字体风格。可选 normal, italic | font-weight | 字重。可选 normal, lighter, bold, bolder；也可以指定数值 |



可以指定字体的复合属性，例如

```css
.title {
    font: bold italic 100px "微软雅黑";
}
```

注意字体大小和字体族需要按照顺序放在最后两位。



#### 文本属性

| 属性           | 作用                                                         |
| -------------- | ------------------------------------------------------------ |
| color          | 文本颜色                                                     |
| letter-spacing | 字母间距（两两文字）                                         |
| word-spacing   | 单词间距（通过空格区分单词）                                 |
| text-indent    | 文本缩进宽度                                                 |
| text-align     | 文本对齐方式。可选 left, center, right                       |
| line-height    | 行高。可选 normal（自动调整）或像素值，数字（相对 font-size 的倍数） |



##### line-height

使用 line-height 时，建议设置数字倍数，这样在继承时会比较方便。例如

```html
<p class="title">
    一行文字
</p>
<p>
    另一行文字
</p>
```

对应的样式为

```css
.title {
    font-size: 40px;
    line-height: 1.5;
}

p {
    font-size: 30px;
}
```

此时对于第一个 p 标签，它继承了 title 样式中的行高倍数，根据其中的 font-size 的倍数设置具体的行高。



##### vertical-align

vertical-align 控制行内元素的垂直对齐

* baseline（默认）：使元素基线与父元素的基线对齐
* top：使元素顶部与其所在行的顶部对齐
* bottom：使元素底部与其所在行的底部对齐
* middle：使元素中部与父元素基线加上父元素字母 x 的一半对齐

注意它不能控制块元素的位置。



##### text-decoration

指定文本修饰符

```css
.title {
    text-decoration: overline dotted red;
}
```

分别指定线的位置，样式和颜色

* 线的位置：overline(上划线), underline(下划线), line-through(删除线), none(无修饰)；
* 样式：dotted(虚线), wavy(波浪线)；



#### 列表属性

| 属性             | 作用                                       | 属性                | 作用                                       |
| ---------------- | ------------------------------------------ | ------------------- | ------------------------------------------ |
| list-style-type  | 设置列表符号。例如 square, decimal         | list-style-position | 列表符号的位置。可选 inside, outside       |
| list-style-image | 设置列表图片。取值格式为 `url("test.gif")` | list-style          | 复合属性。将前三个属性写在一起，用空格分隔 |



#### 边框属性

| 属性         | 作用     | 属性         | 作用                                       |
| ------------ | -------- | ------------ | ------------------------------------------ |
| border-width | 边框宽度 | border-color | 边框颜色                                   |
| border-style | 边框样式 | border       | 复合属性。将前三个属性写在一起，用空格分隔 |



#### 表格属性

| 属性            | 作用                                              | 属性           | 作用                    |
| --------------- | ------------------------------------------------- | -------------- | ----------------------- |
| table-layout    | 表格列宽度。可选 auto, fixed                      | border-spacing | 单元格间距              |
| border-collapse | 合并相邻边框。可选 separate(分离), collapse(合并) | empty-cells    | 隐藏空单元格。可选 hide |
| caption-side    | 标题位置。可选 top(表格上方), bottom(表格下方)    |                |                         |



#### 背景属性

| 属性              | 作用                                            | 属性                | 作用                                                         |
| ----------------- | ----------------------------------------------- | ------------------- | ------------------------------------------------------------ |
| background-color  | 背景色                                          | background-image    | 背景图片。取值格式为 `url("test.gif")`                       |
| background-repeat | 背景图片重复方式。repeat-x, repeat-y, no-repeat | background-position | 背景图片位置。用两个值表示，例如 left center 表示水平居左+垂直居中；也可取像素值 10px 20px |
| background        | 复合属性。同时设置前面的属性值                  |                     |                                                              |



#### 鼠标属性

| 属性   | 作用                                                         |
| ------ | ------------------------------------------------------------ |
| cursor | 修改鼠标样式。常用 pointer, move, text, crosshair, wait, help |

也可以用 `url("image.png"), pointer` 指定对应 pointer 的样式。



### 盒子模型

#### 长度单位

常用的长度单位除了 **px(像素)** 和**百分比**以外，还有 **em 表示相对于当前元素或者父元素的 font-size 属性的倍数**。例如

```css
p {
    font-size: 10px;
    height: 10em;	/* 相当于 100px */
}
```

如果当前元素没有 font-size，或者是以下情况

```css
p {
    font-size: 1em;
}
```

则根据父元素的 font-size 属性设置。而 **rem 则是相对于根元素的 font-size 属性的倍数**。



#### 显示模式

盒子模式中的元素显式模式主要有三种：

| 显示模式                 | 元素排列  | 默认宽度  | 默认高度 | CSS    |
| :------------------- | :---- | :---- | :--- | :----- |
| 块元素 (block)          | 独占一行  | 父元素宽度 | 内容高度 | 可以设置宽高 |
| 行内元素 (inline)        | 不独占一行 | 内容宽度  | 内容高度 | 不能设置宽高 |
| 行内块元素 (inline-block) | 不独占一行 | 内容宽度  | 内容高度 | 可以设置宽高 |

可以通过 display 属性修改元素的显示模式

```css
div {
    /* 将 div 修改为行内显示 */
	display: inline;
}
```

如果设为 none，则该元素将会消失。



#### 组成部分

css 会将所有 HTML 元素看作一个盒子，其中

* margin 外边距
    * 子元素的 margin 根据父元素的 content 设置；
    * **行内元素的上下外边距无效**；
    * 块元素左右 margin 设为 auto 可以实现在父元素内水平居中；
    * margin 可以为负值；
* border 边框：border-width, border-color, border-style 属性；
* padding 内边距
    * **行内元素的上下内边距**可能会有重叠问题；
* content 内容：width, height, min-width, max-width, min-height, max-height 属性；

盒子的大小是 content + 左右 padding + 左右 border 的大小。**与盒子相关的属性都不能继承**。

![](Screenshot_20230617_140410.jpg)



例如我们绘制三个嵌套的 div 标签

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>文本</title>
    <style>
        #d1 {
            width: 400px;
            height: 400px;
            padding: 50px;
            background-color: gray;
        }

        #d2 {
            width: 300px;
            height: 300px;
            padding: 50px;
            background-color: orange;
        }
        
        #d3 {
            width: 300px;
            height: 300px;
            background-color: green;
            font-size: 50px;
            color: white;
        }
    </style>
</head>

<body>
    <div id="d1">
        <div id="d2">
            <div id="d3">你好啊</div>
        </div>
    </div>
</body>

</html>
```

显示效果如下

![](image-20230618000020455.png)



#### margin 塌陷

在父元素中，第一个元素的 margin-top 和最后一个元素的 margin-bottom 会被父元素获得。例如

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>文本</title>
    <style>
        .outer {
            width: 400px;
            background-color: gray;
        }

        .inner1 {
            width: 100px;
            height: 100px;
            background-color: orange;
            margin-top: 50px;
        }

        .inner2 {
            width: 100px;
            height: 100px;
            background-color: green;
            margin-bottom: 50px;
        }
    </style>
</head>

<body>
    <div class="outer">
        <div class="inner1">inner1</div>
        <div class="inner2">inner2</div>
    </div>
    <div>测试文字</div>
</body>

</html>
```

产生的效果如下

![](image-20230617144919804.png)



解决方案是设置溢出内容的样式

```css
.outer {
    width: 400px;
    background-color: gray;
    overflow: hidden;
}
```

也可以添加 border 或 padding 为零的属性值。最终得到的效果为

![](image-20230617145357622.png)



#### margin 合并

上下元素之间设置的 margin-top 和 margin-bottom 会合并。例如

```html
<p>第一部分</p>
<p>第二部分</p>
```

如果设置如下样式

```css
p {
    margin-top: 10px;
    margin-bottom: 10px;
}
```

则两个 p 标签的上下间距为 10px 而不是 20px 。因此**建议只对一个元素设置外边距**。



#### 内容溢出

使用 overflow 属性设置溢出内容样式。可选

* visible 显示（默认）
* hidden 隐藏
* scroll 滚动条
* auto 自动判断



#### 内容隐藏

有两种隐藏元素的方式

* display: none; 隐藏元素，不占位
* visibility: hidden; 隐藏元素，占位

 

#### 元素居中

1. 行内、行内块元素可以被父元素当做文本处理，因此文本属性对它们有效；
2. 让子元素在父元素中**水平居中**
    1. 若子元素是块元素，给父元素设置 `margin: 0 auto;`
    2. 若子元素是行内、行内块元素，给父元素设置 `text-align: center;`
3. 让子元素在父元素中**垂直居中**
    1. 若子元素是块元素，给子元素设置 `margin-top`
    2. 若子元素是行内、行内块元素，让父元素设置 `height=line-height`，每个子元素添加 `vertical-align: middle;`
    若想绝对垂直居中，父元素设置 `font-size: 0;`



实现如下显示效果

![](image-20230617185105875.png)

样式设置为

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>文本</title>
    <style>
        .outer {
            width: 400px;
            height: 400px;
            background-color: gray;
            /* 取消 margin 塌陷 */
            overflow: hidden;
        }

        .inner {
            width: 200px;
            height: 100px;
            background-color: orange;
            /* 容器水平居中 */
            margin: 0 auto;
            margin-top: 150px;
            text-align: center;
            /* 文本垂直居中 */
            line-height: 100px;
        }
    </style>
</head>

<body>
    <div class="outer">
        <div class="inner">inner</div>
    </div>
</body>

</html>
```



实现如下效果

![](image-20230617185435878.png)

样式设置为

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>文本</title>
    <style>
        .outer {
            width: 400px;
            height: 400px;
            background-color: gray;
            text-align: center;
            line-height: 400px;
        }

        .inner {
            background-color: orange;
            font-size: 20px;
        }
    </style>
</head>

<body>
    <div class="outer">
        <span class="inner">inner</span>
    </div>
</body>

</html>
```



实现如下效果

![](image-20230617190154123.png)

这里由于 vertical-align 属性的基准与字母 x 大小有关，为了**实现绝对的垂直居中，可以将字体大小设为 0**，消除影响

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>文本</title>
    <style>
        .outer {
            width: 400px;
            height: 400px;
            background-color: gray;
            text-align: center;
            line-height: 400px;
            /* 使水平准线居中 */
            font-size: 0;
        }

        span {
            /* 覆盖字体大小，并使文字垂直居中 */
            font-size: 40px;
            vertical-align: middle;
            background-color: orange;
        }

        img {
            width: 100px;
            vertical-align: middle;
        }
    </style>
</head>

<body>
    <div class="outer">
        <span>inner</span>
        ![](香香-AI.jpg)
    </div>
</body>

</html>
```



#### 元素间隙

由于行内、行内块元素之间的空行会被认为是一个空格。例如

```html
<div>
    <span>人之初</span>
	<span>性本善</span>
</div>
```

这里 span 是行内元素，换行后的两个标签之间有空格。



比较好的解决方案是：将父元素中的字体大小设为 0，然后单独设置文本标签的字体大小

```css
div {
    font-size: 0;
}

span {
    font-size: 20px;
}
```



#### 幽灵空白

当我们按照如下样式

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>文本</title>
    <style>
        .outer {
            width: 300px;
            background-color: gray;
        }

        img {
            height: 200px;
        }
    </style>
</head>

<body>
    <div class="outer">
        ![](香香-AI.jpg)
        <span>x</span>
    </div>
</body>

</html>
```

注意没有设置 div 的高度，由于基线对齐，导致图片下方有一片空白。如下所示

![](image-20230617192003008.png)

最好的解决方案是添加 vertical-align 属性

```css
img {
    height: 200px;
    vertical-align: middle;
}
```

当然，也可以将父元素的 font-size 设为 0 来解决。



### 浮动

使用 float 属性设置的样式就是浮动。例如实现文字环绕图片

```htm
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>文本</title>
    <style>
        .outer {
            width: 500px;
            height: 300px;
            background-color: gray;
            font-size: 15px;
        }

        img {
            height: 150px;
            float: left;
            margin: 1em;
        }
    </style>
</head>

<body>
    <div class="outer">
        ![](香香-AI.jpg)
        <span>层叠样式表（Cascading Style
            Sheet，CSS）有助于实现负责任的Web设计。CSS对开发者构建Web站点的影响很大，并且这种影响可能是无止境的。将网页的大部分甚至是全部的表示信息从HTML文件中移出，并将它们保留在一个样式表中有诸多优点，如降低文件大小、节省网络带宽以及易于维护等。此外，站点的表现信息和核心内容相分离，使得站点的设计人员能够在短暂的时间内对整个网站进行各种各样的修改。
            <br>CSS简化了网页的格式代码，外部的样式表还会被浏览器保存在缓存里，加快了下载显示的速度，也减少了需要上传的代码数量（因为重复设置的格式将被只保存一次）。只要修改保存着网站格式的CSS样式表文件就可以改变整个站点的风格特色，在修改页面数量庞大的站点时，显得格外有用。这就避免了一个个网页的修改，大大减少了工作量。</span>
    </div>
</body>

</html>
```

环绕效果为

![](image-20230617193654715.png)



#### 脱离文档流

具有 float 属性的元素

* 会**脱离原本的文档结构**，相邻元素的文本会绕开浮动元素
* 如果未指定宽高，则浮动元素的大小完全由内容决定
* 可以完美设置 margin 和 padding
* 不会被当做文本处理
* 浮动的元素之间具有文档结构，先后浮动的元素会按顺序排列
    * 浮动元素不会影响**之前未浮动元素**的位置
    * **浮动元素排列受到父元素宽度的影响**
    * 浮动可能会被未浮动元素卡住



例如考虑三个块元素的浮动情况

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>文本</title>
    <style>
        .outer {
            width: 500px;
            height: 300px;
            background-color: gray;
            padding: 10px;
        }

        .f1 {
            background-color: skyblue;
        }

        .f2 {
            background-color: orange;
            /* 向左浮动 */
            float: left;
            margin: 10px;
        }

        .f3 {
            background-color: green;
        }
    </style>
</head>

<body>
    <div class="outer">
        <div class="f1">元素1</div>
        <div class="f2">元素2</div>
        <div class="f3">元素3</div>
    </div>
</body>

</html>
```

实现效果为

![](image-20230617194832136.png)



可以构造一个被卡住的浮动

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>文本</title>
    <style>
        .outer {
            width: 200px;
            height: 300px;
            background-color: gray;
            padding: 10px;
        }

        .f1 {
            background-color: skyblue;
            float: left;
            width: 100px;
            height: 110px;
        }

        .f2 {
            background-color: orange;
            float: left;
            width: 100px;
            height: 100px;
        }

        .f3 {
            background-color: green;
            float: left;
            width: 100px;
            height: 100px;
        }
    </style>
</head>

<body>
    <div class="outer">
        <div class="f1">元素1</div>
        <div class="f2">元素2</div>
        <div class="f3">元素3</div>
    </div>
</body>

</html>
```

实现效果为

![](image-20230617200408660.png)



#### 高度塌陷

元素浮动后，**紧跟的元素会占据浮动元素的位置**，其文本内容会绕开浮动元素。并且，浮动的元素不作为父元素的内容，因此可能导致父元素的高度塌陷，但是父元素的宽度仍然控制浮动元素的排列。布局的重要原则是

> 设计浮动时，要么所有兄弟元素都浮动，要么都不浮动。

防止出现元素之间排列的混乱，解决第一个问题。现在剩下的问题就是**高度塌陷**。



典型的情况是

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>文本</title>
    <style>
        .outer {
            width: 500px;
            background-color: gray;
            padding: 10px;
        }

        .f {
            width: 100px;
            height: 100px;
            background-color: skyblue;
            border: 1px black solid;
            margin: 10px;
        }

        .f1,
        .f2,
        .f3,
        .f4 {
            float: left;
        }
    </style>
</head>

<body>
    <div class="outer">
        <div class="f f1">元素1</div>
        <div class="f f2">元素2</div>
        <div class="f f3">元素3</div>
        <div class="f f4">元素4</div>
    </div>
</body>

</html>
```

显示效果如下：由于 4 个元素都浮动，因此父元素的高度塌陷了。

![](image-20230617223920279.png)



要解决这一问题，有如下方案

1. 指定父元素高度
2. 指定父元素也浮动
3. 父元素添加 `overflow: hidden;`
4. 推荐使用伪元素清除浮动

```css
.outer::after {
    content: "";
    display: block;
    clear: both;
}
```

上述方法会选中标签最后的位置，添加一个空标签，设为**块元素**，使用 clear 清除所有浮动影响。等价于

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>文本</title>
    <style>
        .outer {
            width: 500px;
            background-color: gray;
            padding: 10px;
        }

        .f {
            width: 100px;
            height: 100px;
            background-color: skyblue;
            border: 1px black solid;
            margin: 10px;
        }

        .f1,
        .f2,
        .f3,
        .f4 {
            float: left;
        }

		.clear {
			display: block;
   			clear: both;
		}
    </style>
</head>

<body>
    <div class="outer">
        <div class="f f1">元素1</div>
        <div class="f f2">元素2</div>
        <div class="f f3">元素3</div>
        <div class="f f4">元素4</div>
        <div class="clear"></div>
    </div>
</body>
```



### 定位

通过 position 指定定位方式。**设置定位的元素具有比一般元素更高的层级**，因此可能会覆盖其它元素；**定位元素都具有相同的层级**；如果两个元素都开启定位，则后面的元素覆盖前面的元素。



#### 相对定位

开启**相对于当前位置**的移动距离。主要用于微调，或者配合绝对定位使用。指定

```css
p {
    /* 移动后的左边界距离当前位置 10px */
    position: relative;
    left: 10px;
}
```

不能同时设置 `left, right` 或者 `top, bottom`；不建议与浮动、margin 一起使用。



#### 绝对定位

当前元素**脱离文档结构**，开启**相对于包含块位置**的移动距离。

* 使用绝对定位的元素

    * 默认宽高由内容决定
    * 即使是行内元素，也可以调整宽高

* 包含块：

    1. 没有脱离文档结构的元素，以其父元素的**盒子**作为包含块
    2. 脱离文档结构的元素，以**第一个开启定位**的父元素的**盒子**作为包含块

    由于包含块是盒子，因此 padding 不会改变绝对定位。

不能同时设置 `left, right` 或者 `top, bottom`；不建议与 margin 一起使用；不能与浮动一起使用。



由于相对定位不会脱离文档结构，因此使用时通常在某个**父元素中添加相对定位**，然后将**子元素设为绝对定位**

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>文本</title>
    <style>
        .outer {
            width: 300px;
            height: 300px;
            background-color: gray;
            padding: 10px;
            position: relative;
        }

        .f {
            width: 150px;
            height: 150px;
            position: absolute;
            left: 0;
            top: 0;
            background-color: skyblue;
        }
    </style>
</head>

<body>
    <div class="outer">
        <div class="f">元素1</div>
    </div>
</body>

</html>
```



#### 固定定位

脱离文档结构，以**窗口坐标**为基准设置位置，因此对应的标签会随着窗口移动。指定

```css
p {
    position: fixed;
    bottom: 0;
    right: 0;
}
```

则标签始终位于窗口右下角。其特性与绝对定位相同。



#### 粘性定位

具有粘性定位的元素会跟随最近的**具有滚动机制**的父元素。

* 不会脱离文档结构
* 不推荐与浮动、margin 一起使用

粘性定位与相对定位特性基本一致，除了前者会被粘住。



例如实现标题跟随窗口移动的效果

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>文本</title>
    <style>
        /* 增加高度，出现滚动条 */
        body {
            height: 2000px;
        }

        .header {
            text-align: center;
            height: 100px;
            line-height: 100px;
            background-color: orange;
        }

        .tag {
            height: 50px;
            background-color: skyblue;
            font-size: 30px;
            /* 开启粘性定位 */
            position: sticky;
            top: 0;
        }

        .item {
            height: 50px;
            background-color: gray;
            font-size: 20px;
        }
    </style>
</head>

<body>
    <div class="header">标题</div>
    <div>
        <div class="tag">A</div>
        <div class="item">A1</div>
        <div class="item">A2</div>
        <div class="item">A3</div>
        <div class="item">A4</div>
    </div>
    <div>
        <div class="tag">B</div>
        <div class="item">B1</div>
        <div class="item">B2</div>
        <div class="item">B3</div>
        <div class="item">B4</div>
    </div>
    <div>
        <div class="tag">C</div>
        <div class="item">C1</div>
        <div class="item">C2</div>
        <div class="item">C3</div>
        <div class="item">C4</div>
    </div>
</body>

</html>
```



#### 定位层级

正常情况下，所有定位元素层级相同，且都高于一般元素。但是可以通过 z-index 属性改变层级

```css
p {
    position: absolute;
    z-index: 10;
}
```

默认情况下 z-index 为零，z-index 大的元素会覆盖 z-index 小的元素，z-index 为 auto 的元素不参与层级比较，z-index 为负值的元素被普通元素覆盖。**如果 z-index 大的元素没有覆盖 z-index 小的元素，则应该是其包含块的 z-index 太小**。



#### 特殊应用

##### 充满容器

如果希望具有绝对定位的元素充满容器，我们可以直接设置元素的 width 为 100% 。然而，由于 padding 和 border 的存在，总宽度会超出屏幕，导致子元素溢出。此时可以同时使用 left, right, top, bottom 实现。例如

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>文本</title>
    <style>
        .outer {
            width: 300px;
            height: 300px;
            background-color: gray;
            position: relative;
        }

        .inner {
            background-color: orange;
            border: 10px black solid;
            padding: 10px;
            /* 利用绝对定位的方位属性 */
            position: absolute;
            left: 0;
            right: 0;
            top: 0;
            bottom: 0;
        }
    </style>
</head>

<body>
    <div class="outer">
        <div class="inner">元素</div>
    </div>
</body>

</html>
```



##### 元素居中

如果指定了元素的宽高，想要使其水平、垂直居中，也可以借助绝对定位

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>文本</title>
    <style>
        .outer {
            width: 300px;
            height: 300px;
            background-color: gray;
            position: relative;
        }

        .inner {
            width: 100px;
            height: 100px;
            background-color: orange;
            border: 10px black solid;
            padding: 10px;
            /* 利用绝对定位的方位属性 */
            position: absolute;
            left: 0;
            right: 0;
            top: 0;
            bottom: 0;
            margin: auto;
        }
    </style>
</head>

<body>
    <div class="outer">
        <div class="inner">元素</div>
    </div>
</body>

</html>
```



### 布局

#### 版心

在 PC 端网页中，一般会有一个固定宽度水平居中的盒子，显示网页主要内容，称为**版心**。



#### 布局名称

常用布局名称

| 作用   | 名称     | 作用         | 名称              |
| ------ | -------- | ------------ | ----------------- |
| 页头   | header   | 页面主体     | main              |
| 页尾   | footer   | 内容         | content/container |
| 导航   | nav      | 侧栏         | sidebar           |
| 栏目   | column   | 页面外围控制 | wrapper           |
| 菜单   | menu     | 标题         | title             |
| 摘要   | summary  | 标志         | logo              |
| 广告   | banner   | 登陆         | login             |
| 登录条 | loginbar | 注册         | register          |
| 搜索   | search   | 功能区       | shop              |



#### 重置默认样式

为了避免默认样式在不同浏览器中显示效果不同，以及设计需要的问题，有必要重置默认样式。

* 在简单案例中，可以使用全局选择器

```css
* {
	margin: 0;
	padding: 0;
}
```

缺点在于不是所有元素都有默认样式，并且我们通常希望对不同元素具有不同的重置样式。

* 使用 reset.css 自行进行精细的样式重置
* 使用官方提供的 [Normalize.css](http://necolas.github.io/normalize.css/) 清除样式



### 表格样式

从 IE 8 开始提供了新的 display 属性，包括 `table, table-row, table-cell`，它标志着复杂 CSS 布局技术的结束，提供用 CSS 构建 HTML 表格布局的简单方法。

> 经典的 HTML 表格布局在页面完全加载后才显示，而 div 逐行显示，因此使用 CSS 布局可以一边加载一边显示。并且非表格内容也可以通过 div 标签和 table 显示模式布局。

表格显示模式主要如下：

| 显示模式                 | 特性                                |
| :------------------- | :-------------------------------- |
| `table`              | 此元素会作为块级表格来显示，表格前后带有换行符           |
| `inline-table`       | 此元素会作为内联表格来显示，表格前后没有换行符           |
| `table-row-group`    | 此元素 (`<tbody>`) 作为一个或多个行的分组来显示    |
| `table-header-group` | 此元素 (`<thead>`) 作为标题组来显示          |
| `table-footer-group` | 此元素 (`<tfoot>`) 作为脚注组来显示          |
| `table-row`          | 此元素 (`<tr>`) 会作为一个表格行来显示          |
| `table-column-group` | 此元素 (`<colgroup>`) 会作为一个或多个列分组来显示 |
| `table-column`       | 此元素 (`<col>`) 作为一个表格单元格列来显示       |
| `table-cell`         | 此元素 (`<td>, <th>`) 作为一个表格单元格来显示   |
| `table-caption`      | 此元素 (`<caption>`) 会作为一个表格标题来显示    |

还有一些协助属性：

| 属性                | 作用                                          |
| :---------------- | :------------------------------------------ |
| `border-collpase` | 用来决定表格的边框是分开的还是合并的                          |
| `border-spacing`  | 规定相邻单元格边框之间的距离（只适用于边框分离模式）                  |
| `table-layout`    | 定义了用于布局表格单元格，行和列的算法                         |
| `vertical-align`  | 用来指定行内元素（inline）或表格单元格（table-cell）元素的垂直对齐方式 |



#### 封装类

为了便于显示，首先将各种表格显示模式封装为样式类

```css
/* base.css */
.table {
    display: table;
}

.table-fixed {
    table-layout: fixed;
}

.table-row {
    display: table-row;
}

.table-cell {
    display: table-cell;
}

.table-colgroup {
    display: table-column-group;
}

.table-caption {
    display: table-caption;
}

.table-col {
    display: table-column;
}

.table-body {
    display: table-row-group;
}

.table-header {
    display: table-header-group;
}

.table-footer {
    display: table-footer-group;
}

.table-vt {
    vertical-align: top;
}

.table-vm {
    vertical-align: middle;
}

.table-vb {
    vertical-align: bottom;
}

/* 盒子布局 */
html,
body {
    height: 100%;
    margin: 0;
    padding: 0;
    background: #333;
}

.box1 {
    background: #6D5CAE;
}

.box2 {
    background: #48B0F7;
}

.box3 {
    background: #10CFBD;
}

h3 {
    padding-left: 20px;
    color: #fff;
}
```



#### 响应式布局

利用表格布局，将标签作为表格单元，实现响应式布局

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <link rel="stylesheet" href="base.css">
    <style>
        div.demo1 {
            width: 100%;
            height: 200px;
        }
    </style>
</head>

<body>
    <h3>响应式布局</h3>
    <div class="table demo1">
        <div class="box1 table-cell">1</div>
        <div class="box2 table-cell">2</div>
        <div class="box3 table-cell">3</div>
    </div>
</body>

</html>
```

显示效果如下，单元格会根据网页宽度变化

![](image-20240708235719957.png)



#### 纵向填充

使用 `table-row` 样式生成的表格行会自动填满高度

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <link rel="stylesheet" href="base.css">
    <style>
        .demo2 {
            width: 400px;
            height: 300px;
        }

        .demo2 div.table-header-group {
            padding: 5px 20px;
            background: #10CFBD;
        }

        .demo2 .main {
            height: 100%;
            background: #48B0F7;
        }

        .demo2 div.table-footer-group {
            padding: 5px 20px;
            background: #10CFBD;
        }
    </style>
</head>

<body>
    <h3>纵向填充</h3>
    <div class="table demo2">
        <div class="table-header-group">标题</div>
        <div class="main table-row">纵向填充</div>
        <div class="table-footer-group">脚注</div>
    </div>
</body>

</html>
```

显示效果如下

![](image-20240709000319227.png)



#### 动态垂直居中

表格单元配合 `vertical-align: middle;` 实现垂直居中

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <link rel="stylesheet" href="base.css">
    <style>
        .demo3 {
            width: 400px;
            height: 300px;
            background: #10CFBD;
        }

        .center-box {
            width: 100px;
            height: 100px;
            background: #6D5CAE;
            display: inline-block;
        }
    </style>
</head>

<body>
    <h3>动态垂直居中对齐</h3>
    <div class="table demo3">
        <div class="table-cell table-vm">
            <div class="center-box">123</div>
        </div>
    </div>
</body>

</html>
```

显示效果如下

![](image-20240709001227766.png)



#### 动态水平居中

表格元素可通过 `text-align: center;` 实现动态水平居中

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <link rel="stylesheet" href="base.css">
    <style>
        .demo4 {
            width: 400px;
            height: 300px;
            background: #10CFBD;
            text-align: center;
        }

        .center-box {
            width: 100px;
            height: 100px;
            background: #6D5CAE;
            display: inline-block;
        }
    </style>
</head>

<body>
    <h3>动态水平居中对齐</h3>
    <div class="table demo4">
        <div class="center-box">123</div>
    </div>
</body>

</html>
```

显示效果如下

![](image-20240709001457330.png)



#### 动态居中对齐

将上述两种样式结合就得到标签居中样式

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <link rel="stylesheet" href="base.css">
    <style>
        .demo4 {
            width: 400px;
            height: 300px;
            background: #10CFBD;
            text-align: center;
        }

        .center-box {
            width: 100px;
            height: 100px;
            background: #6D5CAE;
            display: inline-block;
        }
    </style>
</head>

<body>
    <h3>动态居中对齐</h3>
    <div class="table demo4">
        <div class="table-cell table-vm">
            <div class="center-box"></div>
        </div>
    </div>
</body>

</html>
```

显示效果如下

![](image-20240709003244173.png)



#### 动态两端对齐

由于表格单元格可以通过 `text-align` 设置，因此很容易实现两端对齐

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <link rel="stylesheet" href="base.css">
    <style>
        div.table {
            width: 100%;
            height: 200px;
        }

        .left {
            text-align: left;
            width: 30%;
        }

        .right {
            text-align: right;
            width: 30%;
        }
    </style>
</head>

<body>
    <h3>动态两端对齐</h3>
    <div class="table">
        <div class="table-cell left box1">左盒子</div>
        <div class="table-cell right box2">右盒子</div>
    </div>
</body>

</html>
```

显示效果如下

![](image-20240709002447566.png)





### CSS3

#### 私有前缀

W3C 标准提出的 CSS 特性，在被浏览器正式支持之前，浏览器厂商会根据浏览器内核，使用私有前缀测试该特性。例如

* Chorme 前缀 `-webkit-`
* Firefox 前缀 `-moz-`

当浏览器正式支持该特性后，就不再需要前缀。



#### 长度单位

* rem 根元素字体大小的倍数
* vw 视口宽度的百分比，例如 10vw 表示 10%
* vh 视口高度的百分比

这里 `vm, vh` 的百分比始终以视口为基准，而一般的百分比 ` % ` 以父元素为基准。



#### 计算函数

##### calc

使用视口高度计算容器高度，使得**头部和内容区恰好填满窗口**

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>文本</title>
    <style>
        body {
            margin: 0;
        }

        .page-header {
            width: 100%;
            height: 70px;
            background-color: #333;
        }

        .page-content {
            width: 100%;
            height: calc(100vh - 70px);
            background-color: skyblue;
        }
    </style>
</head>

<body>
    <header class="page-header"></header>
    <div class="page-content"></div>
</body>

</html>
```



##### min & max

可取两个值之间的最小最大值

```css
.element {
    width: min(50%, 300px);
    height: max(50%, 200px);
}
```



##### clamp

提供一个最小值、中间值和最大值，获得约束范围内的值

```css
.element {
    font-size: clamp(1rem, 2.5vw, 2rem);
}
```



#### 计数器

计数器用于在文档中创建和管理数字计数。它们常用于生成自动编号的列表、章节标题。



##### 创建计数器

在指定标签样式中初始化计数器，这里相当于定义一个 `section-counter` 变量

```css
body {
    counter-reset: section-counter;
}
```



##### 增加计数器

使用 `counter-increment` 属性标记对应的标签，每当遇到对应的标签，则计数器增加

```css
h2 {
    counter-increment: section-counter;
}
```



##### 显示计数器

在伪元素中使用 `content` 属性，结合 `counter` 函数显示计数值

```css
h2::before {
    content: "Section " counter(section-counter) ": ";
}
```



##### 使用范例

在章节前显示章节序号的范例如下

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS Counter Example</title>
    <style>
        body {
            counter-reset: section-counter;
            /* 初始化计数器 */
        }

        h2 {
            counter-increment: section-counter;
            /* 每次遇到 h2 时计数器加一 */
        }

        h2::before {
            /* 显示计数器值 */
            content: "Section " counter(section-counter) ": ";
            font-weight: bold;
        }
    </style>
</head>

<body>
    <h2>介绍</h2>
    <p>这里是介绍部分</p>

    <h2>章节 1</h2>
    <p>第一章节的内容</p>

    <h2>章节 2</h2>
    <p>第二章节的内容</p>
</body>

</html>
```

显示效果如下

![](image-20240709234045102.png)



#### 变量

原生 CSS 不支持变量，因此 LESS, SCSS 等兼容 CSS 的扩展语言弥补了这一问题。后来 CSS 也开始逐步支持变量（自定义属性），并在大多数主流浏览器中得到支持。



##### 声明变量

变量名以连横线开头，可以在伪类 `:root` 中声明，作为全局变量

```css
:root {
	--global-color: #666;
}
```

在一般选择器中声明，作为局部变量，作用域为该选择器和其子元素

```css
.element {   
	--main-bg-color: brown;
}
```

子选择器中声明的变量可以覆盖原先的变量

```css
:root {   
	--main-bg-color: brown;
}

.element {   
	--main-bg-color: red;
}
```



##### 使用变量

通过 `var` 函数使用变量

```css
:root {   
	--main-bg-color: brown;
	--margin-width: 20px;
}

/* 直接使用 */
.element {
	background-color: var(--main-bg-color);
}

/* 第二个参数提供默认值，用于变量未定义的情形 */
.box {
	background-color: var(--main-bg-color, yellow);
	margin: var(--margin-width, 10px 10px 10px 10px);
}
```

还可以用 `var` 声明变量

```css
.element {
	--margin-top: var(--margin-width);
}
```



##### 字符串拼接

两个字符串的值可以拼接

```css
:root {
	--text-one: 'hello';
	--text-two: 'world';
}

.box:after {
	content: var(--text-one)' ' var(--text-two);
}
```



##### 数值转字符串

结合计数器 counter 可以实现数值转字符串

```css
.pie {
	--p: 25;
}

.pie:after {
	counter-reset: p var(--p);
	content: counter(p) '%';
}
```



##### 任意赋值

变量值可以取任意表达式，无论属性值是否合法

```css
:root {
    --margin-top: calc(2vh + 20px);
    --foo: if(x > 5) this.width=10;
}
```





#### 盒子属性

由于显示缩放的问题，当我们设置边框时，它会被缩放。例如

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>文本</title>
    <style>
        .box {
            width: 200px;
            height: 200px;
            padding: 5px;
            background-color: skyblue;
            border: 5px solid black;
        }
    </style>
</head>

<body>
    <div class="box"></div>
</body>

</html>
```

盒子尺寸如下：可以发现边框宽度错误

![](image-20230618164933421.png)



##### box-sizing

新增 box-sizing 属性，调整盒子类型

* content-box 此时 width, height 设置盒子内容的大小（默认）
* border-box 此时 width, height 设置盒子总大小（包括边框）

后者称为**怪异盒模型**。



##### resize

新增 resize 属性允许调整标签大小

* horizontal 水平可调
* vertical 垂直可调
* both 水平、垂直都可调

需要设置 overflow 属性才能启用此属性。



##### box-shadow

新增 box-shadow 属性设置阴影位置

```css
box-shadow: 10px 10px;						/* 水平、垂直位移 */
box-shadow: 10px 10px blue;					/* 水平、垂直位移+颜色 */
box-shadow: 10px 10px 20px blue;			/* 水平、垂直位移+模糊程度+颜色（最常用） */
box-shadow: 10px 10px 20px 10px blue;		/* 水平、垂直位移+模糊程度+外延宽度+颜色 */
box-shadow: 10px 10px 20px 10px blue inset;	/* 水平、垂直位移+模糊程度+外延宽度+颜色+内阴影 */
```



##### opacity

新增 opacity 属性设置标签透明度

```css
opacity: 0.5;
```

属性取值在 0-1 之间。



#### 背景属性

##### background-origin

设置背景图原点的位置。可选

* padding-box 从 padding 区域开始显示背景图像（默认）
* border-box 从 border 区域开始显示背景图像
* content-box 从 content 区域开始显示背景图像



##### background-clip

修剪背景图。可选

* border-box 从 border 区域开始向外裁剪图像（默认）
* padding-box 从 padding 区域开始向外裁剪图像
* content-box 从 content 区域开始向外裁剪图像
* text 背景图**只显示在文字上**（需要 background-clip 增加 `-webkit-` 前缀）



##### background-size

设置背景图的大小，用两个参数设置宽高

```css
background-size: 100px 100px;
background-size: 100% 100%;
```

还可选值

* contain 自动缩放图片，使容器能够容纳整个图片，可能导致容器有部分空白
* cover 自动缩放图片，使图片恰好填满整个容器（推荐）



##### background

之前介绍过这个复合属性，但是它并不常用。不过可以利用它设置多个背景图片，例如

```css
background: url("img1.png") no-repeat,
			url("img2.png") no-repeat right top,
			url("img3.png") no-repeat left bottom,
			url("img4.png") no-repeat right bottom;
```

在容器四角添加不同的背景图片。



#### 边框属性

增加 border-radius 属性指定边框四角的半径。利用该属性可以绘制一个圆，只需要将半径设为宽高的一半

```css
.box {
    width: 400px;
    height: 400px;
    border-radius: 50%;
}
```



#### 文本属性

##### text-shadow

为文本添加阴影，常用于产生光晕效果。例如

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>文本</title>
    <style>
        body {
            background-color: black;
        }

        .box {
            color: white;
            font-size: 30px;
            text-align: center;
            text-shadow: 0 0 20px red;
        }
    </style>
</head>

<body>
    <div class="box">你好啊</div>
</body>

</html>
```

显示效果为

![](image-20230618194327160.png)



##### white-space

控制文本换行。可选值

* normal 文本超出边界自动换行，文本中的换行被识别为空格（默认）
* pre 按照原文显示
* pre-wrap 按照原文显示，超出边界自动换行
* pre-line 按照原文显示，超出边界自动换行，忽略首尾的空格，文本中的换行被识别为空格
* **nowrap 强制不换行，文本中的换行被识别为空格（最常用）**



##### text-overflow

控制文本溢出样式。常用值有

* clip 行内内容溢出时，将溢出部分裁减掉（默认）
* ellipsis 行内内容溢出块容器时，将溢出部分替换为 ...

需要指定 `white-space: nowrap;` 且具有非 visible 的 `overflow` 属性。



#### 渐变

善用渐变可以实现非常好的视觉效果，例如利用透明色和灰色渐变实现网格纸效果。



##### 线性渐变

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>文本</title>
    <style>
        .box {
            width: 200px;
            height: 200px;
            float: left;
            margin-left: 50px;
            font-size: 20px;
            border: 1px solid black;
        }

        /* 三色线性渐变（可以增加更多颜色） */
        .box1 {
            background-image: linear-gradient(red, yellow, green);
        }

        /* 修改渐变方向 */
        .box2 {
            background-image: linear-gradient(to right top, red, yellow, green);
        }

        /* 渐变角度 */
        .box3 {
            background-image: linear-gradient(20deg, red, yellow, green);
        }

        /* 指定 50px 位置为红，100px 位置为黄，150px 位置为绿 */
        .box4 {
            background-image: linear-gradient(red 50px, yellow 100px, green 150px);
        }
    </style>
</head>

<body>
    <div class="box box1">默认情况（从上到下）</div>
    <div class="box box2">修改渐变方向</div>
    <div class="box box3">调整渐变角度</div>
    <div class="box box4">指定颜色位置</div>
</body>

</html>
```

显示效果为

![](image-20230618201825517.png)



##### 径向渐变

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>文本</title>
    <style>
        .box {
            width: 200px;
            height: 200px;
            float: left;
            margin-left: 50px;
            font-size: 20px;
            border: 1px solid black;
        }

        .box1 {
            background-image: radial-gradient(red, yellow, green);
        }

        .box2 {
            background-image: radial-gradient(at 100px 50px, red, yellow, green);
        }
        
        .box3 {
            background-image: radial-gradient(100px 50px, red, yellow, green);
        }

        .box4 {
            background-image: radial-gradient(circle at 100px 50px, red 50px, yellow 100px, green 150px);
        }
    </style>
</head>

<body>
    <div class="box box1">默认情况</div>
    <div class="box box2">调整圆心</div>
    <div class="box box3">调整半径</div>
    <div class="box box4">调整颜色位置，指定正圆，并设置圆心</div>
</body>

</html>
```

显示效果为

![](image-20230618201719756.png)



##### 重复渐变

增加 repeating 前缀设置重复渐变

```css
background-image: repeating-linear-gradient(red, yellow, green);
```

将会在**没有发生渐变的位置开始重复渐变**。对于径向渐变同理。



#### 2D 变换

使用 transform 进行变换。



##### 位移

指定 transform 属性为

| 值         | 含义                                                |
| ---------- | --------------------------------------------------- |
| translateX | 沿 x 方向移动，指定长度值；可以指定自身宽度的百分比 |
| translateY | 沿 y 方向移动，指定长度值；可以指定自身高度的百分比 |
| translate  | 指定两个长度值；可以指定自身宽度、高度的百分比      |

注意点

* 位移不脱离文档结构，且效率更高
* 位移对**行内元素**无效



例如创建一个居中的盒子

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>文本</title>
    <style>
        .outer {
            width: 200px;
            height: 200px;
            font-size: 20px;
            border: 1px solid black;
            position: relative;
        }

        .inner {
            width: 60px;
            height: 60px;
            background-color: skyblue;
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
        }
    </style>
</head>

<body>
    <div class="outer">
        <div class="inner"></div>
    </div>
</body>

</html>
```



##### 缩放

可用缩放实现小于 12 px 的文字。

| 值      | 含义      |
| ------ | ------- |
| scaleX | 沿水平方向缩放 |
| scaleY | 沿垂直方向缩放 |
| scale  | 组合缩放    |



##### 旋转

使用 rotateZ 以**容器中心**为原点进行平面旋转。例如

```css
transform: rotateZ(30deg);
```

注意**旋转会转动自身的 x, y 轴方向，因此再进行平移的效果会有不同**。



##### 多重变换

通过空格间隔，依次指定变换

```css
transform: translate(-50%, -50%) rotateZ(30deg);
```

建议将旋转放在最后，防止坐标轴转动的影响。



##### 变换原点

可以修改转动的原点，使用 transform-origin 属性，可选值如

* left 左边中心
* top 上边中心
* right top 右上角
* right bottom 右下角

依此类推。还可以指定一个或两个像素值/百分比调整。



#### 3D 变换

需要开启 3D 空间，然后设置景深实现透视效果

```css
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>文本</title>
    <style>
        .outer {
            width: 200px;
            height: 200px;
            font-size: 20px;
            border: 1px solid black;
            /* 在父容器中设置变换空间和景深 */
            transform-style: preserve-3d;
            perspective: 100px;
        }

        .inner {
            width: 200px;
            height: 200px;
            background-color: skyblue;
            transform: rotateX(45deg);
        }
    </style>
</head>

<body>
    <div class="outer">
        <div class="inner"></div>
    </div>
</body>

</html>
```

实现效果如下

![](image-20230618211243427.png)



在父元素中可以进一步调整

* perspective-origin 设置透视点（观察点）位置

在 3D 空间中的变换与 2D 变换形式类似

* 增加 translateZ, translate3d
* 增加 rotateX, rotateY（最常使用）
* 增加 scaleZ 缩放透视点的位置



当元素被转动，背部显示出来后，默认能够看到反转的内容。通过

```css
backface-visibility: hidden;
```

设置背部不可见。



#### 过渡

过渡可以在不使用 js 的情况下，实现样式的平滑过渡。



常用属性

* transition-property 指定需要过渡的属性

    * none 不过渡
    * all 过渡所有可以过渡的属性
    * 指定属性名，例如 `transition-property: width, height, background-color;`

    通常能转化为数值的属性都可以过渡。

* transition-duration 指定过渡时间

    * 可以指定一个列表 `transition-duration: 1s, 0.5s, 0.2s` 过渡时间对应前面指定的属性名

* transition-timing-function 指定过渡函数。常用值

    * ease 平滑过渡（默认）
    * linear 线性过渡
    * steps() 指定步数
    * cubic-bezier() 指定贝塞尔曲线过渡

    可以在线制作[贝塞尔曲线](https://cubic-bezier.com/#.17,.67,.83,.67)，复制 css 格式。

* transition 复合属性，其中 duration 和 delay 要先后指定，其它属性没有顺序要求

需要注意过渡最好指定在元素本身上，而不是 `:hover` 等伪元素上。



例如实现一个鼠标悬停产生过渡的效果

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>文本</title>
    <style>
        .outer {
            width: 200px;
            height: 200px;
            border: 1px solid black;
        }

        .box {
            width: 100px;
            height: 100px;
            background-color: orange;
            opacity: 0.5;
            transition-property: all;
            transition-duration: 1s;
        }

        /* 选中 hover 状态下 .outer 对应标签的子标签 .box */
        .outer:hover .box {
            width: 200px;
            height: 200px;
            background-color: green;
            opacity: 1;
            transform: rotate(45deg);
            box-shadow: 0 0 20px black;
            transition-timing-function: cubic-bezier();
        }
    </style>
</head>

<body>
    <div class="outer">
        <div class="box"></div>
    </div>
</body>

</html>
```



#### 动画

指定动画的关键帧，然后为特定的标签添加动画。例如

````html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>文本</title>
    <style>
        .outer {
            width: 600px;
            height: 100px;
            border: 1px solid black;
        }

        @keyframes MoveToRight {
            /* 第一帧 */
            from {}

            /* 最后一帧 */
            to {
                transform: translate(500px);
                background-color: red;
            }
        }

        .box {
            width: 100px;
            height: 100px;
            background-color: orange;
            /* 指定动画名，动画时间，动画延迟 */
            animation-name: MoveToRight;
            animation-duration: 3s;
            animation-delay: 0.3s;
        }
    </style>
</head>

<body>
    <div class="outer">
        <div class="box"></div>
    </div>
</body>

</html>
````

由于浏览器加载延迟的问题，需要通过 delay 设置延迟，确保动画完全播放。



还可以更精细地指定关键帧

```css
@keyframes MoveToRight {
    0% {}

    50% {
        background-color: green;
    }
    
    75% {
        background-color: blue;
    }

    100% {
        transform: translate(500px);
        background-color: red;
    }
}
```



还有更多属性

| 属性                      | 作用                                                         |
| ------------------------- | ------------------------------------------------------------ |
| animation-timing-function | 动画速度函数，与过渡速度函数类似                             |
| animation-iteration-count | 动画播放次数，指定数字或 infinite                            |
| animation-direction       | 动画播放方向。可选 normal, reverse, alternate (交替), alternate-reverse |
| animation-fill-mode       | 动画结束时的状态。可选 forwards (停留在最后一帧), backwards (回到第一帧) |
| animation-play-state      | 动画播放状态。可选 running (默认), paused (暂停)             |

复合属性 animation

> 只要求 duration 和 delay 先后指定，其它属性没有顺序要求。

通常 animation-play-state 会单独使用。



例如添加无限交替播放，当鼠标悬停时暂停的效果

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>文本</title>
    <style>
        .outer {
            width: 600px;
            height: 100px;
            border: 1px solid black;
        }

        @keyframes MoveToRight {
            0% {}

            50% {
                background-color: green;
            }

            100% {
                transform: translate(500px);
                background-color: red;
            }
        }

        .box {
            width: 100px;
            height: 100px;
            background-color: orange;
            /* 指定动画名，动画时间，动画延迟 */
            animation-name: MoveToRight;
            animation-duration: 2s;
            animation-delay: 0.3s;

            /* 无限交替播放 */
            animation-iteration-count: infinite;
            animation-direction: alternate;
            
        }

        .outer:hover .box {
            animation-play-state: paused;
        }
    </style>
</head>

<body>
    <div class="outer">
        <div class="box"></div>
    </div>
</body>

</html>
```



#### 多列布局

使用 column-count 属性指定元素的列数，或者通过 column-width 指定每列宽度，自动计算列数。这种布局的优势是能够自动排列元素，其**内部的元素宽度以每列的宽度为基准**。例如排版图片时，只需要指定宽度为 100% 就可以适应列宽

```css
img {
    width: 100%
}
```

还可以使用更多属性

| 属性              | 作用         |
| ----------------- | ------------ |
| column-gap        | 调整列间距   |
| column-rule-width | 每列边框宽度 |
| column-rule-style | 每列边框风格 |
| column-rule-color | 每列边框颜色 |

如果要允许其中的元素跨列，需要**在子元素中**指定

```css
column-span: all;
```



#### 弹性盒子

它可以轻松地控制：元素分布方式、元素对齐方式、元素视觉顺序等。以它为基础，演变出新的布局方案——flex 布局。

* 传统布局是指基于 display + position + float 的布局方案
* flex 布局在移动端应用比较广泛



弹性项目不会脱离文档结构，因此使用弹性盒子的容器的默认大小由内容确定，**不会出现浮动产生的高度塌陷**。

* 弹性盒子：开启 `display: flex;` 的元素成为弹性盒子
    * 设置 `display: inline-flex;` 使弹性盒子成为行内块元素（很不常用）
* 弹性项目：弹性盒子的所有**子元素**自动成为弹性项目
    * 弹性项目会自动成为块元素



弹性盒子由两个垂直的轴确定延伸方向

* 主轴：main axis
* 侧轴（交叉轴）：cross axis

![](Screenshot_20230619_103122.jpg)



例如利用弹性盒子实现并排的块

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>文本</title>
    <style>
        .outer {
            width: 600px;
            height: 600px;
            background-color: gray;
            display: flex;
        }

        .inner {
            width: 200px;
            height: 200px;
            background-color: skyblue;
            border: 1px solid black;
            box-sizing: border-box;
        }
    </style>
</head>

<body>
    <div class="outer">
        <div class="inner">1</div>
        <div class="inner">2</div>
        <div class="inner">3</div>
    </div>
</body>

</html>
```



##### 主轴方向

使用 flex-direction 属性设置主轴方向

| 值             | 含义             |
| -------------- | ---------------- |
| row            | 从左向右（默认） |
| row-reverse    | 从右向左         |
| column         | 从上到下         |
| column-reverse | 从下到上         |

当主轴方向改变，侧轴方向也会对应地改变。



##### 主轴换行

使用 flex-wrap 属性设置主轴换行方式

| 值           | 含义                                                         |
| ------------ | ------------------------------------------------------------ |
| nowrap       | 不换行（默认）。当内容宽度超过容器给定宽度时，**会压缩内容宽度** |
| wrap         | 换行（最常用）。超出容器给定宽度就会换行                     |
| wrap-reverse | 反向换行                                                     |



可以将上述两个属性进行复合，没有顺序要求

```css
flex-flow: row wrap;
```



##### 主轴对齐

使用 justify-content 属性设置主轴对齐方式

| 值            | 含义                               |
| ------------- | ---------------------------------- |
| flex-start    | 主轴起点对齐（默认）               |
| flex-end      | 主轴终点对齐                       |
| center        | 居中对齐（所有元素紧挨）           |
| space-between | 均匀分布，两端对齐（最常用）       |
| space-around  | 均匀分布，两端距离是中间距离的一半 |
| space-evenly  | 均匀分布，两端距离和中间距离相等   |



##### 侧轴对齐

使用 align-items 属性设置侧轴**单行对齐**方式

| 值         | 含义                                             |
| ---------- | ------------------------------------------------ |
| flex-start | 侧轴起点对齐                                     |
| flex-end   | 侧轴终点对齐                                     |
| center     | 侧轴中点对齐                                     |
| stretch    | 如果内容未指定高度，将占满整个容器的高度（默认） |



使用 align-content 属性设置侧轴**多行对齐**方式

| 值            | 含义                                             |
| ------------- | ------------------------------------------------ |
| flex-start    | 侧轴起点对齐                                     |
| flex-end      | 侧轴终点对齐                                     |
| center        | 居中对齐（所有元素紧挨）                         |
| space-between | 均匀分布，两端对齐（最常用）                     |
| space-around  | 均匀分布，两端距离是中间距离的一半               |
| space-evenly  | 均匀分布，两端距离和中间距离相等                 |
| stretch       | 如果内容未指定高度，将占满整个容器的高度（默认） |

如果指定了高度，则每行的高度以最高的元素为准。



##### 水平垂直居中

使用弹性盒子，有两种简单的水平垂直居中方案

```css
/* 方案一 */
.outer {
    width: 400px;
    height: 400px;
    background-color: gray;
    display: flex;
    justify-content: center;
    align-items: center;
}

.inner {
    width: 100px;
    height: 100px;
    background-color: orange;
}

/* 方案二 */
.outer {
    width: 400px;
    height: 400px;
    background-color: gray;
    display: flex;
}

.inner {
    width: 100px;
    height: 100px;
    background-color: orange;
    margin: auto;
}
```



##### 伸缩性

使用 flex-grow 属性定义弹性项目的放大比例，默认为 0 。例如

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>文本</title>
    <style>
        .outer {
            width: 400px;
            height: 400px;
            background-color: gray;
            display: flex;
        }

        .inner {
            width: 100px;
            height: 100px;
            background-color: orange;
            border: 1px solid black;
        }

        /* 按照 1:2:3 的比例获得剩余空间 */
        .inner1 {
            flex-grow: 1;
        }

        .inner2 {
            flex-grow: 2;
        }

        .inner3 {
            flex-grow: 3;
        }
    </style>
</head>

<body>
    <div class="outer">
        <div class="inner inner1">1</div>
        <div class="inner inner2">2</div>
        <div class="inner inner3">3</div>
    </div>
</body>

</html>
```

正常排列的三个元素共占用 300px，指定 flex-grow 后，将会按照比例获得剩下的 100px 空间。



使用 flex-shrink 属性定义弹性项目的压缩样式，默认为 1 。规则是

1. 将 flex-shrink 与对应的项目宽度相乘相加得到总宽度
2. 将**项目宽度与总宽度的比值**作为项目的压缩比
3. 根据压缩比分配需要收缩的宽度

使用这种压缩方式，而不是直接分配收缩宽度，是为了防止某个元素被压缩为零。



##### 排序

使用 order 属性对弹性项目进行排序，默认情况下每个元素的 order 值为零。设置 order 后，按照 order 值从大到小排列。



#### 响应式布局

响应式布局能够根据窗口大小、不同设备的不同，修改布局方式。



##### 媒体查询

可以根据设备不同设置不同样式。例如

```css
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <title>文本</title>
    <style>
        .page-header {
            width: 100%;
            height: 70px;
            background-color: #333;
        }

        /* 在屏幕上显示的样式 */
        @media screen {
            body {
                background-color: yellow;
            }
        }

        /* 在打印时显示的样式 */
        @media print {
            body {
                background-color: blue;
            }
        }
    </style>
</head>

<body>
    <header class="page-header"></header>
    <div class="page-content">
    </div>
</body>

</html>
```

正常网页将显示黄色背景，而打印时显示蓝色背景。

![](image-20230619191017414.png)



媒体查询可用的值有

| 值     | 含义         |
| ------ | ------------ |
| all    | 检测所有设备 |
| print  | 检测打印机   |
| screen | 检测屏幕     |



##### 媒体特性

进一步增加条件样式

```css
/* 当宽度等于 800px 时应用此样式 */
@media (width:800px) {
    body {
        background-color: white;
    }
}

/* 当宽度大于 800px 时应用此样式 */
@media (min-width:800px) {
    body {
        background-color: yellow;
    }
}

/* 当宽度小于 800px 时应用此样式 */
@media (max-width:800px) {
    body {
        background-color: orange;
    }
}
```

高度属性也可以设置条件。值得一提的是

* device-width 属性，检测设备宽度
* orientation 属性，检测旋转方向
    * portrait 纵向
    * landscape 横向

使用它根据设备宽度改变样式。**当多个条件样式冲突时，后面的会覆盖前面的**。



条件可以使用运算符

```css
/* 屏幕显示，且宽度在 800-1200 之间 */
@media screen and (min-width:800px) and (max-width:1200px) {
    body {
        background-color: yellow;
    }
}

/* 宽度小于 800 或大于 1200 */
@media (min-width:1200px) or (max-width:800px) {
    body {
        background-color: yellow;
    }
}

/* 宽度大于 800 */
@media not (max-width:800px) {
    body {
        background-color: orange;
    }
}
```



#### BFC

BFC 可用于解决三类问题，一旦开启 BFC

* 子元素没有 margin 塌陷问题
* 自己不会被浮动元素覆盖
* 子元素浮动不会导致高度塌陷

开启 BFC 的元素有

* 根元素
* 浮动元素
* 绝对定位、固定定位元素
* 行内块元素
* 表格单元格
* overflow 值不为 visible 的块元素
* 弹性盒子
* 多列容器
* column-span 为 all 的元素
* `display: flow-root;`

其中最后一种方案问题最少，但可能有 IE 的兼容问题。



## Bootstrap

Bootstrap 是一个灵活的前端框架，提供了预设的标签和样式框架。



### 使用方式

从官网下载编译版本，解压后放在工作目录下即可开始使用。

```embed
title: "Bootstrap中文网"
image: "https://www.bootcss.com/assets/brand/bootstrap-social.png"
description: "Bootstrap是Twitter推出的一个用于前端开发的开源工具包。它由Twitter的设计师Mark Otto和Jacob Thornton合作开发，是一个CSS/HTML框架。目前，Bootstrap最新版本为5.0 。Bootstrap中文网致力于为广大国内开发者提供详尽的中文文档、代码实例等，助力开发者掌握并使用这一框架。"
url: "https://www.bootcss.com/"
```



#### 创建框架

引入兼容性框架和 bootstrap 样式

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <!--[if lt IE 9]-->
    <script src="" https://oss.maxcdn.com/html5shiv/3.7.3/html5shiv.min.js"></script>
    <script src="https://oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>
    <!--[endif]-->
    <!-- bootstrap 核心样式 -->
    <link rel="stylesheet" href="bootstrap/css/bootstrap.min.css">
    <title>Document</title>
</head>

<body>
    <button type="button" class="btn btn-success">登录</button>
    <div class="btn btn-danger">未登录</div>
</body>

</html>
```

只需要修改类名就可以调整标签样式，也可以覆盖原先的样式。



### 布局容器

Bootstrap 提供默认的布局容器类

* `container` 响应式布局容器，固定宽度
* `container-fluid` 流式布局容器，百分百宽度，适用于单独移动端开发

只需要将类名填充在对应标签中即可使用。



#### 栅格系统

栅格系统将页面布局划分为等宽的列，然后通过列数定义来模块化页面布局。Bootstrap 提供响应式移动设备优先的流式栅格系统，随着屏幕或视口 viewport 尺寸的增加，系统自动划分为 12 列。



栅格系统使用 `row, column` 的组合创建页面布局。通过对应类前缀实现划分，每一列默认有 15 像素的内边距，可以同时指定多个类前缀（`col-md-4 col-sm-6`）。选项参数包括

|                  | 手机 `<768px` | 平板 `>=768px` | 桌面显示器 `>=992px` | 大桌面显示器 `>=1200px` |
| :--------------- | :---------- | :----------- | :-------------- | :---------------- |
| `container` 最大宽度 | 自动 `100%`   | `750px`      | `970px`         | `1170px`          |
| 类前缀              | `col-xs-`   | `col-sm-`    | `col-md-`       | `col-lg-`         |
| 列数               | `12`        | `12`         | `12`            | 12                |



#### 列数划分

列数会影响布局划分方式，例如

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <!--[if lt IE 9]-->
    <script src="" https://oss.maxcdn.com/html5shiv/3.7.3/html5shiv.min.js"></script>
    <script src="https://oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>
    <!--[endif]-->
    <!-- bootstrap 核心样式 -->
    <link rel="stylesheet" href="bootstrap/css/bootstrap.min.css">
    <title>Document</title>
    <style>
        [class^="col"] {
            border: 1px solid #ccc;
        }
    </style>
</head>

<body>
    <div class="container">
    	<!-- 如果列数等于 12，则填满宽度 -->
        <div class="row">
            <div class="col-lg-3">1</div>
            <div class="col-lg-3">2</div>
            <div class="col-lg-3">3</div>
            <div class="col-lg-3">4</div>
        </div>
        <div class="row">
            <div class="col-lg-6">1</div>
            <div class="col-lg-2">2</div>
            <div class="col-lg-2">3</div>
            <div class="col-lg-2">4</div>
        </div>
        <!-- 如果列数小于 12，则留空 -->
        <div class="row">
            <div class="col-lg-6">1</div>
            <div class="col-lg-2">2</div>
            <div class="col-lg-1">3</div>
            <div class="col-lg-1">4</div>
        </div>
        <!-- 如果列数大于 12，则换行 -->
        <div class="row">
            <div class="col-lg-6">1</div>
            <div class="col-lg-2">2</div>
            <div class="col-lg-3">3</div>
            <div class="col-lg-3">4</div>
        </div>
    </div>
</body>

</html>
```

显示效果如下

![](image-20240711201746467.png)



#### 列嵌套

在每个列中也可以嵌套表格

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <!--[if lt IE 9]-->
    <script src="" https://oss.maxcdn.com/html5shiv/3.7.3/html5shiv.min.js"></script>
    <script src="https://oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>
    <!--[endif]-->
    <!-- bootstrap 核心样式 -->
    <link rel="stylesheet" href="bootstrap/css/bootstrap.min.css">
    <title>Document</title>
    <style>
        [class^="col-lg"] {
            background-color: rgb(145, 255, 0);
            height: 50px;
        }

        [class^="col-md"] {
            background-color: rgb(0, 255, 200);
            height: 50px;
        }
    </style>
</head>

<body>
    <div class="container">
        <div class="row">
            <div class="col-lg-3">
                <div class="row">
                    <div class="col-md-3">1</div>
                    <div class="col-md-3">2</div>
                    <div class="col-md-3">3</div>
                    <div class="col-md-3">4</div>
                </div>
            </div>
            <div class="col-lg-3">2</div>
            <div class="col-lg-3">3</div>
            <div class="col-lg-3">4</div>
        </div>
    </div>
</body>

</html>
```

显示效果如下

![](image-20240711203419955.png)



#### 列偏移

使用 `offset` 类向右偏移列，例如

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <!--[if lt IE 9]-->
    <script src="" https://oss.maxcdn.com/html5shiv/3.7.3/html5shiv.min.js"></script>
    <script src="https://oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>
    <!--[endif]-->
    <!-- bootstrap 核心样式 -->
    <link rel="stylesheet" href="bootstrap/css/bootstrap.min.css">
    <title>Document</title>
    <style>
        [class^="col-lg"] {
            background-color: rgb(145, 255, 0);
            height: 50px;
        }
    </style>
</head>

<body>
    <div class="container">
        <div class="row">
            <div class="col-lg-3">左侧</div>
            <div class="col-lg-3 col-lg-offset-6">右侧</div>
        </div>
        <div class="row">
            <div class="col-lg-8 col-lg-offset-2">中间</div>
        </div>
    </div>
</body>

</html>
```

显示效果如下

![](image-20240711221638443.png)



#### 列偏移

使用 `push, pull` 推移列的位置，例如

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <!--[if lt IE 9]-->
    <script src="" https://oss.maxcdn.com/html5shiv/3.7.3/html5shiv.min.js"></script>
    <script src="https://oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>
    <!--[endif]-->
    <!-- bootstrap 核心样式 -->
    <link rel="stylesheet" href="bootstrap/css/bootstrap.min.css">
    <title>Document</title>
    <style>
        [class^="col-lg"] {
            background-color: rgb(145, 255, 0);
            height: 50px;
        }
    </style>
</head>

<body>
    <div class="container">
        <div class="row">
            <div class="col-lg-3 col-lg-push-8">左侧</div>
            <div class="col-lg-3 col-lg-pull-4">右侧</div>
        </div>
    </div>
</body>

</html>
```

显示效果如下

![](image-20240711221936020.png)



#### 响应式工具

Bootstrap 提供响应式工具类针对不同屏幕大小的设备隐藏页面内容

| 类名          | 超小屏 | 小屏  | 中屏  | 大屏  |
| :---------- | :-- | :-- | :-- | :-- |
| `hidden-xs` | ×   | √   | √   | √   |
| `hidden-sm` | √   | ×   | √   | √   |
| `hidden-md` | √   | √   | ×   | √   |
| `hidden-lg` | √   | √   | √   | ×   |

例如要求一个列**只在**中屏下隐藏

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <!--[if lt IE 9]-->
    <script src="" https://oss.maxcdn.com/html5shiv/3.7.3/html5shiv.min.js"></script>
    <script src="https://oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>
    <!--[endif]-->
    <!-- bootstrap 核心样式 -->
    <link rel="stylesheet" href="bootstrap/css/bootstrap.min.css">
    <title>Document</title>
    <style>
        [class^="col-lg"] {
            background-color: rgb(145, 255, 0);
            height: 50px;
        }
    </style>
</head>

<body>
    <div class="container">
        <div class="row">
            <div class="col-lg-3 hidden-md">中屏时隐藏</div>
        </div>
    </div>
</body>

</html>
```

类似的，使用 `visible-md` 实现只在中屏时显示。


