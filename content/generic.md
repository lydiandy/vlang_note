## 泛型

目前的泛型主要有这三种：泛型结构体,泛型函数,泛型方法

### 泛型结构体

```v
struct DB {
    driver string
}

struct User {
	db DB
mut:
	name string
}

struct Repo<T> {
	db DB
mut:
	model  T
}

fn new_repo<U>(db DB) Repo<U> {
	return Repo<U>{db: db}
}

fn test_generic_struct() {
	mut a :=  new_repo<User>(DB{})
	a.model.name = 'joe'
	mut b := Repo<User>{db: DB{}}
	b.model.name = 'joe'
	assert a.model.name == 'joe'
	assert b.model.name == 'joe'
}
```

### 泛型函数

判断2个数组是否相等的泛型函数

```v
// vlib/builtin/array.v
fn array_eq<T>(a1 []T, a2 []T) bool {
	if a1.len != a2.len {
		return false
	}
	for i := 0; i < a1.len; i++ {
		if a1[i] != a2[i] {
			return false
		}
	}
	return true
}

```

### 泛型方法

```v
struct Point {
mut:
    x int
    y int
}

fn (mut p Point) translate<T>(x T, y T) {
    p.x += x
    p.y += y
}

fn main() {
    mut p := Point{}
    p.translate(1,3)
}
```

