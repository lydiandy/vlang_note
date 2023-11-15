## 注解(attribute)

V语言可以对模块，枚举，结构体，结构体字段，函数/方法，变量增加注解。

注解的格式统一是：@[注解名] 或 [注解名:注解扩展内容]。

### 编译器内置注解

#### 模块注解

- @[deprecated]

  参考：[模块章节](module.md)

- @[deprecated_after]

  参考：[模块章节](module.md)

#### 枚举注解

- @[flag]注解

  参考：[枚举章节](enum.md)

#### 结构体注解

结构体注解统一放在结构体定义前。

- @[typedef]

  参考：[结构体章节](struct.md)
  
- @[heap]

  参考：[结构体章节](struct.md)
  
- @[noinit]

  参考：[结构体章节](struct.md)

- @[params]

  参考：[结构体章节](struct.md)

- @[table]

  参考：[结构体章节](struct.md)
  
- @[packed]
  
  参考：[结构体章节](struct.md)
  
- @[autostr: allowrecurse]

  参考：[结构体章节](struct.md)



#### 结构体字段注解

结构体字段注解统一放在字段定义后。

- @[required]

  参考：[结构体章节-字段注解](struct.md)
  
- @[deprecated]

  参考：[结构体章节-字段注解](struct.md)
  
- @[json]

  参考：[json章节](json.md)
  
- @[toml]
  
  
  参考：[toml章节](std_toml.md)
  
- @[skip]

  参考：[json章节](json.md)
  
- @[row]

  参考：[json章节](json.md)

#### 函数注解

函数注解统一放在函数定义之前。

- @[deprecated]

  参考：[函数章节](fn.md)
  
- @[inline]

  参考：[函数章节](fn.md)

- @[live]

  参考：[函数章节](fn.md)

- @[unsafe]

  参考：[unsafe章节](unsafe.md)

- @[trusted]

  参考：[unsafe章节](unsafe.md)
  
- @[export]
  
  
  参考：[函数章节](fn.md)
  
- @[weak]

  参考：[函数章节](fn.md)

- @[if xxx平台]

  参考：[函数章节](fn.md)

- @[if debug]

  这个注解表示该函数只有在编译时加上了-d 调试模式的时候，才会被编译生成，并且可以调用。加上@[if debug]注解后函数不可以有返回值。

  ```v
  @[if debug]
  fn foo() {
  }
  ```

- @[windows_stdcall]

  这个注解只能用于Win32 API，如果需要传递回调函数的时候使用。
  
  ```v
  @[windows_stdcall]
  fn C.DefWindowProc(hwnd int, msg int, lparam int, wparam int)
  ```

- @[console]

  这个注解只能用在main函数前，导入了图形库模块(比如gg,ui)后，命令行窗口就不再显示了，查看不到命令行输出,加上这个注解，命令行窗口就会出现。

  ```v
  @[console]
  fn main() {}
  ```

- @[noinline]

  

- @[direct_array_access]

  加了这个注解的函数，在生成C代码时会直接使用C语言中的数组操作，省略边界检查，这样在遍历数组元素时速度会提高不少，代价是不会进行边界检查，函数是不安全的，边界检查由用户代码自己判断。

  ```v
  module main
  
  fn main() {
  	direct_modification1()
  	direct_modification2()
  }
  
  fn direct_modification1() { // 不带direct_array_access
  	mut foo := [2, 0, 5]
  	foo[1] = 3
  	foo[0] *= 7
  	foo[1]--
  	foo[2] -= 2
  }
  
  @[direct_array_access] 
  fn direct_modification2() { // 带direct_array_access
  	mut foo := [2, 0, 5]
  	foo[1] = 3
  	foo[0] *= 7
  	foo[1]--
  	foo[2] -= 2
  }
  ```

  生成的C代码：

  ```c
  VV_LOCAL_SYMBOL void main__main(void) {
  	main__direct_modification1();
  	main__direct_modification2();
  }
  
  VV_LOCAL_SYMBOL void main__direct_modification1(void) {
  	Array_int foo = new_array_from_c_array(3, 3, sizeof(int), _MOV((int[3]){2, 0, 5}));
  	array_set(&foo, 1, &(int[]) { 3 });
  	(*(int*)array_get(foo, 0)) *= 7;
  	(*(int*)/*ee elem_typ */array_get(foo, 1))--;
  	(*(int*)array_get(foo, 2)) -= 2;
  }
  
  // Attr: [direct_array_access]
  VV_LOCAL_SYMBOL void main__direct_modification2(void) {
  	Array_int foo = new_array_from_c_array(3, 3, sizeof(int), _MOV((int[3]){2, 0, 5}));
  	((int*)foo.data)[1] = 3;
  	((int*)foo.data)[0] *= 7;
  	((int*)foo.data)[1]--;
  	((int*)foo.data)[2] -= 2;
  }
  ```

- @[manualfree]

  参考[内存管理章节-手动内存管理](memory.md)。


- @[noreturn]

  表示这个函数不会有返回值给函数的调用方，一般来说是函数内有存在exit，panic，无限循环for {}，或调用了另一个没有返回值的函数。标记了noreturn的函数，调用方就不会等待该函数返回。

  ```v
  @[noreturn]
  fn forever() {
  	for {}
  }
  ```

- @[keep_args_alive]

  函数加上这个注解后，函数的指针参数在函数返回之前不会被GC释放。

  ```v
  @[keep_args_alive]
  fn C.calc_expr_after_delay(voidptr, int, voidptr) int
  ```
#### 变量注解

- @[c_extern]

  使用@ [c_extern]注解，在生成的C代码中，给全局变量增加extern关键字。

  具体内容可以参考：[变量章节](var.md)

### 自定义注解

实际上除了编译器内置的注解外，结构体，结构体字段，函数，枚举，联合类型的定义，都可以增加各种自定义注解，然后自己解析，自己使用。

注解的扩展性还是比较灵活的，目前结构体注解和结构体字段注解，已经可以通过$for编译时反射来获取所有的注解内容，具体内容可以参考：[编译时反射章节](comptime.md)。

```v
@[testing]
struct StructAttrTest {
	foo string [attr1;required] //结构体字段自定义注解
	bar int [attr1; attr2; required]
}

@[testing]
struct PubStructAttrTest {
	foo string
	bar int
}

@[testing]
type Name = string

@[testing]
type SumType = int | string | bool

@[testing]
enum EnumAttrTest {
	one
	two
}

@[testing]
enum PubEnumAttrTest {
	one
	two
}

@[attrone] 
@[attrtwo] //可以同时有多个注解
enum EnumMultiAttrTest {
	one
	two
}

@[attr1;attr2] //在同一个注解中括号内,注解用分号来区隔
fn fn_attribute() {
}

@[testing]
fn pub_fn_attribute() {
}

@[attrone]
@[attrtwo]
fn fn_multi_attribute() {
}

@[attrone]
@[attrtwo]
fn fn_multi_attribute_single() {
}
```
