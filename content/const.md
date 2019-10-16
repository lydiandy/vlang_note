## 常量

### 定义常量

通过const关键字来定义常量

常量只能在模块级别定义,不能在函数内部定义

```
module mymodule

//模块级别定义常量
const (  
	my_const = 'abc'  	//注意是=号,而不是:=
	my_int_const = 123
)

fn my_fn() {
	//不能在函数内部定义
}
```

### 常量类型

跟其他语言不太一样的是,V语言的常量不仅只可以是基本类型,

也可以是数组,字典,枚举,结构体等复杂类型,甚至可以是函数调用的返回结果

```
struct Color {
        r int
        g int
        b int
}

fn (c Color) str() string { return '{$c.r, $c.g, $c.b}' }

fn rgb(r, g, b int) Color { return Color{r: r, g: g, b: b} }

const (
        Numbers = [1, 2, 3] //数组

        Red  = Color{r: 255, g: 0, b: 0} //结构体类型
        
        Blue = rgb(0, 0, 255) //函数调用返回的结果,作为常量的值
)
```



### 常量名的大小写规则

目前版本的V语言没有限制常量是否要全部大写,或者首字母大写,或者小写加下划线,都可以全部编译通过

V源代码里面的常量用的是大骆驼峰风格

```
module main

const (
    PI=3.14
    num=3
    built_mode='mode'
    Version='1.1.1'
)

fn main() {
    println(PI)
    println()
    println(built_mode)
    println(Version)
}
```

### 