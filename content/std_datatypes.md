## 数据类型

数据类型模块使用泛型实现了常用的抽象数据类型ADT：栈，队列，链表，双链表，堆，Btree。

### 栈

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

### 队列

常用操作：

```v

```

示例：

```v

```

### 链表

常用操作：

```v

```

示例：

```v

```

### 双链表

常用操作：

```v

```

示例：

```v

```

### 堆

队列的常用操作：

```v

```

示例：

```v

```
