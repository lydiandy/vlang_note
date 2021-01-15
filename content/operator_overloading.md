## 运算符重载

有限的运算符重载

目前实现了四则运算符和比较运算符的重载

四则运算符: +,  -,  *,  /,

比较运算符: >,  <, !=, ==, <=, >= 

可以自定义复杂类型的运算符语义,让自定义类型的代码可读性会非常好

为了提高安全性和可维护性，重载有以下限制：

- 重载运算符的函数内部禁止调用其它函数

- 重载运算符的函数不能修改它们的参数

- 四则运算符的返回值必须跟结构体类型一致,比较运算符必须返回bool类型

  

  ```v
  module main
  
  struct Vec {
  	x int
  	y int
  }
  
  //四则运算符
  fn (a Vec) str() string {
  	return '{$a.x, $a.y}'
  }
  
  fn (a Vec) + (b Vec) Vec {
  	return Vec{a.x + b.x, a.y + b.y}
  }
  
  fn (a Vec) - (b Vec) Vec {
  	return Vec{a.x - b.x, a.y - b.y}
  }
  
  //比较运算符
  fn (a Vec) == (b Vec) bool {
  	return a.x == b.x && a.y == b.y
  }
  
  fn (a Vec) > (b Vec) bool {
  	return a.x > b.x && a.y > b.y
  }
  
  fn (a Vec) < (b Vec) bool {
	return a.x < b.x && a.y < b.y
  }
  
  fn main() {
  	a := Vec{2, 3}
  	b := Vec{4, 5}
  	println(a + b) // "{6, 8}" 
  	println(a - b) // "{-2, -2}" 
  	println(a == b) // false
  	println(a > b) // false
  	println(a < b) // true
  }
  
  ```
  
  