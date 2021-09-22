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

### go表达式

除了使用标准的chanel和waitgroup方式外,还可以使用go表达式来简化并发代码,go表达式更像是一种并发语法糖/简化版.

```v
module main

import time

//无返回值的函数
fn do_something() { 
  println('start do_something...')
	time.sleep(2*time.second) //休眠2秒,模拟并发持续的时间
	println('end do_something')
}
//有返回值的函数
fn add(x int, y int) int { 
	println('add并发开始...')
	time.sleep(4*time.second) //休眠4秒,模拟并发持续的时间
	println('add并发完成')
	return x + y
}

fn main() {
  //并发调用无返回值的函数
  g:=go do_something()
	//并发调用带有返回值的函数,完成后返回
  g2 := go add(3, 2) 
	//这段期间主线程可以继续执行别的代码...
  g.wait() //阻塞等待线程完成
	result := g2.wait() //阻塞等待线程结果返回
	println(result)
}
```

### thread线程数组

go表达式实现了并发执行后,然后返回单个结果给主线程

而thread线程数组实现了并发执行多个线程,然后返回结果数组给主线程,用起来挺简洁明了的

```v
module main

fn f(x f64) f64 {
	y := x * x
	return y
}

fn g(shared a []int, i int) {
	lock a { //读写锁定,共享变量a
		a[i] *= a[i] + 1
	}
}

fn main() {
	mut r := []thread f64{cap: 10} //轻量级线程数组,每一个线程的返回值类型是f64
	for i in  0 .. 10 {
		r << go f(f64(i) + 0.5)
	}
	x := r.wait() //线程数组有一个内置的wait方法,等待线程数组的所有线程全部运行完毕,并返回结果数组
	println(x) //[0.25, 2.25, 6.25, 12.25, 20.25, 30.25, 42.25, 56.25, 72.25, 90.25]

	shared a := [2 3 5 7 11 13 17]
	t := [
		go g(shared a, 0)
		go g(shared a, 3)
		go g(shared a, 6)
		go g(shared a, 2)
		go g(shared a, 1)
		go g(shared a, 5)
		go g(shared a, 4)
	]
	println('threads started')
	t.wait()
	rlock a {
		println(a) //[6, 12, 30, 56, 132, 182, 306]
	}
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

go表达式和go线程数组也支持同样的错误处理方式

```v
module main

import time

//无返回值的函数
fn do_something() ? {  //函数带错误处理
  println('start do_something...')
	time.sleep(2*time.second) //休眠2秒,模拟并发持续的时间
	println('end do_something')
}
//有返回值的函数
fn add(x int, y int) ?int {  //函数带错误处理
	println('add并发开始...')
	time.sleep(4*time.second) //休眠4秒,模拟并发持续的时间
	println('add并发完成')
	return x + y
}

fn main() {
  //并发调用无返回值的函数
  g :=go do_something() 
	//并发调用带有返回值的函数,完成后返回
 	g2 := go add(3, 2)
	//这段期间主线程可以继续执行别的代码...
  g.wait() or { panic(err) } //支持错误处理的并发
	result := g2.wait() or { panic(err) } //支持错误处理的并发
	println(result)
}
```

### if条件语句读取chan

```v
fn main() {
	mut res := []f64{cap:3}
	ch := chan f64{cap: 10}
	ch <- 6.75
	ch <- -3.25
	ch.close()
	for _ in 0 .. 3 {
		//读取chan成功,则if条件表达式返回true;读取chan失败,chan被关闭,if条件表达式返回false
		if x:= <-ch {
			res << x
		} else {
			res << -37.5
		}
	}
	println(res) // 返回[6.75, -3.25, -37.5]
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
		time.sleep(3*time.second)
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
	time.sleep(1*time.second)
}
```

### 线程之间的变量共享/锁定

可以使用shared/lock/rlock来实现

多个线程之间,可以通过定义shared类型的变量,来实现线程间共享,所有线程都可以读写该变量.

当某个线程要进行读写共享变量时,为了防止线程之间的数据竞争:

在读写之前,要使用lock代码块(读写锁)来锁定共享变量

在只读之前,要使用rlock代码块(只读锁)来锁定共享变量

共享变量可以是基本类型,array,map,struct类型

```v
module main

import time

struct St {
mut:
	x f64
}

fn f(x int, y f64, shared s St,shared a []string, shared m map[string]string) {
	time.sleep(50*time.second)
	//在这个线程中,如果要对共享变量进行读写,使用lock代码块来锁定,对于读写锁,其他线程只能阻塞等待,不能读写该变量,退出代码块后,自动解锁
	lock s,a,m {  //可以同时对多个共享变量进行锁定
		s.x = x * y
		println(s.x)
		a[0]='abc'
		unsafe {
			m['a']='aa'
		}
		println(a[0])
		println(m['a'])
	}
	return
}

fn main() {
	shared s := St{}  // struct共享变量
	shared a := []string{len:1} // 数组共享变量
	shared m := map[string]string // 字典共享变量
	unsafe {
		m['a']='aa'
	}
	r := go f(3, 4.0, shared s,shared a, shared m)  //把共享变量传递给另一个线程,默认传递引用
	r.wait()
	//在这个线程中,如果只是要读共享变量,使用rlock代码来锁定,对于只读锁,其他线程可以读该变量,不能修改,退出代码块后,自动解锁
	rlock s { 
		println(s.x)
	}
}
```

函数返回shared类型

```v
struct St {
mut:
	x f64
}

fn f() shared St { //函数可以返回shared的变量,用于线程之间的读写锁
	shared x := St{ x: 3.25 }
	return x
}

fn g(good bool) ?shared St { //函数可以返回shared的变量,用于线程之间的读写锁,结合错误处理
	if !good {
		return error('no shared St created')
	}
	shared x := St{ x: 12.75 }
	return x
}

fn shared_opt_propagate(good bool) ?f64 {
	shared x := g(good) ?
	ret := rlock x { x.x }
	return ret
}

fn main() {
	shared x := f()
	val := rlock x { x.x }
	println(val)

	res := shared_opt_propagate(true) or { 1.25 }
	println(res)
}

```

读写锁表达式

```v
struct St {
mut:
	i int
}

fn main() {
	shared xx := St{ i: 173 }
	shared y := St{ i: -57 }
	mut m := 0
	m = lock y { y.i } //读写锁表达式
	n := rlock xx { xx.i } //读表达式
	println(m)
	println(n)
}
```

### 易变变量

跟C语言一样，V语言也有volatile关键字，用来标识易变变量。

V语言中的变量根据可变性有以下三种：不可变(默认)，可变(mut)，易变(volatile)。

易变变量一定也是可变的，所以一定是跟mut同时使用的。

用volatile关键字修饰的变量表示：该变量可能被某些编译器未知的因素更改，比如操作系统、硬件或者其它线程等，

volatile 关键字告诉编译器，该变量的值是随时可能发生变化的，编译器对访问该变量的代码就不再进行优化，每次使用它的时候必须从内存中取出该变量的值。

```v
module main

fn myfn() {
	mut volatile zzz := 123	//易变变量
	println(zzz)
}

fn main() {
	x := 123
	y := 456
	mut volatile p := unsafe { &x }	//易变变量
	println(p)
	p = unsafe { &y }
	println(p)
	myfn()
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
			time.sleep(1*time.second)
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

