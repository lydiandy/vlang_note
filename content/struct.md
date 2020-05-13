## 结构体

### 结构体定义

v结构体的名字必须是大写字母开头,如果是小写字母开头会编译报错

```c
struct Point {
	x int
	y int
}

p := Point{
	x: 10
	y: 20
}
println(p.x) // 结构体字段通过点号来访问
```

结构体被分配到内存的栈中,引用类型

取结构体地址:&

```c
p := &Point{10, 10}
println(p.x)
```

空结构体:

```c
struct Empty {}

fn main() {
	println(sizeof(Empty)) //空结构体占用内存的大小为0
}
```

结构体占用内存:

sizeof( )可以返回结构体占用内存字节大小

### 字段

结构体字段默认也是不可变,使用var为可变

结构体字段的可变性和访问控制,参考[访问控制](./access_controll.md)章节

如果结构体字段名需要是关键字,可以通过使用@作为前缀也可以编译通过

这一点在跟C库集成时,比较常用,一些C库的struct的字段有些刚好是V的关键字,可以使用@前缀,编译成C代码时@前缀会被去掉,刚好就是C的struct中的字段名

```c
struct Foo {
	@type string
}

fn main() {
	foo := Foo{ @type: 'test' }
	println(foo.@type)
}
```

结构体字段支持默认值

```c
struct Foo {
	a int
	b int = 7 //默认值7
}
fn main() {
	foo := Foo{a:1}
	println(foo.a) //输出1
	println(foo.b) //输出默认值7
	foo2:= Foo{a:1,b:2}
	println(foo2.b) //输出2
}
```

结构体变量基于另一个变量创建,同时合并新的字段值

```c
struct Foo {
	a int
	b int = 7 //默认值7
}
fn main() {
	foo := Foo{a:1}
	foo2 := { foo | a: 42 }  //foo2是在foo的基础上,通过|合并新的字段值
	println(foo2.a) //输出合并后的新值42
	println(foo2.b) //输出未改变的值7
}
```

短字面量创建结构体变量

当函数的参数是结构体变量时,这个语法可以简化结构体变量的创建,这个语法在ui模块比较常用到,用来简化函数参数

```c
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
	add(name:'tt',age:33) //更简短的方式,省略类型和大括号
}
```

### 访问控制

结构体默认是模块级别的,使用pub关键字定义公共级别

```c
pub struct Point { //公共级别,可以被别的模块使用
	x int
	y int
}
```

### 组合

struct 可以组合, 用来实现继承的效果(目前还没有实现)

```c
struct Button {
	Widget //组合
	title string
}

button := new_button('Click me')
button.set_pos(x, y)
```

### 泛型结构体

参考[泛型章节](./generic.md)

### 结构体标注

结构体支持字段的标注功能,目前主要用于内置json解析支持,详细参考:[json章节](./json.md)

