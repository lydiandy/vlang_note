## V报错定位及处理

实际使用V开发的过程中，有时会碰到一些奇怪的，难以定位的报错。

有些是自己代码本身的写法有问题，有一些确实会碰到编译器的bug。

如何快速地对错误进行定位和处理，会节省很多无谓的时间浪费。

从原理上来说V编译器的过程是: V代码=>C代码=>可执行文件。

正常来说V编译器会尽可能地发现编译错误，并给出比较明确的提示哪里出错了。

可是如果V编译器在生成C代码的过程中，生成有错的C代码，导致C编译器编译出错，就需要进一步的调试跟踪选项来定位错误。

### C后端调试

常用编译带调试选项：

- -g

  不做代码优化，加入更多的调试信息在生成的可执行文件中，编译器会强制把v源代码中的行号加入到跟踪堆栈stacktraces中。

- -cg

  不做代码优化，加入更多的调试信息在生成的可执行文件中，编译器会强制把C源代码中的行号加入到跟踪堆栈stacktraces中,一般会配合-keepc使用，每次编译都会保留生成的C源代码。

- -showcc

  显示使用的C编译器名字

- -show-c-output

  显示C编译器编译报错时的输出。

- -keepc

  不删除生成的C源代码，方便跟踪调试。

```shell
v -cg run main.v

如果报错,输出类似这样的:
C compiler=/var/folders/lk/k709921d2gl4jrh31mt8_ktm0000gn/T/v/main.tmp.c:
329:2:error: must use 'struct' tag to refer to type 'timeval'
        timeval timeout;
        ^
        struct 
```

命令行调试：

```shell
v -keepc -cg -showcc yourprogram.v	//编译带调试信息的二进制文件
lldb hello		//然后使用lldb或者gdb进行调试
gdb hello
```

可以使用help子命令查看所有生成C的选项：

```shell
v help build-c
```

### js后端调试

要调式生成的js代码，可以激活sourcemap：

```shell
v -b js -sourcemap hello.v -o hello.js
```

可以使用help子命令查看所有生成js的选项：

```
v help build-js
```

```shell
Interfacing the Javascript Backend code generation, passing options to it:
   -prod
      Do not create any JS Doc comments

   -sourcemap
      Create a source map for debugging

   -sourcemap-inline
      Embed the source map directly into the JavaScript source file
      (currently default, external source map files not implemented)

   -sourcemap-src-include
      Include the orginal V source files into the generated source map
      (default false, all files in the source map are currently referenced by their absolute system file path)

      The supported targets for the JS backend are: ES5 strict
```

目前js后端生成的是ES5 strict的代码，ES6的还没有。

### native后端调试

目前native后端还在开发中，目前没有还不支持调试。
