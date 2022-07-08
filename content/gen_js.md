## 编译生成js代码

V语言的开发重点在编译器前端，目前主要的编译器后端有4个：C/x64/js/go。

js作为事实上的web前端汇编语言，已经有各种语言支持编译生成js代码，而且生成的js代码质量都不错，比如typescript，rescript等。

总感觉V还是集中精力定位在服务端，定位为"better C"就已经很好了，web前端不太可能采用V来生成js。

### 生成js代码

使用-o参数就可以

把当前目录的main.v代码生成main.js代码

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

V的每一个基本类型都对应一个js同名函数

```js
// builtin type casts
const [i8, i16, int, i64, u8, u16, u32, u64, f32, f64, int_literal, float_literal, size_t, bool, string, map, array] = [
	function(val) { return new builtin.i8(val) },
	function(val) { return new builtin.i16(val) },
	function(val) { return new builtin.int(val) },
	function(val) { return new builtin.i64(val) },
	function(val) { return new builtin.u8(val) },
	function(val) { return new builtin.u16(val) },
	function(val) { return new builtin.u32(val) },
	function(val) { return new builtin.u64(val) },
	function(val) { return new builtin.f32(val) },
	function(val) { return new builtin.f64(val) },
	function(val) { return new builtin.int_literal(val) },
	function(val) { return new builtin.float_literal(val) },
	function(val) { return new builtin.size_t(val) },
	function(val) { return new builtin.bool(val) },
	function(val) { return new builtin.string(val) },
	function(val) { return new builtin.map(val) },
	function(val) { return new builtin.array(val) }
]
```

### 代码对照表

#### 常量



#### 枚举



#### 模块



#### 函数



#### 数组



#### 字符串



#### 字典



#### 结构体



#### 结构体方法



#### 结构体访问控制



#### 流程控制语句



#### 类型定义



#### 接口



#### 泛型



#### 错误处理



#### 联合类型



#### 运算符重载



#### 条件编译







