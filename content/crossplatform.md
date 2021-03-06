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

  | 后缀名 | 编译器后端                       |
  | ------ | -------------------------------- |
  | .c     | C语言为编译器后端时才会参与编译  |
  | .js    | js语言为编译器后端时才会参与编译 |
  | .x64   | 直接生成x64机器码时才会参与编译  |

  以上2个维度排列组合:

  ```shell
  file.v	//所有操作系统,所有编译后端都参与编译
  file.c.v	//所有操作系统,C编译后端才参与编译,不会被特定平台覆盖,而是都编译
  file.js.v	//所有操作系统,js编译后端才参与编译,不会被特定平台覆盖,而是都编译
  
  file_default.c.v //同目录如果存在特定平台的C后端文件,此文件会被忽略,不参与编译
  file_linux.c.v
  file_macos.c.v
  file_windows.c.v
  file_windows.js.v
  file_windows.x64.v
  ...
  ```

### $if条件编译

以下表格是所有可以在编译时进行条件判断的变量:

| OS                            | Compilers        | Platforms          | Other                     |
| ----------------------------- | ---------------- | ------------------ | ------------------------- |
| `windows`, `linux`, `macos`   | `gcc`, `tinyc`   | `amd64`, `aarch64` | `debug`, `prod`, `test`   |
| `mac`, `darwin`, `ios`,       | `clang`, `mingw` | `x64`, `x32`       | `js`, `glibc`, `prealloc` |
| `android`,`mach`, `dragonfly` | `msvc`           | `little_endian`    | `no_bounds_checking`      |
| `gnu`, `hpux`, `haiku`, `qnx` | `cplusplus`      | `big_endian`       |                           |
| `solaris`, `linux_or_macos`   |                  |                    |                           |

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

识别自定义编译选项

```v
$if abc ? { //abc是自定义的编译选项,在条件编译时可以判断是否时候了自定义选项
		println('自定义选项abc存在')
	} 
	//执行 v -d abc
	//或者 v -define abc
```

判断是否使用了-cg,进入调试模式

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

------

## 跨平台交叉编译

编译器有一个选项-os用来编译生成指定平台的可执行文件

目前可以是:linux, mac, windows, msvc

```shell
v -os linux ./main.v
```

## 内置全局变量

编译器中内置了开发和测试时需要的几个全局变量,方便编译,测试使用:

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

另外,如果需要进一步解析v.mod的文件内容,可以导入v.mod模块,这个模块就是用来解析v.mod的

```v
import v.vmod
vm := vmod.decode( @VMOD_FILE ) or { panic(err) }
eprintln('$vm.name $vm.version\n $vm.description')
```

