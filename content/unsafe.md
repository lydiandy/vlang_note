## 不安全代码

### 不安全代码块

标注函数为不安全函数

所有手动控制内存的函数要标注为[unsafe]

```v
[unsafe] //标注为不安全函数
pub fn (a array) free() {
	//if a.is_slice {
		//return
	//}
	C.free(a.data)
}
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

