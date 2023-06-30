## 错误处理

### 内置函数和结构体

V语言内置了以下内置错误函数和结构体，用来进行错误处理：

```v
//内置错误接口
pub interface IError {
	msg() string //返回错误消息
	code() int //返回错误码
}
//内置的消息错误类型
pub struct Error {}

pub fn (err Error) msg() string {
	return ''
}

pub fn (err Error) code() int {
	return 0
}
//内置函数,创建一个错误，错误的类型为内置的MessageError
pub fn error(msg string) IError //抛出带消息的错误
pub fn error_with_code(msg string,code int) IError //带错误消息和错误码
```

### 错误定义

定义函数时：

返回类型前加感叹号!表示：函数可能返回正常的类型或返回错误，即 T  |  IError，

返回类型前加问号?表示：函数可能返回正常的类型或返回空值，即 T  |  none，

其实对应的类型是内置的Option类型

然后在函数代码中根据逻辑，返回空值或错误：

```v
return none //返回空值

return error('error message')  //表示抛出错误

return error_with_code('error message',1)  //表示抛出带错误消息和错误码的错误
```

示例代码：

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

### 错误处理

函数调用时，使用or代码块来处理错误，默认会传递err参数给or代码块，包含错误信息。

err.msg()返回错误信息，err.code()返回错误码。

or代码块必须以：return/panic/exit/continue/break结尾。

```v
//函数定义
fn my_fn(i int) !int {
	if i == 0 {
		return error('Not ok!') //抛出错误,err的值为Not ok!
	}
	return i //正常返回
}

fn my_fn2(i int) !(int, int) { //多返回值时,!放在括号前面
	return 1, 1
}

fn main() {
	//函数调用
	//未触发错误,不执行or代码块,返回函数的返回值
	v1 := my_fn(1) or {
		println('from 2')
		return
	}
	println(v1)
	//触发错误,执行or代码块,程序中断,报错:V panic: Not ok!
	v2 := my_fn(0) or {
		println('from 0')
		panic(err.msg()) //默认会传递err参数给or代码块,包含错误信息
	}
	println(v2)
}

```

也可以通过match语句来匹配不同的错误，进行错误处理：

```v
module main

import semver

fn main() {
	semver.from('asd') or { check_error(err) }
	semver.from('') or { check_error(err) }
}

fn check_error(err IError) {
	match err { //匹配不同的错误类型,进行错误处理
		semver.InvalidVersionFormatError {
			println('wrong format')
		}
		semver.EmptyInputError {
			println('empty input')
		}
		else {
			println('unknown error')
		}
	}
}
```

if guard守护条件处理错误：

```v
module main

//带空值的函数
fn my_fn(i int) ?int {
	if i == 0 {
		return none //返回空值
	}
	return i //正常返回
}

// 多返回值的函数
fn create() ?(int, string, bool) {
	return 5, 'aa', true
}

fn main() {
	// if guard expr
	if c := my_fn(2) { // if guard守护条件,调用函数时,正常返回,执行if分支
		println('$c') // 输出2
	} else {
		println('$err')
	}
	if c := my_fn(0) { // if guard守护条件,调用函数时,返回none,执行else分支
		println('$c') 
	} else {
		println('$err') // 输出none
	}

	if r1, r2, r3 := create() { // 多返回值的if guard守护条件
		println(r1)
		println(r2)
		println(r3)
	} else {
		println('$err')
	}

	// if守护条件,其实等价于
	cc := my_fn(0) or {
		println('$err')
		11
	}
	println(cc)

	// if guard的变量是可变的情况
	if mut c := my_fn(3) {
		c = 2
		println(c)
	} else {
		println('$err')
	}
}
```

for循环结合or代码块使用：

```v
module main

import os

fn main() {
	// for循环结合or代码块,更简洁一些
	// for line in os.read_lines(@FILE) or { panic('文件不存在') } {
	// 报错
	for line in os.read_lines('不存在的文件') or { panic('文件不存在') } {
		println(line)
	}
}
```

若函数无返回值，仍需抛出空值或错误，要使用?或！。

```v
module main

fn main() {
	exec('') or { panic('error is :$err') }
}

fn exec(stmt string) ! { //无返回值,也可抛出错误
	if stmt == '' {
		return error('stmt is null')
	}
	println(stmt)
}
```

返回错误码：

```v
module main

fn main() {
	exec('') or {
		//约定的变量名err
		panic('error text is :$err.msg();error code is $err.code()')
	}
}

fn exec(stmt string) ! {
	if stmt == '' {
		return error_with_code('stmt is null', 123) //需要带错误码
	}
	println(stmt)
}
```

### 向上抛转错误

调用函数时，可以在调用时不处理，而是向调用链的上级函数抛转错误，如果向上抛转到main主函数还没有处理错误，那么就会以panic的方式报错。

向上抛转时，上级函数的返回值必须也是对应的错误或空值，否则编译时会报错。

```v
//函数定义
fn my_fn(i int) !int {
	if i == 0 {
		return error('Not ok') //抛出错误,err的值为Not ok
	}
	return i //正常返回
}

fn my_fn2(i int) ?int {
	if i == 0 {
		return none //返回空值
	}
	return i //正常返回
}

fn my_fn3() ?int {
	v := my_fn2(0)? //本级不处理空值,向上抛转,函数返回的错误类型必须也是?,而不能是!
	return v
}

fn main() {
	v1 := my_fn(1)! //本级不处理,向上抛转错误
	println(v1)

	v2 := my_fn2(0) or { 0 } //本级处理空值
	println(v2)

	v3 := my_fn3() or { -1 } //下级函数向上抛转,未处理的空值,必须在main中处理,否则报错
	println(v3)
}
```

### 自定义错误类型

```v
module main

struct MyError { //组合内置的Error类型,来自定义错误类型
	Error
}

pub fn (error MyError) msg() string { //定义错误消息
	return 'my error message'
}

struct MyError2 {} //通过实现IError接口,来自定义错误类型

pub fn (error MyError2) msg() string {
	return 'my error2 message'
}

pub fn (error MyError2) code() int {
	return 11
}

pub fn my_fn(name string) !string {
	if name == '' {
		return MyError{} //抛出自定义错误
	} else {
		return name
	}
}

pub fn my_fn2(name string) !string {
	if name == '' {
		return MyError2{}
	} else {
		return name
	}
}

fn main() {
	name := ''
	// my_fn(name) or {
	// 	println(err.code())
	// 	panic(err)
	// }
	my_fn2(name) or {
		println(err.code())
		panic(err)
	}
}
```
