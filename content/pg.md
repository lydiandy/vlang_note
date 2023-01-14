## pg

### 安装postgres数据库

linux安装:

```shell
sudo dnf install postgresql-server postgresql-contrib
sudo systemctl enable postgresql
sudo systemctl start  postgresql
```

mac安装:

```shell
brew install postgresql
```

### C客户端库

使用的是postgres官方发布的C版本postgres客户端库。

如果没有安装postgresql数据库，则import db.pg时会报错：缺失<libpq-fe.h>。

具体API可以参考C头文件：<libpg-fe.h>。

```shell
brew install libpq
```

### 使用pg库

vlib/db/pg.v：

```v
pub struct Config { //数据库连接配置结构体
pub:
	host string 
	user string
	password string
	dbname string
}

pub struct DB { //DB对象
mut:
	conn &C.PGconn
}

pub struct Row { //查询结果集的行
pub mut:
	vals []string
}


pub fn connect(config pg.Config) ?DB //连接数据库函数,返回DB对象
//然后就是DB的各种方法:
pub fn (db DB) exec(query string) []pg.Row //执行SQL语句
pub fn (db DB) exec_one(query string) ?pg.Row //执行SQL语句,返回结果的第一行
pub fn (db DB) exec_param(query string, param string) []pg.Row //带1个参数
pub fn (db DB) exec_param2(query string, param, param2 string) []pg.Row //带2个参数
pub fn (db DB) exec_param_many(query string, params []string) []pg.Row //带多个参数
```

测试代码：

``` v
module main

import db.pg

struct User {
	id   int
	name string
	age  int
}

pub fn (user User) next() ?User {
	return user
}

fn main() {
	db_config := pg.Config{ //数据库连接配置
		host: 'localhost'
		user: 'postgres'
		password: ''
		dbname: 'test_db'
	}
	db := pg.connect(db_config) or { //连接数据库,如果连接失败,抛出错误
		panic('连接错误:err')
	}

	rows := db.exec('select * from users') or { panic(err) } //方式1:执行SQL语句字符串
	for row in rows {
		println(row)
	}

	users := sql db {
		select from User where id == 1
	}
	println(users)
	for u in users {
		println(u)
	}
}
```

这种db.的语法，目前还是比较原型阶段的，编译器内置实现了这几个SQL语句，都是解析器解析到pg.DB类型的变量才可以进行的语法，只针对pg有效。

```v
db.select
db.insert
db.update 
```

目前的pg库还是太简单，只能进行进行简单的查询，很多地方还需要完善。

但是pg库是基于postgres官方的C客户端库，可以把里面的C函数和结构体直接拿来使用，或者进行进一步的封装，开发起来还是比较快的。

目前需要注意的2个坑：

1. 使用db.select from User时，会转为小写复数的users表名，如果没有建好对应的表名，会报错。
2. 定义模型结构体时，如果结构体的字段多于对应数据库表的字段，执行db.exec()的时候不会返回结果，也没有提示。

### libpg客户端库使用参考

官方中文参考：http://www.postgres.cn/docs/14/libpq.html

教程：https://geek-docs.com/postgresql/postgresql-postgresql/postgresqlc.html







