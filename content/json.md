## 内置json支持

V语言内置了对json格式的支持,通过cJSON库来实现,没有使用运行时反射,性能是比较好的

使用的时候,先导入标准库的json包

### 编码

encode(object)    //object是要编码的变量

### 解码

 decode(struct,string)   //第一个参数是结构体模板,第二个参数是要解码的字符串

### 结构体标注

可以在结构体的字段上增加标注:

[skip]          //忽略这个字段不解析

[json:xxx]  // 设置对应的json字段名

[raw]         // 解码的时候,该字段不解析,直接保留原始的字符串返回

```c
import json //导入json包

struct User { //定义结构体模板
	name string
	age  int

	// 使用了skip属性,这个字段会被忽略不参与编码和解码
	foo Foo [skip]  

	// 设置对应的json字段名
	last_name string [json:lastName]  
}

data := '{ "name": "Frodo", "lastName": "Baggins", "age": 25 }'

user := json.decode(User, data) or {
	eprintln('Failed to decode json')
	return
}

println(user.name)
println(user.last_name)
println(user.age)
```

另一个例子:

```c
import json

struct User {
	name          string
	age           int
mut:
	is_registered bool
}

fn main() {
	s := '[{ "name":"Frodo", "age":25}, {"name":"Bobby", "age":10}]'
	users := json.decode([]User, s) or {
		eprintln('Failed to parse json')
		return
	}
	for user in users {
		println('$user.name: $user.age')
	}
	println('')
	for i, user in users {
		println('$i) $user.name')
		if !user.can_register() {
			println('Cannot register $user.name, they are too young')
		}
	}
	// Let's encode users again just for fun
	println('')
	println(json.encode(users))
}

fn (u User) can_register() bool {
	return u.age >= 16
}

fn (u mut User) register() {
	u.is_registered = true
}
```

