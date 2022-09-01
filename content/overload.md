## 运算符重载

为了保持语言的简单性，V只支持有限的运算符重载，支持最常用的四则运算符，比较运算符，分配运算符的重载。重载通过自定义复杂类型的运算符语义，让代码可读性更好。

四则运算符：+，-，*，/，%

比较运算符：>,  <, !=, ==, <=, >= 

分配运算符：+=，-=，*=，/=

- 比较运算符重载不用自己定义，编译器会基于小于和等于自动生成。
- 分配运算符重载不用自己定义，编译器会基于四则运算符自动生成。

为了提高安全性和可维护性，运算符重载有以下限制：

- 重载运算符的函数内部禁止调用其它函数。

- 重载运算符的函数不能修改它们的参数。

- 运算符两边的参数类型必须是同一类型。

- 四则运算符的返回值必须跟结构体类型一致，比较运算符必须返回bool类型。

目前支持对：字典，结构体，泛型结构体，类型别名，进行运算符重载。

###结构体运算符重载

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
  
  fn (a Vec) % (b Vec) Vec {
  	return Vec{a.x % b.x, a.y % b.y}
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
  	mut a := Vec{8, 15}
  	b := Vec{4, 5}
  	//四则运算符
  	println(a + b) // {12,20}
  	println(a - b) // {4,10}
  	println(a * b) // {32,75}
  	println(a / b) // {2,3}
  	println(a % b) // {0,0}
  	//分配运算符,分配运算符不用自己定义,会基于四则运算符自动生成
  	a += b
  	println('a+=b is: $a')
  	a -= b
  	println('a-=b is: $a')
  	a *= b
  	println('a*=b is: $a')
  	a /= b
  	println('a/=b is: $a')
  	//比较运算符
  	println(a == b) // false
  	println(a != b) // true
  	println(a > b) // true
  	println(a >= b) // true
  	println(a < b) // false
  	println(a <= b) // false
  }
  
  ```

### 泛型结构体运算符重载

```v
module main

struct Matrix<T> {
	row int
	col int
mut:
	data [][]T
}

fn from_array<T>(arr [][]T) Matrix<T> {
	return Matrix<T>{
		row: arr.len
		col: arr[0].len
		data: arr.clone()
	}
}

fn (m1 Matrix<T>) + (m2 Matrix<T>) Matrix<T> { //对泛型结构体进行运算符重载
	if m1.row != m2.row || m1.col != m2.col {
		panic('Addition can only be performed on matrix with same size')
	}
	mut res := m1
	for i in 0 .. m2.row {
		for j in 0 .. m2.col {
			res.data[i][j] += m2.data[i][j]
		}
	}
	return res
}

fn (m1 Matrix<T>) == (m2 Matrix<T>) bool {
	return m1.row == m2.row && m1.col == m2.col && m1.data == m2.data
}

fn (m1 Matrix<T>) < (m2 Matrix<T>) bool {
	return m1.row < m2.row && m1.col < m2.col
}

fn main() {
	mut a1 := from_array([[1, 2, 3], [4, 5, 6]])
	a2 := from_array([[7, 8, 9], [10, 11, 12]])

	plus_ret := a1 + a2
	println(plus_ret)
	assert plus_ret.row == 2
	assert plus_ret.col == 3
	assert plus_ret.data == [[8, 10, 12], [14, 16, 18]]

	a1 += a2
	println(a1)
	assert a1.row == 2
	assert a1.col == 3
	assert a1.data == [[15, 18, 21], [24, 27, 30]]

	eq_ret := a1 == a2
	println(eq_ret)
	assert !eq_ret

	ne_ret := a1 != a2
	println(ne_ret)
	assert ne_ret

	lt_ret := a1 < a2
	println(lt_ret)
	assert !lt_ret

	le_ret := a1 <= a2
	println(le_ret)
	assert le_ret

	gt_ret := a1 > a2
	println(gt_ret)
	assert !gt_ret

	ge_ret := a1 >= a2
	println(ge_ret)
	assert ge_ret
}

```

### 类型别名运算符重载

  除了可以对结构体定义运算符重载，还可以对类型别名进行运算符重载。

```v
module main

type Vector = []f64 //类型别名

[heap]
struct HeapArray {
mut:
	data []f64
}

fn new_vector(x f64, y f64, z f64, w f64) &Vector {
	a := HeapArray{
		data: [x, y, z, w]
	}
	return &Vector(&a.data)
}

pub fn (a &Vector) + (b &Vector) &Vector { //对类型别名进行运算符重载
	mut res := HeapArray{
		data: []f64{len: a.len}
	}
	for i := 0; i < a.len; i++ {
		res.data[i] = (*a)[i] + (*b)[i]
	}
	return &Vector(&res.data)
}

fn main() {
	mut a := new_vector(12, 4.5, 6.7, 6)
	b := new_vector(12, 4.5, 6.7, 6)
	dump(a)
	dump(b)
	c := a + b
	dump(c)
}

```

如果结构体定义了运算符重载，类型别名也可以直接使用

  ```v
  module main
  
  pub struct Vector {
  	vec []f64
  }
  
  pub fn (a Vector) + (b Vector) Vector {
  	size := a.vec.len
  	if size != b.vec.len {
  		panic('unequal sizes')
  	}
  	mut c := []f64{len: size}
  	for i in 0 .. size {
  		c[i] = a.vec[i] + b.vec[i]
  	}
  	return Vector{
  		vec: c
  	}
  }
  
  type Vec = Vector //结构体定义了运算符重载，类型别名也可以使用
  
  fn main() {
  	a := Vec{
  		vec: [0.1, 0.2]
  	}
  	b := Vec{
  		vec: [0.3, 0.2]
  	}
  	c := a + b
  	println(c)
  }
  ```
