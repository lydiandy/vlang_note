## http模块

### 公共枚举

#### http协议版本

用来表示http协议版本的枚举

```v
pub enum Version { 
	unknown
	v1_1
	v2_0
	v1_0
}
```

#### http方法

用来表示http方法的枚举

```v
pub enum Method {
	get
	post
	put
	head
	delete
	options
	trace
	connect
	patch
}
```

#### http状态码

```v
pub enum Status {
	unknown = -1
	unassigned = 0
	cont = 100
	switching_protocols = 101
	processing = 102
	checkpoint_draft = 103
	ok = 200
	created = 201
	accepted = 202
	non_authoritative_information = 203
	no_content = 204
	reset_content = 205
	partial_content = 206
	。。。
```

### 公共结构体

#### http请求

```v
pub struct Request {
pub mut:
	version    Version = .v1_1
	method     Method
	header     Header
	cookies    map[string]string
	data       string
	url        string
	user_agent string = 'v.http'
	verbose    bool
	user_ptr   voidptr

	read_timeout  i64 = 30 * time.second
	write_timeout i64 = 30 * time.second
	
	validate               bool
	verify                 string
	cert                   string
	cert_key               string
	in_memory_verification bool 
}
```

#### http响应

```v
pub struct Response {
pub mut:
	text         string
	header       Header
	status_code  int
	status_msg   string
	http_version string
}
```

#### header头部

```v
[noinit]
pub struct Header {
mut:
	data map[string][]string
	keys map[string][]string
}
```

#### cookie

```v
pub struct Cookie {
pub mut:
	name        string
	value       string
	path        string   
	domain      string   
	expires     time.Time 
	raw_expires string   

	max_age   int
	secure    bool
	http_only bool
	same_site SameSite
	raw       string
	unparsed  []string
}
```

### http客户端

发起请求的配置结构体：

```v
pub struct FetchConfig {
pub mut:
	url        string
	method     Method
	header     Header
	data       string
	params     map[string]string
	cookies    map[string]string
	user_agent string = 'v.http'
	verbose    bool
	
	validate               bool   
	verify                 string 
	cert                   string 
	cert_key               string 
	in_memory_verification bool   
}
```

发起客户端请求函数：

```v
pub fn fetch(config FetchConfig) ?Response	//通用的发起请求函数，给以下函数统一调用

pub fn get(url string) ?Response	//http.get
pub fn post(url string, data string) ?Response	//http.post
pub fn put(url string, data string) ?Response	//http.put
pub fn head(url string) ?Response	//http.head
pub fn patch(url string, data string) ?Response	//http.patch
pub fn delete(url string) ?Response	//http.delete

pub fn post_json(url string, data string) ?Response  //传输json数据
pub fn post_form(url string, data map[string]string) ?Response	//传输表单数据
pub fn post_multipart_form(url string, conf PostMultipartFormConfig) ?Response//附件
```

### http服务端

```v
pub struct Server {
mut:
	state    ServerStatus = .closed
	listener net.TcpListener
pub mut:
	port           int           = 8080
	handler        Handler       = DebugHandler{}
	read_timeout   time.Duration = 30 * time.second
	write_timeout  time.Duration = 30 * time.second
	accept_timeout time.Duration = 30 * time.second
}
```



### 关键概念

客户端client

服务端server

发送请求request

回复响应response

报文类型：请求报文，响应报文

http协议，http协议版本

http方法

报文结构：报文头部header，报文主体body

报文头部 header，有的报文只可用于请求，用的报文只可用于响应，有的2者都可用

状态码 status

状态码短语 status phase

主机host

URI统一资源定位

无状态协议

小饼干 cookie



