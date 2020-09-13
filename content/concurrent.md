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

channel变量声明时,可以指定cap,cap表示缓冲区大小/容量,指定后不可改变,如果不指定,默认为0

len表示当前被使用的缓冲大小,len不能在声明时指定,初始值为0,只读,根据写入/读取channel自动改变,写入增加len,读取减少len

```go
fn main() {
	ch := chan int{cap: 1000} //声明一个channel变量,类型为int,缓冲区大小为1000,即异步channel
	println(ch.len) //0
	println(ch.cap) //1000
  
  ch2 := chan int{} //不指定cap,默认为0,即同步channel
  println(ch2.len) //0
	println(ch2.cap) //0
  
	ch <- 1
	println(ch.len) //1
	println(ch.cap) //1000
}
```

### 读取channel/接收消息

```go
ch:=chan int{cap:100}
sum:= <-ch //读取channel
```

### 写入channel/发送消息

```go
ch:=chan int{cap:100}
ch <-2 //写入channel
```

### sync标准模块

**Channel**

```go
//使用sync模块创建channel
mut ch := sync.new_channel<int>(0) //泛型风格
ch.len()
ch.push()
ch.pop()
ch.try_push()
ch.try_pop()
ch.close()

sync.channel_select()
```

**WaitGroup**

```go
wg:=sync.new_waitgroup()
wg.add()
wg.wait()
wg.stop()
wg.done()
```

更多参考代码可以查看: vlib/sync

