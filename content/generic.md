## 泛型

目前的泛型主要有这三种：泛型结构体,泛型函数,泛型方法

### 泛型结构体

```v
struct DB {
    driver string
}

struct User {
	db DB
mut:
	name string
}

struct Repo<T> {
	db DB
mut:
	model  T
}

fn new_repo<U>(db DB) Repo<U> {
	return Repo<U>{db: db}
}

fn test_generic_struct() {
	mut a :=  new_repo<User>(DB{})
	a.model.name = 'joe'
	mut b := Repo<User>{db: DB{}}
	b.model.name = 'joe'
	assert a.model.name == 'joe'
	assert b.model.name == 'joe'
}
```

### 泛型函数

泛型函数的调用可以有标准调用和简短调用2种方式

简洁方式只有泛型在参数中使用才可以,编译器会自动根据参数具体的类型,传递类型信息给函数

```v
module main
 
//泛型函数
fn get<T>(typ T) T {
	return typ
}

fn main() {
	a1 := get<string>('hello') //标准的泛型调用方式,带<T>
	a2 := get<int>(1)
	println(a1)
	println(a2)
	b1 := get('hello') //简短的泛型调用方式,不用带<T>
	b2 := get(1)
	println(b1)
	println(b2)
}
```

### 泛型方法

```v
module main

struct Point {
mut:
	x int
	y int
}

//泛型方法
fn (mut p Point) translate<T>(x T, y T) {
	p.x += x
	p.y += y
}

struct Foo {}

//泛型方法
fn (v Foo) new<T>() T {
	return T{}
}

fn main() {
	mut pot := Point{}
	pot.translate<int>(1, 3) //标准调用方式
	pot.translate(1, 3) //简洁调用方式
	println(pot)
	//
	foo := Foo{}
	//泛型类型为数组
	f := foo.new<[]string>()
	println(f)
	//泛型类型为字典
	f2 := foo.new<map[string]string>()
	println(f2)
}

```

