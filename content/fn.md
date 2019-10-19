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

V语言的函数定义(函数签名)基本跟go一样

去除各种视觉干扰的符号,简洁清晰,而函数关键字是跟rust一样的fn,更简洁

函数的命名采用rust一样的小写+下划线的风格，看着很舒服,而不是采用go的大小写来区分可访问性,这样就不会强制要大小写

### 函数访问控制

默认模块内部访问,使用pub才可以被模块外部访问

```
module mymodule
fn private_fn() { //模块内部可以访问,模块外部不可以访问

}
pub public_fn(){  //模块内部和外部都可以访问

}
```

### 函数参数

函数的参数采用的是类型后置,相同类型的相邻参数可以合并类型

函数的参数默认是不可变的,如果在函数内部要改变传进来的参数,要加上mut

```
fn my_fn(arr mut []int) {
	for i := 0; i < arr.len; i++ {
		arr[i] *= 2
	}
}

mut nums := [1, 2, 3] //传进来的参数要是可变的
my_fn(mut nums) //参数定义也要是可变的
println(nums) // ==> "[2, 4, 6]" //函数执行后,传进来的参数被改变了
```



### 函数返回值

函数的返回值除了可以是单返回值外,也可以是多返回值

```
fn bar() int { //单返回值
	return 2
}

fn foo() (int, int) { //多返回值
	return 2, 3
}

a, b := foo()
println(a) // 2
println(b) // 3
```



### 函数defer语句

在函数退出前执行defer代码段,一般用来在函数执行完毕后,释放资源的占用

一个函数可以有多个defer代码块,采用后定义先执行的原则

```
fn main(){
    println('main start')
    
    defer {defer_fn1()} 
    defer {defer_fn2()}
    
    println('main end')
}

fn defer_fn1(){
    println('from defer_fn1')
}

fn defer_fn2(){
    println('from defer_fn2')
}
```

执行结果:

```
main start
main end
from defer_fn2
from defer_fn1
```



### 函数类型

相同的函数签名,表示同一类函数,可以用type定义为函数类型

```
type mid_fn fn(int,string) int
```

### 函数作为参数或返回值

函数是first class,可以将函数作为参数,作为返回值

```
fn sqr(n int) int {
        return n * n
}

fn run(value int, op fn(int) int) int {
        return op(value)
}

fn main()  {
        println(run(5, sqr)) // "25"
}
```



### 函数递归

函数也是可以递归调用的

### 泛型函数

也是支持的,目前感觉还不是太成熟