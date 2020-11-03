## 访问控制

### 模块一级成员访问控制

目前模块的一级元素有6个:const常量,enum枚举,fn函数/方法,struct结构体,interface接口,type类型

默认是模块级别,只有在模块内部才能访问

加了pub以后,就是公共级别

```c
pub const ( // 公共常量
	pi = 3.14
)

pub enum Color { // 公共枚举
	blue
	green
	red
}

pub fn my_fn() { // 公共函数
	println('my_fn is public')
}

pub struct Point { // 公共结构体
	x int
	y int
}

pub fn (mut p Point) move(x int, y int) { // 公共方法
	p.x += x
	p.u += y
}

pub interface MyReader { // 公共接口
	read() int
}

pub type myint = int  // 公共类型别名

```

在主模块中,所有一级元素都不能加pub,编译器会检查报错,因为主模块作为顶级模块,不能被其他模块导入,加pub没有意义



### 结构体字段访问控制

结构体字段默认是:私有且不可变

pub可以变为公有

mut可以变为可变

有以下4种常用的组合,以及1种不推荐使用的全局字段:__global

```c
struct Foo {
	a int // 私有,不可变(默认).在模块内部可访问,不可修改;模块外不可访问,不可修改
mut:
	b int // 私有,可变.在模块内部可访问,可修改,模块外部不可访问,不可修改
	c int // (相同访问控制的字段可以放在一起)
pub:
	d int // 公共,不可变,只读.在模块内部和外部都可以访问,但是不可修改
pub mut:
	e int // 公共,模块内部可访问,可修改;模块外部可访问,但是不可修改
__global:
	f int // 全局字段,模块内部和外部都可访问,可修改,这样等于破坏了封装性,不推荐使用
}
          
```

看看string结构体的定义:

```c
struct string {
pub:
	str byteptr  //都是公共,不可变
	len int 		//都是公共,不可变
}
```

所以以下代码会报错:

```c
fn main() {
	str := 'hello'
	len := str.len //len公共可访问
	str.len++      // 不可变,不可修改,尝试修改会编译报错
}
```

