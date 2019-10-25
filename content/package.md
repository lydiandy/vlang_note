## 包管理器

V的包管理器,采用的是集中式的包服务器,全部要发布模块到 https://vpm.best/

上传模块,登录[https://vpm.best](https://vpm.best/),

然后github账号集成登录,就可以上传自己的第三方模块

安装模块:

```
v install nedpals.args
v install regex
```

执行完毕后,会把包下载到~/.vmodules目录中

```
~/.vmodules/nedpals/args

~/.vmodules/regex
```

使用的时候import regex就可以了,v会到.vmodules中查找对应的包

目前还没有实现完整,应该实现像go那样的版本化的包管理器,使用json作为描述文件,

比如module.json,跟node的package.json类似,然后把下载的包统一放到.vmodules文件夹中,同一个包区分版本,提供个本机的所有项目使用

然后再实现 v new xxx项目 或者 v init xxx项目,用来创建一个项目

目前社区有2个人各自实现了2个包管理器:

https://github.com/v-pkg/vpkg   //目前选择这个了解一下

https://github.com/yue-best-practices/vpm

