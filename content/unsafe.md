## 不安全代码

### 内存安全/不安全函数

默认情况下:

所有的V函数调用都是内存安全的,可信的,

所有的C函数调用都是内存不安全的,不可信的,

如果要把某个V函数定义为内存不安全的,要给函数加上unsafe标注,

如果要把C函数定义为内存安全的,可信的,要给函数加上trusted标注,

不管是V函数还是C函数,所有内存不安全的函数都要在unsafe代码块中调用,否则编译器会报错.

```v
[unsafe] //标注V函数为不安全的函数,因为里面有手动的内存控制或指针运算等内存不安全操作
pub fn (a array) free() {
	//if a.is_slice {
		//return
	//}
	C.free(a.data)
}
```

```v
// v/builtin/cfns.c.v
[trusted] //标注C函数为安全的,信任的函数
fn C.calloc(int, int) byteptr

fn C.malloc(int) byteptr

fn C.realloc(a byteptr, b int) byteptr

fn C.free(ptr voidptr)

[trusted]
fn C.exit(code int)
```

unsafe函数必须在unsafe代码块中调用

```v
fn my_fn() {
	unsafe {
		//在这里才能调用不安全函数
	}
	//在unsafe代码块之外调用,编译器会警告
}
```

在不安全代码块中进行指针运算和多级指针,编译器啥都不管,自己掌控,就跟C一样

```v
module main

fn main() {
	unsafe {
		v := 4
		mut p := &v
		// unsafe代码块中可以进行指针运算,编译器不负责检查
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
	mut q := byteptr(10)
	println(q)
	unsafe {
		q -= 2
		q = q + 1
	}
	println(q)
	//不安全表达式,返回指针运算后的结果
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
