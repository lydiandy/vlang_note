## 集成C代码库

### 优势

V的代码库很多都直接调用C标准库函数来实现，对C标准库的依赖还是很重的，

由于V代码编译后生成的是C代码，然后再调用C编译器编译成可执行文件，

这样的机制决定了V语言可以很方便地调用C世界的各种代码库，

这对于V语言来说，是个很大的一个优势，毕竟C代码库经过多年的积累，很丰富。

### 如何调用

在V代码里调用C代码也非常简单，在V标准库里随处可见，以下是调用C代码库的基本步骤：

1. **定义#flag**

    定义flag这一步是可选的，比如调用C标准库的函数就不需要，一般调用第三方库才需要用到，flag的定义要放在C语言宏之前。

2. **在V代码中使用C语言宏**

    比如#include宏或#define宏，编译时这些宏会被原封不动地搬到生成的C代码中。


3. **用V的语法定义要使用的C函数或C结构体的声明**

    函数名或结构体名一定要在C名称的基础上添加C.前缀。


4. **然后就可以在V代码中使用C函数或结构体**

    调用时，名称前要使用C.作为前缀。

    原理其实很简单：V编译时，会统一把函数和结构体名称前面的C.前缀统一去掉，这样在C代码里面就可以正常调用了。


调用标准库的例子：

```v
module main
    
#include <stdio.h>  //使用C语言宏，包含头文件

fn C.getpid() int //先定义要使用的C函数声明，名字在C函数名字前统一加上C.前缀
    
fn main(){
	pid:=C.getpid() //然后就可以在V代码中调用
	println(pid)
}
```

myslq库中的参考代码：

```v
//宏
#flag -lmysqlclient
#flag linux -I/usr/include/mysql
#include <mysql.h>
//声明要使用的C结构体
struct C.MYSQL
struct C.MYSQL_RES
//声明要使用的C函数
fn C.mysql_init(mysql &C.MYSQL) &C.MYSQL
fn C.mysql_real_connect(mysql &C.MYSQL, host &byte, user &byte, passwd &byte, db &byte, port u32, unix_socket &byte, clientflag u64) &C.MYSQL
fn C.mysql_query(mysql &C.MYSQL, q &byte) int
fn C.mysql_select_db(mysql &C.MYSQL, db &byte) int
fn C.mysql_error(mysql &C.MYSQL) &byte
fn C.mysql_errno(mysql &C.MYSQL) int
fn C.mysql_num_fields(res &C.MYSQL_RES) int
fn C.mysql_store_result(mysql &C.MYSQL) &C.MYSQL_RES
fn C.mysql_fetch_row(res &C.MYSQL_RES) &&byte
fn C.mysql_free_result(res &C.MYSQL_RES)
fn C.mysql_real_escape_string_quote(mysql &C.MYSQL, to &byte, from &byte, len u64, quote byte) u64
fn C.mysql_close(sock &C.MYSQL)

//调用C函数或结构体
pub fn connect(server, user, passwd, dbname string) ?DB {
	conn := C.mysql_init(0)
	if isnil(conn) {
		return error_with_code(get_error_msg(conn), get_errno(conn))
	}
	conn2 := C.mysql_real_connect(conn, server.str, user.str, passwd.str, dbname.str, 0, 0, 0)
	if isnil(conn2) {
		return error_with_code(get_error_msg(conn), get_errno(conn))
	}
	return DB {conn: conn2}
}
```

另一个集成C代码库的例子：vlib/clipboard/clipboard_linux.c.v。

使用了结构体注解[typedef]来定义C语言的结构体。

```v
//定义C宏
#flag -lX11
#include <X11/Xlib.h>

//定义C结构体
[typedef]
struct C.Display  //在v代码中只需使用C结构体，不使用结构体字段
  
[typedef]
struct C.XSelectionRequestEvent{ //在v代码中要使用结构体字段，要定义所需的字段
	mut:
	display &C.Display	/* Display the event was read from */
	owner C.Window
	requestor C.Window
	selection C.Atom
	target C.Atom
	property C.Atom
	time int
}

//定义C函数
fn C.XInitThreads() int
fn C.XCloseDisplay(d &Display)
fn C.XFlush(d &Display)
fn C.XDestroyWindow(d &Display, w C.Window)
```

### 使用$env编译时函数

$env也可以在#flay和#include等C宏中使用，让C宏的定义更灵活。

```v
module main

//可以在C宏语句中使用,让C宏的定义更灵活
#flag linux -I $env('JAVA_HOME')/include

fn main() {
	compile_time_env := $env('PATH')
	println(compile_time_env)
}

```

### 简单封装

其实，直接在V代码中使用C函数或者结构体也是可以的。

不过，由于命名方式不一致的原因，习惯上也可以对C函数或结构体,进行一层简单封装,名字可以重新改为V风格(小写加下划线的小蛇风格)，或者更为简短的名字。

一般来说，对一个C代码库中的简单封装会涉及到：结构体，联合体，函数，枚举这4大类，经过集成和封装，就可以得到一个V风格的封装库。

如果是小的C代码库，可以直接把这3类的简单封装都放在一个V源文件中。

如果C代码库规模大一些，也可以这3类，各自单独一个V源文件，归属于同一个V模块。

以下代码是GUI代码库中引用了sokol C代码库后，进行的简单封装：

vlib/sokol/sokol.v部分代码：

```v
//只要在同模块中的任何一个V源文件中引入，该模块的其他源文件就可以直接使用C代码库内容
#define SOKOL_IMPL
#define SOKOL_NO_ENTRY
#include "sokol_app.h"

#define SOKOL_IMPL
#define SOKOL_NO_DEPRECATED
#include "sokol_gfx.h"

//可以直接使用，函数以C.作为前缀
pub fn init_sokol() {
	C.sapp_isvalid()
	C.sapp_width()
}
//也可以进行简单的封装
module sapp

[inline] //可以给函数添加inline注解，变为内联函数
pub fn isvalid() bool {
	return C.sapp_isvalid()
}

[inline]
pub fn width() int {
	return C.sapp_width()
}
```

### 启用全局变量

默认情况下编译器是没有全局变量声明的，但是为了跟C代码集成，有时候需要定义全局变量，可以在调用编译器时，通过增加 -enable-globals选项来启用。

```
v -enable-globals run main.v
```

```v
module main

// 单个全局变量定义
__global g1 int

// 组定义全局变量，类似常量的定义
__global (
	g2 byte 
	g3 byte 
)

fn main() {
	g1 = 1
  g2 = 2
  g3 = 3
	println(g1)
	println(g2)
	println(g3)
}
```

### 函数的[inline]注解

对C函数进行简单的封装时，可以给函数添加inline注解，编译生成C代码时，这个函数就会变成C语言里的static inline函数。

内联函数有些类似于宏，内联函数的代码会被直接嵌入在它被调用的地方，调用几次就嵌入几次，没有使用call指令。

这样省去了函数调用时的一些额外开销，不过调用次数多的话，会使可执行文件变大，这样会降低整个程序的运行速度。

像上面那个简单的封装，函数只有1行代码，嵌入到被调用的地方也还是1还代码，既能省去函数调用时的额外开销，提升性能，又不会使可执行文件变大。

同时也可以统一和简化C函数的命名，变为V风格的简短命名，一举多得。

```v
[inline]
pub fn width() int {
	return C.sapp_width()
}
```

### 结构体简单封装

定义C结构体等价的V版本结构体，V版本结构体名称以C.做前缀，

这样就可以直接使用V版本的结构体来创建变量，

**用V版本的结构体创建变量，编译生成的C代码中，就是用C版本结构体创建了变量**。

C代码库中的结构体：

```v
typedef struct sapp_event {
    uint64_t frame_count;
    sapp_event_type type;
    sapp_keycode key_code;
    uint32_t char_code;
    bool key_repeat;
    uint32_t modifiers;
    sapp_mousebutton mouse_button;
    float mouse_x;
    float mouse_y;
    float scroll_x;
    float scroll_y;
    int num_touches;
    sapp_touchpoint touches[SAPP_MAX_TOUCHPOINTS];
    int window_width;
    int window_height;
    int framebuffer_width;
    int framebuffer_height;
} sapp_event;
```

定义等价的V版本结构体:字段数量一样，字段名称一样，字段类型一样，

如果字段类型是枚举的，也可以再定义等价的V版本枚举，

```v
pub struct C.sapp_event {
pub:
    frame_count u64
    @type EventType //这个type字段有点特殊，因为是V的关键字，要用@开头才可以
    key_code KeyCode //枚举类型
    char_code u32
    key_repeat bool
    modifiers u32
    mouse_button MouseButton //枚举类型，下面有MouseButton的V版本定义
    mouse_x f32
    mouse_y f32
    scroll_x f32
    scroll_y f32
    num_touches int
    touches [8]sapp_touchpoint //数组类型
    window_width int
    window_height int
    framebuffer_width int
    framebuffer_height int
}
```

### 联合体简单封装

基本的原理和步骤跟结构体的封装一样，就是关键字为union。

```v
//如果不需要访问到联合体内部的字段，只是类型引用，定义一个联合体声明就可以
[typedef]
pub union C.MIR_val_t { }
//如果需要访问到联合体内部的字段，则需要展开定义
[typedef]
pub union C.MIR_val_t {
pub mut:
	a  voidptr
	i  i64
	u  u64
	f  f32
	d  f64
	ld f64
}
```

### 枚举简单封装

其实就是定义一个跟C版本枚举一样的枚举，枚举名可以按V的风格自由定义，枚举项的整数值一定要跟C版本的枚举值对应正确，其实最后都是把枚举项的值转为整数，传递给C代码使用。

```v
pub enum MouseButton {
    invalid = -1
    left = 0
    right = 1
    middle = 2
}
```

### 类型别名封装

可以对C.xxx的结构体和联合体进一步定义类型别名，这样在使用的时候，可以用更简洁的类型名

```v
pub type Context = C.MIR_context_t

pub type Module = C.MIR_module_t

pub type Item = C.MIR_item_t

pub type Func = C.MIR_func_t
```

### 关于#flag标记

这个#flag标记跟v编译器的-cflags选项的用法是一样的，用于传递额外的flag参数给C编译器。

关于C编译器的flag参数可以参考:https://colobu.com/2018/08/28/15-Most-Frequently-Used-GCC-Compiler-Command-Line-Options/。

1. 要在使用C宏之前先定义#flag。

2. `-L` 在库文件的搜索路径列表中添加指定的路径。

   `-l` 在要链接的库名称。

   `-I` 在头文件的搜索路径中添加指定的路径。

   `-D`  设置编译时变量。

3. 还可以在#flag后增加平台标识，针对不同的平台配置不同的flag，目前支持的平台有:linux，darwin，windows。

以下例子，提供参考：

```v
//#flag文档里面的:
#flag linux -lsdl2
#flag linux -Ivig
#flag linux -DCIMGUI_DEFINE_ENUMS_AND_STRUCTS=1
#flag linux -DIMGUI_DISABLE_OBSOLETE_FUNCTIONS=1
#flag linux -DIMGUI_IMPL_API=
```

```v
//mysql包里面的:
#flag -lmysqlclient //-l开头，表示在库文件的搜索路径中添加mysqlclient
#flag linux -I /usr/include/mysql //-I开头，在头文件的搜索路径中添加指定的路径， 针对linux平台，这样才可以搜索到下面的mysql.h头文件
#include <mysql.h>
```

```v
//sokol包里面的:
#flag -I @VROOT/thirdparty/sokol //@VROOT指向v编译器的根路径
#flag -I @VROOT/thirdparty/sokol/util


#flag darwin -fobjc-arc  //针对mac平台，提供额外的flag参数:-fobjc-arc
#flag linux -lX11 -lGL    //针对linux平台，提供额外的flag参数


#flag windows -lgdi32 //针对window平台，提供额外的flag参数:-l开头，表示链接gdi32


// OPENGL
#flag -DSOKOL_GLCORE33 //提供额外的flag参数:-DSOKOL_GLCORE33
#flag darwin -framework OpenGL -framework Cocoa -framework QuartzCore
```
ex:

目录结构
```
D:\LINKCTEST
│  linkcTest.code-workspace
│  linkcTest.v
│  v.mod
│
├─myCheader
│      test.h
│
└─myClib
        testlib.c
```
![目录结构](/image/linkC库Test.PNG)<br>
test.h文件内容
```
//声明自定义c函数TestFunc
int TestFunc();
```
testlib.c文件内容
```
//自定义c函数TestFunc的具体实现
int TestFunc()
{
    return 0000;
}
```
linkcTest.v文件内容
```
module main

//添加库文件搜索路径
#flag -LD:\linkcTest\myClib
//添加库文件名(前缀lib和后缀名.a不要指定)	
#flag -ltestlib
//添加头文件搜索路径
#flag -ID:\linkcTest\myCheader
//引入头文件test.h
#include <test.h>
//在v中声明一下自定义C函数
fn C.TestFunc() int

fn main() {
	println('Hello World!')
	//调用自定义C函数
	println(C.TestFunc())
}
```
首先把testlib.c文件编译为静态库
```shell
PS D:\linkcTest> gcc -c .\myClib\testlib.c -o .\myClib\testlib.o   --用gcc -c命令编译为.o文件
PS D:\linkcTest> ar -rcs .\myClib\libtestlib.a .\myClib\testlib.o  --使用ar命令打包成静态库.a文件
PS D:\linkcTest> dir .\myClib\

    Directory: D:\linkcTest\myClib

Mode                 LastWriteTime         Length Name        
----                 -------------         ------ ----        
-a---           2021/9/19    22:35            846 libtestlib.a   --库文件testlib编译成功
-a---           2021/9/19    22:29             81 testlib.c   
-a---           2021/9/19    22:35            700 testlib.o   

PS D:\linkcTest>
```
使用v编译linkcTest.v
```shell
PS D:\linkcTest> v -cc gcc -showcc .\linkcTest.v  --使用-showcc参数查看v使用的编译命令
> C compiler cmd: gcc "@C:\Users\xxxxx\AppData\Local\Temp\v\linkcTest.14127686245975582114.tmp.c.rsp"
> C compiler response file "C:\Users\xxxxx\AppData\Local\Temp\v\linkcTest.14127686245975582114.tmp.c.rsp":
  -std=c99 -D_DEFAULT_SOURCE -o "D:\\linkcTest\\linkcTest.exe"
  -L "D:\\linkcTest\\myClib"     --#flag定义
  -I "D:\\linkcTest\\myCheader"  --#flag定义
  "C:\\Users\\xxxxx\\AppData\\Local\\Temp\\v\\linkcTest.14127686245975582114.tmp.c" -municode -ldbghelp
  -ltestlib                      --#flag定义
PS D:\linkcTest> .\linkcTest.exe
Hello World!
0                                --TestFunc调用成功
```


### pkgconfig

pkg-config是C广泛使用的编译依赖配置工具，V也实现了V版本的pkgconfig，使用方法基本跟C版本保持了兼容。

#### 命令行

V版本的pkgconfig的源代码位于V源代码中的：vlib/v/pkgconfig

- 编译pkgconfig

切换到vlib/v/pkgconfig目录中，然后执行：

```shell
v ./bin/pkgconfig.v
```

- 使用pkgconfig命令行

  基本跟C版本的使用兼容，具体使用参考：

```shell
./bin/pkgconfig --help
```

一般来说pkgconfig命令行比较少直接使用，一般都是通过在V源代码中加载pc配置文件。

#### 加载pc配置文件

V版本的pkgconfig配置文件跟C版本一致，配置文件的扩展名也是.pc。

直接在V源代码中加载.pc配置文件，就可以更方便地实现跟C代码库实现集成，能够正确编译。

一般来说，在加载之前先使用$pkgconfig('xxx')编译时函数来检查xxx.pc配置文件是否存在，如果存在，就可以使用#pkgconfig标记来加载.pc配置文件，

配置文件的搜索路径跟C版本一样，默认会搜索/usr/lib/pkg-config目录，若找不到，则会去PKG_CONFIG_PATH环境变量指定的路径下查找。

```v
$if $pkgconfig('mysqlclient') {	//编译时判断mysqlclient.pc配置文件是否存在
	#pkgconfig mysqlclient //如果存在，则加载mysqlclient.pc配置文件
} $else $if $pkgconfig('mariadb') { 
	#pkgconfig mariadb
}
```

在实际的例子中，V编译器的可选垃圾收集器使用了C版本的bdw-gc，在vlib/builtin/builtin_d_gcboehm.c.v源代码中有这么一段代码：

```v
	$if macos { 	//在mac系统中
		#pkgconfig bdw-gc		//加载bdw-gc.pc配置文件到C源代码中
	} $else $if openbsd || freebsd {
		#flag -I/usr/local/include
		#flag -L/usr/local/lib
	}
```

#### 解析pc配置文件内容

除了可以在源代码中直接加载pc配置文件到生成的C源代码中，还可以使用v.pkgconig模块来解析pc配置文件中的内容：

Vlib/v/pkgconfig/test_sample目录中有许多pc配置的示例，通过以下代码解析出配置文件对应字段的内容，效果跟解析v.mod类似。

```v
import os
import v.pkgconfig

const vexe = os.getenv('VEXE')

const vroot = os.dir(vexe)

const samples_dir = os.join_path(vroot, 'vlib', 'v', 'pkgconfig', 'test_samples')

fn main() {
	pc_files := os.walk_ext(samples_dir, '.pc')
	assert pc_files.len > 0
	for pc in pc_files {
		pcname := os.file_name(pc).replace('.pc', '')
		x := pkgconfig.load(pcname, use_default_paths: false, path: samples_dir) or {
			if pcname == 'dep-resolution-fail' {
				continue
			}
			println('>>> err: $err')
			assert false
			return
		}
		assert x.name != ''
		assert x.modname != ''
		assert x.version != ''
		if pcname == 'gmodule-2.0' {
			assert x.name == 'GModule'
			assert x.modname == 'gmodule-2.0'
			assert x.url == ''
			assert x.version == '2.64.3'
			assert x.description == 'Dynamic module loader for GLib'
			assert x.libs == ['-Wl,--export-dynamic', '-L/usr/lib/x86_64-linux-gnu', '-lgmodule-2.0',
				'-pthread', '-lglib-2.0', '-lpcre']
			assert x.libs_private == ['-ldl', '-pthread']
			assert x.cflags == ['-I/usr/include', '-pthread', '-I/usr/include/glib-2.0',
				'-I/usr/lib/x86_64-linux-gnu/glib-2.0/include',
			]
			assert x.vars == {
				'prefix':            '/usr'
				'libdir':            '/usr/lib/x86_64-linux-gnu'
				'includedir':        '/usr/include'
				'gmodule_supported': 'true'
			}
			assert x.requires == ['gmodule-no-export-2.0', 'glib-2.0']
			assert x.requires_private == []
			assert x.conflicts == []
		}
		if x.name == 'expat' {
			assert x.url == 'http://www.libexpat.org'
		}
		if x.name == 'GLib' {
			assert x.modname == 'glib-2.0'
			assert x.libs == ['-L/usr/lib/x86_64-linux-gnu', '-lglib-2.0', '-lpcre']
			assert x.libs_private == ['-pthread']
			assert x.cflags == ['-I/usr/include/glib-2.0',
				'-I/usr/lib/x86_64-linux-gnu/glib-2.0/include', '-I/usr/include']
			assert x.vars == {
				'prefix':          '/usr'
				'libdir':          '/usr/lib/x86_64-linux-gnu'
				'includedir':      '/usr/include'
				'bindir':          '/usr/bin'
				'glib_genmarshal': '/usr/bin/glib-genmarshal'
				'gobject_query':   '/usr/bin/gobject-query'
				'glib_mkenums':    '/usr/bin/glib-mkenums'
			}
			assert x.requires_private == ['libpcre']
			assert x.version == '2.64.3'
			assert x.conflicts == []
		}
	}
}
```
