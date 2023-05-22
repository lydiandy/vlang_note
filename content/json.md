## 内置json支持

v标准库的json模块有点特别：

1. 基于C语言的cJSON库实现。

2. 没有使用运行时反射,性能会更好。

3. 解析JSON功能在编译器内部实现，内置支持JSON。

### 编码

```v
json.encode(object) string   
```

参数object是某个结构体类型的变量，如果编码成功,则返回对应的json字符串，如果编码失败，则返回空对象。

### 解码

```v
 json.decode(Type,s) !Type   
```

第一个参数是结构体类型，作为模板，第二个参数是要解码的json字符串。如果解码成功，返回结构体类型的变量。如果解码失败，抛出错误。

### 结构体注解

可以在结构体的字段上增加注解：

[skip]          //忽略这个字段不解析

[json:xxx]  // 设置对应的json字段名

[raw]         // 解码的时候,该字段不解析，直接保留原始的字符串返回

```v
module main

import json   // 导入json包

struct User { // 定义结构体模板
	age           int
	nums          []int
	last_name     string	[json:lastName] // 设置对应的json字段名
	is_registered bool 		[json:IsRegistered] // 设置对应的json字段名
	typ           int 		[json:'type'] //如果json中的字段是V关键字,需要加''
	skip_filed    string 	[skip] // 使用了skip注解,这个字段会被忽略不参与编码和解码
}

struct Color { // 定义结构体模板
	space string
	point string [raw] // 解码时,该字段保留原始节点的字符串
}

fn main() {
	s := '{"age": 10, "nums": [1,2,3], "type": 0, "lastName": "Johnson", "IsRegistered": true}'
	u := json.decode(User,s) or {
		exit(1)
	}
	println(u.age == 10)
	println(u.last_name == 'Johnson')
	println(u.is_registered == true)
	println(u.nums.len == 3)
	println(u.nums[0] == 1)
	println(u.nums[1] == 2)
	println(u.nums[2] == 3)
	println(u.typ == 0)
	usr := User{
		age: 10
		nums: [1, 2, 3]
		last_name: 'Johnson'
		is_registered: true
		typ: 0
	}
	expected := '{"age":10,"nums":[1,2,3],"lastName":"Johnson","IsRegistered":true,"type":0}'
	out := json.encode(usr)
	println(out == expected)

	color := json.decode(Color,'{"space": "YCbCr", "point": {"Y": 123}}') or {
		println('text')
		return
	}
	println(color.point == '{"Y":123}')
	println(color.space == 'YCbCr')

	obj:=[1,3,5]
	res:=json.encode(obj) //如果对象不是结构体类型,则返回变量值
	println(res)
}


```
