## 接口

接口跟go一样,没有接口实现的关键字,

鸭子类型:只要结构体实现了接口定义的方法,就满足该接口的使用

```
struct Dog {}
struct Cat {}

fn (d Dog) speak() string {
	return 'woof'
}

fn (c Cat) speak() string {
	return 'meow' 
}

interface Speaker {
	speak() string
}

fn perform(s Speaker) {
	println(s.speak())
}

dog := Dog{}
cat := Cat{}
perform(dog) // "woof" 
perform(cat) // "meow" 
```

