## 结构体

### 结构体定义

结构体命名必须是大写字母开头，如果是小写字母开头会编译报错。

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

结构体被分配到内存的栈中，引用类型。

使用&来取结构体的地址 

```v
p2 := &Point{10, 10}
println(p2.x)
```

空结构体：

```v
struct Empty {}

fn main() {
	println(sizeof(Empty)) //空结构体占用内存的大小为0
}
```

sizeof( )可以返回结构体占用内存字节大小。

### 字段

结构体字段默认也是不可变，使用mut为可变。

结构体字段的可变性和访问控制，参考[访问控制章节](./access_controll.md)。

如果结构体字段名需要是关键字，可以通过使用@作为前缀也可以编译通过。

这一点在跟C库集成时比较常用，一些C库的struct的字段有些刚好是V的关键字，可以使用@前缀，编译成C代码时@前缀会被去掉，刚好就是C的struct中的字段名。

```v
struct Foo {
	@type string
}

fn main() {
	foo := Foo{ @type: 'test' }
	println(foo.@type)
}
```

结构体字段支持默认值：

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

结构体字段的类型可以是任何类型，甚至可以是函数类型：

```v
module main

struct Abc {
	f1 int             
	f2 int
	f3 fn (int, int) int //结构体字段类型为函数类型
}

fn add(x int, y int) int {
	return x + y
}

fn main() {
	a1 := Abc{
		f1: 123
		f3: fn (x int, y int) int {
			return x + y
		}
	}
	a2 := Abc{
		f1: 123
		f2: 789
		f3: add
	}
	println(a1)
	println(a2)
}

```

结构体字段初始化时必录：

```v
struct Point {
	x int
	y int
	z int [required] //字段注解required表示必须初始化赋值
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

结构体变量可以基于另一个变量创建，同时合并新的字段值：

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
		...c  // 在c的基础上创建c2变量
		capital: City{
			name: 'city2'
			population: 200
		}
	}
	println(c2)
}
```

当函数的参数是结构体变量时，这个语法可以简化结构体变量的创建，这个语法在ui模块比较常用到，用来简化函数参数。

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
	add(name:'tt',age:33) //简短的方式,省略类型和大括号,有命名参数调用的感觉
}
```

### 访问控制

结构体默认是模块级别的，使用pub关键字定义公共级别。

```v
pub struct Point { //公共级别,可以被别的模块使用
	x int
	y int
}
```

### 组合

跟go一样，V的结构体没有继承，但是有组合，可以用组合来实现继承一样的效果，多个组合也是可以的。

```v
module main

struct Widget {
mut:
	x int
	y int
}

pub fn new_widget(x int, y int) Widget {
	return Widget{
		x: x
		y: y
	}
}

pub fn (mut w Widget) move(x_step int, y_step int) {
	w.x += x_step
	w.y += y_step
}

struct Widget2 {
mut:
	z int
}

pub fn new_widget2(z int) Widget2 {
	return Widget2{
		z: z
	}
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

### 匿名结构体

在定义结构体字段时，除了使用已经定义好的结构体作为类型，也可以使用匿名结构体。

```v
module main

struct Book {
	x      Foo
	author struct  { //匿名结构体
		name string
		age  int
	}

	title string
}

fn main() {
	book := Book{author:struct{'sdf', 23}} // 初始化匿名结构体字段时，也需要使用struct关键字
	println(book.author.age)
}
```



### 结构体初始化

常用的结构体初始化有2种：

1. 字面量初始化
2. 构造函数初始化

```v
//字面量初始化	
mut button := Button{title: 'Click me'}

//构造函数:就是一个普通的函数,一般都是new或者new_xxx,返回对应的结构体
pub fn new_button(title string) Button {
	return Button{
		title: title
	}
}
```

带组合的结构体初始化

```v
struct Button {
	Widget //组合1
	Widget2 //组合2
pub:
	title string
}
// 带组合的结构体初始化
mut button3 := Button{
		Widget: Widget{
			x: 1
			y: 2
		}
		Widget2: Widget2{
			z: 3
		}
		title: 'button3'
}
// 使用构造函数也可以
mut button4 := Button{
		Widget: new_widget(1, 2)
		Widget2: new_widget2(3)
		title: 'button4'
}
```

### 泛型结构体

参考[泛型章节](./generic.md)。

### 结构体注解

结构体像函数那样支持注解，可以针对结构体，结构体字段，结构体方法进行注解。

#### [typedef]

```v
[typedef]
struct Point {
}
```

typedef注解目前主要用在集成C代码库，详细参考[集成C代码章节](c.md)。

#### [heap]

heap表示该结构体只能在内存的堆上创建，只能以引用的形式被使用。

  ```v
[heap]
struct Window {  //只能通过引用的形式(&Window)来使用这个结构体
}
  ```

可以参考标准库sync中WaitGroup的实际使用：

vlib/sync/waitgroup.v

```v
[heap] 
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

#### [noinit]

使用[noinit]标志后，结构体只能在本模块内使用Foo{ }来创建变量，在其他模块中禁止直接使用字面量Foo{ }来初始化变量，只能使用模块提供的构造函数来创建。

```v
module mymodule

[noinit]
pub struct Result {
}

//模块内的构造函数
pub fn new_result() Result {
	return Result{}
}
```

```v
module main

import mymodule

fn main() {
  // 报错：it cannot be initialized with `mymodule.Result{}`
	// res := mymodule.Result{} 
	res := mymodule.new_result() //调用模块提供的构造函数是可以的
	println(res)
}
```

#### [params]

使用结构体的参数注解，可以用来实现函数参数的默认值和命名参数。

```v
//参数注解
[params]
struct ButtonConfig {
	text        string
	is_disabled bool
	width       int = 70
	height      int = 20
}

struct Button {
	text   string
	width  int
	height int
}

fn new_button(c ButtonConfig) &Button {
	return &Button{
		width: c.width
		height: c.height
		text: c.text
	}
}

fn main() {
	button := new_button(text: 'button1', width: 100)
	println(button.height) // height没有初始化,使用默认值

	button2 := new_button(width: 100, text: 'button2') //结构体参数的字段顺序无所谓
	println(button2)

	//结构体参数加了params注解,函数的参数可以什么都不传,直接使用结构体的默认值
	button3 := new_button()
	println(button3) //所有参数都使用默认值
}
```

#### [table]

在内置的orm中使用，用于定义模型结构体对应的数据库表名

```v
[table: 'userlist']	//自定义表名
struct User {
	id             int    [primary; sql: serial]
	age            int
	name           string [sql: 'username']
	is_customer    bool
	skipped_string string [skip]
}
```

#### [minify]

对结构体的内存布局进行优化，占用内存空间更小，会使用C语言的位字段，把布尔类型从占用1个字节精简为1个位，把枚举占用的内存字节数也降低。这个参数已被使用到V的编译器代码中，编译器的内存占用大约降低了8M。

```v
[minify]
struct Point {
	x int
	y int
}
```

### 结构体字段注解

#### [required]

要求结构体字段在初始化的时候必须赋值

1. 用于内置json解析支持，详细参考[json章节](./json.md)。

2. 结构体字段必须初始化赋值注解。

```v
struct Point {
	x int
	y int
	z int [required] //字段注解required表示必须初始化赋值
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

#### [deprecated]

结构体字段作废

可以将结构体的某个字段标注为作废字段，并且可以加上作废开始日期。

只有在其他模块中直接访问作废字段,才会出发过期作废的提示或警告。

abc.v

```v
module abc

pub struct Point {
pub mut:
	x_new int
	x int [deprecated: 'use Point.x_new instead'; deprecated_after: '2999-03-01']
}
```

main.v

```v
module main

import abc

fn main() {
	a := abc.Point{
		x: 1
	}
	println(a.x) //直接访问了结构体的作废字段,就会提示或警告
}

```

### 结构体方法注解

跟函数注解一样，也可以对结构体方法进行注解。

```v
[inline] //方法注解
pub fn (p Point) position() (int, int) {
	return p.x, p.y
}
```

### 结构体反射

可以通过编译时反射，实现动态获取结构体的所有字段，所有方法，并且可以动态设置字段值。

具体可以参考[编译时反射章节](comptime.md)。

### 结构体字段内存位置偏移

__offsetof函数实现C的offsetof函数那样,返回结构体中某个字段的内存偏移量。

```v
module main

struct User {
	name [50]u8
	age int
	desc string
}

fn main() {
	offset_name:=__offsetof(User,name)
	offset_age:=__offsetof(User,age)
	offset_desc:=__offsetof(User,desc)
	println(offset_name)
	println(offset_age)
	println(offset_desc)
}
```
