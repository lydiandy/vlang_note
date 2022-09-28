## 生成native代码

### 可执行文件格式

目前native支持生成这三个平台的64位的可执行文件格式，没有使用其他的依赖或中间表示IR，根据AST直接生成对应平台的可执行文件格式。

**linux elf**

ELF (Executable and Linkable Format)是一种为可执行文件，目标文件，共享链接库和内核转储(core dumps)准备的标准文件格式。 Linux和很多类Unix操作系统都使用这个格式。

**macos macho**

Mach-O是Mach Object文件格式的缩写，是mac以及iOS上可执行文件的格式。是一种用于可执行文件、目标代码、动态库的文件格式。

**windows pe**

PE文件是Windows操作系统下使用的一种可执行文件，由COFF（UNIX平台下的通用对象文件格式）格式文件发展而来。32位成为PE32，64位称为PE+或PE32+。

### 使用native后端

目前native后端的编译还非常不成熟，只能编译简单的代码，不过编译后的可执行文件非常小。

使用native后端，就不用再依赖C编译器，直接从AST语法树生成对应平台的可执行文件格式。

目前支持的操作系统：64位的linux/macos/windows。

目前生成的可执行文件，不带调式信息，不支持调试。

最简单的代码，使用native后端：

```v
fn main() {
	println('hello from native V')
}
```

native编译参数说明：

```shell
v help build-native
```

```shell
# Interfacing the Native code generation, passing options to it:
   -v
      Display the assembly code generated (that may change to `-showasm` in the future)

   -arch <arch>
      Select target architecture, right now only `arm64` and `amd64` are supported

   -os <os>, -target-os <os>
      Change the target OS that V compiles for.

      The supported targets for the native backend are: `macos`, `linux` and 'windows'
```

```shell
v -b native main.v #编译
v -b native run main.v #编译并运行
v -b native -arch amd64 run main.v #amd64架构
v -b native -arch arm64 run main.v #arm64架构
v -b native -os macos -arch amd64 run main.v #指定操作系统，指定架构
v -b native -os macos -arch amd64 -v run main.v #-v verbose模式，显示编译过程的额外信息，包括生成的汇编代码
```

### 已支持的语言特性清单

- 内置的打印输出函数print，println
- 基本的函数定义，函数调用
- 基本的结构体定义，结构体创建变量
- 基本的方法定义，方法调用
- 基本的枚举定义，使用
- 内置的typeof函数
- 流程控制语句：if，match，for
- 基本表达式，运算符
- 函数defer语句
- 测试断言语句assert
- inline汇编代码块
