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

```c
struct Foo {}

struct BoolExpr {
	foo int
}

struct BinExpr {

}

struct UnaryExpr {

}

pub type Expr = Foo | BoolExpr |  BinExpr | UnaryExpr //定义联合类型

fn expr1() Expr {
	mut e := Expr{} //定义变量e为联合类型Expr
	e = BinExpr{} //可以联合类型中的其中一种具体类型赋值
	return e
}

fn expr2() Expr { //联合类型作为函数返回值
	return BinExpr{}
}

fn handle_expr(e Expr) { //联合类型作为函数参数

}

fn parse_bool() BoolExpr {
	return BoolExpr{}
}

fn main() {
	b := parse_bool()
	handle_expr(b)
}

```

可以使用match语句来判断,类型具体是联合类型中的哪一个

```c
match sumtype {
	Foo {
	
	}
	BoolExpr {
	
	}
	else {
	
	}

}
```

