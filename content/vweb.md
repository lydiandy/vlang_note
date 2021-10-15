## vweb框架

一般来说，语言的标准库不会内置web框架。

V不太一样，vweb是内置的web框架,甚至在编译器内部都有语法上的优化。

go的web框架实在是太多,选择起来眼花缭乱的，V的作者不希望V的生态圈有那么多的框架，难以选择，所以官方内置了一个，让大家一起开发这个。

V直接内置web框架，感觉不应该内置，一方面无谓增加编译器的复杂度。

另一方面毕竟众口难调，不同场景，不同人喜欢的web框架风格不一样。

感觉是走了两个极端，所以一直没有什么想法看这个内置的。

V的并发功能基本开发完成，目前vweb也是采用并发来处理请求，跟go一样。

### vweb

```v
import vweb

fn main() {
	vweb.run(&App{}, 8080)
}

struct App {
	vweb.Context
}

fn (mut app App) hello() vweb.Result {
	return app.text('Hello')
}

['/foo']
fn (mut app App) world() vweb.Result {
	return app.text('World')
}

[post]
fn (mut app App) say() vweb.Result {
	return app.text('World')
}

['/hello/:user']
fn (mut app App) hello_user(user string) vweb.Result {
	return app.text('Hello $user')
}

```

