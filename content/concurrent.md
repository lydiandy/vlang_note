## 并发

V语言并发的思路和语法跟go一样,甚至关键字也一样:go/chan/select

目前已经实现了一个早期版本的并发,可以初步使用

go可以添加在函数调用,方法调用,匿名函数调用前,即可创建并发任务单元

```v
module main
const (
	num_iterations = 10000
)

fn do_send(ch chan int) {
	for i in 0 .. num_iterations {
		ch <- i //写入channel,也叫发送消息
	}
}

fn main() {
	ch := chan int{cap: 1000}
	go do_send(ch) //在函数调用前使用go关键字,即可创建并发任务单元
	mut sum := i64(0)
	for _ in 0 .. num_iterations {
		sum += <-ch //读取channel,也叫接收消息
	}
	println(sum)
}

```

### 声明channel变量

channel变量声明时,可以指定cap,cap表示缓冲区大小/容量,指定后不可改变,如果不指定,默认为0

len表示当前被使用的缓冲大小,len不能在声明时指定,初始值为0,只读,根据写入/读取channel自动改变,写入增加len,读取减少len

没有指定cap,就是同步模式,同步模式下,发送和接收双方配对,然后读写同时完成,如果接收之前,还没有发送,就会出现阻塞

有指定cap,就是异步模式,异步模式下,在缓冲大小的范围内,发送方不用等待接收方,数据写入后,继续往下执行,不会出现阻塞,如果超出了缓冲大小范围,发送方还是要阻塞等待接收方接收数据

channel从底层实现上来说,是一个队列,通过push()把数据写入到队列中,通过pop()把数据读取出来,

```v

fn main() {
	ch := chan int{cap: 1000} //声明一个channel变量,类型为int,缓冲区大小为1000,即异步channel
	println(ch.len) // 0
	println(ch.cap) // 1000
	ch2 := chan int{} //不指定cap,默认为0,即同步channel
	println(ch2.len) // 0
	println(ch2.cap) // 0
	ch <- 1
	println(ch.len) // 1
	println(ch.cap) // 1000
}

```

### 读取channel/接收消息

```v
fn main() {
	ch := chan int{cap: 100}
	sum := <-ch //读取channel
	println(sum)
	//也可以使用try_pop()
	//尝试读channel,把channel的值,读取到i变量中,并返回ChanState枚举:.success/.not_ready/.colsed
	i := 0
	res := ch.try_pop(&i)
	println(res)
}

```

### 写入channel/发送消息

```v
fn main() {
	ch := chan int{cap: 100}
	ch <- 2 //写入channel
	//也可以使用try_push()
	//尝试写channel,把i的值写入到channel中,并返回ChanState枚举:.success/.not_ready/.colsed
	i := 3
	res := ch.try_push(&i)
	println(res)
}

```

### 错误处理

可以在读写channel中增加or代码块,实现错误处理

```v
module main

const n = 1000

const c = 100

fn f(ch chan int) {
	for _ in 0 .. n {
		_ := <-ch
	}
	ch.close()
}

fn main() {
	ch := chan int{cap: c}
	go f(ch)
	mut j := 0
	for {
		ch <- j or {  //错误处理
			break 
		}
    // ch <-j ? //向上抛转错误
		j++
	}
}
```

### 关闭channel

```v
ch.close()
```

关闭channel或者写入channel都会解除阻塞

关闭channel以后,使用try_push()和try_pop函数都会返回.closed枚举

### select语句

select语句可以同时监听多个channel的读写事件,并且可以进行监听的超时处理

一般都会结合for循环使用,实现持续监听

```v
import time

fn main() {
	ch1 := chan int{}
	ch2 := chan int{}
	go send(ch1, ch2)
	mut x := 0
	mut y := 0
	for {
		select { // select可以同时监听多个channel的读写事件
			x = <-ch1 { // 监听读channel
				println('$x')
			}
			y = <-ch2 {
				println('$y')
			}
			> 2 * time.second { // 监听的超时处理
				break
			}
		}
	}
}

fn send(ch1 chan int, ch2 chan int) {
	ch1 <- 1
	ch2 <- 2
	ch1 <- 3
	ch2 <- 4
	ch1 <- 5
	ch2 <- 6
}


```

### if select语句

```v
fn main() {
	ch1 := chan int{}
	ch2 := chan int{}
	go send(ch1, ch2)
	mut x := 0
	mut y := 0
	// ch1.close()
	// ch2.close()
	if select {
		x = <-ch1 {
			println('from x')
		}
		y = <-ch2 {
			println('from y')
		}
	} { // 如果select中的所有channel未关闭,则执行if代码块
		println('from if')
	} else { // 如果select中的所有channel都关闭,则执行else代码块
		println('from else')
	}
}

fn send(ch1 chan int, ch2 chan int) {
	ch1 <- 1
	ch2 <- 2
	println('from send')
}


```

### for select语句

for select语句主要在并发中使用,用来循环监听多个chanel

```v
fn main() {
	ch1 := chan int{}
	ch2 := chan f64{}
	go do_send(ch1, ch2)
	mut a := 0
	mut b := 0
	for select { // 循环监听channel的写入,写入后执行for代码块,直到所有监听的channel都已关闭
		x := <-ch1 {
			a += x
		}
		y := <-ch2 {
			a += int(y)
		}
	} { // for代码块
		b++ // 写入3次
		println('${b}. event')
	}
	println(a)
	println(b)
}

fn do_send(ch1 chan int, ch2 chan f64) {
	ch1 <- 3
	ch2 <- 5.0
	ch2.close()
	ch1 <- 2
	ch1.close()
}
```

### 主进程阻塞等待

一般来说,主进程执行完毕后,不会等待其他子线程的结果,就直接退出返回,其他子线程也随着终止

可以在主进程末尾增加阻塞等待子线程的运行结果

```v
module main

import time

fn main() {
	ch := chan int{} //创建同步channel
	go fn (c chan int) {
		time.sleep(3)
		println('goroutine done')
		c.close() //关闭子线程或者写channel
		// c <- 333
	}(ch)
	println('main...')
	i := <-ch // 主线程阻塞,等待子线程返回数据或者关闭channel
	println('main exit...$i')
}

```

### 泛型函数/方法

除了使用标准函数/方法作为go的并发单元,泛型函数/方法也可以

```v
module main

import time

struct Point {
}

fn (p Point) add<T>(c chan int, x T, y T) T {
	println('from generic method')
	c <- 1
	return x + y
}

fn add<T>(c chan int, x T, y T) T {
	println('from generic function')
	c <- 2
	return x + y
}

fn main() {
	ch := chan int{}
	//泛型函数
	go add<int>(ch, 1, 3)
	go add<f64>(ch, 1.1, 3.3)
	//泛型方法
	p := Point{}
	go p.add<int>(ch, 2, 4)
	go p.add<f64>(ch, 2.2, 4.4)
	i := <-ch
	println(i)
	time.sleep(1)
}
```

### sync标准模块

**Channel**

```v
//使用sync模块创建channel
mut ch := sync.new_channel<int>(0) //泛型风格
ch.cap //返回channel的缓冲区大小
ch.len() //返回channel当前已使用的缓冲大小
ch.push(&i) //写channel,一定要使用指针引用
ch.pop(&i) //读channel,一定要使用指针引用,返回bool类型,true读取成功,false读取失败
ch.try_push() //尝试写channel,返回ChanState枚举:.success/.not_ready/.colsed
ch.try_pop()  //尝试读channel,返回ChanState枚举:.success/.not_ready/.colsed
ch.close()  //关闭channel
//遍历channel
sync.channel_select()
```

**WaitGroup**

如果要等待多个并发任务结束,可以使用WaitGroup

通过设定计数器,让每一个线程开始时递增计数,退出时递减计数,直到计数归零时,解除阻塞

```v
mut wg:=sync.new_waitgroup() //创建WaitGroup
wg.add(int) //递增计数
wg.done() //递减计数
wg.wait() //阻塞等待,直到计数归零
```

```v
module main

import sync
import time

fn main() {
	mut wg := sync.new_waitgroup()
	for i := 0; i < 10; i++ {
		wg.add(1) //递增计数
		go fn (i int, mut w sync.WaitGroup) {
			defer {
				w.done() //完成后递减计数
			}
			time.sleep(1)
			println('goroutine $i done')
		}(i, mut wg)
	}
	println('main start...')
	wg.wait() //阻塞等待,直到计数器归零
	println('main end...')
}

```

输出:

```shell
main start...
goroutine 2 done
goroutine 0 done
goroutine 1 done
goroutine 3 done
goroutine 4 done
goroutine 5 done
goroutine 6 done
goroutine 8 done
goroutine 7 done
goroutine 9 done
main end...
```

更多参考代码可以查看: vlib/sync

