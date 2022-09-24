## 快速总览

本章节试图展示V语言的总体特性，形成对V语言的一个总体印象，后续章节再展开逐个部分详细介绍。

- V语言是一门静态类型，编译型语言。

- 语法追求简洁，基本就是吸收了go和rust中简洁的部分，go的部分更多一些，代码看起来，写起来都很舒服，这个应该也是大多数人第一眼看到V语言的感觉，被吸引的主要原因之一。

- 无GC/可选GC，错误处理机制，泛型，这三点一直是go有争议的地方，V语言都有。

- 无全局变量，无空值null，变量默认不可变，结构体默认不可变，从rust那边吸收了一些。

- 模块化支持，包管理工具。

- 跟go一样的并发。

- 6个一级元素：const常量，enum枚举，fn函数/方法，struct结构体，interface接口，type类型。

------

- 编译速度很快，可以跨平台交叉编译，编译出来的可执行文件很小，运行速度就是C的速度。
- 实现自举，编译器也是用V语言写的。
- 基本的编译器思路是：把V源代码编译生成C源代码，然后调用C编译器编译生成单一可执行文件。
- 很容易跟C集成，方便使用C成熟丰富的代码库。
- 可以像C一样，手动管理内存，实现对内存的手工释放。
- 也可以使用内置的GC，实现对内存的自动释放。
- 编译器后端支持：C/native/js/webassembly/go，目前C后端比较成熟完整，其他后端还在开发中。

---

- vscript可以像python脚本那样简单，方便地写系统shell。

- 内置json支持，非运行时反射实现，性能更好。

- 内置SQL语法支持，实现更简单的ORM。

- 内置一个轻量的跨平台GUI库。

- 内置一个web框架。

- 代码热更新，修改代码，实时显示结果。

------

以上是V语言的特性总览，里面有的特性已经实现，有的还未实现，未实现的功能正在逐步实现。

希望作者和开发团队能持续下去，周边生态能起来。

------

V语言代码初步印象：

```v
//单行注释
/*
多行注释
多行注释
*/
//定义主模块，编译生成可执行程序
module main

//导入模块
import os
import time

//主函数，程序运行入口
pub fn main() {
	println('say hello world') // 语句结尾不需要分号
	println(os.args) // 使用os模块的args变量
	println(time.now()) // 调用time模块的now函数

	go my_fn(1, 2) // 跟go一样的并发

	my_fn2(1) // 匿名联合类型作为参数或返回值

	my_fn3(0) or { // or代码块,进行错误处理
		panic(err.msg())
	}
}

//模块内6个一级元素：常量，枚举，函数/方法，结构体，接口，类型
// 1.常量
// 单行常量
pub const usage = 'usage ...' 
// 常量组
pub const (
	version             = '0.1.21'
	supported_platforms = ['windows', 'mac', 'linux']
)

// 2.枚举
pub enum OS {
	mac
	linux
	windows
}

// 3.函数-函数定义
// 函数的定义风格基本跟go一样，只是关键字改为更简短的fn，支持不确定个数参数，支持多返回值
// pub表示公共级别的访问控制，可以被模块外使用，否则只能在模块内使用
pub fn my_fn(x int, y int) int {
	i := 1 //强类型，类型推断
	s := 'abc' //变量默认不可变，约定用单引号表示字符串，双引号也可以，反引号才是单字符
	mut a := 3 //可变用mut
	a = 5 //声明可变后，才可修改
	println(i)
	println(s)
	return a
}

// 3.函数-匿名联合类型作为参数或返回值
pub fn my_fn2(x int | string | bool) int | string | bool {
	println(x)
	return x
}

// 3.函数-简洁的错误处理
pub fn my_fn3(x int) !int {
	if x == 0 {
		return error('this is an error.') // 抛出错误,然后在调用函数的or代码块进行错误处理
	}
	return x
}

// 3.函数-泛型函数
pub fn g_fn<T, U>(x T, y U) (T, U) {
	return x, y
}

// 3.方法-方法只是指定了接收者的函数，跟go一样
pub fn (mut p Point) move(x int, y int) {
	p.x += x
	p.y += y
}

// 4.结构体-结构体定义
pub struct Point {
	//结构体字段一共有5种访问控制
	// 1.默认私有且不可变
	a int
	// 2.私有，但可变
mut:
	x int
	y int
	// 3.公共，但不可变
pub:
	d int
	// 4.模块内可访问且可变;模块外可访问，但是只读
pub mut:
	e int
	// 5.全局字段，模块内部和外部都可访问，可修改，这样等于破坏了封装性，不推荐使用
__global:
	f int
}

// 4.结构体-泛型结构体
pub struct Repo<T> {
	db DB
mut:
	model T
}

pub struct DB {
}

// 5.接口-接口无须显式声明实现，鸭子类型，跟go一样
pub interface Walker {
	name string //接口支持字段约束,表示实现该接口的结构体必须有该字段
	walk(int, int) int
}

// 5.接口-泛型接口，跟泛型结构体使用基本一致
pub interface Gettable<T> {
	get() T
}

// 6.类型-类型别名，可以基于基本类型，也可基于结构体类型创建类型别名
pub type Myint = int

// 6.类型-函数类型，表示一类相同签名的函数
pub type Fn_type = fn (int) int

// 6.类型-联合类型，跟typescript类似，表示类型Expr可以是这几种类型的其中一种
pub struct BinExpr {}

pub struct BoolExpr {}

pub struct UnaryExpr {} 

pub type Expr = BinExpr | BoolExpr | UnaryExpr

// 6.类型-泛型联合类型
pub type MyOption<T> = Error | None | T

struct None {}

```
