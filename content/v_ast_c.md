## V/AST/C代码对照

V语言编译器的基本思路:V代码=>AST语法树=>C代码/js代码/...

### 源文件

编译单个V源文件生成ast.File,编译一组源文件,生成[ ]ast.File

```v
//AST代码
pub struct File {
pub:
	path         string
	mod          Module
	global_scope &Scope
pub mut:
	scope        &Scope
	stmts        []Stmt //整个源文件的AST都在这里
	imports      []Import
	errors       []errors.Error
	warnings     []errors.Warning
}
```

### 模块

#### 模块定义

```v
//V代码
module main
fn main() {
  println('from main')
}
```

```v
//AST代码
pub struct Module {
pub:
	name       string
	path       string
	expr       Expr
	pos        token.Position
	is_skipped bool // module main can be skipped in single file programs
}
//AST实例
{
			"ast_type":	"Module",
			"name":	"main",
			"path":	"",
			"expr":	"",
			"is_skipped":	false,
			"pos":	{
				"line_nr":	0,
				"pos":	0,
				"len":	19
			}
}
```

V模块,生成C代码后,只是对应元素名称的前缀,毕竟C语言中没有模块的概念

常量,结构体,接口,类型等一级元素生成C代码后的名称规则是:"模块名__名称",用双下划线区隔

比如mymodule模块中的add()函数生成C代码后的名称为:mymodule__add()

结构体的方法等二级元素生成C代码后的名称规则是:"模块名 __ 类名 _ 方法名",用单下划线区隔

比如mymodule模块中的Color结构体的str()方法生成C代码后的名称为:mymodule__Color_str()

```v
//C代码
static void main__main();
static void main__main() {
	println(tos_lit("from main"));
}
```

#### 导入模块

```v
//V代码
import os
```

```v
//AST代码
pub struct Import {
pub:
	pos   token.Position
	mod   string
	alias string
pub mut:
	syms  []ImportSymbol
}
//AST实例
{
			"ast_type":	"Import",
			"mod":	"os",
			"alias":	"os",
			"syms":	[],
			"pos":	{
				"line_nr":	2,
				"pos":	20,
				"len":	2
			}
}
```



### 常量

#### 常量定义



### 枚举

#### 枚举定义

```v
//V代码
enum Color {
	white = 2
	black
	blue
}
```

```v
//AST代码
pub struct EnumField {
pub:
	name     string
	pos      token.Position
	comments []Comment
	expr     Expr
	has_expr bool
}

pub struct EnumDecl {
pub:
	name             string
	is_pub           bool
	is_flag          bool // true when the enum has [flag] tag
	is_multi_allowed bool
	comments         []Comment // enum Abc { /* comments */ ... }
	fields           []EnumField
	attrs            []table.Attr
	pos              token.Position
}
//AST实例
{
			"ast_type":	"EnumDecl",
			"name":	"main.Color",
			"is_pub":	false,
			"is_flag":	false,
			"is_multi_allowed":	false,
			"pos":	{
				"line_nr":	9,
				"pos":	59,
				"len":	10
			},
			"fields":	[{
					"ast_type":	"EnumField",
					"name":	"white",
					"has_expr":	true,
					"expr":	{
						"ast_type":	"IntegerLiteral",
						"val":	"2",
						"pos":	{
							"line_nr":	10,
							"pos":	81,
							"len":	1
						}
					},
					"pos":	{
						"line_nr":	10,
						"pos":	73,
						"len":	5
					},
					"comments":	[]
				}, {
					"ast_type":	"EnumField",
					"name":	"black",
					"has_expr":	false,
					"expr":	"",
					"pos":	{
						"line_nr":	11,
						"pos":	84,
						"len":	5
					},
					"comments":	[]
				}, {
					"ast_type":	"EnumField",
					"name":	"blue",
					"has_expr":	false,
					"expr":	"",
					"pos":	{
						"line_nr":	12,
						"pos":	91,
						"len":	4
					},
					"comments":	[]
				}],
			"comments":	[],
			"attrs":	[]
		}
```

```c
//C代码
typedef enum {
	main__Color_white = 2, // 2
	main__Color_black, // 2+1
	main__Color_blue, // 2+2
} main__Color;
```



### 内置类型

#### 数组

#### 字典

### 函数

#### 函数定义

#### 函数调用

#### 函数多返回值



### 流程控制

#### 条件

#### 匹配

#### 循环

### 结构体

#### 结构体定义

#### 结构体初始化

#### 结构体方法

```v
//V代码

```



### 类型

#### 类型别名

```v
//V代码
type Myint = int
```

```v
//AST代码
pub struct AliasTypeDecl {
pub:
	name        string
	is_pub      bool
	parent_type table.Type
	pos         token.Position
}
//AST实例
{
			"ast_type":	"AliasTypeDecl",
			"name":	"Myint",
			"is_pub":	false,
			"parent_type":	"int",
			"pos":	{
				"line_nr":	20,
				"pos":	131,
				"len":	10
			}
		}
```

```c
//C代码
typedef int main__Myint;
```



#### 函数类型

```v
type MyFn = fn ( int,  int) int
```

```v
pub struct FnTypeDecl {
pub:
	name   string
	is_pub bool
	typ    table.Type
	pos    token.Position
}

{
			"ast_type":	"FnTypeDecl",
			"name":	"main.MyFn",
			"is_pub":	false,
			"typ":	"main.MyFn",
			"pos":	{
				"line_nr":	24,
				"pos":	171,
				"len":	9
			}
}
```

```c
typedef int (*main__MyFn)(int,int);
```

#### 联合类型

```v
type MySumtype = f32 | int | string
```

```v
pub struct SumTypeDecl {
pub:
	name      string
	is_pub    bool
	sub_types []table.Type
	pos       token.Position
}

{
			"ast_type":	"SumTypeDecl",
			"name":	"MySumtype",
			"is_pub":	false,
			"sub_types":	["f32", "int", "string"],
			"pos":	{
				"line_nr":	26,
				"pos":	204,
				"len":	14
			}
}
```

```c
typedef struct {
    void* _object;
    int typ;
} main__MySumtype;

```



### 接口

#### 接口定义

### 泛型

#### 泛型函数

#### 泛型结构体

