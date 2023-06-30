## 方法

### 方法定义

跟go一样：在函数名前面加接收者，就是方法。

接收者默认不可变，如果要修改接收者，要加上mut。

结构体方法可以定义在同一个模块目录的不同源文件中。

```v
struct User {
mut:
	name string
	age  int
}

fn (u User) get_name() string {
	return u.name
}

fn (mut u User) set_name(name string) { //需要修改接收者,要加上mut
	u.name = name
}

fn (_ User) str() string { //如果不需要在方法中使用接收者，可使用下划线来忽略，当然也可以命名
	return 'User'
}
```

除了可以给结构体添加方法外，还可以给以下类型添加方法：

- 枚举
- 类型别名
- 函数类型
- 联合类型
- 自定义数组类型
- 自定义字典类型

```v
module main

struct Point {
	x int
	y int
}

enum Color {
	white
	black
	blue
}

type Myint = int

type MyFn = fn (int, int) int

type Mysumtype = int | string

pub fn (c Color) str() string { // 枚举的方法
	return 'from color'
}

pub fn (m Myint) str() string { // 类型别名的方法
	return 'from myint'
}

pub fn (myfn MyFn) str2() string { // 函数类型的方法
	return 'from myfn'
}

pub fn (mysum Mysumtype) str() string { // 联合类型的方法
	return 'from mysumtype'
}

pub fn (myarr []Point) point_arr_method() string { // 自定义数组类型的方法,接收者是对应的数组类型
	return 'from []Point'
}

pub fn (mymap map[string]Point) point_map_method() string { // 自定义字典类型的方法,接收者是对应的字典类型
	return 'from map[string]Point'
}

fn fn2(f MyFn) {
	println(f(1, 3)) // 直接调用函数
	println(f.str2()) // 调用函数类型的方法
}

fn add(x int, y int) int {
	return x + y
}

fn main() {
	// 枚举方法
	c := Color.white
	println(c.str())
	// 类型别名方法
	i := Myint(11)
	println(i.str())
	// 联合类型方法
	mut m := Mysumtype{}
	m = int(11)
	println(m.str())
	// 函数类型方法
	fn2(add)
	// 直接定义函数类型
	f := MyFn(add)
	println(f.str2()) // 调用函数类型的方法
	// 自定义数组类型方法
	p := Point{
		x: 1
		y: 2
	}
	mut p_array := []Point{}
	p_array << p
	println(p_array.point_arr_method())
	// 自定义字典类型方法
	mut mp := map[string]Point{}
	println(mp.point_map_method())
}

```

### 方法作为变量

方法也可以像函数那样，作为变量，作为函数的参数和返回值，只要方法签名和函数类型的签名一致就可以。

```v
module main

struct Foo {
	s string
mut:
	i int
}

fn (f Foo) get_s() string {
	return f.s
}

fn (f &Foo) get_s_ref() string {
	return f.s
}

fn (f Foo) add(a int) int {
	return a + f.i
}

fn (f &Foo) add_ref(a int) int {
	return a + f.i
}

fn main() {
	mut f := Foo{
		s: 'hello'
		i: 1
	}

	get_s := f.get_s // 方法作为变量
	get_s_ref := unsafe { f.get_s_ref } // 方法作为变量
	add := f.add // 方法作为变量
	add_ref := unsafe { f.add_ref } // 方法作为变量

	println(typeof(get_s).str())
	println(typeof(get_s_ref).str())
	println(typeof(add).str())
	println(typeof(add_ref).str())

	println(get_s()) // 方法作为变量,也可直接调用
	println(get_s_ref()) // 方法作为变量,也可直接调用
	println(add(2)) // 方法作为变量,也可直接调用
	println(add_ref(2)) // 方法作为变量,也可直接调用
}

```

更复杂的例子：

```v
module main

import gg
import gx

[heap]
struct App {
mut:
	gg     &gg.Context = 0
	radius f64 = 100.0
}

fn (mut app App) frame(x voidptr) {
	app.gg.begin()
	app.gg.draw_circle_empty(150, 150, f32(app.radius), gx.blue)
	app.gg.draw_text(20, 20, 'radius: $app.radius')
	app.gg.end()
}

fn main() {
	mut app := &App{}
	app.gg = gg.new_context(
		bg_color: gx.white
		width: 300
		height: 300
		window_title: 'Circles'
		frame_fn: app.frame // 方法也可以跟函数一样,作为变量使用
		user_data: app
	)
	app.gg.run()
}
```

### 静态方法/类型方法

V语言支持静态方法，也叫类型方法，可以用来作为结构体的构造器，使用静态方法的好处是，静态方法的前缀默认是类名，这样就可以让调用时，静态方法名有了类名的限定，看起来更直观，更易理解。

不过，如果是跨模块调用，就会变为：模块名.类名.方法名，多了一个层级，除非使用短名称导入类。大规模使用短名称导入，又会导致导入方式不一致，看起来不是那么的统一。

```v
module main

pub struct Point {
	x int
	y int
}

//静态方法或类型方法,作为构造器
pub fn Point.new(x int, y int) Point {
	return Point{x, y}
}

//普通函数作为构造器
pub fn new_point(x int, y int) Point {
	return Point{x, y}
}

pub fn main() {
	p := Point.new(1, 2) //静态方法作为构造器
	println(p)
	p2 := new_point(2, 3) //普通函数作为构造器
	println(p2)
}

```



### 方法链式调用

V语言支持方法链式调用。

只是新的V编译器增加了限制，如果方法的接收者是可变的话，也就是在方法中修改了结构体字段，就不允许链式调用，希望后续的版本能够重新支持。

以下示例代码可以编译通过：

```v
module main

[heap]
struct DB {
mut:
	sql string
}

fn new() DB {
	return DB{
		sql: ''
	}
}

fn (db DB) table(name string) DB {
	// db.sql += name
	println('from table')
	return db
}

fn (db DB) where(condition string) DB {
	// db.sql += condition
	println('where')
	return db
}

fn (db DB) first() DB {
	// db.sql += 'limit 1'
	println('from first')
	return db
}

fn main() {
	mut db := new()
	// 链式调用
	db.table('select * from user ').where('where id=1 ').first()
	println(db.sql)
}
```

以下示例代码无法编译通过，因为方法的接收者是可变的：

```v
module main

struct DB {
mut:
	sql string
}

fn new() &DB { // &表示取地址,引用
	return &DB{
		sql: ''
	}
}

fn (mut db DB) table(name string) &DB { // &表示取地址,引用
	db.sql += name
	return db
}

fn (mut db DB) where(condition string) &DB {
	db.sql += condition
	return db
}

fn (mut db DB) first() &DB {
	db.sql += 'limit 1'
	return db
}

fn main() {
	mut db := new()
	// 链式调用
	db.table('select * from user ').where('where id=1 ').first()
	println(db.sql) // 输出:select * from user where id=1 limit 1
}
```

