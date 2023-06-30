## 数据类型

datatypes数据类型模块使用泛型实现了常用的抽象数据类型ADT：集合，栈，队列，链表，双链表，堆，Btree。

### 集合 Set

集合的常用操作：

```v
mut set := Set[string]{} //创建集合
s.add(element T) //添加元素
s.remove(element T) //删除元素
s.size() int //返回集合大小
s.exists(element T) bool //判断元素是否存在集合中
s.is_empty() bool //是否为空
s.copy() Set[T] Set[T]//拷贝集合
s.subset(Set[T]) bool //参数中的集合是否是s的子集
s.@union(Set[T]) Set[T]  //并集
s.intersection(Set[T]) Set[T] //交集
s1==s2 //判断集合是否相等，重载了==运算符
s3=s1-s2  //差集
```

### 栈 Stack

栈的常用操作：

```v
push() //压栈
pop() //出栈
peek() //返回栈尾
is_empty() //判断栈是否为空
len() //返回栈的长度
```

示例：

```v
module main

import datatypes as dt

fn main() {
	mut stack := dt.Stack[int]{} //基于泛型的栈
	println(stack.is_empty()) //判断栈是否为空
	stack.push(1) //压栈
	stack.push(2)
	stack.push(3)
	println(stack)
	println(stack.len()) //返回栈的长度
	e := stack.pop() ! //出栈,并返回出栈的元素,如果栈为空,则抛出错误
	println(e)
	println(stack)
	p := stack.peek() ! //返回栈尾,如果栈为空,则抛出错误
	println(p)
}
```

### 队列 Queue

常用操作：

```v

```

示例：

```v

```

### 链表 LinkedList

常用操作：

```v

```

示例：

```v

```

### 双链表 DoublyLinkedList

常用操作：

```v

```

示例：

```v

```

### 堆 Head

队列的常用操作：

```v

```

示例：

```v

```

B树 BTree

```v

```

