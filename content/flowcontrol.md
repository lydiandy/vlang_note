## 流程控制

### if 条件语句

```
a := 10
b := 20
if a < b {
	println('$a < $b')
} else if a > b {
	println('$a > $b')
} else {
	println('$a == $b')
}
```

if表达式:

```
num := 777
s := if num % 2 == 0 {
	'even'
}
else {
	'odd'
}
println(s) // "odd"
```



### match 分支/匹配语句

```
os:='macos'
match os {
	'windows' {
    	println('windows')
	}
	'linux' {
    	println('linux')
	}
	'macos' {
    	println('macos')
	}
	else  {
   	 println('unknow')
	}
}
```

match表达式:

```
os:='macos'
price:=match os {
    'windows' {
        100
    }
    'linux' {
        120
    }
    'macos' {
        150
    }
    else {
        0
    }
}
println(price) //输出150
```



### for 循环语句

for的四种形式：

1. 传统的：for i=0;i<100;i++ {}

   ```
   for i := 0; i < 10; i++ { 
   	//跳过6
   	if i == 6 {
   		continue
   	}
   	println(i)
   }
   ```

   为了简洁的目的,for里面的i默认就是mut可变的

2. 替代while：for i<100 {}

   ```
   mut sum := 0
   mut i := 0
   for i <= 100 {
   	sum += i
   	i++
   }
   println(sum) // 输出"5050"
   ```

3. 无限循环：for {}


```
mut num := 0
   for {
   	num++
   	if num >= 10 {
   		break
   	}
   }
   println(num) // "10"
```

4. 遍历：for i in xxx {}

    for in可以用来遍历数组，字符串，字典这三种类型

   遍历数组:

   ```
   numbers := [1, 2, 3, 4, 5]
   for num in numbers {
   	println('i:$i,num:$num')
   }
   ```

​	遍历字符串:

```
str:='abcdef'
//遍历value
for s in str {
    println(s.str())
}
//遍历index和value
for i,s in str {
    println('index:$i,value:${s.str()}')
}
```

​	遍历字典:

```
m:={"name":"jack","age":"20","desc":"good man"}

for key,value in m {
	println('key:$key,value:$value')
}
```



### goto语句

关键字有看到还保留着goto,但是还没有看到具体的使用例子

