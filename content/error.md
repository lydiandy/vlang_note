## 错误处理

### 错误定义

函数定义时,返回类型前面加?,表示这个函数可能返回一个错误

在函数代码中根据逻辑丢出错误:return error('error message') 或者return none

### 错误处理

函数调用时,使用or代码块来处理错误,默认会传递err参数给or代码块,包含错误信息,

如果return none,那么参数err的值为空

or代码块必须以:return/panic/exit/continue/break结尾

```c
fn my_fn(i int) ?int {
	if i==0 {
		return error('Not ok!')
	}
	if i==1 {
	    return none
	}
	return i
}

fn main() {
    //触发错误,程序中断,报错:V panic: Not ok!
	v1:=my_fn(0) or {
	    println('from 0')
	    println(err)
	    panic(err) //默认会传递err参数给or代码块,包含错误信息
	}
	println(v1)

    //触发错误,因为是return none,所以err为空,
	v2 := my_fn(1) or {
	    println('from 1')
	    if err=='' {
	        println('err is empty')
	    }
		return
	}
	println(v2)

    //未触发错误
	v3:=my_fn(2) or {
	    println('from 2')
	    return
	}
	println(v3)

}
```

error函数是内置函数,定义在:vlib/builtin/option.v

```c
module builtin

struct Option {
	data     [255]byte
	error    string
	ok       bool
	is_none  bool
}

// `fn foo() ?Foo { return foo }` => `fn foo() ?Foo { return opt_ok(foo); }`
fn opt_ok(data voidptr, size int) Option {
	if size >= 255 {
		panic('option size too big: $size (max is 255), this is a temporary limit')
	}
	res := Option {
		ok: true
	}
	C.memcpy(res.data, data, size)
	return res
}

fn opt_none() Option {
	return Option{ is_none: true }
}

pub fn error(s string) Option {
	return Option {
		error: s
	}
}
```



### 向上抛转错误

以下错误处理代码:

```c
resp := http.get(url)? //在调用函数后加上?,可以向上抛转错误
println(resp.body)
```

http.get函数中,定义的返回值是:?Response

当用上面的方式调用get函数时,如果触发了错误,错误会被向上抛转给调用get函数的上级函数,

上级函数的返回类型必须也有错误处理,如果上级函数是main主函数,那么就会以panic的方式处理错误

类似以下的代码:

```go
resp := http.get(url) or {
	panic(err)
}
println(resp.body)
```







