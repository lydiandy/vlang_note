## json模块

### 使用

v标准库的json模块有点特别:

1. 基于C语言的cJSON库实现

2. 没有使用运行时反射,性能会更好

3. 解析JSON功能在编译器内部实现,内置支持JSON

具体如何使用json.encode()和json.decode()函数,[参考内置JSON章节](json.md)

### 实现原理

下面主要分析如何实现内置JSON的基本原理:



### 