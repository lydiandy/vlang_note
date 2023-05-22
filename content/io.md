## io模块

### 接口

#### 可读接口

可读接口表示一个可以被读取的对象，比如文件或网络。

读取的内容会被写入到参数中的buf字节数组，返回值表示已读取的字节数，如果全部读取完成，即EOF，则抛出错误，可在调用的or代码块中进行读取完毕的操作。

相对于随机读，可读取对象只能从头开始读取。

```v
pub interface Reader {
mut:
	read(mut buf []u8) !int
}
```

#### 可写接口

可写接口表示一个可以被写入的对象，比如文件或网络。

把参数中的buf字节数组，逐字节写入到可写入对象中，返回值表示已写入的字节数，如果全部写入完成，则抛出错误，可在调用的or代码块中进行写入完毕的操作。

相对于随机写，可写对象只能从头开始写入。

```v
pub interface Writer {
mut:
	write(buf []u8) !int
}
```

#### 可读写接口

可读写接口是可读接口和可写接口的组合，表示一个可以被读取和写入的对象。

```v
pub interface ReaderWriter {
  Reader
  Writer
}
```

#### 随机读接口

随机读接口表示一个可以从指定位置开始读取的对象，其他跟可读接口一样。

```v
pub interface RandomReader {
	read_from(pos u64, mut buf []u8) !int
}
```

#### 随机写接口

随机写接口表示一个可以从指定位置开始写入的对象，其他跟可写接口一样。

```v
pub interface RandomWriter {
	write_to(pos u64, buf []u8) !int
}
```

实际应用中，读写文件或网络的时候，内存代码执行的速度总是比硬盘和网络的读写速度快，一般会有阻塞或延迟，所以都是把读写方法放在一个for循环中，进行等待。

完整示例，可以参考vlib/reader.v中的read_all函数实现。

```v
for {
	new_read := r.read(mut b[read..]) or { break }
}
```

### 函数

```v
//读取所有字节，如果ReadAllConfig选项中read_to_end_of_stream设置
//为true,就算是read函数读取不到任何字节，也不退出，继续等待读取，
//直到读取完Reader的所有字节,抛出错误才退出。如果read_to_end_of_stream
//设置为false，并且read函数读取不到任何字节，就会退出
pub fn read_all(config ReadAllConfig) ![]u8	
//读取所有字节，但是只要中途出现调用read函数时返回0字节就退出
pub fn read_any(mut r Reader) ![]u8	
```

read_all的参数配置

```v
pub struct ReadAllConfig {
	read_to_end_of_stream bool //true表示要等到全部读取完，才能退出
mut:
	reader Reader	//要读取的对象
}
```

### 结构体

#### 带缓冲区的读

```v
struct BufferedReader {
mut:
	reader Reader 	// 要读取的对象
	buf    []u8 	// 缓冲区的字节数组
	offset int 			// 当前缓冲已读取的位置
	len    int			// 当前缓冲的长度
	fails  int 			// 调用read函数时，返回0字节的次数 
	mfails int 			// 最多允许出现返回0字节的次数 
pub mut:
	end_of_stream bool // 是否已读取到EOF
}
```

#### 带缓冲区读的参数配置

```v
pub struct BufferedReaderConfig {
	reader  Reader 						// 要读取的对象
	cap     int = 128 * 1024 	// 缓冲区的初始容量
	retries int = 2					  // 读取返回0字节时，允许重试的次数
}
```

构造器，创建一个带缓冲区读取对象

```v
pub fn new_buffered_reader(o BufferedReaderConfig) &BufferedReader
```

方法：

```v
//从缓冲中读取数据到参数中的字节数组，返回值表示已读取的字节数，如果已读取
//完毕，则抛出错误
pub fn (mut r BufferedReader) read(mut buf []u8) !int		
//从缓冲区读取一行，返回读取的字符串
pub fn (mut r BufferedReader) read_line() !string		
//是否读取完毕
pub fn (r BufferedReader) end_of_stream() bool		
//释放缓冲区占用的内存资源
pub fn (mut r BufferedReader) free() 			
```
