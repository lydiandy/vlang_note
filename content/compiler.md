## V语言编译器源代码

新版本的V代码中,编译器已经从主模块被移到了vlib/compiler模块中

V编译器的入口文件已经变为根目录的v.v

编译器的主模块,主函数都在v.v里面,编译成v可执行文件

------

源代码结构介绍文档:https://github.com/vlang/v/blob/master/CONTRIBUTING.md

vlib/compiler编译器目录:

1. main.v  --编译器模块主文件

- 识别出编译模式
- 构造编译器对象(struct V)
- 生成一个需要被解析的.v文件的列表
- 为每一个.v文件生成一个解析对象,然后运行解析函数,解析器直接生成C或者x64代码,由于性能原因,没有中间步骤(没有AST语法树或者汇编代码生成)
- 如果解析成功,则生成一个单一的C源文件
- 最后调用C编译器,编译生成可执行文件或者库文件

2. parser.v

   - 编译器的核心,这是最大的一个文件,解析函数通过扫描器生成一组需要被解析的token列表,

   - 在V对象能够被使用以前,会被2次扫描,第一次扫描会忽略函数体的内容,仅仅是扫描各种函数签名,结构体签名,然后被存储在内存中;第二次扫描才会扫描函数体内部的代码,然后生成C代码或机器码

   - 格式化器被嵌入在解析器中,在解析代码的过程中,同时格式化代码

3. scanner.v

- 扫描器的作用是,解析一组字符,然后把它们转成token

4. token.v

   token.v仅仅简单列出所有的token列表

5. table.v

   V编译器创建了一个表格对象,共享给所有解析器使用,它包含了所有的类型,函数,常量,以及好几个帮助器,用来按名字搜索所有的对象

6. cgen.v

   这个小cgen结构体帮助生成C代码,

7. fn.v

   控制声明,调用正常或者异步的函数和方法,这个文件大约有1000行代码,有一些复杂的逻辑,今后需要被清理和简化一下

8. json.v

   定义json代码生成,这个文件将被删除,一旦V支持编译时代码生成

   

9. x64/

   这个目录里面包含了所有机器代码生成的逻辑,很明显这个是编译器代码中最复杂的部分,这个部分定义了各种函数,生成汇编代码和机器码

### 编译器源代码文件说明

编译器的源代码位于compiler目录中,29个V源文件:

| 源代码文件名      | 功能说明                                                     |
| ----------------- | ------------------------------------------------------------ |
| main.v            | 编译器的主文件,定义v.v所需的各种函数,以及new_v()创建V变量    |
| vhelp.v           | 定义v编译器帮助文本常量                                      |
| vfmt.v            | 实现代码格式化,未实现                                        |
| token.v           | 词法单元                                                     |
| table.v           | 符号表,存放要编译的源代码中:所有的模块,导入,常量,函数,方法,类型(结构体) |
| scanner.v         | 扫描                                                         |
| repl.v            | 交互式模式                                                   |
| query.v           |                                                              |
| parser.v          | 解析语法                                                     |
| parser2.v         |                                                              |
| optimization.v    |                                                              |
| msvc.v            | 使用微软VC编译器                                             |
| modules.v         | 模块                                                         |
| module_header.v   | 生成.vh相关                                                  |
| live.v            | 实时热更新                                                   |
| jsgen.v           |                                                              |
| gen_js.v          | 生成js代码                                                   |
| gen_c.v           | 生成C代码                                                    |
| struct.v          | 结构体相关                                                   |
| fn.v              | 函数相关                                                     |
| depgraph.v        | 模块依赖图                                                   |
| comptime.v        |                                                              |
| cc.v              |                                                              |
| cflags.v          |                                                              |
| cgen.v            | 生成C代码                                                    |
| cheaders.v        | 统一存放V编译器所需的所有C代码的头文件,包括V所有的基本类型定义 |
| enum.v            | 解析器结构体处理枚举的代码                                   |
| vtest.v           |                                                              |
| compiles_errors.v | 编译器错误处理相关                                           |
| test目录          | 编译器的各种测试文件,可以从这里看到很多V基本功能的使用代码   |

### 编译器代码中的核心类型

- struct V  //编译器V类型,表示一个编译器实例

  一个V编译器包含:

  一个Perfercences编译参数,

  一个Table符号表,

  一个CGen代码生成,

  一个解析器数组[]Parser, 一个文件对应一个解析器,一个解析器里包含一个扫描器

  ------

  

- struct Preferences //编译参数配置类型,用来保存编译的相关参数

  ------

  

- struct Table //符号表,包含了整个程序所有的一级元素

  []modules //一个应用包含的所有模块

  []imports //应用导入/依赖的模块

  consts 一个应用所有的常量,是[]Var数组

  typesmap 一个应用所有的类型

  fns 一个应用所有的函数

  file_imports 一个应用导入的所有文件

  ------

  

- struct Parser //解析器,一个文件一个解析器,一个解析器包含一个扫描器

  file_path //文件路径

  file_name //文件名称

  

  scanner  //解析器包含一个扫描器

  

  

  

  

  ------

  

- struct Scanner //扫描器

  一个源文件一个扫描器

  

  

  

  

  ------

  

- struct Var //变量

  typ //变量类型

  name //变量名称

  is_const //是否是常量

  is_arg //是否是函数的参数

  is_mut //是否可变

  mod //所属模块

  ...

  ------

  

- struct Fn //函数

  name //函数名

  mod //函数所属模块

  args  //函数参数数组, []Var

  is_c  

  is_public

  is_mothod

  ...

  ------

  

- struct Type //类型

  mod 所属的模块

  name 类型名

  cat 类型类别

  fields 类型包含的字段,是[]Var

  methods 类型包含的方法,是[]Fn

  parent 父类型

  is_c 是否是c类型

  ...

  ------

  

- struct TypeCategory //类型分类

  ------

  

- struct GenTable //泛型表

  ------

  

- struct FileImportTable //源文件导入的文件/模块信息

  ------

  

- struct CGen // 生成C代码

  ------

  

- enum Token   //词法单元枚举,所有的语言基本词法单元,运算符,关键字都在这里用枚举进行对应

  ------

  

- enum AcessMod //5种访问控制

------

