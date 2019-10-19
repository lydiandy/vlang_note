# 结构体



### 结构体定义

```
struct Point {
	x int
	y int
}

p := Point{
	x: 10
	y: 20
}
println(p.x) // 结构体字段通过点号来访问
```

结构体被分配到内存的栈中,引用类型

取结构体地址:&

```
p := &Point{10, 10}
println(p.x)
```

空结构体:

```
struct {}
```

结构体占用内存:

sizeof( )可以返回结构体占用内存字节大小

### 结构体字段

结构体字段默认也是不可变

结构体字段的可变性和访问控制,参考[访问控制](access_controll.md)章节

### 结构体访问控制

结构体目前默认都是pub公共的,不支持模块级别的结构体

### 结构体的组合

struct 可以组合, 用来实现继承的效果(目前还没有实现)

```
struct Button {
	Widget
	title string
}

button := new_button('Click me')
button.set_pos(x, y)
```

### 泛型结构体

有计划要实现,目前还没有实现全:

```
struct Repo<T> {
	db DB
}

fn new_repo<T>(db DB) Repo<T> {
	return Repo<T>{db: db}
}

// This is a generic function. V will generate it for every type it's used with.
fn (r Repo<T>) find_by_id(id int) ?T {
	table_name := T.name // in this example getting the name of the type gives us the table name
	return r.db.query_one<T>('select * from $table_name where id = ?', id)
}

db := new_db()
users_repo := new_repo<User>(db)
posts_repo := new_repo<Post>(db)
user := users_repo.find_by_id(1)?
post := posts_repo.find_by_id(1)?
```

### 类型别名

自定义类型别名

```
type myint int
```

函数类型:

相同的函数签名,表示这一类函数,可以定义为函数类型

```
type mid_fn fn(int) int
```

