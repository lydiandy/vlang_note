## 运算符重载

有限的运算符重载

目前只实现了+ - * / 这四种运算符的重载,但是基本够用了

可以自定义复杂类型的加减乘除的语义,让自定义类型加减乘除的代码可读性会非常好

为了提高安全性和可维护性，重载有以下限制：

- 只有`+, -, *, /`可以重载

- 重载运算符的函数内部禁止调用其它函数

- 重载运算符的函数不能修改它们的参数

- 参数和返回值必须有相同的类型（和V中的所有运算符一样）

  

  ```
  struct Vec {
  	x int
  	y int
  }
  
  fn (a Vec) str() string {
  	return '{$a.x, $a.y}'
  }
  
  fn (a Vec) + (b Vec) Vec {
  	return Vec {
  		a.x + b.x,
  		a.y + b.y
  	}
  }
  
  fn (a Vec) - (b Vec) Vec {
  	return Vec {
  		a.x - b.x,
  		a.y - b.y
  	}
  }
  
  fn main() {
  	a := Vec{2, 3}
  	b := Vec{4, 5}
  	println(a + b) // "{6, 8}" 
  	println(a - b) // "{-2, -2}" 
  }
  ```

  