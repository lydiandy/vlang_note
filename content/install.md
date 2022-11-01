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
make # 默认用tcc编译，速度极快，一般1-3秒，生成文件7MB左右，至少需要一次make，只要有V编译器可执行文件，后续升级就可不用再使用make，直接使用v up升级。
```

编译成功后，会在当前目录生成V编译器的可执行文件，生产环境编译（使用gcc编译，带-prod选项）可执行文件大小为3M多，小巧得很。

查看V编译器的版本：

```shell
v -v
v version
v -v version #更详细的版本信息
```

#### 运行代码

编译成功后，可以运行代码：

```v
//main.v
module main

fn main() {
	println("hello vlang")
}
```

在终端中执行，输出hello vlang，则安装成功。

```shell
v run main.v
```

编译器命令行的使用参考：[编译器命令行使用章节](toolchain.md)。

#### 增加环境变量

除了手工将V编译器增加到PATH环境变量中，也可通过以下命令，让V编译器随处可用。

在unix/linux/mac系统中，进入到v可执行文件所在目录，然后执行以下命令，会创建/usr/local/bin/v 链接。

```shell
sudo ./v symlink
```

在windows中，使用系统管理员打开命令行窗口，进入到v.exe所在目录，然后执行以下命令，会创建v环境变量：

```
.\v.exe symlink
```

以上命令只需执行一次，如果v命令更换了位置，每次启动会自动更新快捷方式和环境变量。

### 安装可选依赖

如果要使用v install安装模块，或使用net.http, net.websocket模块，就需要ssl库，编译器默认使用内置的轻量级ssl库[mbedtls](https://github.com/Mbed-TLS/mbedtls)。

如果不想使用内置的ssl库，而是使用openssl，可以先安装openssl，然后在编译时加上编译选项：-d use_openssl。

```shell
#macOS:
brew install openssl

#Debian/Ubuntu:
sudo apt install libssl-dev

#Arch/Manjaro:
openssl is installed by default

#Fedora:
sudo dnf install openssl-devel
```

如果使用的是新版的macos12，运行V代码时出现openssl的报错，需要从openssl的源码来编译安装，就可以解决报错：

```shell
git clone https://github.com/openssl/openssl.git
cd openssl
./configure
make
make install
```

### 后续升级

方式一:

  ```shell
v up #抓取github上V代码库的主干代码，然后自动重新编译
  ```

方式二:

  ```shell
git pull
make
  ```

### 预编译版本

在[官网](https://vlang.io/)直接下载对应平台的预编译版本，这个目前不推荐使用，更新比较慢，只有发布了新版本才会更新，也可以下载[周版本](https://github.com/vlang/v/releases)。

下载预编译的压缩文件后，只要解压缩，然后把解压缩文件中的v可执行文件加入到PATH路径中就可以使用了。

```shell
v version #输出预编译版本的版本号
```

