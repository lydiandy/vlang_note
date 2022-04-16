## net模块

### TCP

#### 公共结构体

```v
//tcp连接
[heap]
pub struct TcpConn {  //实现io.Reader,io.Writer接口
pub mut:
	sock TcpSocket //Socket对象
mut:
	write_deadline time.Time
	read_deadline  time.Time
	read_timeout   time.Duration
	write_timeout  time.Duration
	is_blocking    bool
}

//tcp socket对象
struct TcpSocket {
pub:
	handle int //socket的文件描述符
}
```

方法：

```v
pub fn (mut c TcpConn) close() ?	//关闭连接，一般在函数的defer中调用

pub fn (mut c TcpConn) set_read_timeout(t time.Duration)	//设置读超时的时间
pub fn (mut c TcpConn) set_write_timeout(t time.Duration)	//设置写超时的时间

pub fn (c &TcpConn) read_timeout() time.Duration	//获取读超时的时间
pub fn (c &TcpConn) write_timeout() time.Duration	//获取写超时的时间

pub fn (mut c TcpConn) set_read_deadline(deadline time.Time) //设置读的限制时间点
pub fn (mut c TcpConn) set_write_deadline(deadline time.Time) //设置写的限制时间点

pub fn (mut c TcpConn) write_deadline() ?time.Time //获取写的限制时间点
```

#### 客户端

```v
//客户端拨号，成功则返回一个TCP连接
pub fn dial_tcp(address string) ?TcpConn
```

客户端连接例子:

```v
module main

import net
import io

fn main() {
	mut client_conn := net.dial_tcp('baidu.com:80') ?
	defer {
		client_conn.close() ?
	}
	client_conn.write_string('GET /index.html HTTP/1.0\r\n\r\n') ?
	client_conn.set_read_timeout(net.infinite_timeout)
	result := io.read_all(io.ReadAllConfig{ reader: client_conn }) ?
	println(result.bytestr())
}

```

#### 服务端

```v
//服务端启动监听
pub fn listen_tcp(family AddrFamily, saddr string) ?&TcpListener 
```

```v
//TCP监听器
pub struct TcpListener {
	sock TcpSocket

mut:
	accept_timeout time.Duration
	accept_deadline time.Time
}
```

方法：

```v
//监听成功后，等待客户端请求,调用accept以后,开始监听,接收一次请求成功后，返回TCP连接，
pub fn (l TcpListener) accept() ?TcpConn

pub fn (mut c TcpListener) set_accept_deadline(deadline time.Time) 
pub fn (mut c TcpListener) set_accept_timeout(t time.Duration)
```

服务端连接例子:

```v
module main

import net
import time

const (
	port = 9999
)

fn setup() (&net.TcpListener, &net.TcpConn, &net.TcpConn) {
	mut listener := net.listen_tcp(.ip6, ':$port') or { panic(err) }

	mut client := net.dial_tcp('127.0.0.1:$port') or { panic(err) }

	client.set_read_timeout(3 * time.second)
	client.set_write_timeout(3 * time.second)

	mut server := listener.accept() or { panic(err) }
	server.set_read_timeout(3 * time.second)
	server.set_write_timeout(3 * time.second)

	return listener, client, server
}

fn cleanup(mut listener net.TcpListener, mut client net.TcpConn, mut server net.TcpConn) {
	listener.close() or {}
	client.close() or {}
	server.close() or {}
}

fn main() {
	mut listener, mut client, mut server := setup()
	defer {
		cleanup(mut listener, mut client, mut server)
	}
	message := 'Hello World'
	// server
	server.write_string(message) ?
	println('message send: $message')
	println('server socket: $server.sock.handle')
	// client
	mut buf := []u8{len: 1024}
	nbytes := client.read(mut buf) ?
	received := buf[0..nbytes].bytestr()
	println('message received: $received')
	println('client socket: $client.sock.handle')
}

```

### UDP

实现思路基本跟TCP一致

#### 客户端

```v
//客户端拨号
pub fn dial_udp(raddr string) ?&UdpConn 
```

```v
//返回udp连接
pub struct UdpConn { //实现io.Reader,io.Writer接口
	sock UdpSocket //Socket对象

mut:
	write_deadline time.Time
	read_deadline time.Time

	read_timeout time.Duration
	write_timeout time.Duration
}
```

```v
//udp socket对象
struct UdpSocket {
	handle int //socket的文件描述符

mut:
	has_r bool
	r ?Addr //remote addr
}
```

#### 服务端

```v
//服务端启动监听,返回udp连接,udp不需要accept,直接开始
pub fn listen_udp(laddr string) ?&UdpConn
```

服务端连接例子:

```v
import net
import time

const (
	local_addr  = ':40003'
	remote_addr = '127.0.0.1:40003'
)

fn echo_server(mut c &net.UdpConn) {
	c.set_read_timeout(10 * time.second)
	c.set_write_timeout(10 * time.second)
	for { //无限循环监听,接收数据后,原样返回给客户端
		mut buf := []u8{len: 100, init: 0}
		read, addr := c.read(mut buf) or { continue }
		c.write_to(addr, buf[..read]) or {
			println('Server: connection dropped')
			return
		}
	}
}

fn echo() ? {
	//客户端拨号
	mut c := net.dial_udp(remote_addr) ?
	defer {
		c.close() or {}
	}

	c.set_read_timeout(10 * time.second)
	c.set_write_timeout(10 * time.second)

	data := 'Hello from vlib/net!'
	c.write_string(data) ?
	mut buf := []u8{len: 100, init: 0}
	read, addr := c.read(mut buf) ?
	assert read == data.len
	println('Got address $addr')
	for i := 0; i < read; i++ {
		assert buf[i] == data[i]
	}
	println('Got "$buf.bytestr()"')
	c.close() ?
	return
}

fn main() {
	mut l := net.listen_udp(local_addr) or { //服务端启动udp监听
		println(err)
		panic(err)
	}
	go echo_server(mut l)
	echo() or { println(err) }
	l.close() or {}
}


```

### websocket

#### 客户端

```v
//创建websocket客户端
pub fn new_client(address string) ?&Client
```

```v
//返回客户端对象
pub struct Client {
	is_server bool
mut:
	ssl_conn          &openssl.SSLConn // secure connection used when wss is used
	flags             []Flag     // flags used in handshake
	fragments         []Fragment // current fragments
	message_callbacks []MessageEventHandler // all callbacks on_message
	error_callbacks   []ErrorEventHandler   // all callbacks on_error
	open_callbacks    []OpenEventHandler    // all callbacks on_open
	close_callbacks   []CloseEventHandler   // all callbacks on_close
pub:
	is_ssl bool   // true if secure socket is used
	uri    Uri    // uri of current connection
	id     string // unique id of client
pub mut:
	header            http.Header  // headers that will be passed when connecting
	conn              &net.TcpConn // underlying TCP socket connection
	nonce_size        int = 16 // size of nounce used for masking
	panic_on_callback bool     // set to true of callbacks can panic
	state             State    // current state of connection
	logger            &log.Log // logger used to log messages
	resource_name     string   // name of current resource
	last_pong_ut      i64      // last time in unix time we got a pong message
}
```

```v
//定义4个回调函数
ws.on_open(fn (mut ws websocket.Client) ? 	//连接回调
ws.on_error(fn (mut ws websocket.Client, err string) ?	//报错回调
ws.on_message(fn (mut ws websocket.Client, msg &websocket.Message) ? //接收消息回调
ws.on_close(fn (mut ws websocket.Client, code int, reason string) ? //关闭连接回调

```

```v
//跟服务端建立连接
pub fn (mut ws Client) connect() ?
```

```v
//监听服务端发送的消息
pub fn (mut ws Client) listen() ?
```

```v
//实现io模块的读写接口,读写消息
pub fn (mut ws Client) write(bytes []u8, code OPCode) ?
pub fn (mut ws Client) write_str(str string) ?

```



#### 服务端

```v
//创建websocket服务端
pub fn new_server(family net.AddrFamily, port int, route string) &Server
```

```v
//返回服务端对象
pub struct Server {
mut:
	logger                  &log.Log // logger used to log
	ls                      &net.TcpListener      // listener used to get incoming connection to socket
	accept_client_callbacks []AcceptClientFn      // accept client callback functions
	message_callbacks       []MessageEventHandler // new message callback functions
	close_callbacks         []CloseEventHandler   // close message callback functions
pub:
	family net.AddrFamily = .ip
	port   int  // port used as listen to incoming connections
	is_ssl bool // true if secure connection (not supported yet on server)
pub mut:
	clients       map[string]&ServerClient // clients connected to this server
	ping_interval int = 30 // interval for sending ping to clients (seconds)
	state         State // current state of connection
}

struct ServerClient {
pub:
	resource_name string // resource that the client access
	client_key    string // unique key of client
pub mut:
	server &Server
	client &Client
}

```

```v
//创建3个回调函数
s.on_connect(fn (mut s websocket.ServerClient) ?bool //连接回调
s.on_message(fn (mut ws websocket.Client, msg &websocket.Message) ? //消息接收回调
s.on_close(fn (mut ws websocket.Client, code int, reason string) ?  //关闭连接回调            
```

```v
//开始启动监听
pub fn (mut s Server) listen() ?
```

#### 演示实例代码

```v
module main

import net.websocket
import time

fn main() {
	go start_server()
	time.sleep(100 * time.millisecond)
	ws_client('ws://127.0.0.1:30000') ?
}

fn start_server() ? {
	mut s := websocket.new_server(.ip6, 30000, '')
	s.ping_interval = 100
	s.on_connect(fn (mut s websocket.ServerClient) ?bool {
		// Here you can look att the client info and accept or not accept
		// just returning a true/false
		if s.resource_name != '/' {
			panic('unexpected resource name in test')
			return false
		}
		return true
	}) ?
	s.on_message(fn (mut ws websocket.Client, msg &websocket.Message) ? {
		ws.write(msg.payload, msg.opcode) or { panic(err) }
	})
	s.on_close(fn (mut ws websocket.Client, code int, reason string) ? {
		println('client ($ws.id) closed connection')
	})
	s.listen() or { println('error on server listen: $err') }
}

fn ws_client(uri string) ? {
	eprintln('connecting to $uri ...')
	mut ws := websocket.new_client(uri) ?
	ws.on_open(fn (mut ws websocket.Client) ? {
		println('open!')
		ws.pong() or { panic(err) }
	})
	ws.on_error(fn (mut ws websocket.Client, err string) ? {
		println('error: $err')
		// this can be thrown by internet connection problems
	})
	ws.on_close(fn (mut ws websocket.Client, code int, reason string) ? {
		println('closed')
	})
	ws.on_message(fn (mut ws websocket.Client, msg &websocket.Message) ? {
		println('client got type: $msg.opcode payload:\n$msg.payload')
		if msg.opcode == .text_frame {
			smessage := msg.payload.bytestr()
			println('Message: $smessage')
		} else {
			println('Binary message: $msg')
		}
	})
	ws.connect() or { panic('fail to connect') }
	go ws.listen()
	text := ['a'].repeat(2)
	for msg in text {
		ws.write(msg.bytes(), .text_frame) or { panic('fail to write to websocket') }
		// wait to give time to recieve response before send a new one
		time.sleep(100 * time.millisecond)
	}
	// wait to give time to recieve response before asserts
	time.sleep(500 * time.millisecond)
}

```

### urllib

把url字符串解析成URL类型：

```v
pub fn parse(rawurl string) ?URL
```

URL结构体：

```v
pub struct URL {
pub mut:
	scheme      string
	opaque      string    // encoded opaque data
	user        &Userinfo // username and password information
	host        string    // host or host:port
	path        string    // path (relative paths may omit leading slash)
	raw_path    string    // encoded path hint (see escaped_path method)
	force_query bool      // append a query ('?') even if raw_query is empty
	raw_query   string    // encoded query values, without '?'
	fragment    string    // fragment for references, without '#'
}

struct Userinfo {
pub:
	username     string
	password     string
	password_set bool
}
```

URL方法：

```v
pub fn (u &URL) hostname() string 
pub fn (u &URL) port() string
pub fn (u &URL) query() Values
```

其他函数：

```v
pub fn parse_query(query string) ?Values

```

```v
struct Value {
pub mut:
	data []string
}

struct Values {
pub mut:
	data map[string]Value
	len  int
}
```



### 附录:TCP网络连接示意图

![TCP协议通讯流程](std_net.assets/SouthEast.png)

---











