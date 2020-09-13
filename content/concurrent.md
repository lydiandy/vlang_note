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

没有指定cap的就是同步模式,同步模式下,发送和接收双方配对,然后读写同时完成,如果接收之前,还没有发送,就会出现阻塞

有指定cap的就是异步模式,异步模式下,在缓冲大小的范围内,发送方不用等待接收方,数据写入后,继续往下执行,不会出现阻塞,如果超出了缓冲大小范围,发送方还是要阻塞等待接收方接收数据

channel从底层实现上来说,是一个队列,通过push()把数据写入到队列中,通过pop()把数据读取出来,

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

//也可以使用try_pop()
//尝试读channel,把channel的值,读取到i变量中,并返回ChanState枚举:.success/.not_ready/.colsed
i:=0
res:=ch.try_pop(&i) 
```

### 写入channel/发送消息

```go
ch:=chan int{cap:100}
ch <-2 //写入channel

//也可以使用try_push()
//尝试写channel,把i的值写入到channel中,并返回ChanState枚举:.success/.not_ready/.colsed
i:=3
res:=ch.try_push(&i)

```

### 关闭channel

```go
ch.close()
```

关闭channel以后

### sync标准模块

**Channel**

```go
//使用sync模块创建channel
mut ch := sync.new_channel<int>(0) //泛型风格
ch.cap //返回channel的缓冲区大小
ch.len() //返回channel当前已使用的缓冲大小
ch.push(&i) //写channel,一定要使用指针引用
ch.pop(&i) //读channel,一定要使用指针引用,返回bool类型,true读取成功,false读取失败
ch.try_push() //尝试写channel,返回ChanState枚举:.success/.not_ready/.colsed
ch.try_pop()  //尝试读channel,返回ChanState枚举:.success/.not_ready/.colsed
ch.close()  //关闭channel

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

