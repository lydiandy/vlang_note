## 条件编译

目前主要针对不同平台,实现条件编译

按照作者的说法,为了保持V的简单,不会加入预处理的部分,通过这个来实现条件编译

```
$if windows {
	println('windows')
}
$if linux {
	println('linux')
}
$if mac {
	println('mac')
}
```

另一个用途,判读是否使用了-debug

```c
$if debug {
	println('from debug')
}
执行:
v run main.v -debug
```

源文件跨平台的编译:

源文件后缀为 _nix 的表示linux,unix,mac下才会编译,

后缀为 _win的表示在windows下才会编译

------



## 跨平台交叉编译

