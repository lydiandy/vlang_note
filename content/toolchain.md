## V编译器命令行使用

### 编译器命令行

V编译器命令行大小就2M多,实在是小巧得很

执行v help就可以查看到下面的命令行使用说明

Usage:v [options] [command] [arguments]

- 直接执行V,不带任何参数时,直接进入交互模式
- 编译某个指定的.v源代码文件,会生成同名的可执行文件,也可以使用-o选项,生成特定的可执行文件名
- 编译某一个指定的目录,编译器会编译目录中所有*.v的源代码,生成一个单一的可执行文件或库文件,文件名跟目录同名
- 所有以  _ test.v结束的文件,会被编译器当做测试文件,编译器执行测试文件时,会按顺序执行所有的以test_开头的测试函数
- 可以把常用的编译选项维护到 VFLAGS 的系统环境变量中,运行编译器时,会合并VFLAGS中的选项,这样就不用每次都重复输入相同的选项

```shell
Usage:v [options] [command] [arguments]

新建项目子命令:
   new               创建新的V项目,主要是生成v.mod项目文件
   init              对现有已存在的V项目生成v.mod项目文件
   
标准开发子命令:
   run               编译并运行指定的V源文件或目录
   test              运行指定目录的测试文件
   fmt               格式化代码
   vet							 分析代码存在的错误
   doc               生成指定模块的文档
   repl              运行交互式模式
   watch 						 编译项目,并监控源文件修改,保存后自动重新编译
   ast							 将V源代码生成json格式的AST语法树,直观展现V的语法树
   vlib-docs 				 调用v doc生成vlib标准库的文档	
   interpret				 直接解释执行V代码
   
安装和升级子命令:
   symlink           unix系统在/usr/local/bin/v生成链接,windows生成环境变量

   up                升级编译器V到最新版本,等同于git pull,然后make
   self [-prod]      让V编译器自己编译自己(不执行git pull,不使用make)
   									 可使用-prod优化编译
   version           查看编译器版本
   
包管理子命令:
   install           从vpm/git/hg安装指定的一个或多个模块
   remove            删除已安装的模块
   search            搜索模块
   update            升级指定已安装的模块
   upgrade					 升级所有已安装的模块
   list							 列出所有已安装的模块
   outdated					 列出所有过时需要升级的模块
   show 						 显示模块的详细信息
   
其他子命令:
   translate         把C源代码翻译成V源代码[开发中,估计0.3版本才可以使用]
   doctor						 输出当前电脑的基本环境信息,用于提单到github时,报告环境信息
   tracev						 生成一个带跟踪调试信息的V编译器
```

可以使用v help xxx进一步查看各个子命令的具体帮助文本:

```shell
v help build //显示编译通用选项
v help build-c //显示编译器后端为c(默认)时的编译选项
```

可查看build和run的子命令详细内容,此部分较为重要,同时build和run子命令的编译选项是共用的

```shell
v或v -或v -repl进入交互模式

v -b或-backend c ./main.v //指定编译器后端类型:默认是c,也可以是js,native
v -b js ./main.v	 //指定编译器后端类型为js,目前还是试验性质的,不完善
v -b native ./main.v	 //指定编译器后端类型为native,目前还是试验性质的,不完善
  
v -o或-output main.c ./main.v //编译生成C源文件,而不是可执行文件

v -prod xxx.v //生产优化模式编译,生成更小的可执行文件 不指定-prod选项时优先尝试使用tcc编译(v make时会自动下载),指定-prod选项选项后使用gcc msvc等进行编译

v -skip-unused xxx.v //V代码编译生成C代码时,忽略未使用的C函数,可以进一步缩小可执行文件大小，目前最简单的V程序正常编译成C代码大概是1W行左右，使用了-skip-unused后，减小到5000行左右
v -skip-unused -prod xxx.v //V代码编译生成C代码时,忽略未使用的C函数,并且进行生产编译,可以进一步缩小可执行文件大小
v -skip-unused -o xxx.c xxx.v //生成最小的C文件,忽略未使用的C函数

v -usecache xxx.v  //使用标准库的缓存,而不是每次都重新编译标准库,编译速度快很多
v -usecache -prod xxx.v //使用标准库缓存,生产优化编译,速度也会快很多
v -nocache -prod xxx.v  //取消标准库的缓存,全部重新编译
v -wipe-cache xxx.v //取消标准库的缓存,全部重新编译

v -autofree xxx.v //以自动释放内容方式生成可执行文件
v -obf或-obfuscate //混淆编译生成可执行文件
v -stats //编译时显示额外的统计信息,编译多少行,多少字节,编译时间,每秒编译行数
v -show-timings xxx.v //显示每个编译阶段花费多少时间:扫描,解析,检查,生成C,编译C
v -cg run xxx.v //如果编译报错,-cg选项可以提示报错更多的信息,以及报错对应的C代码行,可以更快地定位错误
v -compress //调用upx，压缩加壳生成二进制文件
v -os <os> //跨平台交叉编译,编译生成指定os的可执行文件,OS可以是:linux, mac, windows, msvc
v -arch x64 //指定编译的架构,可以是x86或x64,默认是x64
v -live   //启用代码热更新编译(只对注解为[live]的函数生效,修改函数内容会实时编译)
v -shared  //编译生成共享库
...
```

### 常用命令例子

```shell
v main.v 			//编译当前目录中的main.v源文件,生成同名的main可执行文件
v run main.v  //编译并运行当前目录中的main.v源文件
v watch main.v //编译并运行当前目录中的main.v源文件,并监控,保存自动重新运行
v interpret ./main.v	 //直接解释执行代码

v project-dir //编译整个目录

v -gc boehm main.v 	//带GC编译，详细选项参考内存管理章节

v -usecache main.v //使用标准库缓存进行编译
v run main.v 	//编译当前目录中的main.v源文件,生成同名的main可执行文件，并运行
v -autofree run main.v //以自动释放内容方式,编译,并运行

v -o myexe main.v 	//编译当前目录中的main.v源文件，生成的可执行文件名为myexe
v -o myexe.c main.v //编译当前目录中的main.v源文件，生成对应的C源文件，而不是可执行文件
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

v install xxx模块 //从https://vpm.vlang.io官方VPM安装指定的模块
v install --git https://github.com/vlang/markdown  //从git代码库安装模块
v install --hg  xxx代码库url //从hg代码库安装模块

v fmt -w main.v //统一格式化指定源文件或目录中的代码

v ast main.v //将V源代码生成json格式的AST语法树,生成main.json
v ast -t main.v //生成简洁的json格式的AST语法书，去除一些不重要的字段
v ast -w main.v //生成main.json,并且监控源文件变化,保存后自动重新生成
v ast -c main.v //将V源代码同时生成AST语法树文件main.json和C源代码main.c,并且监控源文件变化,保存后自动重新生成

v vet ./main.v //分析main.v源文件代码中存在的错误
v vet .			//分析当前目录中所有V源文件代码中存在的错误

v test mymodule //执行mymodule中的测试文件

v test-compiler  //执行v源代码中所有的测试文件,用于测试编译器本身

v vlib/v/compiler_errors_test.v //执行编译器错误测试,checker/tests,parser/tests
VTEST_ONLY=xxx v vlib/v/compiler_errors_test.v //执行编译器错误测试,并且名字包含xxx

v doctor //输出当前电脑的基本环境信息,主要跟V编译相关,用于提单到github时,报告环境信息,方便排查

v self -prod //编译器自己编译自己
v -d time_v self //编译器自己编译自己,并增加自定义编译选项
```

### glibc和musl libc编译

大部分的linux应用都是基于glibc作为C标准库，不过[musl](http://musl.libc.org/)也是一个很优秀的C标准库，体积小，可以静态链接，有自身的特色和场景，目前比较适合在移动端作为C标准库使用，alpine linux发行版的也是使用musl作为C标准库，系统很小型，轻量。

V编译器也支持musl编译，调用musl-gcc作为编译器。

#### 安装musl

在linux系统中，glibc是内置的，musl需要安装，下面是从源代码编译安装的步骤：

```shell
git clone git://git.musl-libc.org/musl		//下载musl源代码库
cd musl
./configure
make
make install	//安装完成后，默认会把musl安装到:/usr/local/musl目录中，也可以自定义安装目录

//编译成功后，musl-gcc编译器默认在/usr/local/musl/bin目录中，需要添加到环境变量中。
export PATH="/usr/local/musl/bin:$PATH"	
```

#### 编译对比

安装成功后，就可以使用musl-gcc选项来编译V源代码，以下是glibc和musl的不同编译结果对比，

最简单的V源代码,如果依赖的libc内容越多，差异应该越大。

main.v

```v
module main

fn main() {
	println('abc')
}
```

使用-cc musl-gcc就是使用musl编译，

以下是在linux mint 20.2版本中的编译对比:

| 编译选项                                                     | 编译大小 |
| ------------------------------------------------------------ | -------- |
| 普通编译                                                     |          |
| v  main.v   (开发默认采用tcc进行编译，速度最快，就是v -cc tcc main.v) | 564.2K   |
| v -cc gcc main.v                                             | 235.3K   |
| v -cc musl-gcc  main.v                                       | 230K     |
| 生产编译                                                     |          |
| v -prod -cc gcc main.v                                       | 97.6K    |
| v -prod -cc musl-gcc main.v                                  | 91.8K    |
| 静态链接                                                     |          |
| v -cc gcc -cflags -static main.v                             | 1.1M     |
| v -cc musl-gcc -cflags -static main.v                        | 264.2K   |
| 生产编译+静态链接                                            |          |
| v -prod  -cc gcc -cflags -static main.v                      | 967.5K   |
| v -prod -cc musl-gcc -cflags -static main.v                  | 91.9K    |
|                                                              |          |

### 自定义编译选项

对于自己的程序也可以使用-d或-define来自定义编译选项，并且可以在代码中接收选项的传入值：

```v
module main

fn main() {
	$if time_v ? {	//自定义编译选项，可以在编译时if的条件语句中进行判断是否有增加了该编译选项
		println('time_v')
	}
	$if abc ? { 
		println('abc')
	}
	$if value ? {
		println('1')
	}
}
```

```shell
v  -d abc -d time_v main.v	#增加自定义编译选项abc和time_v，类型默认为bool类型，默认值为true
v  -d abc -d time_v -d value='0' main.v #也可以使用‘1’或‘0’，等价于true和false
./main 	#输出time_v和abc
```

编译V编译器自身的时候可以添加一些自定义编译选项，让V有不同的行为,比如：

```shell
v -d time_parsing -d time_checking -d time_cgening -d time_v self
```

使用以上几个时间自定义选项编译V编译器后，编译器在编译文件时会详细显示每一个文件在不同阶段消耗的时间：

```shell
0.004    ms v start
0.446    ms parse_CLI_args
0.195    ms parse_file /Users/xx/v/v/vlib/builtin/array.c.v
0.732    ms checker_check /Users/xx/v/v/vlib/strings/builder.v
1.355    ms cgen_file /Users/xx/v/v/vlib/strings/builder.v
837.339  ms v total
```

### 相关环境变量

#### VEXE

VEXE环境变量用于指定V源代码的根目录,一般来说V命令行也在这个目录中.

除非有特殊场景,一般不建议人为指定.

V命令行启动的时候,会检查是否人为指定了该环境变量,如果指定了,就直接使用该环境变量.

如果没有指定,则将VEXE环境变量设置为V命令行所在的目录,

如果fork了V编译器源代码,要修改V编译器源代码,那么最好不要人为设置这个环境变量,让V命令行自动设置该环境变量.

#### VMODULES

VMODULES环境变量用于指定存放第三方包的目录,所有通过v install下载的第三方包都会统一下载到该目录中,如果没有设置该环境变量,默认存放在~/.vmodules中.

#### VOSARGS

用于设置V编译器默认使用的flags字符串，优先级高于os.args和VFLAGS的设置，如果环境变量设置了VOSRAGS则会忽略os.args和VFLAGS的设置，不过感觉这个环境变量没有什么存在的用处，一般用VFLAGS也就够了。

#### VFLAGS

用于设置V编译器默认使用的flags字符串,环境变量中设置的VFLAGS会和os.args中的flags合并，如果两者中有相同的flag，os.args中的flag优先。

#### CFLAGS

用于设置调用C编译器要使用的c flag字符串.

#### LDFLAGS

用于设置调用C编译器要使用的ldflags字符串.

#### VTMP

用于设置V的临时目录,如果没有设置,则使用操作系统临时目录下的v目录作为V的临时目录.

#### VCACHE

用于设置V的缓存目录,如果没有设置,则使用操作系统临时目录下的vcache_folder作为缓存目录.

#### VWATCH_TIMEOUT

 用于设置v watch子命令即使文件没有发生变化,也自动重启程序的时间,以秒为单位.如果没有设置该环境变量,v watch默认是300秒自动重启程序.

#### VCOLORS

用于设置V的终端是否显示彩色字体,可选值为:always,never.

#### VERBOSE

用于设置编译器是否为详细输出模式,可选值为:1/0.

#### VCHILD

待定未尝试.

#### VDIFF_TOOL

v fmt中,如果指定了该环境变量,v fmt -diff不再使用系统默认的比较工具diff,而是使用指定的比较工具.

#### VDIFF_OPTIONS

v fmt中,传递给VDIFF_TOOL比较工具的选项.

#### PKG_CONFIG_PATH

C编译工具pkg-config使用的环境变量,V版本的pkg-config工具也使用了这个环境变量，默认会搜索/usr/lib/pkg-config目录，若找不到，则会去PKG_CONFIG_PATH环境变量指定的路径下查找.