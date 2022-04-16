## 基本类型

V语言是一门静态类型，编译型语言，以下是内置的基本类型：

### 布尔类型

bool，布尔类型，值只能是true或false。

```v
module main

fn main() {
	x := true
	y := false
	println(x) // true
	println(y) // false
	println(sizeof(bool)) // 1个字节
}
```

布尔类型从定义的C代码看是u8的类型别名。

true是常量1，false是常量0。

使用sizeof(bool)可以看到bool类型的默认长度是1个字节。

```c
typedef uint8_t u8;
...
#ifndef bool
		#ifdef CUSTOM_DEFINE_4bytebool
			typedef int bool;
		#else
			typedef u8 bool;
		#endif
		#define true 1
		#define false 0
	#endif
```

### 数值类型

数值类型分为整数和小数。

#### 整数

| 类型  | 长度  | 取值范围                                                     | 说明                   |
| :---- | ----- | :----------------------------------------------------------- | ---------------------- |
| i8    | 1字节 | -128 到 127                                                  | 有符号8位整数          |
| i16   | 2字节 | -32,768 到 32,767                                            | 有符号16位整数         |
| int   | 4字节 | -2，147,483,648 到 2,147,483,647                             | int总是32位整数，即i32 |
| i64   | 8字节 | -9223372036854775808到9223372036854775807                    | 有符号64位整数         |
| isize | arch  | 等价于C的ptrdiff_t类型，长度取决于运行的计算机架构，64 位是i64，32 位是i32 | 有符号整数             |
|       |       |                                                              |                        |
| u8    | 1字节 | 0 到 255，byte是u8的别名                                     | 无符号8位整数          |
| u16   | 2字节 | 0 到 65,535                                                  | 无符号16位整数         |
| u32   | 4字节 | 0 到 4,294,967,295                                           | 无符号32位整数         |
| u64   | 8字节 | 0 到 18446744073709551615                                    | 无符号64位整数         |
| usize | arch  | 等价于C的size_t类型，长度取决于运行的计算机架构，64位是u64，32 位是u32 | 无符号整数             |

#### 小数

| 类型 | 长度  | 取值范围          | 说明     |
| ---- | ----- | ----------------- | -------- |
| f32  | 4字节 | 等价于C中的float  | 32位小数 |
| f64  | 8字节 | 等价于C中的double | 64位小数 |

数字可以通过前缀标识对应的进制，也可以增加下划线，进行视觉上的分隔，不影响本来的值。

```v
mut c := 0xa_0 //十六进制数字a0
println(c) //输出160
c = 0b10_01 //二进制数字1001
println(c) //输出9
c = 1_000_000 //十进制数字1000000
println(c) //输出1000000
```

小数只能使用类似1.0的风格，不允许使用1.的风格，小数点后面的0不允许省略。

```v
	f1 := 1.0
	println(f1)
	//f2 := 1. //不允许使用1.的风格
	//println(f2)
```

如果不希望由编译器自动类型推断，可以通过T(value)的格式，显式声明变量类型，T是类型，value是变量值。

```v
x:=3	//默认自动推断为int
y:=1.2	//默认自动推断为f64
println(typeof(x).name) //输出int
println(typeof(y).name)	//输出f64

x:=i64(3) //x是i64类型，而不是默认推断的int
y:=f32(3.0) //y是f32类型，而不是默认推断的f64
```

字节类型

```v
module main

fn main() {
	b := u8(98)
	println(b.str()) // 98
	println(b.ascii_str()) // b
}
```

### 字符串类型

string字符串类型，字符串是一个只读字节数组，默认不可变，UTF-8编码。

单引号和双引号都可以，习惯上都还是使用单引号，V编译器中的代码都是统一使用的单引号。

按作者的说法是可以少按一个shift键，更自然简单一些，不过打破了之前C的使用习惯，让人一开始有些不习惯。

当使用双引号的时候，编译器会自动把双引号自动替换为单引号。

```v
s1:='abc'
s2:="abc"
```

字符串长度: 

```v
s:='abc'
println(s.len) //输出3
```

字符串连接: 

```v
s1:='abc'
s2:='def'
s3:=s1+s2
```

字符串追加:

```v
mut s:='hello ' //必须是可变才可以追加
s+='world'
println(s) //输出hello world
```

字符串插值:

```v
name:='Bob'
println('hello $name') //方式1
println('hello ${name}') //方式2，效果一样，更常用于复杂的表达式，把表达式放在{}里面
```

判断字符串是否包含子字符串:

```v
fn main() {
    s:='abcd'
    println(s.contains('c')) //true
    println(s.contains('bc')) //true
    println(s.contains('bb')) //false
}
```

遍历字符串:

```v
str := 'abcdef'
//遍历value
for s in str {
	println(s.str())
}
//遍历index和value
for i， s in str {
	println('index:$i，value:$s。str()')
}
```

**字符串切片/区间：**

采用的是左闭右开。

```v
module main

fn main() {
	s := 'hello_world'
	println(s[..3]) //输出hel
	println(s[2..]) //输出llo_world
	println(s[2..5]) //输出llo
	//区间支持负数的索引值，不过要在字符串名后特别加一个#,否则会报错
	println(s#[-5..-2]) //输出wor
	//字符串区间越界的or代码块处理
	println(s[..s.len])
	println(s[..s.len + 1] or { 'oh,no' })
	println(range_check(s) or { 'out of range' })
}

fn range_check(s string) ?string {
	return s[..20] ?
}
```

字符串从定义的v代码看，也是一个struct。

```v
pub struct string {
pub: 					 //pub表示这两个字符串的属性都是：公共且只读的
	str &u8 //一个u8类型的指针，指向字符串的首字节地址
	len int  //字符串的长度
}
```

**字符串单引号和双引号混合使用**

主要的逻辑是:

- 以单引号为主要使用的符号，双引号也可以使用。
- 如果有字符串嵌套使用，内外必须不能同时是单引号或者同时是双引号，不然无法正确配对。
- 如果有加上反斜杠进行转义，则都可以。

```v
module main

fn main() {
	s0:='name=tom' 		//ok
	s1:="name=tom" 		//ok
	s2:="name='tom'" 	//ok
	s3:='name="tom"' 	//ok
	//s4:='name='tom''	//error
	//s5:="name="tom""	//error
	s6:='name=\'tom\''  //ok
	s7:="name=\'tom\'"	//ok
	s8:="name=\"tom\""	//ok
	s9:='name=\"tom\"'	//ok
	println(s0)
	println(s1)
	println(s2)
	println(s3)
	// println(s4)
	// println(s5)
	println(s6)
	println(s7)
	println(s8)
	println(s9)
}
```

### rune类型

rune是u32的类型别名，用4个字节来表示一个unicode字符/码点，跟string不是同一个类型。

rune使用反引号来表示。

```v
module main

fn main() {
	s1 := 'a' //单引号，string类型
	s2 := 'a' //双引号，string类型
	s3 := `a` //反引号，rune类型
	println(typeof(s1).name)
	println(typeof(s2).name)
	println(typeof(s3).name)
	println(int(s3)) // 97
	//
	// c2 := `aa` //编译不通过，报错，只能是单字符
	c3 := `中`
	println(typeof(c3).name) // rune类型
	println(sizeof(c3)) // 4个字节，unicode4.0
	println(int(c3)) // 20013
	println(c3)
}

```

常用字符串内置函数，可以参考后面的[标准库](./std_builtin.md)章节，也可以直接参考vlib/builtin/string.v源代码。

### 指针类型

voidptr指针类型，用来存储变量的内存地址。

指针本身占用的内存大小就是C语言里面的size_t类型，通常在32位系统上的长度是32位，64位系统上是64位。

```v
module main

fn main() {
//测试机器是64位操作系统
println(sizeof(voidptr)) //输出8个字节
}
```

变量前加&表示取内存地址，返回指针类型。

指针类型前加*表示取内存地址对应变量的值。

```v
fn main() {
	a := 'abc'
	println(&a) // 取变量地址，返回地址
	b := &a
	println(*b) // 取指针对应的值，返回abc
}
```

正常情况下，由V创建的变量，因为声明和初始化一定是同时进行的，

所以变量一定会有初始值，V指针一定不会有空值，即0值。

内置函数中的isnil(ptr)，是用来判断由C代码生成的指针，是否是空指针。

```v
fn main() {
	a := 1
	println(isnil(&a)) // 返回false，变量只能通过:=来初始化，一定会有初始值
	// 但是通过调用C代码返回的指针，有可能是空指针，所以在使用前可以用isnil函数来判断一下
	f := C.popen('ls'， 'r')
	if isnil(&f) {
		// ...
		println('f is nil')
	} else {
		println('f is not nil')
	}
}
```

在V语言中，指针只是表示引用，不能进行指针运算。

只有在unsafe代码块中，V编译器不进行任何检查，允许指针像C那样可以进行指针运算，指针偏移，多级指针。

详细内容可以参考:[不安全代码](。/unsafe。md)。

### 类型占用内存大小sizeof()

使用内置函数sizeof(T)来返回类型占用内存大小。

```v
println(sizeof(int)) //4
println(sizeof(u8)) //1
println(sizeof(bool)) //1
```

### 变量类型typeof().name

使用内置函数typeof(var).name来返回变量的类型。

```v
module main

struct Point {
	x int
}

type MyFn = fn (int) int

type MyFn2 = fn ()

fn myfn(i int) int {
	return i
}

fn myfn2() {
}

fn main() {
	a := 123
	s := 'abc'
	aint := []int{}
	astring := []string{}
	astruct_static := [2]Point{}
	astruct_dynamic := [Point{}, Point{}]

	//使用typeof().name获取变量的类型
	println(typeof(a).name) // int
	println(typeof(s).name) // string
	println(typeof(aint).name) // array_int
	println(typeof(astring).name) // array_string
	println(typeof(astruct_static).name) // [2]Point
	println(typeof(astruct_dynamic).name) // array_Point

	//函数类型
	println(typeof(myfn).name) // fn (int) int
	println(typeof(myfn2).name) // fn ()
}
```

### 类型推断及类型转换

```v
module main

fn main() {
	i := 8 // 默认类型推断为int
	b := u8(8) // 明确指定类型为u8
	ii := int(b) // 强制转换为int
	f := 3.2 // 默认推断类型为f64
	ff := f32(3.2) // 明确指定类型为f32
	f3 := f64(f) // 强制转换为f64
	s := 'abc' // 默认推断为string
	c := `c` // 默认推断为u8，也就是单字符类型

	// 布尔类型可以转换为u8/int或其他整数类型
	yes := true
	no := false
  yes_u8 := u8(yes) // 输出1
	no_u8 := u8(no) // 输出0
	yes_int := int(yes)  // 输出1
	no_int := int(no) // 输出0
	// 将字节数组转成字符串
	mut u8_arr := []u8{} // 字节数组
	u8_arr << `a`
	u8_arr << `b`
	println(u8_arr) // 输出[a,b]
	str := u8_arr.str() // 将字节数组转成字符串
	println(str) // 输出[a,b]
}
```

### 判断变量是否为引用类型

```v
module main

struct Point {
	x int
	y int
}

fn main() {
	i := 8
	println(isreftype(i)) //基本类型除了string，都不是引用类型

	s := 'abc'
	println((isreftype(s))) // string是引用类型

	mut m := map[string]string{} // map是引用类型
	m['name'] = 'tom'
	println(isreftype(m))

	a := [1,2,3]
	println(isreftype(a)) // array是引用类型

	p := Point{
		x: 1
		y: 2
	}
	pp := &Point{
		x: 1
		y: 2
	}
	println(isreftype(p)) // p不是引用类型
	println(isreftype(pp)) // pp是引用类型
}
```

