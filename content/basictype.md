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
| int   | 4字节 | -2,147,483,648 到 2,147,483,647                              | int总是32位整数，即i32 |
| i64   | 8字节 | -9,223,372,036,854,775,808 到 9,223,372,036,854,775,807      | 有符号64位整数         |
| isize | arch  | 等价于C的ptrdiff_t类型，长度取决于运行的计算机架构，64 位是i64，32 位是i32 | 有符号整数             |
|       |       |                                                              |                        |
| u8    | 1字节 | 0 到 255                                                     | 无符号8位整数          |
| u16   | 2字节 | 0 到 65,535                                                  | 无符号16位整数         |
| u32   | 4字节 | 0 到 4,294,967,295                                           | 无符号32位整数         |
| u64   | 8字节 | 0 到 18,446,744,073,709,551,615                              | 无符号64位整数         |
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
	x := 3 //默认自动推断为int
	y := 1.2 //默认自动推断为f64
	println(typeof(x).name) //输出int
	println(typeof(y).name) //输出f64

	xx := i64(3) // xx是i64类型，而不是默认推断的int
	yy := f32(3.0) // yy是f32类型，而不是默认推断的f64
	println(typeof(xx).name) //输出i64
	println(typeof(yy).name) //输出f32
```

### 字节类型

字节类型u8，就是单字节byte。

早期的V版本，确实有byte这个类型，是u8的类型别名，后来被声明取消了，因为作者希望坚持one way的原则，目前byte还可以照常使用，但是在不远的将来，会被取消。

```v
module main

fn main() {
	b := u8(98)
	println(b.str()) // 98
	println(b.ascii_str()) // b
}
```

### 字符串类型

string字符串类型，默认不可变，UTF-8编码。

单引号和双引号都可以，习惯上都还是使用单引号，V编译器中的代码都是统一使用的单引号。

按作者的说法是可以少按一个shift键，更自然简单一些，不过打破了之前C的使用习惯，让人一开始有些不习惯。

当使用双引号的时候，编译器会自动把双引号自动替换为单引号。

```v
s1:='abc'
s2:="abc"
```

字符串长度：

```v
s:='abc'
println(s.len) //输出3
```

字符串连接：

```v
s1:='abc'
s2:='def'
s3:=s1+s2
```

字符串追加：

```v
mut s:='hello ' //必须是可变才可以追加
s+='world'
println(s) //输出hello world
```

字符串插值：

字符串插值只会有一种格式，即\${变量名}

```v
name:='Bob'
println('hello ${name}')
println("hello ${name}")
```

判断字符串是否包含子字符串：

```v
fn main() {
    s:='abcd'
    println(s.contains('c')) //true
    println(s.contains('bc')) //true
    println(s.contains('bb')) //false
}
```

遍历字符串：

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

**字符串切片/区间**

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

字符串从定义的V源代码代码看，也是通过结构体struct来实现的。

vlib/builtin/string.v：

```v
pub struct string {
pub: 					 //pub表示这两个字符串的属性都是：公共且只读的
	str &u8 //一个u8类型的指针，指向字符串的首字节地址
	len int  //字符串的长度
}
```

**字符串单引号和双引号混合使用**

主要的逻辑是：

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

**原始字符串**

在单引号或双引号前加上小写r，就表示是raw字符串，在原始字符串中不支持插值和转译。

```v
module main

fn main() {
	name := 'tom'
	str := 'name is: {name} \n' //字符串插值和转译
	raw_str := r'name is: {name} \n' //原始字符串,在单引号或双引号之前加上r前缀
	raw_str2 := r"name is: {name} \n" //原始字符串,在单引号或双引号之前加上r前缀
	println(str)
	println(raw_str)
	println(raw_str2)
}
```

输出：

```
name is: tom 

name is: {name} \n
name is: {name} \n
```

### rune类型

rune是u32的类型别名，用4个字节来表示一个unicode字符/码点，跟string不是同一个类型。

rune使用反引号来表示。

```v
module main

fn main() {
	s1 := 'a' //单引号，string类型
	s2 := "a" //双引号，string类型
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

voidptr，通用类型指针，用来存储变量的内存地址，可以保存任何类型的地址。

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

nil，表示空值或者空指针，等价于voidptr(0)。

正常情况下，由V创建的变量，因为声明和初始化一定是同时进行的，所以变量一定会有初始值。V指针一定不会有空值nil，即voidptr(0)值。

但是通过调用C代码返回的指针，有可能是空指针，所以在使用前可以用isnil(ptr)内置函数判断一下，判断由C代码生成的指针是否是空指针。

```v
fn main() {
	a := 1
	println(isnil(&a)) // 返回false，变量只能通过:=来初始化，一定会有初始值
	// 但是通过调用C代码返回的指针，有可能是空指针，所以在使用前可以用isnil函数判断一下
	f := C.popen('ls'.str, 'r'.str)
	if isnil(&f) {
		println('f is nil')
	} else {
		println('f is not nil')
	}
}
```

在V语言中，指针只是表示引用，不能进行指针运算。

只有在unsafe代码块中，V编译器不进行任何检查，允许指针像C那样可以进行指针运算，指针偏移，多级指针。

详细内容可以参考：[不安全代码](./unsafe.md)。

除了voidptr通用类型指针，V还有2种指针类型，一般情况比较少用：字节类型指针byteptr，字符类型指针charptr。

```v
module main

fn main() {
	u := u8(10)
	b := &u //&u类型
	c := `a`
	b_ptr := byteptr(b) // byteptr类型,等价于&u
	c_ptr := charptr(&c) // charptr类型,一般用于跟C集成使用,等价于*char

	println(typeof(b).name) 
	println(typeof(b_ptr).name) 
	println(typeof(c_ptr).name) 
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

### 获取变量和类型占用内存大小

使用内置函数sizeof来获取变量和类型占用内存大小：

- 普通函数，获取变量的占用内存大小
  - `sizeof(var)`
- 泛型函数，获取类型的占用内存大小
  - `sizeof[T]()`

```v
module main

fn main() {
	x := 1
	u := u8(1)
	b := true
	//普通函数
	println(sizeof(x)) // 4
	println(sizeof(u)) // 1
	println(sizeof(b)) // 1
	//泛型函数
	println(get_size[int]()) // 4
	println(get_size[u8]()) // 1
	println(get_size[bool]()) // 1
}

fn get_size[T]() u32 {
	return sizeof(T) //泛型函数
}

```

### 获取变量和类型的类型信息

使用内置函数typeof来返回变量的类型和类型的id

- 普通函数，获取变量的类型和类型的id
  - `typeof(var).name``
  - `typeof(var).idx`
- 泛型函数，获取类型的类型和类型的id
  - `typeof[T]().name`
  - `typeof[T]().idx`

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

	//使用typeof().idx获取变量类型的id
	//使用typeof().name获取变量的类型
	println(typeof(a).idx) // 7
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

获取类型的名字和id：

```v
module main

struct MyStruct {}

struct MyGenericStruct2[T, U] {}

type Abc = int | string
type AnAlias = int

enum EFoo {
	a
	b
	c
}

fn main() {
	println(typeof[int]().idx) //返回类型的id,int类型在编译器内部的id是7
	println(typeof[int]().name) //返回类型的名字: int
	println(typeof[?string]().name) //返回类型的名字: ?string
	println(typeof[[]string]().name) //返回数组类型的名字: []string
	println(typeof[map[string]int]().name) //返回字典类型的名字: map[string]int
	println(typeof[fn (s string, x u32) (int, f32)]().name) //返回函数类型的名字: fn (string, u32) (int, f32)
	println(typeof[MyGenericStruct]().name) //返回结构体类型的名字: MyStruct
	println(typeof[MyGenericStruct2[string, int]]().name) //返回泛型结构体类型的名字: MyGenericStruct2[string, int]
	println(typeof[Abc]().name) //返回联合体类型的名字: Abc
	println(typeof[EFoo]().name) //返回枚举类型的名字: EFoo
	println(typeof[AnAlias]().name) //返回类型别名名字: AnAlias
}
```

### 判断变量和类型是否为引用类型

- 普通函数，判断变量是否为引用类型

  `isreftype(var)`	

- 泛型函数，判断类型是否为引用类型

  `isreftype[T]()`

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
	println((isreftype(s))) // string是引用类型,因为字符串是使用结构体来实现的

	mut m := map[string]string{} // map是引用类型,因为字典是使用结构体来实现的
	m['name'] = 'tom'
	println(isreftype(m))

	a := [1, 2, 3]
	println(isreftype(a)) // array是引用类型,因为数组是使用结构体来实现的

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

判断类型是否为引用类型：

```v
module main

fn main() {
	println(isreftype(int))
	println(isreftype[string]())
	println(get_is_ref_type[array]())

}

fn get_is_ref_type[T]() bool {
	return isreftype[T]()
}
```

