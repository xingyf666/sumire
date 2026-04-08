# WebGL

## 基础知识
### Canvas

首先创建默认的画布，默认宽高为 `300x150`，不能通过 css 修改，这样仅仅会缩放画布而不改变分辨率

```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <style>
        canvas {
            background-color: #ccc;
        }
    </style>
</head>
<body>
    <canvas class="canvas"></canvas>
    <script>
        const canvas = document.querySelector('.canvas');
        const ctx = canvas.getContext('2d');
    </script>
</body>
</html>
```

如果需要修改默认画布大小，需要在 `canvas` 标签内部修改

```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <style>
        canvas {
            background-color: #ccc;
        }
    </style>
</head>
<body>
	// 在这里修改尺寸
    <canvas class="canvas" width="600" height="600"></canvas>
    <script>
        const canvas = document.querySelector('.canvas');
        const ctx = canvas.getContext('2d');
		
		// 加载 Img 后使用 ctx 绘制
        const img = new Image();
        img.src = './00003.png';
        img.onload = function() {
            ctx.drawImage(img, 0, 0);
        };
    </script>
</body>
</html>
```



### 类型化数组

类型化数组用于操作二进制数据对象，例如图像、视频、音频数据。相较于普通数组，它在一个连续的内存块上操作，访问修改更为高效。普通数组由于其动态、可容纳不同类型数据，因此效率更低。例如

```js
// 普通数组
const arr = [1, 2, "ac", true, {name: "John"}];

// 类型化数组
const int8 = new Int8Array(8);		// 8 位 1 字节
const uint8 = new Uint8Array(8);	// 8 位 1 字节
```



