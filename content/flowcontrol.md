## 流程控制

### if 条件语句

```c
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

```c
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

match要求包括所有可能,所以基本都要带上else语句

```c
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

匹配的值也可以多个,用逗号分隔:

```c
os:='macos'
match os {
	'windows' {
    	println('windows')
	}
	'macos','linux' {
    	println('macos or linux')
	}
	else  {
   	 println('unknow')
	}
}
```

match表达式:

```c
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

match的同时,加上mut ,可以修改匹配变量,通常是配合for in 语句结合使用

```go
//参考代码
	for stmt in file.stmts {
			match mut stmt {
				ast.ConstDecl {
					c.stmt(*it)
				}
				else {}
			}
		}
```

使用match判断联合类型的具体类型

```c
module main

struct User {
	name string
	age int
}
pub fn (m &User) str() string {
	return 'name:$m.name,age:$m.age'
}

type MySum= int|string|User //联合类型声明

pub fn (ms MySum) str() string {
	match ms { //如果函数的参数或者接收者是联合类型,可以使用match进一步判断类型
		int { //会在这个代码块中,自动生成一个类型为int,名为it的变量,可以直接使用
			return it.str()
		}
		string { 
			return it //it的类型是string
		}
		User { 
			return it.str() //it的类型是User
		}
		else {
			return 'unknown'
		}
	}
}
```



### for 循环语句

for的四种形式：

1. 传统的：for i=0;i<100;i++ {}

```c
   for i := 0; i < 10; i++ { 
   	//跳过6
   	if i == 6 {
   		continue
   	}
   	println(i)
   }
```

   为了简洁的目的,for里面的i默认就是mut可变的,不需要特别声明为mut,如果声明了编译器会报错

2. 替代while：for i<100 {}

```c
   mut sum := 0
   mut i := 0
   for i <= 100 {
   	sum += i
   	i++
   }
   println(sum) // 输出"5050"
```

3. 无限循环：for {}


```c
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

    for in可以用来遍历字符串,数组,字典这三种类型
    

遍历字符串:

```go
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

遍历数组:

```c
numbers := [1, 2, 3, 4, 5]
for num in numbers {
	println('num:$num')
}
for i,num in numbers {
	println('i:$i,num:$num')
}
//或者这种区间的写法也可以
for i in 0..numbers.len {
	println('num:${numbers[i]}')
}
```

遍历字典:

```c
m:={"name":"jack","age":"20","desc":"good man"}

for key,value in m {
	println('key:$key,value:$value')
}
```

###  

goto语句

关键字有看到还保留着goto,但是还没有看到具体的使用例子

