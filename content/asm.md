## 内联汇编代码(inline asm)

V语言可以像C语言那样,在V代码中直接编写/嵌入汇编代码

使用asm代码块来编写汇编代码

```v
fn main() {
	a := 100
	b := 20
	mut c := 0
	asm amd64 { //行内汇编代码块
		mov eax, a
		add eax, b
		mov c, eax
		; =r (c) // output 
		; r (a) // input 
		  r (b)
	}
	println('a: $a') // 100
	println('b: $b') // 20
	println('c: $c') // 120
}

```

目前支持的架构:

| 架构名称                               | 推荐使用名称 | 描述          |
| -------------------------------------- | ------------ | ------------- |
| amd64, x86_64, x64, x86                | amd64        | x86_64        |
| aarch64, arm64                         | aarch64      | 64-bit arm    |
| arm32, aarch32, arm                    | aarch32      | 32-bit arm    |
| rv64, riscv64, risc-v64, riscv, risc-v | rv64         | 64-bit risc-v |
| rv32, riscv32                          | rv32         | 32-bit risc-v |
| x86_32, x32, i386, IA-32, ia-32, ia32  | i386         | i386          |

