## 函数

### 函数定义

使用fn关键字定义函数

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

V语言的函数定义(函数签名)基本跟go一样

去除各种视觉干扰的符号,简洁清晰,而函数关键字是跟rust一样的fn,更简洁

函数的命名强制采用rust一样的小蛇式风格(lower snake case),即小写+下划线的风格，看着很舒服,而不是采用go的大小写来区分可访问性,这样就不会强制要大小写

### 函数访问控制

默认模块内部访问,使用pub才可以被模块外部访问

```v
module mymodule

fn private_fn() { // 模块内部可以访问,模块外部不可以访问
}

pub fn public_fn() { // 模块内部和外部都可以访问
}

```

### 函数参数

函数的参数采用的是类型后置,相同类型的相邻参数可以合并类型

函数的参数默认是不可变的,如果在函数内部要改变传进来的参数,要在函数参数的定义和调用中加上mut

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

不确定个数参数也是支持的,不确定参数要放在参数的最后一个

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

### 函数返回值

函数的返回值可以是单返回值,也可以是多返回值

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

返回值也可以返回指针类型

```v
fn get_the_dao_way() voidptr { //返回通用指针
	return voidptr(0)
}

fn multi_voidptr_ret() (voidptr, bool) { //返回通用指针
	return voidptr(0), true
}

fn multi_byteptr_ret() (byteptr, bool) { //返回字节指针
	return byteptr(0), true
}
```

忽略返回值

跟go一样,可以使用下划线来忽略函数的某个返回值

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

在函数退出前执行defer代码段,一般用来在函数执行完毕后,释放资源的占用

一个函数可以有多个defer代码块,采用先定义先执行的原则

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
from defer_fn1
from defer_fn2
```

### 函数类型

相同的函数签名,表示同一类函数,可以用type定义为函数类型

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



### 函数递归

函数也是可以递归调用的

### 泛型函数

参考[泛型章节](./generic.md)

### 函数重载

V不会有函数重载

### 函数默认值

函数的参数目前还没有参数默认值这个特性

### 内联函数

可以对函数添加[inline]标注,主要的用途是在调用C代码库的时候使用最多,内联函数跟C的内联函数概念一样,生成的也是C的内联函数

```v
[inline]
pub fn width() int {
	return C.sapp_width()
}
```

详细参考:[调用C代码库](./c.md)

### 匿名函数

可以在函数内部定义匿名函数:

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

### 函数标注

- 作废标注

  模块发布给其他用户使用后,如果模块的某个函数想要声明作废,可以使用作废标注

```v
[deprecated] //函数作废标注
pub fn ext(path string) string {
	panic('Use `filepath.ext` instead of `os.ext`') //结合panic进行报错提示
}
[deprecated] //方法作废标注
pub fn (p Point) position() (int, int) {
	return p.x, p.y
}
```

- inline标注

  inline标注的功能跟C函数的inline函数一样

```v
[inline] //函数inline标注
pub fn sum(x y int) int {
  return x+y
}

[inline] //方法inline标注
pub fn (p Point) position() (int, int) {
	return p.x, p.y
}
```

