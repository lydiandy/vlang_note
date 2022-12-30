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

V代码：

```v

```

C代码：

```c

```

#### 泛型

V代码：

```v

```

C代码：

```c

```

#### 错误处理

V代码：

```v

```

C代码：

```c

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
	int _typ;
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

V代码：

```v

```

C代码：

```c

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
