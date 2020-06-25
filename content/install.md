## 安装

### 源代码安装

目前V语言还处在快速开发的不稳定阶段,首选源代码安装方式.

#### 编译准备

目前V语言的编译需要依赖C编译器:gcc或clang

如果没有C编译器,可以参考以下文档,进行安装:

[windows下安装C编译器](https://github.com/vlang/v/wiki/Installing-a-C-compiler-on-Windows)

[linux/macOS下安装C编译器](https://github.com/vlang/v/wiki/Installing-a-C-compiler-on-Linux-and-macOS)

#### 下载源码/编译

  ```shell
git clone https://github.com/vlang/v
cd v	
make
  ```

编译成功后,会在当前目录生成V编译器的可执行文件,大小1M左右,小巧得很.

可以使用v version查看当前的版本

编译器命令行的使用参考:[编译器命令行使用章节](toolchain.md)

#### 可选安装

如果需要编译vui相关模块,需要安装freetype

如果需要编译http相关模块,需要安装openssl

```
macOS:
brew install freetype openssl

Debian/Ubuntu:
sudo apt install libfreetype6-dev libssl-dev

Arch/Manjaro:
sudo pacman -S freetype2

Fedora:
sudo dnf install freetype-devel
```

#### 后续升级

方式一:

  ```c
v up //抓取github上V代码库的主干代码,然后自动重新编译
  ```

方式二:

  ```shell
git pull
make
  ```



### 下载预编译安装包

在[官网](https://vlang.io/)直接下载对应平台的安装包

这个目前不推荐使用,更新太慢

