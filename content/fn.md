## 函数

### 函数定义

```
fn main() {
	println(add(77, 33))
	println(sub(100, 50))
}

fn add(x int, y int) int {
	return x + y
}

fn sub(x, y int) int {
	return x - y
}
```

V语言的函数定义基本跟go一样,非常简洁清晰,而函数关键字是跟rust一样的fn,更简洁









V的函数签名真的看着很舒服，很简洁，基于go，但是关键字却是采用rust更简洁的fn

函数的命名采用rust一样的小写+下划线的风格，看着很舒服；而不是采用go的大小写来区分可访问性,这样就不会强制要大小写

函数的参数默认是不可变的,如果在函数内部要改变传进来的参数,要加上mut

```
fn multiply_by_2(arr mut []int) {
	for i := 0; i < arr.len; i++ {
		arr[i] *= 2
	}
}

mut nums := [1, 2, 3] //传进来的参数要是可变的
multiply_by_2(mut nums) //参数定义也要是可变的
println(nums) // ==> "[2, 4, 6]" //函数执行后,传进来的参数被改变了
```

函数是一级class,可以定义一类函数,可以将函数作为参数,作为返回值

函数也跟go一样支持defer

在函数退出前执行defer代码段,一般用来释放资源的占用

一个函数可以有多个defer代码块,采用后定义先执行的原则





