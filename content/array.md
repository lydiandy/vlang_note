## 数组

除了内置的基本类型外,数组和字典也是内置类型

### 数组实现

从数组的源代码实现看，也是一个struct

vlib/builtin/array.v

```c
struct array {
pub: //公共只读
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

### 数组定义

根据首个元素的类型来决定数组的类型

[1, 2, 3]的数组类型是[]int

['a', 'b']的数组类型是[]string

数组元素的类型必须是一样的,不允许异构数组,[1, 'a'] 编译不会通过

定义固定长度的数组,然后赋值:

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

### in操作符

判断一个元素是否在数组里面

```c
mut names := ['John']
names << 'Peter'
names << 'Sam'
println('Alex' in names) // "false"
```

### 遍历数组

```c
numbers := [1, 2, 3, 4, 5]
for num in numbers {
	println('num:$num')
}
for i,num in numbers {
	println('i:$i,num:$num')
}
```

### 数组切片/区间

左闭右开原则

```c
n := [1,2,3,4,5]
println(n)
println(n[..2])  //返回[1, 2]
println(n[2..])  //返回[3, 4, 5]
println(n[2..4]) //返回[3, 4]
```

### 多维数组

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

------

数组相关的源代码可以参考v源代码中的：vlib/builtin/array.v

更多数组相关的函数,参考[标准库章节](std_builtin.md)