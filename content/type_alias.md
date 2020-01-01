## type alias 类型别名

可以在某一个类型的基础上，定义类型别名

### 基于基本类型-定义类型别名

```c
type Myint int
type Myf32 f32
type Myf64 f64

fn test_type_alias() {
	i := Myint(10) 
	assert i + 100 == 110

	f := Myf32(1.0)
	assert f + 3.14 == 4.14

}
```



### 基于结构体类型-定义类型别名

```c
struct Human { name string }

pub fn (h Human) str() string { return 'Human: $h.name' }

type Person Human

fn test_type_print() {
	p := Person{'Bilbo'}
	println(p)
	assert p.str() == 'Human: Bilbo'
}

pub fn (h Person) str() string { return 'Person: $h.name' }

fn test_person_str() {
	p := Person{'Bilbo'}
	println(p)
	assert p.str() == 'Person: Bilbo'
}
```

