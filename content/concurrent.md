## 并发

V语言并发的思路和语法跟go一样,甚至关键字也一样:go和chan

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

更多参考代码可以查看: vlib/sync

