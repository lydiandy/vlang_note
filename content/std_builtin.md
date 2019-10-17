## builtin内置模块

#### 字符串

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

------



#### 数组





------



#### 字典









------

