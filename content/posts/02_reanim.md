---
title: "Reanim 动画文件详解"
date: '2025-04-29T10:00:00+08:00'
# weight: 1
# aliases: ["/first"]
tags: [文件, 动画]
author: "XuFly"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "对于原版游戏使用的 Reanim 动画文件的详尽解析（大概），以及对应 Godot 动画的转换"
# canonicalURL: "https://canonical.url/to/page"
disableShare: true
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: false
ShowWordCount: false
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
# editPost:
#     URL: "https://github.com/<path_to_repo>/content"
#     Text: "Suggest Changes" # edit text
#     appendFilePath: true # to append file path to Edit link
---

本文需要一定数学基础  
本文相关项目: https://github.com/mcxufly/neopvzhelper

---

## 前言

Reanim 动画文件是由 Adobe Flash 动画导出的，所以其实可以找到原版游戏的`.fla`动画文件（我确实找到了）然后使用 Adobe Animate 软件打开然后做编辑等操作。但是 Adobe Animate 是一个商业软件而且是 Windows 软件，同时`.fla`动画文件标准目前为止也尚未公开，而我使用的操作系统是 Arch Linux，所以我决定不使用 Adobe Animate 软件在 Godot 中使用原版游戏提取的Reanim 动画文件数据重置原版游戏的动画。

## 提取原版 Reanim 动画文件

使用任意原版游戏 PAK 文件解包工具解包`main.pak`文件，`compiled/reanim`目录下是压缩过的`reanim.compiled`文件，需要解压缩，可以使用工具转换，如 [PopStudio](https://github.com/YingFengTingYu/PopStudio_Old)。  

### `reanim.compiled`文件

根据 PopStudio 项目的 [YFTYLib/Reanim/PC.cs](https://github.com/YingFengTingYu/PopStudio_Old/blob/UsingMoonSharp/PopStudio.Shared/YFTYLib/Reanim/PC.cs)，`reanim.compiled`文件有 32 位文件头`DEADFED4(-559022380)`，后 32 位`int`有符号整形树为原二进制数据字节数，后为 ZLib 压缩数据标志头`9C78`，后为压缩数据，解压后获得原二进制数据流。reanim 数据具体解析参见 [LoadFromCompiled](https://github.com/mcxufly/neopvzhelper/blob/main/ReanimHelper/Utils.cs#L8)，在此不做过多赘述。  

## Reanim 动画数据

解压解析后的 Reanim 动画数据具有以下结构（以 JSON 格式为例）  
```json
{
  "fps": 12,
  "tracks": [
    // track
    {
      "name": "track_name",
      "transforms": [
        // transform
        {
          "f": -1,
          "i": "IMAGE_",
          "a": 1,
          "x": 0,
          "y": 0,
          "sx": 1,
          "sy": 1,
          "kx": 0,
          "ky": 0,
          "font": "FONT",
          "text": "TEXT"
        },
        ...
      ]
    },
    ...
  ]
}
```

1. fps: 不用多说是动画帧率  
2. tracks: 轨道列表  
	**track: 轨道**
	1. name: 轨道名称，anim 开头为控制轨道，其他为普通轨道  
	2. transforms: 帧数据/变换数据列表  
		**transform: 帧数据/变换数据**
		1. f: 帧类型，-1 为空白帧，0为关键帧  
		2. i: 显示的图片  
		3. a: 透明度  
		4. x, y: XY 坐标，原点位于左上角  
		5. sx, sy: XY 缩放  
		6. kx, ky: XY 旋转  
		7. font: 字体  
		8. text: 文字  

需要注意的是，如果某一帧的某一参数和上一帧的一样那么就不会重复存储相同的数据。例如：
```json
[
  {
    "X": 18.9,
    "Y": 15.6,
    "XScale": 0.572,
    "YScale": 0.521
  },
  {
    "X": 19.3,
    "Y": 13.7,
    "XScale": 0.555,
    "YScale": 0.555
  },
  {
    // "X": 19.3,
    "Y": 13.2,
    // "XScale": 0.555,
    "YScale": 0.565
  },
  {
    // "X": 19.3,
    "Y": 12.7,
    // "XScale": 0.555,
    "YScale": 0.575
  }
]
```

## 动画转换

Godot 引擎中 Node2D 包含 4 个变换: Position, Rotation, Scale, Skew  
Position 是位移对应 x, y，Scale 是缩放对应 sx, sy，Rotation 是旋转，Skew 是倾斜 Godot 引擎给出的解释是“如果设置为非零值，则会向某一方向倾斜”。

Rotation/旋转变换的矩阵是：
$$
\begin{pmatrix}
	cos(\theta) & -sin(\theta) \newline
	sin(\theta) & cos(\theta)
\end{pmatrix}
$$
Skew/半旋转的矩阵是：
$$
\begin{pmatrix}
	1 & -sin(-\alpha) \newline
	0 & cos(-\alpha)
\end{pmatrix}
\=
\begin{pmatrix}
	1 & sin(\alpha) \newline
	0 & cos(\alpha)
\end{pmatrix}
$$
可以根据 [Transform2D::set_rotation_scale_and_skew](https://github.com/godotengine/godot/blob/1cf573f44de045ee73fd938fbf2bdccc0e31b7bb/core/math/transform_2d.h#L243) 确认正确

Reanim 中的 kx, ky: XY 旋转变换矩阵是：
$$
\begin{pmatrix}
	cos(kx) & -sin(ky) \newline
	sin(kx) & cos(ky)
\end{pmatrix}
$$
那么 Godot 引擎中 Rotation 和 Skew 的组合可以等价于 Reanim 中的 kx, ky: XY 旋转变换：
$$
\begin{pmatrix}
	cos(\theta) & -sin(\theta) \newline
	sin(\theta) & cos(\theta)
\end{pmatrix}
\begin{pmatrix}
	1 & sin(\alpha) \newline
	0 & cos(\alpha)
\end{pmatrix}
\=
\begin{pmatrix}
	cos(kx) & -sin(ky) \newline
	sin(kx) & cos(ky)
\end{pmatrix}
$$

容易得出：
$$
\theta = kx \newline
\alpha = kx - ky
$$

所以两种动画可以相互转换，转换关系如下：  
$$
\begin{align*}
\text{Position} & = \text{Vector2}(x, y) \newline
\text{Rotation} & = kx \newline
\text{Scale} & = \text{Vector2}(sx, sy) \newline
\text{Skew} & = kx -ky \newline
\end{align*}
$$
要是自己一个一个输入这些动画帧也太多了，不过好在 Godot 的动画轨道在`.tscn`中可以直接编辑，下面是一个动画轨道示例
```
[sub_resource type="Animation" id="Animation_wtcfe"]
resource_name = "test"
step = 0.1
tracks/0/type = "value"
tracks/0/imported = false
tracks/0/enabled = true
tracks/0/path = NodePath("Sprite0:skew")
tracks/0/interp = 1
tracks/0/loop_wrap = true
tracks/0/keys = {
"times": PackedFloat32Array(0),
"transitions": PackedFloat32Array(1),
"update": 0,
"values": [0.0]
}
```
这里需要注意的是 path 和 keys 里的 times, transitions, values。path 是与这个轨道相关的属性节点它决定了 values 列表的值，times 是时间序列它决定了 values 值对应的时间位置，transitions 是缓动系数这里应该为每一帧设置为 1，values 是值序列值的类型取决于 path ，对于 Rotation 和 Skew 是浮点数单位是弧度，对于 Position 和 Rotation 是 Vector2。所以只需要把对应的值填好然后重新加载就可以修改动画了。  
不过四个轨道还是有点多了，其实还可以使用 Transform2D 节点，这一个节点就可以代替 Position, Rotation, Scale, Skew 4 个节点。Transform2D 是代表 2D 变换的 2×3 矩阵，其实是仿射变换矩阵去掉第三行：
$$
\begin{pmatrix}
1 & 0 & t_{x} \newline
0 & 1 & t_{y} \newline
0 & 0 & 1
\end{pmatrix}
\rightarrow
\begin{pmatrix}
1 & 0 & t_{x} \newline
0 & 1 & t_{y} \newline
\end{pmatrix}
$$
具体应该是：
$$
\begin{pmatrix}
	s_{x} \cdot cos(kx) & -s_{y} \cdot sin(ky) & x \newline
	x_{x} \cdot sin(kx) & s_{y} \cdot cos(ky) & y
\end{pmatrix}
或
\begin{pmatrix}
	s_{x} \cdot cos(\theta) & s_{y} \cdot sin(\alpha - \theta) & x \newline
	x_{x} \cdot sin(\theta) & s_{y} \cdot cos(\theta - \alpha) & y
\end{pmatrix}
$$
第一个矩阵对应 Reanim 动画参数，第二个对应 Godot Node2D Transform 的参数，$s_{x,y},\theta,\alpha$分别是缩放、Rotation 角度和 Skew 角度。
在 values 中表示为：  
Transform2D(1, 0, 0, 1, 0, 0)

以上再结合亿点点辅助的代码和亿点点手动调整就可以实现 Reanim 动画到 Godot 动画的转换了，希望对您有所帮助。