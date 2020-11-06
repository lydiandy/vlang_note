## 内置sql支持

V语言在编译器中实现了对sql的支持

这种在语言编译器内实现sql支持的做法比较少见,会给编译器增加复杂性,

但是对于使用者来说,比库级别实现的orm框架要简单,好用许多

V内置sql的好处有:

- 针对不同的数据库,统一的一套语法,这样迁移到其他数据库变得更容易

- SQL语法内置在V语言的语法中,不需要学习其他的语法

- 安全,不可以通过注入生成SQL语句

- 编译时检查,语法错误在编译时就可以被捕捉到

- 简单易读,不再需要手工解析结果和构造对象

  

  ```v
  module main
  
  import sqlite
  
  struct User { // 数据库表对应到结构体,结构体名目前要求跟表名一致
  	id             int // 第一个字段必须是一个整型的id字段
  	age            int
  	name           string
  	is_customer    bool
  	skipped_string string [skip]
  }
  
  fn main() {
    //连接数据库,返回DB类型
  	// db := sqlite.connect(':memory:') or {  		//使用sqlite的内存数据库
  	db := sqlite.connect('./database.sqlite') or { // 使用文件数据库
  		panic(err)
  	}
  	// 定义表结构
  	db.exec('drop table if exists User')
  	db.exec("create table User (id integer primary key, age int default 0, name text default '', is_customer int default 0);")
  	// 插入数据
  	db.exec("insert into User (name, age) values ('Sam', 29)")
  	db.exec("insert into User (name, age) values ('Peter', 31)")
  	db.exec("insert into User (name, age, is_customer) values ('Kate', 30, 1)")
  	// v变量可以在sql中直接使用
  	name := 'Peter'
  	// 就像定义一个fn或struct那样,定义一个sql代码块,db是数据库连接变量,大括号中间只能是一条完整的sql语句
  	// 结构体直接作为表名,表名反倒不允许
  	result := sql db {
  		select count from User 
  	}
  	println(result)
  	//
  	nr_users1 := sql db {
  		select count from User where id == 1 
  	}
  	println(nr_users1)
  	//
  	nr_peters := sql db {
  		select count from User where id == 2 && name == 'Peter' 
  	}
  	println(nr_peters)
  	// sql中可以直接使用v变量
  	nr_peters2 := sql db {
  		select count from User where id == 2 && name == name 
  	}
  	println(nr_peters2)
  	nr_peters3 := sql db {
  		select count from User where name == name 
  	}
  	println(nr_peters3)
  	//
  	peters := sql db {
  		select from User where name == name 
  	}
  	println(peters[0].name)
  	// limit子句
  	one_peter := sql db {
  		select from User where name == name limit 1 
  	}
  	println(one_peter.name)
  	// offset子句
  	y := sql db {
  		select from User limit 2 offset 1
  	}
  	println(y)
  	users3 := sql db {
  		select from User where age == 29 || age == 31 
  	}
  	println(users3)
  	//
  	new_user := User{
  		name: 'New user'
  		age: 30
  	}
  	// insert语句,结构体直接作为表名,变量直接作为数据插入,后台自动把结构体和表结构对应起来
  	sql db {
  		insert new_user into User
  	}
  	// update语句
  	sql db {
  		update User set age = 31 where name == 'Kate'
  	}
  	new_age := 33
  	sql db {
  		update User set age = new_age, name = 'Kate N' where id == 3
  	}
  	// delete语句
  	// sql db {
  	// delete from User where id==4
  	// }
  }
  
  ```

更详细的SQL内容,可以参考[pg章节](./pg.md)