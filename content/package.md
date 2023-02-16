## 包管理器/模块管理器

包就是模块，两个所指的含义完全一样，包管理器也叫模块管理器 。  

vpm(v package management)是V语言的包管理器，采用集中式的包服务器，所有第三方模块可以发布模块到 [https://vpm.vlang.io](https://vpm.vlang.io) 提供下载安装，也可以直接从github或者hg代码库直接安装。

### 上传模块

登录[https://vpm.vlang.io](https://vpm.vlang.io)，使用github账号集成登录，就可以上传自己的第三方模块。

### 安装模块

```shell
v install nedpals.args #使用作者账号的名称作为路径，用点号分隔
v install regex
```

如果设置了环境变量VMODULES，则会安装到VMODULES环境变量指向的目录。

如果没有设置环境变量，mac/linux系统会默认下载到：~/.vmodules目录中，windows系统会默认下载到：C:\Users\xxx\ .vmodules目录中。

```shell
~/.vmodules/nedpals/args

~/.vmodules/regex
```

使用import语句就可以导入模块，编译器会到VMODULES路径中查找对应的模块。

常用的模块管理命令：

```shell
v search 模块名 		#搜索指定关键字的模块
v intall 模块名 		#安装模块
v update 模块名 		#升级模块
v remove 模块名 		#删除模块
v update 模块名 		#升级指定已安装的模块
v upgrade				  #升级所有已安装的模块
v list			 			#列出所有已安装的模块
v outdated			  #列出所有过时需要升级的模块
```

默认情况下，v install默认从vpm网址安装模块，也可以通过增加参数，从git或hg代码库安装模块。

```shell
v install 模块名					 #默认从vpm官网安装模块
v install -git 模块url		#从git代码库安装模块
v install 模块url					#如果install后面是git代码库网址，也可以忽略-git
v install -hg	模块名url		#从mercurial代码库安装模块
```

如果希望强制重新安装，不管模块是否已经存在，可以增加-f或-force参数

```shell
v install -f 模块名
v install -force 模块名
```

更多v install的用法可以查看完整命令行帮助文档：

```shell
v install -help
```

如果是从git直接下载的源代码，或作者没有上传包到vpm上，

也可以使用创建link链接的方式，把目录链接创建到~/.vmodules目录中：

```shell
git clone https://github.com/xxx #下载源代码
ln -s xxx ~/.vmodules/xxx #在vmodules目录中，创建目录链接，xxx记得使用绝对路径
```

### 模块描述文件

vpm使用v.mod作为模块描述文件， json格式，跟node的package.json类似。

```json
Module {
        name: 'ui'
        author: 'medvednikov'
        version: '0.0.1'
        repo_url: 'https://github.com/vlang/ui'
        vcs: 'git'
        tags: ['gui'，'user interface']
        description: 'V UI is a cross-platform UI toolkit for Windows， macOS， Linux， and soon Android， iOS and the web (JS/WASM).'
        license: 'GPL3 + commercial'
}
```

### 创建模块项目

```shell
v new #创建一个新项目，根据提示输入项目名称，描述等，生成的项目目录带有v.mod
v new <project_name> <description> <version> <license>
v init #把当前目录作为项目，创建项目v.mod
```

### 解析模块描述文件

可以在代码中导入v.mod模块来解析v.mod中的内容。

通过vmod.decode进行解码，这样就可以根据v.mod文件的内容方便实现各种库功能。

```v
import v.vmod
vm := vmod.decode( @VMOD_FILE ) or { panic(err) } //@VMOD_FILE是内置的全局变量，返回v.mod文件内容，字符串类型
eprintln('$vm.name $vm.version\n $vm.description')
```

### 标准模块缓存

V编译器默认会启用vlib标准库的缓存，编译一次标准模块后会缓存在~/.vmodules/cache中，缩短编译时间。

### 模块存储方式

使用vpm工具下载第三方依赖包时，包会统一放到VMODULES文件夹中，同一个包会区分版本，不同版本存放在不同目录，提供给本机的所有项目使用。
