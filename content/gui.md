## GUI

按照目前作者的想法是:基于openGL,glad,glfw来创建一个比较轻量的GUI

vui库已经发布,代码库:https://github.com/vlang/ui

### 安装

使用前先安装依赖:

```
macOS:
brew install glfw freetype openssl

Debian/Ubuntu:
sudo apt install libglfw3 libglfw3-dev libfreetype6-dev libssl-dev

Arch/Manjaro:
sudo pacman -S glfw-x11 freetype2

Fedora:
sudo dnf install glfw glfw-devel freetype-devel

Windows:
git clone --depth=1 https://github.com/ubawurinna/freetype-windows-binaries [path to v repo]/thirdparty/freetype/
```

目前还是早期版本

安装方式1:直接从git代码库下载代码,链接到~/.vmodules/ui

```
git clone https://github.com/vlang/ui
ln -s /path/to/ui ~/.vmodules/ui
```

安装方式2:使用vpm安装

```
v install ui
```



### vui涉及到的标准模块

#### gx模块

源代码位置:vlib/gx

主要维护了颜色,文本,字体相关的基础结构体及常量

Color结构体

标准颜色的常量值

Image结构体

TextCfg结构体

#### gl模块

源代码位置:vlib/gl

glad的缩写,GLAD是继GL3W，GLEW之后，当前最新的用来访问OpenGL规范接口的第三方库

OpenGL loading libraries

官方网址为https://glad.dav1d.de/

源代码:https://github.com/Dav1dde/glad

gl模块就是glad库主要的C函数简单封装后,成为V函数

#### glm模块

源代码位置:vlib/glm

OpenGL Mathematics（GLM） - 几何数学库

Math Libraries

OpenGl中在进行图形变换的时候需要使用几何数学库

矩阵变换，四元数，数据打包，随机数，噪声等等

代码:https://github.com/g-truc/glm

#### freetype模块

源代码位置:vlib/freetype

FreeType是一个完全开源的、可扩展、可定制且可移植的字体引擎，它提供TrueType字体驱动的实现统一的接口来访问多种字体格式文件

官网:https://www.freetype.org/

#### stbi模块

源代码位置:vlib/stbi

图像库

源代码:https://github.com/nothings/stb

#### glfm模块

源代码位置:vlib/glfm

GLFW官网链接:https://www.glfw.org/

源代码:https://github.com/glfw/glfw

**GLFW** 是一个专门针对 OpenGL 的 C 语言库，它允许用户创建 OpenGL 上下文并显示窗口，它提供了一些渲染物体所需的最低限度的接口。其用来代替之前的 GLUT 库

Context/Window Toolkits

GLFW使用感受：轻量，简单，好使

glfm模块基本是对C函数进行对应的V函数封装,提供给ui模块使用

#### gg模块

源代码位置:vlib/gg

V 2D/3D graphics library with an OpenGL backend (DirectX, Vulkan, Metal coming soon)

V绘图模块,用来在窗体上绘制各种图形

gg是glad和glfw的缩写

#### 资料参考

OpenGL**（英语：*Open Graphics Library*，译名：**开放图形库**或者“开放式图形库”）是用于[渲染](https://baike.baidu.com/item/渲染)[2D](https://baike.baidu.com/item/2D)、[3D](https://baike.baidu.com/item/3D)[矢量图形](https://baike.baidu.com/item/矢量图形)的跨[语言](https://baike.baidu.com/item/语言)、[跨平台](https://baike.baidu.com/item/跨平台)的[应用程序编程接口](https://baike.baidu.com/item/应用程序编程接口)（API）。这个接口由近350个不同的函数调用组成，用来绘制从简单的图形比特到复杂的三维景象。而另一种程序接口系统是仅用于[Microsoft Windows](https://baike.baidu.com/item/Microsoft Windows)上的[Direct3D](https://baike.baidu.com/item/Direct3D)。OpenGL常用于[CAD](https://baike.baidu.com/item/CAD)、[虚拟现实](https://baike.baidu.com/item/虚拟现实)、科学可视化程序和电子游戏开发。

OpenGL的高效实现（利用了图形加速硬件）存在于[Windows](https://baike.baidu.com/item/Windows)，部分[UNIX](https://baike.baidu.com/item/UNIX)平台和[Mac OS](https://baike.baidu.com/item/Mac OS)。这些实现一般由显示设备厂商提供，而且非常依赖于该厂商提供的硬件

OpenGL是Khronos Group开发维护的一个规范，它主要为我们定义了用来操作图形和图片的一系列函数的API，需要注意的是OpenGL本身并非API。
GPU的硬件开发商则需要提供满足OpenGL规范的实现，这些实现通常被称为“驱动”，它们负责将OpenGL定义的API命令翻译为GPU指令。

OpenGL并非一个能够直接安装的库或包，它只是一个规范。我们只需要安装显卡的驱动即可，因为显卡驱动中就包括了对OpenGL规范的实现

由于 OpenGL 只是一个标准/规范，具体的实现是由驱动开发商针对特定显卡实现的。而 OpenGL 驱动版本众多，它大多数函数的位置都无法在编译时确定下来，需要在运行时查询。所以任务就落在了开发者身上，开发者需要在运行时获取函数地址并将其保存在一个函数指针中供以后使用。但这样写出的代码复杂繁琐，因此我们需要 GLAD,GLAD 是目前最流行的开源库，能帮我们简化这个流程

### 组件

刚发布的版本,目前的组件还比较少

除了最基本的window组件是基于glfw window外,其他组件都是自行绘制的,可以在所有组件代码的draw()函数中看到自行绘制的代码

基本的思路是:使用glfw的window,context,event,然后在窗体上自行绘制所有组件:

以window组件为例,显示通用的组件创建过程

1. 定义window结构体
2. 通过new_window(cfg WindowConfig) window 创建窗体,其中WindowConfig是配置类
3. 在每一个组件中定义draw()函数,包含绘制组件代码
4. 把每一个窗体内的组件都添加到window的children[]中
5. 使用window的context,也就是UI结构体的实例来进行绘制
6. 最后调用ur.run(window ui.Window)函数,进入for{}无限循环,然后循环调用window中所有children[]中每一个组件的draw()函数,渲染组件,最后调用window.ctx.gg.render()函数,完成渲染,并监听事件

因为都是采用自行绘制的,所以才有可能同一套UI代码,除了win,linux,mac外,以后也可以自行绘制成js前端组件,wasm组件,才有可能自行完全控制,实现响应式UI,监控代码修改,然后实时更新UI,类似swiftUI

也因为是采用自行绘制的,组件和组件的各种属性,方法,事件都要基于glfm重新定义

#### UI结构体

UI结构体主要包含了:绘制图形的gg,绘制文字的ft,系统剪贴板clipboard

window中的context就是UI结构体的实例,用来进行绘制图形,绘制文字,处理剪贴板

一般来说全局只有:1个glfm.window实例,1个ui.window实例,1个ui.UI实例即context

干嘛不把UI结构体,直接命名为Context,更清楚一些

#### Widget接口

所有的组件都实现了该接口

#### Window窗体



#### Canvas画布



#### Label标签



#### Button按钮



#### Textbox文本框



#### Checkbox复选框



#### Radio单选框



#### Progressbar进度条



#### Picture图像



#### Menu菜单





