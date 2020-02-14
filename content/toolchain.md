## V工具链使用

V编译器文件大小就700多K,实在是小巧得很

### V编译器命令行使用说明

执行v help就可以查看到下面的命令行使用说明

Usage: v [options/commands] [file.v | directory]

- 直接执行V,不带任何参数时,直接进入交互模式
- 编译某个指定的.v源代码文件,会生成同名的可执行文件,也可以使用-o选项,生成特定的可执行文件名
- 编译某一个指定的目录,编译器会编译目录中所有*.v的源代码,生成一个单一的可执行文件,文件名跟目录同名
- 所有以  _ test.v结束的文件,会被编译器当做测试文件,编译器执行测试文件时,会按顺序执行所有的以test_开头的测试函数
- 可以把常用的选项参数维护到 VFLAGS 的系统环境变量中,这样就不用每次都重复输入相同的选项

```
Options/commands:   选项/命令
  -h, help          显示编译器的帮助信息
  -o <file>         指定编译后生成的可执行文件名
  -o <file>.c       编译生成C源文件,而不是可执行文件
  -o <file>.js      编译生成js源文件
  -prod             生产优化模式编译,生成更小的可执行文件
  -v, version       显示编译器的版本信息
  -verbose          编译的同时生成log，显示编译器正在进行的内容
  -live             启用代码热更新编译(只对标注为[live]的函数生效,修改函数内容会实时编译)
  -os <OS>          跨平台交叉编译,编译生成指定os的可执行文件
                    OS可以是:linux, mac, windows, msvc
  -shared           编译生成共享库
  -stats            编译时显示额外的统计信息,比如:`v -stats test .`

  -cache            启用预编译模块的缓存,这样在第二次编译时,编译速度会显著加快

  -obf              混淆编译生成可执行文件
  -compress         调用upx，压缩加壳生成二进制文件
  -                 进入交互式模式

Commands:           子命令:
  up                升级编译器V到最新版本
  run <file.v>      编译并运行指定的V源代码文件
  build <module>    编译模块,生成.o目标文件
  runrepl           运行交互式模式
  symlink           对unix系操作系统比较有用,会在/usr/local/bin/v生成V命令行的替身,
  									这样就可以在任何目录执行v命令行
  install <module>  从https://vpm.vlang.io/,安装指定的一个或多个模块
  test v            执行V编译器源代码中的所有测试文件和V example
  test folder/      运行指定目录及子目录中的所有测试文件,也可执行指定的某一个测试文件
  fmt               执行vfmt工具,进行代码格式化
  doc               执行vdoc文档工具,生成源代码文档[开发中]
  translate         把C源代码翻译成V源代码[开发中,估计0.3版本才可以使用]
  create            以交互方式创建一个v项目,生成v.mod项目文件以及主文件


Options for debugging/troubleshooting v programs:

  -g                Generate debugging information in the backtraces. Add *V* line numbers to the generated executable.
  -cg               Same as -g, but add *C* line numbers to the generated executable instead of *V* line numbers.
  -keep_c           Do NOT remove the generated .tmp.c files after compilation.
                    It is useful when using debuggers like gdb/visual studio, when given after -g / -cg .
  -show_c_cmd       Print the full C compilation command and how much time it took.
  -cc <ccompiler>   Specify which C compiler you want to use as a C backend.
                    The C backend compiler should be able to handle C99 compatible C code.
                    Common C compilers are gcc, clang, tcc, icc, cl...
  -cflags <flags>   Pass additional C flags to the C backend compiler.
                    Example: -cflags `sdl2-config --cflags`
```

常用命令例子：

```shell
v main.v //编译当前目录中的main.v源文件,生成同名的main可执行文件
v run main.v //编译当前目录中的main.v源文件，生成同名的main可执行文件，并运行

v -o myexe main.v //编译当前目录中的main.v源文件，生成的可执行文件名为myexe
v -o myexe.c mani.v //编译当前目录中的main.v源文件，生成对应的C源文件，而不是可执行文件
v -o myexe.js main.v //编译当前目录中的main.v源文件，生成对应的js源文件，而不是可执行文件

v -prod main.v //生产模式编译当前目录中的main.v源文件，生成更小的可执行文件

v -h //查看编译器帮助文本
v help //查看编译器帮助文本

v -v      //查看编译器版本
v version //查看编译器版本

v    //进入交互模式
v -  //进入交互模式

v build mymodule //编译mymodule模块（当前位置要在mymodule的上级目录）

v up //升级V编译器到最新版本,等价于git pull && make

v install xxx模块 //从https://vpm.vlang.io/安装指定的模块

v fmt main.v //统一格式化指定源文件或目录中的代码

v test mymodule //执行mymodule中的测试文件

v test v  //执行v源代码中所有的测试文件

```

### V代码编译报错定位

​	编译V代码的时候,如果报错,可以在编译时加上-cg选项,可以提示报错更多的信息,以及报错对应的C代码行,可以更快地定位错误:

```c
v run -cg main.v
如果报错可以输出类似这样的:
C compiler=
/var/folders/lk/k709921d2gl4jrh31mt8_ktm0000gn/T/v/main.tmp.c:329:2: error: must use 'struct' tag to refer to type 'timeval'
        timeval timeout;
        ^
        struct 

```

