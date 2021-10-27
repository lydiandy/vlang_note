## 函数

### 函数定义

使用fn关键字定义函数：

```v
fn main() {
	println(add(77, 33))
	println(sub(100, 50))
}

fn add(x int, y int) int {
	return x + y
}

fn sub(x int, y int) int {
	return x - y
}
```

V语言的函数定义(函数签名)基本跟go一样，去除各种视觉干扰的符号，简洁清晰，而函数关键字是跟rust一样的fn，更简洁。

函数的命名强制采用rust一样的小蛇式风格(lower snake case)，即小写+下划线的风格，看着很舒服，而不是采用go的大小写来区分可访问性，这样就不会强制要大小写。

### 主函数

程序从主函数开始运行，主函数的签名中没有参数，也没有返回值，简单干净。

从命令行启动程序时，传递给程序的参数会统一存放在os.args这个字符串数组中，只读，全局可以访问。

主函数如果中途要退出，使用exit(code int)内置函数，并返回错误码给操作系统。

```v
module main	//主模块
import os

fn main() { //主函数
	println('main function')
	println(os.args) //命令行中传递给程序的所有参数保存在os.args字符串数组中
	has_error := true
	if has_error {
		println('has error,exit code is 1')
		exit(1)
	}
}
```

### 函数访问控制

默认模块内部访问，使用pub才可以被模块外部访问。

```v
module mymodule

fn private_fn() { // 模块内部可以访问,模块外部不可以访问
}

pub fn public_fn() { // 模块内部和外部都可以访问
}

```

### 函数参数

函数的参数采用的是类型后置，相同类型的相邻参数可以合并类型。

函数的参数默认是不可变的，如果在函数内部要改变传进来的参数，要在函数参数的定义和调用中加上mut。

```v
fn my_fn(mut arr []int) { // 1.参数定义也要是可变的
	for i := 0; i < arr.len; i++ {
		arr[i] *= 2
	}
}

fn main() {
	mut nums := [1, 2, 3] // 2.传进来的参数要是可变的
	my_fn(mut nums) // 3.调用函数时,还需要在参数前加上mut,不然会报错
	println(nums) // 返回[2, 4, 6],函数执行后,传进来的参数被改变了
}

```

不确定个数参数也是支持的，不确定参数要放在参数的最后一个。

```v
fn my_fn(i int, s string, others ...string) {
	println(i)
	println(s)
	println(others[0])
	println(others[1])
	println(others[2])
}

fn main() {
	my_fn(1, 'abc', 'de', 'fg', 'hi')
}
```

数组可以传递给不确定参数函数，不确定参数函数之间也可以传递参数。

```v
module main

fn main() {
	a := ['a', 'b', 'c']
	println(variadic_fn_a(...a)) //数组解构赋值后,传递给不确定参数数组
}

fn variadic_fn_a(a ...string) string {
	return variadic_fn_b(...a) //数组解构赋值后,传递给不确定参数数组
}

fn variadic_fn_b(a ...string) string {
	a0 := a[0]
	a1 := a[1]
	a2 := a[2]
	return '$a0$a1$a2'
}


```

### 函数返回值

函数的返回值可以是单返回值，也可以是多返回值。

```v
fn bar() int { //单返回值
	return 2
}

fn foo() (int, int) { //多返回值
	return 2, 3
}

fn some_multiret_fn(a int, b int) (int, int) {
	return a + 1, b + 1 //可以返回表达式
}

fn main() {
	a, b := foo()
	println(a) // 2
	println(b) // 3
}
```

返回值也可以返回指针类型。

```v
fn get_the_dao_way() voidptr { //返回通用指针
	return voidptr(0)
}

fn multi_voidptr_ret() (voidptr, bool) { //返回通用指针
	return voidptr(0), true
}

fn multi_&byte_ret() (&byte, bool) { //返回字节指针
	return &byte(0), true
}
```

忽略返回值，跟go一样，可以使用下划线来忽略函数的某个返回值。

```v
fn main() {
	a, _ := foo() // 忽略第二个返回值
	_, _ := foo() // 也可以忽略全部的返回值
	println(a) // 2
	b := bar()
	_ := bar()
	println(b) // 2
}

fn bar() int {
	return 2
}

fn foo() (int, int) {
	return 2, 3
}

```

### 函数defer语句

在函数退出前执行defer代码块，一般用来在函数执行完毕后，释放资源的占用。一个函数可以有多个defer代码块，采用后定义先执行(后进先出)的原则。

同时在defer代码块内有一些特殊的注意事项：

- defer代码块内不可调用return
- defer代码块内不可嵌套定义defer代码块
- defer代码块内调用函数时，不可向上抛转错误，例如 a=test1() ?

```v
fn main() {
	println('main start')
	// defer {defer_fn1()} //写成单行也可以
	// defer {defer_fn2()}
	defer {
		defer_fn1()
	}
	defer {
		defer_fn2()
	}
	println('main end')
}

fn defer_fn1() {
	println('from defer_fn1')
}

fn defer_fn2() {
	println('from defer_fn2')
}

```

执行结果:

```
main start
main end
from defer_fn2
from defer_fn1
```

### 函数类型

相同的函数签名,表示同一类函数，可以用type定义为函数类型。

```v
type Mid_fn = fn (int, string) int
```

### 函数作为参数

```v
fn sqr(n int) int {
	return n * n
}

fn run(value int, op fn (int) int) int {
	return op(value)
}

fn main() {
	println(run(5, sqr)) // "25" sql函数作为参数传递
}
```

### 函数作为返回值

```v
module main

fn add(x int) int {
	return x
}

fn sum() fn (int) int { // 函数类型作为返回值
	return add
}

fn main() {
	a := sum()
	println(a(2))
}

```

### 闭包的参数传递

V的函数支持闭包，上级函数和闭包函数是两个单独的作用域，要将上级函数中的变量要传递给闭包函数，需要用中括号特别列出来，这样闭包函数内才可以正常使用上级函数的变量。

```v
module main

fn main() {
	i := myfn(1)
}

type Handler = fn (string) int

fn myfn(x int) Handler {
	mut y := 3
  //上级函数的x,y变量要传递进去，如果是可变的，也要像函数参数那样，使用mut
	return fn [x, mut y] (name string) int { 
		y++
		return x
	}
}
```

如果要传递的是变量的引用也可以，不过要确保变量在调用函数的时候始终可用，否则就会产生错误：

```v
mut i := 0
mut ref := &i
print_counter := fn [ref] () {
	println(*ref)
}

print_counter() // 0
i = 10
print_counter() // 10
```

### 函数递归

函数也是可以递归调用的。

### 泛型函数

参考[泛型章节](./generic.md)。

### 函数重载

V不会有函数重载。

### 函数参数默认值

函数的参数没有默认值这个特性。不过可以使用结构体作为参数来实现类似的效果。

### 函数命名参数调用

函数调用时可以使用命名参数，这样参数的传递只根据参数名来匹配，而跟参数的顺序无关。

通过使用结构体作为参数，可以实现类似的效果，这个方式比语言上直接实现多了一个步骤，就是要先创建一个结构体参数。

这个语法在ui模块比较常用到，用来简化函数参数，也有点像命名参数的感觉：

```v
module main

struct User {
	name string
	age int
}

fn add(u User) {
	println(u)
}

fn main(){
	add(User{name:'jack',age:22}) //标准方式
	add({name:'tom',age:23}) //简短方式,省略类型
	add(name:'tt',age:33) //更简短的方式,省略类型和大括号,有命名参数调用的感觉
}
```

### 结构体参数

就是使用结构体作为参数，并且结构体使用了[params]注解，就可以实现参数的默认值和命名参数传参的效果。

```v
[params]
struct ButtonConfig {
	text        string
	is_disabled bool
	width       int = 70
	height      int = 20
}

struct Button {
	text   string
	width  int
	height int
}

fn new_button(c ButtonConfig) &Button {
	return &Button{
		width: c.width
		height: c.height
		text: c.text
	}
}

fn main() {
	button := new_button(text: 'Click me', width: 100)
	println(button.height) // height没有初始化,使用默认值
	
	//结构体参数加了params注解,就可以什么都不传,直接使用结构体的默认值
	button2 := new_button()
	println(button2) //所有参数都使用默认值
}
```

### 内联函数

可以对函数添加[inline]注解，主要的用途是在调用C代码库的时候使用最多，内联函数跟C的内联函数概念一样，生成的也是C的内联函数。

```v
[inline]
pub fn width() int {
	return C.sapp_width()
}
```

详细参考：[调用C代码库](./c.md)

### 匿名/内部函数

可以在函数内部定义匿名函数：

```v
import sync

fn main() {
	f1 := fn (a int) { // 定义函数类型变量
		println('hello from f1')
	}
	f1(1)
	f2 := fn (a int) { // 定义函数类型变量
		println('hello from f2')
	}
	f2(1)
	fn (a int) { // 匿名函数定义同时调用
		println('hello from anon_fn')
	}(1)
	// 匿名函数结合go使用
	mut wg := sync.new_waitgroup()
	go fn (mut wg sync.WaitGroup) {
		println('hello from go')
		wg.done()
	}(mut wg)
	wg.wait()
}
```

### 函数注解

#### [deprecated]

模块发布给其他用户使用后，如果模块的某个函数想要声明作废，可以使用作废注解。

```v
[deprecated] //函数作废注解
pub fn ext(path string) string {
	panic('Use `filepath.ext` instead of `os.ext`') //结合panic进行报错提示
}
[deprecated:'可以在这里写说明'] //带说明的作废注解,编译器提示函数作废的时候会显示出来
pub fn (p Point) position() (int, int) {
	return p.x, p.y
}

[deprecated:'这是作废说明']
[deprecated_after: '2021-03-01']  //在某个日期后开始作废,一定要放在deprecated后,才会警告
pub fn fn1() (int, int) (int, int) {
	return p.x, p.y
}
//函数被调用,就会出现如下警告:
//warning: function `fn1` has been deprecated since 2021-03-01; 这是作废说明
```

#### [inline]

inline注解的功能跟C函数的inline函数一样。

```v
[inline] //函数inline注解
pub fn sum(x y int) int {
  return x+y
}

[inline] //方法inline注解
pub fn (p Point) position() (int, int) {
	return p.x, p.y
}
```

#### [unsafe]

参考[不安全代码章节](unsafe.md)

#### [trusted]

参考[不安全代码章节](unsafe.md)

#### [live]

代码热更新功能,实际生产用处不大，开发时可以使用，像脚本语言那样，保存即生效，不用重启服务器，正式发布的时候还是要去除live注解，有些麻烦。

```v
module main

import time

[live]
fn print_message() {
	println('更新时间:${time.now()}')
	println('该函数范围内的代码修改后,不需要重新编译运行,保存即生效')
}

fn main() {
	for {
		print_message()
		time.sleep(1*time.second)
	}
}
```

运行时也需要加上-live选项，才会有效果。

```
v -live run  main.v
```


### 内置函数

V内置了一些函数，可以全局使用：

#### dump函数

跟C语言中的dump函数功能一样,把某个表达式的数据转储，并输出，dump函数主要用于调试，比println函数更为方便，清晰。

表达式被调用了几次，每次的结果如何，代码的位置，都会被清楚地打印出来。

```v
fn factorial(n u32) u32 {
	if dump(n <= 1) {
		return dump(1)
	}
	return dump(n * factorial(n - 1))
}

fn main() {
	println(factorial(5))
}
```

输出：

```shell
[xxx/main.v:2] n <= 1: false
[xxx/main.v:2] n <= 1: false
[xxx/main.v:2] n <= 1: false
[xxx/main.v:2] n <= 1: false
[xxx/main.v:2] n <= 1: true
[xxx/main.v:3] 1: 1
[xxx/main.v:5] n * factorial(n - 1): 2
[xxx/main.v:5] n * factorial(n - 1): 6
[xxx/main.v:5] n * factorial(n - 1): 24
[xxx/main.v:5] n * factorial(n - 1): 120
```

