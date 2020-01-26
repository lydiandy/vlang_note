## builtin内置模块

在builtin内置模块中定义的函数,结构体等,编译器在编译代码前会默认加载vlib/builtin目录,

内置模块中的所有函数,结构体可以在任何模块中直接使用,并且不用带buitlin前缀

### 内置函数

源代码位置:vlib/builtin/builtin.v

#### 打印输出

- print(s string)  

    打印字符串,不换行

- println(s string) 

    打印字符串,换行

- eprint(s string) 

    打印错误,不换行,控制台是红色字

- eprintln(s string) 
	
	打印错误,换行,控制台是红色字

#### 退出进程/报错

- exit(code int)     

    退出程序

- panic(s string) 

    程序报错,然后终止进程,退出码为1

- on_panic(f fn(int) int) 

    尚未实现,应该是恐慌后的回调处理

- is_atty(fd int) int 

    判断程序是否在终端执行,一般是is_atty(1) **>** 0,如果返回1,即>0,则表示在终端中执行,返回0则不是在终端中执行


#### 手动分配内存

- isnil(ptr voidptr)     

    正常由V创建的变量都会有初始值,所以不存在空指针,这个只用来集成C代码库或手动分配内存时,判断生成的C指针是否是空指针

- malloc(n int) byteptr    

  分配所需的内存空间，并返回一个指向它的指针,对C标准库的malloc的简单封装,成为内置函数,用途一样

- calloc(n int) byteptr      

  分配所需的内存空间，并返回一个指向它的指针,对C标准库的calloc的简单封装,不一样的是C的calloc有2个参数,V简化为1个size参数,成为内置函数,用途一样.malloc 和 calloc 之间的不同点是，malloc 不会设置内存为零，而 calloc 会设置分配的内存为零

- memdup(src voidptr, sz int) voidptr   

    内存拷贝

- free(ptr voidptr) 

  释放内存,对C标准库free的简单封装,成为内置函数,用途一样

以下3个常用的C分配内存函数没有定义成为内置函数,还需要通过C.xxx来使用:

- C.realloc(byteptr,int) 

    重新调整内存大小

- C.memcpy(byteptr,byteptr,int) 

    内存拷贝

- C.memmove(byteptr,byteptr,int) 

  内存移动

  基本上V代码中,都是使用这几个函数来实现手动内存分配,实现跟C一样的内存控制

  参考:[内存管理章节](memory.md)
  
  ------
  
  

### 字符串

源代码位置:vlib/builtin/string.v

- ends_with(string) bool	

    判断字符串是否以给定的字符串结尾

------

- starts_with(string) bool	

    判断字符串是否以给定的字符串开始

------

- find_between('[',']') string

    返回字符串中包在这两个字符中间的子字符串

------

- index(string) int

    返回子字符串在字符串中的位置,如果没有包含,则返回-1

------

- last_index(string) int 

    返回子字符串在字符串中,最后出现的位置

------

- index_any(string) int 

    返回子字符串中的任意单个字符,在字符串中出现的位置,如果没有包含,则返回-1

------

- index_after(str,n) int

    从字符串第n个开始查找起,返回子字符串在整个字符串中的位置,如果没有包含,则返回-1	

------

- at(int) byte

    返回字符串第几个位置的字符串

------

- split(string) [ ]string

    按照给定的分割符,把字符串分割,形成数组

------

- trim_space() string

    去掉字符串左右两边的空格,中间的空格不去掉

------

- trim(string) string

    去掉字符串中包含子字符串中的字符

------

- trim_left(string) string

    去掉字符串中,左边包含参数字符的字符,如果参数是空格,就是去掉左边的空格

------

- trim_right(string) string

    去掉字符串中,包含参数字符的字符,如果参数是空格,就是去掉右边的空格

------

- all_before(string) string

    提取字符串中,包含参数字符串前面的所有内容

------

- all_after(string) string

    提取字符串中,包含参数字符串后面的所有内容

------

- limit(int) string

    返回字符串的前几个字符串

------

- reverse() string

    反转字符串

------

- clone()

    复制字符串

------

- replace(string,string) string

    替换字符串中指定的子字符串替换为新的字符串

------

- to_lower()

    转小写

------

- to_upper()

    转大写

------

- left(int) string

    取字符串从左边开始,到第几个字符的部分

------

- right(int) string

    取字符串从左边开始第几个字符开始,所有右边的部分

------

- containis(string) bool

    判断字符串是否包含参数中的子字符串

------

- substr(int,int) string

    取给定开始和结束位置的子字符串

------

- int() int


- u32() u32


- f32() f32


- f64() f64

    把字符串转换为整数,函数会尝试从左边开始逐个字符尝试转换成整数,碰到非数字的字符,则返回转换成功的部分,如果第一个字符就不能转换,则返回0

------

- count(string) int

    计算字符串中出现参数字符串的次数

------

- capitalize()

    把字符串的首字母大写

------

- title()

    把字符串中的每个单词的首字母大写,用空格区分单词

------

- [ ]string.sort()

    给字符串数组排序

------

- []string.join(string) string

    把字符串数组,根据给定的连接符,连接成字符串

------

- eq(string) bool 

    判断两个字符串是否相等

------

- ne(string) bool

    比较两个字符串是否不相等

------

- lt(string) bool 

    比较字符串是否小于参数的字符串
- le(string) bool
  
    比较字符串是否小于等于参数的字符串
- gt(string) bool
    比较字符串是否大于参数的字符串

- ge(string) bool

    比较字符串是否大约等于参数的字符串

    ------

    字符串比较大小: < > == != ≠

    比较的规则是:字符串的大小是由左边开始最前面的字符决定的,跟长度没有关系,比较的时候，从字符串左边开始，一次比较每个字符，直接出现差异、或者其中一个串结束为止

    每一次字符的比较大小,实际是字符表中字符对应的整数的大小

    ASCII字符表:

    http://ascii.911cha.com/

    至今为止共定义了128个字符，其中33个字符无法显示

    在33个字符之外的是95个可显示的字符

### 数组

源代码位置:vlib/builtin/array.v

- str()  	
    数组转字符串

------
- first() 	
    返回数组的第一个元素
------

- last()	
	返回数组的最后一个元素
------

- delete(int)	

    删除数组的第几个元素

------

​    

- left(int) array	

    返回从左边开始,到第几个元素的子数组

------

​    

- right(int) array	

    返回从左边开始第几个之后,  右边的所有元素的子数组

------

​    

- slice(start,end)	

    返回给定位置区间的子数组,左闭右开

------



- reverse()	

    数组反转

------

​    

- clone()	

    克隆数组

------

​    

- insert(int,voidptr)	

    在数组的第几个位置插入新的元素,第二个参数是指针类型

------

​    

- prepend(voidptr)	

    在数组的第一个位置插入新的元素

------

​    

- free()	

    释放数组的内存

------

​    

- [ ]int.sort() 	

    针对整型数组的排序

------

- filter()	

    针对int和string数组进行过滤,返回满足条件的元素数组

    filter函数有点特殊,是在编译器中实现的,而不是builtin库中,因为有it这个特殊的迭代器参数

    it是参数表达式中,约定的iterator迭代器,表示每一次迭代时,数组的元素,满足过滤器表达式的元素会被返回

```c
	a := [1, 2, 3, 4, 5, 6]
	b := a.filter(it % 2 == 0) //b的结果为:[2,4,6]
	c := ['v', 'is', 'awesome']
	d := c.filter(it.len > 1) //d的结果为:['is','awesome']
```

------

- map() 	

    针对int和string数组的每一个元素进行一个运算,返回运算后的新数组

    map函数有点特殊,是在编译器中实现的,而不是builtin库中,因为有it这个特殊的迭代器参数

    it是参数表达式中,约定的iterator迭代器,表示每一次迭代时,数组的元素

```c
a := [1, 2, 3, 4, 5, 6]
b := a.map(it * 10)
println(b)
```

------

- reduce(iter fn (accum, curr int) int, accum_start int) int	

    针对int数组,给定一个初始的累计值accum_start,以及累计值与数组元素的累加关系,返回最终的累加结果

```c
module main

fn sum(accum int, curr int) int {
	return accum + curr
}

fn sub(accum int, curr int) int {
	return accum - curr
}

fn main() {
	a := [1, 2, 3, 4, 5]
	b := a.reduce(sum, 0)
	c := a.reduce(sum, 5)
	d := a.reduce(sum, -1)
	println(b) //返回15
	println(c) //返回20
	println(d) //返回14
	e := [1, 2, 3]
    f := e.reduce(sub, 0)
    g := e.reduce(sub, -1)
    println(f) //返回-6
    println(g) //返回-7
}
```



------

### 字典

源代码位置:vlib/builtin/map.v

- m.keys() 	

    获取map的所有key,返回keys数组

    ------

    

- m.delete(key)	

    删除map的某一个key

    ------

    

- m.str()	

    map转成字符串输出

    ------

    

- m.free()	

    释放map的内存



------

### 错误处理

源代码位置:vlib/builtin/option.v

- error(string) Option 

    错误处理,返回Option变量