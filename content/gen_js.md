## 编译生成js代码

V语言的开发重点在编译器前端,目前主要的编译器后端有3个:C/js/x64

js作为事实上的web前端汇编语言,已经有各种语言支持编译生成js代码

### 生成js代码

使用-o参数就可以

把当前目录的main.v代码生成main.js代码,

最简单的V代码:

```v
module main

fn main() {
	println('from main')
}
```

编译生成js代码:

```shell
v -o main.js ./main.v 
```

生成的js代码可以通过node运行

```shell
node ./main.js
```

### 基本类型对应



### 代码对照表





