## 基本类型

### 布尔类型

bool

bool类型的值只能是:true或false

------

```c
x:=true
y:=false
println(x) //true
println(y) //false
println(sizeof(bool)) //4
```

bool从定义的C代码看，是C的一个int类型别名,true是常量1，false是常量0

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
| i8   | 1字节  | -128 到 127                               |                       |
| i16  | 2字节  | -32,768 到 32,767                         |                       |
| int  | 4字节  | -2,147,483,648 到 2,147,483,647           | 即i32,int总是32位整数 |
| i64  | 8字节  | -9223372036854775808到9223372036854775807 |                       |
| i128 | 16字节 | 未实现                                    | 未实现                |
| byte | 1字节  | 0 到 255                                  | 无符号,即u8           |
| u16  | 2字节  | 0 到 65,535                               | 无符号                |
| u32  | 4字节  | 0 到 4,294,967,295                        | 无符号                |
| u64  | 8字节  | 0 到 18446744073709551615                 | 无符号                |
| u128 | 16字节 | 未实现                                    | 未实现                |
| rune | 4字节  | 0 到 4,294,967,295                        | 表示一个unicode码点   |
| f32  | 4字节  |                                           | 32位小数              |
| f64  | 8字节  |                                           | 64位小数              |

### 字符串类型

string

字符串是一个只读字节数组,默认不可变,UTF-8编码

单引号和双引号都可以

习惯上都还是使用单引号,V编译器中的代码都是统一使用的单引号,按作者的说法是可以少按一个shift键,更自然简单一些

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
mut s:='hello ' //必须是mut才可以
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



------

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

常用字符串内置函数,可以参考后面的[标准库](stdlibrary.md)章节,也可以直接参考vlib/builtin/string.v源代码

------

### 指针类型

byteptr 

字节指针：指针指向的地址是byte类型

voidptr

通用指针：可以指向任何类型

### sizeof() 内置函数

可以通过内置的sizeof(T)来返回各种类型占用的内存大小

```c
println(sizeof(int)) //4
println(sizeof(byte)) //1
println(sizeof(bool)) //4
```

