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
- 使用is关键字联合类型具体是哪一种类型
- 使用as关键字将联合类型转换为另一种类型,当然要转换的类型在联合类型定义的类型范围内

```c
module main

struct User {
	name string
	age  int
}

fn (m &User) str() string {
	return 'name:$m.name,age:$m.age'
}

type MySum = User | int | string // 联合类型声明
fn (ms MySum) str() string {
	if ms is int { // 使用is关键字,判断联合类型具体是哪种类型
		println('ms type is int')
	}
	match ms {// 对接收到的联合类型,使用match语句进行类型判断,每个match分支的ms变量都会被自动造型为分支中对应的类型
		int { 
			return ms.str()
		}
		string {
			return ms
		}
		User {
			return ms.str()
		}
	}
}

fn add(ms MySum) { // 联合类型作为参数
	match ms { // 可以对接收到的联合类型,使用match语句进行类型判断,每个match分支的ms变量都会被自动造型为分支中对应的类型
		int {
			println('ms is int,value is $ms.str()')
		}
		string {
			println('ms is string,value is $ms')
		}
		User {
			println('ms is User,value is $ms.str()')
		}
	}
}
fn add2(ms MySum) { // 联合类型作为参数
	match ms as m { // 可以对接收到的联合类型,使用match语句进行类型判断,增加了as m后,就可以使用自定义的m变量名,作为match分支中造型后的变量
		int { 
			println('ms is int,value is $m.str()')
		}
		string {
			println('ms is string,value is $m')
		}
		User {
			println('ms is User,value is $m.str()')
		}
	}
}
//有了前2种用法,自动生成的it迭代器变量用法会被移除,目前还暂时可用
fn add3(ms MySum) { // 联合类型作为参数
	match ms { // 可以对接收到的联合类型,使用match语句,进行类型判断
		int { // 会在这个代码块中,自动生成一个类型为int,名为it(迭代器iterator的缩写)的变量,可以使用it
			println('ms is int,value is $it.str()')
		}
		string {
			println('ms is string,value is $it')
		}
		User {
			println('ms is User,value is $it.str()')
		}
	}
}


fn sub(i int, s string, u User) MySum { // 联合类型作为返回值
	return i
	// return s //这个也可以
	// return User{name:'tom',age:3} //这个也可以
}

fn main() {
	i := 123
	s := 'abc'
	u := User{
		name: 'tom'
		age: 33
	}
	mut res := MySum{} // 声明联合类型变量
	res = i
	println(res) // 输出123
	res = s
	println(res) // 输出abc
	res = u
	println(res) // 输出name:tom,age:33
	match res { // 判断具体类型
		int { println('res is:$it.str()') }
		string { println('res is:$it') }
		User { println('res is:$it.str()') }
	}
	user := res as User // 也可以通过as,进行显示造型
	println(user.name)
}
```

