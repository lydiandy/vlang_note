## io模块

### 接口

#### 可读接口

```v
pub interface Reader {
 mut:
	read(mut buf []byte) ?int
}
```

#### 可写接口

```v
pub interface Writer {
mut:
	write(buf []byte) ?int
}
```

#### 随机读接口

```v
interface RandomReader {
	read_from(pos u64, mut buf []byte) ?int
}
```

#### 随机写接口

```v
pub interface RandomWriter {
	write_to(pos u64, buf []byte) ?int
}
```

#### 读写接口

```v
pub interface ReaderWriter {
mut:
	read(mut buf []byte) ?int
	write(buf []byte) ?int
}
//等价于接口多重组合的写法
pub interface ReaderWriter {
  Reader
  Writer
}
```

### 结构体

#### 带缓存的读取

```v
struct BufferedReader {
mut:
	reader Reader
	buf    []byte
	offset int // current offset in the buffer
	len    int
	fails  int // how many times fill_buffer has read 0 bytes in a row
	mfails int // maximum fails, after which we can assume that the stream has ended
pub mut:
	end_of_stream bool // whether we reached the end of the upstream reader
}
```

#### 带缓存读取的配置

```v
pub struct BufferedReaderConfig {
	reader  Reader
	cap     int = 128 * 1024 // large for fast reading of big(ish) files
	retries int = 2 // how many times to retry before assuming the stream ended
}
```

构造器，创建一个带缓存读取对象

```v
pub fn new_buffered_reader(o BufferedReaderConfig) &BufferedReader
```

方法：

```v
pub fn (mut r BufferedReader) read(mut buf []byte) ?int		//读取数据
pub fn (mut r BufferedReader) read_line() ?string		//一次读一行
pub fn (r BufferedReader) end_of_stream() bool		//是否读取完毕
pub fn (mut r BufferedReader) free() 			//释放缓存
```









