## 条件编译

目前主要针对不同平台,实现条件编译

按照作者的说法,为了保持V的简单,不会加入预处理的部分,但是增加这个来实现条件编译

判断不同的操作系统平台

```c
$if windows {
	println('windows')
}
$if linux {
	println('linux')
}
$if macos {
	println('mac')
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
	$if little_endian {
		println('system is little endian')
		x = 1
	}
	$if big_endian {
		println('system is big endian')
		x = 2
	}
}
```



## 源文件跨平台的编译

后缀为 _nix 的表示linux,unix,mac下才会编译

后缀为_darwin的在mac下才会编译

后缀为_linux的在linux下才会编译

后缀为 _windows的表示在windows下才会编译

------



## 跨平台交叉编译

