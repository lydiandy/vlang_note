## 注解(attribute)

V语言可以针对结构体,结构体字段,函数/方法进行注解

注解的格式统一是:[注解名] 或 [注解名:注解扩展内容]

### 编译器内置注解

#### 结构体注解

结构体注解统一放在结构体定义前

- [typedef]

  参考:[结构体章节](struct.md)
  
- [heap]

  参考:[结构体章节](struct.md)

#### 结构体字段注解

结构体字段注解统一放在字段定义后

- [required]

  参考:[结构体章节-字段注解](struct.md)
  
- [json]

  参考:[json章节](json.md)
  
- [skip]

  参考:[json章节](json.md)
  
- [row]

  参考:[json章节](json.md)

#### 函数注解

函数注解统一放在函数定义之前

- [deprecated]

  参考:[函数章节](fn.md)
  
- [inline]

  参考:[函数章节](fn.md)

- [unsafe]

  参考:[unsafe章节](unsafe.md)

- [if debug]

  ```v
  
  ```

- [windows_stdcall]

  ```v
  
  ```

### 自定义注解

实际上除了编译器内置的注解外,结构体,函数,枚举的定义者可以增加各种自定义注解,然后自己解析,自己使用

注解的扩展性还是比较灵活的

目前结构体注解和结构体字段注解,已经可以通过$for编译时反射来获取所有的注解内容,具体内容可以参考:[编译时反射章节](crossplatform.md)

```v
[testing]
struct StructAttrTest {
	foo string
	bar int
}

[testing]
struct PubStructAttrTest {
	foo string
	bar int
}

[testing]
enum EnumAttrTest {
	one
	two
}

[testing]
enum PubEnumAttrTest {
	one
	two
}

[attrone]
[attrtwo]
enum EnumMultiAttrTest {
	one
	two
}

[testing]
fn fn_attribute() {
}

[testing]
fn pub_fn_attribute() {
}

[attrone]
[attrtwo]
fn fn_multi_attribute() {
}

[attrone]
[attrtwo]
fn fn_multi_attribute_single() {
}

```

