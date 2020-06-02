## 错误处理

### 错误定义

函数定义时,返回类型前面加?,表示这个函数可能返回一个错误

在函数代码中根据逻辑丢出错误:

return error('error message') 或 return none 表示抛出错误

### 错误处理

函数调用时,使用or代码块来处理错误,默认会传递err参数给or代码块,包含错误信息,

如果return none,那么参数err的值为空

or代码块必须以:return/panic/exit/continue/break结尾

```c
//函数定义
fn my_fn(i int) ?int {
	if i==0 {
		return error('Not ok!') //抛出错误,err的值为Not ok!
	}
	if i==1 {
	    return none //抛出错误,但是没有错误信息,err的值为空字符串
	}
	return i //正常返回
}

fn main() {
    //函数调用
    //触发错误,执行or代码块,程序中断,报错:V panic: Not ok!
	v1:=my_fn(0) or {
	    println('from 0')
	    println(err)
	    panic(err) //默认会传递err参数给or代码块,包含错误信息
	}
	println(v1)

    //触发错误,执行or代码块,因为是return none,所以err为空,
	v2 := my_fn(1) or {
	    println('from 1')
	    if err=='' {
	        println('err is empty')
	    }
		return
	}
	println(v2)

    //未触发错误,不执行or代码块,返回函数的返回值 
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
	ok      bool
	is_none bool
	error   string
	ecode   int
	data    [400]byte
}

pub fn (o Option) str() string {
   if o.ok && !o.is_none {
	  return 'Option{ data: ' + o.data[0..32].hex() + ' }'
   }
   if o.is_none {
	  return 'Option{ none }'
   }
   return 'Option{ error: "${o.error}" }'
}

// `fn foo() ?Foo { return foo }` => `fn foo() ?Foo { return opt_ok(foo); }`
fn opt_ok(data voidptr, size int) Option {
	if size >= 400 {
		panic('option size too big: $size (max is 400), this is a temporary limit')
	}
	res := Option{
		ok: true
	}
	C.memcpy(res.data, data, size)
	return res
}

// used internally when returning `none`
fn opt_none() Option {
	return Option{
		ok: false
		is_none: true
	}
}

pub fn error(s string) Option {
	return Option{
		ok: false
		is_none: false
		error: s
	}
}

pub fn error_with_code(s string, code int) Option {
	return Option{
		ok: false
		is_none: false
		error: s
		ecode: code
	}
}
```

若函数无返回值,仍需抛出错误,要使用?

```c
module main

fn main() {
	exec('') or {
		panic('error is :$err')
	}
}

fn exec(stmt string) ? { //无返回值,也可抛出错误
	if stmt == '' {
		return error('stmt is null')
	}
	println(stmt)
}
```

返回错误码

```c
module main

fn main() {
	exec('') or {
    //约定的变量名err和errcode
		panic('error text is :$err;error code is $errcode') 
	}
}

fn exec(stmt string) ? {
	if stmt == '' {
		return error_with_code('stmt is null', 123) //需要带错误码
	}
	println(stmt)
}

```



### 向上抛转错误

```c
resp := http.get(url)? //在调用函数后加上?,表示如果函数执行出现错误,当前调用层级不处理,直接向上抛转错误
println(resp.body)
```

http.get函数中,定义的返回值是:?Response

当用上面的方式调用get函数时,如果触发了错误,错误会被向上抛转给调用get函数的上级函数,

上级函数的返回类型必须也有错误处理,如果上级函数是main主函数,那么就会以panic的方式处理错误

类似以下的代码:

```c
resp := http.get(url) or {
	panic(err)
}
println(resp.body)
```







