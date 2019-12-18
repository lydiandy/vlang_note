## 泛型

目前泛型函数,泛型方法已经开发完成

泛型结构体还未开发完成

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



### 泛型函数

判断2个数组是否相等的泛型函数

```
//vlib/builtin/array.v
fn array_eq<T>(a1, a2 []T) bool {
   if a1.len != a2.len {
      return false
   }
   for i := 0; i < a1.len; i++ {
      if a1[i] != a2[i] {
         return false
      }
   }
   return true
}
```

### 泛型方法



### 泛型结构体