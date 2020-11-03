## 流程控制

### if条件语句

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

条件赋值(if表达式)

```c
num := 777
// 简单的条件赋值
s := if num % 2 == 0 { 'even' } else { 'odd' }
println(s)
// "odd"
// 多条件赋值
a, b, c := if true { 1, 'awesome', 13 } else { 0, 'bad', 0 }
println(a)
println(b)
println(c)

```



### match分支语句

match要求穷尽所有可能,所以基本都要带上else语句

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

match赋值(match表达式)

```rust
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
//多变量match赋值
a,b,c := match false {
		true { 1,2,3 }
		false { 4,5,6 }
		else { 7,8,9 }
}
println(a)
println(b)
println(c)
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
		int { 
			return ms.str()
		}
		string { 
			return ms //ms的类型是string
		}
		User { 
			return ms.str() //ms的类型是User
		}
		// else { //如果之前的分支已经穷尽了所有可能,else语句不需要,如果没有穷尽所有可能,则else语句是必须的
		// 	return 'unknown'
		// }
	}
}
```



### for 循环语句

for的四种形式：

1. 传统的：for i=0;i<100;i++ {}

```c
for i := 0; i < 10; i++ {
	// 跳过6
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
println(sum)
// 输出"5050"

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
println(num)
// "10"

```

4. 遍历：for i in xxx {}

    for in可以用来遍历字符串,数组,区间,字典这四种类型
    

遍历字符串:

```go
str := 'abcdef'
// 遍历value
for s in str {
	println(s.str())
}
// 遍历index和value
for i, s in str {
	println('index:$i,value:$s.str()')
}

```

遍历数组:

```c
numbers := [1, 2, 3, 4, 5]
for num in numbers {
	println('num:$num')
}
for i, num in numbers {
	println('i:$i,num:$num')
}
// 或者这种区间的写法也可以
for i in 0 .. numbers.len {
	println('num:${numbers[i]}')
}

```

遍历区间:

```c
mut sum := 0
for i in 1 .. 11 { // 左闭右开,遍历区间
	sum += i
}
println(sum)	// 55

```

遍历字典:

```c
m:={"name":"jack","age":"20","desc":"good man"}

for key,value in m {
	println('key:$key,value:$value')
}
```

### for select语句

for select语句主要在并发中使用,用来循环监听多个chanel,更多内容可以参考[并发章节](concurrent.md)

```go
fn main() {
	ch1 := chan int{}
	ch2 := chan f64{}
	go do_send(ch1, ch2)
	mut a := 0
	mut b := 0
	for select { // 循环监听channel的写入,写入后执行for代码块,直到所有监听的channel都已关闭
		x := <-ch1 {
			a += x
		}
		y := <-ch2 {
			a += int(y)
		}
	} { // for代码块
		b++ // 写入3次
		println('${b}. event')
	}
	println(a)
	println(b)
}

fn do_send(ch1 chan int, ch2 chan f64) {
	ch1 <- 3
	ch2 <- 5.0
	ch2.close()
	ch1 <- 2
	ch1.close()
}
```

### goto语句

goto语句只能在函数内部跳转

```go
fn main() {
	mut i := 0
	a:	//定义跳转标签
	i++

	// a: i++ //标签和语句在同一行也可以正常运行
	if i < 3 {
		goto a //跳转到a标签
	}
	println(i)
}
```

