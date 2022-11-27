## x模块

### json2

标准库中的json模块是基于CJSON实现。

json2则是纯V实现，目前还处在x实验性模块中，稳定后估计会替换掉标准库中的json模块。

#### 类型

```v
//Any是联合类型,表示json任意类型的节点
pub type Any = Null
	| []Any
	| []int
	| bool
	| f32
	| f64
	| i16
	| i64
	| i8
	| int
	| map[string]Any
	| string
	| u16
	| u32
	| u64
	| u8

//联合类型转换成具体类型的方法,主要用于实现from_json方法
pub fn (f Any) as_map() map[string]Any //转字典
pub fn (f Any) arr() []Any	//转数组
pub fn (f Any) str() string	//转string
pub fn (f Any) int() int	//转int
pub fn (f Any) i64() i64	//转i64
pub fn (f Any) f32() f32	//转f32
pub fn (f Any) f64() f64	//转f64
pub fn (f Any) bool() bool	//转bool
```

#### 接口

```V
//JSON序列化接口,要进行JSON序列化的类型需要实现
pub interface Decodable {
	from_json(f Any) //decode中使用
}
pub interface Encodable {
	json_str() string //encode中使用
}
```

#### 编码

```v
//泛型版本的编码函数,将类型为T的变量编码为json字符串
pub fn encode[T](typ T) string //类型需要实现Encodable接口
```

#### 解码

```v
//泛型版本的解码函数
pub fn decode[T](src string) ?T //返回类型为T的变量,类型需要实现Decodable接口

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
mut:
	name   string
	age    int
	salary f32
	title  JobTitle
}

// 实现Encodable接口,给encode使用
pub fn (e Employee) json_str() string {
	mut mp := map[string]json2.Any{}
	mp['name'] = e.name
	mp['age'] = e.age
	mp['salary'] = e.salary
	mp['title'] = int(e.title)
	return mp.str()
}

// 实现Decodable接口,给decode使用
pub fn (mut e Employee) from_json(any json2.Any) {
	mp := any.as_map()
	e.name = mp['name'] or { '' }.str()
	e.age = mp['age'] or { 0 }.int()
	e.salary = mp['salary'] or { 0 }.f32()
	e.title = unsafe { JobTitle(mp['title'] or { '' }.int()) }
}

fn main() {
	x := Employee{'Peter', 28, 95000.5, .worker}
	s := json2.encode[Employee](x)
	// s := json2.encode(x) //也可以省略泛型类型
	println(s)
	// generic decode
	gy := json2.decode[Employee](s) or { panic(err) }
	println('Employee y: ${gy}')
	// raw_decode
	y := json2.raw_decode(s) or { panic(err) }
	println('Employee y: ${y}')
}
```

泛型版本的编码与解码：

```v
module main

import x.json2

const person_str = '
{
    "name": "abc",
    "age": 22,
    "pets": [
        "a",
        "b",
        "c"
    ]
}
'

struct Person {
mut:
    name string
    age  int = 20
    pets []string
}

pub fn (mut p Person) from_json(f json2.Any) {
    obj := f.as_map()
    for k, v in obj {
        match k {
            'name' { p.name = v.str() }
            'age' { p.age = v.int() }
            'pets' { p.pets = v.arr().map(it.str()) }
            else {}
        }
    }
}

pub fn (p Person) json_str() string {
    mut obj := map[string]json2.Any
    obj['name'] = p.name
    obj['age'] = p.age
    obj['pets'] = p.pets.str()
    return obj.str()
}

fn main() {
    person := json2.decode[Person](person_str)! //泛型版本的解码
    println(person)
    person_json := json2.encode[Person](person) //泛型版本的编码
    println(person_json)
}
```

