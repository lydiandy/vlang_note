## 内联汇编代码

V语言可以像C语言那样,在v代码中直接编写/嵌入汇编代码(inline asm)

使用asm代码块来编写汇编代码,asm代码块必须被包含在unsafe代码块中

```v
module main

fn main() {
	a := 10
	b := 0
	unsafe {	//unsafe代码块
		asm x64 {	//asm代码块,里面可以直接写汇编代码
			"movl %1, %%eax;"
			"movl %%eax, %0;"
			:"=r"(b)
			:"r"(a)
			:"%eax"
		}
	}
	println(a) //返回10
	println(b) //返回10,直接通过汇编代码修改了b的值

	e := 0
	unsafe {
		asm x64 {
			"movl $5, %0"
			:"=a"(e)
		}
	}
	println(e) //返回5,直接通过汇编代码修改了e的值
}
 
```

