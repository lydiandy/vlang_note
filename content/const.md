## 常量

### 定义常量

const关键字定义常量

默认是模块常量,使用pub变为公共常量,可以别其他模块访问

常量只能在模块级别定义,不能在函数内部定义

```c
module mymodule

//公共常量，可以全局使用
pub const (  
	p_my_const = 'abc'  	//注意是=号,而不是:=
	p_my_int_const = 123
)

//模块常量,只能在模块内部使用
const (  
	my_const = 'abc'  
	my_int_const = 123
)

fn my_fn() {
	//不能在函数内部定义常量
}
```

### 常量类型

跟其他语言不太一样,V语言的常量不仅只可以是基本类型,

也可以是数组,字典,枚举,结构体等复杂类型,甚至可以是函数调用的返回结果

因为V语言没有全局变量,模块变量,所有的变量只能在函数内部定义,所以常量比较常被使用,常量的类型跟变量类型一样多样

```c
module main
struct Color {
	r int
	g int
	b int
}

pub fn (c Color) str() string {
	return '{$c.r, $c.g, $c.b}'
}

fn rgb(r, g, b int) Color {
	return Color {
		r: r
		g: g
		b: b
	}
}

const (
	Numbers = [1, 2, 3] // 数组
	Red = Color {
		r: 255
		g: 0
		b: 0
	} // 结构体类型
	Blue = rgb(0, 0, 255) // 函数调用返回的结果,作为常量的值
)

fn main() {
	println(Numbers)
	println(Red)
	println(Blue)
}

```

### 常量名规则

V语言常量名没有强制约定：可以是大写开头，全大写，全小写，小写加下划线

```c
module main

const (
    PI=3.14 
    num=3
    built_mode='mode'
    Version='1.1.1'
)

fn main() {
    println(PI)
    println(num)
    println(built_mode)
    println(Version)
}
```

### 