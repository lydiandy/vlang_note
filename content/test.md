## 代码测试

### 编写测试文件

模块目录中：

测试文件:以 xxx_test.v结尾。

测试函数:以test_xxx()开头。

### assert 断言

assert后面的表达式结果不为true，即为测试不通过。

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

### 执行测试

#### 执行单个测试文件：

```shell
v test xxx_test.v
```

#### 执行模块测试文件：

```shell
v test xxx(模块名/目录名)
```

会逐个执行模块，包括子模块中的所有测试文件，所有以test_开头的测试函数。

执行测试时，可增加-stats选项，显示更为详细的测试结果。

```shell
v -stats test xxx.v
```

#### 忽略testdata目录

如果希望有一些测试文件要忽略执行，可以创建名为testdata的目录，测试框架会忽略这个目录。

实际开发场景中，这个目录还是挺实用的，可以把临时不用的测试文件挪到该目录中。
