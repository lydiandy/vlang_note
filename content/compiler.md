---
typora-root-url: ../image
---

## V编译器源代码

作者在发布0.2版本之前,重构了编译器,改用基于AST抽象语法树的编译方式,代码也更为清晰易读,

源代码先生成AST,再生成目标代码,有很多好处,是目前各编译器的主流方式

后续编译器再增加新东西,速度会更快

重构了以后,编译器代码更清晰,更好维护

AST可以更快速地生成其他目标代码

AST可以给开发环境以及编译工具使用,更快开发对应的插件和工具

### 代码目录

V命令行代码位于cmd目录

| 子目录 | 说明                                            |
| ------ | ----------------------------------------------- |
| v      | V命令行,入口文件v.v,编译成V可执行文件           |
| tools  | 各种工具类,如vup,vfmt,vpm,vrepl,vtest,vcreate等 |

编译器代码位于vlib/v目录

| 子目录   | 说明                                |
| -------- | ----------------------------------- |
| ast      | AST抽象语法树相关                   |
| token    | 词法单元相关                        |
| table    | 符号表相关                          |
| prep     | 编译选项/参数相关                   |
| scanner  | 词法分析/扫描器相关                 |
| parser   | 语法分析/解析器相关                 |
| checker  | 语法检查器相关                      |
| eval     | 表达式求值相关                      |
| depgraph | 模块依赖相关                        |
| builder  | 代码生成器相关                      |
| gen      | 生成具体平台代码相关,如生成C,js,x64 |
| fmt      | 代码格式化相关                      |
| doc      | 代码文档生成相关                    |
| tests    | 编译器测试代码                      |

### 主要编译过程

- V命令行的入口文件是v/cmd/v/v.v

  .			 		

- V命令行负责处理命令参数,根据参数创建V编译器对象(compiler.V)或调用tools中的各种工具

  . 		

- V编译器对象(compiler.V)创建后,首先根据参数,创建编译器参数对象(pref.Preferences)

  . 		

- V编译器对象根据编译参数,分析得到所有需要编译的源文件数组[ ]File,源文件列表也包括了vlib/builtin中的内置源文件

   .		

- 通过调用v.compile()方法,开始编译,首先创建代码生成器对象(builder.Builder)

   .		

- 通过调用代码生成器对象的b.buildc()方法,把源代码文件数组[]File中的所有V源代码文件生成C源文件

  . 			

- 一个源代码文件对应一个语法分析器,一个语法分析器对应一个词法扫描器

  .

- 代码生成器对象负责创建语法分析器对象(parser.Parser),启动语法分析

  .			

- 语法分析器负责创建词法扫描器对象(scanner.Scanner),启动词法扫描

  . 			

- 语法分析和词法扫描实际上是同步进行,同步完成的.语法分析器对[ ]File进行遍历,以词法单元(token)为基本单位,进行语法分析,词法单元由词法扫描器负责扫描,识别,提供给语法分析器

   .			

- 语法分析遍历[ ]File的所有源文件后,生成对应的文件语法树对象数组[ ]ast.File

   .			

- 语法分析器分析完[ ]File源文件后,接着对所有导入的依赖,再次进行分析,将分析结果追加到文件语法树对象数组([ ]ast.File)中

   .			

- 语法分析器对象分析完成后,输出[ ]ast.File给代码生成器对象,由代码生成器对象,调用指定平台的代码生成对象(gen.Gen/gen.JsGen/gen.x64Gen)来生成目标代码,目前主要是调用gen.Gen来生成C源文件

  .	 	

- 最后调用v.cc方法,调用编译器参数指定的C编译器,将C源文件编译生成可执行文件,完成整个编译过程

  .  		

   **简单总结整个编译过程就是:[ ]File => [ ]ast.File => 目标代码 => 可执行文件**



### 编译器类

以下类图是V编译器中主要的类和枚举:

![](/../content/compiler.assets/V编译器类.jpg)

**主要的调用关系是:compiler.V=>builder.Builder=>parser.Parser=>scanner.Scanner**

**类和枚举说明:**

- **cmd.v.internal.compile.V**

  V编译器主类,由v命令行负责创建

  | 字段/方法           | 说明                                              |
  | ------------------- | ------------------------------------------------- |
  | os                  | 保存指定要编译的操作系统                          |
  | out_name_c          | 保存需要编译生成的C文件名                         |
  | out_name            | 保存需要生成的可执行文件名                        |
  | files               | 保存所有需要编译的源文件                          |
  | tables              | 对符号表对象的引用                                |
  | pref                | 对编译选项对象的引用                              |
  | module_lookup_paths | 保存模块搜索路径                                  |
  | ...                 |                                                   |
  | compile()           | 负责启动编译                                      |
  | cc()                | 负责调用C编译器,将生成的C源代码文件生成可执行文件 |
  | ...                 |                                                   |

  

- **pref.Preferences**

  V编译器选项类,保存着所有的编译器选项和参数,由V编译器负责创建

  参数太多,不一一展开,列举几个主要的:

  | 字段       | 说明             |
  | ---------- | ---------------- |
  | build_mode | 编译的模块       |
  | is_script  | 是否vscript脚本  |
  | is_prof    | 是否生产模式编译 |
  | is_run     | 是否v run执行    |
  | ....       |                  |

- **builder.Builder**

  代码生成器类,由V编译器负责创建

  | 字段/方法           | 说明                               |
  | ------------------- | ---------------------------------- |
  | pref                | 对编译选项对象的引用               |
  | tables              | 对符号表对象的引用                 |
  | checker             | 对代码检查器对象的引用             |
  | parsed_files        | 语法分析器生成的文件语法树对象数组 |
  | module_search_paths | 模块搜索路径                       |
  | ...                 |                                    |
  | build_c()           | 负责编译生成C源代码                |
  | build_js()          | 负责编译生成js源代码               |
  | build_x64           | 负责编译生成x64机器码              |
  | ...                 |                                    |

- **parser.Parser**

  语法分析器,由代码生成器负责创建

  | 字段/方法                                  | 说明                              |
  | ------------------------------------------ | --------------------------------- |
  | scanner                                    | 保存对应的词法扫描器引用          |
  | file_name                                  | 保存对应的源文件                  |
  | tables                                     | 对符号表对象的引用                |
  | pref                                       | 对编译选项对象的引用              |
  |                                            | 其他字段都是语法分析中的过程变量: |
  | pos                                        | 当前token                         |
  | peek_tok                                   | 下一个token                       |
  | scope    &ast.Scope                        | 当前作用域                        |
  | ...                                        |                                   |
  | parse_file(path string,table &table.Table) | 对一个源文件进行语法分析          |
  | parse_files(paths []string)                | 对一组源文件进行语法分析          |
  | ...                                        |                                   |

  

- **table.table**

  符号表,保存着语法分析后,得到的所有变量,函数,类型等

  | 字段/方法               | 说明                 |
  | ----------------------- | -------------------- |
  | types   []TypeSymbol    | 所有类型             |
  | local_vars              | 所有类型索引         |
  | fns    map[string]Fn    | 所有函数和方法       |
  | consts   map[string]Var | 所有常量             |
  | imports   []string      | 所有导入模块         |
  | modules   []string      | 所有模块             |
  |                         |                      |
  | register_const()        | 注册常量到符号表     |
  | register_global()       | 注册全局变量到符号表 |
  | register_var()          | 注册变量到符号表     |
  | register_fn()           | 注册函数到符号表     |
  | register_method()       | 注册方法到符号表     |
  | register_type()         | 注册类型到符号表     |
  | ...                     |                      |

- **scanner.Scanner**

  词法扫描器,由语法分析器负责创建,对源文件进行逐个字节进行扫描,识别出token

  | 字段/方法                   | 说明                                        |
  | --------------------------- | ------------------------------------------- |
  | file_path                   | 保存扫描的当前源文件路径                    |
  | text                        | 保存文件所有内容的字符串                    |
  | 其他字段全是扫描的过程变量: |                                             |
  | pos                         | 扫描到的当前字节位置                        |
  | line_nr                     | 扫描到的行                                  |
  | ...                         |                                             |
  | scan()                      | 启动一次扫描,返回一个token,由语法分析器调用 |
  | scan_res()                  | 返回一次扫描结果                            |
  | ...                         | 根据指定的源文件,创建扫描器                 |
  | ...                         |                                             |

- **token.Token**

  词法单元类,表示词法扫描器识别出来的一个token

  | 字段/方法  | 说明                                                         |
  | ---------- | ------------------------------------------------------------ |
  | kind       | token的种类                                                  |
  | lit        | string 字面量值,只有种类为name,number,str,str_inter,chartoken时才会有值 |
  | line_nr    | 行位置                                                       |
  | pos        | 列位置                                                       |
  | position() | 返回当前token的位置,行位置和列位置                           |

- **token.Kind**

  词法单元种类枚举,一共有139个可以被扫描器识别的词法单元

  | 枚举值            | 说明                                                         |
  | ----------------- | ------------------------------------------------------------ |
  | eof               | 表示文件结束                                                 |
  | name              | 标识符,tokon的lit字段有具体值                                |
  | number            | 数字,tokon的lit字段有具体值                                  |
  | str               | 字符串,tokon的lit字段有具体值                                |
  | chartoken         | 单字符,tokon的lit字段有具体值                                |
  | plus              | +加                                                          |
  | minus             | -减                                                          |
  | mul               | *乘                                                          |
  | div               | /除                                                          |
  | mod               | %余                                                          |
  | xor               | ^异或                                                        |
  | bit_not           | ~位取反                                                      |
  | pipe              | \|管道符                                                     |
  | hash              | # 用于C宏                                                    |
  | amp               | &位且,也用于变量取地址                                       |
  | inc               | ++递增                                                       |
  | dec               | --递减                                                       |
  | and               | &&逻辑且                                                     |
  | logical_or        | \|\|逻辑或                                                   |
  | not               | !逻辑非                                                      |
  | dot               | .点运算符                                                    |
  | dotdot            | ..运算符,用于数组区间                                        |
  | ellipsis          | ...展开符,用于函数不确定参数                                 |
  | comma             | ,逗号                                                        |
  | semicolon         | ;分号                                                        |
  | colon             | :冒号,用于字典初始化,结构体初始化,结构体字段访问控制,goto标签 |
  | arrow             | =>箭头                                                       |
  | assign            | =变量赋值/分配                                               |
  | decl_assign       | :=变量声明                                                   |
  | plus_assign       | +=加等于                                                     |
  | minus_assign      | -=减等于                                                     |
  | mult_assign       | *=乘等于                                                     |
  | div_assign        | /=除等于                                                     |
  | xor_assign        | ^=异或等于                                                   |
  | mod_assign        | %=余等于                                                     |
  | or_assign         | \|=位或等于                                                  |
  | and_assign        | &=位且等于                                                   |
  | righ_shift_assign | \>>=位右移等于                                               |
  | left_shift_assign | <<=位左移等于                                                |
  | lcbr              | {左大括号                                                    |
  | rcbr              | }右大括号                                                    |
  | lpar              | (左小括号                                                    |
  | rpar              | )右小括号                                                    |
  | lsbr              | [左中括号                                                    |
  | rsbr              | ]右中括号                                                    |
  | eq                | ==相等                                                       |
  | ne                | ≠不等于                                                      |
  | gt                | >大于                                                        |
  | lt                | <小于                                                        |
  | ge                | >=大于等于                                                   |
  | le                | <=小于等于                                                   |
  | question          | ?问号                                                        |
  | left_shift        | <<位左移                                                     |
  | right_shift       | >>位右移                                                     |
  | line_comment      | //单行注释                                                   |
  | mline_comment     | /*多行注释开头                                               |
  | nl                | NLL空值字符                                                  |
  | dollar            | $美金符号                                                    |
  | str_dollar        | $字符串内的美金符号                                          |
  | keyword_beg       | 表示在keyword_beg和keyword_end之间的都是关键字,本身无意义    |
  | key_assert        | 关键字assert                                                 |
  | key_struct        | 关键字struct                                                 |
  | key_if            | 关键字if                                                     |
  | key_else          | 关键字else                                                   |
  | key_asm           | 关键字asm                                                    |
  | key_return        | 关键字return                                                 |
  | key_module        | 关键字module                                                 |
  | key_sizeof        | 关键字sizeof                                                 |
  | key_go            | 关键字go                                                     |
  | key_goto          | 关键字goto                                                   |
  | key_const         | 关键字const                                                  |
  | key_mut           | 关键字mut                                                    |
  | key_type          | 关键字type                                                   |
  | key_for           | 关键字for                                                    |
  | key_switch        | 关键字switch                                                 |
  | key_fn            | 关键字fn                                                     |
  | key_true          | 关键字true                                                   |
  | key_false         | 关键字false                                                  |
  | key_continue      | 关键字continue                                               |
  | key_break         | 关键字break                                                  |
  | key_import        | 关键字import                                                 |
  | key_embed         | 关键字embed,未使用                                           |
  | key_unsafe        | 关键字unsafe                                                 |
  | key_typeof        | 关键字typeof                                                 |
  | key_enum          | 关键字enum                                                   |
  | key_interface     | 关键字interface                                              |
  | key_pub           | 关键字pub                                                    |
  | key_import_const  | 关键字import_const,用于导入C常量                             |
  | key_in            | 关键字in                                                     |
  | key_atomic        | 关键字atomic,未使用                                          |
  | key_orelse        | 关键字or                                                     |
  | key_global        | 关键字__global,全局变量                                      |
  | key_union         | 关键字union                                                  |
  | key_static        | 关键字static,用于C函数的static                               |
  | key_as            | 关键字as                                                     |
  | key_defer         | 关键字defer                                                  |
  | key_match         | 关键字match                                                  |
  | key_select        | 关键字select,用于db.select                                   |
  | key_none          | 关键字none                                                   |
  | key_offsetof      | 关键字__offsetof,未使用                                      |
  | keyword_end       | 表示在keyword_beg和keyword_end之间的都是关键字,本身无意义    |

- **table.Fn**
  
  函数类型
  
  | 字段/方法        | 说明               |
  | ---------------- | ------------------ |
  | name   string    | 函数名             |
  | args    []Var    | 函数的参数         |
| return_type Type | 函数的返回类型     |
  | is_variadic bool | 是不是可变参数函数 |
  
- **table.Var**
  
  变量和常量类型
  
  | 字段/方法       | 说明                 |
  | --------------- | -------------------- |
  | name   string   | 变量名               |
  | idx     int     | 变量在符号表中的索引 |
| is_mut  bool    | 是否可变             |
  | is_const bool   | 是否常量             |
  | is_global bool  | 是否全局变量         |
  | scope_level int | 作用域级别           |
  | typ    Type     | 变量类型             |
  
- **table.Field**
  
  结构体字段类型
  
  | 字段/方法   | 说明     |
  | ----------- | -------- |
  | name string | 字段名称 |
  | typ Type    | 字段类型 |

- **table.Scope**
  
  作用域
  | 字段/方法         | 说明           |
  | ----------------- | -------------- |
  | parent  &Scope    | 父作用域       |
  | children []&Scope | 子作用域       |
  | start_pos int     | 作用域开始位置 |
  | end_pos  int      | 作用域结束位置 |
  
- **table.TypeSymbol**
  
  类型,描述类型的类型
  
  | 字段/方法           | 说明                             |
  | ------------------- | -------------------------------- |
  | parent  &TypeSymbol | 类型的父类型                     |
  | kind    Kind        | 类型的种类                       |
| info    TypeInfo    | 类型的额外信息类,只对6个类型有用 |
  | name    string      | 类型的名字                       |
  
- **table.Type**

  类型引用,i32的类型别名,表示每一个类型在符号表中的唯一ID

  这个i32的ID有进行位移处理,前16位保存着类型的ID信息,后16保存着类型的指针引用层级信息

  ​	原来是设计为一个TypeRef结构体的,因为在符号表等很多地方都被引用,所以把ID和指针引用层级信息合并成一个int32,性能更好,占用内存更小.

  ```c
  pub type Type int
  ```

- **table.AccessMod**

  结构体字段的访问控制枚举

  | 字段/方法   | 说明                                        |
  | ----------- | ------------------------------------------- |
  | private     | 模块私有,且只读                             |
  | private_mut | 模块私有,但可变                             |
  | public      | 所有模块可访问,但只读                       |
  | public_mut  | 所有模块可访问,且可变                       |
  | global      | 全局字段,所有模块可访问,且可修改,不推荐使用 |

- **table.Kind**
  
  类型的种类,包含了V语言中的所有类型种类,包括基本类型,内置类型,结构体类型等其他种类,对类型进行分类以后,就可以针对不同的种类进行不同的语法分析,语法检查
  
  | 字段/方法    | 说明         |
  | ------------ | ------------ |
  | placeholder  | 占位种类     |
  | void         |              |
  | voidptr      | 通用指针类型 |
  | charptr      | 字符指针类型 |
  | byteptr      | 字节指针     |
  | i8           |              |
  | i16          |              |
  | int          |              |
  | i64          |              |
  | u16          |              |
  | u32          |              |
  | u64          |              |
  | f32          |              |
  | f64          |              |
  | string       | 字符串       |
  | char         | 单字符       |
  | byte         |              |
  | bool         |              |
  | const_       | 常量         |
  | enum_        | 枚举         |
  | struct_      | 结构体       |
  | array        | 数组         |
  | array_fixed  | 固定大小数组 |
  | map          | 字典         |
  | multi_return | 多返回值类型 |
  | sum_type     | 联合类型     |
  | alias        | 类型别名     |
  | unresolved   |              |
  
- **table.TypeInfo**

  类型信息类,该类型是一个联合类型,作为几种特殊类型的额外信息

  ```rust
  pub type TypeInfo = Array | ArrayFixed | Map | Struct | 	
  MultiReturn | Alias
  ```

- **table.Struct**
  
  结构体类型,TypeInfo联合类型的其中一种类型
  
  作为类型信息类的其中一种类型,要描述一个完整的结构体类型所包含的信息,就是TypeSymbol中的基本字段,再加上这个结构体中的字段
  
  | 字段/方法      | 说明             |
  | -------------- | ---------------- |
  | fields []Field | 结构体的所有字段 |
  
- **table.Array**

  数组类型,TypeInfo联合类型的其中一种类型

  | 字段/方法      | 说明     |
  | -------------- | -------- |
  | elem_type Type | 数组类型 |
  | nr_dims    int | 数组维度 |

- **table.ArrayFixed**
  
  固定大小数组类型,TypeInfo联合类型的其中一种类型
  
  | 字段/方法      | 说明     |
  | -------------- | -------- |
  | elem_type Type | 数组类型 |
  | nr_dims    int | 数组维度 |
  | size      int  | 数组大小 |
  
- **table.Map**
  
  字典类型,TypeInfo联合类型的其中一种类型
  
  | 字段/方法       | 说明     |
  | --------------- | -------- |
  | key_type  Type  | 键的类型 |
  | value_type Type | 值的类型 |
  
- **table.MultiReturn**
  
  多返回值类型,TypeInfo联合类型的其中一种类型
  
  V语言的多返回值实际上是返回一个结构体,动态生成的结构体,就是用这个来定义
  
  | 字段/方法    | 说明                   |
  | ------------ | ---------------------- |
  | name string  | 类型名字               |
  | types []Type | 包含多返回值的类型数组 |
  
- **table.Alias**
  
  类型别名,TypeInfo联合类型的其中一种类型
  
  | 字段/方法  | 说明     |
  | ---------- | -------- |
  | foo string | 类型别名 |
  
  
  
- **check.Checker**
  
  代码检查器,用于检查各种语法的合法性
  
  | 字段/方法              | 说明                           |
  | ---------------------- | ------------------------------ |
  | table   &table.Table   | 符号表的引用                   |
  | file      ast.File     | 要检查的文件语法树             |
| nr_errors int          | 错误个数                       |
  | errors    []string     | 错误内容数组                   |
  |                        |                                |
  | check_files()          | 检查多个文件语法树的语法合法性 |
  | check()                | 检查单个文件语法树             |
  | stmt()                 | 检查语句,递归检查              |
  | expr()                 | 检查表达式,递归检查            |
  | error()                | 没有通过检查项,报错            |
  | check_struct_init      | 检查结构体初始化的合法性       |
  | check_assign_expr      | 检查变量赋值的合法性           |
  | call_expr              | 检查函数调用                   |
  | check_method_call_expr | 检查方法调用                   |
  | return_stmt            | 检查返回语句                   |
  | array_init             | 检查数组初始化                 |
  | ...                    |                                |
  
- **gen.Gen**
  
  C源代码生成器
  
  | 字段/方法                   | 说明                                                    |
  | --------------------------- | ------------------------------------------------------- |
  | out   strings.Builder       | 生成的C源代码主体,保存在这个字符串中,字符串生成器对象   |
  | definitions strings.Builder | 生成的C源代码宏部分,保存在这个字符串中,字符串生成器对象 |
  |                             | 完整的C源代码由这两个部分组成:definitions+out           |
  | table    &table.Table       | 符号表的引用                                            |
  | fn_decl   &ast.FnDecl       | 过程变量,生成过程中的当前函数                           |
  | tmp_count   int             | 过程变量,用于生成临时变量的计数,每一个函数重置          |
  |                             |                                                         |
  | cgen() string               | 生成所有[ ]ast.File的C源代码                            |
  | stmts()                     | 遍历stmts生成所有的C代码                                |
  | stmt()                      | 根据一个stmt语句,生成对应的C代码段                      |
  | write() string              | 生成C代码段,不换行                                      |
  | writeln() string            | 生成C代码段,换行                                        |
  
- **gen.JsGen**

  js源代码生成器

  | 字段/方法            | 说明                                               |
  | -------------------- | -------------------------------------------------- |
  | out  strings.Builder | 生成的js源代码,保存在这个字符串中,字符串生成器对象 |
  | table &table.Table   | 符号表的引用                                       |

- **gen.x64.Gen**

  x64机器码生成器

  | 字段/方法                    | 说明                                               |
  | ---------------------------- | -------------------------------------------------- |
  | out_name       string        | 生成的机器代码,保存在这个字符串中,字符串生成器对象 |
  | buf         []byte           |                                                    |
  | sect_header_name_pos int     |                                                    |
  | offset        i64            |                                                    |
  | str_pos       []i64          |                                                    |
  | strings       []string       |                                                    |
  | file_size_pos    i64         |                                                    |
  | main_fn_addr     i64         |                                                    |
  | code_start_pos    i64        |                                                    |
  | fn_addr       map[string]i64 |                                                    |
  | gen()                        | 生成x64代码                                        |

- **fmt.Fmt**
  
  代码格式化器,根据文件语法树,生成格式工整的代码,然后再覆盖回未格式化的代码
  
  | 字段/方法              | 说明                                                    |
  | ---------------------- | ------------------------------------------------------- |
  | out    strings.Builder | 格式化后的代码字符串                                    |
  | table   &table.Table   | 符号表的引用                                            |
  | indent   int           | 格式化过程变量,记录当前代码缩进的等级                   |
  | empty_line bool        | 格式化过程变量,记录当前行是否是空行,如果是,要先生成缩进 |
  | line_len       int     | 格式化过程变量,记录当前行目前的长度,为了处理80换行      |
  | single_line_if bool    | 格式化过程变量,判断是否单行if,为了格式化成1行           |
  | cur_mod        string  | 格式化过程变量,记录当前模块                             |
  |                        |                                                         |
  | fmt()                  | 格式化代码某个文件                                      |
  | wrtie()                | 往out字符串写入代码                                     |
  | writeln()              | 往out字符串写入代码,并换行                              |
  | stmts()                | 格式化多个语句                                          |
  | stmt()                 | 格式化单个语句                                          |
  

### AST语法树类

源代码文件经过词法扫描,语法分析后的输出就是文件语法树对象数组([ ]ast.File)

以下类图就是文件语法树中每一个节点对应的类型

类型中有2个基本的类型:**Stmt语句**类型和**Expr表达式**类型,这两个类型是联合类型,联合类型的含义就是这种类型可以是图中箭头关联的子类型的其中一种,有点像继承关系中基类的作用,但又不是

而且Expr表达式类型通过ExprStmt表达式语句类型的关联,也是Stmt语句类型的子类,这样一来,所有的语法树节点都是Stmt语句类型的子类

文件语法树中以ast.File为根节点,节点包含子节点,形成一棵完整的语法树

![](/../content/compiler.assets/V语法树类.jpg)

联合类型Stmt和Expr,对应的V源代码:

```rust
pub type Expr = InfixExpr | IfExpr | StringLiteral | IntegerLiteral | CharLiteral | 	
FloatLiteral | Ident | CallExpr | BoolLiteral | StructInit | ArrayInit | SelectorExpr | PostfixExpr | 	
AssignExpr | PrefixExpr | MethodCallExpr | IndexExpr | RangeExpr | MatchExpr | 	
CastExpr | EnumVal | Assoc | SizeOf

pub type Stmt = VarDecl | GlobalDecl | FnDecl | Return | Module | Import | ExprStmt | 	
ForStmt | StructDecl | ForCStmt | ForInStmt | CompIf | ConstDecl | Attr | BranchStmt | 	
HashStmt | AssignStmt | EnumDecl | TypeDecl | DeferStmt | GotoLabel | GotoStmt | 	
LineComment | MultiLineComment
```

主要类别说明:

- ast.File

  文件语法树类,保存了一个v源文件的整棵语法树

| 字段/方法        | 说明              |
| ---------------- | ----------------- |
| path  string     | 对应的v源文件路径 |
| mod   Module     | 模块节点          |
| imports []Import | 导入模块节点      |
| stmts  []Stmt    | 语句数组          |
| scope   &Scope   | 文件作用域        |

- ast.Scope

  作用域节点,整个ast.File里也是保存了一棵作用域树,以ast.File的scope为根节点

| 字段/方法                | 说明                       |
| ------------------------ | -------------------------- |
| parent  &Scope           | 父作用域                   |
| children []&Scope        | 子作用域数组               |
| start_pos int            | 作用域开始位置             |
| end_pos  int             | 作用域结束位置             |
| vars  map[string]VarDecl | 该作用域内,声明的变量字典  |
| register_var()           | 注册声明的变量             |
| find_var()               | 在作用域内查找已声明的变量 |
| override_var()           | 覆盖变量                   |

作用域示意图:

![](/../content/compiler.assets/image-20200221225909856.png)

- 其他语法树节点,不详细展开,参考类图



可以使用vast这个工具,从V源文件生成AST json文件,这样就可以直观地看到AST:

```
https://github.com/lydiandy/vast
```

以下仅是举例,生成的语法树JSON方式表示如下:

```json
/main.v生成的语法树为例
{
  "path": "path/to/main.v",
  "mod": {
    //Module类型
    "name": "main",
    "path": "path/to/main.v",
    "expr": {}
  },
  "imports": [
    //[]Import类型
    { "mod": "os", "alias": "", "pos": { "line_nr": 2 } },
    { "mod": "strings", "alias": "", "pos": { "line_nr": 3 } },
    { "mod": "time", "alias": "", "pos": { "line_nr": 4 } }
  ],
  "stmts": [
    //[]Stmt联合类型
    {
      //ConstDecl常量声明
      "fields": [
        {
          "name": "pi",
          "typ": { "idx": 12, "type": "0x13244344", "nr_muls": 0 }
        },
        {
          "name": "my_const",
          "typ": { "idx": 12, "type": "0x13244344", "nr_muls": 0 }
        },
        {
          "name": "my_int_const",
          "typ": { "idx": 12, "type": "0x13244344", "nr_muls": 0 }
        }
      ],
      "exprs": [
        //[]Expr联合类型
        {
          //FloatLiteral小数字面量类型
          "val": "3.14"
        },
        {
          //StringLiteral字符串字面量类型
          "val": "my value"
        },
        {
          //IntegerLiteral整数字面量类型
          "val": 10
        }
      ]
    },
    {
      //FnDecl函数声明
      "name": "main",
      "is_pub": "true",
      "is_variadic": "false",
      "receiver": {},
      "typ": { "idx": 12, "type": "0x13244344", "nr_muls": 0 },
      "args": [
        {
          "name": "a",
          "typ": { "idx": 12, "type": "0x13244344", "nr_muls": 0 }
        },
        {
          "name": "b",
          "typ": { "idx": 12, "type": "0x13244344", "nr_muls": 0 }
        }
      ],
      "stmts": [
        //函数体语句数组,一直往下嵌套语句,就是一棵函数体的语法树
        //[]Stmt联合类型,函数体的语句

        {
          //VarDecl变量声明
          "name": "x",
          "is_mut": "true",
          "type": { "idx": 12, "type": "0x13244344", "nr_muls": 0 },
          "expr": {
            //Expr联合类型,这里是StringLiteral字符串字面量类型
            "val": "abc"
          },
          "pos": { "line_nr": "4" }
        },
        {
          //IfExpr条件语句
        }
      ]
    }
  ]
}

```

---

### 词法扫描器-扫描顺序

以下是扫描器扫描识别token的逻辑:

- s.text:  字符串类型,就是源代码的字符串表示

- s.pos:  扫描的当前字节位置

- s.scan()函数:  每调用一次,就接着s.pos当前位置,继续往下扫描,识别到一个token后返回

![](/../content/compiler.assets/image-20200220173148156.png)

以下是扫描器碰到每一个字节时,进行扫描和识别的顺序:

|      当前字符      | 识别内容                                      | 继续识别         |
| :----------------: | :-------------------------------------------- | :--------------- |
| 字符值为9/10/13/32 | 优先识别tab制表符/换行/回车/空格,跳过这些空格 |                  |
|    字母/下划线     | name名字                                      |                  |
|                    |                                               | keyword关键字    |
|   数字/点加数字    | number数字                                    |                  |
|                    |                                               | 0b 二进制数字    |
|                    |                                               | 0x十六进制数字   |
|                    |                                               | 0八进制数字      |
|                    |                                               | 1-9/0.十进制数字 |
|         )          | )                                             |                  |
|         +          | ++                                            |                  |
|                    | +=                                            |                  |
|                    | +                                             |                  |
|         -          | --                                            |                  |
|                    | -=                                            |                  |
|                    | -                                             |                  |
|         *          | *=                                            |                  |
|                    | *                                             |                  |
|         ^          | ^=                                            |                  |
|                    | ^                                             |                  |
|         %          | %=                                            |                  |
|                    | %                                             |                  |
|         ?          | ?                                             |                  |
|        '或"        | string字符串                                  |                  |
|         \`         | char单字符                                    |                  |
|         (          | (                                             |                  |
|         )          | )                                             |                  |
|         [          | [                                             |                  |
|         ]          | ]                                             |                  |
|         {          | {                                             |                  |
|                    | 字符串内递归识别                              |                  |
|         $          | $                                             |                  |
|                    | 字符串内递归识别                              |                  |
|         }          | }                                             |                  |
|                    | 字符串内递归识别                              |                  |
|         \|         | \|=                                           |                  |
|                    | \|\|                                          |                  |
|         ,          | ,                                             |                  |
|         @          | FN                                            |                  |
|                    | FILE                                          |                  |
|                    | LINE                                          |                  |
|                    | COLUMN                                        |                  |
|                    | VHASH                                         |                  |
|                    | @keyword                                      |                  |
|                    | name名字                                      |                  |
|         .          | ...                                           |                  |
|                    | ..                                            |                  |
|                    | .                                             |                  |
|         #          | #!                                            |                  |
|                    | #宏定义                                       |                  |
|         >          | >=                                            |                  |
|                    | >>                                            |                  |
|                    | >                                             |                  |
|        0xE2        | ≠                                             |                  |
|                    | ≤                                             |                  |
|                    | ≥                                             |                  |
|         <          | <=                                            |                  |
|                    | <<                                            |                  |
|                    | <                                             |                  |
|         =          | ==                                            |                  |
|                    | =>                                            |                  |
|                    | =                                             |                  |
|         :          | :=                                            |                  |
|                    | :                                             |                  |
|         ;          | ;                                             |                  |
|         !          | !=                                            |                  |
|                    | !                                             |                  |
|         ~          | ~                                             |                  |
|         /          | /=                                            |                  |
|                    | //                                            |                  |
|                    | /*                                            |                  |
|         \0         | .eof   windows end of file                    |                  |
|   以上都判断完了   | 如果还有不能识别的,就是invalid character      |                  |
|  s.pos≥s.text.len  | .eof   就是文件结束end of file                |                  |

---

###语法分析器-分析顺序

语法分析器从p.parse_file()或者p.parse_files()开始启动

调用p.read_first_token()进行初始化后,p.tok和p.peek_tok就位,从第一个token开始分析

- p.tok:  当前token

- p.peek_tok:  下一个token

- p.next():  每调用一次,就调用一次扫描器的scan(),返回一个token,并将p.tok向后推进一个

- p.check(token):

  ​	检查p.tok是否为指定的token

  ​	如果是就调用p.next(),将p.tok向后推进一个

  ​	如果不是就报语法错误

- p.check_name() string:

  ​	检查p.tok是否为.name(标识符)的token

  ​	如果是就返回tok.lit(具体的标识名),并调用p.next(),将p.tok向后推进一个
  
  ​	如果不是就报语法错误

![](/../content/compiler.assets/image-20200220183359491.png)

以下是语法分析的分析顺序:

| 当前token | 下个token  | 识别内容(ast中的节点类型) | 说明                             |
| :-------: | :--------: | :------------------------ | -------------------------------- |
|    //     |            | LineComment               | 识别module之前的行注释(未实现)   |
|    /*     |            | MultiLineComment          | 识别module之前的多行注释(未实现) |
|  module   |            | Module                    | 识别模块定义                     |
|  import   |            | [ ]Import                 | 识别导入语句                     |
| 以下开始  | 识别所有的 | 顶级节点                  | 遍历识别后,形成[ ]stmts          |
|    pub    |            |                           | 识别以下公共的顶级节点           |
|           |   const    | ConstDecl                 | 识别一组常量声明                 |
|           |     fn     | FnDecl                    | 识别函数声明或方法声明           |
|           |   struct   | StructDecl                | 识别结构体声明                   |
|           |   union    |                           | 识别C联合类型(未实现)            |
|           | interface  |                           | 识别接口声明(未实现)             |
|           |    enum    | EnumDecl                  | 识别枚举声明                     |
|           |    type    | TypeDecl                  | 识别类型别名/联合类型            |
|     [     |            | Attr                      | 识别函数标注                     |
| __global  |            | GlobalDecl                | 识别全局变量                     |
|   const   |            | ConstDecl                 | 识别一组常量声明                 |
|    fn     |            | FnDecl                    | 识别函数声明或方法声明           |
|  struct   |            | StructDecl                | 识别结构体声明                   |
|     $     |            | CompIf                    | 识别条件编译语句                 |
|     #     |            | HashStmt                  | 识别C宏                          |
|   type    |            | TypeDecl                  | 识别类型别名/联合类型            |
|   enum    |            | EnumDecl                  | 识别枚举声明                     |
|    //     |            | LineComment               | 识别单行注释                     |
|    /*     |            | MultiLineComment          | 识别多行注释                     |

如果还存在以上都不是的顶级节点,  就报不合法的顶级节点错误

---

以下为顶级节点中包含的各种下级token的语法分析:

| 当前token | 下个token | 识别内容(ast中的节点类型) | 说明 |
| ---- | ---- | ---- | ---- |
| [ ]stmts  |           | StmtBlock代码块语句        | 通过p.stmt()的递归调用进行分析   |
|    mut    |           | VarDecl                    | 识别变量声明语句                 |
|    for    |           | ForCStmt/ForStmt/ForInStmt | 识别3种for循环语句               |
|  return   |           | Return                     | 识别函数返回语句                 |
|     $     |           | CompIf                     | 识别条件编译语句                 |
| continue  |           | BranchStmt                 | 识别continue语句                 |
|   break   |           | BranchStmt                 | 识别break语句                    |
|  unsafe   |           |                            | 识别unsafe代码块语句(未实现)     |
|   defer   |           | DeferStmt                  | 识别defer代码块语句              |
|   goto    |           | GotoStmt                   | 识别goto代码块语句               |
|   .name   |     :     | GotoLabel                  | 识别为goto标签语句               |
|   .name   |     :     | AssignStmt                 | 识别为分配语句                   |
|  |  |  |  |
| = |           | Expr,识别表达式的值和类型           | 识别表达式,最复杂的              |
| | .name |  | 继续识别出现名字的各种情况 |
| | .str | StringLiteral, table.string_type | 识别字符串 x='abc' |
| | .dot | EnumVal, table.int_type | 识别枚举值 x=.blue |
| | .chartoken | CharLiteral, table.byte_type | 识别单字符 x=`c` |
| | .key_true, .key_false | BoolLiteral,table.bool_type | 识别布尔类型 x=true |
| | .minus, .amp, .mul, .not, .bit_not | PrefixExpr | 识别前缀表达式 |
| | .key_match | MatchExpr | 识别match赋值语句 |
| | .number | IntegerLiteral/FloatLiteral | 识别数值 |
| | .lpar |  | 继续递归识别表达式 |
| | .key_if | IfExpr | 识别if赋值语句 |
| | .lsbr | ArrayInit | 识别数组初始化语句 |
| | .key_none | None, table.none_type | 识别none类型 |
| | .key_sizeof | SizeOf,table.int_type | 识别sizeof()函数 |
| | .lcbr | | 继续识别各种情况 |
| | 如果以上都不是 | 报错:无效表达式 |  |
| |  |  |  |
| |  |  |  |



### 语法检查器-检查项



### 代码生成器

