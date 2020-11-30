## 包管理器

模块就是包,两个所指的含义完全一样

vpm是v的模块管理器/包管理器,采用集中式的包服务器,所有第三方模块全部要发布模块到[https://vpm.best/](https://vpm.best/)网站提供给别人使用

### 上传模块

登录[https://vpm.best](https://vpm.best/)

然后github账号集成登录,就可以上传自己的第三方模块

### 安装模块

```
v install nedpals.args //使用作者账号的名称作为路径,用点号分隔
v install regex
```

如果设置了环境变量VMODULES,则会安装到VMODULES环境变量指向的目录,如果没有设置环境变量,mac/linux系统会下载到:~/.vmodules目录中,windows系统会把包下载到:C:\Users\xxx\ .vmodules目录中

```
~/.vmodules/nedpals/args

~/.vmodules/regex
```

使用的时候import regex就可以了,v会到VMODULES中查找对应的包

如果是从git直接下载的源代码,或者作者没有上传包到vpm上,也可以使用创建link链接的方式,把目录链接创建到~/.vmodules目录中

```
git clone https://github.com/xxx //下载源代码
ln -s xxx ~/.vmodules/xxx //创建目录链接,记得使用绝对路径
```

常用的模块管理命令:

```shell
v search xxx //搜索指定关键字的包
v intall xxx //安装包
v update xxx //升级包
v remove xxx //删除包
v update xxx //升级指定已安装的包
v upgrade		 //升级所有已安装的包
v list			 //列出所有已安装的包
v outdated	 //列出所有过时需要升级的包
```

### 模块搜索路径

当使用import xxx导入模块时,编译器会按以下顺序搜索模块:

1. 当前目录
2. 标准模块目录,即v/vlib
3. 第三方模块目录:如果设置了环境变量VMODULES,就是VMODULES环境变量指向的目录,如果没有设置VMODULES,默认是~/.vmodules目录

### 模块描述文件

vpm使用v.mod作为模块描述文件,json格式:

```json
Module {
        name: 'ui'
        author: 'medvednikov'
        version: '0.0.1'
        repo_url: 'https://github.com/vlang/ui'
        vcs: 'git'
        tags: ['gui','user interface']
        description: 'V UI is a cross-platform UI toolkit for Windows, macOS, Linux, and soon Android, iOS and the web (JS/WASM).'
        license: 'GPL3 + commercial'
}
```

跟node的package.json类似,然后把下载的包统一放到VMODULES文件夹中,同一个包区分版本,提供个本机的所有项目使用

### 创建模块项目

```
v new //创建一个新项目,根据提示输入项目名称,描述等,生成的项目目录带有v.mod
v init //把当前目录作为项目，创建项目v.mod
```

### 解析模块描述文件

可以在代码中导入v.mod模块来解析v.mod,通过vmod.decode进行解码,这样就可以根据v.mod文件的内容方便实现各种库功能

```v
import v.vmod
vm := vmod.decode( @VMOD_FILE ) or { panic(err) } //@VMOD_FILE是内置的全局变量,返回v.mod文件内容,字符串类型
eprintln('$vm.name $vm.version\n $vm.description')
```

``