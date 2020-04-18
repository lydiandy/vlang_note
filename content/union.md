# 联合类型

联合类型:union types或sum types

### 定义联合类型

语法类似typescript，使用type 和 | 来定义一个联合类型

```c
 //定义联合类型,表示类型Expr可以是这几种类型的其中一种
type Expr = Foo | BoolExpr |  BinExpr | UnaryExpr
```

使用pub关键字,定义公共的联合类型

```c
pub type Expr = Foo | BoolExpr |  BinExpr | UnaryExpr
```

### 使用联合类型

- 联合类型作为函数的参数或返回值,也可以作为变量声明,结构体字段
- 使用match语句,进行类型的进一步判断

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
	match ms {
		int { //会在这个代码块中,自动生成一个类型为int,名为it的变量,可以直接使用
			return it.str()
		}
		string {
			return it
		}
		User {
			return it.str()
		}
		else {
			return 'unknown'
		}
	}
}

pub fn add(ms MySum) { //联合类型作为参数
	match ms { //可以对接收到的联合类型,使用match语句,进行类型判断

		int { //会在这个代码块中,自动生成一个类型为int,名为it的变量,可以直接使用
			println('ms is int,value is $it.str()')	
		}
		string {
			println('ms is string,value is $it')	
		}
		User {
			println('ms is User,value is $it.str()')	
		}
		else {
			println('unknown')
		}
	}
}
pub fn sub(i int,s string,u User) MySum { //联合类型作为返回值
	return i
	// return s //这个也可以
	// return User{name:'tom',age:3} //这个也可以
}

fn main() {
	i:=123
	s:='abc'
	u:=User{name:'tom',age:33}
	var res:=MySum{} //声明联合类型变量
	res=i
	println(res) //输出123
	res=s
	println(res) //输出abc
	res=u
	println(res) //输出name:tom,age:33
	match res { //判断具体类型
		int {
			println('res is:$it.str()')
		}
		string {
			println('res is:$it')
		}
		User {
			println('res is:$it.str()')
		}
		else {
			println('unknown')
		}
	}
	user:=res as User //也可以通过as,进行显示造型
	println(user.name)
}

```

