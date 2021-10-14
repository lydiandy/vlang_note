## 字典

除了内置的基本类型外，数组和字典也是内置类型。

### map实现

从map的源代码定义看，map是通过2个struct实现的。

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

map的key除了string类型，也可以是其他类型。

string类型的key：

```v
map[string]int
map[string]User
map[string][]int
```

非string类型的key：

```v
module main

enum Token {
	aa = 2
	bb
	cc
}

fn main() {
	//非字符串key
	// int key 整数
	mut m1 := map[int]int{}
	m1[3] = 9
	m1[4] = 16
	println(m1)
	// voidptr key 通用指针
	mut m2 := map[voidptr]string{}
	v := 5
	m2[&v] = 'var'
	m2[&m2] = 'map'
	println(m2)
	// rune key unicode码
	mut m3 := {
		`!`: 2 //是反引号`,不是单引号'
		`%`: 3
	}
	println(typeof(m3).name) // map[rune]int
	println(m3)
	// byte key 字节
	mut m4 := map[byte]string{}
	m4[byte(1)] = 'a'
	m4[byte(2)] = 'b'
	println(m4)
	// float key 小数
	mut m5 := map[f64]string{}
	m5[1.2] = 'a'
	m5[2.0] = 'b'
	println(m5)
	// enum key 枚举值
	mut m6 := map[Token]string{}
	m6[Token.aa] = 'abc'
	m6[Token.bb] = 'def'
	println(m6)
}

```

map字面量初始化：

```v
fn main() {
	m := map {'one':1,'two':2,'three':3}
	m2 := map {1 :'a', 2 :'b', 3 :'c'}
	println(m)
	println(m2)
}
```

map.len返回字典的大小：

```v
fn main() {
	mut m := map[string]int{}
	m['one'] = 1
	m['two'] = 2
	println(m.len) // 返回2
}
```

### in操作符

判断某一个元素是否包含在map的key中：

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

### 访问字典成员错误处理

```v
module main

fn main() {
	sm := {
		'abc': 'xyz'
	}
	val := sm['bad_key']
	println(val) // 如果字典元素不存在,会返回该类型的默认值:空字符串
	intm := {
		1: 1234
		2: 5678
	}
	s := intm[3]
	println(s) // 如果字典元素不存在,会返回该类型的默认值:0
	//也可以加上or代码块来进行错误处理
	mm := map[string]int{}
	val2 := mm['bad_key'] or { panic('key not found') }
	println(val2)
	myfn()
}

fn myfn() ? {
	mm := map[string]int{}
	x := mm['bad_key']? //也可以本层级不处理,向上抛转错误
	println(x)
}
```

### if条件语句判断字典成员是否存在

```v
fn main() {
	mut m := {'xy': 5, 'zu': 7}
	mut res := []int{cap:2}
	for k in ['jk', 'zu'] {
    //检查字典m[k]是否存在,如果存在,则赋值,if条件返回true,如果不存在,则返回false
		if x := m[k] { 
			res << x
		} else {
			res << -17
		}
	}
	println(res) //[-17,7]
}
```



字典相关的源代码可以参考v源代码中的：vlib/builtin/map.v。

更多字典相关的函数，参考[标准库章节](./std_builtin.md)。

