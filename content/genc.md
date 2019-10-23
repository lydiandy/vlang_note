## 编译生成C代码

编译器本质上就是一门语言生成另一门语言的过程

**V目前的编译思路是:编译生成对应的C代码,然后调用C编译器来生成可执行文件**

V语言的开发重点在编译器前端,C就是编译器后端

现在也生成了js代码,估计很快就会生成wasm代码

或者考虑基于LLVM,生成LLVM IR

或者直接生成机器码

------

想要查看V代码生成的C代码,只要增加-o参数就可以了,编译器只会生成C代码

比如,想把当前目录的main.v代码生成main.c代码,执行以下命令就可以:

```
v -o main.c ./main.v
```

通过查看V代码生成的C代码,可以更容易理解V编译器是如何编译的

- 基本类型对应

  v的基本类型通过c的类型别名typedef来实现

  ```C
  typedef int64_t i64;
  typedef int16_t i16;
  typedef int8_t i8;
  typedef uint64_t u64;
  typedef uint32_t u32;
  typedef uint16_t u16;
  typedef uint8_t byte;
  typedef uint32_t rune;
  typedef float f32;
  typedef double f64;
  typedef unsigned char* byteptr; //字节指针
  typedef int* intptr; //整数指针
  typedef void* voidptr; //通用指针
  typedef struct array array;
  typedef struct map map;
  typedef array array_string;
  typedef array array_int;
  typedef array array_byte;
  typedef array array_f32;
  typedef array array_f64;
  typedef map map_int;
  typedef map map_string;
  #ifndef bool
  	typedef int bool; //布尔类型在C里面通过int类型来实现,32位系统是2字节,64位系统是4字节
  	#define true 1 //true是整数常量1
  	#define false 0 //false是整数常量0
  #endif
  ```

- 函数

  生成对应的c的函数

- 数组

  V的数组是用struct来实现的,生成C代码也是struct

- 字符串

  V的字符串是用struct来实现的,生成C代码也是struct

- 字典

  V的字典是用struct来实现的,生成C代码也是struct

- 常量

  通过C的宏定义#define来实现

- 枚举

  通过C的宏定义#define来实现,常量名字的规则是:"模块名__ 枚举名__值名"

  ```C
  #define main__myenum_x 0
  #define main__myenum_y 1
  #define main__myenum_z 2
  ```

- 结构体

  生成对应的C结构体

- 结构体方法

  生成对应的C函数,函数的第一个参数是对应的结构体类型,生成的C函数的命名规则是:"结构体名__方法名"

- 接口

  具体实现方式还没有细看,不过也是通过结构体,函数,指针来实现的

- 流程控制语句

  - if

    生成C的if语句

  - switch

    生成C的 if-else if-else语句

  - match

    生成C的 if-else if-else语句

  - for

    for i:=0;i<10;i++{} 生成对应C的for循环

    for in {}  生成C的for循环

    for x<10 {}  生成C的while循环

    for {} 无限循环  生成C的while(1)无限循环

    

- 类型定义type

  类型定义type通过C的类型别名typedef来实现

  

- 模块及模块导入的C代码对应

  

- 访问控制的C代码对应

  

- 泛型的C代码对应

  

- 错误处理的C代码对应

  

  除了以上比较大块的对应关系,每一个V语言的详细语法,都可以在C代码中找到对应的实现方式: