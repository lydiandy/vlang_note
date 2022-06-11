## V语言服务(VLS)

V实现了语言服务协议LSP v3.15版本，叫做V Language Server(VLS)。

源代码：https://github.com/vlang/vls

目前vls支持vs code和sublime text的插件调用，idea和其他开发环境还未适配。

### 安装

#### 方式一：直接在vs code上安装

在vs code上安装V语言插件后，可以通过命令面板搜索并执行：V:Update VLS，此命令的默认安装路径为：~/.vls。

安装后可以重复执行命令，升级VLS。

#### 方式二：从源代码安装

```shell
#下载vls源代码
git clone https://github.com/vlang/vls.git && cd vls
#编译vls，编译完成后可以在vls/bin目录中看到vls可执行文件
v build.vsh
#也可以选择指定的C编译器来编译
v run build.vsh cc/gcc/clang/msvc
#windows下要明确指定C编译器：
v run build.vsh gcc/msvc
```

编译完成后，在vs code上通过命令面板搜索并执行：V:Restart VLS，重启VLS。

后续的日常更新

由于vls目前还不是太稳定,还在不断地更新中,如果想快速使用最新版本的vls可以自己更新代码,自己重新编译:

```shell
#更新vls,在vls代码库目录中执行
git pull
#重新编译vls
v build.vsh
也可以选择指定的C编译器来编译
v run build.vsh cc/gcc/clang/msvc
```

如果使用-gc boehm编译报了以下错误,那就是还没有安装可选GC,可以参考[内存管理章节中的安装可选GC](memory.md)

```shell
error: Cannot find "bdw-gc" pkgconfig file
29 | } $else {
   30 |     $if macos {
   31 |         #pkgconfig bdw-gc
      | ~~~~~~~~~~~~~~~~~~
   32 |     } $else $if openbsd || freebsd {
   33 |         #flag -I/usr/local/include
```

#### 方式三：直接下载预编译的二进制文件

下载地址：https://github.com/vlang/vls/releases

如果不希望vs code插件使用默认的vls，而是想使用方式二和方式三的vls,可以在vs code中配置vls可执行文件的自定义路径：

![](vls.assets/instructions.png)

安装并配置完成后，就可以在vs code中使用V语言插件提供的代码完成，代码大纲，跳转到定义等功能。

### VLS出错后的重启

目前VLS还不是太稳定，如果在vscode使用的过程中，出现vls进程报错退出，可以执行cmd(ctrl)+shift+p调出命令面板，找到V: Restart VLS命令，执行即可重启VLS。

### 实现

vls基于tree-sitter实现，tree-sitter相关代码库:

**tree-sitter:**

https://tree-sitter.github.io/tree-sitter

https://github.com/tree-sitter/tree-sitter

**tree-sitter-v:**

https://github.com/vlang/vls/tree/use-tree-sitter/tree_sitter_v