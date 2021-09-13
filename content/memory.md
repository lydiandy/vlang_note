## 内存管理

### 内存管理思路

V语言的内存管理还在开发中,目前的思路是这样:

1. 当编译器编译代码时,如果能够判断一部分变量确定不再被引用,编译器会自动处理,自动插入free()函数到生成的代码中,自动释放变量.

   

2. 如果编译器在编译时不能判断,会启用引用计数.

   

3. 开发者可以让编译器自动释放自定义结构体的内存,通过增加.free()方法.

   

4. 如果开发者真的希望,也可以在函数前加上[manualfree]注解,告诉编译器这个函数不需要自动内存控制,由开发者自己手工调用free()函数来管理内存的分配和释放

目前为止,你可以在编译时使用-autofree选项,试用编译器的自动释放内存功能,一旦这个内存管理功能开发完成后,就会变为编译器默认的选项.

V语言中限制没有全局变量,没有模块级变量,只有局部变量

变量只能在函数或者方法内部定义,所以当函数调用结束时,会自动回收函数栈内存

V的0.2版本发布后,增加了一个编译选项-autofree,可以实现自动释放内存

目前该选项还没有正式发布,不是编译器默认选项,需要人工添加,计划在0.3版本作为默认选项

### 可选GC

在V的内存管理没有成熟之前,可选GC算是一个备用方案 

V使用的是开源的,通用的,C的垃圾回收器: 

Boehm-Demers-Weiser Garbage Collector (bdwgc)

官网: https://www.hboehm.info/gc/

源代码: https://github.com/ivmai/bdwgc

boehm的发音是/bame/，是美式英语中一个从德文来的姓氏,是一个C的垃圾回收器

使用mark and sweep算法的增量式分代垃圾回收器


llvm, mono, gnu d compiler, gnu java compiler等也都使用该gc

一般的C代码中，只要将malloc, realloc替换成boehm相应的分配函数，再删除free调用，就带gc功能

#### 安装

- 安装libgc-dev包

  macos:

  ```shell
  brew install libgc
  ```

  linux:

  ```shell
  sudo apt-get install libgc-dev
  ```

  

- 也可手工下载发布的稳定版本: 

  https://github.com/ivmai/bdwgc/releases, 目前最新的稳定版是8系列

  下载完成后编译:

  ```shell
  cd bdwgc
  ./configure
  make -j
  make check
  ```

  编译完成后,把bdwgc目录复制到V源代码目录的thirdparty中

#### 使用

在编译的时候加上 -gc boehm就可以把GC加入到你自己的代码中

```shell
v -gc boehm xxx.v       // 带GC的编译
v -gc boehm run main.v  // 带GC的运行
v -prod -gc boehm main.v  // 生产编译带GC的代码
```

#### 编译后文件大小对比

最简单的测试代码:

```v
fn main() {
    println('from main')
}
```

带GC的大概增加300K左右的大小

|  v   | v -gc boehm | v -prod | v -prod -gc boehm |
| :--: | :---------: | :-----: | :---------------: |
| 209K |    509K     |   52K   |       351K        |

#### 代码中判断是否启用GC

一般的V代码很少用到

```v
$if gcboehm  {
	
}
```



### 手动内存管理

除了使用-autofree自动管理内存,也可以使用--manualfree手动管理内存

也可以在模块或函数,使用[manualfree]注解,针对某个具体模块或函数,进行手动管理内存,如果进行手动内存管理,需要自行调用变量的free()方法进行释放

```v
[manualfree] // 如果注解在模块上,该模块的所有函数都进行手动内存管理
module main

fn abc() {
	x := 'abc should be autofreed'
	println(x)
}

[manualfree] // 如果注解在函数/方法上,该函数内进行手动内存管理
fn xyz() {
	x := 'xyz should do its own memory management'
	println(x)
	x.free() // 手动释放
}

fn main() {
	abc()
	xyz()
}

```

V目前依赖libc

所以C标准库中手动管理内存的主要函数都可以使用,实现像C那样对内存的精确控制

- **malloc**

  C 库函数 **void \*malloc(size_t size)** 分配所需的内存空间，并返回一个指向它的指针

  ```v
  void *malloc(size_t size)
  ```

  **size** -- 内存块的大小，以字节为单位

- **calloc**

  C 库函数 **void \*calloc(size_t nitems, size_t size)** 分配所需的内存空间，并返回一个指向它的指针。**malloc** 和 **calloc** 之间的不同点是，malloc 不会设置内存为零，而 calloc 会设置分配的内存为零

  ```v
  void *calloc(size_t nitems, size_t size)
  ```

  - **nitems** -- 要被分配的元素个数

  - **size** -- 元素的大小

      

- **realloc**

  C 库函数 **void \*realloc(void \*ptr, size_t size)** 尝试重新调整之前调用 **malloc** 或 **calloc** 所分配的 **ptr** 所指向的内存块的大小

  ```v
  void *realloc(void *ptr, size_t size)
  ```

  - **ptr** -- 指针指向一个要重新分配内存的内存块，该内存块之前是通过调用 malloc、calloc 或 realloc 进行分配内存的。如果为空指针，则会分配一个新的内存块，且函数返回一个指向它的指针

  - **size** -- 内存块的新的大小，以字节为单位。如果大小为 0，且 ptr 指向一个已存在的内存块，则 ptr 所指向的内存块会被释放，并返回一个空指针

  - 该函数返回一个指针 ，指向重新分配大小的内存。如果请求失败，则返回 NULL

      

- **memcpy**

  C 库函数 **void \*memcpy(void \*str1, const void \*str2, size_t n)** 从存储区 **str2** 复制 **n** 个字符到存储区 **str1**。

  ```v
  void *memcpy(void *str1, const void *str2, size_t n)
  ```

  - **str1** -- 指向用于存储复制内容的目标数组，类型强制转换为 void* 指针

  - **str2** -- 指向要复制的数据源，类型强制转换为 void* 指针

  - **n** -- 要被复制的字节数

      

- **memmove**

  内存移动

  C 库函数 **void \*memmove(void \*str1, const void \*str2, size_t n)** 从 **str2** 复制 **n** 个字符到 **str1**，但是在重叠内存块这方面，memmove() 是比 memcpy() 更安全的方法。如果目标区域和源区域有重叠的话，memmove() 能够保证源串在被覆盖之前将重叠区域的字节拷贝到目标区域中，复制后源区域的内容会被更改。如果目标区域与源区域没有重叠，则和 memcpy() 函数功能相同

  ```v
  void *memmove(void *str1, const void *str2, size_t n)
  ```

  

- **free**

  释放指针内存