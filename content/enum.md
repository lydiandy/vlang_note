## 枚举

### 定义枚举

枚举默认是模块内访问，通过pub关键字来定义公共枚举。

枚举的命名跟结构体命名一样，必须以大写字母开头，枚举项的名称跟函数命名一样，必须是小写加下划线。

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
	c = .blue // 第二次修改赋值，也可以忽略枚举名称，直接使用.枚举值就可以了
	println(c) // 输出blue
}

```

也可以指定枚举值的值，枚举值也可以是负数：

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

也可以指定枚举的值为16进制：

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

### 枚举值类型

枚举值默认是int类型，也可以使用as来明确指定枚举值的类型，枚举值的类型只能是内置的整数类型。

```v
enum Color { //默认是int类型
	red
	green = 2147483646
	blue
}

enum ColorI8 as i8 {
	red
	green = 126
	blue
}

enum ColorI16 as i16 {
	red
	green = 32766
	blue
}

enum ColorI32 {
	red
	green = 2147483646
	blue
}

enum ColorI64 as i64 {
	red
	green = 9223372036854775806
	blue
}

enum ColorU8 as u8 {
	green = 127
}

enum ColorU16 as u16 {
	green = 32768
}

enum ColorU32 as u32 {
	green = 2147483647
}

enum ColorU64 as u64 {
	green = 9223372036854775807
}

```

### 枚举方法

枚举也可以像结构体那样添加方法：

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

枚举类型和整数类型可以相互转换，不过有一点需要特别注意，将整数转换为枚举类型，属于跨类型的强制转换，必须在不安全代码块中执行，不然编译器会报错。

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
	e := unsafe { Color(i) } // 将整数转换为枚举类型，属于跨类型的强制转换，必须在不安全代码块中
	println(e == .blue) // 输出true
	ii := int(e) // 枚举类型转换为int
	println(ii) // 输出3
	iii := e // iii是枚举类型
	println(typeof(iii).name)
	println(iii)
}
```

### 枚举类型数组

可以定义枚举类型的数组，数组的值是某个枚举项：

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

### 遍历枚举值

可以使用$for in语句来遍历所有的枚举值：

```v
enum Color {
	red = 1
	green
	blue
}

fn main() {
	$for e in Color.values { // $for in语句可以用来遍历所有的枚举值,内置的values,表示所有的枚举值,
		if e.name == 'red' { //内置的name,表示当前枚举名
			println(e.value == Color.red) //内置的value,表示当前枚举值
		} else if e.name == 'green' {
			println(e.value == Color.green)
		} else if e.name == 'blue' {
			println(e.value == Color.blue)
		}
	}
}
```

###  枚举注解

#### @[flag]注解

如果给枚举加了@[flag]注解，就表示这个枚举是位字段类型的枚举，枚举项的值不能自定义设置，由编译器自动设置，按顺序是2的n次方，n从0开始：1，2，4，8，16 ...。

如果枚举是位字段类型的枚举，可以使用以下内置方法：

```v
has() //判断枚举项是否包含参数中的某一项值
all() //判断枚举项是否等于参数的所有值
set() //在枚举项已有值的基础上，追加参数的值,为什么不用append更准确？
toggle() //在枚举项已有值的基础上，去掉参数的值，为什么不用delete更准确？
clear()  //跟toggle一样
is_empty() //判断枚举值是否为空，如果枚举的值都被清空了，返回true
```

```v
@[flag]
enum BitField { //编译器自动设置枚举项的值
	read
	write
	delete
	other
}

fn main() {
	println(1 == int(BitField.read)) // true
	println(2 == int(BitField.write)) // true
	println(4 == int(BitField.delete)) // true
	println(8 == int(BitField.other)) // true
	mut bf := BitField.read
	println(bf.has(.read | .other)) // true,测试bf枚举值是否包含参数的某一个值
	println(bf.all(.read | .other)) // false,测试bf枚举值是否等于参数的所有值
	bf.set(.write | .other) //在枚举项已有值的基础上，追加.write和.other
	println(bf) //.read | .write | .other
	println(bf.has(.read | .write | .other)) // true
	println(bf.all(.read | .write | .other)) // true
	bf.toggle(.other) //在枚举项已有值的基础上，去掉.other
	println(bf) //.read | .write
	println(bf == BitField.read | .write) // false
	println(bf.all(.read | .write)) // true
	println(bf.has(.other)) // false
	println(bf.has(.read)) // true
	bf.clear(.read) // 跟toggle一样
	bf.clear(.write) // 跟toggle一样
	println(bf.is_empty()) // 判断枚举项是否为空,true
	println(bf)
}

```

#### 自定义注解

可以像结构体和函数那样，给枚举和枚举值添加自定义注解，然后自己解析使用。

关于注解的进一步使用，可以参考[注解章节](attribute.md)。

枚举注解：

```v
@[attr1]
@[attr2]
pub enum Color {
	black 【attr3】
	white
	blue
}
```

枚举值注解：

```v
enum Color {
	red = 1 + 1  [json: 'Red'] //枚举值注解
	blue = 10 / 2  [json: 'Blue']
}

fn main() {
	$for e in Color.values {
		if e.name == 'red' {
			println(e.value == Color.red)
		} else if e.name == 'blue' {
			println(e.value == Color.blue)
		}
	}
}
```

