## V shell script

V语言还可以用来写系统shell脚本，比如开发脚本，构建脚本等。

用V语言来写shell脚本还是比较舒服的，不仅语法简洁，而且是跨平台的。

V脚本的文件名后缀为 .vsh，跟.v源文件相比在.vsh中：

- 不用定义主模块
- 不用定义主函数
- 不用导入os模块，可以直接调用os模块函数，省略os前缀，就像内置函数
- 跟shell脚本一样，可以在首行中使用#!来设置执行脚本的工具

script.vsh

```v
#! /Users/zhijiayou02/v/src/v

for _ in 0 .. 5 {
	println('V script')
}
println('deploying...')
println('Files')
foo := ls('.') or { panic(err) }
println(foo)
println('')
rm('a.out') or { panic(err) }
println('Making dir name and creating foo.txt')
mkdir('name')!
create('foo.txt')!
foo_ls := ls('.') or { panic(err) }
println(foo_ls)
println('')
println('Entering into name')
chdir('name') or { panic(err) }
foo_ls2 := ls('.') or { panic(err) }
println(foo_ls2)
println('')
println('Removing name and foo.txt')
println('')
chdir('../') or { panic(err) }
rmdir('name') or { panic(err) }
rm('foo.txt') or { panic(err) }
again := ls('.') or { panic(err) }
println(again)

```

直接运行：

```shell
v script.vsh	#执行脚本，默认是使用crun子命令，可以省略
./script.vsh #执行脚本，如果脚本首行有设置指定v命令行来执行，可以直接运行
v run script.vsh #执行脚本，先自动编译生成可执行文件，然后执行，执行完删除
v crun script.vsh #执行脚本，先自动编译生成可执行文件，然后执行，执行完保留可执行文件，如果下次再执行，脚本源代码没有改动，则跳过编译，直接运行可执行文件，这样运行速度更快
```

以下是[vls](https://github.com/vlang/vls)的构建脚本：

```v
#!/usr/local/bin/v  //跟shell脚本一样，可以使用#!来设置执行此脚本的工具，也是要提前设置为可执行：chmod +x

import os

// use system default C compiler if found
mut cc := 'cc'
if os.args.len >= 2 {
	if os.args[1] in ['cc', 'gcc', 'clang', 'msvc'] {
		cc = os.args[1]
	} else {
		println('> Usage error: parameter must be in cc/gcc/clang/msvc')
		return
	}
}
println('> Building VLS...')

ret := system('v -gc boehm -cg -cc $cc cmd/vls -o vls')	//可以直接使用os模块内部的函数，就像内置函数那样，不用模块前缀
if ret != 0 {
	println('Failed building VLS')
	return
}

println('> VLS built successfully!')

```

os模块函数可以参考[标准库os模块](std_os.md)章节。
