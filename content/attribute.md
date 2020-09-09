## 标注(attribute)

V语言可以针对结构体,结构体字段,函数/方法进行标注

标注的格式统一是:[标注名] 或 [标注名:标注扩展内容]

### 编译器内置标注

#### 结构体标注

结构体标注统一放在结构体定义前

- [typedef]

  参考:[结构体章节](struct.md)
  
- [ref_only]

  参考:[结构体章节](struct.md)

#### 结构体字段标注

结构体字段标注统一放在字段定义后

- [required]

  参考:[结构体章节-字段标注](struct.md)
  
- [json]

  参考:[json章节](json.md)
  
- [skip]

  参考:[json章节](json.md)
  
- [row]

  参考:[json章节](json.md)

#### 函数标注

函数标注统一放在函数定义之前

- [deprecated]

  参考:[函数章节](fn.md)
  
- [inline]

  参考:[函数章节](fn.md)

- [unsafe]

  参考:[unsafe章节](unsafe.md)

- [if debug]

  ```go
  
  ```

- [windows_stdcall]

  ```go
  
  ```

### 自定义标注

实际上除了编译器内置的标注外,结构体和函数的定义者可以增加各种自定义标注,然后自己解析,自己使用

标注的扩展性还是比较灵活的

目前结构体标注和结构体字段标注,已经可以通过$for编译时反射来获取所有的标注内容,具体内容可以参考:[编译时反射章节](crossplatform.md)

```rust
[testing]
struct StructAttrTest {
	foo string
	bar int
}

[testing]
pub struct PubStructAttrTest {
	foo string
	bar int
}

[testing]
enum EnumAttrTest {
	one
	two
}

[testing]
pub enum PubEnumAttrTest {
	one
	two
}


[attrone]
[attrtwo]
pub enum EnumMultiAttrTest {
	one
	two
}

[testing]
fn test_fn_attribute() {
	assert true
}

[testing]
pub fn test_pub_fn_attribute() {
	assert true
}

[attrone]
[attrtwo]
fn test_fn_multi_attribute() {
	assert true
}

[attrone; attrtwo]
fn test_fn_multi_attribute_single() {
	assert true
}

```

