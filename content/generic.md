## 泛型

泛型，最简单的理解就是：**类型作为参数**，在结构体或函数中使用。

有了泛型就可以编写出更为简洁，通用，抽象的代码。

V的泛型目前支持五种：

- 泛型函数
- 泛型结构体
- 泛型方法
- 泛型接口
- 泛型联合类型

### 泛型函数

在泛型函数中，所有普通类型能使用的场景，泛型也应该要能使用。

泛型跟普通类型一样可以作为：

- 函数参数的类型
- 函数返回值的类型
- 函数不确定个数参数的类型
- 函数数组参数的类型
- 数组的类型
- 多维数组的类型
- 固定大小数组的类型
- 泛型支持类型系统中的：所有基本类型，数组，字典类型，信道chan的类型
- 甚至泛型函数和泛型结构体也可以组合起来使用
- 其他各种普通类型能用的场景

```v
module main

fn main() {}

fn simple<T>(p T) T { // 泛型作为函数的参数,返回值
	return p
}

fn multi<T, U>(a T, b U) (T, U) { // 多个泛型
	return a, b
}

fn max<T>(brug string, a ...T) T { //泛型作为不确定参数的类型
	mut max := a[0]
	for item in a[1..] {
		if max < item {
			max = item
		}
	}
	return max
}

fn get_element<T>(arr [3]T) string { // 泛型作为固定大小数组的类型
	return '${arr[1]}'
}

fn f_array<T>(a []T) T { // 泛型作为数组的类型
	return a[0]
}

fn example2<T>(data [][]T) [][][]T { // 泛型作为多维数组的类型
	return [data]
}

fn generic_return_map<M>() map[string]M { // 泛型作为map的value类型
	return {
		'': M{}
	}
}

fn generic_return_nested_map<M>() map[string]map[string]M { //泛型作为嵌套的map类型
	return {
		'': {
			'': M{}
		}
	}
}

fn opt<T>(v T) ?T { //泛型作为返回值,并结合错误处理
	if sizeof(T) > 1 {
		return v
	}
	return none
}

fn mk_chan<T>(f fn () T) chan T { // 泛型作为chan类型
	gench := chan T{cap: 1}
	return gench
}

struct Test<T> {
	v T
}

fn get_test<T>(v T) Test<T> { // 泛型函数和泛型结构体组合使用
	return Test{
		v: v
	}
}

```

泛型函数的调用方式有**标准方式**和**简洁方式**这两种。

简洁方式只有泛型在函数参数中有使用才可以，编译器会自动根据参数的类型，作为泛型的类型。

让调用泛型函数看起来像是调用普通函数那样，简化泛型函数的调用，让代码看起来更舒服。

```v
module main

//泛型函数
fn get<T>(typ T) T {
	return typ
}

//多个类型的泛型函数,参数,函数体,返回值都可以使用
fn get_multi<T, U>(typ T, user U) (T, U) {
	println(typ)
	println(user)
	return typ, user
}

fn main() {
	a1 := get<string>('hello') //标准的泛型调用方式,带<T>
	a2 := get<int>(1)
	println(a1)
	println(a2)
	b1 := get('hello') //简短的泛型调用方式,不用带<T>,就像调用普通函数那样,编译器会根据参数的类型,作为泛型的类型
	b2 := get(1)
	println(b1)
	println(b2)

	x, y := get_multi<int, f64>(2, 2.2) //标准的泛型调用方式,带<T,U>
	println('x is $x,y is $y')
	c, d := get_multi(3, 3.3) //简短的泛型调用方式,不用带<T,U>
	println('c is $c,d is $d')
}

```

### 泛型结构体

在泛型结构体中，所有普通类型能使用的场景，泛型也应该要能使用。

泛型跟普通类型一样，在结构体中可以作为：

- 结构体字段的类型
- 结构体的引用类型&
- 结构体函数类型字段的参数或返回值
- 泛型函数的嵌套使用
- 泛型结构体的嵌套使用
- 泛型结构体的泛型方法
- 甚至泛型函数和泛型结构体也可以组合起来使用
- 其他各种普通类型能用的场景

```v
module main

struct Info<T> { //泛型结构体
	data T //泛型作为字段的类型
}

struct Foo<A, B> {
mut: // 多个泛型
	a A
	b B
}

struct MyStruct<T> {
	arr  []T  // 泛型作为结构体字段的数组
	arr2 [3]T //泛型作为结构体字段的固定大小数组
	m1   map[string]T //泛型作为结构体字段的map
	c    chan T       //泛型作为结构体字段的chan
}

struct Scope<T> {
	before fn () T    // 泛型作为结构体函数类型字段的返回值
	specs  []fn (T) T //// 泛型作为结构体函数类型字段的参数和返回值
	after  fn (T)     // 泛型作为结构体函数类型字段的参数
}

struct Item<T> {
	value T
}

fn (i Item<T>) unwrap() T {
	return i.value
}

fn process<T>(i Item<T>) { //泛型函数和泛型结构体组合使用,泛型结构体作为泛型函数的返回值
	n := i.unwrap()
	println(n)
}

```

### 泛型方法

泛型方法基本跟泛型函数一样，唯一的差别是：如果结构体也是泛型结构体，方法的接收者也都要带上泛型符号。

```v
module main

struct Point {
mut:
	x int = 1
	y int = 1
}

//结构体是普通结构体,方法是泛型方法
fn (mut p Point) translate<T>(x T, y T) {
	p.x += x
	p.y += y
}

struct Abc<T> {
	value T
}

//结构体是泛型结构体,方法的接收者都要带上<T>
fn (s Abc<T>) get_value() T { //泛型结构体的泛型方法
	return s.value
}

fn (s Abc<T>) normal_fn() { //泛型结构体的普通方法
	println('from normal_fn')
}

fn main() {
	mut pot := Point{}
	pot.translate<int>(1, 3)
	println(pot)

	s := Abc<string>{'hello'}
	println(s.get_value())
	s.normal_fn()
}

```

泛型结构体和泛型方法的泛型类型可以不一样：

```v
interface Something {
	i int
}

struct Some {
	i int
}

struct App<M> {
	f M
}

fn (mut self App<M>) next1<M, T>(input T) f64 { // 结构体泛型为M,方法泛型为M,T
	$if M is Something {
		return 0
	} $else {
		panic('${typeof(M.typ).name} is not supported')
		return 1
	}
	return 1
}

fn (mut self App<M>) next2<T, M>(input T) f64 { // 结构体泛型为M,方法泛型为T,M
	$if M is Something {
		return 0
	} $else {
		panic('${typeof(M.typ).name} is not supported')
		return 1
	}
	return 1
}

fn (mut self App<M>) next3<T>(input T) f64 { // 结构体泛型为M,方法泛型为T
	$if M is Something {
		return 0
	} $else {
		panic('${typeof(M.typ).name} is not supported')
		return 1
	}
	return 1
}

fn main() {
	mut app := App<Some>{
		f: Some{
			i: 10
		}
	}
	assert app.next1(1) == 0
	assert app.next2(1) == 0
	assert app.next3(1) == 0
}
```

泛型方法可以嵌套调用：

```v
struct SSS<T> {
mut:
	x T
}

fn (s SSS<T>) inner() T {
	return s.x
}

fn (s SSS<T>) outer() string {
	ret := s.inner<T>() //泛型方法可以嵌套调用
	println(ret)
	return '$ret'
}

fn main() {
	s1 := SSS<int>{100}
	s1.outer()

	s2 := SSS<string>{'hello'}
	s2.outer()
}
```

### 泛型接口

泛型接口的定义跟泛型结构体基本一样。

只有泛型结构体才能实现泛型接口，普通结构体无法实现泛型接口。

```v
//定义泛型接口
interface Gettable<T> {
	get() T
}

struct Animal<T> {
	metadata T
}

// Animal实现泛型接口
fn (a Animal<T>) get<T>() T {
	return a.metadata
}

struct Mineral<T> {
	value T
}

// Mineral也实现泛型接口
fn (m Mineral<T>) get<T>() T {
	return m.value
}

fn extract<T>(xs []Gettable<T>) []T { //使用泛型接口
	return xs.map(it.get())
}

fn extract_basic<T>(xs Gettable<T>) T { //使用泛型接口
	return xs.get()
}

fn main() {
	a := Animal<int>{123}
	b := Animal<int>{456}
	c := Mineral<int>{789}

	arr := [Gettable<int>(a), Gettable<int>(b), Gettable<int>(c)]
	println(typeof(arr).name) //输出:[]Gettable<int>

	x := extract<int>(arr)
	println(x)

	aa := extract_basic(a)
	bb := extract_basic(b)
	cc := extract_basic(c)

	println('$aa | $bb | $cc') //输出:123 | 456 | 789
}

```

### 泛型联合类型

联合类型也支持泛型的结合使用。

```v
struct None {}

//定义泛型联合类型,把泛型作为联合类型中的子类
type MyOption<T> = Error | None | T

struct Foo<T> {
	x T
}

struct Bar<T> {
	x T
}

//定义泛型联合类型,把泛型作为联合类型中的泛型子类
type MyType<T> = Bar<T> | Foo<T>

fn unwrap_if<T>(o MyOption<T>) T {
	if o is T {
		return o
	}
	panic('no value')
}

fn unwrap_match<T>(o MyOption<T>) string {
	match o {
		None {
			return 'none'
		}
		Error {
			return 'none'
		}
		T {
			return 'value'
		}
	}
}

fn main() {
	y := MyOption<bool>(false)

  println(unwrap_if(y)) //输出false
  println(unwrap_match(y)) //输出value

	f := Foo<string>{'hi'}
	t := MyType<string>(f)
	println(t.type_name()) //输出Foo<string>
	println(t.x)	//输出hi
}
```

### 泛型的实现方式

V的泛型实现方式跟rust一样：在编译时，由编译器对泛型函数和泛型结构体进行分析，穷举，把泛型函数所有实际调用到的具体类型转化成具体类型对应的普通函数和普通结构体。

这种实现方式的优点是：性能好，没有运行时开销；

缺点是：如果实际调用到的类型很多，穷举后生成的普通函数和普通结构体也会很多，编译后的可执行文件会变大。

这种取舍在所难免，不过运行快最重要，可执行文件变大，只要不是太夸张还是可以接受的。

V泛型代码：

```v
module main

//泛型函数
fn simple<T>(p T) T {
	return p
}

//泛型结构体
struct Info<T> {
	data T
}

fn main() {
	simple<int>(1)
	simple<int>(1 + 0)
	simple<string>('g')
	simple<[]int>([1])
	simple<map[string]string>({
		'a': 'b'
	})

	info1 := Info<bool>{true}
	info2 := Info<int>{1}
	info3 := Info<f32>{1.1}
	println('$info1,$info2,$info3')
}

```

生成的C代码：

```c
// V typedefs:
typedef struct main__Info_T_bool main__Info_T_bool;
typedef struct main__Info_T_int main__Info_T_int;
typedef struct main__Info_T_f32 main__Info_T_f32;

struct main__Info_T_bool {
	bool data;
};

struct main__Info_T_int {
	int data;
};

struct main__Info_T_f32 {
	f32 data;
};

VV_LOCAL_SYMBOL int main__simple_T_int(int p) {
	return p;
}
VV_LOCAL_SYMBOL string main__simple_T_string(string p) {
	return p;
}
VV_LOCAL_SYMBOL Array_int main__simple_T_Array_int(Array_int p) {
	return p;
}
VV_LOCAL_SYMBOL Map_string_string main__simple_T_Map_string_string(Map_string_string p) {
	return p;
}

VV_LOCAL_SYMBOL void main__main(void) {
	main__simple_T_int(1);
	main__simple_T_int(1 + 0);
	main__simple_T_string(_SLIT("g"));
	main__simple_T_Array_int(new_array_from_c_array(1, 1, sizeof(int), _MOV((int[1]){1})));
	main__simple_T_Map_string_string(new_map_init(&map_hash_string, &map_eq_string, &map_clone_string, &map_free_string, 1, sizeof(string), sizeof(string), _MOV((string[1]){_SLIT("a"), }), _MOV((string[1]){_SLIT("b"), })));
	main__Info_T_bool info1 = (main__Info_T_bool){.data = true,};
	main__Info_T_int info2 = (main__Info_T_int){.data = 1,};
	main__Info_T_f32 info3 = (main__Info_T_f32){.data = 1.1,};
	println( str_intp(4, _MOV((StrIntpData[]){{_SLIT0, 0xfe10, {.d_s = main__Info_T_bool_str(info1)}}, {_SLIT(","), 0xfe10, {.d_s = main__Info_T_int_str(info2)}}, {_SLIT(","), 0xfe10, {.d_s = main__Info_T_f32_str(info3)}}, {_SLIT0, 0, { .d_c = 0 }}})) );
}
```

### 尚未支持的特性

目前V的泛型还没实现：对类型进行where限定，给类型增加条件限定。

