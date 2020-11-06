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



### 常量

#### 常量定义



### 枚举

#### 枚举定义

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

#### 函数类型

#### 联合类型

### 接口

#### 接口定义

### 泛型

#### 泛型函数

#### 泛型结构体

