## http模块

### http服务端

Request结构体

```v
pub struct Request {
pub:
	method     string
	headers    map[string]string
	cookies    map[string]string
	data       string
	url        string
	user_agent string
	verbose    bool
mut:
	user_ptr   voidptr
	ws_func    voidptr
}
```

Response结构体

```v
pub struct Response {
pub:
	text        string
	headers     map[string]string
	cookies     map[string]string
	status_code int
}
```



### http客户端

