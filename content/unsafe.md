## 不安全代码

1. #### 标注函数为不安全函数

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

2. #### unsafe函数必须在unsafe代码块中调用

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
fn test_pointer_arithmetic() {
	arr := [1,2,3,4]
	unsafe {
		mut parr := *int(arr.data)
		parr++
		assert 2 == *parr
		parr++
		assert 3 == *parr
		assert *(parr + 1) == 4
	}
}

fn test_multi_level_pointer_dereferencing() {
	n := 100
	pn := &n
	ppn := &pn

	unsafe {
		mut pppn := &ppn
		***pppn = 300
		pppa := ***int(pppn)
		assert 300 == ***pppa
	}

	assert n == 300 // updated by the unsafe pointer manipulation
}
```

