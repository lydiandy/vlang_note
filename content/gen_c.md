## 编译生成C代码

编译器本质上就是一门语言生成另一门语言的过程。

**V目前的编译思路是：编译生成对应的C代码，然后调用C编译器来生成可执行文件。**

V语言的开发重点在编译器前端，C就是编译器后端。

现在也生成了js代码，估计也会生成wasm代码。

或者考虑基于LLVM，生成LLVM IR。

或者直接生成机器码。

------

### 生成C代码

使用-o参数就可以，把当前目录的main.v代码生成main.c代码：

```shell
v -o main.c ./main.v 
```

V代码编译后，生成单个文件的C代码。

通过查看V代码生成的C代码，可以更容易理解V编译器是如何编译的。

### 基本类型对应

V的基本类型通过C的类型别名typedef来实现：

```c
//int类型没有类型别名,直接就是C的int类型
typedef int64_t i64;
typedef int16_t i16;
typedef int8_t i8;
typedef uint64_t u64;
typedef uint32_t u32;
typedef uint8_t u8;
typedef uint16_t u16;
//typedef uint8_t byte; 
typedef uint32_t rune;
typedef size_t usize;
typedef ptrdiff_t isize;
#ifndef VNOFLOAT
typedef float f32;
typedef double f64;
#else
typedef int32_t f32;
typedef int64_t f64;
#endif
typedef int64_t int_literal;
#ifndef VNOFLOAT
typedef double float_literal;
#else
typedef int64_t float_literal;
#endif
typedef unsigned char* byteptr; //字节指针
typedef void* voidptr;//通用指针
typedef char* charptr; //C字符指针
typedef u8 array_fixed_byte_300 [300];

typedef struct sync__Channel* chan;

#ifndef __cplusplus
	#ifndef bool
		#ifdef CUSTOM_DEFINE_4bytebool
			typedef int bool;
		#else
			typedef u8 bool; //布尔类型在C里面默认通过u8类型来实现,1字节
		#endif
		#define true 1 //true是整数常量1
		#define false 0 //false是整数常量0
	#endif
#endif
```

### 代码对照表

#### 常量

int类型常量，生成C的宏定义，其他类型常量，生成C的全局变量。这样就很好理解，V语言中的常量可以是任何类型，跟变量一样，甚至可以是函数调用的结果。

常量的不可修改，由V编译器负责检查。

V代码：

```v
const (
	i  = 1 // int类型的常量
	pi = 3.14 //非int类型的常量
	s  = 'abc' //非int类型的常量
)
```

C代码：

```c
//整数类型的常量通过C宏定义
#define _const_main__i 1

const f64 _const_main__pi = 3.14;

string _const_main__s;
_const_main__s = _SLIT("abc");

VV_LOCAL_SYMBOL void main__main(void) {
	println(int_literal_str(_const_main__i));
	println(float_literal_str(_const_main__pi));
	println(_const_main__s);
}
```



```v
//V代码
const (
	a=i8(11)
	b=i16(12)
	c=i64(13)
	d=byte(14)
	e=u16(15)
	f=u32(16)
	g=u64(17)
	h=f32(1.1)
	i=f64(1.2)
  bb=true
)
//C代码
i8 _const_a; // inited later
i16 _const_b; // inited later
i64 _const_c; // inited later
byte _const_d; // inited later
u16 _const_e; // inited later
u32 _const_f; // inited later
u64 _const_g; // inited later
f32 _const_h; // inited later
f64 _const_i; // inited later
bool _const_bb; // inited later

void _vinit() { //然后在_vinit函数进行初始化
  _const_a = ((i8)(11));
	_const_b = ((i16)(12));
	_const_c = ((i64)(13));
	_const_d = ((byte)(14));
	_const_e = ((u16)(15));
	_const_f = ((u32)(16));
	_const_g = ((u64)(17));
	_const_h = ((f32)(1.1));
	_const_i = ((f64)(1.2));
	_const_bb = true;
}
```

#### 枚举

V的枚举生成等价的C枚举，枚举的pub属性在C中没有对应，由V编译器负责控制。

V代码：

```v
pub enum Color {
	blue = 1 //如果没有指定初始值，默认从0开始，然后往下递增1
	green
	white
	black
}

fn main() {
	c := Color.blue
	println(c)
}
```

C代码：

```c
typedef enum {
	main__Color__blue = 1, // 1
	main__Color__green, // 1+1
	main__Color__white, // 1+2
	main__Color__black, // 1+3
}  main__Color;
 
main__Color c = main__Color__blue;
```

#### 模块

V的模块，在生成对应C代码后，只是对应元素名称的前缀，毕竟C语言中没有模块的概念。

常量，结构体，接口，类型等一级元素生成C代码后的名称规则是：`模块名__名称`，用双下划线区隔。例如：mymodule模块中的add()函数生成C代码后的名称为：`mymodule__add()`。

结构体的方法等二级元素生成C代码后的名称规则是：`模块名__类名_方法名`，用单下划线区隔。例如：模块中的Color结构体的str()方法生成C代码后的名称为：`mymodule__Color_str()`。

V代码：

```v
module main

pub fn main() {
	println('abcd')
}

struct MyStruct {
	x int
	y int
}

pub fn (my_struct MyStruct) add() {
}

```

C代码：

```c
void main__main(void) {
	println(_SLIT("abcd"));
}

struct main__MyStruct {
	int x;
	int y;
};

void main__MyStruct_add(main__MyStruct my_struct) {
}
```



#### 函数

模块中的函数，生成等价的C的函数。

V代码：

```v
module main

fn main() {
	println('from main')
	add(1, 3)
}
// pub的模块访问控制由V编译器负责检查，C没有pub的对应
pub fn add(x int, y int) int { 
	if x > 0 {
		return x + y
	} else {
		return x + y
	}
}
```

C代码：

```c
//生成函数声明段
VV_LOCAL_SYMBOL void main__main(void);
int add(int x, int y); 

//主函数生成主函数
int main(int ___argc, char** ___argv){ 
	g_main_argc = ___argc;
	g_main_argv = ___argv;
#if defined(_VGCBOEHM)
	GC_set_pages_executable(0);
	GC_INIT();
#endif
	_vinit(___argc, (voidptr)___argv);
	main__main();
	_vcleanup();
	return 0;
}

//函数实现段
VV_LOCAL_SYMBOL void main__main(void) {
	println(_SLIT("from main"));
	main__add(1, 3);
}

int main__add(int x, int y) {
	if (x > 0) {
		int _t1 = x + y;
		return _t1;
	} else {
		int _t2 = x + y;
		return _t2;
	}
	return 0;
}
```

#### 函数defer语句

C没有defer语句，V编译的时候就是把函数中的defer语句去掉，然后按后进先出的顺序，放在defer语句之后的所有return语句前，以及函数末尾，有各自独立的代码块。

V代码：

```v
fn main() {
	defer {
		defer_fn1()
	}
	defer {
		defer_fn2()
	}
	println('main start')
	if 1 < 2 {
		return
	}
	if 1 == 1 {
		return
	}

	println('main end')
}

fn defer_fn1() {
	println('from defer_fn1')
}

fn defer_fn2() {
	println('from defer_fn2')
}
```

C代码：

```c
VV_LOCAL_SYMBOL void main__main(void) {
	bool main__main_defer_0 = false;
	bool main__main_defer_1 = false;
	main__main_defer_0 = true;
	main__main_defer_1 = true;
	println(_SLIT("main start"));
  //defer语句之后的所有return语句之前
	if (true) {
			// Defer begin
			if (main__main_defer_1) {
				main__defer_fn2();
			}
			// Defer end
			// Defer begin
			if (main__main_defer_0) {
				main__defer_fn1();
			}
			// Defer end
		return;
	}
  //defer语句之后的所有return语句之前
	if (true) {
			// Defer begin
			if (main__main_defer_1) {
				main__defer_fn2();
			}
			// Defer end
			// Defer begin
			if (main__main_defer_0) {
				main__defer_fn1();
			}
			// Defer end
		return;
	}
	println(_SLIT("main end"));
  //放在函数的最后
	// Defer begin
	if (main__main_defer_1) {
		main__defer_fn2();
	}
	// Defer end
	// Defer begin
	if (main__main_defer_0) {
		main__defer_fn1();
	}
	// Defer end
}

VV_LOCAL_SYMBOL void main__defer_fn1(void) {
	println(_SLIT("from defer_fn1"));
}

VV_LOCAL_SYMBOL void main__defer_fn2(void) {
	println(_SLIT("from defer_fn2"));
}
```

#### 函数不确定个数参数

不确定参数就是根据返回值的类型，编译时动态生成一个C数组，作为函数的最后一个参数。

V代码：

```v
fn my_fn(i int, s string, others ...string) {
	println(i)
	println(s)
	println(others[0])
	println(others[1])
	println(others[2])
}

fn main() {
	my_fn(1, 'abc', 'de', 'fg', 'hi')
}
```

C代码：

```c
struct array {
	int element_size;
	voidptr data;
	int offset;
	int len;
	int cap;
	ArrayFlags flags;
};

typedef array Array_string;

VV_LOCAL_SYMBOL void main__my_fn(int i, string s, Array_string others) {
	println(int_str(i));
	println(s);
	println((*(string*)array_get(others, 0)));
	println((*(string*)array_get(others, 1)));
	println((*(string*)array_get(others, 2)));
}

VV_LOCAL_SYMBOL void main__main(void) {
	main__my_fn(1, _SLIT("abc"), new_array_from_c_array(3, 3, sizeof(string), _MOV((string[3]){_SLIT("de"), _SLIT("fg"), _SLIT("hi")})));
}

//C代码
struct varg_string { //自动生成不确定参数结构体
  int len;
  string args[4];
};

 void main__my_fn(int i, string s, varg_string *others) {
   printf("%d\n", i);
   println(s);
   println(others->args[0]);
   println(others->args[1]);
   println(others->args[2]);
 }

 void main__main() {
   main__my_fn(
       1, tos3("abc"),
       &(varg_string){.len = 3, .args = {tos3("de"), tos3("fg"), 				tos3("hi")}});
 }
```

#### 函数多返回值

C的函数返回值只有1个，V的函数多返回值，就是把多返回值的组合，编译时动态生成一个结构体，然后返回结构体。并且生成的返回值组合的结构体，还可以给其他相同类型的多返回值的函数公用。

V代码：

```v
fn foo() (int, int) { //多返回值
	return 2, 3
}

fn multi_return_fn(a int, b int) (int, int) {
	return a + 1, b + 1 //可以返回表达式
}

fn main() {
	a, b := foo()
	println(a) // 2
	println(b) // 3
}
```

C代码：

```c
typedef struct multi_return_int_int multi_return_int_int;

struct multi_return_int_int {
	int arg0;
	int arg1;
};

VV_LOCAL_SYMBOL multi_return_int_int main__multi_return_fn(int a, int b);

//生成的返回值组合的结构体,还可以给其他函数共用
VV_LOCAL_SYMBOL multi_return_int_int main__multi_return_fn(int a, int b) {
	return (multi_return_int_int){.arg0=a + 1, .arg1=b + 1};
}

VV_LOCAL_SYMBOL void main__main(void) {
	multi_return_int_int mr_271 = main__foo();
	int a = mr_271.arg0;
	int b = mr_271.arg1;
	println(int_str(a));
	println(int_str(b));
}
```

#### 数组

V的数组是用struct来实现的，生成C代码也是struct。

V代码：

```v
fn main() {
	a := [1, 3, 5]
	b := ['a', 'b', 'c']
	println(a)
	println(b)
}
```

C代码：

```v
//第一部分: 从内置的vlib/built/array.v生成
typedef struct array array;
struct array {
	int element_size;
	voidptr data;
	int offset;
	int len;
	int cap;
	ArrayFlags flags;
};

//第二部分:从内置的vlib/built/array.v生成
typedef array Array_string;
typedef array Array_u8;
typedef array Array_int;
typedef array Array_voidptr;
typedef array Array_VCastTypeIndexName;
typedef array Array_MethodArgs;
typedef array Array_u8_ptr;
typedef array Array_rune;
typedef array Array_u64;
typedef array Array_u32;
typedef array Array_strconv__Uint128;
typedef array Array_f64;

//第三部分:数组使用的代码,字面量方式创建
VV_LOCAL_SYMBOL void main__main(void) {
	Array_int a = new_array_from_c_array_noscan(3, 3, sizeof(int), _MOV((int[3]){1, 3, 5}));
	Array_string b = new_array_from_c_array(3, 3, sizeof(string), _MOV((string[3]){_SLIT("a"), _SLIT("b"), _SLIT("c")}));
	println(Array_int_str(a));
	println(Array_string_str(b));
}
```

#### 字符串

V的字符串是用struct来实现的，生成C代码也是struct。

V代码：

```v
fn main(){
	mystr:='abc'
	mystr2:="def"
	println(mystr)
	println(mystr2)
}
```

C代码：

```c
#define _SLIT(s) ((string){.str=(byteptr)("" s), .len=(sizeof(s)-1), .is_lit=1})

typedef struct string string;

struct string {
	u8* str;
	int len;
	int is_lit;
};

VV_LOCAL_SYMBOL void main__main(void) {
	string mystr = _SLIT("abc");
	string mystr2 = _SLIT("def");
	println(mystr);
	println(mystr2);
}
```

#### 字典

V的字典是用struct来实现的，生成C代码也是struct。

V代码：

```v
fn main() {
	mut m := map[string]int{}
	m['one'] = 1
	m['two'] = 2
	println(m['one'])
	println(m['bad_key'])
}
```

C代码：

```c
typedef struct map map;
typedef map Map_string_int;

struct map {
	int key_bytes;
	int value_bytes;
	u32 even_index;
	u8 cached_hashbits;
	u8 shift;
	DenseArray key_values;
	u32* metas;
	u32 extra_metas;
	bool has_string_keys;
	MapHashFn hash_fn;
	MapEqFn key_eq_fn;
	MapCloneFn clone_fn;
	MapFreeFn free_fn;
	int len;
};

struct DenseArray {
	int key_bytes;
	int value_bytes;
	int cap;
	int len;
	u32 deletes;
	u8* all_deleted;
	u8* keys;
	u8* values;
};

typedef u64 (*MapHashFn)(voidptr);
typedef bool (*MapEqFn)(voidptr, voidptr);
typedef void (*MapCloneFn)(voidptr, voidptr);
typedef void (*MapFreeFn)(voidptr);

VV_LOCAL_SYMBOL map new_map_noscan_value(int key_bytes, int value_bytes, u64 (*hash_fn)(voidptr ), bool (*key_eq_fn)(voidptr , voidptr ), void (*clone_fn)(voidptr , voidptr ), void (*free_fn)(voidptr )) {
	int metasize = ((int)(sizeof(u32) * (_const_init_capicity + _const_extra_metas_inc)));
	bool has_string_keys = _us32_lt(sizeof(voidptr),key_bytes);
	return ((map){
		.key_bytes = key_bytes,
		.value_bytes = value_bytes,
		.even_index = _const_init_even_index,
		.cached_hashbits = _const_max_cached_hashbits,
		.shift = _const_init_log_capicity,
		.key_values = new_dense_array_noscan(key_bytes, false, value_bytes, true),
		.metas = ((u32*)(vcalloc_noscan(metasize))),
		.extra_metas = _const_extra_metas_inc,
		.has_string_keys = has_string_keys,
		.hash_fn = (voidptr)hash_fn,
		.key_eq_fn = (voidptr)key_eq_fn,
		.clone_fn = (voidptr)clone_fn,
		.free_fn = (voidptr)free_fn,
		.len = 0,
	});
}

VV_LOCAL_SYMBOL void main__main(void) {
	Map_string_int m = new_map_noscan_value(sizeof(string), sizeof(int), &map_hash_string, &map_eq_string, &map_clone_string, &map_free_string)
	;
	map_set(&m, &(string[]){_SLIT("one")}, &(int[]) { 1 });
	map_set(&m, &(string[]){_SLIT("two")}, &(int[]) { 2 });
	println(int_str((*(int*)map_get(ADDR(map, m), &(string[]){_SLIT("one")}, &(int[]){ 0 }))));
	println(int_str((*(int*)map_get(ADDR(map, m), &(string[]){_SLIT("bad_key")}, &(int[]){ 0 }))));
}
```

#### 结构体

生成对应的C结构体：

V代码：

```v
pub struct Point {
	x int
	y int
}

pub fn (p Point) str() string {
	return 'x is ${p.x},y is:${p.y}'
}

fn main() {
	p := Point{
		x: 1
		y: 3
	}
	println(p)
}

```

C代码：

```c
//结构体和函数声明段
typedef struct main__Point main__Point;
string main__Point_str(main__Point p);

//结构体和函数实现段
struct main__Point {
	int x;
	int y;
};

string main__Point_str(main__Point p) {
	string _t1 =  str_intp(3, _MOV((StrIntpData[]){{_SLIT("x is "), /*100 &int*/0xfe07, {.d_i32 = p.x}}, {_SLIT(",y is:"), /*100 &int*/0xfe07, {.d_i32 = p.y}}, {_SLIT0, 0, { .d_c = 0 }}}));
	return _t1;
}

VV_LOCAL_SYMBOL void main__main(void) {
	main__Point p = ((main__Point){.x = 1,.y = 3,});
	println(main__Point_str(p));
}
```

#### 结构体方法

结构体方法生成C函数，只是函数的第一个参数是对应结构体类型的指针，

生成的C函数命名规则是：`结构体名_方法名`。

V代码：

```v
pub fn (mut a array) insert(i int, val voidptr) {
...
}
```

C代码：

```v
void array_insert(array *a, int i, void *val) { //默认第一个参数是对应类型指针
...
}
```

#### 结构体访问控制

访问控制在C代码中没有体现，全部在V编译器中控制。

V代码：

```v
struct Foo {
	a int //私有,不可变(默认).在模块内部可访问,不可修改;模块外不可访问,不可修改
mut:
	b int // 私有,可变.在模块内部可访问,可修改,模块外部不可访问,不可修改
	c int // (相同访问控制的字段可以放在一起)
pub:
	d int // 公共,不可变,只读.在模块内部和外部都可以访问,但是不可修改
pub mut:
	e int //公共,模块内部可访问,可修改;模块外部可访问,但是不可修改
__global:
	f int // 全局字段,模块内部和外部都可访问,可修改,这样等于破坏了封装性,不推荐使用
}

fn main() {
	f := Foo{}
	println(f)
}
```

C代码：

```c
typedef struct main__Foo main__Foo;

struct main__Foo {
	int a;
	int b;
	int c;
	int d;
	int e;
	int f;
};

VV_LOCAL_SYMBOL void main__main(void) {
	main__Foo f = ((main__Foo){.a = 0,.b = 0,.c = 0,.d = 0,.e = 0,.f = 0,});
	println(main__Foo_str(f));
}
```

#### 流程控制语句

##### 条件

if语句，生成C if语句：

V代码：

```v
fn main() {
	a := 10
	b := 20
	if a < b {
		println('a<b')
	} else if a > b {
		println('a>b')
	} else {
		println('a=b')
	}
}
```

C代码：

```c
VV_LOCAL_SYMBOL void main__main(void) {
	int a = 10;
	int b = 20;
	if (a < b) {
		println(_SLIT("a<b"));
	} else if (a > b) {
		println(_SLIT("a>b"));
	} else {
		println(_SLIT("a=b"));
	}
}
```

if表达式语句，生成C的三元运算符 ? :

V代码：

```v
fn main() {
	num := 777
	s := if num % 2 == 0 {
		'even'
	} else {
		'odd'
	}
	println(s) // "odd"
}
```

C代码：

```c
VV_LOCAL_SYMBOL void main__main(void) {
	int num = 777;
	string s = (num % 2 == 0 ? (_SLIT("even")) : (_SLIT("odd")));
	println(s);
}
```

##### 分支

match语句，生成C的if-else if-else语句。

V代码：

```v
fn main() {
os:='macos'
match os {
	'windows' {
    	println('windows')
	}
	'linux' {
    	println('linux')
	}
	'macos' {
    	println('macos')
	}
	else  {
   	 println('unknow')
	}
}

```

C代码：

```c
VV_LOCAL_SYMBOL void main__main(void) {
	string os = _SLIT("macos");

	if (string__eq(os, _SLIT("windows"))) {
		println(_SLIT("windows"));
	}
	else if (string__eq(os, _SLIT("linux"))) {
		println(_SLIT("linux"));
	}
	else if (string__eq(os, _SLIT("macos"))) {
		println(_SLIT("macos"));
	}
	else {
		println(_SLIT("unknow"));
	}
}
```

match表达式语句，生成嵌套的三元运算符语句。

V代码：

```v
fn main() {
	os := 'macos'
	price := match os {
		'windows' {
			100
		}
		'linux' {
			120
		}
		'macos' {
			150
		}
		else {
			0
		}
	}
	println(price) //输出150
}
```

C代码：

```c
VV_LOCAL_SYMBOL void main__main(void) {
	string os = _SLIT("macos");
	int price = ((string__eq(os, _SLIT("windows")))? (100) : (string__eq(os, _SLIT("linux")))? (120) : (string__eq(os, _SLIT("macos")))? (150) : (0));
	println(int_str(price));
}
```

##### 循环

步长：for i=0;i<100;i++ {}

V代码：

```v
fn main() {
	for i := 0; i < 10; i++ {
		//跳过6
		if i == 6 {
			continue
		}
		println(i)
	}
}

```

C代码：

```c
VV_LOCAL_SYMBOL void main__main(void) {
	for (int i = 0; i < 10; i++) {
		if (i == 6) {
			continue;
		}
		println(int_str(i));
	}
}
```

循环：for i<100 {}

V代码：

```v
fn main() {
	mut sum := 0
	mut i := 0
	for i <= 100 {
		sum += i
		i++
	}
	println(sum) // 输出"5050"
}

```

C代码：

```c
VV_LOCAL_SYMBOL void main__main(void) {
	int sum = 0;
	int i = 0;
	for (;;) {
		if (!(i <= 100)) break;
		sum += i;
		i++;
	}
	println(int_str(sum));
}
```

无限循环：for {}

V代码：

```v
fn main() {
	mut num := 0
	for {
		num++
		if num >= 10 {
			break
		}
	}
	println(num) // "10"
}

```

C代码：

```v
VV_LOCAL_SYMBOL void main__main(void) {
	int num = 0;
	for (;;) {
		num++;
		if (num >= 10) {
			break;
		}
	}
	println(int_str(num));
}
```

遍历：for i in xxx {}

V代码：

```v
fn main() {
	numbers := [1, 2, 3, 4, 5]
	for i, num in numbers {
		println('for-in')
	}
}
```

C代码：

```c
VV_LOCAL_SYMBOL void main__main(void) {
	Array_int numbers = new_array_from_c_array_noscan(5, 5, sizeof(int), _MOV((int[5]){1, 2, 3, 4, 5}));
	for (int i = 0; i < numbers.len; ++i) {
		int num = ((int*)numbers.data)[i];
		println(_SLIT("for-in"));
	}
}
```

#### 类型定义

类型定义type生成C的类型别名typedef。

V代码：

```v
pub struct Point {
	x int
	y int
}

type Myint = int
type MyPoint = Point

```

C代码：

```c
typedef struct main__Point main__Point;
typedef int main__Myint;
typedef main__Point main__MyPoint;

struct main__Point {
	int x;
	int y;
};
```

#### 接口

在编译时穷举所有实现了接口的结构体，构造接口的结构体，接口结构体包含了一个匿名联合体和类型id。然后为每一个实现了接口的联合体生成一个创建函数，返回接口的结构体，并通过类型id标识具体的类型。

V代码：

```v
module main

//接口要求结构体要实现方法为 pub fn (s MyStruct) write(a string) string
pub interface Foo {
	write(string) string
}

struct MyStruct {}

pub fn (s MyStruct) write(a string) string {
	return a
}

fn main() {
	s1 := MyStruct{}
	fn1(s1)
}

fn fn1(s Foo) {
	println(s.write('Foo'))
}
```

C代码：

```c
static char * v_typeof_interface_main__Foo(int sidx);

typedef struct main__Foo main__Foo;
typedef struct main__MyStruct main__MyStruct;
typedef struct main__MyStruct2 main__MyStruct2;
//实现了接口的结构体
struct main__MyStruct {
	EMPTY_STRUCT_DECLARATION;
};

struct main__MyStruct2 {
	EMPTY_STRUCT_DECLARATION;
};
//结构体的接口实现方法
string main__MyStruct_write(main__MyStruct s, string a) {
	return a;
}
//结构体的接口实现方法
string main__MyStruct2_write(main__MyStruct2 s, string a) {
	return a;
}
//接口的结构体，编译时穷举所有实现了该接口的结构体，包在匿名联合体内，并通过类型id标识
struct main__Foo {
	union {
		void* _object;
		main__MyStruct* _main__MyStruct;
		voidptr* _voidptr;
		main__MyStruct2* _main__MyStruct2;
	};
	int _typ;
};

// Methods wrapper for interface "main__Foo"
static inline string main__MyStruct_write_Interface_main__Foo_method_wrapper(main__MyStruct* s, string a) {
	return main__MyStruct_write(*s, a);
}
static inline string main__MyStruct2_write_Interface_main__Foo_method_wrapper(main__MyStruct2* s, string a) {
	return main__MyStruct2_write(*s, a);
}

struct _main__Foo_interface_methods {
	string (*_method_write)(void* _, string );
};

 struct _main__Foo_interface_methods main__Foo_name_table[3] = {
	{
		._method_write = (void*) main__MyStruct_write_Interface_main__Foo_method_wrapper,
	},
	{
		._method_write = (void*) 0,
	},
	{
		._method_write = (void*) main__MyStruct2_write_Interface_main__Foo_method_wrapper,
	},
};
//把具体的结构体转换成接口结构体函数
static main__Foo I_main__MyStruct_to_Interface_main__Foo(main__MyStruct* x);
 const int _main__Foo_main__MyStruct_index = 0;

static main__Foo I_voidptr_to_Interface_main__Foo(voidptr* x);
 const int _main__Foo_voidptr_index = 1;

static main__Foo I_main__MyStruct2_to_Interface_main__Foo(main__MyStruct2* x);
 const int _main__Foo_main__MyStruct2_index = 2;

// Casting functions for converting "main__MyStruct" to interface "main__Foo"
static inline main__Foo I_main__MyStruct_to_Interface_main__Foo(main__MyStruct* x) {
	return (main__Foo) {
		._main__MyStruct = x,
		._typ = _main__Foo_main__MyStruct_index,
	};
}

// Casting functions for converting "voidptr" to interface "main__Foo"
static inline main__Foo I_voidptr_to_Interface_main__Foo(voidptr* x) {
	return (main__Foo) {
		._voidptr = x,
		._typ = _main__Foo_voidptr_index,
	};
}

// Casting functions for converting "main__MyStruct2" to interface "main__Foo"
static inline main__Foo I_main__MyStruct2_to_Interface_main__Foo(main__MyStruct2* x) {
	return (main__Foo) {
		._main__MyStruct2 = x,
		._typ = _main__Foo_main__MyStruct2_index,
	};
}
//
static char * v_typeof_interface_main__Foo(int sidx) { /* main.Foo */ 
	if (sidx == _main__Foo_main__MyStruct_index) return "MyStruct";
	if (sidx == _main__Foo_voidptr_index) return "voidptr";
	if (sidx == _main__Foo_main__MyStruct2_index) return "MyStruct2";
	return "unknown Foo";
}

static int v_typeof_interface_idx_main__Foo(int sidx) { /* main.Foo */ 
	if (sidx == _main__Foo_main__MyStruct_index) return 95;
	if (sidx == _main__Foo_voidptr_index) return 2;
	if (sidx == _main__Foo_main__MyStruct2_index) return 96;
	return 94;
}
//主函数
VV_LOCAL_SYMBOL void main__main(void) {
	main__MyStruct *s1 = HEAP(main__MyStruct, (((main__MyStruct){EMPTY_STRUCT_INITIALIZATION})));
	main__fn1(/*&main.Foo*/I_main__MyStruct_to_Interface_main__Foo(&(*(s1))));
}

VV_LOCAL_SYMBOL void main__fn1(main__Foo s) {
	println(main__Foo_name_table[s._typ]._method_write(s._object, _SLIT("Foo")));
}
```

#### 泛型

##### 泛型函数/方法

编译时，穷举所有泛型函数实际调用的所有类型，为每一个实际调用的类型，生成对应版本的C函数。

V代码：

```v
module main

fn simple[T](p T) T { // 泛型作为函数的参数,返回值
	return p
}

fn multi[T, U](a T, b U) (T, U) { // 多个泛型
	return a, b
}

fn main() {
  //实际调用过：int，string，bool类型
	simple(1) 
	simple('abc')
	simple(true)
	//实际调用过：int+string，f64+bool这两种类型的组合
	multi(1,'abc')
	multi(1.2,true)

}
```

C代码：

```c
VV_LOCAL_SYMBOL int main__simple_T_int(int p);
VV_LOCAL_SYMBOL string main__simple_T_string(string p);
VV_LOCAL_SYMBOL bool main__simple_T_bool(bool p);
VV_LOCAL_SYMBOL multi_return_int_string main__multi_T_int_string(int a, string b);
VV_LOCAL_SYMBOL multi_return_f64_bool main__multi_T_f64_bool(f64 a, bool b);
 //实际调用过：int，string，bool类型
VV_LOCAL_SYMBOL int main__simple_T_int(int p) {
	return p;
}
VV_LOCAL_SYMBOL string main__simple_T_string(string p) {
	return p;
}
VV_LOCAL_SYMBOL bool main__simple_T_bool(bool p) {
	return p;
}
//实际调用过：int+string，f64+bool这两种类型的组合
VV_LOCAL_SYMBOL multi_return_int_string main__multi_T_int_string(int a, string b) {
	return (multi_return_int_string){.arg0=a, .arg1=b};
}
VV_LOCAL_SYMBOL multi_return_f64_bool main__multi_T_f64_bool(f64 a, bool b) {
	return (multi_return_f64_bool){.arg0=a, .arg1=b};
}

VV_LOCAL_SYMBOL void main__main(void) {
	main__simple_T_int(1);
	main__simple_T_string(_SLIT("abc"));
	main__simple_T_bool(true);
	main__multi_T_int_string(1, _SLIT("abc"));
	main__multi_T_f64_bool(1.2, true);
}
```

##### 泛型结构体

编译时，穷举所有泛型类型实际调用的所有类型，为每一个实际调用的类型，生成对应版本的C结构体。

V代码：

```v
module main

struct Info[T] { //泛型结构体
	data T //泛型作为字段的类型
}

struct Foo[A, B] {
mut: // 多个泛型
	a A
	b B
}

fn main() {
	i := Info[int]{
		data: 1
	}
	s := Info[string]{
		data: 'abc'
	}
	b := Info[bool]{
		data: true
	}
	println(i)
	println(s)
	println(b)

	x := Foo[int, string]{
		a: 1
		b: 'abc'
	}
	y := Foo[f64, bool]{
		a: 1.2
		b: true
	}
	println(x)
	println(y)
}
```

C代码：

```c
typedef struct main__Info_T_string main__Info_T_string;
typedef struct main__Info_T_int main__Info_T_int;
typedef struct main__Info_T_bool main__Info_T_bool;
typedef struct main__Foo_T_int_string main__Foo_T_int_string;
typedef struct main__Foo_T_f64_bool main__Foo_T_f64_bool;

struct main__Info_T_string {
	string data;
};
struct main__Info_T_int {
	int data;
};
struct main__Info_T_bool {
	bool data;
};

struct main__Foo_T_int_string {
	int a;
	string b;
};
struct main__Foo_T_f64_bool {
	f64 a;
	bool b;
};

VV_LOCAL_SYMBOL void main__main(void) {
	main__Info_T_int i = ((main__Info_T_int){.data = 1,});
	main__Info_T_string s = ((main__Info_T_string){.data = _SLIT("abc"),});
	main__Info_T_bool b = ((main__Info_T_bool){.data = true,});
	println(main__Info_T_int_str(i));
	println(main__Info_T_string_str(s));
	println(main__Info_T_bool_str(b));
	main__Foo_T_int_string x = ((main__Foo_T_int_string){.a = 1,.b = _SLIT("abc"),});
	main__Foo_T_f64_bool y = ((main__Foo_T_f64_bool){.a = 1.2,.b = true,});
	println(main__Foo_T_int_string_str(x));
	println(main__Foo_T_f64_bool_str(y));
}
```

##### 泛型接口

编译时，穷举所有泛型接口实际调用的所有类型，为每一个实际调用的类型，生成对应版本的C结构体。

V代码：

```v
//定义泛型接口
interface Gettable[T] {
	get() T
}

struct Animal[T] {
	metadata T
}

// Animal实现泛型接口
fn (a Animal[T]) get[T]() T {
	return a.metadata
}

struct Mineral[T] {
	value T
}

// Mineral也实现泛型接口
fn (m Mineral[T]) get[T]() T {
	return m.value
}

fn extract[T](xs []Gettable[T]) []T { //使用泛型接口
	return xs.map(it.get())
}

fn extract_basic[T](xs Gettable[T]) T { //使用泛型接口
	return xs.get()
}

fn main() {
	a := Animal[int]{123}
	b := Animal[int]{456}
	c := Mineral[int]{789}

	arr := [Gettable[int](a), Gettable[int](b), Gettable[int](c)]
	println(typeof(arr).name) //输出:[]Gettable[int]

	x := extract[int](arr)
	println(x)

	aa := extract_basic(a)
	bb := extract_basic(b)
	cc := extract_basic(c)

	println('${aa} | ${bb} | ${cc}') //输出:123 | 456 | 789
}

```

C代码：

```c
#define HEAP(type, expr) ((type*)memdup((void*)&((type[]){expr}[0]), sizeof(type)))
//
typedef struct main__Animal_T_int main__Animal_T_int;
typedef struct main__Mineral_T_int main__Mineral_T_int;

typedef struct main__Gettable main__Gettable;
typedef struct main__Gettable_T_int main__Gettable_T_int;

static char * v_typeof_interface_main__Gettable_T_int(int sidx);

VV_LOCAL_SYMBOL Array_int main__extract_T_int(Array_main__Gettable_T_int xs);
VV_LOCAL_SYMBOL int main__extract_basic_T_int(main__Gettable_T_int xs);

static main__Gettable_T_int I_main__Animal_T_int_to_Interface_main__Gettable_T_int(main__Animal_T_int* x);
 const int _main__Gettable_T_int_main__Animal_T_int_index = 0;
static main__Gettable_T_int I_voidptr_to_Interface_main__Gettable_T_int(voidptr* x);
 const int _main__Gettable_T_int_voidptr_index = 1;
static main__Gettable_T_int I_main__Mineral_T_int_to_Interface_main__Gettable_T_int(main__Mineral_T_int* x);
 const int _main__Gettable_T_int_main__Mineral_T_int_index = 2;
// ^^^ number of types for interface main__Gettable_T_int: 3

// Methods wrapper for interface "main__Gettable_T_int"
static inline int main__Animal_T_int_get_T_int_Interface_main__Gettable_T_int_method_wrapper(main__Animal_T_int* a) {
	return main__Animal_T_int_get_T_int(*a);
}
static inline int main__Mineral_T_int_get_T_int_Interface_main__Gettable_T_int_method_wrapper(main__Mineral_T_int* m) {
	return main__Mineral_T_int_get_T_int(*m);
}

struct _main__Gettable_T_int_interface_methods {
	int (*_method_get)(void* _);
};

 struct _main__Gettable_T_int_interface_methods main__Gettable_T_int_name_table[3] = {
	{
		._method_get = (void*) main__Animal_T_int_get_T_int_Interface_main__Gettable_T_int_method_wrapper,
	},
	{
		._method_get = (void*) 0,
	},
	{
		._method_get = (void*) main__Mineral_T_int_get_T_int_Interface_main__Gettable_T_int_method_wrapper,
	},
};


// Casting functions for converting "main__Animal_T_int" to interface "main__Gettable_T_int"
static inline main__Gettable_T_int I_main__Animal_T_int_to_Interface_main__Gettable_T_int(main__Animal_T_int* x) {
	return (main__Gettable_T_int) {
		._main__Animal_T_int = x,
		._typ = _main__Gettable_T_int_main__Animal_T_int_index,
	};
}

// Casting functions for converting "voidptr" to interface "main__Gettable_T_int"
static inline main__Gettable_T_int I_voidptr_to_Interface_main__Gettable_T_int(voidptr* x) {
	return (main__Gettable_T_int) {
		._voidptr = x,
		._typ = _main__Gettable_T_int_voidptr_index,
	};
}

// Casting functions for converting "main__Mineral_T_int" to interface "main__Gettable_T_int"
static inline main__Gettable_T_int I_main__Mineral_T_int_to_Interface_main__Gettable_T_int(main__Mineral_T_int* x) {
	return (main__Gettable_T_int) {
		._main__Mineral_T_int = x,
		._typ = _main__Gettable_T_int_main__Mineral_T_int_index,
	};
}

VV_LOCAL_SYMBOL Array_int main__extract_T_int(Array_main__Gettable_T_int xs) {
	Array_int _t2 = {0};
	Array_main__Gettable_T_int _t2_orig = xs;
	int _t2_len = _t2_orig.len;
	_t2 = __new_array_noscan(0, _t2_len, sizeof(int));

	for (int _t3 = 0; _t3 < _t2_len; ++_t3) {
		main__Gettable_T_int it = ((main__Gettable_T_int*) _t2_orig.data)[_t3];
		int ti = main__Gettable_T_int_name_table[it._typ]._method_get(it._object);
		array_push_noscan((array*)&_t2, &ti);
	}
	Array_int _t1 =_t2;
	return _t1;
}

VV_LOCAL_SYMBOL int main__extract_basic_T_int(main__Gettable_T_int xs) {
	int _t1 = main__Gettable_T_int_name_table[xs._typ]._method_get(xs._object);
	return _t1;
}

VV_LOCAL_SYMBOL Array_int main__extract_T_int(Array_main__Gettable_T_int xs) {
	Array_int _t2 = {0};
	Array_main__Gettable_T_int _t2_orig = xs;
	int _t2_len = _t2_orig.len;
	_t2 = __new_array_noscan(0, _t2_len, sizeof(int));

	for (int _t3 = 0; _t3 < _t2_len; ++_t3) {
		main__Gettable_T_int it = ((main__Gettable_T_int*) _t2_orig.data)[_t3];
		int ti = main__Gettable_T_int_name_table[it._typ]._method_get(it._object);
		array_push_noscan((array*)&_t2, &ti);
	}
	Array_int _t1 =_t2;
	return _t1;
}

VV_LOCAL_SYMBOL int main__extract_basic_T_int(main__Gettable_T_int xs) {
	int _t1 = main__Gettable_T_int_name_table[xs._typ]._method_get(xs._object);
	return _t1;
}
//泛型结构体，实际调用的结构体
struct main__Animal_T_int {
	int metadata;
};
struct main__Mineral_T_int {
	int value;
};
//泛型接口，实际调用的结构体
struct main__Gettable_T_int {
	union {
		void* _object;
		main__Animal_T_int* _main__Animal_T_int;
		voidptr* _voidptr;
		main__Mineral_T_int* _main__Mineral_T_int;
	};
	int _typ;
};

//主函数
VV_LOCAL_SYMBOL void main__main(void) {
	main__Animal_T_int *a = HEAP(main__Animal_T_int, (((main__Animal_T_int){.metadata = 123,})));
	main__Animal_T_int *b = HEAP(main__Animal_T_int, (((main__Animal_T_int){.metadata = 456,})));
	main__Mineral_T_int *c = HEAP(main__Mineral_T_int, (((main__Mineral_T_int){.value = 789,})));
	Array_main__Gettable_T_int arr = new_array_from_c_array(3, 3, sizeof(main__Gettable_T_int), _MOV((main__Gettable_T_int[3]){/*&main.Gettable[int]*/I_main__Animal_T_int_to_Interface_main__Gettable_T_int(&(*(a))), /*&main.Gettable[int]*/I_main__Animal_T_int_to_Interface_main__Gettable_T_int(&(*(b))), /*&main.Gettable[int]*/I_main__Mineral_T_int_to_Interface_main__Gettable_T_int(&(*(c)))}));
	println(_SLIT("[]Gettable[int]"));
	Array_int x = main__extract_T_int(arr);
	println(Array_int_str(x));
	int aa = main__extract_basic_T_int(/*&main.Gettable[int]*/I_main__Animal_T_int_to_Interface_main__Gettable_T_int(&(*(a))));
	int bb = main__extract_basic_T_int(/*&main.Gettable[int]*/I_main__Animal_T_int_to_Interface_main__Gettable_T_int(&(*(b))));
	int cc = main__extract_basic_T_int(/*&main.Gettable[int]*/I_main__Mineral_T_int_to_Interface_main__Gettable_T_int(&(*(c))));
	println( str_intp(4, _MOV((StrIntpData[]){{_SLIT0, /*100 &int*/0xfe07, {.d_i32 = aa}}, {_SLIT(" | "), /*100 &int*/0xfe07, {.d_i32 = bb}}, {_SLIT(" | "), /*100 &int*/0xfe07, {.d_i32 = cc}}, {_SLIT0, 0, { .d_c = 0 }}})));
}
```

##### 泛型联合类型

编译时，也是穷举所有实际调用的类型，生成对应的C结构体。

V代码：

```v
struct None {}

//定义泛型联合类型,把泛型作为联合类型中的子类
type MyOption[T] = Error | None | T

fn unwrap_if[T](o MyOption[T]) T {
	if o is T {
		return o
	}
	panic('no value')
}


fn main() {
	y := MyOption[bool](false)
	println(unwrap_if(y)) //输出false
}
```

C代码：

```c
typedef struct main__MyOption_T_bool main__MyOption_T_bool;
VV_LOCAL_SYMBOL bool main__unwrap_if_T_bool(main__MyOption_T_bool o);

struct main__MyOption_T_bool {
	union {
		Error* _Error;
		main__None* _main__None;
		bool* _bool;
	};
	int _typ;
};

static inline main__MyOption_T_bool bool_to_sumtype_main__MyOption_T_bool(bool* x) {
	bool* ptr = memdup(x, sizeof(bool));
	return (main__MyOption_T_bool){ ._bool = ptr, ._typ = 18};
}

char * v_typeof_sumtype_main__MyOption_T_bool(int sidx) { /* main.MyOption[bool] */ 
	switch(sidx) {
		case 97: return "MyOption[bool]";
		case 75: return "Error";
		case 94: return "None";
		case 18: return "bool";
		default: return "unknown MyOption[bool]";
	}
}

int v_typeof_sumtype_idx_main__MyOption_T_bool(int sidx) { /* main.MyOption[bool] */ 
	switch(sidx) {
		case 97: return 97;
		case 75: return 75;
		case 94: return 94;
		case 18: return 18;
		default: return 97;
	}
}
//
VV_LOCAL_SYMBOL bool main__unwrap_if_T_bool(main__MyOption_T_bool o) {
	if ((o)._typ == 18 /* bool */) {
		return (*o._bool);
	}
	_v_panic(_SLIT("no value"));
	VUNREACHABLE();
	return 0;
}
//
VV_LOCAL_SYMBOL void main__main(void) {
	main__MyOption_T_bool y = bool_to_sumtype_main__MyOption_T_bool(ADDR(bool, (false)));
	println(main__unwrap_if_T_bool(y) ? _SLIT("true") : _SLIT("false"));
}
```

#### 错误处理

跟函数的多返回值类似，编译器为每一种返回类型，生成一个对应类型的结构体，全局共用。

V代码：

```v
module main

// ?表示疑问,表示可能返回期望的类型,也可能返回一个空值none
pub fn return_type_or_none(x int) ?int { 
	match x {
		0 { return none }
		else { return x }
	}
}

// !表示警告,表示可能返回期望的类型,也可能返回一个错误IError
pub fn return_type_or_error(x int) !int { 
	match x {
		0 { return error('error: x can not be 0') }
		else { return x }
	}
}

fn main() {
	v1 := return_type_or_none(0) or {
		match err {
			none { 10 } //如果返回空值none,可以指定默认值
			else { panic(err) }
		}
	}
	println(v1)

	v2 := return_type_or_none(1) or {
		match err {
			none { 10 } //如果返回空值none,可以指定默认值
			else { panic(err) }
		}
	}
	println(v2)

	v3 := return_type_or_error(2) or { panic(err) }
	println(v3)

	v4 := return_type_or_error(0) or { panic(err) }
	println(v4)
}
```

C代码：

```c
typedef struct _option_int _option_int; 
typedef struct _result_int _result_int;

struct IError {
	union {
		void* _object;
		None__* _None__;
		voidptr* _voidptr;
		Error* _Error;
		MessageError* _MessageError;
	};
	int _typ;
	string* msg;
	int* code;
};

struct _option_int { //整型的选项类型
	byte state;
	IError err;
	byte data[sizeof(int) > 1 ? sizeof(int) : 1];
};

struct _result_int { //整型的错误类型
	bool is_error;
	IError err;
	byte data[sizeof(int) > 1 ? sizeof(int) : 1];
};

_option_int main__return_type_or_none(int x) {
	switch (x) {
		case 0: {
				return (_option_int){ .state=2, .err=_const_none__, .data={EMPTY_STRUCT_INITIALIZATION} };
		}
		default: {
				_option_int _t2;
				_option_ok(&(int[]) { x }, (_option*)(&_t2), sizeof(int));
				return _t2;
		}
	}
	
	return (_option_int){0};
}

_result_int main__return_type_or_error(int x) {
	switch (x) {
		case 0: {
				return (_result_int){ .is_error=true, .err=_v_error(_SLIT("error: x can not be 0")), .data={EMPTY_STRUCT_INITIALIZATION} };
		}
		default: {
				_result_int _t2;
				_result_ok(&(int[]) { x }, (_result*)(&_t2), sizeof(int));
				return _t2;
		}
	}
	
	return (_result_int){0};
}

VV_LOCAL_SYMBOL void main__main(void) {
	_option_int _t1 = main__return_type_or_none(0);
	if (_t1.state != 0) {
		IError err = _t1.err;
		int_literal _t2 = 0;
		if (err._typ == _IError_None___index) {
			_t2 = 10;
		}
		
		else {
			_v_panic(IError_str(err));
			VUNREACHABLE();
		}
		*(int*) _t1.data = _t2;
	}
	
 	int v1 =  (*(int*)_t1.data);
	println(int_str(v1));
	_option_int _t3 = main__return_type_or_none(1);
	if (_t3.state != 0) {
		IError err = _t3.err;
		int_literal _t4 = 0;
		if (err._typ == _IError_None___index) {
			_t4 = 10;
		}
		
		else {
			_v_panic(IError_str(err));
			VUNREACHABLE();
		}
		*(int*) _t3.data = _t4;
	}
	
 	int v2 =  (*(int*)_t3.data);
	println(int_str(v2));
	_result_int _t5 = main__return_type_or_error(2);
	if (_t5.is_error) {
		IError err = _t5.err;
		_v_panic(IError_str(err));
		VUNREACHABLE();
	;
	}
	
 	int v3 =  (*(int*)_t5.data);
	println(int_str(v3));
	_result_int _t6 = main__return_type_or_error(0);
	if (_t6.is_error) {
		IError err = _t6.err;
		_v_panic(IError_str(err));
		VUNREACHABLE();
	;
	}
	
 	int v4 =  (*(int*)_t6.data);
	println(int_str(v4));
}
```

#### 联合类型

联合类型，使用C结构体，结构体中包含一个匿名联合体，以及类型id。并且为联合类型的每一个类型，自动生成一个创建函数。

V代码：

```v
struct User {
	name string
	age  int
}

pub fn (m &User) str() string {
	return 'name:${m.name},age:${m.age}'
}

type MySum = User | int | string //联合类型声明

pub fn add(ms MySum) {
	match ms {
		int {
			println('ms is int,value is ${ms.str()}')
		}
		string {
			println('ms is string,value is ${ms}')
		}
		User {
			println('ms is User,value is ${ms.str()}')
		}
	}
}

pub fn main() {
	i := 1
	add(i)
	s := 'abc'
	add(s)
	u := User{
		name: 'n'
		age: 10
	}
	add(u)
}
```

C代码：

```c
typedef struct main__MySum main__MySum;
void main__add(main__MySum ms);

struct main__User {
	string name;
	int age;
};

struct main__MySum { //联合类型结构体
	union {  //匿名联合体
		main__User* _main__User;
		int* _int;
		string* _string;
	};
	int _typ; //类型id
};

string main__User_str(main__User* m) {
	string _t1 =  str_intp(3, _MOV((StrIntpData[]){{_SLIT("name:"), /*115 &string*/0xfe10, {.d_s = m->name}}, {_SLIT(",age:"), /*100 &int*/0xfe07, {.d_i32 = m->age}}, {_SLIT0, 0, { .d_c = 0 }}}));
	return _t1;
}

void main__add(main__MySum ms) {
	if (ms._typ == 7 /* int */) {
		println( str_intp(2, _MOV((StrIntpData[]){{_SLIT("ms is int,value is "), /*115 &string*/0xfe10, {.d_s = int_str((*ms._int))}}, {_SLIT0, 0, { .d_c = 0 }}})));
	}
	else if (ms._typ == 20 /* string */) {
		println( str_intp(2, _MOV((StrIntpData[]){{_SLIT("ms is string,value is "), /*115 &string*/0xfe10, {.d_s = (*ms._string)}}, {_SLIT0, 0, { .d_c = 0 }}})));
	}
	else if (ms._typ == 94 /* main.User */) {
		println( str_intp(2, _MOV((StrIntpData[]){{_SLIT("ms is User,value is "), /*115 &string*/0xfe10, {.d_s = main__User_str(&(*ms._main__User))}}, {_SLIT0, 0, { .d_c = 0 }}})));
	}
	
}

//联合类型有几个类型，就有几个创建函数，统一转换成联合体类型
//整型的创建函数
static inline main__MySum int_to_sumtype_main__MySum(int* x) {
	int* ptr = memdup(x, sizeof(int));
	return (main__MySum){ ._int = ptr, ._typ = 7};
}

//字符串类型的创建函数
static inline main__MySum string_to_sumtype_main__MySum(string* x) {
	string* ptr = memdup(x, sizeof(string));
	return (main__MySum){ ._string = ptr, ._typ = 20};
}
//User类型的创建函数
static inline main__MySum main__User_to_sumtype_main__MySum(main__User* x) {
	main__User* ptr = memdup(x, sizeof(main__User));
	return (main__MySum){ ._main__User = ptr, ._typ = 94};
}

void main__main(void) {
	int i = 1;
	main__add(int_to_sumtype_main__MySum(&i));
	string s = _SLIT("abc");
	main__add(string_to_sumtype_main__MySum(&s));
	main__User u = ((main__User){.name = _SLIT("n"),.age = 10,});
	main__add(main__User_to_sumtype_main__MySum(&u));
}
```

#### 运算符重载

编译时，将重载运算符，转换成普通C函数。

V代码：

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
	return '{${a.x}, ${a.y}}'
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
	println('a+=b is: ${a}')
	a -= b
	println('a-=b is: ${a}')
	a *= b
	println('a*=b is: ${a}')
	a /= b
	println('a/=b is: ${a}')
	//比较运算符
	println(a == b) // false
	println(a != b) // true
	println(a > b) // true
	println(a >= b) // true
	println(a < b) // false
	println(a <= b) // false
}
```

C代码：

```c
typedef struct main__Vec main__Vec;

struct main__Vec {
	int x;
	int y;
};
//将重载运算符，转换成普通C函数
main__Vec main__Vec__plus(main__Vec a, main__Vec b) {
	main__Vec _t1 = ((main__Vec){.x = a.x + b.x,.y = a.y + b.y,});
	return _t1;
}
main__Vec main__Vec__minus(main__Vec a, main__Vec b) {
	main__Vec _t1 = ((main__Vec){.x = a.x - b.x,.y = a.y - b.y,});
	return _t1;
}
main__Vec main__Vec__mult(main__Vec a, main__Vec b) {
	main__Vec _t1 = ((main__Vec){.x = a.x * b.x,.y = a.y * b.y,});
	return _t1;
}
main__Vec main__Vec__div(main__Vec a, main__Vec b) {
	main__Vec _t1 = ((main__Vec){.x = a.x / b.x,.y = a.y / b.y,});
	return _t1;
}
VV_LOCAL_SYMBOL main__Vec main__Vec__mod(main__Vec a, main__Vec b) {
	main__Vec _t1 = ((main__Vec){.x = a.x % b.x,.y = a.y % b.y,});
	return _t1;
}
VV_LOCAL_SYMBOL bool main__Vec__eq(main__Vec a, main__Vec b) {
	bool _t1 = a.x == b.x && a.y == b.y;
	return _t1;
}
VV_LOCAL_SYMBOL bool main__Vec__lt(main__Vec a, main__Vec b) {
	bool _t1 = a.x < b.x && a.y < b.y;
	return _t1;
}

VV_LOCAL_SYMBOL string main__Vec_str(main__Vec a) {
	string _t1 =  str_intp(3, _MOV((StrIntpData[]){{_SLIT("{"), /*100 &int*/0xfe07, {.d_i32 = a.x}}, {_SLIT(", "), /*100 &int*/0xfe07, {.d_i32 = a.y}}, {_SLIT("}"), 0, { .d_c = 0 }}}));
	return _t1;
}

//主函数
VV_LOCAL_SYMBOL void main__main(void) {
	main__Vec a = ((main__Vec){.x = 8,.y = 15,});
	main__Vec b = ((main__Vec){.x = 4,.y = 5,});
	println(main__Vec_str(main__Vec__plus(a, b)));
	println(main__Vec_str(main__Vec__minus(a, b)));
	println(main__Vec_str(main__Vec__mult(a, b)));
	println(main__Vec_str(main__Vec__div(a, b)));
	println(main__Vec_str(main__Vec__mod(a, b)));
	a = main__Vec__plus(a, b);
	println( str_intp(2, _MOV((StrIntpData[]){{_SLIT("a+=b is: "), /*115 &main.Vec*/0xfe10, {.d_s = main__Vec_str(a)}}, {_SLIT0, 0, { .d_c = 0 }}})));
	a = main__Vec__minus(a, b);
	println( str_intp(2, _MOV((StrIntpData[]){{_SLIT("a-=b is: "), /*115 &main.Vec*/0xfe10, {.d_s = main__Vec_str(a)}}, {_SLIT0, 0, { .d_c = 0 }}})));
	a = main__Vec__mult(a, b);
	println( str_intp(2, _MOV((StrIntpData[]){{_SLIT("a*=b is: "), /*115 &main.Vec*/0xfe10, {.d_s = main__Vec_str(a)}}, {_SLIT0, 0, { .d_c = 0 }}})));
	a = main__Vec__div(a, b);
	println( str_intp(2, _MOV((StrIntpData[]){{_SLIT("a/=b is: "), /*115 &main.Vec*/0xfe10, {.d_s = main__Vec_str(a)}}, {_SLIT0, 0, { .d_c = 0 }}})));
	println(main__Vec__eq(a, b) ? _SLIT("true") : _SLIT("false"));
	println(!main__Vec__eq(a, b) ? _SLIT("true") : _SLIT("false"));
	println(main__Vec__lt(b, a) ? _SLIT("true") : _SLIT("false"));
	println(!main__Vec__lt(a, b) ? _SLIT("true") : _SLIT("false"));
	println(main__Vec__lt(a, b) ? _SLIT("true") : _SLIT("false"));
	println(!main__Vec__lt(b, a) ? _SLIT("true") : _SLIT("false"));
}
```

#### 条件编译

##### 判断操作系统

V代码：

```v
fn main() {
	$if windows {
		println('windows')
	}
	$if linux {
		println('linux')
	}
	$if macos {
		println('mac')
	}
}
```

C代码：

```c
VV_LOCAL_SYMBOL void main__main(void) {
	println(_SLIT("mac"));
}
```

##### 判断操作系统位数

V代码：

```v
fn main() {
	mut x := 0
	$if x32 {
		println('system is 32 bit')
		x = 1
	}
	$if x64 {
		println('system is 64 bit')
		x = 2
	}
}

```

C代码：

```c
#if INTPTR_MAX == INT32_MAX
	#define TARGET_IS_32BIT 1
#elif INTPTR_MAX == INT64_MAX
	#define TARGET_IS_64BIT 1
#else
	#error "The environment is not 32 or 64-bit."
#endif

VV_LOCAL_SYMBOL void main__main(void) {
	int x = 0;
	#if defined(TARGET_IS_32BIT)
	{
		println(_SLIT("system is 32 bit"));
		x = 1;
	}
	#endif
	#if defined(TARGET_IS_64BIT)
	{
		println(_SLIT("system is 64 bit"));
		x = 2;
	}
	#endif
}
```

##### 判断大端序/小端序

V代码：

```v
fn main() {
	mut x := 0
	$if little_endian {
		println('system is little endian')
		x = 1
	}
	$if big_endian {
		println('system is big endian')
		x = 2
	}
}

```

C代码：

```c
#if defined(__BYTE_ORDER__) && __BYTE_ORDER__ == __ORDER_BIG_ENDIAN__ || defined(__BYTE_ORDER) && __BYTE_ORDER == __BIG_ENDIAN || defined(__BIG_ENDIAN__) || defined(__ARMEB__) || defined(__THUMBEB__) || defined(__AARCH64EB__) || defined(_MIBSEB) || defined(__MIBSEB) || defined(__MIBSEB__)
	#define TARGET_ORDER_IS_BIG 1
#elif defined(__BYTE_ORDER__) && __BYTE_ORDER__ == __ORDER_LITTLE_ENDIAN__ || defined(__BYTE_ORDER) && __BYTE_ORDER == __LITTLE_ENDIAN || defined(__LITTLE_ENDIAN__) || defined(__ARMEL__) || defined(__THUMBEL__) || defined(__AARCH64EL__) || defined(_MIPSEL) || defined(__MIPSEL) || defined(__MIPSEL__) || defined(_M_AMD64) || defined(_M_X64) || defined(_M_IX86)
	#define TARGET_ORDER_IS_LITTLE 1
#else
	#error "Unknown architecture endianness"
#endif

VV_LOCAL_SYMBOL void main__main(void) {
	int x = 0;
	#if defined(TARGET_ORDER_IS_LITTLE)
	{
		println(_SLIT("system is little endian"));
		x = 1;
	}
	#endif
	#if defined(TARGET_ORDER_IS_BIG)
	{
		println(_SLIT("system is big endian"));
		x = 2;
	}
	#endif
}
```

#### 内联汇编代码

生成等价的C内联汇编代码：

V代码：

```v
fn main() {
	a := 100
	b := 20
	mut c := 0
	asm amd64 {
		mov eax, a
		add eax, b
		mov c, eax
		; =r (c) // output
		; r (a) // input
		  r (b)
	}
	println('a: ${a}') // 100
	println('b: ${b}') // 20
	println('c: ${c}') // 120
}
```

C代码：

```c
VV_LOCAL_SYMBOL void main__main(void) {
	int a = 100;
	int b = 20;
	int c = 0;
	__asm__ (
		"mov %[a], %%eax;"
		"add %[b], %%eax;"
		"mov %%eax, %[c];"
		: [c] "=r" (c)
		: [a] "r" (a),
		[b] "r" (b)
	);
	println( str_intp(2, _MOV((StrIntpData[]){{_SLIT("a: "), /*100 &int*/0xfe07, {.d_i32 = a}}, {_SLIT0, 0, { .d_c = 0 }}})));
	println( str_intp(2, _MOV((StrIntpData[]){{_SLIT("b: "), /*100 &int*/0xfe07, {.d_i32 = b}}, {_SLIT0, 0, { .d_c = 0 }}})));
	println( str_intp(2, _MOV((StrIntpData[]){{_SLIT("c: "), /*100 &int*/0xfe07, {.d_i32 = c}}, {_SLIT0, 0, { .d_c = 0 }}})));
}
```
