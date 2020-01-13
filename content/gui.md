## GUI

按照目前作者的想法是:基于openGL,glfw来创建一个比较轻量的GUI

vui库已经发布,代码库:https://github.com/vlang/ui

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

目前还是早期版本,安装步骤:

```
git clone https://github.com/vlang/ui
ln -s /path/to/ui ~/.vmodules/ui
```







GUI背景资料参考:

GLFW官网链接:https://www.glfw.org/

OpenGL**（英语：*Open Graphics Library*，译名：**开放图形库**或者“开放式图形库”）是用于[渲染](https://baike.baidu.com/item/渲染)[2D](https://baike.baidu.com/item/2D)、[3D](https://baike.baidu.com/item/3D)[矢量图形](https://baike.baidu.com/item/矢量图形)的跨[语言](https://baike.baidu.com/item/语言)、[跨平台](https://baike.baidu.com/item/跨平台)的[应用程序编程接口](https://baike.baidu.com/item/应用程序编程接口)（API）。这个接口由近350个不同的函数调用组成，用来绘制从简单的图形比特到复杂的三维景象。而另一种程序接口系统是仅用于[Microsoft Windows](https://baike.baidu.com/item/Microsoft Windows)上的[Direct3D](https://baike.baidu.com/item/Direct3D)。OpenGL常用于[CAD](https://baike.baidu.com/item/CAD)、[虚拟现实](https://baike.baidu.com/item/虚拟现实)、科学可视化程序和电子游戏开发。

OpenGL的高效实现（利用了图形加速硬件）存在于[Windows](https://baike.baidu.com/item/Windows)，部分[UNIX](https://baike.baidu.com/item/UNIX)平台和[Mac OS](https://baike.baidu.com/item/Mac OS)。这些实现一般由显示设备厂商提供，而且非常依赖于该厂商提供的硬件

OpenGL是Khronos Group开发维护的一个规范，它主要为我们定义了用来操作图形和图片的一系列函数的API，需要注意的是OpenGL本身并非API。
GPU的硬件开发商则需要提供满足OpenGL规范的实现，这些实现通常被称为“驱动”，它们负责将OpenGL定义的API命令翻译为GPU指令。

OpenGL并非一个能够直接安装的库或包，它只是一个规范。我们只需要安装显卡的驱动即可，因为显卡驱动中就包括了对OpenGL规范的实现

**GLFW** 是一个专门针对 OpenGL 的 C 语言库，它允许用户创建 OpenGL 上下文并显示窗口，它提供了一些渲染物体所需的最低限度的接口。其用来代替之前的 GLUT 库

GLFW使用感受：轻量，简单，好使

由于 OpenGL 只是一个标准/规范，具体的实现是由驱动开发商针对特定显卡实现的。而 OpenGL 驱动版本众多，它大多数函数的位置都无法在编译时确定下来，需要在运行时查询。所以任务就落在了开发者身上，开发者需要在运行时获取函数地址并将其保存在一个函数指针中供以后使用。但这样写出的代码复杂繁琐，因此我们需要 GLAD,GLAD 是目前最流行的开源库，能帮我们简化这个流程

