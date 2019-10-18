## 方法

跟go一样的方法,在函数名前面加接收者

接收者也是默认不可变,如果要修改接收者,也要加上mut

```
fn (u User) getName() string {}

fn (u mut User) setName(string) {}
```

