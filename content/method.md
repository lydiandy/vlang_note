## 方法

### 方法定义

跟go一样,在函数名前面加接收者,就是方法

接收者默认不可变,如果要修改接收者,要加上mut

结构体方法,可以定义在同一个模块目录的不同源文件中

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

```

除了可以给结构体添加方法外,还可以给以下类型添加方法:

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
	f := main.MyFn(add)
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

### 方法链式调用

方法支持链式调用

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

