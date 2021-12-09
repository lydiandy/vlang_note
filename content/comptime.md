## 编译时代码

编译时就是编译器生成C代码时，会根据编译时代码，生成指定平台的C代码,其他平台的C代码不会生成。

有别于运行时代码,不管是哪个平台的代码都会生成对应的C代码。

在V语言中所有的编译时代码都是以$开头的。

### 动态获取字段和方法

$for用来实现反射的效果,目前只实现了结构体的反射,可以在运行时获得某一个结构体所有字段和方法的信息。

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
	h byte
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

### 动态字段赋值

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

### 编译时判断字段类型/内存大小/是否引用类型

可以在$for循环中使用内置函数：sizeof()，isreftype()，type(T).name来判断字段大小，字段类型，字段是否为引用类型。

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

### 动态调用方法

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

### 编译时预置的全局变量

```v
module main

fn main() {
	println('module: ${@MOD}')			//当前模块
	println('fn: ${@FN}')				//当前函数
	println('sturct: ${@STRUCT}')		//当前结构体
	println('method: ${@METHOD}')		//当前方法
	println('vexe: ${@VEXE}')			//当前V编译器命令行可执行文件
	println('vexeroot: ${@VEXEROOT}')	//当前V编译器命令行所在的目录
	println('file: ${@FILE}')			//当前源代码文件名
	println('line: ${@LINE}')			//当前代码所在的行
	println('column: ${@COLUMN}')		//当前代码在当前行中的列数
	println('vhash: ${@VHASH}')			//当前V命令行编译时的hash	
	println('vmod_file: ${@VMOD_FILE}')	//当前文件所处项目的v.mod文件内容
	println('vmodroot: ${@VMODROOT}')	//当前文件所处项目的v.mod文件所在的目录
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

### 编译时判断pkgconfig配置文件是否存在

可以使用$pkgconfig编译时函数来判断pc配置文件是否存在，

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

### 编译模板

V内置了一个简单的txt和html模板，可以通过$tmpl编译时函数,进行渲染。

```v
fn build() string {
	name := 'Peter'
	age := 25
	numbers := [1, 2, 3]
	return $tmpl('1.txt') //读取模板文件,然后把变量替换到模板中对应的同名变量
}

fn main() {
	println(build())
}
```

1.txt模板文件内容

```
name: @name //@开头的都是模板文件中需要替换的同名变量

age: @age

numbers: @numbers

@for number in numbers //也可以遍历数组类型变量
  @number
@end
```
