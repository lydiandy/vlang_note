## V和Go基本语法代码参照

V和Go的基本语法代码参照,方便熟悉Go的开发者,快速了解V:

### 模块

```v
module mymodule
```

```go
package mymodule
```

导入模块

```v
import os
import time as t
import time {now,Time}
import mymodule.submodule
```

```go
import (
	"os"
  "time"
  "mymodule/submodule"
)
```

### 函数

```v
fn add(x int, y int) int {
	return x + y
}
pub fn foo() (int, int) {
	return 2, 3
}
```

```go
func add(x int, y int) int {
    return x + y
}
func Foo() (int, int) {
    return 2, 3
}
```

### 结构体

```v
struct User {
	name string
pub mut:
	age  int
}
```

```go
type User struct {
	name string
	Age  int
}
```

### 方法

```v
fn (m &User) str() string {
	return 'name:$m.name,age:$m.age'
}
```

```go
func (u User) str() string {
	return "name:"+u.name+","+"age:"+strconv.Itoa(u.Age)
}
```

### hello world

```v
fn main() {
  println('Hello World!')
}
```

```go
package main
import "fmt"
func main() {
  fmt.Println("Hello World!")
}
```

### 切片初始化

```v
numbers := [1, 2, 3, 4]
```

```go
numbers := []int{1, 2, 3, 4}
```

### 追加元素到切片

```v
numbers << 5
```

```go
numbers = append(numbers, 5)
```



### 过滤切片中满足条件的元素

```v
even := numbers.filter(it % 2 == 0)
```

```go
even := make([]int, 0)
for _, num := range numbers {
  if num % 2 == 0 {
    even = append(even, num)
  }
}
```



### 检查切片是否包含某个元素

```v
contains := x in numbers
```

```go
contains := false
for _, num := range numbers {
        if num == x {
            contains = true
            break
        }
    }
```

### 读取文件

```v
import os
text := os.read_file(path)or{
  eprintln(err)
  return
}
```

```go
import (
    "io/ioutil"
    "log"
)
b, err := ioutil.ReadFile(path)
if err != nil {
        log.Println(err)
        return
}
text := string(b)
```

### 测试函数

```v
fn test_hello() {
    assert hello() == 'hello'
}
```

```go
package greeter_test
import (
    "testing"
)
func TestHello(t *testing.T) {
    if Hello() != "Hello" {
        t.Fatalf("Hello() failed")
```

