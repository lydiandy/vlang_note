## net模块

### TCP

#### 客户端

```v
//客户端拨号
pub fn dial_tcp(address string) ?TcpConn
```

```v
//返回tcp连接
pub struct TcpConn { //实现io.Reader,io.Writer接口
pub:
	sock TcpSocket //Socket对象

mut:
	write_deadline time.Time
	read_deadline time.Time

	read_timeout time.Duration
	write_timeout time.Duration
}
```

```v
//tcp socket对象
struct TcpSocket {
pub:
	handle int //socket的文件描述符
}
```

客户端连接例子:

```v
module main

import net
import io

fn main() {
	mut client_conn := net.dial_tcp('baidu.com:80')?
	defer {
		client_conn.close()
	}
	client_conn.write_str('GET /index.html HTTP/1.0\r\n\r\n') ?
	client_conn.set_read_timeout(net.infinite_timeout)
	result := io.read_all(client_conn) ?
	println(result.bytestr())
}
```

#### 服务端

```v
//服务端启动监听
pub fn listen_tcp(port int) ?TcpListener
```

```v
//返回监听器/服务器对象
pub struct TcpListener {
	sock TcpSocket

mut:
	accept_timeout time.Duration
	accept_deadline time.Time
}
```

```v
//监听器/服务器对象,调用accept以后,开始监听,返回TcpConn对象
pub fn (l TcpListener) accept() ?TcpConn
```

服务端连接例子:

```v
module main

import net
import time

const (
	server_port = 22334
)

fn setup() (net.TcpListener, net.TcpConn, net.TcpConn) {
	listener := net.listen_tcp(server_port) or {
		panic(err)
	}
	mut client := net.dial_tcp('127.0.0.1:$server_port') or {
		panic(err)
	}
	client.set_read_timeout(3 * time.second)
	client.set_write_timeout(3 * time.second)
	mut server := listener.accept() or {
		panic(err)
	}
	server.set_read_timeout(3 * time.second)
	server.set_write_timeout(3 * time.second)
	return listener, client, server
}

fn cleanup(listener &net.TcpListener, client &net.TcpConn, server &net.TcpConn) {
	listener.close() or { }
	client.close() or { }
	server.close() or { }
}

fn main() {
	listener, client, server := setup()
	defer {
		cleanup(listener, client, server)
	}
	message := 'Hello World'
	//server
	server.write_str(message) ?
	println('message send: $message')
	println('server socket: $server.sock.handle')
	//client
	mut buf := []byte{len: 1024}
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
pub fn dial_udp(laddr string, raddr string) ?UdpConn
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

	l Addr //local addr
	r ?Addr //remote addr
}
```

#### 服务端

```v
//服务端启动监听,返回udp连接,udp不需要accept,直接开始
pub fn listen_udp(port int) ?UdpConn
```

服务端连接例子:

```v
import net
import time

fn echo_server(_c net.UdpConn) {
	mut c := _c
	c.set_read_timeout(10 * time.second)
	c.set_write_timeout(10 * time.second)
	for { //无限循环监听,接收数据后,原样返回给客户端
		mut buf := []byte{len: 100, init: 0}
		read, addr := c.read(mut buf) or {
			continue
		}
		c.write_to(addr, buf[..read]) or {
			println('Server: connection dropped')
			return
		}
	}
}

fn echo() ? {
  //客户端拨号
	mut c := net.dial_udp('127.0.0.1:40003', '127.0.0.1:40001') ?
	defer {
		c.close() or { }
	}
	c.set_read_timeout(10 * time.second)
	c.set_write_timeout(10 * time.second)
	data := 'Hello from vlib/net!'
	c.write_str(data) ?
	mut buf := []byte{len: 100, init: 0}
	read, addr := c.read(mut buf) ?
	assert read == data.len
	println('Got address $addr')
	for i := 0; i < read; i++ {
		assert buf[i] == data[i]
	}
	println('Got "$buf.bytestr()"')
	c.close() ?
	return none
}

fn test_udp() {
	l := net.listen_udp(40001) or { //服务端启动udp监听
		println(err)
		panic('')
	}
	go echo_server(l)
	echo() or {
		println(err)
	}
	l.close() or { }
}

fn main() {
	test_udp()
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
	is_server         bool
mut:
	ssl_conn          &openssl.SSLConn
	flags             []Flag
	fragments         []Fragment
	logger            &log.Log
	message_callbacks []MessageEventHandler
	error_callbacks   []ErrorEventHandler
	open_callbacks    []OpenEventHandler
	close_callbacks   []CloseEventHandler
pub:
	is_ssl            bool
	uri               Uri
	id                string
pub mut:
	conn              net.TcpConn
	nonce_size        int = 16 // you can try 18 too
	panic_on_callback bool
	state             State
	resource_name     string
	last_pong_ut      u64
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
pub fn (mut ws Client) write(bytes []byte, code OPCode) ?
pub fn (mut ws Client) write_str(str string) ?

```



#### 服务端

```v
//创建websocket服务端
pub fn new_server(port int, route string) &Server
```

```v
//返回服务端对象
pub struct Server {
mut:
	clients                 map[string]&ServerClient
	logger                  &log.Log
	ls                      net.TcpListener
	accept_client_callbacks []AcceptClientFn
	message_callbacks       []MessageEventHandler
	close_callbacks         []CloseEventHandler
pub:
	port                    int
	is_ssl                  bool
pub mut:
	ping_interval           int = 30 // in seconds
	state                   State
}
//
struct ServerClient {
pub:
	resource_name string
	client_key    string
pub mut:
	server        &Server
	client        &Client
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

import x.websocket
import time

fn main() {
	go start_server()
	time.sleep(100*time.millisecond)
	ws_client('ws://localhost:30000') ?
}

fn start_server() ? {
	mut s := websocket.new_server(30000, '')
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
		ws.write(msg.payload, msg.opcode) or {
			panic(err)
		}
	})
	s.on_close(fn (mut ws websocket.Client, code int, reason string) ? {
		println('client ($ws.id) closed connection')
	})
	s.listen() or {
		println('error on server listen: $err')
	}
}

fn ws_client(uri string) ? {
	eprintln('connecting to $uri ...')
	mut ws := websocket.new_client(uri) ?
	ws.on_open(fn (mut ws websocket.Client) ? {
		println('open!')
		ws.pong()
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
	ws.connect() or {
		panic('fail to connect')
	}
	go ws.listen()
	text := ['a'].repeat(2)
	for msg in text {
		ws.write(msg.bytes(), .text_frame) or {
			panic('fail to write to websocket')
		}
		// wait to give time to recieve response before send a new one
		time.sleep(100*time.millisecond)
	}
	// wait to give time to recieve response before asserts
	time.sleep(500*time.millisecond)
}
```



### 附录:TCP网络连接示意图

![TCP协议通讯流程](std_net.assets/SouthEast.png)

---











