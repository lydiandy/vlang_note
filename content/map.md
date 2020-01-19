## 字典

### map实现

从map的源代码定义看,map是通过2个struct实现的

vlib/builtin/map.v

```c
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

```c
fn main() {
    mut m := map[string]int
    m['one'] = 1
    m['two'] = 2
    println(m['one']) //返回对应的value
    println(m['bad_key']) // 如果指定key不存在,返回0
}
```

目前map的key只能是string类型

```c
map[string]int
map[string]User
map[string][]int
...
```

map字面量初始化

```c
m:={'ont':1,'two':2,'three':3}
```

map.size返回字典的大小

```c
fn main() {
    mut m := map[string]int
    m['one'] = 1
    m['two'] = 2
    println(m.size) //返回2
}
```

### map的in操作符

判断某一个元素是否包含在map的key中

```c
fn main() {
    mut m := map[string]int
    m['one'] = 1
    m['two'] = 2
    println('one' in m) //返回true
    println('three' in m) //返回false
}
```

### 遍历map

```c
fn main() {
    mut m := map[string]int
    m['one'] = 1
    m['two'] = 2
    m['three'] = 3
    
    for key,value in m {
        println('key:$key,value:$value')
    }
    for key,_ in m {
        println('key:$key')
    }
    for _,value in m {
            println('value:$value')
        }

}
```

------

### map常用函数

m.keys() 	//获取map的所有key,返回keys数组

m.delete(key)	//删除map的某一个key

m.str()	//map转成字符串输出

m.free()	//释放map的内存

