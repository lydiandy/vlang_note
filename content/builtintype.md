## 内置类型

除了内置的基本类型外,数组和字典也是内置类型

内置类型相关的源代码可以参考v源代码中的： vlib/builtin目录

### 数组array

**数组实现:**

从数组的源代码实现看，也是一个struct

vlib/builtin/array.v

```c
struct array {
pub:
	data voidptr //数组的起始指针
	len int //数组的长度
	cap int //数组的容量
	element_size int //数组元素的字节大小
}
```

```c
arr := [1,2,3,4,5]  //字面量定义数组
println(arr.data) //返回数组地址:0x7ff0d2c01270
println(arr.len)  //返回5
println(arr.cap)  //返回5
println(arr.element_size) //返回4
```

**数组定义:**

根据首个元素的类型来决定数组的类型

[1, 2, 3]的数组类型是[]int

['a', 'b']的数组类型是[]string

数组元素的类型必须是一样的,不允许异构数组

[1, 'a'] 编译不会通过

**定义固定长度的数组,然后赋值:**

不过挺奇怪的是这种固定长度定义的数组,无法像字面量定义的那样,可以使用arr.len,无法使用<<追加元素

```c
fn main() {
	mut arr:=[8]int //定义长度固定的数组,所有数组元素默认都是0值初始化
	println(arr[1]) //返回0
	arr=[0,1,2,3,4,5,6,7]!!  //注意数组后面有2个!!,否则会报错
	println(arr[1]) //赋值成功后,返回1
	println(arr[2]) //赋值成功后,返回2

	x := 2.32
	mut v := [8]f32
	println(v[1]) //返回0.000000
	v = [1.0, x, 3.0,4.0,5.0,6.0,7.0,8.0]!! 
	println(v[1]) //赋值成功后,返回2.320000
}
```

定义一个指定长度,指定默认值的数组:

```c
arr := [0].repeat(50) //元素初始值为0,长度为50,容量为50
println(arr.len) //返回50
println(arr.cap) //返回50
```

```c
arr := ['a','b'].repeat(3) //元素初始值为a,b,重复3次,长度为6,容量为6
println(arr.len) //返回6
println(arr.cap) //返回6
println(arr) //返回['a','b','a','b','a','b']
```

数组的追加运算符: <<

可以把一个元素追加到数组中,也可以把一个数组追加到数组中

```c
mut nums := [1, 2, 3]
println(nums) // "[1, 2, 3]"
nums << 4 //把一个元素追加到数组中
println(nums) // "[1, 2, 3, 4]"
```

```c
mut nums := [1, 2, 3, 4]
println(nums) // "[1, 2, 3, 4]"
nums << [5, 6, 7] //把一个数组追加到数组中
println(nums) // "[1, 2, 3, 4, 5, 6, 7]"
```

指针类型数组:

```c
mut arr := []&int
	a := 1
	b := 2
	c := 3
	arr << &a
	arr << &b
	arr << &c
  for i in arr {
    println(i) //输出数组的3个地址元素
  }
```

in操作符:**

判断一个元素是否在数组里面

```c
mut names := ['John']
names << 'Peter'
names << 'Sam'
println('Alex' in names) // "false"
```

**遍历数组:**

```c
numbers := [1, 2, 3, 4, 5]
for num in numbers {
	println('i:$i,num:$num')
}
```

**数组切片/区间:**

左闭右开原则

```c
n := [1,2,3,4,5]
println(n)
println(n[..2])  //返回[1, 2]
println(n[2..])  //返回[3, 4, 5]
println(n[2..4]) //返回[3, 4]
```

**多维数组:**

维度的数量不仅仅于1,2维,不限

```c
module main

fn main() {
  	a := [1, 2, 3]
    b := [4, 5, 6]
    c := [7, 8, 9]
    d := [[a, b, c], [b, c, a], [c, a, b]]
    println(d) 
    println(d[0].len) //返回3
    println(d[0][0].len) //返回3
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

------

filter()	//针对int和string数组进行过滤,返回满足条件的元素数组

​	filter函数有点特殊,是在编译器中实现的,而不是builtin库中,因为有it这个特殊的迭代器参数

​	it是参数表达式中,约定的iterator迭代器,表示每一次迭代时,数组的元素,满足过滤器表达式的元素会被返回

```c
	a := [1, 2, 3, 4, 5, 6]
	b := a.filter(it % 2 == 0) //b的结果为:[2,4,6]
	c := ['v', 'is', 'awesome']
	d := c.filter(it.len > 1) //d的结果为:['is','awesome']
```

------

map() 	//针对int和string数组的每一个元素进行一个运算,返回运算后的新数组

map函数有点特殊,是在编译器中实现的,而不是builtin库中,因为有it这个特殊的迭代器参数

​	it是参数表达式中,约定的iterator迭代器,表示每一次迭代时,数组的元素

```c
a := [1, 2, 3, 4, 5, 6]
b := a.map(it * 10)
println(b)
```

------

reduce(iter fn (accum, curr int) int, accum_start int) int	//针对int数组,给定一个初始的累计值accum_start,以及累计值与数组元素的累加关系,返回最终的累加结果

```c
module main

fn sum(accum int, curr int) int {
	return accum + curr
}

fn sub(accum int, curr int) int {
	return accum - curr
}

fn main() {
	a := [1, 2, 3, 4, 5]
	b := a.reduce(sum, 0)
	c := a.reduce(sum, 5)
	d := a.reduce(sum, -1)
	println(b) //返回15
	println(c) //返回20
	println(d) //返回14
	e := [1, 2, 3]
    f := e.reduce(sub, 0)
    g := e.reduce(sub, -1)
    println(f) //返回-6
    println(g) //返回-7
}
```



------



### 字典map

**map实现:**

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

**map定义:**

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

**map的in操作符:**

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

**遍历map:**

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

**map常用函数:**

m.keys() 	//获取map的所有key,返回keys数组

m.delete(key)	//删除map的某一个key

m.str()	//map转成字符串输出

m.free()	//释放map的内存

