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

### 获取节点的值

解析完toml文件或文本后，返回一个Doc类型，可以调用Doc.value方法，获取节点的值，value返回Any类型，Any是一个联合类型，包含所有节点的类型种类。

并且value的参数key，可以通过点号表示toml的层级，这样就可以直接通过层级关系，获取所需节点的值。

```v
pub fn (d Doc) value(key string) Any
```

```v
pub type Any = Date	//日期类型
	| DateTime	//日期时间类型
	| Null	//空值类型
	| Time
	| []Any		//数组类型
	| bool
	| f32
	| f64
	| i64
	| int
	| map[string]Any	//字典类型
	| string
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

### toml注解

就像结构体的json注解那样，结构体也支持toml注解，实现结构体字段和toml字段的自定义

```v
import toml

const toml_text = '# This TOML can reflect to a struct
name = "Tom"
age = 45
height = 1.97

birthday = 1980-04-23

strings = [
  "v matures",
  "like rings",
  "spread in the",
  "water"
]

bools = [true, false, true, true]

floats = [0.0, 1.0, 2.0, 3.0]

int_map = {"a" = 0, "b" = 1, "c" = 2, "d" = 3}

[bio]
text = "Tom has done many great things"
years_of_service = 5

[field_remap]
txt = "I am remapped"
uint64 = 100

[config]
data = [ 1, 2, 3 ]
levels = { "info" = 1, "warn" = 2, "critical" = 3 }
'

struct FieldRemap {
	text string [toml: 'txt'] //支持toml字段名自定义注解
	num  u64    [toml: 'uint64']
}

struct Bio {
	text             string
	years_of_service int
}

struct User {
	name     string
	age      int
	height   f64
	birthday toml.Date
	strings  []string
	bools    []bool
	floats   []f32
	int_map  map[string]int

	config toml.Any
mut:
	bio   Bio
	remap FieldRemap
}

fn main() {
	toml_doc := toml.parse_text(toml_text) or { panic(err) }

	mut user := toml_doc.reflect<User>()
	user.bio = toml_doc.value('bio').reflect<Bio>()
	user.remap = toml_doc.value('field_remap').reflect<FieldRemap>() //根据自定义toml字段名进行反射解析

	assert user.name == 'Tom'
	assert user.age == 45
	assert user.height == 1.97
	assert user.birthday.str() == '1980-04-23'
	assert user.strings == ['v matures', 'like rings', 'spread in the', 'water']
	assert user.bools == [true, false, true, true]
	assert user.floats == [f32(0.0), 1.0, 2.0, 3.0]
	assert user.int_map == {
		'a': 0
		'b': 1
		'c': 2
		'd': 3
	}
	assert user.bio.text == 'Tom has done many great things'
	assert user.bio.years_of_service == 5

	assert user.remap.text == 'I am remapped'
	assert user.remap.num == 100

	assert user.config.value('data[0]').int() == 1
	assert user.config.value('levels.warn').int() == 2
}

```



### 将toml转化为json

```v
import toml.to
to.json(doc)
```

### 完整示例

```v
import toml
import toml.to

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
	doc := toml.parse_text(toml_text) or { panic(err) } //解析文本
	// doc := toml.parse_file("./conifg.toml") or { panic(err) } //解析文件
	title := doc.value('title').string()
	title2 := doc.value('title') as string
	owner := doc.value('owner') as map[string]toml.Any
	println('title: "$title"')
	println('title: "$title2"')
	println('owner: $owner')
	ip := doc.value('servers.alpha.ip').string() // value可以通过点号表示toml的层级
	println('Server IP: "$ip"')

	toml_json := to.json(doc) //转化为json格式
	println(toml_json)
}
```
