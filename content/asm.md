## 调用汇编代码

目前已初步实现调用汇编代码，目前主要用在bare metal（裸机环境中），参考V源代码中：

vlib/builtin/bare/linuxsys_bare.v相关行内汇编代码

inline asm代码必须被包含在unsafe代码块中

```go
fn test_inline_asm() {
	a := 10
	b := 0
	unsafe {
		asm {
			"movl %1, %%eax;"
			"movl %%eax, %0;"
			:"=r"(b)
			:"r"(a)
			:"%eax"
		}
	}
	assert a == 10
	assert b == 10
	//
	e := 0
	unsafe {
		asm {
			//".intel_syntax noprefix;"
			//"mov %0, 5"
			"movl $5, %0"
			:"=a"(e)
		}
	}
	assert e == 5
}
```

