## net模块

基于C标准库中的网络编程函数进行了V语言风格的封装,基本思路一样

**公共函数:**

```c
//创建socket结构体
pub fn new_socket(family int, _type int, proto int) ?Socket
//创建udp类型的socket结构体
pub fn socket_udp() ?Socket 
//帮助类函数,用于服务端,指定端口,创建socket,bind(),listen()全部完成
pub fn listen(port int) ?Socket
//帮助器类函数,用于客户端,指定地址和端口,创建socket,connect()全部完成
pub fn dial(address string, port int) ?Socket
//返回错误码
fn error_code() int
//获取主机名
pub fn hostname() ?string

```

---

**Socket结构体:**

```c
pub struct Socket {
pub:
	sockfd int //socket的文件描述符
	family int //协议族,主要是C.AF_INET和C.AF_INET6
	_type  int //套接字类型,主要是:C.SOCK_STREAM和C.SOCK_DGRAM
	proto  int //传输协议,主要是:C.IPPROTO_TCP和C.IPPROTO_UDP
  //当第proto参数为0时，会自动选择参数类型对应的默认协议
}
```

**Socket方法:**

```c
//设置socket选项
pub fn (s Socket) setsockopt(level int, optname int, optvalue &int) ?int
//绑定端口
pub fn (s Socket) bind(port int) ?int 
//启动监听
pub fn (s Socket) listen() ?int
//启动监听,指定backlog
pub fn (s Socket) listen_backlog(backlog int) ?int
//开始接收数据
pub fn (s Socket) accept() ?Socket
//客户端连接到指定服务器
pub fn (s Socket) connect(address string, port int) ?int
//发送数据,指定字节指针和长度
pub fn (s Socket) send(buf byteptr, len int) ?int
//发送字符串数据
pub fn (s Socket) send_string(sdata string) ?int
//接收数据
pub fn (s Socket) recv(bufsize int) (byteptr,int)
//简单封装C的read,接收数据
pub fn (s Socket) cread(buffer byteptr, buffersize int) int
//简单封装C的recv,接收数据
pub fn (s Socket) crecv(buffer byteptr, buffersize int) int
//发送字符串数据,并且以\r\n结尾
pub fn (s Socket) write(str string) ?int
//接收一行数据,根据换行符\n
pub fn (s Socket) read_line() string 
//接收所有数据
pub fn (s Socket) read_all() string
//获取socket所在端口
pub fn (s Socket) get_port()
//关闭连接
pub fn (s Socket) close() ?int

```

附上网络连接经典图示:

![TCP协议通讯流程](https://img-blog.csdn.net/20140519112921000?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWF0cml4X2xhYm9yYXRvcnk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

---











