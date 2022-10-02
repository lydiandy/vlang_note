## maps字典模块

除了内置模块中包含了对字典的常用函数，maps模块也提供了更多的实用函数，详细内容可以参考vlib/maps源代码，这些实用函数使用泛型函数的方式提供：

- filter<K, V>(m map[K]V, f fn (K, V) bool) map[K]V

- to_array<K, V, I>(m map[K]V, f fn (K, V) I) []I

- flatten<K, V, I>(m map[K]V, f fn (K, V) []I) []I

  

- to_map<K, V, X, Y>(m map[K]V, f fn (K, V) (X, Y)) map[X]Y

  

- invert<K, V>(m map[K]V) map[V]K

  

- from_array<T>(a []T) map[int]T

