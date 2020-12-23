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

