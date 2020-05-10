## 接口

### 接口定义

使用interface关键字定义接口,跟go一样

默认是模块级别,使用pub变为公共级别

接口命名跟结构体一样,要求首字母大写,建议以er风格结尾,非强制

```go
pub interface Speaker {
		speak() string
}
```

目前没有看到接口可以进行组合或者继承的代码

### 接口实现

没有接口实现的关键字,类型无需显示声明所要实现的接口,

鸭子类型:只要结构体实现了接口定义的方法,就满足该接口的使用

```c
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
	speak()string
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

```c
struct Foo {
	speaker Speaker //接口类型字段
	speakers []Speaker //接口类型数组字段
}	
```

### 接口参数类型判断

可以使用is关键字,对接口参数的具体类型进行判断

```c
fn perform(s Speaker) {
	if s is Dog { //通过is关键字,判断接口类型是否是某一个具体类型
		println('s is Dog')
	}

	println(s.speak())
}
```

