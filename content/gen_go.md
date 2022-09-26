## 编译生成go代码

V语言的开发重点在编译器前端，目前主要的编译器后端有4个：C/native/js/go。

生成go代码目前还是实验性质的，还只能生成一些基本的代码。

目前源文件名需要命名为：xxx.go.v

```v
module main

#import "fmt"

fn println(x string){
	#fmt.Println(x)
}

fn main() {
	println('hello golang')
}
```

然后终端执行：

```shell
v -b go ./main.go.v
#或者
v -b go -o main.go main.v
```

