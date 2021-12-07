## 常量

### 定义常量

使用const关键字定义常量，默认是模块级常量，使用pub变为公共常量，可以被其他模块访问。

常量只能在模块级别定义，不能在函数内部定义。

```v
module mymodule

//公共常量，可以全局使用
pub const (
	p_my_const     = 'abc' //注意是=号,而不是:=
	p_my_int_const = 123
)

//模块常量,只能在模块内部使用
const (
	my_const     = 'abc'
	my_int_const = 123
)

//也可以定义单个常量
const pi = 3.14
pub const single_const = 'abc'

fn my_fn() {
	//不能在函数内部定义常量
}

```

### 常量类型

跟其他语言不太一样，V语言的常量不仅可以是基本类型，也可以是数组，字典，枚举，结构体等复杂类型，甚至可以是函数调用的返回结果。

```v
module main

struct Color {
	r int
	g int
	b int
}

pub fn (c Color) str() string {
	return '{$c.r, $c.g, $c.b}'
}

fn rgb(r int, g int, b int) Color {
	return Color{
		r: r
		g: g
		b: b
	}
}

const (
	numbers = [1, 2, 3] // 数组
	red     = Color{
		r: 255
		g: 0
		b: 0
	} // 结构体类型
	blue    = rgb(0, 0, 255) // 函数调用返回的结果,作为常量的值
)

fn main() {
	println(numbers)
	println(red)
	println(blue)
}

```

### 常量名规则

跟其他语言不太一样，V语言常量名强制约定：小蛇式命名风格(lower snake case)，即全小写或小写加下划线，跟变量，函数的命名风格一致。

同时，为了比较好地在模块代码中识别常量和变量，在格式化代码时，除了主模块外，会把模块前缀加到常量名前，以区别局部变量。

```v
module main

const (
	PI         = 3.14 //警告
	Version    = '1.1.1' //警告
	num        = 3
	built_mode = 'mode'
)

fn main() {
	println(PI)
	println(num)
	println(built_mode)
	println(Version)
}

```
