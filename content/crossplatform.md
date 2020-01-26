## 条件编译

目前主要针对不同平台,实现条件编译

按照作者的说法,为了保持V的简单,不会加入预处理,但是增加这个来实现条件编译

目前的条件编译有2种主要方式:

1.根据源文件名后缀来进行条件编译

2.根据代码中的$if来实现条件编译 

### 源文件后缀名方式

后缀为 _nix 的源文件,表示linux,unix,mac下才会编译

后缀为_darwin的源文件,表示在mac下才会编译

后缀为_linux的源文件,表示在linux下才会编译

后缀为 _windows的源文件,表示在windows下才会编译

### 源代码中的$if方式

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
$if macos {
	println('mac')
}

$if windows {
    
} $else { //用来处理$if以外,其他平台的代码
    
}
$if !windows { //还可以使用!,取否
    
}
```

判断是否使用了-debug

```c
$if debug {
	println('from debug')
}
//执行:
//v run main.v -debug
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

------

## 跨平台交叉编译

编译器有一个选项-os用来编译生成指定平台的可执行文件

目前可以是:linux, mac, windows, msvc

```c
v -os linux ./main.v
```

