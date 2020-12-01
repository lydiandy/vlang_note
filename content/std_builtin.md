## builtin内置模块

在builtin内置模块中定义的函数,结构体等,编译器在编译代码前会默认加载vlib/builtin目录,

内置模块中的所有函数,结构体可以在任何模块中直接使用,并且不用带buitlin前缀

### 内置函数

源代码位置:vlib/builtin/builtin.v

#### 打印输出

- print(s string)  

    打印字符串,不换行

------

- println(s string) 

    打印字符串,换行

------

- eprint(s string) 

    打印错误,不换行,控制台是红色字

------

- eprintln(s string) 
	
	打印错误,换行,控制台是红色字
	
------


#### 退出进程/报错

- exit(code int)     

    退出程序

------

- panic(s string) 

    程序报错,然后终止进程,退出码为1

------

- on_panic(f fn(int) int) 

    尚未实现,应该是恐慌后的回调处理

------

- is_atty(fd int) int 

    判断程序是否在终端执行,一般是is_atty(1) **>** 0,如果返回1,即>0,则表示在终端中执行,返回0则不是在终端中执行
    
------


#### 手动分配内存

- isnil(ptr voidptr) bool

    正常由V创建的变量都会有初始值,所以不存在空指针,这个只用来集成C代码库或手动分配内存时,判断生成的C指针是否是空指针

------

- malloc(n int) byteptr    

  分配所需的内存空间，并返回一个指向它的指针,对C标准库的malloc的简单封装,成为内置函数,用途一样

------

- calloc(n int) byteptr      

  分配所需的内存空间，并返回一个指向它的指针,对C标准库的calloc的简单封装,不一样的是C的calloc有2个参数,V简化为1个size参数,成为内置函数,用途一样.malloc 和 calloc 之间的不同点是，malloc 不会设置内存为零，而 calloc 会设置分配的内存为零

------

- memdup(src voidptr, sz int) voidptr   

    内存拷贝

------

- free(ptr voidptr) 

  释放内存,对C标准库free的简单封装,成为内置函数,用途一样
  
------

以下3个常用的C分配内存函数没有定义成为内置函数,还需要通过C.xxx来使用:

- C.realloc(byteptr,int) voidptr

    重新调整内存大小

------


- C.memcpy(byteptr,byteptr,int)  voidptr

    内存拷贝

------

- C.memmove(byteptr,byteptr,int) voidptr

  内存移动

------

  基本上V代码中,都是使用这几个函数来实现手动内存分配,实现跟C一样的内存控制

  参考:[内存管理章节](./memory.md)

------


### 字符串

源代码位置:vlib/builtin/string.v

**字符串函数:**

- vstrlen(s byteptr) int
    传入字节指针,计算指针指向的字符串的长度,一般用于C指针指向的字符串
- tos(s byteptr,len int) string
    传入字节指针和长度,转换成字符串,一般用于C指针转换成V字符串
- tos2(s byteptr) string

    传入字节指针,转换成字符串,字符串的长度自动识别,一般用于C指针转换成V字符串

- tos3(s charptr) string

    传入字符指针,转换成字符串,一般用于C指针转换成V字符串

- tos_clone(s byteptr) string

    传入字节指针,通过克隆的方式,生成新的字符串,内容跟字节指针指向的字符串一致

**字符串方法:**

- s.index(p string) ?int

    返回子字符串第一次在字符串中出现的位置,如果没有出现过,则抛出错误
    
- s.last_index(p string) ?int 

    返回子字符串最后一次在字符串中出现的位置,如果没有出现过,则抛出错误
    
- s.index_after(str string,n int) int
    从字符串第n个开始查找起,返回子字符串第一次出现的位置,如果没有包含,则返回-1
    
- s.index_byte(c byte) int

    返回单字符第一次在字符串中出现的位置,如果没有出现过,则返回-1
    
- s.last_index_byte(c byte) int

    返回单字符最后一次在字符串中出现的位置,如果没有出现过,则返回-1
    
- s.starts_with(p string) bool

    判断字符串是否以给定的字符串开始
    
- s.ends_with(p string) bool	

    判断字符串是否以给定的字符串结尾
    
- s.contains(p string) bool
  
  
    判断字符串是否包含参数中的子字符串

- s.find_between(start string,end string) string

    返回字符串中包在这两个字符中间的子字符串,如果start字符串不存在则返回空字符串,如果start字符串存在,而end字符串不存在,则返回start字符串之后的所有字符串

- s.index_any(s string) int 

    返回子字符串中的任意单个字符,在字符串中出现的位置,如果没有包含,则返回-1

- s.at(n int) byte

    返回字符串指定位置的字符

- s.split(s string) [ ]string

    按照给定的分割符,把字符串分割,形成数组,比较常用的是分隔符为空格进行分割,形成数组

- s.trim_space() string

    去掉字符串左右两边的空格,中间的空格不去掉

- s.trim(s string) string

    去掉字符串中包含子字符串中的字符

- s.trim_left(s string) string

    去掉字符串中,左边包含参数字符的字符,如果参数是空格,就是去掉左边的空格

- s.trim_right(s string) string

    去掉字符串中,包含参数字符的字符,如果参数是空格,就是去掉右边的空格

- s.all_before(s string) string

    提取字符串中,包含参数字符串(如果出现多个,取第一个出现)前面的所有内容

- s.all_before_last(s string) string 

    提取字符串中,包含参数字符串(如果出现多个,取最后出现)前面的所有内容

- s.all_after(s string) string

    提取字符串中,包含参数字符串(如果出现多个,取最后出现)后面的所有内容

- s.after(s string) string

    同all_after函数

- s.limit(n int) string

    返回字符串的前几个字符串

- s.reverse() string

    反转字符串

- s.clone() string

    复制字符串,生成新的字符串

- s.replace(rep string,with string) string

    替换字符串中所有的子字符串rep,替换为新的子字符串with

- s.replace_once(rep string,with string) string

    替换字符串中第一次出现的子字符串rep,替换为新的字符串with
    
- s.to_lower()

    转小写

- s.to_upper()

    转大写

- s.left(n int) string

    取字符串从左边开始,到第几个字符的部分

- s.right(n int) string

    取字符串从左边开始第几个字符开始,所有右边的部分

- s.substr(start int,end int) string

    取给定开始和结束位置的子字符串

- s.int() int


- s.u32() u32


- s.f32() f32


- s.f64() f64

    把字符串转换为整数,函数会尝试从左边开始逐个字符尝试转换成整数,碰到非数字的字符,则返回转换成功的部分,如果第一个字符就不能转换,则返回0

- s.count(p string) int

    返回字符串中出现参数字符串的次数,如果没有出现过,则返回-1

- s.capitalize()

    把字符串的首字母大写

- s.title()

    把字符串中的每个单词的首字母大写,用空格区分单词

- [ ]string.sort()

    给字符串数组排序

- []string.contains(p string) bool

    判断字符串数组是否包含指定的字符串
    
- []string.join(string) string

    把字符串数组,根据给定的连接符,连接成字符串

- eq(string) bool 

    判断两个字符串是否相等

- ne(string) bool

    比较两个字符串是否不相等

- lt(string) bool 

    比较字符串是否小于参数的字符串
    
- le(string) bool
  
    比较字符串是否小于等于参数的字符串
    
- gt(string) bool
    比较字符串是否大于参数的字符串

- ge(string) bool

    比较字符串是否大约等于参数的字符串

    字符串比较大小: < > == != ≠

    比较的规则是:字符串的大小是由左边开始最前面的字符决定的,跟长度没有关系,比较的时候，从字符串左边开始，一次比较每个字符，直接出现差异、或者其中一个串结束为止

    每一次字符的比较大小,实际是字符表中字符对应的整数的大小

    ASCII字符表:

    http://ascii.911cha.com/

    至今为止共定义了128个字符，其中33个字符无法显示

    在33个字符之外的是95个可显示的字符
    
- filter(fn (b type) bool) string

    字符串过滤函数,逐个字节判断,满足返回条件的字符串

    ```v
    module main
    
    fn main() {
    	foo := 'V is awesome!!!!'.filter(fn (b byte) bool {
    		return b != `!`
    	})
    	println(foo)
    }
    
    ```

    

**字符方法:**

- c.is_white() bool 

  判断字符是否是空格

- c.is_digit() bool

  判断字符是否是数字

- c.is_bin_digit() bool

  判断字符是否是二进制数字0或1

- c.is_oct_digit() bool

  判断字符是否是八进制数字0-7

- c.is_hex_digit() bool

  判断字符是否是16进制数字0-f或0-F

- c.is_letter() bool

  判断字符是否是字母

### 数组

源代码位置:vlib/builtin/array.v

**数组方法:**

- a.str()  string	
    数组转字符串,数组默认实现了str函数,就可以直接使用print或println函数用于输出

- a.first() voidptr
    返回数组的第一个元素
    
    之所以函数返回voidptr,是因为要兼容不同类型的数组,返回的还是第一个元素的值,使用上没有影响

- a.last() voidptr
	返回数组的最后一个元素
	
	之所以函数返回voidptr,是因为要兼容不同类型的数组,返回的还是最后一个元素的值,使用上没有影响

- a.delete(i int)	

    删除数组的第几个元素,该方法会改变数组本身

- a.reverse() array

    数组反转

- a.clone() array

    克隆数组

- a.insert(i int,val voidptr) array

    在数组的第几个位置插入新的元素,第二个参数是指针类型,,该方法会改变数组本身

- a.prepend(val voidptr)	

    在数组的第一个位置插入新的元素,,该方法会改变数组本身

- a.repeat(n int) array 

    定义一个指定长度,指定默认值的数组

    ```v
    arr := [0].repeat(50) //元素初始值为0,长度为50,容量为50
    println(arr.len) //返回50
    println(arr.cap) //返回50
    ```

    ```v
    arr := ['a','b'].repeat(3) //元素初始值为a,b,重复3次,长度为6,容量为6
    println(arr.len) //返回6
    println(arr.cap) //返回6
    println(arr) //返回['a','b','a','b','a','b']
    ```

- a.free()	

    释放数组的内存

- a.filter()	

    对数组元素进行过滤,返回满足条件的元素数组

    filter函数有点特殊,是在编译器中实现的,而不是builtin库中,因为有it这个特殊的迭代器参数

    it是参数表达式中,约定的iterator迭代器,表示每一次迭代时,数组的元素,满足过滤器表达式的元素会被返回

```v
fn main() {
	a := [1, 2, 3, 4, 5, 6]
	b := a.filter(it % 2 == 0)
	println(b) // 输出:[2,4,6]
	c := ['v', 'is', 'awesome']
	d := c.filter(it.len > 1)
	println(d) // 输出:['is','awesome']
}
```

- a.map() 	

    对数组的每一个元素进行一个运算,返回运算后的新数组

    map函数有点特殊,是在编译器中实现的,而不是builtin库中,因为有it这个特殊的迭代器参数

    it是参数表达式中,约定的iterator迭代器,表示每一次迭代时,数组的元素

```v
fn main() {
	a := [1, 2, 3, 4, 5]
	b := a.map(it * 10)
	println(b) // 返回[10, 20, 30, 40, 50]
}
```



- []int.reduce(iter fn (accum, curr int) int, accum_start int) int	

    针对int数组,给定一个初始的累计值accum_start,以及累计值与数组元素的累加关系,返回最终的累加结果

```v
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

- [ ]int.sort() 	

  对整型数组进行排序

- []int.index(v int) int

  返回指定的值,在整型数组中的位置

- []string.index(v int) int

  返回指定的值,在字符串数组中的位置
  
- []byte.index(v byte) int
  
  返回指定的值,在字节数组中的位置

------

### 字典

源代码位置:vlib/builtin/map.v

- m.keys() []string

    获取map的所有key,返回keys数组

- m.delete(key string)	

    删除map的某一个key

- m.str() string

    map转成字符串输出

- m.free()	

    释放map的内存

------

### 错误处理

源代码位置:vlib/builtin/option.v

- error(s string) Option 

    抛出错误,带错误消息,返回Option
    
- error_with_code(s string, code int) Option

    抛出错误,带错误消息和错误码,返回Option

    