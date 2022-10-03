## array数组模块

除了内置模块中包含了数组的常用函数，arrays模块提供了更多的实用函数，详细内容可以参考vlib/arrays，这些实用函数使用了泛型：

```v
pub fn min<T>(a []T) ?T
pub fn max<T>(a []T) ?T
pub fn idx_min<T>(a []T) ?int
pub fn idx_max<T>(a []T) ?int
pub fn merge<T>(a []T, b []T) []T
...
```

