## 并发

V语言并发的思路和语法跟go一样,甚至关键字也一样:go/chan/select

目前已经实现了一个早期版本的并发,可以初步使用

```go
module main
const (
	num_iterations = 10000
)

fn do_send(ch chan int) {
	for i in 0 .. num_iterations {
		ch <- i
	}
}

fn main() {
	ch := chan int{cap: 1000}
	go do_send(ch)
	mut sum := i64(0)
	for _ in 0 .. num_iterations {
		sum += <-ch
	}
	println(sum)
}

```

### 声明channel变量

channel变量声明时,cap表示容量,len表示长度,可以指定容量cap,但是长度len不能在初始化时指定,初始值为0,只读,根据写入channel的结果自动改变,表示被已被写入的长度

```go
fn main() {
	ch := chan int{cap: 1000} //声明一个channel变量,类型为int,容量为1000
	println(ch.len) //0
	println(ch.cap) //1000
  
  ch2 := chan int{} //不指定cap,默认为0
  println(ch2.len) //0
	println(ch2.cap) //0
  
	ch <- 1
	println(ch.len) //1
	println(ch.cap) //1000
}
```

### 读channel/接收消息

```go
ch:=chan int{cap:100}
sum:= <-ch //从channel读取
```

### 写channel/发送消息

```go
ch:=chan int{cap:100}
ch <-2 //将2写入channel
```

### sync标准模块

​	WaitGroup



更多参考代码可以查看: vlib/sync

