## 编译时反射

### 动态获取字段和方法

$for用来实现反射的效果,目前只实现了结构体的反射,可以在运行时获得某一个结构体所有字段和方法的信息

遍历结构体字段,返回字段信息数组:[]FieldData

```v
FieldData {
    name: 'a' 		//字段名称
    attrs: [] 		//字段标注
    is_pub: false //是否公共
    is_mut: false //是否可变
    Type: 18		 	//字段类型
}
```

遍历结构体方法,返回方法信息数组:[]FunctionData

```v
FunctionData {
    name: 'int_method1' //方法名称
    attrs: []						//方法标注
    args: []						//方法参数
    ReturnType: 7				//方法返回类型
    Type: 0							//未知
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
		$if method.Type is fn () {
			assert method.name in ['run', 'method2']
		}
		$if method.ReturnType is int {
			assert method.name in ['int_method1', 'int_method2']
		}
		$if method.args[0].Type is string {
			assert method.name == 'string_arg'
		}
	}
}

fn main() {
	println(@FN)
	$for field in App.fields { //遍历结构体所有字段
		println('  field: $field.name | ' + no_lines('$field'))
		$if field.Type is string {
			assert field.name in ['a', 'b', 'g']
		}
		$if field.Type is f32 {
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

### 动态调用方法

```v

```



## 编译时嵌入文件

可以使用$embed_file编译时函数,把各种类型的文件在编译时嵌入到二进制文件中,更方便编译打包成单一可执行文件,方便部署,目前vweb框架中有使用到.

```v
module main

import os

fn main() {
	mut embedded_file := $embed_file('v.png')
	mut fw := os.create('exported.png') or { panic(err) }
	fw.write_bytes(embedded_file.data(), embedded_file.len)
	fw.close()
}
```

