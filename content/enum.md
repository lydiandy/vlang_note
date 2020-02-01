## 枚举

### 定义枚举

枚举默认是模块内访问，通过pub关键字来定义公共枚举

```c
pub enum Color {
	blue 			//如果没有指定初始值，默认从0开始，然后往下递增1
	green
	white
	black
}
fn main() {
    mut c:=Color.green //第一次定义要使用：枚举名称.枚举值
    println(c) //目前只能输出枚举值1,还没有办法返回枚举值的名称

    c=.blue //第二次修改赋值，直接使用.枚举值就可以了
    println(c) //输出0
}
```

也可以指定枚举值的值，枚举值也可以是负数

```c
enum Color2 {
	blue =2 //可以指定初始值
	green
	white
	black
}
enum Color3 {
	blue =-4  //初始值也可以是负数
	green
	white
	black
}
fn main() {
    mut c:=Color.green
    println(c) //输出4

    c=.blue
    println(c) //输出3
}
```

也可以指定枚举的值为16进制

```c
enum w_hex {
	a = 0x001 //枚举值也支持16进制
	b = 0x010
	c = 0x100
}
fn main() {
	println(w_hex.a) //输出1
	println(w_hex.b) //输出16
	println(w_hex.c) //输出256
}
```

枚举值的名称限制必须是小写加下划线

为枚举添加方法:

```c
enum Color {
	red=1
	green
	blue
	black
	white
}

fn (c Color) is_blue() bool { //枚举方法
	return c==.blue 
}

fn main(){
	b:=Color.blue
	if b.is_blue() {
		println('yes')
		println(b)
	} else {
		println('no')
		println(b)
	}
}
```

