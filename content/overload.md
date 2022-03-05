## 运算符重载

为了保持语言的简单性，V只支持有限的运算符重载，支持最常用的四则运算符和比较运算符的重载。重载通过自定义复杂类型的运算符语义，让代码可读性更好。

四则运算符: +,  -,  *,  /,

比较运算符: >,  <, !=, ==, <=, >= 

其中比较运算符，只需要重载小于和等于，其他比较运算符重载不用自己定义，编译器会基于小于和等于自动生成。

为了提高安全性和可维护性，运算符重载有以下限制：

- 重载运算符的函数内部禁止调用其它函数。

- 重载运算符的函数不能修改它们的参数。

- 四则运算符的返回值必须跟结构体类型一致，比较运算符必须返回bool类型。

  

  ```v
  module main
  
  struct Vec {
  	x int
  	y int
  }
  
  //四则运算符
  pub fn (a Vec) + (b Vec) Vec {
  	return Vec{a.x + b.x, a.y + b.y}
  }
  
  pub fn (a Vec) - (b Vec) Vec {
  	return Vec{a.x - b.x, a.y - b.y}
  }
  
  pub fn (a Vec) * (b Vec) Vec {
  	return Vec{a.x * b.x, a.y * b.y}
  }
  
  pub fn (a Vec) / (b Vec) Vec {
  	return Vec{a.x / b.x, a.y / b.y}
  }
  
  //比较运算符,只需要重载<和==，其他比较运算符不用自己定义，编译器会基于<和==自动生成
  fn (a Vec) == (b Vec) bool {
  	return a.x == b.x && a.y == b.y
  }
  
  fn (a Vec) < (b Vec) bool {
		return a.x < b.x && a.y < b.y
  }
  
  fn (a Vec) str() string {
  	return '{$a.x, $a.y}'
  }
  
  fn main() {
  	a := Vec{2, 3}
  	b := Vec{4, 5}
  	//四则运算符
  	println(a + b) // "{6,8}"
  	println(a - b) // "{-2,-2}"
  	println(a * b) // "{8,15}"
  	println(a / b) // "{0,0}"
  	//比较运算符
  	println(a == b) // false
  	println(a!=b)   // true
  	println(a > b)  // false
  	println(a >= b) // false
  	println(a < b)  // true
  	println(a <= b) // true
  }
  ```
  
  完整的运算符重载示例，也可以参考：vlib/gx/color.v。