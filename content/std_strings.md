## strings字符串生成器模块

字符串生成器用来生成一个动态的字符串,可以随时追加内容到字符串中

```v
pub struct Builder {
mut:
	buf          []byte //以字节数组的方式存储字符串,并可以追加内容到字节数组中
pub mut:
	len          int //动态字符串长度
	initial_size int=1 //初始大小
}
```

创建新的字符串生成器对象:

- strings.new_builder(initial_size int) Builder 

  根据给定的初始大小,生成一个字符串生成器


字符串生成器的方法:

- Builder.write(s string) 

    追加写入字符串

- Builder.writeln(s string) 

    追加写入字符串,并且换行

- Builder.write_b(data byte)

    追加写入单个字节

- Builder.write_bytes(bytes &byte,howmany int)

    追加写入多个字节

- Builder.str() string 

    字符串对象输出

- Builder.free() 

    释放字符串生成器内存,字符串生成器内容清空,长度归零

    

