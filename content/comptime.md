## 编译时代码

按照V语言作者的说法，为了保持V语言的简单，不会加入像C语言那样的预处理器，而是通过编译时代码来实现类似的功能。

编译时就是在编译阶段，根据编译时代码，动态生成代码。编译时代码使用最多的场景就是动态生成指定平台的代码，其他平台的代码不会生成。

V语言中编译时代码以$开头。

### 条件编译

条件编译是编译时代码最主要的应用场景：根据编译时条件，动态生成指定平台的代码，其他平台的代码不会生成，在C语言中使用了预处理器来实现。

目前的条件编译有2种主要方式：

1.根据源文件名后缀来实现条件编译

2.根据代码中的$if来实现条件编译

#### 按源文件后缀名进行条件编译

源文件后缀包含了2个维度的条件编译：

- 通用源文件

  | 后缀名  | 编译条件                        |
  | ------- | ------------------------------- |
  | 只有 .v | 所有操作系统,所有后端都参与编译 |

- 操作系统(os)

  | 后缀名           | 编译条件                                                     |
  | :--------------- | ------------------------------------------------------------ |
  | _default         | 默认的,表示所有操作系统都参与编译,比如file_default.c.v.同个目录中,如果同时存在默认的和平台特有的,平台特有的文件会参与编译,默认的被忽略 |
  | _nix             | linux,unix,darwin,solaris下才会参与编译,或者说是非windows    |
  | _macos或 _darwin | mac下才会参与编译                                            |
  | _linux           | linux下才会参与编译                                          |
  | _solaris         | solaris下才会参与编译                                        |
  | _windows         | windows下才会参与编译                                        |
  | _android         | android平台下才会参与编译                                    |
  | _ios             | ios平台下才会参与编译                                        |
  | _bare            | 编译成裸机(metal)环境运行代码,才会参与编译,不进行排列组合    |

- 编译器后端(backend)

  | 后缀名  | 编译器后端                       |
  | ------- | -------------------------------- |
  | .c      | C语言为编译器后端时才会参与编译  |
  | .js     | js语言为编译器后端时才会参与编译 |
  | .native | 直接生成x64机器码时才会参与编译  |

  以上2个维度排列组合：

  ```shell
  file.v	#所有操作系统,所有编译后端都参与编译
  file.c.v	#所有操作系统,C编译后端才参与编译,不会被特定平台覆盖,而是都编译
  file.js.v	#所有操作系统,js编译后端才参与编译,不会被特定平台覆盖,而是都编译
  file.native.v #所有操作系统，native编译后端才参与编译
  
  file_default.c.v #同目录如果存在特定平台的C后端文件,此文件会被忽略,不参与编译
  file_linux.c.v
  file_macos.c.v
  file_windows.c.v
  file_windows.js.v
  file_windows.x64.v
  ...
  ```

  正常情况下，在同一个模块中函数是不允许重复定义的，但是可以在.c.v或.js.v重复定义.v已经定义过的函数，.c.v或.js.v的同名函数会优先被执行，也就是覆盖了.v定义的函数。

  这样的好处是可以在.v定义通用版本的函数，在.c.v定义针对C后端的函数，在.js.v定义针对js后端的函数。

  mymodule/wrapper.v

  ```v
  module mymodule
  
  pub fn value() int {
  	return 666
  }
  ```

  mymodule/wrapper.c.v

  ```v
  module mymodule
  
  pub fn value() int {
  	return 123
  }
  ```

  mymodule/wrapper.js.v

  ```v
  module mymodule
  
  pub fn value() int {
  	return 456
  }
  ```

  主模块的main.v

  ```v
  module main
  
  import mymodule
  
  fn main() {
  	println(mymodule.value())	//C编译器后端输出123
  }
  ```

#### 条件编译选项

#### 内置条件编译选项

以下内置的条件编译选项，可以在代码中使用：

| OS                                      | Compilers        | Platforms          | Other                     |
| --------------------------------------- | ---------------- | ------------------ | ------------------------- |
| `windows`, `linux`, `macos`             | `gcc`, `tinyc`   | `amd64`, `aarch64` | `debug`, `prod`, `test`   |
| `mac`, `darwin`, `ios`,                 | `clang`, `mingw` | `x64`, `x32`       | `js`, `glibc`, `prealloc` |
| `android`,`mach`, `dragonfly`，`termux` | `msvc`           | `little_endian`    | `no_bounds_checking`      |
| `gnu`, `hpux`, `haiku`, `qnx`           | `cplusplus`      | `big_endian`       | freestanding              |
| `solaris`, `linux_or_macos`             |                  |                    |                           |

```v
fn main() {
	$if windows {
		println('windows')
		$if msvc { // 可以在windows平台中,进一步判断是msvc还是mingw
		}
		$if mingw { // 可以在windows平台中,进一步判断是msvc还是mingw
		}
	}
	$if linux {
		println('linux')
	}
	$if macos { // 或者mac
		println('mac')
	}
	$if windows {
	} $else $if macos { // else if分支
		println('macos')
	} $else $if linux { // else if分支
		println('linux')
	} $else { // else分支
		println('others')
	}
	$if termux { // 在安卓终端模拟器termux环境中
		println('termux')
	}
	$if !windows { // 使用非运算
	}
	$if linux || macos { // 使用或运算符
	}
	$if linux && x64 { // 使用且运算符
	}
	// 其他条件编译的选项有:
	// freebsd,openbsd,netbsd,bsd,dragonfly,android,solaris
	// js,tinyc,clang,msvc,mingw
}

```

判断是否使用了-cg，进入调试模式

```v
$if debug {
	println('from debug')
}
//执行:
//v run main.v -cg
```

判断是否在测试代码中执行

```v
fn test_comptime_if_test() {
	mut i := 0
	$if test { //如果在测试函数中,则执行
		i++
	}
	$if !test { //非测试函数中
		i--
	}
	assert i == 1
}
```

判断平台是32位还是64位

```v
fn main() {
	mut x := 0
	$if x32 {
		println('system is 32 bit')
		x = 1
	}
	$if x64 {
		println('system is 64 bit')
		x = 2
	}
}
```

判断平台使用的字节序是小字节序，还是大字节序

```v
fn main() {
	mut x := 0
	$if little_endian { //小字节序
		println('system is little endian')
		x = 1
	}
	$if big_endian { //大字节序
		println('system is big endian')
		x = 2
	}
}
```

判断是否-prod生产编译

```v
fn main() {
	$if prod {
		println('prod')
	} $else {
		println('not prod')
	}
}
```

#### 自定义编译选项

除了内置的条件编译选项，也可以识别自定义条件编译选项。

使用-d或-define来自定义编译选项，并且可以在代码中接收选项的传入值。

要特别注意的是：自定义条件编译选项名后面一定要加一个问号。

```v
//main.v
module main

fn main() {
	$if abc ? {  //如果编译时没有传递abc编译选项，这段代码就不会被生成C代码
		println('自定义编译选项abc存在')
	}
}
```

编译时，传递自定义条件编译变量：

```shell
v -d abc main.v
v -define abc main.v
#执行main时，输出：自定义编译选项abc存在
```

同时使用多个自定义编译选项：

```v
module main

fn main() {
	$if time_v ? {
		println('time_v')
	}
	$if abc ? { 
		println('abc')
	}
	$if value ? {
		println('1')
	}
}
```

```shell
v  -d abc -d time_v main.v	#增加自定义编译选项abc和time_v，类型默认为bool类型，默认值为true
v  -d abc -d time_v -d value='0' main.v #也可以使用‘1’或‘0’，等价于true和false
./main 	#输出time_v和abc
```

### 编译时反射

$for用来实现反射的效果，目前只实现了结构体的反射，可以在运行时获得某一个结构体所有字段和方法的信息。

遍历结构体字段,返回字段信息数组：[]FieldData

```v
FieldData {
    name: 'a' 		//字段名称
    attrs: [] 		//字段注解
    is_pub: false //是否公共
    is_mut: false //是否可变
    typ: 18		 	//字段类型
}
```

遍历结构体方法,返回方法信息数组：[]FunctionData

```v
FunctionData {
    name: 'int_method1' //方法名称
    attrs: []						//方法注解
    args: []						//方法参数
    return_type: 7				//方法返回类型
    typ: 0							//未知
}
```

```v
struct App {
	a string
	b string
mut:
	c int
	d f32
pub:
	e f32
	f u64
pub mut:
	g string
	h u8
}

['foo/bar/three']
fn (mut app App) run() {
}

['attr2']
fn (mut app App) method2() {
}

fn (mut app App) int_method1() int {
	return 0
}

fn (mut app App) int_method2() int {
	return 1
}

fn (mut app App) string_arg(x string) {
}

fn no_lines(s string) string {
	return s.replace('\n', ' ')
}

fn f1() {
	println(@FN)
	methods := ['run', 'method2', 'int_method1', 'int_method2', 'string_arg']
	$for method in App.methods { //遍历结构体所有方法
		println('  method: $method.name | ' + no_lines('$method'))
		assert method.name in methods
	}
}

fn f2() {
	println(@FN)
	$for method in App.methods { //遍历结构体所有方法
		println('  method: ' + no_lines('$method'))
		$if method.typ is fn () {
			assert method.name in ['run', 'method2']
		}
		$if method.return_type is int {
			assert method.name in ['int_method1', 'int_method2']
		}
		$if method.args[0].typ is string {
			assert method.name == 'string_arg'
		}
	}
}

fn main() {
	println(@FN)
	$for field in App.fields { //遍历结构体所有字段
		println('  field: $field.name | ' + no_lines('$field'))
		$if field.typ is string {
			assert field.name in ['a', 'b', 'g']
		}
		$if field.typ is f32 {
			assert field.name in ['d', 'e']
		}
		if field.is_mut {
			assert field.name in ['c', 'd', 'g', 'h']
		}
		if field.is_pub {
			assert field.name in ['e', 'f', 'g', 'h']
		}
		if field.is_pub && field.is_mut {
			assert field.name in ['g', 'h']
		}
	}
	f1()
	f2()
}

```

### 编译时动态字段赋值

```v
struct Foo {
	immutable int
mut:
	test string
	name string
}

fn comptime_field_selector_read<T>() []string {
	mut t := T{}
	t.name = '2'
	t.test = '1'
	mut value_list := []string{}
	$for f in T.fields {
		$if f.typ is string {
			value_list << t.$f.name
		}
	}
	return value_list
}

fn test_comptime_field_selector_read() {
	assert comptime_field_selector_read<Foo>() == ['1', '2']
}

fn comptime_field_selector_write<T>() T {
	mut t := T{}
	$for f in T.fields {
		$if f.typ is string {
			t.$f.name = '1'
		}
		$if f.typ is int {
			t.$f.name = 1
		}
	}
	return t
}

fn test_comptime_field_selector_write() {
	res := comptime_field_selector_write<Foo>()
	assert res.immutable == 1
	assert res.test == '1'
	assert res.name == '1'
}

struct Foo2 {
	f Foo
}

fn nested_with_parentheses<T>() T {
	mut t := T{}
	$for f in T.fields {
		$if f.typ is Foo {
			t.$(f.name).test = '1'
		}
	}
	return t
}

fn test_nested_with_parentheses() {
	res := nested_with_parentheses<Foo2>()
	assert res.f.test == '1'
}

```

### 编译时获取结构体字段

可以在$for循环中使用内置函数：sizeof()，isreftype()，typeof(T).name来获取字段大小，字段类型，字段是否为引用类型。

```v
module main

struct App {
	a int
	b string
	c f32
}

fn main() {
	$for field in App.fields {
		println(field.name) //字段名字
		println(sizeof(field)) //字段内存大小
		println(typeof(field).name) //字段类型名字
		println(isreftype(field)) //字段是否引用类型
	}
}
```

### 编译时动态调用方法

```v
struct TestStruct {}

fn (t TestStruct) one_arg(a string) {
	println(a)
}

fn (t TestStruct) two_args(a string, b int) {
	println('$a:$b')
}

fn main() {
	t := TestStruct{}
	$for method in TestStruct.methods { // 获取结构体的所有方法
    	if method.name == 'two_args' {
      	  t.$method('hello', 42) // 动态调用方法
    	}
	}
}

```

```v
struct MyStruct {}

struct TestStruct {}

fn (t TestStruct) three_args(m MyStruct, arg1 string, arg2 int) {
	println('$arg1,$arg2')
}

fn main() {
	t := TestStruct{}
	m := MyStruct{}
	args := ['one' '2'] //数组参数
	$for method in TestStruct.methods {
    	if method.name == 'three_args' {
        	t.$method(m, args) //动态调用方法时,也可以传递一个数组,会自动解构展开
    	}
	}
}

```

### 编译时动态判断泛型类型

可以使用编译时来动态判断泛型的具体类型：

```v
module main

fn kind<T>() {
	$if T is $Int {
		println('Int')
	}
	$if T is $Float {
		println('Float')
	}
	$if T is $Array {
		println('Array')
	}
	$if T is $Map {
		println('Map')
	}
	$if T is $Struct {
		println('Struct')
	}
	$if T is $Interface {
		println('Interface')
	}
	$if T is $Enum {
		println('Enum')
	}
	$if T is $Sumtype {
		println('Sumtype')
	}
}

struct Abc {}

interface Def {}

enum Xyz {
	x
	y
	z
}

type Sumtype = Abc | f32 | int

struct GenericStruct<T> {
	x T
}

pub fn (s GenericStruct<T>) m() {
	$if T is $Int {
		println('Int in method')
	}
	
	$if T is $Array {
		println('Array in method')
	} $else {
		println('other')
	}
}

fn main() {
	// int
	kind<i8>()
	kind<i16>()
	kind<int>()
	kind<i64>()
	// int
	kind<u8>()
	kind<u16>()
	kind<u32>()
	kind<u64>()
	// float
	kind<f32>()
	kind<f64>()
	// array
	kind<[]int>()
	// map
	kind<map[string]string>()
	// struct
	kind<Abc>()
	// interface
	kind<Def>()
	// enum
	kind<Xyz>()
	// sumtype
	kind<Sumtype>()
	// generic struct
	s1 := GenericStruct<int>{}
	s1.m()
	s2 := GenericStruct<[]string>{}
	s2.m()
	s3 := GenericStruct<f32>{}
	s3.m()
}
```

### 编译时全局变量

内置了开发和测试时需要的几个编译时全局变量，方便编译和测试使用：

```v
module main

fn main() {
	println('module: ${@MOD}')					//当前模块
	println('fn: ${@FN}')						//当前函数
	println('sturct: ${@STRUCT}')				//当前结构体
	println('method: ${@METHOD}')				//当前方法
  
	println('file: ${@FILE}')					//当前源代码文件名
	println('line: ${@LINE}')					//当前代码所在的行
	println('column: ${@COLUMN}')				//当前代码在当前行中的列数
  
	println('vhash: ${@VHASH}')					//当前V编译器的hash版本号	
 	println('vexe: ${@VEXE}')					//当前V编译器命令行文件
	println('vexeroot: ${@VEXEROOT}')			//当前V编译器命令行所在的目录
	
	println('vmod_file: ${@VMOD_FILE}')	//当前文件所处项目的v.mod文件内容
	println('vmodroot: ${@VMODROOT}')		//当前文件所处项目的v.mod文件所在的目录
}
```

### 编译时获取环境变量

可以使用$env编译时函数获取环境变量，

运行时代码中os.get_env函数也可以实现相同的效果，

比较特别的是$env也可以在#flay和#include等C宏中使用,让C宏的定义更灵活。

```v
module main

//可以在C宏语句中使用,让C宏的定义更灵活
#flag linux -I $env('JAVA_HOME')/include

fn main() {
	compile_time_env := $env('PATH')
	println(compile_time_env)
}

```

### 编译时获取pkgconfig配置文件

可以使用$pkgconfig编译时函数，来判断配置文件是否存在。

具体代码参考：[集成C代码库章节](c.md)。

### 编译时嵌入静态文件

可以使用$embed_file编译时函数，把各种类型的文件在编译时嵌入到二进制文件中，更方便编译打包成单一可执行文件，方便部署，目前vweb框架中有使用到。

如果没有特别指定，使用-prod进行生产编译时，$embed_file函数会使用zlib对嵌入二进制的静态文件进行压缩。

```v
module main

import os

fn main() {
	mut embedded_file := $embed_file('v.png',.zlib) //可以使用zlib进行压缩
	mut fw := os.create('exported.png') or { panic(err) }
  //data函数返回字节数据，len是字节长度
	unsafe { fw.write_ptr(embedded_file.data(), embedded_file.len) }
	fw.close()
}
```

### 编译时模板渲染

V语言内置了一个简单的txt和html模板，可以通过$tmpl编译时函数,进行渲染。

```v
fn build() string {
	name := 'Peter'
	age := 25
	numbers := [1, 2, 3]
	return $tmpl('a.txt') //读取模板文件,然后把变量替换到模板中对应的同名变量
}

fn main() {
	println(build())
}
```

a.txt 模板文件内容

```text
name: @name //@开头的都是模板文件中需要替换的同名变量

age: @age

numbers: @numbers

@for number in numbers //也可以遍历数组类型变量
  @number
@end
```

### 编译时解析v.mod文件

在编译阶段，如果需要解析v.mod的文件内容，可以导入v.mod模块，解析v.mod文件的内容。

```v
import v.vmod
vm := vmod.decode( @VMOD_FILE ) or { panic(err) }
eprintln('$vm.name $vm.version\n $vm.description')
```

### 编译时错误与警告

V语言内置2个编译时函数：compile_error和compile_warn，专门用来抛出编译时的错误和警告。

```v
module main

fn main() {
	$if !linux {
   		 $compile_warn('这是编译时警告!') //抛出编译警告，会继续编译,执行
	}
	println('警告后代码')

	$if !macos {
		$compile_error('这是个编译时错误！') //抛出编译错误，终止编译,执行
	}
}
```

