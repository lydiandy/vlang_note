## V报错定位及处理

实际使用V开发的过程中,有时会碰到一些奇怪的,难以定位的报错

有些是自己代码本身的写法有问题,有一些确实会碰到编译器的bug

如何快速地对错误进行定位和处理,会节省很多无谓的时间浪费

从原理上来说V编译器的过程是: V代码=>C代码=>可执行文件

正常来说V编译器会尽可能地发现编译错误,并给出比较明确的提示,哪里出错了

可是如果V编译器在生成C代码的过程中,生成有错的C代码,导致C编译器编译出错

方法一:

编译V代码时,如果报的是C代码部分的错误,可以在编译时加上-cg选项,显示更多的报错信息,以及报错对应的C代码行,可以更快地定位错误:

```c
v -cg run main.v

如果报错,输出类似这样的:
C compiler=/var/folders/lk/k709921d2gl4jrh31mt8_ktm0000gn/T/v/main.tmp.c:
329:2:error: must use 'struct' tag to refer to type 'timeval'
        timeval timeout;
        ^
        struct 

```

方法二:

先手工将V代码编译生成C代码,然后使用C的调试工具,比如gdb,lldb等,进行定位,分析问题

```shell
v -o main.c main.v //生成对应的C代码
```

