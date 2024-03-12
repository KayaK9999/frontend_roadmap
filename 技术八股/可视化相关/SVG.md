学习资源：
https://developer.mozilla.org/zh-CN/docs/Web/SVG
文档链接：
https://www.notion.so/SVG-Canvas-D3-ECharts-Vega-Lite-e7c3907646284044b73f816cfb3c93af

可缩放矢量图形（Scalable Vector Graphics，SVG）基于 XML 标记语言，用于描述二维的矢量图形
## 引言

SVG 提供了一些元素，用于定义圆形、矩形、简单或复杂的曲线。

一个简单的 SVG 文档由 `[<svg>](<https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element/svg>)` 根元素和基本的形状元素构成。另外还有一个`g`元素，它用来把若干个基本形状编成一个组。

SVG 支持渐变、旋转、动画、滤镜效果、与 JavaScript 交互等等功能

## 入门

### SVG 文件的基本属性

渲染顺序：SVG 文件全局有效的规则是“后来居上”，越后面的元素越可见

SVG 文件可以直接在浏览器上展示，或者通过以下方法嵌入到 HTML 文件中：

- 如果 HTML 是 XHTML 并且声明类型为 `application/xhtml+xml`，可以直接把 SVG 嵌入到 XML 源码中
- SVG 可以直接被嵌入到 HTML 中
- 可以使用 `img` 元素
- 可以通过 `object` 元素或`iframe`元素引用 SVG 文件：

```html
<object data="image.svg" type="image/svg+xml"></object>
<iframe src="image.svg"></iframe>
```

- SVG 可以通过 JavaScript 动态创建并注入到 HTML DOM 中

### SVG 文件类型

普通 SVG 文件是包含 SVG 标记的简单文本文件。推荐使用“.svg”

由于在某些应用（比如地图应用等）中使用时，SVG 文件可能会很大，SVG 标准同样允许 gzip 压缩的 SVG 文件。推荐使用“.svgz”

## 坐标定位

### 网格

对于所有元素，SVG 使用的坐标系统或者说网格系统：以页面的左上角为 (0,0) 坐标点，坐标以像素为单位，x 轴正方向是向右，y 轴正方向是向下

在 SVG 文档中的 1 个像素对应输出设备（比如显示屏）上的 1 个像素，

## 基本形状

### 矩形

```html
<!-- rx: 圆角的 x 方位的半径; ry: 圆角的 y 方位的半径 -->
<rect x="60" y="10" rx="10" ry="10" width="30" height="30"/>
```

### 圆形

```html
<circle cx="25" cy="75" r="20"/>
```

### 椭圆

```html
<ellipse cx="75" cy="75" rx="20" ry="5"/>
```

### 线条

```html
<line x1="10" x2="50" y1="110" y2="150" stroke="black" stroke-width="5"/>
```

### 折线

```html
<polyline points="60, 110 65, 120 70, 115 75, 130 80, 125 85, 140 90, 135 95, 150 100, 145"/>
```

两两一组，可以用空白、逗号、终止命令符或者换行符分隔开

### 多边形

```html
<polygon points="50, 160 55, 180 70, 180 60, 190 65, 205 50, 195 35, 205 40, 190 30, 180 45, 180"/>
```

`polygon`的路径在最后一个点处自动回到第一个点

## 路径

元素是SVG基本形状中最强大的，只需要设定很少的点，就可以创建平滑流畅的线条（比如曲线）

path 元素的形状是通过属性 `[d](<https://developer.mozilla.org/zh-CN/docs/Web/SVG/Attribute/d>)` 定义的，属性`d`的值是一个“命令 + 参数”的序列

每一个命令都用一个关键字母来表示，其中大写字母代表绝对定位，小写字母代表相对定位

### 直线命令

`M`：需要两个参数，把画笔移动到的点的 x 轴和 y 轴的坐标，但不画线

`L`：需要两个参数，分别是一个点的 x 轴和 y 轴坐标，L 命令将会在当前位置和新位置（L 前面画笔所在的点）之间画一条线段

`H`：需要一个参数，绘制水平线

`V`：需要一个参数，绘制垂直线

`Z`：不需要参数，闭合路径

### 曲线命令

绘制平滑曲线的命令有3个，2个画贝塞尔曲线：三次贝塞尔曲线 C，和二次贝塞尔曲线 Q，1个画弧形或者说是圆的一部分

C：需要设置三组坐标参数，创建三次贝塞尔曲线，前面两组是控制点，最后是曲线终点的坐标

```html
C x1 y1, x2 y2, x y
(or)
c dx1 dy1, dx2 dy2, dx dy
```

S：可以用来创建与前面一样的贝塞尔曲线，但是，如果 S 命令跟在一个 C 或 S 命令后面，则它的第一个控制点会被假设成前一个命令曲线的第二个控制点的中心对称点。如果 S 命令单独使用，前面没有 C 或 S 命令，那当前点将作为第一个控制点。

```html
S x2 y2, x y
(or)
s dx2 dy2, dx dy
```

```html
<svg width="190" height="160" xmlns="<http://www.w3.org/2000/svg>">
  <path d="M 10 80 C 40 10, 65 10, 95 80 S 150 150, 180 80" stroke="black" fill="transparent"/>
</svg>
```

Q：2组参数，创建2次贝塞尔曲线，只需要一组控制点

T：1组参数，类似S命令

A：创建弧形命，rx,ry类似于椭圆的设置，x-axis-rotation旋转角，large-arc-flag弧线角，0 表示小角度弧，1 表示大角度弧。sweep-flag 表示弧线的方向，0 表示从起点到终点沿逆时针画弧，1 表示从起点到终点沿顺时针画弧，x,y为弧线终点

```html
 A rx ry x-axis-rotation large-arc-flag sweep-flag x y
 a rx ry x-axis-rotation large-arc-flag sweep-flag dx dy
```

## 填充和边框

### Fill 和 Stroke 属性

`fill` 属性设置对象内部的颜色，`stroke` 属性设置绘制对象的线条的颜色

属性 `fill-opacity` 控制填充色的不透明度，属性 `stroke-opacity` 控制描边的不透明度

描边是以路径为中心线绘制的，与上面线条区分

`stroke-width` 属性定义了描边的宽度

`stroke-linecap`属性，它控制边框终点的形状，有3个可能值

- `butt`用直边结束线段，它是常规做法，线段边界 90 度垂直于描边的方向、贯穿它的终点。
- `square`的效果差不多，但是会稍微超出`实际路径`的范围，超出的大小由`stroke-width`控制。
- `round`表示边框的终点是圆角，圆角的半径也是由`stroke-width`控制的。

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/35893a6a-05a0-4c20-bf22-bad7fc395628/10a07ff3-f9db-4e2e-a78e-a1a80c5fca6c/Untitled.png)

`stroke-linejoin`属性，用来控制两条描边线段之间，用什么方式连接：它有三个可用的值，

- `miter`是默认值，表示用方形画笔在连接处形成尖角，
- `round`表示用圆角连接，实现平滑效果。
- 最后还有一个值`bevel`，连接处会形成一个斜接

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/35893a6a-05a0-4c20-bf22-bad7fc395628/5ecb5757-15c4-49d8-bf0e-be81b0f869cc/Untitled.png)

`stroke-dasharray`属性，将虚线类型应用在描边上。参数是一组用逗号分割的数字组成的数列，填色空白循序出现，如果是奇数，则循环两次

其他属性：

`fill-rule`，用于定义如何给图形重叠的区域上色

`stroke-miterlimit`，定义什么情况下绘制或不绘制边框连接的`miter`效果

`stroke-dashoffset`，定义虚线开始的位置

### 使用 CSS

除了定义对象的属性外，。语法和在 HTML 里使用 CSS 一样

只不过要把`background-color`、`border`改成`fill`和`stroke`可以通过 CSS 来样式化`填充`和`描边`

`[<defs>](<https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element/defs>)` 表示定义，这里面可以定义一些不会在 SVG 图形中出现、但是可以被其他元素使用的元素。


## 渐变

有两种类型的渐变：线性渐变和径向渐变。**必须**给渐变内容指定一个 id 属性，否则文档内的其他元素就不能引用它。

为了让渐变能被重复使用，渐变内容需要定义在‘defs’标签内部，而不是定义在形状上面。

### 线性渐变

线性渐变沿着直线改变颜色，要插入一个线性渐变，需要在 SVG 文件的`defs 元素`内部，创建一个`[<linearGradient>](<https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element/linearGradient>)` 节点。

```html
<defs>
  <linearGradient id="Gradient2" x1="0" x2="0" y1="0" y2="1">
    <stop offset="0%" stop-color="red" />
    <stop offset="50%" stop-color="black" stop-opacity="0" />
    <stop offset="100%" stop-color="blue" />
  </linearGradient>
</defs>    
```

x1、x2、y1 和 y2，这些属性定义了渐变路线走向

### 径向渐变

径向渐变与线性渐变相似，只是它是从一个点开始发散绘制渐变

```html
<defs>
    <radialGradient id="RadialGradient2" cx="0.25" cy="0.25" r="0.25">
      <stop offset="0%" stop-color="red" />
      <stop offset="100%" stop-color="blue" />
    </radialGradient>
  </defs>
```

它需要一个中心点，由 cx 和 cy 属性及半径 r 来定义，描述了渐变边缘位置

焦点，由 fx 和 fy 属性定义，描述了渐变的中心

`spreadMethod` 属性可以有三个值：pad、reflect 或 repeat

- Pad 即当渐变到达终点时，最终的偏移颜色被用于填充对象剩下的空间
- reflect 会让渐变一直持续下去，不过它的效果是与渐变本身是相反的，以 100% 偏移位置的颜色开始，逐渐偏移到 0% 位置的颜色，然后再回到 100% 偏移位置的颜色
- repeat 也会让渐变继续，但是它不会像 reflect 那样反向渐变，而是跳回到最初的颜色然后继续渐变

`gradientUnits（渐变单元）`的属性中，默认值为 `objectBoundingBox`，`userSpaceOnUse` 使用绝对单元，而不是自动缩放到对象相同大小

## Patterns

patterns 可以让里面元素一直重复，重复次数由width和height设置

```html
<pattern id="Pattern" x="0" y="0" width=".25" height=".25">
	<rect x="0" y="0" width="50" height="50" fill="skyblue" />
</pattern>
```

并且它也具有`gradientUnits（渐变单元）` 属性

## 文本

SVG 文本有两种不同的模式：写在图像中的文本、SVG 字体

```html
<text x="10" y="10">Hello World!</text>
```

start、middle、end 或 inherit，允许决定从这一点开始的文本流的方向。

属性`fill`可以给文本填充颜色，属性`stroke`可以给文本描边，形状元素和文本元素都可以引用渐变或图案

文本的一个至关重要的部分是它显示的字体。SVG 提供了一些属性，类似于它们的 CSS 同行，用来激活文本选区。下列每个属性可以被设置为一个 SVG 属性或者成为一个 CSS 声明：`font-family`、`font-style`、`font-weight`、`font-variant`、`font-stretch`、`font-size`、`font-size-adjust`、`kerning`、`letter-spacing`、`word-spacing`和`text-decoration`。

### 其他文本相关的元素

`tspan` 该元素用来标记大块文本的子部分，它必须是一个`text`元素或别的`tspan`元素的子元素。一个典型的用法是把句子中的一个词变成粗体红色。

具有以下的自定义属性：

**x** 为容器设置一个新绝对`x`坐标。它覆盖了默认的当前的文本位置。这个属性可以包含一个数列，它们将一个一个地应用到`tspan`元素内的每一个字符上。

**dx** 从当前位置，用一个水平偏移开始绘制文本。这里，你可以提供一个值数列，可以应用到连续的字体，因此每次累积一个偏移。

此外还有属性**y**和属性**dy**作垂直转换。

**rotate** 把所有的字符旋转一个角度。如果是一个数列，则使每个字符旋转分别旋转到那个值，剩下的字符根据最后一个值旋转。

**textLength** 这是一个很模糊的属性，给出字符串的计算长度。它意味着如果它自己的度量文字和长度不满足这个提供的值，则允许渲染引擎精细调整字型的位置。

`tref` 元素允许引用已经定义的文本，高效地把它复制到当前位置。你可以使用`xlink:href`属性，把它指向一个元素，取得其文本内容。你可以独立于源样式化它、修改它的外观。

textPath 该元素利用它的`xlink:href`属性取得一个任意路径，把字符对齐到路径，于是字体会环绕路径、顺着路径走：

## 基础变形

‘g' 元素`g`是用来组合对象的容器。添加到`g`元素上的变换会应用到其所有的子元素上。添加到`g`元素的属性会被其所有的子元素继承。此外，`g`元素也可以用来定义复杂的对象，之后可以通过`[<use>](<https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element/use>)`元素来引用它们。

### 平移

```html
<rect x="0" y="0" width="10" height="10" transform="translate(30,40)" />
```

### 旋转

```html
<rect x="20" y="20" width="20" height="20" transform="rotate(45)" />
```

### 斜切

利用一个矩形制作一个斜菱形。可用`skewX()`变形和`skewY()`变形。每个需要一角度以确定元素斜切到多远。

### 缩放

`scale() 变形`改变了元素的尺寸。它需要两个数字，作为比率计算如何缩放。0.5 表示收缩到 50%。_如果第二个数字被忽略了，它默认等于第一个值。_

### 使用 matrix() 实现复杂变形

可以直接用 `matrix(a, b, c, d, e, f)` 变形设置结果矩阵：

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/35893a6a-05a0-4c20-bf22-bad7fc395628/894193a2-28cd-4142-bc97-54f4e8152d91/Untitled.png)

### SVG 嵌在 SVG 内部

SVG 允许你无缝嵌入别的 svg 元素。可以利用内部`svg`元素的属性`viewBox`、属性`width`和属性`height`简单创建一个新的坐标系统

## 剪切和遮罩

擦除已经创建的元素的部分内容，可以使用以下属性：

- **Clipping** 用来移除在别处定义的元素的部分内容。在这里，任何半透明效果都是不行的。它只能要么显示要么不显示。
- **Masking** 允许使用透明度和灰度值遮罩计算得的软边缘。

### 创建剪切

```html
<svg>
  <defs>
    <clipPath id="cut-off-bottom">
      <rect x="0" y="0" width="200" height="100" />
    </clipPath>
  </defs>

  <circle cx="100" cy="100" r="100" clip-path="url(#cut-off-bottom)" />
</svg>
```

属性`clip-path`引用了一个带单个 rect 元素的`[<clipPath>](<https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element/clipPath>)`元素。它内部的这个矩形将把画布的上半部分涂黑。

### 遮罩

```html
<svg>
  <defs>
    <linearGradient id="Gradient">
      <stop offset="0" stop-color="white" stop-opacity="0" />
      <stop offset="1" stop-color="white" stop-opacity="1" />
    </linearGradient>
    <mask id="Mask">
      <rect x="0" y="0" width="200" height="200" fill="url(#Gradient)" />
    </mask>
  </defs>

  <rect x="0" y="0" width="200" height="200" fill="green" />
  <rect x="0" y="0" width="200" height="200" fill="red" mask="url(#Mask)" />
</svg>
```

一个绿色填充的矩形在底层，一个红色填充的矩形在上层。

后者有一个`mask`属性指向一个`mask`元素。`mask`元素的内容是一个单一的`rect`元素，它填充了一个透明到白色的渐变。

作为红色矩形继承`mark`内容的`alpha`值（透明度）的结果

### 用 opacity 定义透明度

描边将绘制在填充的上面。

如果你在一个元素上设置了描边透明度，但它同时设有填充，则描边的一半应用填充色，另一半将应用背景色。

## 其他 SVG 内容

SVG 有一个`image`元素，支持嵌入任意光栅（以及矢量）图像。它的规格要求应用至少支持 PNG、JPG 和 SVG 格式文件。

可以在 SVG 文档的任何位置嵌入任意 XML，`<foreignObject>` 元素。它的唯一的目的是作为其他标记的容器和 SVG 样式属性的载体

`foreignObject`元素的内容是能被浏览器加工的。一个独立的 SVG 浏览器不太可能呈现 HTML 或 MathML。

## [滤镜效果](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Tutorial/Filter_effects)：没看懂

滤镜（Filter）是 SVG 中用于创建复杂效果的一种机制。

滤镜通过 `[<filter>](<https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element/filter>)` 元素进行定义，并且置于 `<defs>` 区块中。

### 步骤1

```html
<feGaussianBlur in="SourceAlpha" stdDeviation="4" result="blur" />
```

设置 `[<feGaussianBlur>](<https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element/feGaussianBlur>)` 中的 `in` 属性为 `"SourceAlpha"` 值，即原图像的 alpha 通道，并设置了模糊度为 4 以及把 `result` 保存在了一个名为 "blur" 的临时缓冲区中。

### 步骤2

```html
<feOffset in="blur" dx="4" dy="4" result="offsetBlur" />
```

`[<feOffset>](<https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element/feOffset>)` 设置 `in` 的值为 "blur"，即我们前面保存 `result` 的那个临时缓冲区。然后设置相对坐标，向右偏移 4，向下偏移 4。最后把结果 `result` 保存到名为 "offsetBlur" 的缓冲区中。步骤 1、2 其实是创建图形阴影的两个图元。

### 步骤3

```html
<feSpecularLighting
  in="offsetBlur"
  surfaceScale="5"
  specularConstant=".75"
  specularExponent="20"
  lighting-color="#bbbbbb"
  result="specOut">
  <fePointLight x="-5000" y="-10000" z="20000" />
</feSpecularLighting>
```

`[<feSpecularLighting>](<https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element/feSpecularLighting>)` 设置输入源 `in` 为 "offsetBlur"，将会生成一个光照效果，并将结果 `result` 保存在 "specOut" 中。

### 步骤4

```html
<feComposite in="specOut" in2="SourceAlpha" operator="in" result="specOut" />
```

第一个 `[<feComposite>](<https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element/feComposite>)` 元素设置输入源为 "specOut"，第二个输入源（`in2`）为 "SourceAlpha"，将 "specOut" 的结果效果遮盖掉，以致于最后的结果不会大于 "SourceAlpha"（源图像），最后覆盖输出结果 `result` 为 "specOut"。

### 步骤5

```html
<feComposite in="SourceGraphic" in2="specOut"
             operator="arithmetic"
             k1="0" k2="1" k3="1" k4="0"
             result="litPaint"/>
```

第二个 `[<feComposite>](<https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element/feComposite>)` 设置 `in` 为 "SourceGraphic" 和 "specOut"，即在 "SourceGraphic" 之上添加 "specOut" 的效果，复合模式为 "arithmetic"，然后保存结果为 "litPaint"。

### 步骤6

```html
<feMerge>
  <feMergeNode in="offsetBlur"/>
  <feMergeNode in="litPaint"/>
</feMerge>
```

最后，`[<feMerge>](<https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element/feMerge>)` 元素合并了阴影效果 "offsetBlur" 和源图像的光照效果 "litPaint"。

## SVG 字体

SVG 字体当前只在 Safari 和 Android 浏览器中受支持。Chrome移除，FireFox 无限期推迟

## SVG image元素

- 如果你没有设置 x 属性或 y 属性，它们自动被设置为 0。
- 如果你没有设置 height 属性或 width 属性，它们自动被设置为 0。
- 如果 width 属性或 height 等于 0，将不会呈现这个图像。

## SVG 工具

可以制作和呈现 SVG 文件的工具

Inkscape~URL：[www.inkscape.org](https://inkscape.org/)

Adobe Illustratoir~URL：[www.adobe.com/products/illustrator/](https://www.adobe.com/products/illustrator.html)

Apache Batik~URL：[xmlgraphics.apache.org/batik/](https://xmlgraphics.apache.org/batik/)

Snap.svg~URL: [snapsvg.io](http://snapsvg.io/)

Google Docs~URL：[www.google.com/google-d-s/drawings/](https://docs.google.com/drawings)

Science

## SVG 和 CSS

- 这个 SVG 文档使用常见连接方法引入样式表。
- SVG 有自己一套 CSS 属性和对应的值。其中一些和 HTML 使用的 CSS 属性相似。

## 内容类型

[https://developer.mozilla.org/zh-CN/docs/Web/SVG/Content_type](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Content_type)

一些元素值可以多种表示方式

## 使用 SMIL 的 SVG 动画

SMIL 允许：

- 变动一个元素的数字属性（x、y……）
- 变动变形属性（translation 或 rotation）
- 变动颜色属性
- 物件方向与运动路径方向同步

通过<animate>元素，属性有：**`attributeName`**变动属性的属性名。`from`变动的初始值。`to`结束值。`dur`动画的持续时间（例如，写“5s”代表 5 秒钟）。

`<animateTransform>` 元素用于变动 **transform** 属性

`[<animateMotion>](<https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element/animateMotion>)` 元素使一个元素的位置动起来，并顺着路径同步旋转。

- 定义这个路径是与在 `[<path>](<https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element/path>)` 元素中定义路径的方法相同。
- 把 **repeatCount** 属性的值设置为 `indefinite`，我们指明了反复循环的动画，只要 SVG 图像还存在就会一直循环下去。
- `rotate` 属性定义对象是否与跟着路径的正切值旋转。

