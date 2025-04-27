---
title: "原版游戏的音乐音效"
date: '2025-04-27T07:00:00+08:00'
# weight: 1
# aliases: ["/first"]
tags: [file, sound]
author: "XuFly"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "关于原版游戏的音乐音效的提取和 mo3 文件的转换"
# canonicalURL: "https://canonical.url/to/page"
disableShare: true
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
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

### 提取游戏音频文件

使用任意原版游戏 PAK 文件解包工具解包`main.pak`文件，在`sounds`目录下是游戏所有的音频文件，包括所有音效和音乐。  
以`1.2.0.1065 EN`版本为例，`sounds`目录下共有 170 个文件，绝大多数是 Ogg 音频文件，还有几个不太一样：  
`diamond.au` `tapglass.au` `mainmusic.mo3` `mainmusic_hihats.mo3`  
<!-- `Ogg`音频文件相信大家已经比较熟悉了，那么`.au` `.mo3`文件将会在下文 -->

### ULAW (Sun) `.au` 音频文件

维基百科关于 Au 音频文件的页面: https://en.wikipedia.org/wiki/Au_file_format  
简单来说 Au 音频文件就是一种比较古早简单的音频文件，直接使用 FFmpeg 转换成 Ogg 文件就可以了。  
不知道宝开出于什么原因只在两个音频使用了这种格式。  

### MO3 `.mo3` 音频文件

维基百科关于 MO3 文件的页面: https://en.wikipedia.org/wiki/MO3  
简单来说 MO3 文件是一种模块(Module)文件，可以类比 midi 文件，包含了音色、轨道等信息，体积极小。例如`mainmusic.mo3`仅 2.1MB 但是却包含了游戏中所有的音乐，除了 ZombiesOnYourLawn 猜测是因为这首歌包含人声演唱部分，可以想到宝开应该是在原版游戏集成了一个 MO3 文件播放器，根据我的观察这项功能很可能与原版游戏的`bass.dll`文件有关。所以替换音效非常容易但是替换背景音乐就非常困难了。  
但是我在复刻开发过程中不想再实现或者给游戏添加 MO3 文件播放功能，通过维基百科关于 MO3 文件的页面可知 [OpenMPT](https://openmpt.org/) 可以打开和编辑（但是不能保存）MO3 文件，使用 OpenMPT 可以导出轨道到 Ogg 文件，这样就可以将背景音乐全部导出为 Ogg 音频文件，同时 OpenMPT 也能查看 bpm 和循环开始位置，这样就可以在 Godot 引擎中导入音频时设置循环播放了。所有音乐文件的大小可比原 MO3 文件大了不止几百倍……不过这在存储空间相当充裕的现在应该不算什么问题吧。  
![openmpt](https://github.com/mcxufly/neopvzblog/blob/master/assets/images/01_openmpt.png)

以上就是关于原版游戏的音乐音效的相关信息，希望对你有所帮助。
