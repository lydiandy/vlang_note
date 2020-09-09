## 条件编译

目前主要针对不同平台,实现条件编译

按照作者的说法,为了保持V的简单,不会加入预处理,但是支持条件编译

目前的条件编译有2种主要方式:

1.根据源文件名后缀来实现条件编译

2.根据代码中的$if来实现条件编译 

### 源文件后缀名

源文件后缀包含了2个维度的条件编译:

- 通用源文件

  | 后缀名  | 编译条件                        |
  | ------- | ------------------------------- |
  | 只有 .v | 所有操作系统,所有后端都参与编译 |

- 操作系统(os)

  | 后缀名   | 编译条件                                                  |
  | :------- | --------------------------------------------------------- |
  | _nix     | linux,unix,mac,solaris下才会参与编译,或者说是非windows    |
  | _darwin  | mac下才会参与编译                                         |
  | _linux   | linux下才会参与编译                                       |
  | _solaris | solaris下才会参与编译                                     |
  | _windows | windows下才会参与编译                                     |
  | _bare    | 编译成裸机(metal)环境运行代码,才会参与编译,不进行排列组合 |

- 编译器后端(backend)

  | 后缀名 | 编译器后端                       |
  | ------ | -------------------------------- |
  | .c     | C语言为编译器后端时才会参与编译  |
  | .js    | js语言为编译器后端时才会参与编译 |
  | .x64   | 直接生成x64机器码时才会参与编译  |

  以上2个维度排列组合:

  ```
  file.v
  file_linux.c.v
  file_windows.c.v
  file_windows.js.v
  ...
  ```

  

### $if条件编译

```c
$if windows {
	println('windows')
    $if msvc { //可以在windows平台中,进一步判断是msvc还是mingw
        
    }
    $if mingw { //可以在windows平台中,进一步判断是msvc还是mingw
        
    }
}
$if linux {
	println('linux')
}
$if macos {  //或者mac
	println('mac')
}

$if windows {
    
} $else { //用来处理$if以外,其他平台的代码
    
}
$if !windows { //还可以使用!,取否
    
}
//其他条件编译的选项有:
//freebsd,openbsd,netbsd,bsd,dragonfly,android,solaris
//js,tinyc,clang,msvc,mingw
```

判断是否使用了-debug

```c
$if debug {
	println('from debug')
}
//执行:
//v run main.v -debug
```

判断是否在测试代码中执行

```c
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

```c
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

```c
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

### $for 编译时反射

$for用来实现反射的效果,目前只实现了结构体的反射,可以在运行时获得某一个结构体所有字段和方法的信息

遍历结构体字段,返回字段信息数组:[]FieldData

```c
FieldData {
    name: 'a' 		//字段名称
    attrs: [] 		//字段标注
    is_pub: false //是否公共
    is_mut: false //是否可变
    Type: 18		 	//字段类型
}
```

遍历结构体方法,返回方法信息数组:[]FunctionData

```c
FunctionData {
    name: 'int_method1' //方法名称
    attrs: []						//方法标注
    args: []						//方法参数
    ReturnType: 7				//方法返回类型
    Type: 0							//未知
}
```



```rust
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

fn no_lines(s string) string { return s.replace('\n', ' ') }

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
		$if method.Type is fn() {
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



------

## 跨平台交叉编译

编译器有一个选项-os用来编译生成指定平台的可执行文件

目前可以是:linux, mac, windows, msvc

```c
v -os linux ./main.v
```



## 内置全局变量

编译器中内置了开发和测试时需要的几个全局变量,方便编译,测试使用:

```c
module main

fn main() {
	println(@MOD) 		// 当前模块main
	println(@FN) 		// 当前函数
	println(@STRUCT) 	// 当前结构体
	println(@VEXE) 		// 编译器的当前位置
	println(@FILE) 		// 当前源文件
	println(@LINE) 		// 当前行数
	println(@COLUMN) 	// 当前列数
	println(@VHASH)		// 当前V编译器的vhash号
	println(@VMOD_FILE) // 当前v.mod文件内容,以字符串形式返回.执行前确保存在v.mod文件,否则会编译报错
}
```

另外,如果需要进一步解析v.mod的文件内容,可以导入v.mod模块,这个模块就是用来解析v.mod的

```c
import v.vmod
vm := vmod.decode( @VMOD_FILE ) or { panic(err) }
eprintln('$vm.name $vm.version\n $vm.description')
```

