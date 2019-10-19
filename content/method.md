## 方法

跟go一样的方法,在函数名前面加接收者

接收者也是默认不可变,如果要修改接收者,也要加上mut

结构体的方法,可以定义在同一个模块目录的不同的源文件中

```
struct User {
	name string
	age int
	
}

fn (u User) getName() string {
	return u.name
}

fn (u mut User) setName(name string) {
	u.name=name
}
```

