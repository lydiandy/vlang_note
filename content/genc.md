## 编译生成C代码

编译器本质上就是一门语言生成另一门语言的过程

**V目前的编译思路是:编译生成对应的C代码,然后调用C编译器来生成可执行文件**

V语言的开发重点在编译器前端,C就是编译器后端

现在也生成了js代码,估计也会生成wasm代码

或者考虑基于LLVM,生成LLVM IR

或者直接生成机器码

------

### 查看生成的C代码

想要查看V代码生成的C代码,只要增加-o参数就可以了,编译器只会生成C代码

比如,想把当前目录的main.v代码生成main.c代码,执行以下命令就可以:

```
v -o main.c ./main.v
```

通过查看V代码生成的C代码,可以更容易理解V编译器是如何编译的

### 基本类型对应

v的基本类型通过C的类型别名typedef来实现

```C
//int类型没有类型别名,直接就是C的int类型
typedef int64_t i64;
typedef int16_t i16;
typedef int8_t i8;
typedef uint64_t u64;
typedef uint32_t u32;
typedef uint16_t u16;
typedef uint8_t byte;
typedef uint32_t rune;
typedef float f32;
typedef double f64;
typedef unsigned char* byteptr; //字节指针
typedef int* intptr; //整型指针
typedef void* voidptr; //通用指针
typedef struct array array;
typedef struct map map;
typedef array array_string;
typedef array array_int;
typedef array array_byte;
typedef array array_f32;
typedef array array_f64;
typedef map map_int;
typedef map map_string;
#ifndef bool
	typedef int bool; //布尔类型在C里面通过int类型来实现,4字节
	#define true 1 //true是整数常量1
	#define false 0 //false是整数常量0
#endif

```

### V代码=>C代码对照表

#### 常量

int,bool类型,生成C的宏定义:

```c
//V代码
const (
	i=1 //int类型的常量
    b=true //booll类型的常量
)

//C代码
#define main__i  1
#define main__b  1
```

其他类型生成C的全局变量,常量的不可修改,由V编译器负责检查

```c
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
)
//C代码
i8 main__a;
i16 main__b;
i64 main__c;
byte main__d;
u16 main__e;
u32 main__f;
u64 main__g;
f32 main__h;
f64 main__i;
main__a =  ((i8)( 11 ) );
main__b =  ((i16)( 12 ) );
main__c =  ((i64)( 13 ) );
main__d =  ((byte)( 14 ) );
main__e =  ((u16)( 15 ) );
main__f =  ((u32)( 16 ) );
main__g =  ((u64)( 17 ) );
main__h =  ((f32)( 1.1 ) );
main__i =  ((f64)( 1.2 ) );
```

#### 枚举

```c
//V代码
enum Color {
	blue = 1
	green
	red
}
c := Color.blue
//C代码
typedef int Color;

#define main__Color_blue 1 
#define main__Color_green 2
#define main__Color_red 3

Color c= main__Color_blue ;
```

#### 模块

V的模块,在生成对应C代码后,只是对应元素名称的前缀,毕竟C语言中没有模块的概念

常量,结构体,接口,类型等一级元素生成C代码后的名称规则是:"模块名__名称",用双下划线区隔

例如主模块中的add()函数生成C代码后的名称为:main__add()

结构体的方法等二级元素生成C代码后的名称规则是:"模块名 __ 类名 _ 方法名",用单下划线区隔

例主模块中的Color结构体的str()方法生成C代码后的名称为:main__Color_str()

#### 函数

生成等价的C的函数

```c
//V代码
pub fn add(x,y int) int { //pub的模块访问控制由V编译器负责检查,C没有pub的对应
	if x>0 {
		return x+y
	} else {
		return x+y
	}
}
//C代码
int main__add (int x, int y); //函数声明

int main__add(int x, int y) {
  if (x > 0) {
    return x + y;
  } else {
    return x + y;
  };
}
```

函数defer语句

C没有defer语句,V编译的时候就是把函数中的defer语句去掉,然后按后进先出的顺序放在函数末尾,有各自独立的代码块

```c
//V代码
fn main(){
    println('main start')
    
    defer {defer_fn1()} 
    defer {defer_fn2()}
    
    println('main end')
}

fn defer_fn1(){
    println('from defer_fn1')
}

fn defer_fn2(){
    println('from defer_fn2')
}
//C代码
 void main__main() {
   println(tos3("main start"));
   println(tos3("main end"));
   { main__defer_fn2(); }
   { main__defer_fn1(); }
 }
 void main__defer_fn1() { println(tos3("from defer_fn1")); }
 void main__defer_fn2() { println(tos3("from defer_fn2")); }
```

函数不确定个数参数

不确定参数就是把最后一个参数变为指针类型,指向最后一个数组参数

```c
//V代码
fn my_fn(i int,s string, others ...string) {
    println(i)
    println(s)
    println(others[0])
    println(others[1])
    println(others[2])
}

fn main() {
    my_fn(1,'abc','de','fg','hi')
}
//C代码
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

函数多返回值

C的函数返回值只有1个,V的函数多返回值,就是把多返回值的组合,生成一个结构体,然后返回结构体

```c
//V代码
fn foo() (int, int) { //多返回值
	return 2, 3
}

fn some_multiret_fn(a int, b int) (int, int) {
	return a+1, b+1 //可以返回表达式
}
fn main() {
	a, b := foo()
	println(a) // 2
	println(b) // 3
}
//C代码
typedef struct _V_MulRet_int_V_int _V_MulRet_int_V_int;
struct _V_MulRet_int_V_int { //生成的返回值组合的结构体,还可以给其他函数共用
	int var_0;
	int var_1;
};

 _V_MulRet_int_V_int main__foo() {
   return (_V_MulRet_int_V_int){.var_0 = 2, .var_1 = 3};
 }
 _V_MulRet_int_V_int main__some_multiret_fn(int a, int b) {
   return (_V_MulRet_int_V_int){.var_0 = a + 1, .var_1 = b + 1};
 }
 void main__main() {
   _V_MulRet_int_V_int _V_mret_49_a_b = main__foo();
   int a = _V_mret_49_a_b.var_0;
   int b = _V_mret_49_a_b.var_1;
   /*opt*/ printf("%d\n", a);
   /*opt*/ printf("%d\n", b);
 }
```



#### 数组

V的数组是用struct来实现的,生成C代码也是struct

```c
//V代码
fn main() {
	a:=[1,3,5]
	b:=['a','b','c']
	println(a)
	println(b)
}
//C代码
//第一部分: 从内置的vlib/built/array.v生成
struct array {
	void* data;
	int len;
	int cap;
	int element_size;
};
//第二部分:从内置的vlib/built/array.v生成
typedef struct array array;
typedef array array_string;
typedef array array_int;
typedef array array_byte;
typedef array array_f32;
typedef array array_f64;
typedef array array_u16;
typedef array array_u32;
typedef array array_u64;
//第三部分:数组使用的代码,字面量方式创建
array_int a=new_array_from_c_array(3, 3, sizeof(int), EMPTY_ARRAY_OF_ELEMS( int, 3 ) {  1 ,  3 ,  5  }) ;
array_string b=new_array_from_c_array(3, 3, sizeof(string), EMPTY_ARRAY_OF_ELEMS( string, 3 ) {  tos3("a") ,  tos3("b") ,  tos3("c")  }) ;
 println (array_int_str( a ) ) ;
 println (array_string_str( b ) ) ;
 }
```

#### 字符串

V的字符串是用struct来实现的,生成C代码也是struct

```c
//V代码
//vlib/builtin/string.v
pub struct string {
	// mut:
	// hash_cache int
pub:
	str byteptr // points to a C style 0 terminated string of bytes.
	len int // the length of the .str field, excluding the ending 0 byte. It is always equal to strlen(.str).
}
//string的各种内置方法:
...
//使用代码:
fn main(){
	mystr:='abc'
	mystr2:="def"
	println(mystr)
	println(mystr2)
}

//C代码
//第一部分:vlib/builtin/string.v生成:
typedef struct string string;
struct string {
	byte* str;
	int len;
};
//string的各种内置方法生成:
string string_left (string s, int n);
string string_right (string s, int n);
string string_substr2 (string s, int start, int _end, bool end_max);
string string_substr (string s, int start, int end);

//第二部分,使用代码:
void main__main () {
string mystr= tos3("abc") ;
string mystr2= tos3("def") ;
 println ( mystr ) ;
 println ( mystr2 ) ;
 }
```

#### 字典

V的字典是用struct来实现的,生成C代码也是struct

```c
//V代码
//第一部分:定义map
pub struct map {
	element_size int
	root         &mapnode
pub:
	size         int
}

struct mapnode {
	left     &mapnode
	right    &mapnode
	is_empty bool // set by delete()
	key      string
	val      voidptr
}
//其他map内置方法
...
//第二部分:使用map
fn main() {
    mut m := map[string]int
    m['one'] = 1
    m['two'] = 2
    println(m['one']) 
    println(m['bad_key'])
}

//C代码
//第一部分:定义map
typedef struct map map;
typedef struct mapnode mapnode;

typedef map map_int;
typedef map map_string;

struct map {
	int element_size;
	mapnode* root;
	int size;
};
struct mapnode {
	mapnode* left;
	mapnode* right;
	bool is_empty;
	string key;
	void* val;
};

//map的各种内置方法
...
//第二部分:使用map:
void main__main () {
	map_int m= new_map(1, sizeof(int)) ;
	map_set(& m , tos3("one") , & (int []) {  1 }) ;
	map_set(& m , tos3("two") , & (int []) {  2 }) ; 
 	int tmp1 = 0; bool tmp2 = map_get(m , tos3("one"), & tmp1); 

 	printf ("%d\n",  tmp1 ) ; 
 	int tmp3 = 0; bool tmp4 = map_get(m , tos3("bad_key"), & tmp3); 

 	printf ("%d\n",  tmp3 ) ;
 }

```

#### 结构体

生成对应的C结构体

```c
//V代码
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

//C代码
struct Point {
	int x;
	int y;
};
string Point_str (Point p); //函数声明

string Point_str (Point p) { //函数定义,默认第一个参数是自己
	return  _STR("x is %d,y is:%d", p .x, p .y) ;
}
void main__main () {
	Point p= (Point) { .x =  1 , .y =  3 } ;
 	println (Point_str( p ) ) ;
}
```

#### 结构体方法

生成对应的C函数,默认第一个参数是对应的结构体类型的指针

生成的C函数的命名规则是:"结构体名_方法名"

```c
//V代码
pub fn (a mut array) insert(i int, val voidptr) {
	if i < 0 || i > a.len {
		panic('array.insert: index out of range (i == $i, a.len == $a.len)')
	}
	a.ensure_cap(a.len + 1)
	size := a.element_size
	C.memmove(a.data + (i + 1) * size, a.data + i * size, (a.len - i) * size)
	C.memcpy(a.data + i * size, val, size)
	a.len++
}
//C代码
void array_insert(array *a, int i, void *val) { //默认第一个参数是对应类型指针
  if (i < 0 || i > a->len) {
    v_panic(_STR("array.insert: index out of range (i == %d, a.len == %d)", i,
                 a->len));
  };
  array_ensure_cap(a, a->len + 1);
  int size = a->element_size;
  memmove((byte *)a->data + (i + 1) * size, (byte *)a->data + i * size,
          (a->len - i) * size);
  memcpy((byte *)a->data + i * size, val, size);
  a->len++;
}
```

#### 结构体访问控制

访问控制在C代码中没有体现,全部在V编译器中控制

```c
//V代码
struct Foo {
	a int     //私有,不可变(默认).在模块内部可访问,不可修改;模块外不可访问,不可修改
mut: 
	b int     // 私有,可变.在模块内部可访问,可修改,模块外部不可访问,不可修改
	c int     // (相同访问控制的字段可以放在一起)   
pub: 
	d int   // 公共,不可变,只读.在模块内部和外部都可以访问,但是不可修改
pub mut: 
	e int  //公共,模块内部可访问,可修改;模块外部可访问,但是不可修改
__global:
	f int // 全局字段,模块内部和外部都可访问,可修改,这样等于破坏了封装性,不推荐使用
}    

fn main() {
	f:=Foo{}
	println(f)
}

//C代码
typedef struct Foo Foo;
struct Foo {
	int a;
	int b;
	int c;
	int d;
	int e;
	int f;
};
string Foo_str();
void main__main() {
   Foo f = (Foo){.a = 0, .b = 0, .c = 0, .d = 0, .e = 0, .f = 0};
   println(Foo_str(f));
 }
string Foo_str(Foo a) {
   return _STR("{\n	a: %d\n	b: %d\n	c: %d\n	d: %d\n	e: %d\n	f: %d\n}", a.a,a.b, a.c, a.d, a.e, a.f);
 }
```

#### 流程控制语句

if语句,生成C if语句

```c
//V代码
fn main() {
	a := 10
	b := 20
	if a < b {
		println('$a < $b')
	}
	else if a > b {
		println('$a > $b')
	}
	else {
		println('$a == $b')
	}
}

//C代码
void main__main () {
	int a= 10 ;
	int b= 20 ;
	 if ( a < b ) {
		printf( "%d < %d\n", a, b ) ;
 	}
 	 else  if ( a > b ) {
		printf( "%d > %d\n", a, b ) ;
	 }
 	 else { 
		printf( "%d == %d\n", a, b ) ;
}
```

if表达式语句,生成C的三元运算符 ? :

```c
//V代码
fn main() {
	num := 777
	s := if num % 2 == 0 {
	'even'
	}
	else {
	'odd'
	}
	println(s) // "odd"
}
//C代码
void main__main() {
  int num = 777;
  string s = ((num % 2 == 0) ? (tos3("even")) : (tos3("odd")));
  println(s);
}
```

match语句

```c
//V代码
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
}
//C代码
void main__main() {
  string os = tos3("macos");
  string tmp1 = os;

  if (string_eq(tmp1, tos3("windows"))) {
    println(tos3("windows"));
  } else if (string_eq(tmp1, tos3("linux"))) {
    println(tos3("linux"));
  } else if (string_eq(tmp1, tos3("macos"))) {
    println(tos3("macos"));
  } else // default:
  {
    println(tos3("unknow"));
  };
}
```

match表达式语句

```c
//V代码
fn main() {
os:='macos'
price:=match os {
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

//C代码 
//生成嵌套的三元表达式
void main__main() {
  string os = tos3("macos");
  string tmp1 = os;

  int price = ((string_eq(tmp1, tos3("windows")))
                   ? (100)
                   : ((string_eq(tmp1, tos3("linux")))
                          ? (120)
                          : ((string_eq(tmp1, tos3("macos"))) ? (150) : (0))));
  /*opt*/ printf("%d\n", price);
}
```

for循环

传统的：for i=0;i<100;i++ {}

```c
//V代码
fn main() {
  for i := 0; i < 10; i++ { 
   	//跳过6
   	if i == 6 {
   		continue
   	}
   	println(i)
   }
}

//C代码
void main__main() {
  for (int i = 0; i < 10; i++) {

    if (i == 6) {
      continue;
    };
    /*opt*/ printf("%d\n", i);
  };
}
```

替代while：for i<100 {}

```c
//V代码
fn main() {
   mut sum := 0
   mut i := 0
   for i <= 100 {
   	sum += i
   	i++
   }
   println(sum) // 输出"5050"
}
//C代码
void main__main () {
	int sum= 0 ;
	int i= 0 ;
	 while ( i <= 100 ) {
 
 	sum  +=  i ;
 	i ++ ;
 }
```

无限循环：for {}

```c
//V代码
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
//C代码
 void main__main() {
   int num = 0;
   while (1) {
     num++;
     if (num >= 10) {
       break;
     };
   };
   /*opt*/ printf("%d\n", num);
 }
```

遍历：for i in xxx {}

```c
//V代码
fn main() {
	numbers := [1, 2, 3, 4, 5]
	for i,num in numbers {
		println('i:$i,num:$num')
	}
}
//C代码
void main__main() {
   array_int numbers = new_array_from_c_array(
       5, 5, sizeof(int), EMPTY_ARRAY_OF_ELEMS(int, 5){1, 2, 3, 4, 5});
   array_int tmp1 = numbers;
   for (int i = 0; i < tmp1.len; i++) {
     int num = ((int *)tmp1.data)[i];

     printf("i:%d,num:%d\n", i, num);
   };
 }
```

#### 类型定义

类型定义type生成C的类型别名typedef

```c
//V代码
pub struct Point {
	x int
	y int
}
type myint int
type myPoint Point
   
//C代码
struct Point {
	int x;
	int y;
};
typedef  int myint;
typedef  Point myPoint;
```

#### 接口

```c
//V代码

//C代码
```

#### 泛型

```c
//V代码

//C代码
```

#### 错误处理

```c
//V代码

//C代码
```

#### 联合类型

```c
//V代码

//C代码
```

#### 运算符重载

```c
//V代码

//C代码
```

#### 条件编译

生成等价的C宏定义

```c
//V代码
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
//C代码
void main__main() {
#ifdef _WIN32
#endif
   ;
#ifdef __linux__
#endif
   ;
#ifdef __APPLE__
   println(tos3("mac"));
#endif
   ;
 }
```

```c
//V代码
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

//C代码
void main__main() {
   int x = 0;
	#ifdef TARGET_IS_32BIT
  	 println(tos3("system is 32 bit"));
   	x = 1;
	#endif
   	;
	#ifdef TARGET_IS_64BIT
   	println(tos3("system is 64 bit"));
   	x = 2;
	#endif
   	;
```

```c
//V代码
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

//C代码
 void main__main() {
   int x = 0;
	#ifdef TARGET_ORDER_IS_LITTLE
   println(tos3("system is little endian"));
   x = 1;
	#endif
   ;
	#ifdef TARGET_ORDER_IS_BIG
   println(tos3("system is big endian"));
   x = 2;
	#endif
   ;
```

#### 内联汇编代码

生成等价的C内联汇编的代码

```c
//V代码
fn main() {
	a := 10
	b := 0
	unsafe {	//unsafe代码块
		asm {	//asm代码块,里面可以直接写汇编代码
			"movl %1, %%eax;"
			"movl %%eax, %0;"
			:"=r"(b)
			:"r"(a)
			:"%eax"
		}
	}
	println(a) //返回10
	println(b) //返回10,直接通过汇编代码修改了b的值

	e := 0
	unsafe {
		asm {
			"movl $5, %0"
			:"=a"(e)
		}
	}
	println(e) //返回5,直接通过汇编代码修改了e的值
}

//C代码
 void main__main() {
   int a = 10;
   int b = 0;
   {
     asm("movl %1, %%eax;"
         "movl %%eax, %0;"
         : "=r"(b)
         : "r"(a)
         : "%eax");
     ;
   };
   /*opt*/ printf("%d\n", a);
   /*opt*/ printf("%d\n", b);
   int e = 0;
   {
     asm("movl $5, %0" : "=a"(e));
     ;
   };
   /*opt*/ printf("%d\n", e);
 }
```

