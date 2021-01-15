## 枚举

### 定义枚举

枚举默认是模块内访问，通过pub关键字来定义公共枚举

枚举的命名跟结构体命名一样,必须以大写字母开头;枚举项的名称跟函数命名一样,必须是小写加下划线

```v
enum Color {
	blue // 如果没有指定初始值，默认从0开始，然后往下递增1
	green
	white
	black
}

fn main() {
	mut c := Color.green // 第一次定义要使用：枚举名称.枚举值
	println(c) // 输出green
	c = Color.blue
	c = .blue // 第二次修改赋值，也可以忽略枚举名称,直接使用.枚举值就可以了
	println(c) // 输出blue
}

```

也可以指定枚举值的值，枚举值也可以是负数

```v
enum Color2 {
	blue = 2 //可以指定初始值
	green
	white
	black
}

enum Color3 {
	blue = -4 //初始值也可以是负数
	green
	white
	black
}

fn main() {
	mut c := Color2.green
	println(c) //输出green
	c = .blue
	println(c) //输出blue
}

```

也可以指定枚举的值为16进制

```v
enum W_hex {
	a = 0x001 //枚举值也支持16进制
	b = 0x010
	c = 0x100
}

fn main() {
	println(W_hex.a) //输出a
	println(W_hex.b) //输出b
	println(W_hex.c) //输出c
}
```

### 枚举方法

枚举也可以像结构体那样添加方法

```v
enum Color {
	red = 1
	green
	blue
	black
	white
}

fn (c Color) is_blue() bool { // 枚举方法
	return c == .blue
}

fn main() {
	b := Color.blue
	if b.is_blue() {
		println('yes')
		println(b)
	} else {
		println('no')
		println(b)
	}
}
```

### 枚举值/整型相互转换

枚举类型和整数类型是可以相互转换的

```v
enum Color {
	red = 1
	green
	blue
	black
	white
}

fn main() {
	i := 3 // 推断为int
	// println(i==.blue) 	//报错,类型不匹配
	e := Color(i) // 转换为枚举类型
	println(e == .blue) // 输出true
	ii := int(e) // 枚举类型转换为int
	println(ii) // 输出3
	iii := e // iii是枚举类型
	println(typeof(iii).name)
	println(iii)
}
```

### 枚举类型数组

可以定义枚举类型的数组,数组的值是某个枚举项

```v
struct Abc {
mut:
	flags []Flag
}

enum Flag {
	flag_one
	flag_two
	flag_three
}

fn main() {
	mut a := Abc{}
	a.flags << .flag_one
	a.flags << .flag_two
	a.flags << .flag_three
	println(a.flags)
}

```

###  枚举标注

​	可以像结构体和函数那样,给枚举添加自定义标注

```v
[attr1]
[attr2]
pub enum Color {
	black
	white
	blue
}
```

关于标注的进一步使用,可以参考[标注章节](attribute.md)