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

fn (mut u User) set_name(name string) {
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

fn main() {
	i := Myint(11)
	println(i.str())
	mut m := Mysumtype{}
	m = int(11)
	println(m.str())
	c := Color.white
	println(c.str())
}

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

type Myfn = fn ( int,  int) int

type Mysumtype = int | string

pub fn (c Color) str() string { // 枚举的方法
	return 'from color'
}

pub fn (m Myint) str() string { // 类型别名的方法
	return 'from myint'
}

pub fn (myfn Myfn) str() string { // 函数类型的方法
	return 'from myfn'
}

pub fn (mysum Mysumtype) str() string { // 联合类型的方法
	return 'from mysumtype'
}

pub fn (myarr []Point) point_arr_method() { // 自定义数组类型的方法,接收者是对应的数组类型
}

pub fn (mymap map[string]Point) point_map_method() { // 自定义字典类型的方法,接收者是对应的字典类型
}


```



### 方法链式调用

结构体方法支持链式调用

按照目前的测试结果,&取地址符号,在函数返回值上加就可以了,不需要在接收者上也添加

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

