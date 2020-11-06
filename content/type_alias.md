##类型别名(type alias)

可以在某一个类型的基础上定义类型别名,使用上完全一样

### 基于基本类型

```v
type Myint = int

type Myf32 = f32

type Myf64 = f64

fn main() {
	i := Myint(10)
	println(i + 100 == 110)
	f := Myf64(1.0)
	println(f + 3.14 == 4.14)
}

```

### 基于结构体类型

```v
module main

struct Human {
	name string
}

pub fn (h Human) str() string {
	return 'Human: $h.name'
}

type Person = Human

pub fn (h Person) str() string {
	return 'Person: $h.name'
}

fn main() {
	p := Person{'Bilbo'}
	p2 := Human{'jack'}
	println(p)
	println(p2)
}

```

### 类型别名方法

可以像结构体那样,给类型别名添加方法

```v
module main

fn main() {
	i := Myint(11)
	println(i.str())
}

type Myint = int

pub fn (m Myint) str() string { // 类型别名的方法
	return 'from myint'
}

```

