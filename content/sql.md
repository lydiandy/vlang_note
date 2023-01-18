## 内置sql支持

V语言编译器内置实现了类似sql语句的支持，在语法级别实现了一个轻量级的orm。

语法效果比库级别实现的orm框架要简单，好用许多。

这样实现的好处有：

- 针对不同的数据库，统一的一套语法，这样迁移到其他数据库变得更容易。

- SQL语法内置在V语言的语法中，不需要学习其他的语法。

- 安全，不可以通过注入生成SQL语句。

- 编译时检查，语法错误在编译时就可以被捕捉到。

- 简单易读，不再需要手工解析结果和构造对象。

基本步骤：

1. 定义结构体
2. 创建数据库连接
3. 使用sql代码块

### 定义结构体

基本的思路跟大部分orm框架一样：结构体对应表，结构体字段对应表字段。

```v
struct Foo {
    id          int         [primary; sql: serial]
    name        string      [nonull]
    created_at  time.Time   [sql_type: 'DATETIME']
    updated_at  string      [sql_type: 'DATETIME']
    deleted_at  time.Time
    children    []Child     [fkey: 'parent_id']
}

struct Child {
    id        int    [primary; sql: serial]
    parent_id int
    name      string
}
```

使用注解对结构体和字段进行标注

#### 结构体注解

- [table: 'name']

  默认情况，生成的数据库表名就是结构体名，大小写一致，没有加复数。

  也可以使用这个注解自定义表名

#### 字段注解

- [sql: 'name']

  自定义字段名

- [primary]

  标记字段为主键

- [unique]

  标记字段唯一，不可重复

- [unique: 'foo']

  将字段加入到unique group中

- [nonull]

  字段不允许为空值

- [default: 'sql default']

  设置字段的默认值

- [skip]

  跳过字段，不生成表字段

- [sql: type]

  字段V语言中的类型，其中有一个特别的类型：serial自增长。

  这个标注有点多余，感觉应该合并到sql_type注解中

- [sql_type: 'SQL TYPE']

  指定字段对应生成sql的类型

- [fkey: 'parent_id']

  指定外键的字段名

#### 主键类型

结构体中的主键可以是int类型，也可以是string类型。

int类型可以设置注解[sql:serial]，自动生成

string类型可以设置注解[default: 'gen_random_uuid()'; primary; sql_type: 'uuid']，自动生成

time类型可以设置注解[default: 'CURRENT_TIMESTAMP'; sql_type: 'TIMESTAMP']，自动生成

其中gen_random_uuid()，CURRENT_TIMESTAMP为pg数据库的内置函数，放在default注解中，会自动执行，并返回结果作为字段默认值。

```v
struct User {
	// id   int    [primary; sql: serial]
	id         string [default: 'gen_random_uuid()'; primary; sql_type: 'uuid']
	name       string [nonull; unique]
	age        int
  created_at string [default: 'CURRENT_TIMESTAMP'; sql_type: 'TIMESTAMP']
}
```

### 创建数据库连接

开始使用sql代码块前，要先创建数据库连接，然后将该连接给sql代码块使用。

```v
import db.sqlite
mut db := sqlite.connect('./database.sqlite') or { panic(err) }
```

### 使用sql代码块

#### 创建表

```v
sql db { //db就是创建连接返回的变量
	create table Foo
}
```

#### 删除表

```v
sql db {
	drop table Foo
}
```

#### 插入语句

```v
var := Foo{
    name:       'abc'
    created_at: time.now()
    updated_at: time.now().str()
    deleted_at: time.now()
    children: [
        Child{
            name: 'abc'
        },
        Child{
            name: 'def'
        },
    ]
}
//结构体名作为表名,变量作为数据插入,编译器根据结构体定义跟表结构对应起来
sql db {
    insert var into Foo //使用的是结构体名Foo，而不是表名
}
```

#### 更新语句

```v
sql db {
    update Foo set name = 'cde', updated_at = time.now() where name == 'abc'
}
```

#### 删除语句

```v
sql db {
    delete from Foo where id > 10
}
```

#### 查询语句

只有查询语句可以返回结果

```v
result := sql db {
    select from Foo where id == 1
}
println(typeof(result).name) //单条记录返回User对象，多条记录返回[]User数组
```

```v
result := sql db {
    select from Foo where id > 1 && name != 'lasanha' limit 5
}
```

```v
result := sql db {
    select from Foo where id > 1 order by id
}
```

指定字段返回：

目前在sql代码块中还不支持，只能通过orm_select_gen()方法动态构造。

```v
result := sql db {
  select id,name from User  //暂不支持
}
```

#### sql代码块错误处理

如果sql代码块返回错误，可以使用or代码块进行错误处理：

```v
users := sql db {
	select from User where id == 1
} or { println('there is an error: ${err}') }
```

完整例子：

```v
import db.pg

struct Member {
	id         string [default: 'gen_random_uuid()'; primary; sql_type: 'uuid']
	name       string
	created_at string [default: 'CURRENT_TIMESTAMP'; sql_type: 'TIMESTAMP']
}

fn main() {
	db := pg.connect(pg.Config{
		host: 'localhost'
		port: 5432
		user: 'user'
		password: 'password'
		dbname: 'dbname'
	}) or {
		println(err)
		return
	}

	defer {
		db.close()
	}

	sql db {
		create table Member
	}

	new_member := Member{
		name: 'John Doe'
	}

	sql db {
		insert new_member into Member
	}

	selected_member := sql db {
		select from Member where name == 'John Doe' limit 1
	}

	sql db {
		update Member set name = 'Hitalo' where id == selected_member.id
	}
}
```

#### 使用db.exec()方法

除了可以使用代码块语句，也可以直接使用db.exec()方法来执行各种sql语句

```v
db.exec('drop table if exists User') or {panic(err)}
db.exec('select id,name from User')  or {panic(err)}
```

#### sql事务

暂无

#### 关联查询

暂无

#### 聚合函数

```v
c := sql db {
	select count from User
}
```

### 生成sql语句函数

orm模块中提供了生成sql语句的函数，生成更复杂的sql语句。

```v
//create table
pub fn orm_table_gen()
//insert,update,delete
pub fn orm_stmt_gen()
//select
pub fn orm_select_gen()
```

参考测试代码：

```v
import orm

fn test_orm_stmt_gen_update() {
	query, _ := orm.orm_stmt_gen('Test', "'", .update, true, '?', 0, orm.QueryData{
		fields: ['test', 'a']
		data: []
		types: []
		kinds: []
	}, orm.QueryData{
		fields: ['id', 'name']
		data: []
		types: []
		kinds: [.ge, .eq]
	})
	assert query == "UPDATE 'Test' SET 'test' = ?0, 'a' = ?1 WHERE 'id' >= ?2 AND 'name' = ?3;"
}

fn test_orm_stmt_gen_insert() {
	query, _ := orm.orm_stmt_gen('Test', "'", .insert, true, '?', 0, orm.QueryData{
		fields: ['test', 'a']
		data: []
		types: []
		kinds: []
	}, orm.QueryData{})
	assert query == "INSERT INTO 'Test' ('test', 'a') VALUES (?0, ?1);"
}

fn test_orm_stmt_gen_delete() {
	query, _ := orm.orm_stmt_gen('Test', "'", .delete, true, '?', 0, orm.QueryData{
		fields: ['test', 'a']
		data: []
		types: []
		kinds: []
	}, orm.QueryData{
		fields: ['id', 'name']
		data: []
		types: []
		kinds: [.ge, .eq]
	})
	assert query == "DELETE FROM 'Test' WHERE 'id' >= ?0 AND 'name' = ?1;"
}

fn get_select_fields() []string {
	return ['id', 'test', 'abc']
}

fn test_orm_select_gen() {
	query := orm.orm_select_gen(orm.SelectConfig{
		table: 'test_table'
		fields: get_select_fields()
	}, "'", true, '?', 0, orm.QueryData{})

	assert query == "SELECT 'id', 'test', 'abc' FROM 'test_table';"
}

fn test_orm_select_gen_with_limit() {
	query := orm.orm_select_gen(orm.SelectConfig{
		table: 'test_table'
		fields: get_select_fields()
		has_limit: true
	}, "'", true, '?', 0, orm.QueryData{})

	assert query == "SELECT 'id', 'test', 'abc' FROM 'test_table' LIMIT ?0;"
}

fn test_orm_select_gen_with_where() {
	query := orm.orm_select_gen(orm.SelectConfig{
		table: 'test_table'
		fields: get_select_fields()
		has_where: true
	}, "'", true, '?', 0, orm.QueryData{
		fields: ['abc', 'test']
		kinds: [.eq, .gt]
		is_and: [true]
	})

	assert query == "SELECT 'id', 'test', 'abc' FROM 'test_table' WHERE 'abc' = ?0 AND 'test' > ?1;"
}

fn test_orm_select_gen_with_order() {
	query := orm.orm_select_gen(orm.SelectConfig{
		table: 'test_table'
		fields: get_select_fields()
		has_order: true
		order_type: .desc
	}, "'", true, '?', 0, orm.QueryData{})

	assert query == "SELECT 'id', 'test', 'abc' FROM 'test_table' ORDER BY '' DESC;"
}

fn test_orm_select_gen_with_offset() {
	query := orm.orm_select_gen(orm.SelectConfig{
		table: 'test_table'
		fields: get_select_fields()
		has_offset: true
	}, "'", true, '?', 0, orm.QueryData{})

	assert query == "SELECT 'id', 'test', 'abc' FROM 'test_table' OFFSET ?0;"
}

fn test_orm_select_gen_with_all() {
	query := orm.orm_select_gen(orm.SelectConfig{
		table: 'test_table'
		fields: get_select_fields()
		has_limit: true
		has_order: true
		order_type: .desc
		has_offset: true
		has_where: true
	}, "'", true, '?', 0, orm.QueryData{
		fields: ['abc', 'test']
		kinds: [.eq, .gt]
		is_and: [true]
	})

	assert query == "SELECT 'id', 'test', 'abc' FROM 'test_table' WHERE 'abc' = ?0 AND 'test' > ?1 ORDER BY '' DESC LIMIT ?2 OFFSET ?3;"
}

fn test_orm_table_gen() {
	query := orm.orm_table_gen('test_table', "'", true, 0, [
		orm.TableField{
			name: 'id'
			typ: typeof[int]().idx
			default_val: '10'
			attrs: [
				StructAttribute{
					name: 'primary'
				},
				StructAttribute{
					name: 'sql'
					has_arg: true
					arg: 'serial'
					kind: .plain
				},
			]
		},
		orm.TableField{
			name: 'test'
			typ: typeof[string]().idx
		},
		orm.TableField{
			name: 'abc'
			typ: typeof[i64]().idx
			default_val: '6754'
		},
	], sql_type_from_v, false) or { panic(err) }
	assert query == "CREATE TABLE IF NOT EXISTS 'test_table' ('id' SERIAL DEFAULT 10, 'test' TEXT, 'abc' INT64 DEFAULT 6754, PRIMARY KEY('id'));"

	alt_query := orm.orm_table_gen('test_table', "'", true, 0, [
		orm.TableField{
			name: 'id'
			typ: typeof[int]().idx
			default_val: '10'
			attrs: [
				StructAttribute{
					name: 'primary'
				},
				StructAttribute{
					name: 'sql'
					has_arg: true
					arg: 'serial'
					kind: .plain
				},
			]
		},
		orm.TableField{
			name: 'test'
			typ: typeof[string]().idx
		},
		orm.TableField{
			name: 'abc'
			typ: typeof[i64]().idx
			default_val: '6754'
		},
	], sql_type_from_v, true) or { panic(err) }
	assert alt_query == "IF NOT EXISTS (SELECT * FROM sysobjects WHERE name='test_table' and xtype='U') CREATE TABLE 'test_table' ('id' SERIAL DEFAULT 10, 'test' TEXT, 'abc' INT64 DEFAULT 6754, PRIMARY KEY('id'));"

	unique_query := orm.orm_table_gen('test_table', "'", true, 0, [
		orm.TableField{
			name: 'id'
			typ: typeof[int]().idx
			default_val: '10'
			attrs: [
				StructAttribute{
					name: 'primary'
				},
				StructAttribute{
					name: 'sql'
					has_arg: true
					arg: 'serial'
					kind: .plain
				},
			]
		},
		orm.TableField{
			name: 'test'
			typ: typeof[string]().idx
			attrs: [
				StructAttribute{
					name: 'unique'
				},
			]
		},
		orm.TableField{
			name: 'abc'
			typ: typeof[i64]().idx
			default_val: '6754'
		},
	], sql_type_from_v, false) or { panic(err) }
	assert unique_query == "CREATE TABLE IF NOT EXISTS 'test_table' ('id' SERIAL DEFAULT 10, 'test' TEXT, 'abc' INT64 DEFAULT 6754, PRIMARY KEY('id'), UNIQUE('test'));"

	mult_unique_query := orm.orm_table_gen('test_table', "'", true, 0, [
		orm.TableField{
			name: 'id'
			typ: typeof[int]().idx
			default_val: '10'
			attrs: [
				StructAttribute{
					name: 'primary'
				},
				StructAttribute{
					name: 'sql'
					has_arg: true
					arg: 'serial'
					kind: .plain
				},
			]
		},
		orm.TableField{
			name: 'test'
			typ: typeof[string]().idx
			attrs: [
				StructAttribute{
					name: 'unique'
					has_arg: true
					arg: 'test'
					kind: .string
				},
			]
		},
		orm.TableField{
			name: 'abc'
			typ: typeof[i64]().idx
			default_val: '6754'
			attrs: [
				StructAttribute{
					name: 'unique'
					has_arg: true
					arg: 'test'
					kind: .string
				},
			]
		},
	], sql_type_from_v, false) or { panic(err) }
	assert mult_unique_query == "CREATE TABLE IF NOT EXISTS 'test_table' ('id' SERIAL DEFAULT 10, 'test' TEXT, 'abc' INT64 DEFAULT 6754, /* test */UNIQUE('test', 'abc'), PRIMARY KEY('id'));"
}

fn sql_type_from_v(typ int) !string {
	return if typ in orm.nums {
		'INT'
	} else if typ in orm.num64 {
		'INT64'
	} else if typ in orm.float {
		'DOUBLE'
	} else if typ == orm.type_string {
		'TEXT'
	} else if typ == -1 {
		'SERIAL'
	} else {
		error('Unknown type ${typ}')
	}
}
```

更详细的代码可以参考内置的orm模块：vlib/orm。
