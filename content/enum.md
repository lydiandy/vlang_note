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

枚举值的名称限制必须是小写加下划线

