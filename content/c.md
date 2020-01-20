## 调用C代码

### 优势

V的代码库很多都直接调用C标准库函数来实现，对C标准库的依赖还是很重的

由于V代码编译后生成的是C代码,然后再调用C编译器编译成可执行文件

这样的机制决定了V语言可以很方便地调用C世界的各种代码库,

这对于V语言来说,是个很大的一个优势,毕竟C代码库经过多年的积累,很丰富

### 如何调用

在V代码里调用C代码也非常简单,在V标准库里随处可见:

1.先定义#flag,#flag的定义要放在C语言宏之前

2在V代码中使用C语言宏,比如#include宏或#define宏,编译时这些宏会被原封不动地搬到生成的C代码中

3.然后用V的函数或结构体语法,定义要使用的C函数或C结构体声明,函数名或结构体名一定要以C.开头

4.就可以在后续的V代码中使用C函数或结构体,调用的时候,名称前可以使用C.作为前缀,也可以不使用,不过还是建议统一使用C.前缀,代码更清晰,哪些是调用C函数或结构体

其实V编译的时候,会统一把函数和结构体名称前面的C.统一去掉,这样在C代码里面就可以正常调用了



**关于#flag标记:**

这个#flag标记跟v编译器的-cflags选项的用法是一样的,用于传递额外的flag参数给C编译器

关于C编译器的flag参数可以参考:

https://colobu.com/2018/08/28/15-Most-Frequently-Used-GCC-Compiler-Command-Line-Options/

1.要在使用C宏之前先定义#flag

2. -l	表示在库文件的搜索路径列表中添加指定的路径

​	  -I	表示在头文件的搜索路径中添加指定的路径

​	  -D	表示设置编译时变量

3.还可以在#flag后增加平台标识,针对不同的平台配置不同的flag,目前支持的平台有:linux,darwin,windows

下面是实际的例子,提供参考:

```c
//#flag文档里面的:
#flag linux -lsdl2
#flag linux -Ivig
#flag linux -DCIMGUI_DEFINE_ENUMS_AND_STRUCTS=1
#flag linux -DIMGUI_DISABLE_OBSOLETE_FUNCTIONS=1
#flag linux -DIMGUI_IMPL_API=
```

```c
//mysql包里面的:
#flag -lmysqlclient //-l开头,表示在库文件的搜索路径中添加mysqlclient
#flag linux -I /usr/include/mysql //-I开头,在头文件的搜索路径中添加指定的路径, 针对linux平台,这样才可以搜索到下面的mysql.h头文件
#include <mysql.h>

//sokol包里面的:
#flag -I @VROOT/thirdparty/sokol 
#flag -I @VROOT/thirdparty/sokol/util

#flag darwin -fobjc-arc  //针对mac平台,提供额外的flag参数:-fobjc-arc
#flag linux -lX11 -lGL	//针对linux平台,提供额外的flag参数

#flag windows -lgdi32 //针对window平台,提供额外的flag参数:-l开头,表示链接gdi32

// OPENGL
#flag -DSOKOL_GLCORE33 //提供额外的flag参数:-DSOKOL_GLCORE33
#flag darwin -framework OpenGL -framework Cocoa -framework QuartzCore
```



```c
module main
    
#include <mysql.h>
    
fn C.getpid() int //定义要使用的C函数声明,名字在C函数名字前统一加上C.前缀
struct C.MYSQL 	  //C结构体声明
    
fn main(){
	pid:=C.getpid()
	pid2:=getpid() //C.前缀去掉也可以
	println(pid)
	println(pid2)
}
```

myslq库中的参考代码:

```c
//宏
#flag -lmysqlclient
#flag linux -I/usr/include/mysql
#include <mysql.h>
//声明要使用的C结构体
struct C.MYSQL
struct C.MYSQL_RES
//声明要使用的C函数
fn C.mysql_init(mysql &C.MYSQL) &C.MYSQL
fn C.mysql_real_connect(mysql &C.MYSQL, host byteptr, user byteptr, passwd byteptr, db byteptr, port u32, unix_socket byteptr, clientflag u64) &C.MYSQL
fn C.mysql_query(mysql &C.MYSQL, q byteptr) int
fn C.mysql_select_db(mysql &C.MYSQL, db byteptr) int
fn C.mysql_error(mysql &C.MYSQL) byteptr
fn C.mysql_errno(mysql &C.MYSQL) int
fn C.mysql_num_fields(res &C.MYSQL_RES) int
fn C.mysql_store_result(mysql &C.MYSQL) &C.MYSQL_RES
fn C.mysql_fetch_row(res &C.MYSQL_RES) &byteptr
fn C.mysql_free_result(res &C.MYSQL_RES)
fn C.mysql_real_escape_string_quote(mysql &C.MYSQL, to byteptr, from byteptr, len u64, quote byte) u64
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

### 简单封装

其实,直接在V代码中使用C函数或者结构体也是可以的,不过,由于命名方式不一致的原因,习惯上也可以对C函数或结构体,进行一层简单封装,名字可以重新改为V风格,或者更为简短的名字

一般来说,对一个C代码库中的简单封装会涉及到:结构体,函数,枚举这3大类

如果是小的C代码库,可以直接把这3类的简单封装都放在一个V源文件中

如果C代码库规模大一些,也可以这3类,各自单独一个V源文件,归属于同一个V模块

以下代码是GUI代码库中引用了sokol C代码库后,进行的简单封装:

vlib/sokol/sokol.v部分代码:

```c
//只要在同模块中的任何一个V源文件中引入,该模块的其他源文件就可以直接使用C代码库内容
#define SOKOL_IMPL
#define SOKOL_NO_ENTRY
#include "sokol_app.h"

#define SOKOL_IMPL
#define SOKOL_NO_DEPRECATED
#include "sokol_gfx.h"

//可以直接使用,函数以C.作为前缀
pub fn init_sokol() {
	C.sapp_isvalid()
	C.sapp_width()
}
//也可以进行简单的封装
module sapp

[inline] //可以给函数添加inline标注,变为内联函数
pub fn isvalid() bool {
	return C.sapp_isvalid()
}

[inline]
pub fn width() int {
	return C.sapp_width()
}

```

### 函数的[inline]标注

对C函数进行简单的封装时,可以给函数添加inline标注,编译生成C代码时,这个函数就会变成C语言里的static inline函数

内联函数有些类似于宏,内联函数的代码会被直接嵌入在它被调用的地方，调用几次就嵌入几次，没有使用call指令。这样省去了函数调用时的一些额外开销,不过调用次数多的话，会使可执行文件变大，这样会降低整个程序的运行速度

像上面那个简单的封装,函数只有1行代码,嵌入到被调用的地方也还是1还代码,既能省去函数调用时的额外开销,提升性能,又不会使可执行文件变大

同时也可以统一和简化C函数的命名,变为V风格的简短命名,一举多得

```c
[inline]
pub fn width() int {
	return C.sapp_width()
}
```

### 结构体简单封装

定义C结构体等价的V版本结构体,V版本结构体名称以C.做前缀

这样就可以直接使用V版本的结构体来创建变量

**用V版本的结构体创建变量,编译生成的C代码中,就是用C版本结构体创建了变量**

C代码库中的结构体:

```c
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

定义等价的V版本结构体:字段数量一样,字段名称一样,字段类型一样

如果字段类型是枚举的,也可以再定义等价的V版本枚举

```c
pub struct C.sapp_event {
pub:
    frame_count u64
    @type EventType //这个type有点特殊,因为是V的关键字,所以用@开头就可以
    key_code KeyCode //枚举类型
    char_code u32
    key_repeat bool
    modifiers u32
    mouse_button MouseButton //枚举类型,下面有MouseButton的V版本定义
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

### 枚举简单封装

其实就是定义一个跟C版本枚举一样的枚举,枚举名和枚举值一样

```c
pub enum MouseButton {
    invalid = -1
    left = 0
    right = 1
    middle = 2
}
```

