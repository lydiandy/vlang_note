## 结构体

### 结构体定义

结构体命名必须是大写字母开头,如果是小写字母开头会编译报错

```v
struct Point {
	x int
	y int
}

fn main() {
	p := Point{
		x: 10
		y: 20
	}
	println(p.x) // 结构体字段通过点号来访问
}
```

结构体被分配到内存的栈中,引用类型

取结构体地址:&

```v
p2 := &Point{10, 10}
println(p2.x)
```

空结构体:

```v
struct Empty {}

fn main() {
	println(sizeof(Empty)) //空结构体占用内存的大小为0
}
```

结构体占用内存:

sizeof( )可以返回结构体占用内存字节大小

### 字段

结构体字段默认也是不可变,使用mut为可变

结构体字段的可变性和访问控制,参考[访问控制](./access_controll.md)章节

如果结构体字段名需要是关键字,可以通过使用@作为前缀也可以编译通过

这一点在跟C库集成时,比较常用,一些C库的struct的字段有些刚好是V的关键字,可以使用@前缀,编译成C代码时@前缀会被去掉,刚好就是C的struct中的字段名

```v
struct Foo {
	@type string
}

fn main() {
	foo := Foo{ @type: 'test' }
	println(foo.@type)
}
```

结构体字段支持默认值

```v
struct Foo {
	a int
	b int = 7 //默认值7
}

fn main() {
	foo := Foo{
		a: 1
	}
	println(foo.a) //输出1
	println(foo.b) //输出默认值7
	foo2 := Foo{
		a: 1
		b: 2
	}
	println(foo2.b) //输出2
}

```

结构体变量基于另一个变量创建,同时合并新的字段值

目前有2种方式,估计会保留第2种

```v
struct Foo {
	a int
	b int = 7 //默认值7
}

fn main() {
	foo := Foo{
		a: 1
		b: 33
	}
	foo2 := {
		foo |
		a: 42
	} // foo2是在foo的基础上,通过|合并新的字段值
	println(foo2.a) //输出合并后的新值42
	println(foo2.b) //输出未改变的值33
}

```

```v
module main

struct City {
	name       string
	population int
}

struct Country {
	name    string
	capital City
}

fn main() {
	c := Country{
		name: 'test'
		capital: City{
			name: 'city'
		}
	}
	c2 := Country{
		...c  //第2种方式
		capital: City{
			name: 'city2'
			population: 200
		}
	}
	c3 := {
		c | //第1种方式
		name: 'test3'
		captial: City{
		name: 'city3'
		population: 300
	}
	}
	println(c2)
	println(c3)
}
```

短字面量创建结构体变量

当函数的参数是结构体变量时,这个语法可以简化结构体变量的创建,这个语法在ui模块比较常用到,用来简化函数参数

```v
module main

struct User {
	name string
	age int
}

fn add(u User) {
	println(u)
}

fn main(){
	add(User{name:'jack',age:22}) //标准方式
	add({name:'tom',age:23}) //简短方式,省略类型
	add(name:'tt',age:33) //更简短的方式,省略类型和大括号,这个用法感觉分辨不出来参数是结构体,推荐使用方式二,兼具简短和清晰性
}
```

### 访问控制

结构体默认是模块级别的,使用pub关键字定义公共级别

```v
pub struct Point { //公共级别,可以被别的模块使用
	x int
	y int
}
```

### 组合

跟go一样,V的结构体没有继承,但是有组合,可以用组合来实现继承一样的效果,甚至可以多个组合

```v
module main

struct Widget {
mut:
	x int
	y int
}

pub fn (mut w Widget) move(x_step int, y_step int) {
	w.x += x_step
	w.y += y_step
}

struct Widget2 {
mut:
	z int
}

pub fn (mut w Widget2) move_z(z_step int) {
	w.z += z_step
}

struct Button {
	Widget 	//组合
	Widget2 //多个组合
	title string
}

fn main() {
	mut button := Button{
		title: 'Click me'
	}
	button.x = 3 // Button的x字段来自Widget
	button.z = 4 // Button的z字段来自Widget2
	println('x:$button.x,y:$button.y,z:$button.z')
	button.move(3, 4) // Button的move方法来自Widget
	println('x:$button.x,y:$button.y,z:$button.z')
	button.move_z(5) // Button的move_z方法来自Widget2
	println('x:$button.x,y:$button.y,z:$button.z')
}

```

### 泛型结构体

参考[泛型章节](./generic.md)

### 结构体标注

**[typedef]**

```v
[typedef]
struct Point {
}
```

typedef标注目前主要用在集成C代码库,详细参考:

**[ref_only]**

ref_only表示该结构体创建的变量不能被复制,只能以引用的形式被使用

  ```v
[ref_only]
struct Window {  //只能通过引用的形式(&Window)来使用这个结构体
}

  ```

可以参考标准库sync中WaitGroup的实际使用

vlib/sync/waitgroup.v

```v
[ref_only] 
struct WaitGroup {
mut:
	task_count       int 
	task_count_mutex &Mutex = &Mutex(0) 
	wait_blocker     &Waiter = &Waiter(0) 
}

pub fn new_waitgroup() &WaitGroup { //不能被复制,只能以引用的方式被使用
	return &WaitGroup{
		task_count_mutex: new_mutex()
		wait_blocker: new_waiter()
	}
}
```

### 结构体字段标注

1. 用于内置json解析支持

   详细参考:[json章节](./json.md)

2. 结构体字段必须初始化赋值标注

```v
struct Point {
	x int
	y int
	z int [required] //字段标注required表示必须初始化赋值
}

fn main() {
	a := Point{
		// x: 1 		//不会报错,x不是必须初始化赋值
		y: 3
		z: 5
	}
	b := Point{
		x: 2
		y: 4
		// z: 6     //报错: field `Point.z` must be initialized
	}
	println(a)
	println(b)
}

```

### 结构体方法标注

跟函数标注一样,也可以对结构体方法进行标注

```v
[inline] //方法标注
pub fn (p Point) position() (int, int) {
	return p.x, p.y
}
```

### 结构体反射

可以通过编译时反射,实现动态获取结构体的所有字段,素有方法,并且可以动态设置字段值

具体可以参考[编译时反射章节](comptime.md)