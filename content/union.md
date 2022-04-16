## 联合体

跟C的联合体类型用法完全一致,联合体的定义方式跟结构体是一样的,

只是把关键字struct改为union,不过在V语言中比较少用到.

### 联合体定义

```v
union MyAnything {
mut:
	bytes [10]u8
	xu64  u64
	xu32  u32
	xu16  u16
	xint  int
}

fn main() {
	mut x := MyAnything{}
	x.xint = 1234
	for i, b in x.bytes {
		println('x.bytes[$i]: $b.hex()')
	}
}

```

### 联合体方法

可以像结构体那样,给联合体添加方法

```v
union MyAnything {
mut:
	bytes [10]u8
	xu64  u64
	xu32  u32
	xu16  u16
	xint  int
}

pub fn (m MyAnything) str() string {
	return 'from union'
}

fn main() {
	mut x := MyAnything{}
	x.xint = 1234
	println(x.str())
}

```

