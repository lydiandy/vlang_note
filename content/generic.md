## 泛型

目前的泛型主要有这三种：泛型结构体,泛型函数,泛型方法

### 泛型结构体

```v
module main

struct User {
mut:
	name string
}

struct DB {
	driver string
}

struct Group {
pub mut:
	name       string
	group_name string
}

struct Permission {
pub mut:
	name string
}

struct Repo <T, U> {
	db DB
pub mut:
	model      T
	permission U
}

fn main() {
	mut a := Repo<User,Permission>{
		model: User{
			name: 'joe'
		}
	}
	println(a.model.name)
	mut b := Repo<Group,Permission>{
		permission: Permission{
			name: 'superuser'
		}
	}
	println(b.model.name)
	println(b.permission.name)
	println(typeof(a.model).name)
	println(typeof(b.model).name)
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

//多类型泛型函数
fn get_multi<T,U>(typ T,user U) T {
	println(typ)
	println(user)
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

	x:=get_multi<int,f64>(2,2.2)
	println(x)
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

