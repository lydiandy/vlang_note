## 枚举

### 定义枚举

```
enum Color {
	blue
	green
	white
	black
}
fn main() {
    mut c:=Color.green
    println(c) //目前只能输出枚举值1,还没有办法返回枚举值的名称

    c=Color.blue
    println(c) //输出0
}
```

可以指定枚举值的值

```
enum Color {
	blue=3
	green
	white
	black
}
fn main() {
    mut c:=Color.green
    println(c) //目前只能输出枚举值1,还没有办法返回枚举值的名称

    c=Color.blue
    println(c) //输出3
}
```

