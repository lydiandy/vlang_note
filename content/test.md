## 代码测试

**编写测试文件**

模块目录中:

代码测试文件:以 xxx_test.v结尾

测试文件中的测试函数,以test_xxx开头

**assert 断言**

如果assert后面的表达式不为true，就是测试不通过

vlib/builtin/string_test.v

```
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

**执行测试:**

单个测试文件执行：

v run xxx_test.v

执行模块的所有测试文件：

v test xxx

会逐个执行模块中的所有测试文件,所有以test_开头的测试函数

