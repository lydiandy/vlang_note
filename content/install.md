## 安装

### 源代码安装

目前V语言还处在快速开发的不稳定阶段，首选源代码安装方式。

#### 编译准备

目前V语言的编译需要依赖C编译器：gcc或clang，如果没有C编译器，可以参考以下文档，进行安装：

[windows下安装C编译器](https://github.com/vlang/v/wiki/Installing-a-C-compiler-on-Windows)

[linux/macOS下安装C编译器](https://github.com/vlang/v/wiki/Installing-a-C-compiler-on-Linux-and-macOS)

#### 下载源码/编译

  ```shell
git clone https://github.com/vlang/v
cd v	
make
  ```

编译成功后，会在当前目录生成V编译器的可执行文件，大小3M左右，小巧得很。

查看V编译器的版本：

```shell
v -v
v version
v -version
```

#### 运行代码

编译成功后，可以尝试运行代码：

```v
//main.v
module main

fn main() {
	println("hello V")
}
```

在终端中执行，输出hello V，则安装成功。

```shell
v run main.v
```

编译器命令行的使用参考：[编译器命令行使用章节](toolchain.md)。

### 安装可选依赖

#### openssl

如果需要执行v install安装模块或编译http相关模块，需要安装openssl：

```shell
macOS:
brew install openssl

Debian/Ubuntu:
sudo apt install libssl-dev

Arch/Manjaro:
openssl is installed by default

Fedora:
sudo dnf install openssl-devel
```

如果你使用的是新版的macos12，运行V代码时出现openssl的报错，需要从openssl的源码来编译安装，就可以解决报错：

```shell
git clone https://github.com/openssl/openssl.git
cd openssl
./configure
make
make install
```

#### 可选GC

- 安装libgc-dev包

  macos:

  ```shell
  brew install libgc
  ```

  linux:

  ```shell
  sudo apt-get install libgc-dev
  ```

- 也可手工下载发布的稳定版本: 

  https://github.com/ivmai/bdwgc/releases, 目前最新的稳定版是8系列

  在编译之前需要先安装automake工具

  ```shell
  brew install automake
  ```

  然后下载并编译:

  ```shell
  git clone git://github.com/ivmai/bdwgc.git
  #建议先切换到最新的稳定版的分支，然后再编译
  cd bdwgc
  git clone git://github.com/ivmai/libatomic_ops.git
  ./autogen.sh
  ./configure
  make -j
  make check
  ```

  安装完成后，可以在终端上输入，如果正常输出，那就是安装成功

  ```shell
  gc #输出On branch release-8_2
  ```

​		关于可选GC可进一步参考：[内存管理章节](memory.md)。

### 后续升级

方式一:

  ```v
v up //抓取github上V代码库的主干代码，然后自动重新编译
  ```

方式二:

  ```shell
git pull
make
  ```

### 预编译安装包

在[官网](https://vlang.io/)直接下载对应平台的安装包，这个目前不推荐使用，更新太慢。

### 增加symlink

可以通过增加symbol link,让v编译器随处可用。

在unix，linux，mac系统中，进入到v可执行文件所在的目录，然后执行：

```shell
sudo ./v symlink
```

会创建/usr/local/bin/v链接。

在windows中，使用系统管理员打开命令行窗口，进入到v.exe所在的目录，然后执行以下命令，会创建v环境变量：

```
.\v.exe symlink
```

以上命令只需执行一次，如果v命令更换了位置，每次启动会自动更新快捷方式和环境变量。
