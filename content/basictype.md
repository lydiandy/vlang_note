## 基本类型

V语言是一门静态类型,编译型语言,以下是内置的基本类型:

### 布尔类型

bool

bool类型的值只能是true或false

------

```c
x:=true
y:=false
println(x) //true
println(y) //false
println(sizeof(bool)) //4
```

bool类型从定义的C代码看，是C的一个int类型别名

true是常量1，false是常量0

使用sizeof(bool)，可以看到bool类型的长度是4个字节

```c
#ifndef bool
	typedef int bool;
	#define true 1
	#define false 0
#endif
```



------

### 数值类型

整数和小数

| 类型 | 长度   | 取值范围                                  | 说明                  |
| ---- | ------ | ----------------------------------------- | --------------------- |
| i8   | 1字节  | -128 到 127                               | 有符号8位整数         |
| i16  | 2字节  | -32,768 到 32,767                         | 有符号16位整数        |
| int  | 4字节  | -2,147,483,648 到 2,147,483,647           | int总是32位整数,即i32 |
| i64  | 8字节  | -9223372036854775808到9223372036854775807 | 有符号64位整数        |
| i128 | 16字节 | 未实现                                    | 有符号128位整数       |
| byte | 1字节  | 0 到 255                                  | 无符号8位整数,即u8    |
| u16  | 2字节  | 0 到 65,535                               | 无符号16位整数        |
| u32  | 4字节  | 0 到 4,294,967,295                        | 无符号32位整数        |
| u64  | 8字节  | 0 到 18446744073709551615                 | 无符号64位整数        |
| u128 | 16字节 | 未实现                                    | 无符号128位整数       |
| rune | 4字节  | 0 到 4,294,967,295                        | 表示一个unicode码点   |
| f32  | 4字节  |                                           | 32位小数              |
| f64  | 8字节  |                                           | 64位小数              |

数字可以通过前缀标识对应的进制

也可以增加下划线,进行视觉上的分隔,不影响本来的值

```c
	mut c := 0xa_0 //十六进制数字a0
	println(c) //输出160
	c = 0b10_01 //二进制数字1001
	println(c) //输出9
	c = 1_000_000 //十进制数字1000000
	println(c) //输出1000000
```

显示指定类型

如果不希望由编译器自动类型推断,可以通过T(value)的格式明确变量类型,T是类型,value是变量值

```c
x:=i64(3) //x是i64类型，而不是默认推断的int
y:=f32(3.0) //y是f32类型，而不是默认推断的f64
```



### 字符串类型

string

字符串是一个只读字节数组,默认不可变,UTF-8编码

单引号和双引号都可以

习惯上都还是使用单引号,V编译器中的代码都是统一使用的单引号,按作者的说法是可以少按一个shift键,更自然简单一些,不过打破了之前C的使用习惯,让人一开始有些不习惯

当使用双引号的时候,编译器会自动把双引号自动替换为单引号

```c
s1:='abc'
s2:="abc"
```

字符串长度: 

```c
s:='abc'
println(s.len) //输出3
```

字符串连接: 

```c
s1:='abc'
s2:='def'
s3:=s1+s2
```

字符串追加:

```c
mut s:='hello ' //必须是可变才可以追加
s+='world'
println(s) //输出hello world
```

字符串插值:

```c
name:='Bob'
println('hello $name') //方式1
println('hello ${name}') //方式2,效果一样,更常用于复杂的表达式,把表达式放在{}里面
```

遍历字符串:

```c
str:='abcdef'
//遍历value
for s in str {
    println(s.str())
}
//遍历index和value
for i,s in str {
    println('index:$i,value:${s.str()}')
}
```

**字符串切片/区间：**

采用的是左闭右开

```c
s:='hello_world'
println(s[..3]) //输出hel
println(s[2..]) //输出llo_world
println(s[2..5]) //输出llo
```



字符串从定义的v代码看，也是一个struct

```c
struct string {
pub: 					 //pub表示这两个字符串的属性都是：公共且只读的
	str byteptr //一个byte类型的指针,指向字符串的首字节地址
	len int  //字符串的长度
}
```

------

### 单字符类型

用反引号来表示单字符类型,跟字符串不是同一个类型,单字符类型对应的是byte类型

```c
s1:='a' //单引号，字符串类型
s2:="aa" //双引号，也是字符串类型

c1:=`a` //反引号，字符类型,单字符
// c2:=`aa` //编译不通过,报错,只能是单字符
// c2:=`中` //编译不通过,报错,只能是单字符
println(s1) //输出a
println(s2) //输出aa
println(c1) //输出97
println(c1.str()) //通过str()函数转换为字符串后,输出a
```

常用字符串内置函数,可以参考后面的[标准库](./std_builtin.md)章节,也可以直接参考vlib/builtin/string.v源代码

------

### 指针类型

byteptr: 字节类型指针

voidptr: 通用指针,可以指向任何类型

intptr:  整型类型指针

charptr: 字符类型指针

V代码中使用的最多的是byteptr和voidptr

V代码中比较少用intptr,charptr,只有在跟C代码集成时或底层代码用得多,C的很多字符串都是charptr指针

以下是4种指针类型,生成C代码对应的类型定义:

```c
typedef unsigned char* byteptr;
typedef int* intptr;
typedef void* voidptr;
typedef char* charptr;
```

指针本身占用的内存大小:就是C语言里面的size_t类型,通常在32位系统上的长度是32位,64位系统上是64位

```c
	//测试机器是64位操作系统
	println(sizeof(voidptr)) //输出8个字节
	println(sizeof(byteptr)) //输出8个字节
	println(sizeof(charptr)) //输出8个字节
	println(sizeof(intptr))  //输出8个字节
```

变量前加&表示取地址,返回指针类型

指针类型前加*表示取地址对应的值

```c
fn main() {
	a := 'abc'
	println(&a) // 取变量地址,返回地址
	b := &a
	println(*b) // 取指针对应的值,返回abc
}
```

正常情况下,由V创建的变量,因为声明和初始化一定是同时进行的,所以变量一定会有初始值,所以V指针一定不会有空值,即0值

内置函数中的isnil(ptr),是用来判断由C代码生成的指针,是否是空指针

```c
fn main() {
	a := 1
	println(isnil(&a)) //返回false,变量只能通过:=来初始化,一定会有初始值
        
	f := C.popen(cmd.str, 'r') //但是通过调用C代码返回的指针,有可能是空指针,所以在使用前可以用isnil函数来判断一下
	if isnil(f) {
		println(sframe)
		continue
	}
}
```

在V语言中,指针只是表示引用,跟go指针类型基本一致,不能进行指针运算

只有在unsafe代码块中,V编译器不进行任何检查,允许指针像C那样可以进行指针运算,指针偏移,多级指针

详细内容可以参考:[不安全代码](./unsafe.md)

### 类型占用内存大小sizeof()

使用内置函数sizeof(T)来返回类型占用内存大小

```c
println(sizeof(int)) //4
println(sizeof(byte)) //1
println(sizeof(bool)) //4
```

### 变量类型typeof()

使用内置函数typeof(var)来返回变量的类型

```c
module main

struct Point {
	x int
}
type MySumType = int | f32 | Point

type MyFn fn(int) int
type MyFn2 fn()

fn myfn(i int) int {
	return i
}
fn myfn2() {
	
}

fn main() {
	a := 123
	s:='abc'	
	aint := []int
	astring := []string
	astruct_static := [2]Point
	astruct_dynamic := [Point{}, Point{}]
	println(typeof(a)) //int
	println(typeof(s))	//string
	println(typeof(aint)) //array_int
	println(typeof(astring)) //array_string
	println(typeof(astruct_static)) //[2]Point
	println(typeof(astruct_dynamic)) //array_Point

	//联合类型也可以判断具体的类型
	sa := MySumType(32)
	sb := MySumType(123.0)
	sc := MySumType(Point{x:43})
	println(typeof(sa))   //int
	println(typeof(sb)) //f32
	println(typeof(sc)) //Point

	//函数类型
	println(typeof(myfn)) //fn (int) int
	println(typeof(myfn2)) //fn ()
}
```



### 类型推断及类型转换

```c
module main

fn main(){
	i:=8 //默认类型推断为int
	b:=byte(8) //明确指定类型为byte
	ii:=int(b) //强制转换为int

	f:=3.2 //默认推断类型为f64
	ff:=f32(3.2) //明确指定类型为f32
	f3:=f64(f) //强制转换为f64

	s:='abc' //默认推断为string	
	c:=`c` //默认推断为byte,也就是单字符类型
	ss:=string(&c,1) //强制转换为string,因为string(byteptr,int)就可以构造出一个字符串,其中byteptr是首字节指针,int是字符串长度
    
  //将字节数组转成字符串
	mut byte_arr:=[]byte //字节数组
	byte_arr<<`a`
	byte_arr<<`b`
	println(byte_arr) //输出[a,b]
	str:=string(byte_arr,byte_arr.len) //将字节数组转成字符串
	println(str) //输出ab    
}
```

