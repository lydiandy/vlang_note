## x模块

### json2

标准库中的json模块是基于CJSON实现的

json2则是纯V实现,目前还处在x实验性模块中,稳定后估计会替换掉标准库中的json模块

#### 类型

```v
//Any是联合类型,表示json任意类型的节点
pub type Any = string | int | i64 | f32 | f64 | any_int | any_float | bool | Null | []Any | map[string]Any
```

接口

```V
//JSON序列化接口,要进行JSON序列化的类型需要实现
pub interface Serializable {
	from_json(f Any) //decode中使用
	to_json() string //encode中使用
}
```

#### 编码

```v
//泛型版本的编码函数,将类型为T的变量编码为json字符串
pub fn encode<T>(typ T) string //类型需要实现序列化接口的to_json函数
```

#### 解码

```v
//泛型版本的解码函数
pub fn decode<T>(src string) T //直接返回类型为T的变量,类型需要实现序列化接口的from_json函数
//解码函数,会自动转换节点的值为对应类型
pub fn raw_decode(src string) ?Any //仅仅返回Any类型
//快速解码函数,忽略类型转换,节点的值都是字符串
pub fn fast_raw_decode(src string) ?Any
```

编码示例:

```v
import x.json2

enum JobTitle {
	manager
	executive
	worker
}

struct Employee {
	name   string
	age    int
	salary f32
	title  JobTitle
}

pub fn (e Employee) to_json() string {
	mut mp := map[string]json2.Any{}
	mp['name'] = e.name
	mp['age'] = e.age
	mp['salary'] = e.salary
	mp['title'] = int(e.title)
	return mp.str()
}

fn main() {
	x := Employee{'Peter', 28, 95000.5, .worker}
	s := json2.encode<Employee>(x)
	println(s)
	y := json2.raw_decode(s) or {
		panic(err)
	}
	println('Employee y: $y')
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

#### 演示实例代码:

```v
module main

import x.websocket
import time

fn main() {
	go start_server()
	time.sleep_ms(100)
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
		// sleep to give time to recieve response before send a new one
		time.sleep_ms(100)
	}
	// sleep to give time to recieve response before asserts
	time.sleep_ms(500)
}
```

