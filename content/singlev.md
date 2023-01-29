## 单个V文件

如果只是想写一个简单的程序，可以直接省略主模块，主函数的定义，就像脚本文件那样代码从上往下执行，编译器会把这个单文件直接编译成可执行文件。

app.v

```v
fn my_fn() {
	println('from my_fn')
}

println('hello world')
my_fn()
```

编译并运行：

```
v app.v && ./app
```

或者直接运行：

```
v run app.v
```

