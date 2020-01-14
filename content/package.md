## 包管理器

vpm,V的包管理器,采用的是集中式的包服务器,全部要发布模块到 https://vpm.best/

上传模块,登录[https://vpm.best](https://vpm.best/),

然后github账号集成登录,就可以上传自己的第三方模块

安装模块:

```
v install nedpals.args //使用作者账号的名称作为路径,用点号分隔
v install regex
```

执行完毕后,会把包下载到~/.vmodules目录中

```
~/.vmodules/nedpals/args

~/.vmodules/regex
```

使用的时候import regex就可以了,v会到~/.vmodules中查找对应的包

如果是从git直接下载的源代码,或者作者没有上传包到vpm上,也可以使用创建link链接的方式,把目录链接创建到~/.modules目录中

```
git clone https://github.com/xxx //下载源代码
ln -s xxx ~/.vmodules/xxx //创建目录链接
```

常用的包管理命令:

```shell
v search xxx //搜索指定关键字的包
v intall xxx //安装包
v update xxx //升级包
v remove xxx //删除包
```

使用v.mod作为模块描述文件,json格式:

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

跟node的package.json类似,然后把下载的包统一放到~/.vmodules文件夹中,同一个包区分版本,提供个本机的所有项目使用

创建模块项目:

```
v create //创建一个项目,根据提示输入项目名称,描述等,生成的项目目录带有v.mod
```

目前社区有2个人各自实现了2个包管理器,估计以后会合并进v标准的包管理

https://github.com/v-pkg/vpkg   

https://github.com/yue-best-practices/vpm

