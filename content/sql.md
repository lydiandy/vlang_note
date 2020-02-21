## 内置SQL支持

目前仅为alpha阶段,当demo体验而已

V语言有一个内置的ORM,目前只支持postgres和mysql,后续支持sqlite

V ORM的好处有:

- 针对不同的数据库,统一的一套语法,这样迁移到其他数据库变得更容易
- SQL语法内置在V语言的语法中,不需要学习其他的语法
- 安全,不可以通过注入生成SQL语句
- 编译时检查,语法错误在编译时就可以被捕捉到
- 简单易读,不再需要手工解析结果和构造对象

```c
import pg
struct Customer { // 数据库表对应到结构体,结构体名目前要求跟表名一致
	id int // 第一个字段必须是一个整型的id字段
	name string
	nr_orders int
	country string
}

db := pg.connect(db_name, db_user) //连接数据库,返回DB类型

// select count(*) from Customer
nr_customers := db.select count from Customer //db.后面的就可以直接写SQL语句,并返回结果集,数组
println('number of all customers: $nr_customers')

// V syntax can be used to build queries
// db.select返回一个数组
uk_customers := db.select from Customer where country == 'uk' && nr_orders > 0
println(uk_customers.len)
for customer in uk_customers {
	println('$customer.id - $customer.name')
}

customer := db.select from Customer where id == 1 limit 1
println('$customer.id - $customer.name')

//插入数据到数据库
new_customer := Customer{name: 'Bob', nr_orders: 10}
db.insert(new_customer) 
```

更详细的SQL内容,可以参考[pg章节](./pg.md)