## 访问控制

### 模块一级成员访问控制

目前模块的一级成员有7个:const,enum,fn,struct,method,interface,type

默认是模块级别,只有在模块内部才能访问

加了pub以后,就是公共级别

```
pub const ( //公共常量
	pi=3.14
)
pub enum Color { //公共枚举
	blue
	green
	red
}
pub fn my_fn() { //公共函数
	println('my_fn is public')
}
pub struct Point { //公共结构体
	x int
	y int
}
pub fn (p mut Point) move(x,y int) {  //公共方法
	p.x+=x
	p.u+=y
}
pub interface MyReader { //公共接口
	read() int
}
pub type myint int //公共类型别名

```



### 结构体字段访问控制

结构体字段默认是:私有且不可变

pub可以变为公有

mut可以变为可变

有以下5种组合:

```
struct Foo {
	a int     //私有,不可变(默认).在模块内部可以访问,不可修改;模块外不可访问,不可修改
mut: 
	b int     // 私有,可变.在模块内部可以访问,可以修改,模块外部不可访问,不可修改
	c int     // (相同访问控制的字段可以放在一起)   
pub: 
	d int   // 公共,不可变,只读.在模块内部和外部都可以访问,但是不可修改
pub mut: 
	e int  //公共,模块内可变.在模块内部可以访问,可以修改;模块外部可以访问,但是不可修改
pub mut mut:  //2个mut
	f int 	  // 模块内部和外部都可以访问,可以修改,这样等于破坏了封装性,不推荐使用,所以语法才							//会啰嗦一些
}             
```

以上代码实际是不能运行的,只是汇总在一起方便说明

因为同一个结构体内部只能有一个pub,或者只能有一个mut,相同访问控制的字段要放在一起分组定义

看看string结构体的定义:

```
struct string {
pub:
	str byteptr  //都是公共,不可变
	len int 		//都是公共,不可变
}
```

所以以下代码会报错:

```
fn main() {
	str := 'hello'
	len := str.len //len公共可访问
	str.len++      // 不可变,不可修改,尝试修改会编译报错
}
```

