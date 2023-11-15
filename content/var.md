### 变量

V是静态类型语言，每一个变量都有对应类型。

#### 声明和赋值

V语言中只有唯一的一种变量声明和赋值方式 :=，并且声明和赋值是要同时的，这意味着变量总会有一个初始值。

```v
a := false
b := 'abc'
c := 3 //默认推断为int
f := 3.1 //默认推断为f64
```

#### 类型推断

上述的代码并没有体现出变量的类型，是因为编译器会根据变量的值自动进行类型推断。

#### 显示指定类型

如果不希望由编译器自动类型推断，可以通过T(value)的格式明确变量类型，T是类型，value是变量值：

```v
x := i64(3) // x是i64类型，而不是默认推断的int
y := f32(3.0) // y是f32类型，而不是默认推断的f64
```

这个显示指定类型的风格，跟其他语言确实不太一样，一般的风格都是：

```typescript
let x int = 1
let x: int = 1
//或者
var x int = 1
var x: int =1
```

#### 判断变量类型

通过使用typeof内置函数，可以判断变量类型：

```v
x := 3
s := 'abc'
println(typeof(x)) // int
println(typeof(s)) // string
```

#### 默认不可变

跟rust一样，变量默认不可变，要声明为可变，使用mut关键字：

```v
mut age := 20
println(age)
age = 21
println(age)
```

要注意区分 := 和 = 的不同之处：

:= 的含义是为变量声明并赋值。

= 的含义是为变量绑定一个新的值，也可以理解为修改变量值。

变量声明后，如果没有被使用：

开发编译(v run)：编译器只是会警告，但仍然继续编译，方便开发调试，而不用去临时注释掉。

生产编译(v -prod)：编译器会报错，停止编译。

以下几种情况，会编译不通过：

```v
fn main() {
	age = 21 //变量还未声明
}
```

```v
fn main() {
	age := 21 //变量声明和赋值后，没有使用，开发编译，只会警告，生产编译，会报错
}
```

```v
fn main() {
	a := 10
	if true {
		a := 20 //跟其他语言不一样，没有上级变量隐藏，在函数内部，同名的变量只能定义一个
	}
}
```

#### 多变量赋值

```v
fn main() {
	f1()
	f2()
	f3()
	f4()
}

fn f1() {
	a, b, c := 1, 3, 5 // 多变量声明并赋值
	println(a)
	println(b)
	println(c)
}

fn f2() {
	mut a := 1
	mut b := 2
	a, b = b, a // 交换
}

fn f3() {
	mut a := 11
	mut b := 22
	mut c := 33
	mut d := 44
	a, b, c, d = b, a++, d + 1, -c // 多变量赋值
	println(a) // 22
	println(b) // 11
	println(c) // 45
	println(d) // -33
}

fn f4() {
  mut x, y := 1, 3 //注意，只有x是可以变的，y不可变
	mut a, mut b, mut c := 1, 2, 3 //a,b,c都是可变的
	println(a)
	println(b)
	println(c)
}
```

条件赋值

```v
d, e, f := if true {
		1, 'awesome', [13]
	} else {
		0, 'bad', [0]
	}
```

匹配赋值

```v
a, b, c := match false {
	true { 1, 2, 3 }
	false { 4, 5, 6 }
	else { 7, 8, 9 }
}
```

#### 强制类型转换

可以通过T( ) 对类型进行显示声明，或者强制类型转换：

```v
module main

fn main() {
	x := int(3)
	y := u8(x)
	println('y is $y')
	z := f32(x)
	println('z is $z')
	f := 1.2
	i := int(f)
	println(i) // 输出1，强制转换丧失精度
}
```

#### 静态局部变量

跟C的静态局部变量一样，用关键字 static 声明，静态局部变量的值在函数调用结束之后不会消失，而仍然保留其原值。

一般来说，普通的V代码很少会用到静态局部变量，只有跟C集成的时候才会用到。

使用静态局部变量有以下2种方式：

- 在-translated模式中使用
- 在unsafe函数和代码块中使用

```v
module main

fn f() int {
	mut x := 1 //普通的局部变量
	x++
	return x
}

//一定要在unsafe函数中
@[unsafe]
fn f_static() int {
	unsafe { //在unsafe代码块中定义
		mut static x := 1 //静态局部变量
		x++
		return x
	}
}

fn main() {
	println(f()) // 2
	println(f()) // 2
	println(f()) // 2
	unsafe { //调用的时候也要使用unsafe代码块
		println(f_static()) // 2
		println(f_static()) // 3
		println(f_static()) // 4
	}
}
```

#### 默认没有模块级/全局变量

V语言中，变量只能在函数里定义，就是局部变量，默认没有模块级变量，没有全局变量。

默认情况下编译器是没有全局变量声明的，但是为了跟C代码集成，有时候需要定义全局变量，可以在调用编译器时，通过增加 -enable-globals选项来启用。

```shell
v -enable-globals run main.v
```

```v
module main

// 单个全局变量定义
__global g1 int

// 组定义全局变量，类似常量组的定义
__global (
	g2 u8 
	g3 u8 
)

fn main() {
	g1 = 1
	g2 = 2
	g3 = 3
	println(g1)
	println(g2)
	println(g3)
}
```

可以使用 [c_extern]注解，在生成的C代码中，给全局变量增加extern关键字。

```v
module main

// 增加extern关键字
 @[c_extern]
__global g1 int

// 组定义全局变量，类似常量组的定义
 @[c_extern]
__global (
	g2 u8 
	g3 u8 
)

fn main() {
	g1 = 1
	g2 = 2
	g3 = 3
	println(g1)
	println(g2)
	println(g3)
}
```

生成的C代码：

```c
extern int  g1; // global4
extern u8  g2; // global4
extern u8  g3; // global4
```
