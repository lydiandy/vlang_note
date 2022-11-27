## maps字典模块

除了内置模块中包含了字典的常用函数，maps模块提供了更多的实用函数，详细内容可以参考vlib/maps，这些实用函数使用了泛型：

```v
pub fn filter[K, V](m map[K]V, f fn (key K, val V) bool) map[K]V

pub fn to_array[K, V, I](m map[K]V, f fn (key K, val V) I) []I

pub fn flat_map[K, V, I](m map[K]V, f fn (key K, val V) []I) []I

pub fn to_map[K, V, X, Y](m map[K]V, f fn (key K, val V) (X, Y)) map[X]Y

pub fn invert[K, V](m map[K]V) map[V]K

pub fn from_array[T](array []T) map[int]T
...
```

