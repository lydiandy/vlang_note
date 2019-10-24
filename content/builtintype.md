## 内置类型

除了内置的基本类型外,数组和字典也是内置类型

### 数组array

**数组实现:**

从数组的源代码实现看，也是一个struct实现

vlib/builtin/array.v

```
struct array {
pub:
	data voidptr //数组的起始指针
	len int //数组的长度
	cap int //数组的容量
	element_size int //数组元素的字节大小
}
```

**数组定义:**

根据首个元素的类型来决定数组的类型

[1, 2, 3]的数组类型是[]int

['a', 'b']的数组类型是[]string

数组元素的类型必须是一样的,也就是不允许异构数组

[1, 'a'] 编译不会通过



定义一个指定长度,指定默认值的数组:

```
arr := [0].repeat(50) //元素初始值为0,长度为30
println(arr.len) //返回50
```

**数组的追加运算符: <<**

可以把一个元素追加到数组中,也可以把一个数组追加到数组中

```
mut nums := [1, 2, 3]
println(nums) // "[1, 2, 3]"
nums << 4 //把一个元素追加到数组中
println(nums) // "[1, 2, 3, 4]"
```

```
mut nums := [1, 2, 3, 4]
println(nums) // "[1, 2, 3, 4]"
nums << [5, 6, 7] //把一个数组追加到数组中
println(nums) // "[1, 2, 3, 4, 5, 6, 7]"
```

**in操作符:**

判断一个元素是否在数组里面

```
mut names := ['John']
names << 'Peter'
names << 'Sam'
println('Alex' in names) // "false"
```

**遍历数组:**

```
numbers := [1, 2, 3, 4, 5]
for num in numbers {
	println('i:$i,num:$num')
}
```

**多维数组:**

维度的数量不仅仅于1,2维,不限

```
module main

fn main() {
    a := [1, 2, 3]
    b := [4, 5, 6]
    c := [7, 8, 9]
    d := [[a, b, c], [b, c, a], [c, a, b]]
    println(d)
}
```

****

**数组常用函数:**

str()  	//数组转字符串

first() 	//返回数组的第一个元素

last()	//返回数组的最后一个元素

delete(int)	//删除数组的第几个元素

left(int) array	//返回从左边开始,到第几个元素的子数组

right(int) array	//返回从左边开始第几个之后,  右边的所有元素的子数组

slice(start,end)	//返回给定位置区间的子数组,左闭右开

reverse()	//数组反转

clone()	//克隆数组

insert(int,voidptr)	//在数组的第几个位置插入新的元素,第二个参数是指针类型

prepend(voidptr)	//在数组的第一个位置插入新的元素

free()	//释放数组的内存

[ ]int.sort() 	//针对整型数组的排序

filter()	//针对int和string数组进行过滤,返回满足条件的元素数组

​	it是参数表达式中,约定的iterator迭代器,表示每一次迭代时,数组的元素,满足过滤器表达式的元素会被返回

```
	a := [1, 2, 3, 4, 5, 6]
	b := a.filter(it % 2 == 0) //b的结果为:[2,4,6]
	c := ['v', 'is', 'awesome']
	d := c.filter(it.len > 1) //d的结果为:['is','awesome']
```

filter2()	//针对int和string数组进行过滤,返回满足条件的元素数组

​	int类型数组中:filter2参数为函数类型,函数签名为: fn(p_val, p_i int, p_arr []int) bool

​	string类型数组中:filter2参数为函数类型,函数签名为:fn(p_val string, p_i int, p_arr []string) bool

```
fn callback_1(val int, index int, arr []int) bool {
	return val >= 2
}

fn callback_2(val string, index int, arr []string) bool {
	return val.len >= 2
}

fn test_filter2() {
	a := [1, 2, 3, 4, 5, 6]
	b := a.filter2(callback_1) //b的结果为:[2,4,6]

	c := ['v', 'is', 'awesome']
	d := c.filter2(callback_2) //d的结果为:['is','awesome']
}
```



------



### 字典map

**map实现:**

从map的源代码定义看,map是通过2个struct来实现的

vlib/builtin/map.v

```go
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
	key string //键
	value voidptr //值的地址
}
```

**map定义:**

```
fn main() {
    mut m := map[string]int
    m['one'] = 1
    m['two'] = 2
    println(m['one']) //返回对应的value
    println(m['bad_key']) // 如果指定key不存在,返回0
}
```

目前map的key只能是string类型

```
map[string]int
map[string]User
map[string][]int
...
```

map字面量初始化

```
m:={'ont':1,'two':2,'three':3}
```

map.size返回字典的大小

```
fn main() {
    mut m := map[string]int
    m['one'] = 1
    m['two'] = 2
    println(m.size) //返回2
}
```

**map的in操作符:**

判断某一个元素是否包含在map的key中

```
fn main() {
    mut m := map[string]int
    m['one'] = 1
    m['two'] = 2
    println('one' in m) //返回true
    println('three' in m) //返回false
}
```

**遍历map:**

```
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

**map常用函数:**

m.keys() 	//获取map的所有key,返回keys数组

m.delete(key)	//删除map的某一个key

m.str()	//map转成字符串输出

m.free()	//释放map的内存

