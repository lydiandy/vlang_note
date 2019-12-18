## builtin内置模块

在builtin内置模块中定义的函数,结构体等,编译器在编译代码时会默认加载builtin模块,

可以在任何模块中直接使用,并且不用带buitlit前缀,就是内置

#### 内置函数

```
vlib/builtin/builtin.v
```

- print(string)  //打印字符串,不换行

- println(string) //打印字符串,换行

- eprintln(string) //打印错误,控制台是红色字

  ​		

  ------

  

- exit(int) 退出程序

- isnil(ptr voidptr) //判断C指针是否是空指针

  ​		

  ------

  

- panic(s string) //恐慌,报错

- on_panic(f fn(int) int) //尚未实现,应该是恐慌后的回调处理

  ​		

  ------

  

- malloc(int) byteptr //分配内存

  对C标准库的malloc的简单封装,成为内置函数,用途一样

- calloc(int) byteptr //分配内存

  对C标准库的calloc的简单封装,不一样的是C的calloc有2个参数,V简化为1个size参数,成为内置函数,用途一样

- free(ptr voidptr) //释放内存

  对C标准库free的简单封装,成为内置函数,用途一样

  ​		
  
  以下3个常用的C分配内存函数没有定义成为内置函数,还需要通过C.xxx来使用:
  
- C.realloc(byteptr,int) //重新调整内存大小

- C.memcpy(byteptr,byteptr,int) //内存拷贝

- C.memmove(byteptr,byteptr,int) //内存移动

  基本上V代码中,都是使用这几个函数来实现内存分配和内存控制

  ------

  

#### 字符串

```
vlib/builtin/string.v
```

ends_with(string) bool

判断字符串是否以给定的字符串结尾

------

starts_with(string) bool

判断字符串是否以给定的字符串开始

------

find_between('[',']') string

返回字符串中包在这两个字符中间的子字符串

------

index(string) int

返回子字符串在字符串中的位置,如果没有包含,则返回-1

------

last_index(string) int 

返回子字符串在字符串中,最后出现的位置

------

index_any(string) int 

返回子字符串中的任意单个字符,在字符串中出现的位置,如果没有包含,则返回-1

------

index_after(str,n) int

从字符串第n个开始查找起,返回子字符串在整个字符串中的位置,如果没有包含,则返回-1	

------

at(int) byte

返回字符串第几个位置的字符串

------

split(string) [ ]string

按照给定的分割符,把字符串分割,形成数组

------

trim_space() string

去掉字符串左右两边的空格,中间的空格不去掉

------

trim(string) string

去掉字符串中包含子字符串中的字符

------

trim_left(string) string

去掉字符串中,左边包含参数字符的字符,如果参数是空格,就是去掉左边的空格

------

trim_right(string) string

去掉字符串中,包含参数字符的字符,如果参数是空格,就是去掉右边的空格

------

all_before(string) string

提取字符串中,包含参数字符串前面的所有内容

------

all_after(string) string

提取字符串中,包含参数字符串后面的所有内容

------

limit(int) string

返回字符串的前几个字符串

------

reverse() string

反转字符串

------

clone()

复制字符串

------

replace(string,string) string

替换字符串中指定的子字符串替换为新的字符串

------

to_lower()

转小写

------

to_upper()

转大写

------

left(int) string

取字符串从左边开始,到第几个字符的部分

------

right(int) string

取字符串从左边开始第几个字符开始,所有右边的部分

------

containis(string) bool

判断字符串是否包含参数中的子字符串

------

substr(int,int) string

取给定开始和结束位置的子字符串

------

int() int

u32() u32

f32() f32

f64() f64

把字符串转换为整数,函数会尝试从左边开始逐个字符尝试转换成整数,碰到非数字的字符,则返回转换成功的部分,如果第一个字符就不能转换,则返回0

------

count(string) int

计算字符串中出现参数字符串的次数

------

capitalize()

把字符串的首字母大写

------

title()

把字符串中的每个单词的首字母大写,用空格区分单词

------

[ ]string.sort()

给字符串数组排序

------

[]string.join(string) string

把字符串数组,根据给定的连接符,连接成字符串

------

eq(string) bool 

判断两个字符串是否相等

------

ne(string) bool

比较两个字符串是否不相等

------

lt(string) bool 

比较字符串是否小于参数的字符串

le(string) bool

比较字符串是否小于等于参数的字符串

gt(string) bool

比较字符串是否大于参数的字符串

ge(string) bool

比较字符串是否大约等于参数的字符串

    字符串比较大小: < > == != ≠

   比较的规则是:字符串的大小是由左边开始最前面的字符决定的,跟长度没有关系,比较的时候，从字符串左边开始，一次比较每个字符，直接出现差异、或者其中一个串结束为止

   每一次字符的比较大小,实际是字符表中字符对应的整数的大小

   ASCII字符表:

   http://ascii.911cha.com/

   至今为止共定义了128个字符，其中33个字符无法显示

   在33个字符之外的是95个可显示的字符

#### 数组

```
vlib/builtin/array.v
```



------



#### 字典

```
vlib/builtin/map.v
```







------

#### 错误处理

```
vlib/builtin/option.v
```



error(string) Option //错误处理,返回Option变量