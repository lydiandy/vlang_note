## 安装

### 源代码安装

目前V语言还处在快速开发的不稳定阶段,首选使用源代码安装方式.

#### 编译准备

目前V语言的编译需要依赖C编译器:gcc或clang,

在命令行执行gcc -v 看看是否已经安装了编译器

#### 下载源码/编译

  ```shell
git clone https://github.com/vlang/v
cd v	
make
  ```

编译成功后,会在当前目录生成v可执行文件

可以使用v version查看当前的v版本

#### 后续升级

方式一:

  ```c
v up //该命令会更新github上的主干代码,然后自动重新编译
  ```

方式二:

  ```shell
git pull
make
  ```



### 下载预编译安装包

在[官网](https://vlang.io/)直接下载对应平台的安装包

这个目前不推荐使用,更新太慢

