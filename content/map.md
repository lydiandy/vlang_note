## 字典

除了内置的基本类型外,数组和字典也是内置类型

### map实现

从map的源代码定义看,map是通过2个struct实现的

vlib/builtin/map.v

```v
struct map {
	root &mapnode //字典第一个键值对的地址
	element_size int //一个键值对元素占用的内存大小
pub:
	size int //字典的大小,也就是键值对的数量,这是map唯一可以对外直接访问的属性,只读
}
struct mapnode {  //键值对节点
	left &mapnode 
	right &mapnode
	is_empty bool
	key string //键,目前只能是string类型
	value voidptr //值的地址,值可以是任何类型
}
```

### map定义

```v
fn main() {
  	mut m := map[string]int{}
    m['one'] = 1
    m['two'] = 2
    println(m['one']) //返回对应的value
    println(m['bad_key']) // 如果指定key不存在,返回0
}
```

map的key除了string类型,也可以是其他类型

string类型的key

```v
map[string]int
map[string]User
map[string][]int
```

非string类型的key

```v
module main

fn main() {
	//非字符串key
	// int key
	mut m1 := map[int]int{}
	m1[3] = 9
	m1[4] = 16
	println(m1)
	// voidptr key
	mut m2 := map[voidptr]string{}
	v := 5
	m2[&v] = 'var'
	m2[&m2] = 'map'
	println(m2)
	// rune key
	mut m3 := {
		`!`: 2
		`%`: 3
	}
	println(typeof(m3).name) // map[rune]int
	println(m3)
	// byte key
	mut m4:=map[byte]string{}
	m4[byte(1)]='a'
	m4[byte(2)]='b'
	println(m4)
}

```

map字面量初始化

```v
m:={'one':1,'two':2,'three':3}
```

map.len返回字典的大小

```v
fn main() {
	mut m := map[string]int{}
	m['one'] = 1
	m['two'] = 2
	println(m.len) // 返回2
}
```

### in操作符

判断某一个元素是否包含在map的key中

```v
fn main() {
    mut m := map[string]int{}
    m['one'] = 1
    m['two'] = 2
    println('one' in m) //返回true
    println('three' in m) //返回false
}
```

### 遍历map

```v
fn main() {
	mut m := map[string]int{}
	m['one'] = 1
	m['two'] = 2
	m['three'] = 3
	for key, value in m {
		println('key:$key,value:$value')
	}
	for key, _ in m {
		println('key:$key')
	}
	for _, value in m {
		println('value:$value')
	}
}
```

------

字典相关的源代码可以参考v源代码中的：vlib/builtin/map.v

更多字典相关的函数,参考[标准库章节](./std_builtin.md)

