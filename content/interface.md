## 接口

### 接口定义

使用interface关键字定义接口,跟go一样

默认是模块级别,使用pub变为公共级别

接口命名跟结构体一样,要求首字母大写,建议以er风格结尾,非强制

```v
pub interface Speaker {
		speak() string
}
```

### 接口包含字段

接口在其他编程语言中大部分都只能包含方法签名

V语言中接口除了可以包含方法签名,也可以包含字段,虽然包含字段也可以转为方法签名,但是直接约束字段,代码看起来更舒服.

编译时会检查结构体是否实现接口字段,只有字段名,字段类型,是否可变都跟接口中的字段一致,才算实现了这个接口字段.

```v
module main

interface PointInterface {
mut:
  x int //定义接口包含字段
  y int
}

struct Point {
mut:
	x int //接口字段实现
	y int
}

fn add(p1 PointInterface,p2 PointInterface) PointInterface {
	mut p:=Point {}
	p.x=p1.x+p2.x
	p.y=p1.y+p2.y
	return p
}

fn main() {	
	p1:=Point{
		x:1
		y:2
	}
	p2:=Point {
		x:3
		y:4
	}
	p3:=add(p1,p2)
	println('x:$p3.x,y:$p3.y')
}
```

```v
interface Animal {
mut:
	name string
}

struct Cat {
	name string //mut不匹配,编译不通过
}

fn main() {
	mut animals := []Animal{}
	animals << Cat{}
}
```

### 接口方法

接口也可以像结构体那样有自己的方法,只要结构体实现了接口,就可以调用接口的方法

```v
struct Cat {
	breed string
}

interface Animal {
	breed string
}

//接口自己实现的方法
fn (a Animal) info() string {
	return "I'm a ${a.breed} ${typeof(a).name}"
}

fn new_animal(breed string) Animal {
	return &Cat{ breed }
}

fn main() {
	mut a := new_animal('persian')
	println(a.info())
}	
```



### 接口组合

目前还没有,计划0.3版本实现

```v
pub interface Reader{
	 read(mut buf []byte) ?int
}
pub interface Writer {
  write(bug []byte) ?int
}
pub interface ReaderWriter {
	Reader
  Writer
}
```

### 接口实现

没有接口实现的关键字,类型无需显示声明所要实现的接口,

鸭子类型:只要结构体实现了接口定义的方法,就满足该接口的使用

```v
module main

struct Dog {}

struct Cat {}

fn (d Dog) speak() string {
	return 'woof'
}

fn (c Cat) speak() string {
	return 'meow'
}

interface Speaker {
	speak() string
}

fn perform(s Speaker) {
	println(s.speak())
}

fn main() {
	dog := Dog{}
	cat := Cat{}
	perform(dog) // "woof"
	perform(cat) // "meow"
}

```

接口可以作为结构体字段类型使用:

```v
struct Foo {
	speaker Speaker //接口类型字段
	speakers []Speaker //接口类型数组字段
}	
```

### 接口参数类型判断

可以使用跟联合类型那样的match,as,is,对接口参数的具体类型进行判断(目前还未全部实现,只实现了is, !is)

```v
fn perform(s Speaker) {
	if s is Dog { // 通过is操作符,判断接口类型是否是某一个具体类型
		println('s is Dog')
	}
	if s !is Dog { // 通过!is操作符,判断接口类型不是某一个具体类型
		println('s is not Dog')
	}

	println(s.speak())
}
//match未实现的接口类型匹配
	match s {
		Dog { println('s is Dog struct') }
		Cat { println('s is Cat struct') }
		else { println(typeof(s)) }
	}
```

