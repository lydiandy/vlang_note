## V scripts

V语言还可以用来写系统shell脚本,借助简洁的语法,写shell脚本还是比较舒服的,而且还可以是跨平台的

V脚本的文件名后缀为 .vsh

区别于.v文件,在.vsh中:

- 不用定义主模块

- 不用定义主函数

- 不用导入os模块,调用os模块函数时,可以省略os前缀,就像使用内置函数那样

直接像shell脚本那样写,代码从头开始运行

script.vsh

```
for _ in 0..5 {
  println('V script')
}

println('deploying...')
println(ls('.'))
println('')
mv('v.exe', 'bin/v.exe')
rm('tmp.c')

mkdir('name')
create('foo.txt')
println(ls('.'))
println('')

println('Removing name and foo.txt')
println('')
rmdir('name')
rm('foo.txt')

println(ls('.'))
```

编译,运行:

```
v script.vsh && ./script
```

或者直接运行:

```
v run script.vsh
```

具体的os模块常用的函数可以参考[标准库](stdlibrary.md)章节的介绍