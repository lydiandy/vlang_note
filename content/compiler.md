---
typora-root-url: ../image
---

## V编译器源代码

作者在发布0.2版本之前,重构了编译器,改用基于AST的编译方式,代码也更为清晰易读

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

### 主要编译过程

- V命令行的入口文件是:v/cmd/v.v
- V命令行负责处理命令参数,根据参数创建V编译器对象,或者调用tools中的各种工具
- V编译器对象(compiler.V)创建后,首先根据参数,创建编译器参数对象(pref.Preferences)
- V编译器对象根据编译参数,分析得到所有需要编译的源文件数组[ ]File,源文件列表也包括了vlib/builtin中的内置源文件
- 通过调用v.compile()方法,开始编译,首先创建代码生成器对象(builder.Builder)
- 通过调用代码生成器对象的b.buildc()方法,把源代码文件数组[]File中的所有V源代码文件生成C源文件
- 代码生成器对象负责创建语法分析器对象(parser.Parser),启动语法分析
- 语法分析器负责创建词法扫描器对象(scanner.Scanner),启动词法扫描
- 语法分析和词法扫描实际上是同步进行,同步完成的.语法分析器对[ ]File进行遍历,以词法单元(token)为基本单位,进行语法分析,词法单元由词法扫描器负责识别,扫描,提供给语法分析器
- 词法分析遍历[ ]File的所有源文件后,生成对应的文件语法树对象数组[ ]ast.File
- 词法分析器分析完[ ]File源文件后,接着对所有导入的依赖,再次进行分析,将分析结果追加到文件语法树对象数组([ ]ast.File)中
- 语法分析器对象分析完成后,输出[ ]ast.File给代码生成器对象,由代码生成器对象,调用指定平台的代码生成对象(gen.Gen/gen.JsGen/gen.x64Gen)来生成目标代码,目前主要是调用gen.Gen来生成C源文件
- 最后调用v.cc方法,调用编译器参数指定的C编译器,将C源文件编译生成可执行文件,完成整个编译过程

   **简单总结整个编译过程就是:[ ]File => [ ]ast.File => C源代码 => 可执行文件**

### 编译器命令行类





### 编译器类

以下类图是V编译器中主要的类和枚举:

![](/../content/compiler.assets/V编译器类.jpg)

### AST语法树类

源代码文件经过词法扫描,语法分析后的输出就是文件语法树对象数组([ ]ast.File)

以下类图就是文件语法树中每一个节点对应的类型

类型中有2个基本的类型:**Stmt语句**类型和**Expr表达式**类型,这两个类型是联合类型,联合类型的含义就是这种类型可以是图中箭头关联的子类型的其中一种,有点像继承关系中基类的作用,但又不是

而且Expr表达式类型通过ExprStmt表达式语句类型的关联,也是Stmt语句类型的子类,这样一来,所有的语法树节点都是Stmt语句类型的子类

文件语法树中以ast.File为根节点,节点包含子节点,形成一棵完整的语法树

![](/../content/compiler.assets/V语法树类.jpg)

对应的V源代码:

```c
pub type Expr = InfixExpr | IfExpr | StringLiteral | IntegerLiteral | CharLiteral | FloatLiteral | Ident | CallExpr | BoolLiteral | StructInit | ArrayInit | SelectorExpr | PostfixExpr | AssignExpr | PrefixExpr | MethodCallExpr | IndexExpr | RangeExpr | MatchExpr | CastExpr | EnumVal

pub type Stmt = VarDecl | GlobalDecl | FnDecl | Return | Module | Import | ExprStmt | ForStmt | StructDecl | ForCStmt | ForInStmt | CompIf | ConstDecl | Attr | BranchStmt | HashStmt | AssignStmt | EnumDecl | TypeDecl | DeferStmt
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





