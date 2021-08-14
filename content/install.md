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

编译成功后,会在当前目录生成V编译器的可执行文件,大小2M左右,小巧得很.

使用v version查看V编译器的版本

#### 运行代码

编译成功后,可以尝试运行代码:

```v
//main.v
module main

fn main() {
	println("hello V")
}
```

在终端中执行:

```shell
v run main.v
```

输出hello V,则安装成功.

编译器命令行的使用参考:[编译器命令行使用章节](toolchain.md)

#### 开发工具

目前V语言的开发工具首选vs code,有官方维护的插件:https://github.com/vlang/vscode-vlang

基本的功能都有:

- 语法着色
- 代码提示
- 代码格式化
- 代码折叠
- 代码大纲视图
- 代码跳转定义

#### 可选安装

如果需要执行v install安装模块或编译http相关模块,需要安装openssl

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

#### 后续升级

方式一:

  ```v
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

### 增加symlink

可以通过增加symbol link,让v编译器随处可用

在unix,linux,mac系统中,进入到v可执行文件所在的目录,然后执行:

```shell
sudo ./v symlink
```

会创建/usr/local/bin/v链接

在windows中,使用系统管理员打开命令行窗口,进入到v.exe所在的目录,然后执行:

```
.\v.exe symlink
```

会创建v环境变量

以上命令只需执行一次,如果v命令更换了位置,每次启动会自动更新快捷方式和环境变量

###安装技巧

有时也可能存在`https://github.com`访问不了的情况，可以按照以下办法安装或者更新vlang：
1. git clone --depth=1 --single-branch `git`://github.com/vlang/v(更新时跳过)
2. cd v
3. (更新时)git pull
4. git clone --depth=1 --single-branch `git`://github.com/vlang/vc （更新时cd vc, git pull）
5. gcc -std=c99 -w -municode -o v1.exe vc/v_win.c
6. v1 -o v.exe cmd/v
