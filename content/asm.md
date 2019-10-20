## 调用汇编代码

目前还未开发实现,只是设想

```
fn main() {
	a := 10
	asm x64 {
		mov eax, [a]
		add eax, 10
		mov [a], eax
	}
}
```

