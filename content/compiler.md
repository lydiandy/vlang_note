---
typora-root-url: ../image
---

## V编译器源代码

作者在发布0.2版本之前,重构了编译器,改用基于AST抽象语法树的编译方式,代码也更为清晰易读

### 代码目录

V命令行代码位于cmd目录

| 子目录 | 说明                                            |
| ------ | ----------------------------------------------- |
| v      | V命令行,入口文件v.v,编译成V可执行文件           |
| tools  | 各种工具类,如vup,vfmt,vpm,vrepl,vtest,vcreate等 |

编译器代码位于vlib/v目录

| 子目录  | 说明                                |
| ------- | ----------------------------------- |
| ast     | AST抽象语法树相关                   |
| token   | 词法单元相关                        |
| table   | 符号表相关                          |
| prep    | 编译选项/参数相关                   |
| scanner | 词法分析/扫描器相关                 |
| parser  | 语法分析/解析器相关                 |
| checker | 语法检查器相关                      |
| eval    | 表达式求值相关                      |
| builder | 代码生成器相关                      |
| gen     | 生成具体平台代码相关,如生成C,js,x64 |
| fmt     | 代码格式化相关                      |

| 子目录               | 说明                                                     |
| -------------------- | -------------------------------------------------------- |
| vlib/compiler/main.v | 目前V编译器类所在目录,新版编译器完成后应该会移到别的目录 |



### 主要编译过程

- V命令行的入口文件是v/cmd/v/v.v

  .			 		

- V命令行负责处理命令参数,根据参数创建V编译器对象(compiler.V),或调用tools中的各种工具

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

- 语法分析和词法扫描实际上是同步进行,同步完成的.语法分析器对[ ]File进行遍历,以词法单元(token)为基本单位,进行语法分析,词法单元由词法扫描器负责识别,扫描,提供给语法分析器

   .			

- 语法分析遍历[ ]File的所有源文件后,生成对应的文件语法树对象数组[ ]ast.File

   .			

- 语法分析器分析完[ ]File源文件后,接着对所有导入的依赖,再次进行分析,将分析结果追加到文件语法树对象数组([ ]ast.File)中

   .			

- 语法分析器对象分析完成后,输出[ ]ast.File给代码生成器对象,由代码生成器对象,调用指定平台的代码生成对象(gen.Gen/gen.JsGen/gen.x64Gen)来生成目标代码,目前主要是调用gen.Gen来生成C源文件

  .	 	

- 最后调用v.cc方法,调用编译器参数指定的C编译器,将C源文件编译生成可执行文件,完成整个编译过程

  .  		

   **简单总结整个编译过程就是:[ ]File => [ ]ast.File => C源代码 => 可执行文件**



### 编译器类

以下类图是V编译器中主要的类和枚举:

![](/../content/compiler.assets/V编译器类.jpg)

主要的调用关系是:compiler.V=>builder.Builder=>parser.Parser=>scanner.Scanner

类和枚举说明:

- **compiler.V**

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
  | compile()           | 负责启动编译                                      |
  | cc()                | 负责调用C编译器,将生成的C源代码文件生成可执行文件 |

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

  | 字段/方法    | 说明                                   |
  | ------------ | -------------------------------------- |
  | pref         | 对编译选项对象的引用                   |
  | tables       | 对符号表对象的引用                     |
  | checker      | 对代码检查器对象的引用                 |
  | parsed_files | 保存语法分析器生成的文件语法树对象数组 |
  | build_c()    | 负责编译生成C源代码                    |
  | build_js()   | 负责编译生成js源代码                   |
  | build_x64    | 负责编译生成x64机器码                  |

- **parser.Parser**

  语法分析器,由代码生成器负责创建

  | 字段/方法     | 说明                              |
  | ------------- | --------------------------------- |
  | scanner       | 保存对应的词法扫描器引用          |
  | file_name     | 保存对应的源文件                  |
  | tables        | 对符号表对象的引用                |
  | pref          | 对编译选项对象的引用              |
  |               | 其他字段都是语法分析中的过程变量: |
  | pos           | 当前token                         |
  | peek_pos      | 下一个token                       |
  | parse_file()  | 负责语法分析一个源文件            |
  | parse_files() | 负责语法分析一组源文件            |

- **table.table**

  符号表,保存着语法分析后,得到的所有变量,函数,类型等

  | 字段/方法         | 说明                 |
  | ----------------- | -------------------- |
  | types             | 所有类型             |
  | local_vars        | 所有局部变量         |
  | fns               | 所有函数和方法       |
  | consts            | 所有常量             |
  | imports           | 所有导入模块         |
  | modules           | 所有模块             |
  | register_const()  | 注册常量到符号表     |
  | register_global() | 注册全局变量到符号表 |
  | register_var()    | 注册变量到符号表     |
  | register_fn()     | 注册函数到符号表     |
  | register_method() | 注册方法到符号表     |
  | register_type()   | 注册类型到符号表     |

- **scanner.Scanner**

  词法扫描器,由语法分析器负责创建,对源文件进行逐个字节进行扫描,识别出token

  | 字段/方法  | 说明                                                |
  | ---------- | --------------------------------------------------- |
  | file_path  | 保存扫描的当前源文件路径                            |
  | text       | 保存文件所有内容的字符串                            |
  |            | 其他字段全是扫描的过程变量:                         |
  | pos        | 扫描到的当前字节位置                                |
  | scan()     | 启动一次扫描,一次扫描返回一个token,给语法分析器使用 |
  | scan_res() | 返回一次扫描结果                                    |

- **token.Token**

  词法单元类,表示词法扫描器识别出来的一个token

  | 字段/方法  | 说明                                                         |
  | ---------- | ------------------------------------------------------------ |
  | kind       | token的种类                                                  |
  | lit        | string 字面量值,只有种类为name,number,str,str_inter,chartoken时才会有值 |
  | line_nr    | 第几行                                                       |
  | position() | 返回当前token的位置,第几行                                   |

- **token.Kind**

  词法单元种类枚举,一共有139个可以被扫描器识别的词法单元

  | 枚举值                   | 说明         |
  | ------------------------ | ------------ |
  | eof                      | 表示文件结束 |
  | name                     | 标识符       |
  | number                   | 数字         |
  | str                      | 字符串       |
  | ...枚举值太多,不详细展开 |              |

- **table.Fn**
  
  函数类型
  
  | 字段/方法 | 说明 |
  | --------- | ---- |
  |           |      |
  |           |      |
|           |      |
  |           |      |
  
- **table.Var**
  
  变量和常量类型
  
  | 字段/方法 | 说明 |
  | --------- | ---- |
  |           |      |
  |           |      |
|           |      |
  |           |      |
  
- **table.Field**
  
  结构体字段类型
  
  | 字段/方法 | 说明 |
  | --------- | ---- |
  |           |      |
  |           |      |
|           |      |
  |           |      |
  
- **table.Scope**
  
  作用域
  | 字段/方法 | 说明 |
  | --------- | ---- |
  |           |      |
  |           |      |
  |           |      |
  |           |      |
  
- **table.TypeSymbol**
  
  类型
  
  | 字段/方法 | 说明 |
  | --------- | ---- |
  |           |      |
  |           |      |
|           |      |
  |           |      |
  
- **table.AccessMod**

  结构体字段的访问控制枚举

  | 字段/方法 | 说明 |
  | --------- | ---- |
  |           |      |
  |           |      |
  |           |      |
  |           |      |

- **table.Kind**
  
  类型种类,包含了V语言中的所有类型种类,包括基本类型,内置类型,结构体类型等其他种类
  
  | 字段/方法 | 说明 |
  | --------- | ---- |
  |           |      |
  |           |      |
  |           |      |
  |           |      |
  
- **table.TypeInfo**

  类型信息类,该类型是一个联合类型,作为几种特殊类型的额外信息

  | 字段/方法 | 说明 |
  | --------- | ---- |
  |           |      |
  |           |      |
  |           |      |
  |           |      |

- **table.Struct**
  
  结构体类型,TypeInfo联合类型的其中一种类型
  
  作为类型信息类的其中一种类型,要描述一个完整的结构体类型所包含的信息,就是TypeSymbol中的基本字段,再加上这个结构体中的字段
  
  | 字段/方法 | 说明 |
  | --------- | ---- |
  |           |      |
  |           |      |
  |           |      |
  |           |      |
  
- **table.Array**

  数组类型,,TypeInfo联合类型的其中一种类型

  | 字段/方法 | 说明 |
  | --------- | ---- |
  |           |      |
  |           |      |
  |           |      |
  |           |      |

- **table.ArrayFixed**
  
  固定大小数组类型,TypeInfo联合类型的其中一种类型
  
  | 字段/方法 | 说明 |
  | --------- | ---- |
  |           |      |
  |           |      |
  |           |      |
  |           |      |
  
- **table.Map**
  
  字典类型,TypeInfo联合类型的其中一种类型
  
  | 字段/方法 | 说明 |
  | --------- | ---- |
  |           |      |
  |           |      |
  |           |      |
  |           |      |
  
- **table.MultiReturn**
  
  多返回值类型,TypeInfo联合类型的其中一种类型
  
  V语言的多返回值实际上是返回一个结构体,动态生成的结构体,就是用这个来定义
  
  | 字段/方法 | 说明 |
  | --------- | ---- |
  |           |      |
  |           |      |
  |           |      |
  |           |      |
  
- **table.Alias**
  
  类型别名,TypeInfo联合类型的其中一种类型
  
  | 字段/方法 | 说明 |
  | --------- | ---- |
  |           |      |
  |           |      |
  |           |      |
  |           |      |
  
- **check.Checker**
  
  代码检查器,用于检查各种语法的合法性
  
  | 字段/方法 | 说明 |
  | --------- | ---- |
  |           |      |
  |           |      |
|           |      |
  |           |      |
  
- **gen.Gen**
  
  C源代码生成器
  
  | 字段/方法 | 说明 |
  | --------- | ---- |
  |           |      |
  |           |      |
  |           |      |
  |           |      |
  
- **gen.JsGen**

  js源代码生成器

  | 字段/方法 | 说明 |
  | --------- | ---- |
  |           |      |
  |           |      |
  |           |      |
  |           |      |

- **gen.x64.Gen**

  x64机器码生成器

  | 字段/方法 | 说明 |
  | --------- | ---- |
  |           |      |
  |           |      |
  |           |      |
  |           |      |

- **fmt.Fmt**
  
  代码格式化类型,负责源代码格式化
  
  | 字段/方法 | 说明 |
  | --------- | ---- |
  |           |      |
  |           |      |
  |           |      |
  |           |      |
  

### AST语法树类

源代码文件经过词法扫描,语法分析后的输出就是文件语法树对象数组([ ]ast.File)

以下类图就是文件语法树中每一个节点对应的类型

类型中有2个基本的类型:**Stmt语句**类型和**Expr表达式**类型,这两个类型是联合类型,联合类型的含义就是这种类型可以是图中箭头关联的子类型的其中一种,有点像继承关系中基类的作用,但又不是

而且Expr表达式类型通过ExprStmt表达式语句类型的关联,也是Stmt语句类型的子类,这样一来,所有的语法树节点都是Stmt语句类型的子类

文件语法树中以ast.File为根节点,节点包含子节点,形成一棵完整的语法树

![](/../content/compiler.assets/V语法树类.jpg)

对应的V源代码:

```rust
pub type Expr = InfixExpr | IfExpr | StringLiteral | IntegerLiteral | 
CharLiteral |FloatLiteral | Ident | CallExpr | BoolLiteral | StructInit | 
ArrayInit | SelectorExpr | PostfixExpr |
AssignExpr | PrefixExpr | MethodCallExpr | IndexExpr | RangeExpr | 
MatchExpr | CastExpr | EnumVal

pub type Stmt = VarDecl | GlobalDecl | FnDecl | Return | Module | Import 
| ExprStmt | ForStmt | StructDecl | ForCStmt | ForInStmt | CompIf | 
ConstDecl | Attr | BranchStmt | HashStmt | AssignStmt | EnumDecl | 
TypeDecl | DeferStmt
```

实际代码生成的语法树对象实在是太大,嵌套层级多

可以对着V源代码,按着语法分析器的分析方式,想象生成的语法树对象

以下仅是举例,生成的语法树JSON方式表示如下:

```json
/main.v生成的语法树为例
{
  "path": "path/to/main.v",
  "module": {
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





