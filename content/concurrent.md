## 并发

还没有实现,语法基本跟go一样,也是使用go关键字,预计也是跟go一样的轻量级线程

目前如果使用go关键字,代码也能正常运行,只是启用一个新的子进程来执行,没啥意义:

```c
module main

fn add(x,y int) int {
	println('from add')
	return x+y
}

fn main(){
	println('from main')
	go add(2,5) 
}
```

