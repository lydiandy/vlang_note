## 基本类型



### 布尔类型

bool

true,false

### 字符串类型

string

字符串是一个只读字节数组,默认不可变,UTF-8编码

单引号和双引号都可以

习惯上都还是使用单引号,V编译器中的代码都是统一使用的单引号,按作者的说法是可以少按一个shift键,更自然简单一些

当使用双引号的时候,编译器会自动把双引号自动替换为单引号

```
s1:='abc'
s2:="abc"
```

字符串长度: 

```
s:='abc'
println(s.len) //输出3
```

字符串连接: 

```
s1:='abc'
s2:='def'
s3:=s1+s2
```

字符串追加:

```
mut s:='hello ' //必须是mut才可以
s+='world'
println(s) //输出hello world
```

字符串插值:

```
name:='Bob'
println('hello $name') //方式1
println('hello ${name}') //方式2,效果一样,更常用于复杂的表达式,把表达式放在{}里面
```

遍历字符串:

```
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



### 单字符类型

用反引号来表示单字符类型,跟字符串不是同一个类型

```
s1:='a' //字符串类型
s2:=`a` //字符类型,单字符
//s3:=`aa` //编译不通过,报错,只能是单字符
println(s1) //输出a
println(s2) //输出97
println(s2.str()) //通过str()函数转换后,输出a
```



常用字符串内置函数,可以参考后面的[标准库](stdlibrary.md)章节,也可以直接参考vlib/builtin/string.v源代码

### 整数类型

有符号:

i8    i16  int  i64      i128 (soon)

不像C和go,int类型总是32位整数

无符号:

byte  u16  u32  u64      u128 (soon)

unicode:

rune 表示一个unicode码点

### 小数类型

f32 

f64

### 指针类型

byteptr

voidptr