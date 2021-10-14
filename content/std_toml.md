## toml

V标准库包含了toml模块，toml模块是用纯V开发实现的，没有任何第三方依赖，目前完全兼容toml v1.0.0规范，使用toml模块可以很方便地使用toml作为配置文件。

### 解析toml

解析指定的toml文件：

```v
pub fn parse_file(path string) ?Doc
```

解析指定的toml文本：

```v
pub fn parse_text(text string) ?Doc
```

自动解析，如果给的参数是一个路径，则跟parse_file一样，解析文件；如果给的参数不是一个路径，而是文本，则跟parse_text一样，解析文本：

```v
pub fn parse(toml string) ?Doc
```

### 获取节点的值

解析完toml文件或文本后，返回一个Doc类型，可以调用Doc.value方法，获取节点的值，value返回Any类型，Any是一个联合类型，包含所有节点的类型种类。

并且value的参数key，可以通过点号表示toml的层级，这样就可以直接通过层级关系，获取所需节点的值。

```v
pub fn (d Doc) value(key string) Any
```

```v
pub type Any = Null	//空值类型节点
	| []Any	//数组类型节点
	| bool
	| f32
	| f64
	| i64
	| int
	| map[string]Any //字典类型节点
	| string
	| time.Time	//时间类型节点
	| u64
```

可以调用Any对应的方法，把节点值转换为对应的数据类型：

```v
pub fn (a Any) string()	string // 把节点值转换为sting类型
pub fn (a Any) int()	int
...
pub fn (a Any) array() []Any //转换为数组类型
pub fn (a Any) as_map() map[string]Any //转换为字典类型
```

### 将toml转化为json

```v
pub fn (d Doc) to_json() string 
```

### 完整示例

```v
import toml

const toml_text = '# This is a TOML document.

title = "TOML Example"

[owner]
name = "Tom Preston-Werner"
dob = 1979-05-27T07:32:00-08:00 # First class dates

[database]
server = "192.168.1.1"
ports = [ 8000, 8001, 8002 ]
connection_max = 5000
enabled = true

[servers]

  # Indentation (tabs and/or spaces) is allowed but not required
  [servers.alpha]
  ip = "10.0.0.1"
  dc = "eqdc10"

  [servers.beta]
  ip = "10.0.0.2"
  dc = "eqdc10"

[clients]
data = [ ["gamma", "delta"], [1, 2] ]

# Line breaks are OK when inside arrays
hosts = [
  "alpha",
  "omega"
]'

fn main() {
	// doc := toml.parse(toml_text) or { panic(err) } //解析文本
	// doc := toml.parse_file("./conifg.toml") or { panic(err) } //解析文件
	// doc := toml.parse("./conifg.toml") or { panic(err) } //自动解析
	doc := toml.parse(toml_text) or { panic(err) } //自动解析

	title := doc.value('title').string()
	println('title: "$title"')
	ip := doc.value('servers.alpha.ip').string() //value可以通过点号表示toml的层级
	println('Server IP: "$ip"')

	toml_json := doc.to_json() //转化为json格式
	println(toml_json)
}
```


