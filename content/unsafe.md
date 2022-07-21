## 不安全代码

### 内存安全/不安全函数

默认情况下：

- 所有的V函数调用都是内存安全的，可信的。


- 所有的C函数调用都是内存不安全的，不可信的。


- 如果要把某个V函数定义为内存不安全的，要给函数加上unsafe注解。


- 如果要把C函数定义为内存安全的，可信的，要给函数加上trusted注解。


不管是V函数还是C函数，所有内存不安全的函数都要在unsafe代码块中调用，否则编译器会报错。

```v
[unsafe] //注解V函数为不安全的函数，因为里面有手动的内存控制或指针运算等内存不安全操作
pub fn (a array) free() {
	//if a.is_slice {
		//return
	//}
	C.free(a.data)
}
```

```v
// v/builtin/cfns.c.v
[trusted] //注解C函数为安全的，信任的函数
fn C.calloc(int, int) &u8

fn C.malloc(int) &u8

fn C.realloc(a &u8, b int) &u8

fn C.free(ptr voidptr)

[trusted]
fn C.exit(code int)
```

unsafe函数必须在unsafe代码块中调用。

```v
fn my_fn() {
	unsafe {
		//在这里才能调用不安全函数
	}
	//在unsafe代码块之外调用，编译器会警告
}
```

在不安全代码块中进行指针运算和多级指针，编译器啥都不管，自己掌控，就跟C一样。

```v
module main

fn main() {
	unsafe {
		v := 4
		mut p := &v
		// unsafe代码块中可以进行指针运算，编译器不负责检查
		p++
		p += 2
		p = p - 1
		println(p)
		p = p + 1
		println(p)
		r := p++
		println(r)
		//多级指针
		n := 100
		pn := &n
		ppn := &pn
		mut pppn := &ppn
		println(pppn)
	}
}

```

### 不安全表达式

```v
module main

fn main() {
	mut q := &u8(10)
	println(q)
	unsafe {
		q -= 2
		q = q + 1
	}
	println(q)
	//不安全表达式，返回指针运算后的结果
	s := unsafe { q - 1 }
	println(s)
	unsafe {
		q++
		q++
		q--
	}
	println(q)
}

```

### 空值/空指针

nil表示空值或者空指针，等价于voidptr(0)。

正常情况下，由V创建的变量，因为声明和初始化一定是同时进行的，

所以变量一定会有初始值，V指针一定不会有空值nil。

内置函数中的isnil(ptr)，是用来判断由C代码生成的指针，是否是空指针:

```v
fn main() {
	a := 1
	println(isnil(&a)) // 返回false，变量只能通过:=来初始化，一定会有初始值
	// 但是通过调用C代码返回的指针，有可能是空指针，所以在使用前可以用isnil函数来判断一下
	f := C.popen('ls'， 'r')
	if isnil(&f) {
		// ...
		println('f is nil')
	} else {
		println('f is not nil')
	}
}
```

也可以在代码中使用==或!=来判断指针是否是空值：

```v
module main

struct Point {
pub:
	x int
	y int
}

fn main() {
	p1 := unsafe { nil } //初始化为空值nil，一定要在unsafe代码块中，或unsafe表达式
	p2 := &Point{1, 2}
	if p1 == unsafe { nil } { //判断是否为空值nil
		println('p1 is nil')
	}
	if p2 != unsafe { nil } { //判断是否不为空值nil
		println('p2 is not nil')
	}
}
```

