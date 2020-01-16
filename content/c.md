## 调用C代码

V的代码库很多都直接调用C标准库函数来实现，对C标准库的依赖还是很重的

由于V代码编译后生成的是C代码,然后再调用C编译器编译成可执行文件

这样的机制决定了V语言可以很方便地调用C世界的各种代码库,是个很大的一个优势,毕竟C的代码库经过多年的积累,很丰富

在V代码里调用C代码也非常简单,在V标准库的代码里随处可见:

1.在V代码中使用C的宏,比如#include宏,编译时这些宏会被原封不动地搬到生成的C代码中,效果跟模块导入一样

2.然后用V的函数或结构体语法,定义要使用的C函数声明或结构体声明,函数名或结构体名一定要以C.开头

3.就可以在后续的V代码中使用声明好的C函数或结构体,调用的时候,名称前可以使用C.开头,也可以不使用,不过还是建议统一以C.开头,代码更清晰,哪些是调用C函数或结构体

其实V编译的时候,会统一把函数和结构体名称前面的C.统一去掉,这样在C代码里面就可以正常调用了

```c
module main

fn C.getpid() int

fn main(){
	pid:=C.getpid()
	pid2:=getpid() //C.开头的去掉也可以
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



其实,直接在V代码中使用C函数或者结构体也是可以的,不过,由于命名方式不一致的原因,习惯上也可以对C函数或结构体,进行一层简单封装,名字可以重新改为V风格,或者更为简短的名字

以下代码是GUI代码库中引用了glfm后,进行的简单封装:

vlib/glfm/glfm.v部分代码:

```c
#flag darwin -L/opt/local/lib

#flag darwin -lglfw
#flag freebsd -I/usr/local/include
#flag freebsd -Wl,-L/usr/local/lib,-lglfw
#flag linux -lglfw
#flag windows -lgdi32 -lshell32 -lglfw3
#include <GLFW/glfw3.h>

//可以直接使用
pub fn init_glfw() {
	C.glfwInit()
	C.glfwWindowHint(C.GLFW_CONTEXT_VERSION_MAJOR, 3)
	C.glfwWindowHint(C.GLFW_CONTEXT_VERSION_MINOR, 3)
	C.glfwWindowHint(C.GLFW_OPENGL_FORWARD_COMPAT, C.GL_TRUE)
	C.glfwWindowHint(C.GLFW_OPENGL_PROFILE, C.GLFW_OPENGL_CORE_PROFILE)
}
//也可以进行简单的封装
pub fn terminate() {
	C.glfwTerminate()
}
pub fn window_hint(key, val int) {
	C.glfwWindowHint(key, val)
}
```

