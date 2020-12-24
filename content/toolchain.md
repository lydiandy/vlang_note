## V编译器命令行使用

V编译器文件大小就1.5M左右,实在是小巧得很

执行v help就可以查看到下面的命令行使用说明

Usage:v [options] [command] [arguments]

- 直接执行V,不带任何参数时,直接进入交互模式
- 编译某个指定的.v源代码文件,会生成同名的可执行文件,也可以使用-o选项,生成特定的可执行文件名
- 编译某一个指定的目录,编译器会编译目录中所有*.v的源代码,生成一个单一的可执行文件或库文件,文件名跟目录同名
- 所有以  _ test.v结束的文件,会被编译器当做测试文件,编译器执行测试文件时,会按顺序执行所有的以test_开头的测试函数
- 可以把常用的编译选项维护到 VFLAGS 的系统环境变量中,运行编译器时,会合并VFLAGS中的选项,这样就不用每次都重复输入相同的选项

```shell
Usage:v [options] [command] [arguments]
The commands are:
   build             编译器的默认子命令,编译指定的V源文件或目录
   new               创建新的V项目,主要是生成v.mod项目文件
   init              对现有已存在的V项目生成v.mod项目文件
   doc               生成指定模块的文档
   fmt               格式化
   repl              运行交互式模式
   run               编译并运行指定的V源文件或目录
   symlink           unix系统在/usr/local/bin/v生成链接,windows生成环境变量
   test              运行指定目录的测试文件
   translate         把C源代码翻译成V源代码[开发中,估计0.3版本才可以使用]
   up                升级编译器V到最新版本,等同于git pull,然后make
   self [-prod]      让V编译器自己编译自己(不执行git pull,不使用make),
   									 可使用-prod优化编译
   version           查看编译器版本

   install           从https://vpm.vlang.io/安装指定的一个或多个模块
   remove            删除已安装的模块
   search            搜索模块
   update            升级指定已安装的模块
   upgrade					 升级所有已安装的模块
   list							 列出所有已安装的模块
   outdated					 列出所有过时需要升级的模块
```

可以使用v help xxx进一步查看各个子命令的具体帮助文本:

```
v help build //显示编译通用选项
v help build-c //显示编译器后端为c(默认)时的编译选项
```

可查看build和run的子命令详细内容,此部分较为重要,同时build和run子命令的编译选项是共用的

```v
v -b或-backend c ./main.v //指定编译器后端类型:默认是c,也可以是js,x64
v -b js ./main.v	 //指定编译器后端类型为js,目前还是试验性质的,不完善
v -b x64 ./main.v	 //指定编译器后端类型为x64,目前还是试验性质的,不完善
  
v -o main.c ./main.v //编译生成C源文件,而不是可执行文件
v -prod xxx.v //生产优化模式编译,生成更小的可执行文件
v -autofree xxx.v //以自动释放内容方式生成可执行文件
v -obf或-obfuscate //混淆编译生成可执行文件
v -stats //编译时显示额外的统计信息,比如:`v -stats test .`
v -cg run ./main.v //如果编译报错,-cg选项可以提示报错更多的信息,以及报错对应的C代码行,可以更快地定位错误
v -compress //调用upx，压缩加壳生成二进制文件
v -os <os> //跨平台交叉编译,编译生成指定os的可执行文件,OS可以是:linux, mac, windows, msvc
v -arch x64 //指定编译的架构,可以是x86或x64,默认是x64
v -live   //启用代码热更新编译(只对标注为[live]的函数生效,修改函数内容会实时编译)
v -shared  //编译生成共享库
...
```

常用命令例子：

```shell
v main.v 			//编译当前目录中的main.v源文件,生成同名的main可执行文件
v run main.v 	//编译当前目录中的main.v源文件,生成同名的main可执行文件，并运行
v -autofree run main.v //以自动释放内容方式,编译,并运行

v -o myexe main.v 	//编译当前目录中的main.v源文件，生成的可执行文件名为myexe
v -o myexe.c mani.v //编译当前目录中的main.v源文件，生成对应的C源文件，而不是可执行文件
v -o myexe.js main.v //编译当前目录中的main.v源文件，生成对应的js源文件，而不是可执行文件

v -prod main.v //生产模式编译当前目录中的main.v源文件，生成更小的可执行文件

v help //查看编译器帮助文本
v help build //查看build子命令的帮助文本

v version //查看编译器版本
v --version //查看编译器版本

v    //进入交互模式
v -  //进入交互模式

v build mymodule //编译mymodule模块（当前位置要在mymodule的上级目录）
v . //编译当前目录

v up //升级V编译器到最新版本,等价于git pull && make

v install xxx模块 //从https://vpm.vlang.io/安装指定的模块

v fmt -w main.v //统一格式化指定源文件或目录中的代码

v test mymodule //执行mymodule中的测试文件

v test-compiler  //执行v源代码中所有的测试文件,用于测试编译器本身

v vlib/v/compiler_errors_test.v //执行编译器错误测试,checker/tests,parser/tests
VTEST_ONLY=xxx v vlib/v/compiler_errors_test.v //执行编译器错误测试,并且名字包含xxx

v doctor //输出当前电脑的基本环境信息,主要跟V编译相关,用于提单到github时,报告环境信息,方便排查

```
