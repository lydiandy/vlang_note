## 内存管理

### 自动内存管理

因为V语言中限制没有全局变量,没有模块级变量,只有局部变量,变量只能在函数或者方法内部定义,所以当函数调用结束时,会自动回收函数栈内存

### 手动内存管理

V目前依赖libc

所以C标准库中手动管理内存的主要函数都可以使用,实现像C那样对内存的精确控制

- **malloc**

  C 库函数 **void \*malloc(size_t size)** 分配所需的内存空间，并返回一个指向它的指针

  ```C
  void *malloc(size_t size)
  ```

  **size** -- 内存块的大小，以字节为单位

- **calloc**

  C 库函数 **void \*calloc(size_t nitems, size_t size)** 分配所需的内存空间，并返回一个指向它的指针。**malloc** 和 **calloc** 之间的不同点是，malloc 不会设置内存为零，而 calloc 会设置分配的内存为零

  ```C
  void *calloc(size_t nitems, size_t size)
  ```

  - **nitems** -- 要被分配的元素个数

  - **size** -- 元素的大小

      

- **realloc**

  C 库函数 **void \*realloc(void \*ptr, size_t size)** 尝试重新调整之前调用 **malloc** 或 **calloc** 所分配的 **ptr** 所指向的内存块的大小

  ```c
  void *realloc(void *ptr, size_t size)
  ```

  - **ptr** -- 指针指向一个要重新分配内存的内存块，该内存块之前是通过调用 malloc、calloc 或 realloc 进行分配内存的。如果为空指针，则会分配一个新的内存块，且函数返回一个指向它的指针

  - **size** -- 内存块的新的大小，以字节为单位。如果大小为 0，且 ptr 指向一个已存在的内存块，则 ptr 所指向的内存块会被释放，并返回一个空指针

  - 该函数返回一个指针 ，指向重新分配大小的内存。如果请求失败，则返回 NULL

      

- **memcpy**

  C 库函数 **void \*memcpy(void \*str1, const void \*str2, size_t n)** 从存储区 **str2** 复制 **n** 个字符到存储区 **str1**。

  ```c
  void *memcpy(void *str1, const void *str2, size_t n)
  ```

  - **str1** -- 指向用于存储复制内容的目标数组，类型强制转换为 void* 指针

  - **str2** -- 指向要复制的数据源，类型强制转换为 void* 指针

  - **n** -- 要被复制的字节数

      

- **memmove**

  内存移动

  C 库函数 **void \*memmove(void \*str1, const void \*str2, size_t n)** 从 **str2** 复制 **n** 个字符到 **str1**，但是在重叠内存块这方面，memmove() 是比 memcpy() 更安全的方法。如果目标区域和源区域有重叠的话，memmove() 能够保证源串在被覆盖之前将重叠区域的字节拷贝到目标区域中，复制后源区域的内容会被更改。如果目标区域与源区域没有重叠，则和 memcpy() 函数功能相同

  ```c
  void *memmove(void *str1, const void *str2, size_t n)
  ```

  

- **free**

  释放指针内存