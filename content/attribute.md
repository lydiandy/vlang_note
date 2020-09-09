## 标注(attribute)

V语言可以针对结构体,结构体字段,函数/方法进行标注

标注的格式统一是:[标注名] 或 [标注名:标注扩展内容]

### 结构体标注

结构体标注统一放在结构体定义前

- [typedef]

  参考:[结构体章节](struct.md)
  
- [ref_only]

  参考:[结构体章节](struct.md)

### 结构体字段标注

结构体字段标注统一放在字段定义后

- [required]

  参考:[结构体章节-字段标注](struct.md)
  
- [json]

  参考:[json章节](json.md)
  
- [skip]

  参考:[json章节](json.md)
  
- [row]

  参考:[json章节](json.md)

### 函数标注

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