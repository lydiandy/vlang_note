## 代码测试

### 编写测试文件

模块目录中：

测试文件：以xxx_test.v结尾。

测试函数：以test_xxx()开头。

### assert 断言语句

如果assert语句后面的表达式结果为true，表示测试通过，如果为false，表示测试不通过。

示例：vlib/builtin/string_test.v

```v
fn test_add() {
	mut a := 'a'
	a += 'b'
	assert a==('ab')
	a = 'a'
	for i := 1; i < 1000; i++ {
		a += 'b'
	}
	assert a.len == 1000
	assert a.ends_with('bbbbb')
	a += '123'
	assert a.ends_with('3')
}

fn test_ends_with() {
	a := 'browser.v'
	assert a.ends_with('.v')
}

fn test_between() {
	 s := 'hello [man] how you doing'
	assert s.find_between('[', ']') == 'man'
}
```

assert语句还支持附带错误消息：当表达式结果为false时，可以输出一个错误消息，方便测试定位问题。

错误消息必须是字符串类型，或者返回字符串的函数调用。

```v
fn test_abc() {
	i := 123
	assert 4 == 2 * 2
	assert 2 == 6 / 3, 'math works' 
	assert 10 == i - 120, 'i: $i' //附带字符串类型的错误消息，当表达式为false时输出
}
```

执行错误时输出：

```shell
> assert 10 == i - 120, 'i: $i'
    Left value: 10
    Right value: 3
    Message: i: 123
```

### 断言继续

如果测试函数加了assert_continues注解，当断言不为真时，仍然继续往下执行，只是把所有断言不通过时的表达式左值和右值打印出来。如果没有这个注解，碰到第一个不通过的条件，测试就会停止，只打印出第一个不通过时的表达式。

```v
[assert_continues]
fn abc(ii int) {
	assert ii == 2
}

fn test_abc() {
	for i in 0 .. 4 {
		abc(i)
	}
}
```

输出所有断言不通过的表达式值：

```shell
main_test.v:3: ✗ fn abc
   > assert ii == 2
     Left value: 0
    Right value: 2

main_test.v:3: ✗ fn abc
   > assert ii == 2
     Left value: 1
    Right value: 2

main_test.v:3: ✗ fn abc
   > assert ii == 2
     Left value: 3
    Right value: 2
```

### 执行测试

执行单个测试文件：

```shell
v test xxx_test.v
```

执行模块测试文件：

```shell
v test xxx(模块名/目录名)
```

会逐个执行模块，包括子模块中的所有测试文件，所有以test_开头的测试函数。

执行测试时，可增加-stats选项，显示更为详细的测试结果。

```shell
v -stats test xxx.v
```

忽略testdata目录

如果希望有一些测试文件要忽略执行，可以创建名为testdata的目录，测试框架会忽略这个目录。

实际开发场景中，这个目录还是挺实用的，可以把临时不用的测试文件挪到该目录中。

### 特定操作系统测试

如果希望有些测试函数只在特定操作系统下执行，可以使用if注解来实现。

执行测试的时候，只有满足操作系统条件的测试函数才执行。

```v
[if macos]  //只有macos系统中才执行测试
pub fn test_in_macos() {
	println("test in macos")
	assert true
}

[if !macos] //只有非macos系统才执行测试
pub fn test_not_in_macos() {
	$if !macos {
		println("test not in macos")
	}
	$if macos {
		println("test in macos")
	}
	println("test only not in macos")
	assert false
}

[if linux] 
pub fn test_only_in_linux() {
	println("test only in linux")
	assert false
}

[if linux || windows] //在linux或windows系统中才执行测试
pub fn test_in_linux_or_macos() {
	println("test in linux or macos")
	assert false
}
```

